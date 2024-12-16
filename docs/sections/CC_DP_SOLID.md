### Clean Code: Design Principles (SOLID)

The idea behind "Clean Code" is wonderfully expressed in Robert C. Martin's book *Clean Code: A Handbook of Agile Software Craftsmanship*. One of the core aspects of clean code is following **SOLID** principles, which consist of five fundamental design guidelines:

1. **Single Responsibility Principle (SRP)**:  
   "There should never be more than one reason for a class to change." Each class should have only one responsibility.  
   ([Reference](https://web.archive.org/web/20150202200348/http://www.objectmentor.com/resources/articles/srp.pdf))

2. **Open-Closed Principle (OCP)**:  
   "Software entities ... should be open for extension, but closed for modification." Extend behavior through inheritance or composition rather than altering existing code.  
   ([Reference](https://web.archive.org/web/20150905081105/http://www.objectmentor.com/resources/articles/ocp.pdf))

3. **Liskov Substitution Principle (LSP)**:  
   "Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it." This ensures that derived classes enhance, not break, functionality.  
   ([Reference](https://web.archive.org/web/20150905081110/http://www.objectmentor.com/resources/articles/isp.pdf))

4. **Interface Segregation Principle (ISP)**:  
   "Many client-specific interfaces are better than one general-purpose interface." Classes should not be forced to implement methods they don’t use.

5. **Dependency Inversion Principle (DIP)**:  
   "Depend upon abstractions, [not] concretions." High-level modules should rely on abstractions, not on low-level implementation details.

These principles help developers write code that is flexible, maintainable, and easier to refactor. By incorporating SOLID into everyday coding practices, teams can reduce technical debt, improve collaboration, and ensure the longevity of their projects.

---

[Prev: The Source Code](./The_Source_Code.md) | [Back to Index](../README.md) | [Next: Coming soon](https://github.com/gnespolino)

---
## License
This repository is open-source under the [MIT License](/LICENSE.md).
Happy coding! 🎉
##### An html version of this file is available [HERE](https://gnespolino.github.io/devhandbook/index.html)