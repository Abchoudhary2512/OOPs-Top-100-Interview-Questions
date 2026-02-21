##  Beginner Level (Core OOP Practice)

1. **Create a `BankAccount` class**

   * Attributes: accountNumber, balance
   * Methods: deposit(), withdraw(), checkBalance()

2. **Create a `Student` class**

   * Store name, roll number, and marks
   * Add method to calculate average

3. **Create a `Rectangle` class**

   * Methods to calculate area and perimeter

4. **Implement Encapsulation**

   * Make attributes private and use getters/setters

5. **Create a `Car` class**

   * Add constructor and display details method

6. **Create a `Book` class**

   * Add multiple constructors (default & parameterized)

---

##  Inheritance & Polymorphism

7. **Create a `Vehicle` base class**

   * Derive `Car` and `Bike`
   * Override a method like start()

8. **Shape Hierarchy**

   * Base class `Shape`
   * Derived classes `Circle`, `Rectangle`
   * Implement area() method (runtime polymorphism)

9. **Employee Management**

   * Base class `Employee`
   * Derived classes `Manager`, `Developer`
   * Override calculateSalary()

10. **Demonstrate Method Overloading**

* Create a class with multiple add() methods

11. **Demonstrate Method Overriding**

* Use parent and child class with same method name

---

##  Abstract Class & Interface Based

12. **Create an abstract class `Animal`**

* Abstract method makeSound()
* Implement in `Dog`, `Cat`

13. **Create an Interface `Playable`**

* Implement in classes `Football`, `Guitar`

14. **Payment System**

* Interface `Payment`
* Classes: `CreditCardPayment`, `UPIPayment`

---

##  Intermediate Level (Real-World Design)

15. **Library Management System**

* Classes: Book, Member, Library
* Issue and return books

16. **Online Shopping Cart**

* Product, Cart, User
* Add/remove products and calculate total

17. **ATM Simulation**

* User authentication
* Deposit, withdraw, check balance

18. **Hospital Management**

* Patient, Doctor, Appointment classes

19. **Movie Ticket Booking System**

* Movie, Theater, Ticket, Booking classes

20. **Design a Parking Lot System**

* Vehicle types
* Parking slots
* Fee calculation

---
##  Advanced Level (Real-World Design)

21. Design a File System
* Create a system with files and folders where folders can contain other files or folders.
* Use inheritance and the Composite pattern to manage hierarchy and size calculation.


22. Design a Ride Sharing System
* Build a system where users request rides and drivers are assigned automatically.
* Implement fare calculation, ride tracking, and payment handling using good OOP design.


23. Thread-Safe Singleton Logger
* Create a logger class that allows only one object in the entire program.
* Ensure it is thread-safe and prevents copying or multiple instances.


24. Design a Notification System
* Create a system that sends notifications via Email, SMS, or Push.
* Design it so new notification types can be added without modifying existing code.


25. Design a Smart Pointer from Scratch
* Implement your own smart pointer that automatically manages memory.
* Support ownership handling and prevent memory leaks using RAII principles.
