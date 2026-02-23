## How I Tricked Intel's Firmware into Giving Me 5 GHz Wi-Fi AP

*Or: a journey through self-managed regulatory domains, MCC trust hierarchies, and the art of lying to a kernel driver.*

> This article is a detour from the usual software topics — it's about Linux systems, kernel driver internals, and firmware behaviour. If you're a backend developer who sometimes runs their own infrastructure, or just someone who enjoys a good debugging story, read on.

---

### The Setup

The actual problem I was solving: I needed Wi-Fi coverage in my home office. Ethernet cable: yes. Wi-Fi signal from the router in the other room: barely enough for a laptop, not enough for a phone. So my phone kept falling back to mobile data. A 2.4 GHz access point fixed it immediately — `hostapd`, a bridged ethernet port, done in 20 minutes.

Then I noticed 5 GHz wasn't working. I didn't *need* 5 GHz. But I wanted to understand why it didn't work. That question cost me four hours.

This article is about those four hours. It's not a tutorial — the 2.4 GHz solution was already good enough. It's about curiosity, and about pushing something that was deliberately designed not to work in this way until it did.

I run a desktop machine with an Intel Killer Wi-Fi 6E AX1675x card, managed by `hostapd`, bridged to a dedicated ethernet port. 2.4 GHz was already working. The goal was to understand why 5 GHz wasn't — and fix it.

---

### First Obstacle: All 5 GHz Channels Are Blocked

I changed `hw_mode=a` and `channel=149` in `/etc/hostapd/hostapd.conf` and restarted the service. hostapd refused to start. I ran `iw list` to check what the card thought it could do:

```
Frequencies:
    * 5745 MHz [149] (disabled)
    * 5765 MHz [153] (disabled)
    ...
```

Every single 5 GHz channel had the `(no IR)` flag. "No Initiate Radiation" — the card refuses to transmit on those frequencies. The standard reason is regulatory: your country profile hasn't approved those channels.

I tried the obvious fix:

```bash
sudo iw reg set ES
```

Nothing changed. Still `(no IR)`. I double-checked the global regulatory domain:

```
global
country ES: DFS-ETSI
    ...
    * 5735 - 5835 @ 80 (23)  # channels 149-165, no special flags
```

The global domain said ES, channels 149+ should be fine. And yet the card disagreed.

Looking more carefully at `iw reg get`:

```
global
country ES: DFS-ETSI
    ...

phy#0
country 00: DFS-UNSET
    ...
```

There it was. The card had its own regulatory state, completely independent of the system. And it was stuck on `country 00` — the most restrictive profile, where almost nothing is allowed.

---

### Root Cause: The Intel Self-Managed Regulatory Domain

This is where things get interesting. The Intel AX210 / Killer AX1675x uses a Linux kernel feature called `REGULATORY_WIPHY_SELF_MANAGED`. It means the card manages its own regulatory domain through its firmware, and it completely ignores whatever you set via `iw reg set`. The kernel's wireless regulatory infrastructure — CRDA, `iw reg set`, `country_code` in hostapd — simply does not apply.

The firmware uses an internal mechanism called MCC (Mobile Country Code) with a trust hierarchy. Different sources for the country code get different trust levels:

| Source | Level | Behaviour |
|--------|-------|-----------|
| `MCC_SOURCE_WIFI_SCAN` | 7 (trusted) | Firmware learns country from 802.11d Country IEs received during scan |
| `MCC_SOURCE_HOST_SET` | 6 (untrusted) | Firmware receives country hint from host (e.g. `country_code=ES` in hostapd) |

The critical detail: `HOST_SET` is **untrusted**. When the firmware receives a country code via this channel, it **ignores it** and returns `country 00`. The only way to get the firmware to accept a country code is via `WIFI_SCAN` — by actually scanning and receiving a Country IE from a nearby access point.

This is Intel's way of preventing hosts from lying about their location to unlock restricted frequencies. Frustrating when you're legitimate, but understandable.

---

### The Chain of Failed Attempts

**Attempt 1: Scan before starting hostapd.** I wrote a pre-start script that scanned in managed mode, waited for country ES, then brought the interface down for hostapd to use. Result: bringing the interface DOWN resets the firmware MCC state back to `country 00`. Of course.

**Attempt 2: Don't bring the interface down.** Keep `wlp4s0` up, let hostapd take over. Result: hostapd switched the interface from `managed` to `AP` mode via `NL80211_CMD_SET_INTERFACE`. The firmware saw the mode change on the primary interface, decided that was a new context, and reset the country code to `00`.

**Attempt 3: Use `country_code=ES` in hostapd.conf.** This goes through `NL80211_CMD_REQ_SET_REG`, which triggers `MCC_SOURCE_HOST_SET` — the untrusted source. The firmware receives the hint and ignores it. Worse: this overrides whatever trusted state was already there. `country_code=ES` in hostapd *actively prevents* the fix.

---

### The Key Insight: Virtual Interfaces

If the problem was that switching `wlp4s0` from managed to AP mode triggers a firmware reset, what if we didn't switch `wlp4s0` at all?

Linux allows creating virtual interfaces on top of a physical interface. You can have `wlp4s0` stay in managed mode and create a separate `wlap0` that runs in AP mode — both on the same physical radio (`phy#0`).

The hypothesis: switching a *virtual* interface's type doesn't trigger the firmware MCC reset, because the firmware state is tied to the physical phy, not the logical interface.

```bash
iw dev wlp4s0 interface add wlap0 type __ap
```

Then tell hostapd to use `wlap0` instead of `wlp4s0`. Country code: still ES. hostapd started.

And then it hit `EBUSY`.

---

### The EBUSY in HT_SCAN

```
wlap0: interface state UNINITIALIZED->HT_SCAN
nl80211: Failed to set channel (freq=5745): ret=-16 (Device or resource busy)
```

When `[HT40+]` is in `ht_capab`, hostapd performs a coexistence scan before enabling the AP — it checks for overlapping 40 MHz BSSes (HT_SCAN state). This scan conflicted with `wlp4s0` still being active in managed mode, violating the interface combination rule:

```
#{ managed } <= 1, #{ AP } <= 1, total <= 3, #channels <= 1
```

Two managed interfaces doing things simultaneously on the same phy: EBUSY.

The fix was a single hostapd directive:

```
obss_interval=0
```

This disables the 40 MHz coexistence scan. hostapd skips HT_SCAN entirely and goes straight to ENABLED.

---

### The Phy Numbering Trap

During testing I reloaded the driver:

```bash
sudo rmmod iwlmvm && sudo rmmod iwlwifi && sudo modprobe iwlwifi
```

The card came back as `phy#1` instead of `phy#0`. My script had hardcoded `phy#0` in its awk parsing. Everything broke silently.

Fix: derive phy numbers dynamically:

```bash
PHY_NUM=$(iw dev "$IFACE" info 2>/dev/null | awk '/wiphy/{print $2}')
COUNTRY=$(iw reg get | awk "/phy#${PHY_NUM} /{f=1} f && /country/{print \$2; exit}" | tr -d ':')
```

Never hardcode phy numbers. Ever.

---

### The Final Solution

The working approach is elegantly perverse: trick the firmware into giving us 5 GHz by doing a real Wi-Fi scan, then keep the managed interface alive as a silent chaperone while the virtual AP interface does the actual work.

**`/usr/local/sbin/wifi-reg-update.sh`** (runs as `ExecStartPre`):

```bash
#!/bin/bash
IFACE=wlp4s0
APIF=wlap0
MAX_WAIT=20

iw dev "$APIF" del 2>/dev/null
sleep 1

CURRENT_TYPE=$(iw dev "$IFACE" info 2>/dev/null | awk '/type/{print $2}')
if [ "$CURRENT_TYPE" != "managed" ]; then
    ip link set "$IFACE" down
    iw dev "$IFACE" set type managed 2>/dev/null
fi
ip link set "$IFACE" up
sleep 4

# Trigger MCC_SOURCE_WIFI_SCAN → country ES from router's 802.11d IE
iw dev "$IFACE" scan > /dev/null 2>&1
sleep 5

PHY_NUM=$(iw dev "$IFACE" info 2>/dev/null | awk '/wiphy/{print $2}')
for i in $(seq 1 $MAX_WAIT); do
    COUNTRY=$(iw reg get | awk "/phy#${PHY_NUM} /{f=1} f && /country/{print \$2; exit}" | tr -d ':')
    if [ -n "$COUNTRY" ] && [ "$COUNTRY" != "00" ]; then
        logger -t wifi-reg-update "Regulatory domain: $COUNTRY"
        break
    fi
    sleep 1
done

# wlp4s0 stays UP in managed mode to preserve regulatory during mode switch
iw dev "$IFACE" interface add "$APIF" type __ap 2>/dev/null || true
sleep 5
```

**`/etc/hostapd/hostapd.conf`** (key parts):

```
interface=wlap0          # virtual AP interface, not wlp4s0
hw_mode=a
channel=149
ieee80211n=1
ieee80211ac=1
obss_interval=0          # disable HT_SCAN → no EBUSY
ht_capab=[HT40+][SHORT-GI-20][SHORT-GI-40]
vht_oper_chwidth=1
vht_oper_centr_freq_seg0_idx=155   # 80 MHz center: ch 149+153+157+161
# NO country_code — would trigger MCC_SOURCE_HOST_SET → country 00
```

**`/etc/systemd/system/hostapd.service.d/pre-scan.conf`**:

```ini
[Service]
ExecStartPre=/usr/local/sbin/wifi-reg-update.sh
```

After a cold boot:

```
wlap0: interface state UNINITIALIZED->HT_SCAN
wlap0: AP-ENABLED
channel 149 (5745 MHz), width: 80 MHz, center1: 5775 MHz
```

80 MHz at 5745 MHz. Working.

---

### Why This Works (The Full Picture)

Five conditions, all independently necessary:

1. **Scan in managed mode** → router broadcasts `Country: ES` IE → firmware receives it via `MCC_SOURCE_WIFI_SCAN` (trust 7) → channels 149–173 lose `(no IR)`.
2. **Keep `wlp4s0` UP** → if any interface on the phy goes DOWN, firmware reverts to `country 00`.
3. **Use `wlap0` as AP interface** → switching a virtual interface's type doesn't trigger MCC reset; switching the primary does.
4. **No `country_code` in hostapd.conf** → would trigger `MCC_SOURCE_HOST_SET` (trust 6, untrusted), overriding the scan result.
5. **`obss_interval=0`** → disables HT_SCAN coexistence probe, avoiding EBUSY with two concurrent managed interfaces.

Remove any one of these and it fails — in a different and confusing way each time.

---

### The Role of AI in This Debugging Session

I used Claude (via Claude Code CLI) throughout. The AI wasn't magic — it didn't know the answer upfront, and it suggested some invalid hostapd directives (`phyname=`, `noscan=`) that wasted time. But it contributed meaningfully:

*   Recognised the `REGULATORY_WIPHY_SELF_MANAGED` flag from `iw list` output and explained its implications
*   Hypothesised the MCC trust hierarchy and the `WIFI_SCAN` vs `HOST_SET` distinction
*   Suggested the virtual interface approach after the mode-change reset was identified
*   Caught the hardcoded `phy#0` issue before it became a production problem
*   Connected `obss_interval=0` to the specific EBUSY failure mode in HT_SCAN

The collaboration pattern that worked: I ran commands and posted raw output; the AI interpreted the kernel internals and proposed hypotheses; I tested on real hardware. The AI provided the mental model; I provided the ground truth.

---

### Takeaways

*   `iw reg set` and `country_code` in hostapd are essentially useless for Intel AX210/AX1675x. Don't bother.
*   The only way in is `MCC_SOURCE_WIFI_SCAN` — which means you need a nearby AP broadcasting Country IEs.
*   Virtual interfaces are your friend when working around firmware constraints tied to primary interface state.
*   `obss_interval=0` is underdocumented but important for multi-interface setups on single-phy cards.
*   Never hardcode phy numbers.

