### 61. What are design principles (SOLID) in OOP?

**SOLID** is an acronym for five fundamental design principles introduced by Robert C. Martin that make software designs more **understandable, flexible, and maintainable**.

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <memory>

// ==================== S: Single Responsibility Principle (SRP) ====================
// A class should have only one reason to change - one responsibility

// BAD: Multiple responsibilities
class BadReport {
private:
    std::string content;
    
public:
    void generateReport() { /* generate content */ }
    void saveToFile() { /* file I/O */ }
    void printReport() { /* printing logic */ }
    void sendEmail() { /* email logic */ }
};

// GOOD: Separated responsibilities
class Report {
private:
    std::string content;
    
public:
    void generate() { /* generate content */ }
    std::string getContent() const { return content; }
};

class ReportSaver {
public:
    void saveToFile(const Report& report, const std::string& filename) {
        // File saving logic
    }
};

class ReportPrinter {
public:
    void print(const Report& report) {
        // Printing logic
    }
};

// ==================== O: Open/Closed Principle (OCP) ====================
// Classes should be open for extension but closed for modification

// BAD: Modifying existing code for new features
class BadDiscountCalculator {
public:
    double calculate(const std::string& customerType, double amount) {
        if (customerType == "regular") {
            return amount * 0.9;  // 10% discount
        } else if (customerType == "premium") {
            return amount * 0.8;  // 20% discount
        }
        // Need to modify this function for new customer types!
        return amount;
    }
};

// GOOD: Extensible design
class DiscountStrategy {
public:
    virtual ~DiscountStrategy() = default;
    virtual double calculate(double amount) const = 0;
};

class RegularDiscount : public DiscountStrategy {
public:
    double calculate(double amount) const override {
        return amount * 0.9;
    }
};

class PremiumDiscount : public DiscountStrategy {
public:
    double calculate(double amount) const override {
        return amount * 0.8;
    }
};

class VIPDiscount : public DiscountStrategy {  // New type - no existing code modified
public:
    double calculate(double amount) const override {
        return amount * 0.7;
    }
};

class DiscountCalculator {
public:
    double calculate(const DiscountStrategy& strategy, double amount) {
        return strategy.calculate(amount);
    }
};

// ==================== L: Liskov Substitution Principle (LSP) ====================
// Derived classes must be substitutable for their base classes

// BAD: Violates LSP
class Bird {
public:
    virtual void fly() {
        std::cout << "Flying" << std::endl;
    }
};

class Penguin : public Bird {  // Penguins can't fly!
public:
    void fly() override {
        throw std::logic_error("Penguins can't fly!");  // Violates LSP
    }
};

// GOOD: Follows LSP
class Bird2 {
public:
    virtual void move() = 0;
};

class FlyingBird : public Bird2 {
public:
    void fly() { std::cout << "Flying" << std::endl; }
    void move() override { fly(); }
};

class SwimmingBird : public Bird2 {
public:
    void swim() { std::cout << "Swimming" << std::endl; }
    void move() override { swim(); }
};

// ==================== I: Interface Segregation Principle (ISP) ====================
// (Detailed in question 70)

// ==================== D: Dependency Inversion Principle (DIP) ====================
// Depend on abstractions, not concretions

// BAD: High-level module depends on low-level module
class EmailSender {
public:
    void sendEmail(const std::string& message) {
        std::cout << "Sending email: " << message << std::endl;
    }
};

class BadNotification {
private:
    EmailSender emailSender;  // Direct dependency
    
public:
    void notify(const std::string& message) {
        emailSender.sendEmail(message);
    }
};

// GOOD: Depend on abstraction
class MessageSender {
public:
    virtual ~MessageSender() = default;
    virtual void send(const std::string& message) = 0;
};

class EmailSender2 : public MessageSender {
public:
    void send(const std::string& message) override {
        std::cout << "Sending email: " << message << std::endl;
    }
};

class SMSSender : public MessageSender {
public:
    void send(const std::string& message) override {
        std::cout << "Sending SMS: " << message << std::endl;
    }
};

class Notification {
private:
    std::unique_ptr<MessageSender> sender;  // Depends on abstraction
    
public:
    Notification(std::unique_ptr<MessageSender> s) : sender(std::move(s)) {}
    
    void notify(const std::string& message) {
        sender->send(message);
    }
};

// ==================== Main demonstration ====================
int main() {
    std::cout << "=== SOLID Principles Demonstration ===" << std::endl;
    
    // OCP Example
    DiscountCalculator calc;
    RegularDiscount regular;
    PremiumDiscount premium;
    VIPDiscount vip;
    
    std::cout << "Regular discount: $" << calc.calculate(regular, 100) << std::endl;
    std::cout << "Premium discount: $" << calc.calculate(premium, 100) << std::endl;
    std::cout << "VIP discount: $" << calc.calculate(vip, 100) << std::endl;
    
    // DIP Example
    auto notification = Notification(std::make_unique<EmailSender2>());
    notification.notify("Hello SOLID!");
    
    auto smsNotification = Notification(std::make_unique<SMSSender>());
    smsNotification.notify("Hello via SMS!");
    
    return 0;
}
```

---

### 62. What is dependency injection?

**Dependency Injection (DI)** is a design pattern where **objects receive their dependencies from external sources** rather than creating them internally. It implements the Dependency Inversion Principle and makes code more testable and flexible.

```cpp
#include <iostream>
#include <memory>
#include <string>

// ==================== Without Dependency Injection ====================
class Logger {
public:
    void log(const std::string& message) {
        std::cout << "LOG: " << message << std::endl;
    }
};

class UserService {
private:
    Logger logger;  // Direct dependency - created internally
    
public:
    void createUser(const std::string& name) {
        // Business logic...
        logger.log("User created: " + name);
    }
};

// ==================== With Dependency Injection ====================

// Abstraction
class ILogger {
public:
    virtual ~ILogger() = default;
    virtual void log(const std::string& message) = 0;
};

// Concrete implementations
class ConsoleLogger : public ILogger {
public:
    void log(const std::string& message) override {
        std::cout << "CONSOLE: " << message << std::endl;
    }
};

class FileLogger : public ILogger {
private:
    std::string filename;
    
public:
    FileLogger(const std::string& f) : filename(f) {}
    
    void log(const std::string& message) override {
        std::cout << "FILE (" << filename << "): " << message << std::endl;
    }
};

class NullLogger : public ILogger {  // For testing
public:
    void log(const std::string& message) override {
        // Do nothing
    }
};

// Class with dependency injection
class UserServiceWithDI {
private:
    std::shared_ptr<ILogger> logger;  // Dependency injected
    
public:
    // Constructor Injection
    explicit UserServiceWithDI(std::shared_ptr<ILogger> l) : logger(l) {
        if (!logger) {
            throw std::invalid_argument("Logger cannot be null");
        }
    }
    
    void createUser(const std::string& name) {
        // Business logic...
        logger->log("User created: " + name);
    }
    
    // Setter Injection (optional)
    void setLogger(std::shared_ptr<ILogger> l) {
        logger = l;
    }
};

// ==================== Types of Injection ====================

// 1. Interface Injection (through interface)
class ILoggable {
public:
    virtual ~ILoggable() = default;
    virtual void setLogger(std::shared_ptr<ILogger> logger) = 0;
};

// 2. Property Injection (public member - less common)
class ServiceWithPublicLogger {
public:
    std::shared_ptr<ILogger> logger;  // Public property
    
    void doWork() {
        if (logger) {
            logger->log("Working...");
        }
    }
};

// ==================== Simple DI Container ====================
class DIContainer {
private:
    std::shared_ptr<ILogger> defaultLogger;
    
public:
    DIContainer() {
        // Register default implementations
        defaultLogger = std::make_shared<ConsoleLogger>();
    }
    
    std::shared_ptr<UserServiceWithDI> createUserService() {
        return std::make_shared<UserServiceWithDI>(defaultLogger);
    }
    
    std::shared_ptr<UserServiceWithDI> createUserServiceWithLogger(
        std::shared_ptr<ILogger> customLogger) {
        return std::make_shared<UserServiceWithDI>(customLogger);
    }
};

int main() {
    std::cout << "=== Without DI ===" << std::endl;
    UserService service;
    service.createUser("Alice");
    
    std::cout << "\n=== With DI ===" << std::endl;
    
    // Inject different dependencies
    auto consoleLogger = std::make_shared<ConsoleLogger>();
    UserServiceWithDI service1(consoleLogger);
    service1.createUser("Bob");
    
    auto fileLogger = std::make_shared<FileLogger>("app.log");
    UserServiceWithDI service2(fileLogger);
    service2.createUser("Charlie");
    
    // For testing
    auto nullLogger = std::make_shared<NullLogger>();
    UserServiceWithDI service3(nullLogger);
    service3.createUser("Test");  // No output
    
    std::cout << "\n=== Using DI Container ===" << std::endl;
    DIContainer container;
    auto service4 = container.createUserService();
    service4->createUser("David");
    
    return 0;
}
```

**Benefits of Dependency Injection:**
- **Testability**: Easy to inject mock dependencies for unit testing
- **Flexibility**: Can change behavior by injecting different implementations
- **Decoupling**: Classes don't create their own dependencies
- **Reusability**: Components can be reused with different configurations

---

### 63. What is interface vs abstract class?

In C++, an **abstract class** is a class with at least one pure virtual function. C++ doesn't have a formal `interface` keyword like Java/C#, but you can create **pure abstract classes** that serve as interfaces.

```cpp
#include <iostream>
#include <string>
#include <vector>

// ==================== Abstract Class ====================
// Can have both pure virtual and implemented methods
// Can have member variables
class Animal {
protected:
    std::string name;
    int age;
    
public:
    Animal(const std::string& n, int a) : name(n), age(a) {}
    
    // Pure virtual functions
    virtual void makeSound() const = 0;
    virtual void move() const = 0;
    
    // Implemented method
    virtual void sleep() const {
        std::cout << name << " is sleeping" << std::endl;
    }
    
    // Virtual destructor
    virtual ~Animal() = default;
};

// ==================== Interface (Pure Abstract Class) ====================
// No member variables, only pure virtual functions
class IDrawable {
public:
    virtual ~IDrawable() = default;
    virtual void draw() const = 0;
    virtual std::string getDescription() const = 0;
};

class ISerializable {
public:
    virtual ~ISerializable() = default;
    virtual std::string serialize() const = 0;
    virtual void deserialize(const std::string& data) = 0;
};

// ==================== Concrete Classes ====================
class Dog : public Animal, public IDrawable {
public:
    Dog(const std::string& n, int a) : Animal(n, a) {}
    
    // Implement Animal pure virtuals
    void makeSound() const override {
        std::cout << name << " says: Woof!" << std::endl;
    }
    
    void move() const override {
        std::cout << name << " runs on four legs" << std::endl;
    }
    
    // Implement IDrawable
    void draw() const override {
        std::cout << "Drawing a dog: " << name << std::endl;
    }
    
    std::string getDescription() const override {
        return "Dog: " + name + ", age " + std::to_string(age);
    }
};

class Circle : public IDrawable, public ISerializable {
private:
    double radius;
    
public:
    Circle(double r) : radius(r) {}
    
    // IDrawable implementation
    void draw() const override {
        std::cout << "Drawing a circle with radius " << radius << std::endl;
    }
    
    std::string getDescription() const override {
        return "Circle (r=" + std::to_string(radius) + ")";
    }
    
    // ISerializable implementation
    std::string serialize() const override {
        return "circle:" + std::to_string(radius);
    }
    
    void deserialize(const std::string& data) override {
        // Parse data...
        radius = std::stod(data.substr(7));
    }
};

// ==================== Comparison Example ====================
void demonstrateDifferences() {
    std::cout << "=== Abstract Class vs Interface ===" << std::endl;
    
    // Abstract class usage
    Dog dog("Rex", 3);
    
    // Can call abstract class methods
    dog.makeSound();
    dog.move();
    dog.sleep();  // Implemented in abstract class
    
    // Interface usage
    Circle circle(5.0);
    circle.draw();
    
    // Multiple interface implementation
    std::cout << "Description: " << circle.getDescription() << std::endl;
    std::cout << "Serialized: " << circle.serialize() << std::endl;
    
    // Polymorphism with abstract class
    std::vector<Animal*> animals;
    animals.push_back(&dog);
    // animals.push_back(&circle);  // ERROR: Circle not derived from Animal
    
    // Polymorphism with interfaces
    std::vector<IDrawable*> drawables;
    drawables.push_back(&dog);
    drawables.push_back(&circle);
    
    std::cout << "\nDrawing all drawables:" << std::endl;
    for (const auto& d : drawables) {
        d->draw();
    }
}
```

**Key Differences:**

| Feature | Abstract Class | Interface (Pure Abstract Class) |
| :--- | :--- | :--- |
| **Member variables** | ✅ Yes | ❌ No (only static const) |
| **Implemented methods** | ✅ Yes | ❌ No (all pure virtual) |
| **Constructors** | ✅ Yes | ❌ No |
| **Access specifiers** | Can have protected/private | Usually all public |
| **Multiple inheritance** | Single abstract class | Multiple interfaces |
| **Purpose** | Share common code + define contract | Define contract only |

---

### 64. What is composition vs aggregation?

**Composition** and **Aggregation** are two types of associations that describe **"has-a" relationships** between classes. They differ in **ownership and lifetime management**.

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

// ==================== Aggregation ====================
// "Has-a" relationship with weak ownership
// Parts can exist independently of the whole
// Lifetime not tied to container

class Department {
private:
    std::string name;
    
public:
    Department(const std::string& n) : name(n) {
        std::cout << "Department created: " << name << std::endl;
    }
    
    ~Department() {
        std::cout << "Department destroyed: " << name << std::endl;
    }
    
    std::string getName() const { return name; }
    void work() const { std::cout << name << " working" << std::endl; }
};

class Company {
private:
    std::string name;
    std::vector<Department*> departments;  // Aggregation - raw pointer (non-owning)
    
public:
    Company(const std::string& n) : name(n) {
        std::cout << "Company created: " << name << std::endl;
    }
    
    ~Company() {
        std::cout << "Company destroyed: " << name << std::endl;
        // Don't delete departments - they exist independently
    }
    
    void addDepartment(Department* dept) {
        departments.push_back(dept);
    }
    
    void showDepartments() const {
        std::cout << name << " departments: ";
        for (const auto& dept : departments) {
            std::cout << dept->getName() << " ";
        }
        std::cout << std::endl;
    }
};

// ==================== Composition ====================
// "Has-a" relationship with strong ownership
// Parts cannot exist independently of the whole
// Lifetime tied to container (created/destroyed together)

class Room {
private:
    std::string name;
    
public:
    Room(const std::string& n) : name(n) {
        std::cout << "  Room created: " << name << std::endl;
    }
    
    ~Room() {
        std::cout << "  Room destroyed: " << name << std::endl;
    }
    
    std::string getName() const { return name; }
    void describe() const { std::cout << "  Room: " << name << std::endl; }
};

class House {
private:
    std::string address;
    std::vector<std::unique_ptr<Room>> rooms;  // Composition - unique_ptr (owning)
    // Or by value: std::vector<Room> rooms; (stronger composition)
    
public:
    House(const std::string& addr) : address(addr) {
        std::cout << "House created at: " << address << std::endl;
        // Rooms created with the house
        rooms.push_back(std::make_unique<Room>("Living Room"));
        rooms.push_back(std::make_unique<Room>("Kitchen"));
        rooms.push_back(std::make_unique<Room>("Bedroom"));
    }
    
    ~House() {
        std::cout << "House destroyed at: " << address << std::endl;
        // rooms automatically destroyed by unique_ptr
    }
    
    void describe() const {
        std::cout << "House at " << address << " has:" << std::endl;
        for (const auto& room : rooms) {
            room->describe();
        }
    }
};

// ==================== Hybrid Example ====================
class Professor {
private:
    std::string name;
    
public:
    Professor(const std::string& n) : name(n) {
        std::cout << "Professor created: " << name << std::endl;
    }
    
    ~Professor() {
        std::cout << "Professor destroyed: " << name << std::endl;
    }
    
    void teach() const { std::cout << name << " teaching" << std::endl; }
    std::string getName() const { return name; }
};

class University {
private:
    std::string name;
    std::vector<std::unique_ptr<Department>> ownedDepartments;  // Composition
    std::vector<Professor*> affiliatedProfessors;               // Aggregation
    
public:
    University(const std::string& n) : name(n) {
        std::cout << "\nUniversity created: " << name << std::endl;
        // Create departments (composition)
        ownedDepartments.push_back(std::make_unique<Department>("CS"));
        ownedDepartments.push_back(std::make_unique<Department>("Math"));
    }
    
    ~University() {
        std::cout << "University destroyed: " << name << std::endl;
        // ownedDepartments automatically destroyed
        // affiliatedProfessors not destroyed (they exist independently)
    }
    
    void addProfessor(Professor* prof) {
        affiliatedProfessors.push_back(prof);
    }
    
    void showStructure() const {
        std::cout << name << " has departments:" << std::endl;
        for (const auto& dept : ownedDepartments) {
            std::cout << "  - " << dept->getName() << std::endl;
        }
        std::cout << "Affiliated professors:" << std::endl;
        for (const auto& prof : affiliatedProfessors) {
            std::cout << "  - " << prof->getName() << std::endl;
        }
    }
};

int main() {
    std::cout << "=== Aggregation Example ===" << std::endl;
    {
        Department cs("Computer Science");
        Department math("Mathematics");
        
        {
            Company techCorp("TechCorp");
            techCorp.addDepartment(&cs);
            techCorp.addDepartment(&math);
            techCorp.showDepartments();
        }  // Company destroyed, departments still exist
        
        cs.work();  // Department still exists
        math.work();
    }  // Departments destroyed here
    
    std::cout << "\n=== Composition Example ===" << std::endl;
    {
        House myHouse("123 Main St");
        myHouse.describe();
    }  // House and all its rooms destroyed together
    
    std::cout << "\n=== Hybrid Example ===" << std::endl;
    {
        Professor smith("Dr. Smith");
        Professor jones("Dr. Jones");
        
        University uni("State University");
        uni.addProfessor(&smith);
        uni.addProfessor(&jones);
        
        uni.showStructure();
    }  // University destroyed, professors still exist outside
    
    return 0;
}
```

**Key Differences:**

| Aspect | Composition | Aggregation |
| :--- | :--- | :--- |
| **Ownership** | Strong (parent owns child) | Weak (parent doesn't own) |
| **Lifetime** | Child lives/dies with parent | Child independent of parent |
| **Representation** | By value or `unique_ptr` | Raw pointer or reference |
| **Relationship** | "Part-of" | "Has-a" (looser) |
| **Example** | House and Rooms | Company and Departments |

---

### 65. What are design patterns?

**Design patterns** are **reusable solutions to common software design problems**. They represent best practices evolved over time by experienced developers. They are not code, but **templates/descriptions** of how to solve problems.

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

// ==================== Creational Patterns ====================
// Deal with object creation mechanisms

// === Singleton Pattern ===
class Singleton {
private:
    static std::unique_ptr<Singleton> instance;
    std::string data;
    
    // Private constructor
    Singleton(const std::string& d) : data(d) {
        std::cout << "Singleton created with data: " << data << std::endl;
    }
    
public:
    // Delete copy/move
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    static Singleton& getInstance(const std::string& data = "default") {
        if (!instance) {
            instance.reset(new Singleton(data));
        }
        return *instance;
    }
    
    void operation() const {
        std::cout << "Singleton operation with data: " << data << std::endl;
    }
    
    void setData(const std::string& d) { data = d; }
};

std::unique_ptr<Singleton> Singleton::instance = nullptr;

// === Factory Method Pattern ===
class Product {
public:
    virtual ~Product() = default;
    virtual void use() const = 0;
};

class ConcreteProductA : public Product {
public:
    void use() const override {
        std::cout << "Using Product A" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void use() const override {
        std::cout << "Using Product B" << std::endl;
    }
};

class Creator {
public:
    virtual ~Creator() = default;
    virtual std::unique_ptr<Product> createProduct() const = 0;
    
    void someOperation() const {
        auto product = createProduct();
        product->use();
    }
};

class CreatorA : public Creator {
public:
    std::unique_ptr<Product> createProduct() const override {
        return std::make_unique<ConcreteProductA>();
    }
};

class CreatorB : public Creator {
public:
    std::unique_ptr<Product> createProduct() const override {
        return std::make_unique<ConcreteProductB>();
    }
};

// ==================== Structural Patterns ====================
// Deal with class and object composition

// === Adapter Pattern ===
class EuropeanSocket {
public:
    virtual void provide220V() const {
        std::cout << "Providing 220V European power" << std::endl;
    }
    virtual ~EuropeanSocket() = default;
};

class USASocket {
public:
    void provide110V() const {
        std::cout << "Providing 110V USA power" << std::endl;
    }
};

// Adapter makes USA socket work with European interface
class SocketAdapter : public EuropeanSocket {
private:
    std::unique_ptr<USASocket> usaSocket;
    
public:
    SocketAdapter() : usaSocket(std::make_unique<USASocket>()) {}
    
    void provide220V() const override {
        std::cout << "Adapter converting: ";
        usaSocket->provide110V();
        std::cout << "to 220V" << std::endl;
    }
};

// ==================== Behavioral Patterns ====================
// Deal with communication between objects

// === Observer Pattern ===
class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(const std::string& message) = 0;
};

class Subject {
private:
    std::vector<Observer*> observers;
    std::string state;
    
public:
    void attach(Observer* obs) {
        observers.push_back(obs);
    }
    
    void detach(Observer* obs) {
        observers.erase(std::remove(observers.begin(), observers.end(), obs), 
                       observers.end());
    }
    
    void notify() {
        for (auto obs : observers) {
            obs->update(state);
        }
    }
    
    void setState(const std::string& newState) {
        state = newState;
        notify();
    }
};

class ConcreteObserver : public Observer {
private:
    std::string name;
    
public:
    ConcreteObserver(const std::string& n) : name(n) {}
    
    void update(const std::string& message) override {
        std::cout << "Observer " << name << " received: " << message << std::endl;
    }
};

// ==================== Main Demonstration ====================
int main() {
    std::cout << "=== Creational Pattern: Singleton ===" << std::endl;
    auto& s1 = Singleton::getInstance("first");
    auto& s2 = Singleton::getInstance("second");  // Returns same instance
    s1.operation();
    s2.operation();
    std::cout << "Same instance? " << (&s1 == &s2 ? "Yes" : "No") << std::endl;
    
    std::cout << "\n=== Creational Pattern: Factory ===" << std::endl;
    std::unique_ptr<Creator> creatorA = std::make_unique<CreatorA>();
    std::unique_ptr<Creator> creatorB = std::make_unique<CreatorB>();
    
    creatorA->someOperation();
    creatorB->someOperation();
    
    std::cout << "\n=== Structural Pattern: Adapter ===" << std::endl;
    EuropeanSocket* socket = new SocketAdapter();
    socket->provide220V();
    delete socket;
    
    std::cout << "\n=== Behavioral Pattern: Observer ===" << std::endl;
    Subject subject;
    ConcreteObserver obs1("Observer1");
    ConcreteObserver obs2("Observer2");
    
    subject.attach(&obs1);
    subject.attach(&obs2);
    
    subject.setState("State changed to ACTIVE");
    subject.setState("State changed to IDLE");
    
    return 0;
}
```

**Three Categories of Design Patterns:**

| Category | Purpose | Examples |
| :--- | :--- | :--- |
| **Creational** | Object creation mechanisms | Singleton, Factory, Builder, Prototype |
| **Structural** | Class/object composition | Adapter, Decorator, Facade, Proxy |
| **Behavioral** | Communication between objects | Observer, Strategy, Iterator, Command |

---

### 66. What is factory pattern?

The **Factory Pattern** is a creational pattern that provides an **interface for creating objects** without specifying their concrete classes. It encapsulates object creation logic.

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <map>

// ==================== Simple Factory (not a pattern, but common) ====================

class Vehicle {
public:
    virtual ~Vehicle() = default;
    virtual void drive() const = 0;
    virtual std::string getType() const = 0;
};

class Car : public Vehicle {
public:
    void drive() const override {
        std::cout << "Driving a car on roads" << std::endl;
    }
    
    std::string getType() const override {
        return "Car";
    }
};

class Bike : public Vehicle {
public:
    void drive() const override {
        std::cout << "Riding a bike on paths" << std::endl;
    }
    
    std::string getType() const override {
        return "Bike";
    }
};

class Truck : public Vehicle {
public:
    void drive() const override {
        std::cout << "Driving a truck on highways" << std::endl;
    }
    
    std::string getType() const override {
        return "Truck";
    }
};

// Simple Factory
class SimpleVehicleFactory {
public:
    enum VehicleType { CAR, BIKE, TRUCK };
    
    std::unique_ptr<Vehicle> createVehicle(VehicleType type) {
        switch (type) {
            case CAR:   return std::make_unique<Car>();
            case BIKE:  return std::make_unique<Bike>();
            case TRUCK: return std::make_unique<Truck>();
            default:    return nullptr;
        }
    }
};

// ==================== Factory Method Pattern ====================

// Abstract creator
class VehicleFactory {
public:
    virtual ~VehicleFactory() = default;
    
    // Factory method
    virtual std::unique_ptr<Vehicle> createVehicle() const = 0;
    
    // Operation that uses the factory method
    void deliverVehicle() const {
        auto vehicle = createVehicle();
        std::cout << "Delivering ";
        vehicle->drive();
    }
};

// Concrete creators
class CarFactory : public VehicleFactory {
public:
    std::unique_ptr<Vehicle> createVehicle() const override {
        return std::make_unique<Car>();
    }
};

class BikeFactory : public VehicleFactory {
public:
    std::unique_ptr<Vehicle> createVehicle() const override {
        return std::make_unique<Bike>();
    }
};

class TruckFactory : public VehicleFactory {
public:
    std::unique_ptr<Vehicle> createVehicle() const override {
        return std::make_unique<Truck>();
    }
};

// ==================== Abstract Factory Pattern ====================

// Abstract products
class Engine {
public:
    virtual ~Engine() = default;
    virtual void start() const = 0;
};

class Tyre {
public:
    virtual ~Tyre() = default;
    virtual void inflate() const = 0;
};

// Concrete products for Economy vehicles
class EconomyEngine : public Engine {
public:
    void start() const override {
        std::cout << "Economy engine starting quietly" << std::endl;
    }
};

class EconomyTyre : public Tyre {
public:
    void inflate() const override {
        std::cout << "Economy tyres inflated to 32 PSI" << std::endl;
    }
};

// Concrete products for Luxury vehicles
class LuxuryEngine : public Engine {
public:
    void start() const override {
        std::cout << "Luxury engine roaring powerfully" << std::endl;
    }
};

class LuxuryTyre : public Tyre {
public:
    void inflate() const override {
        std::cout << "Luxury tyres inflated to 35 PSI" << std::endl;
    }
};

// Abstract factory
class VehicleComponentFactory {
public:
    virtual ~VehicleComponentFactory() = default;
    virtual std::unique_ptr<Engine> createEngine() const = 0;
    virtual std::unique_ptr<Tyre> createTyre() const = 0;
};

// Concrete factories
class EconomyVehicleFactory : public VehicleComponentFactory {
public:
    std::unique_ptr<Engine> createEngine() const override {
        return std::make_unique<EconomyEngine>();
    }
    
    std::unique_ptr<Tyre> createTyre() const override {
        return std::make_unique<EconomyTyre>();
    }
};

class LuxuryVehicleFactory : public VehicleComponentFactory {
public:
    std::unique_ptr<Engine> createEngine() const override {
        return std::make_unique<LuxuryEngine>();
    }
    
    std::unique_ptr<Tyre> createTyre() const override {
        return std::make_unique<LuxuryTyre>();
    }
};

// Client that uses the abstract factory
class VehicleAssembler {
private:
    std::unique_ptr<VehicleComponentFactory> factory;
    
public:
    VehicleAssembler(std::unique_ptr<VehicleComponentFactory> f) 
        : factory(std::move(f)) {}
    
    void assembleVehicle() const {
        std::cout << "Assembling vehicle:" << std::endl;
        auto engine = factory->createEngine();
        auto tyre = factory->createTyre();
        
        engine->start();
        tyre->inflate();
        std::cout << "Vehicle assembled!" << std::endl;
    }
};

// ==================== Parameterized Factory ====================

class ConfigurableVehicleFactory {
private:
    std::map<std::string, std::function<std::unique_ptr<Vehicle>()>> creators;
    
public:
    ConfigurableVehicleFactory() {
        // Register default types
        registerType("car", [] { return std::make_unique<Car>(); });
        registerType("bike", [] { return std::make_unique<Bike>(); });
        registerType("truck", [] { return std::make_unique<Truck>(); });
    }
    
    void registerType(const std::string& type, 
                      std::function<std::unique_ptr<Vehicle>()> creator) {
        creators[type] = creator;
    }
    
    std::unique_ptr<Vehicle> create(const std::string& type) {
        auto it = creators.find(type);
        if (it != creators.end()) {
            return it->second();
        }
        return nullptr;
    }
};

int main() {
    std::cout << "=== Simple Factory ===" << std::endl;
    SimpleVehicleFactory simpleFactory;
    auto car = simpleFactory.createVehicle(SimpleVehicleFactory::CAR);
    car->drive();
    
    std::cout << "\n=== Factory Method ===" << std::endl;
    std::unique_ptr<VehicleFactory> factory = std::make_unique<CarFactory>();
    factory->deliverVehicle();
    
    factory = std::make_unique<BikeFactory>();
    factory->deliverVehicle();
    
    std::cout << "\n=== Abstract Factory ===" << std::endl;
    auto economyFactory = std::make_unique<EconomyVehicleFactory>();
    VehicleAssembler economyAssembler(std::move(economyFactory));
    economyAssembler.assembleVehicle();
    
    auto luxuryFactory = std::make_unique<LuxuryVehicleFactory>();
    VehicleAssembler luxuryAssembler(std::move(luxuryFactory));
    luxuryAssembler.assembleVehicle();
    
    std::cout << "\n=== Parameterized Factory ===" << std::endl;
    ConfigurableVehicleFactory configFactory;
    configFactory.registerType("suv", [] { 
        std::cout << "Creating custom SUV" << std::endl;
        return std::make_unique<Car>();  // Just for demo
    });
    
    auto suv = configFactory.create("suv");
    if (suv) suv->drive();
    
    return 0;
}
```

---

### 67. What is strategy pattern?

The **Strategy Pattern** defines a family of algorithms, encapsulates each one, and makes them **interchangeable**. It lets the algorithm **vary independently** from clients that use it.

```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <algorithm>

// ==================== Strategy Interface ====================
class SortStrategy {
public:
    virtual ~SortStrategy() = default;
    virtual void sort(std::vector<int>& data) const = 0;
    virtual std::string getName() const = 0;
};

// ==================== Concrete Strategies ====================

class BubbleSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) const override {
        std::cout << "Using Bubble Sort (O(n²))" << std::endl;
        for (size_t i = 0; i < data.size() - 1; ++i) {
            for (size_t j = 0; j < data.size() - i - 1; ++j) {
                if (data[j] > data[j + 1]) {
                    std::swap(data[j], data[j + 1]);
                }
            }
        }
    }
    
    std::string getName() const override {
        return "Bubble Sort";
    }
};

class QuickSort : public SortStrategy {
private:
    void quickSort(std::vector<int>& data, int low, int high) const {
        if (low < high) {
            int pi = partition(data, low, high);
            quickSort(data, low, pi - 1);
            quickSort(data, pi + 1, high);
        }
    }
    
    int partition(std::vector<int>& data, int low, int high) const {
        int pivot = data[high];
        int i = low - 1;
        
        for (int j = low; j < high; ++j) {
            if (data[j] < pivot) {
                i++;
                std::swap(data[i], data[j]);
            }
        }
        std::swap(data[i + 1], data[high]);
        return i + 1;
    }
    
public:
    void sort(std::vector<int>& data) const override {
        std::cout << "Using Quick Sort (O(n log n))" << std::endl;
        quickSort(data, 0, data.size() - 1);
    }
    
    std::string getName() const override {
        return "Quick Sort";
    }
};

class STLSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) const override {
        std::cout << "Using STL sort (Introsort)" << std::endl;
        std::sort(data.begin(), data.end());
    }
    
    std::string getName() const override {
        return "STL Sort";
    }
};

// ==================== Context ====================

class Sorter {
private:
    std::unique_ptr<SortStrategy> strategy;
    std::vector<int> data;
    
public:
    explicit Sorter(std::unique_ptr<SortStrategy> s) : strategy(std::move(s)) {}
    
    void setStrategy(std::unique_ptr<SortStrategy> s) {
        strategy = std::move(s);
        std::cout << "Strategy changed to: " << strategy->getName() << std::endl;
    }
    
    void setData(const std::vector<int>& d) {
        data = d;
    }
    
    void executeSort() {
        if (!strategy) {
            std::cout << "No strategy set!" << std::endl;
            return;
        }
        
        std::cout << "\nOriginal data: ";
        printData();
        
        strategy->sort(data);
        
        std::cout << "Sorted data: ";
        printData();
    }
    
    void printData() const {
        for (int val : data) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }
};

// ==================== Another Example: Payment Strategies ====================

class PaymentStrategy {
public:
    virtual ~PaymentStrategy() = default;
    virtual void pay(double amount) const = 0;
    virtual std::string getMethod() const = 0;
};

class CreditCardPayment : public PaymentStrategy {
private:
    std::string cardNumber;
    std::string name;
    
public:
    CreditCardPayment(const std::string& num, const std::string& n)
        : cardNumber(num), name(n) {}
    
    void pay(double amount) const override {
        std::cout << "Paid $" << amount << " using Credit Card (****" 
                  << cardNumber.substr(cardNumber.length() - 4) 
                  << ") for " << name << std::endl;
    }
    
    std::string getMethod() const override {
        return "Credit Card";
    }
};

class PayPalPayment : public PaymentStrategy {
private:
    std::string email;
    
public:
    PayPalPayment(const std::string& e) : email(e) {}
    
    void pay(double amount) const override {
        std::cout << "Paid $" << amount << " using PayPal account: " 
                  << email << std::endl;
    }
    
    std::string getMethod() const override {
        return "PayPal";
    }
};

class CashPayment : public PaymentStrategy {
public:
    void pay(double amount) const override {
        std::cout << "Paid $" << amount << " in cash" << std::endl;
    }
    
    std::string getMethod() const override {
        return "Cash";
    }
};

class ShoppingCart {
private:
    std::vector<double> items;
    std::unique_ptr<PaymentStrategy> paymentStrategy;
    
public:
    void addItem(double price) {
        items.push_back(price);
    }
    
    void setPaymentStrategy(std::unique_ptr<PaymentStrategy> strategy) {
        paymentStrategy = std::move(strategy);
    }
    
    double calculateTotal() const {
        double total = 0;
        for (double price : items) {
            total += price;
        }
        return total;
    }
    
    void checkout() {
        double total = calculateTotal();
        std::cout << "\nCart total: $" << total << std::endl;
        
        if (paymentStrategy) {
            std::cout << "Payment method: " << paymentStrategy->getMethod() << std::endl;
            paymentStrategy->pay(total);
        } else {
            std::cout << "No payment method selected!" << std::endl;
        }
    }
};

int main() {
    std::cout << "=== Strategy Pattern: Sorting ===" << std::endl;
    
    std::vector<int> data = {64, 34, 25, 12, 22, 11, 90};
    
    // Create sorter with initial strategy
    auto sorter = Sorter(std::make_unique<BubbleSort>());
    sorter.setData(data);
    sorter.executeSort();
    
    // Change strategy at runtime
    sorter.setStrategy(std::make_unique<QuickSort>());
    sorter.executeSort();
    
    sorter.setStrategy(std::make_unique<STLSort>());
    sorter.executeSort();
    
    std::cout << "\n=== Strategy Pattern: Payment ===" << std::endl;
    
    ShoppingCart cart;
    cart.addItem(29.99);
    cart.addItem(49.50);
    cart.addItem(15.75);
    
    // Pay with credit card
    cart.setPaymentStrategy(std::make_unique<CreditCardPayment>(
        "1234-5678-9012-3456", "John Doe"));
    cart.checkout();
    
    // Pay with PayPal
    cart.setPaymentStrategy(std::make_unique<PayPalPayment>("john@example.com"));
    cart.checkout();
    
    // Pay with cash
    cart.setPaymentStrategy(std::make_unique<CashPayment>());
    cart.checkout();
    
    return 0;
}
```

---

### 68. What is iterator pattern?

The **Iterator Pattern** provides a way to **access elements of a collection sequentially** without exposing the underlying representation. It separates traversal from the collection itself.

```cpp
#include <iostream>
#include <vector>
#include <stack>
#include <memory>
#include <stdexcept>

// ==================== Basic Iterator Pattern ====================

template <typename T>
class Iterator {
public:
    virtual ~Iterator() = default;
    virtual bool hasNext() const = 0;
    virtual T next() = 0;
    virtual void reset() = 0;
};

// Concrete collection
template <typename T>
class Playlist {
private:
    std::vector<T> songs;
    
public:
    void addSong(const T& song) {
        songs.push_back(song);
    }
    
    size_t size() const { return songs.size(); }
    
    // Iterator class (can be nested)
    class PlaylistIterator : public Iterator<T> {
    private:
        const std::vector<T>& songs;
        size_t index;
        
    public:
        explicit PlaylistIterator(const std::vector<T>& s) : songs(s), index(0) {}
        
        bool hasNext() const override {
            return index < songs.size();
        }
        
        T next() override {
            if (!hasNext()) {
                throw std::out_of_range("No more elements");
            }
            return songs[index++];
        }
        
        void reset() override {
            index = 0;
        }
    };
    
    std::unique_ptr<Iterator<T>> createIterator() const {
        return std::make_unique<PlaylistIterator>(songs);
    }
};

// ==================== More Complex Example: Tree Iterator ====================

template <typename T>
class TreeNode {
public:
    T data;
    std::unique_ptr<TreeNode<T>> left;
    std::unique_ptr<TreeNode<T>> right;
    
    TreeNode(T val) : data(val), left(nullptr), right(nullptr) {}
};

template <typename T>
class BinaryTree {
private:
    std::unique_ptr<TreeNode<T>> root;
    
public:
    void insert(const T& value) {
        insertImpl(root, value);
    }
    
    // In-order iterator
    class InOrderIterator : public Iterator<T> {
    private:
        std::stack<TreeNode<T>*> nodeStack;
        TreeNode<T>* current;
        
    public:
        explicit InOrderIterator(TreeNode<T>* root) : current(root) {
            // Push all left nodes
            while (current) {
                nodeStack.push(current);
                current = current->left.get();
            }
            if (!nodeStack.empty()) {
                current = nodeStack.top();
            }
        }
        
        bool hasNext() const override {
            return !nodeStack.empty() || current;
        }
        
        T next() override {
            if (!hasNext()) {
                throw std::out_of_range("No more elements");
            }
            
            TreeNode<T>* node = nodeStack.top();
            nodeStack.pop();
            T value = node->data;
            
            // Move to right subtree
            if (node->right) {
                current = node->right.get();
                while (current) {
                    nodeStack.push(current);
                    current = current->left.get();
                }
            }
            
            if (!nodeStack.empty()) {
                current = nodeStack.top();
            } else {
                current = nullptr;
            }
            
            return value;
        }
        
        void reset() override {
            // Would need to rebuild iterator
        }
    };
    
    std::unique_ptr<Iterator<T>> createInOrderIterator() const {
        return std::make_unique<InOrderIterator>(root.get());
    }
    
private:
    void insertImpl(std::unique_ptr<TreeNode<T>>& node, const T& value) {
        if (!node) {
            node = std::make_unique<TreeNode<T>>(value);
            return;
        }
        
        if (value < node->data) {
            insertImpl(node->left, value);
        } else {
            insertImpl(node->right, value);
        }
    }
};

// ==================== C++ STL Style Iterator ====================

template <typename T>
class STLStyleCollection {
private:
    std::vector<T> data;
    
public:
    void add(const T& item) {
        data.push_back(item);
    }
    
    // STL-style iterators
    using iterator = typename std::vector<T>::iterator;
    using const_iterator = typename std::vector<T>::const_iterator;
    
    iterator begin() { return data.begin(); }
    iterator end() { return data.end(); }
    const_iterator begin() const { return data.begin(); }
    const_iterator end() const { return data.end(); }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Iterator Pattern: Playlist ===" << std::endl;
    
    Playlist<std::string> playlist;
    playlist.addSong("Bohemian Rhapsody");
    playlist.addSong("Stairway to Heaven");
    playlist.addSong("Hotel California");
    playlist.addSong("Imagine");
    
    auto iterator = playlist.createIterator();
    
    std::cout << "Playlist songs:" << std::endl;
    while (iterator->hasNext()) {
        std::cout << "  - " << iterator->next() << std::endl;
    }
    
    // Reset and iterate again
    iterator->reset();
    std::cout << "\nPlaying again:" << std::endl;
    while (iterator->hasNext()) {
        std::cout << "  Playing: " << iterator->next() << std::endl;
    }
    
    std::cout << "\n=== Iterator Pattern: Binary Tree ===" << std::endl;
    
    BinaryTree<int> tree;
    tree.insert(5);
    tree.insert(3);
    tree.insert(7);
    tree.insert(2);
    tree.insert(4);
    tree.insert(6);
    tree.insert(8);
    
    auto treeIterator = tree.createInOrderIterator();
    
    std::cout << "Tree in-order traversal:" << std::endl;
    while (treeIterator->hasNext()) {
        std::cout << treeIterator->next() << " ";
    }
    std::cout << std::endl;
    
    std::cout << "\n=== C++ STL Style Iterators ===" << std::endl;
    
    STLStyleCollection<int> collection;
    collection.add(10);
    collection.add(20);
    collection.add(30);
    collection.add(40);
    
    std::cout << "Collection using range-based for loop:" << std::endl;
    for (const auto& item : collection) {
        std::cout << item << " ";
    }
    std::cout << std::endl;
    
    std::cout << "\nUsing explicit iterators:" << std::endl;
    for (auto it = collection.begin(); it != collection.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```

---

### 69. When to use inheritance vs composition?

**Inheritance** ("is-a" relationship) and **composition** ("has-a" relationship) are two fundamental ways to reuse code. Knowing when to use each is crucial for good design.

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

// ==================== Inheritance Example ====================
// Use when: "is-a" relationship, polymorphic behavior needed

class Animal {
protected:
    std::string name;
    
public:
    Animal(const std::string& n) : name(n) {}
    virtual ~Animal() = default;
    
    virtual void speak() const {
        std::cout << name << " makes a sound" << std::endl;
    }
    
    virtual void move() const {
        std::cout << name << " moves" << std::endl;
    }
};

class Dog : public Animal {  // Dog "is-a" Animal
public:
    Dog(const std::string& n) : Animal(n) {}
    
    void speak() const override {
        std::cout << name << " barks: Woof!" << std::endl;
    }
    
    void fetch() const {  // Dog-specific behavior
        std::cout << name << " fetches the ball" << std::endl;
    }
};

class Cat : public Animal {  // Cat "is-a" Animal
public:
    Cat(const std::string& n) : Animal(n) {}
    
    void speak() const override {
        std::cout << name << " meows: Meow!" << std::endl;
    }
};

// ==================== Composition Example ====================
// Use when: "has-a" relationship, more flexibility needed

class Engine {
private:
    int horsepower;
    
public:
    Engine(int hp) : horsepower(hp) {}
    
    void start() const {
        std::cout << "Engine (" << horsepower << "hp) started" << std::endl;
    }
    
    void stop() const {
        std::cout << "Engine stopped" << std::endl;
    }
};

class Wheels {
private:
    int count;
    
public:
    Wheels(int c) : count(c) {}
    
    void rotate() const {
        std::cout << count << " wheels rotating" << std::endl;
    }
};

class Car {  // Car "has-a" Engine, "has-a" Wheels
private:
    std::unique_ptr<Engine> engine;
    std::unique_ptr<Wheels> wheels;
    std::string model;
    
public:
    Car(const std::string& m, int hp, int wheelCount) 
        : model(m), engine(std::make_unique<Engine>(hp)), 
          wheels(std::make_unique<Wheels>(wheelCount)) {}
    
    void start() const {
        std::cout << model << ": ";
        engine->start();
    }
    
    void drive() const {
        std::cout << model << " driving: ";
        wheels->rotate();
    }
    
    // Can easily swap components
    void setEngine(std::unique_ptr<Engine> newEngine) {
        engine = std::move(newEngine);
    }
};

// ==================== When to Choose Each ====================

// BAD: Using inheritance when composition is better
class Database {
public:
    void connect() { /* ... */ }
    void query() { /* ... */ }
};

class UserService : public Database {  // Wrong! UserService is not a Database
public:
    void createUser() {
        connect();  // Using inherited database methods
        // ...
    }
};

// GOOD: Using composition
class UserService2 {
private:
    std::unique_ptr<Database> database;  // Has-a Database
    
public:
    UserService2(std::unique_ptr<Database> db) : database(std::move(db)) {}
    
    void createUser() {
        database->connect();
        // ...
    }
};

// ==================== Strategy Pattern (Composition over Inheritance) ====================

class FlyBehavior {
public:
    virtual ~FlyBehavior() = default;
    virtual void fly() const = 0;
};

class FlyWithWings : public FlyBehavior {
public:
    void fly() const override {
        std::cout << "Flying with wings!" << std::endl;
    }
};

class FlyNoWay : public FlyBehavior {
public:
    void fly() const override {
        std::cout << "Can't fly!" << std::endl;
    }
};

class Bird {
private:
    std::unique_ptr<FlyBehavior> flyBehavior;  // Composition for behavior
    
public:
    Bird(std::unique_ptr<FlyBehavior> fb) : flyBehavior(std::move(fb)) {}
    
    void performFly() const {
        flyBehavior->fly();
    }
    
    void setFlyBehavior(std::unique_ptr<FlyBehavior> fb) {
        flyBehavior = std::move(fb);
    }
};

// ==================== Comparison Example ====================

void demonstrateChoices() {
    std::cout << "=== Inheritance (is-a) ===" << std::endl;
    std::vector<std::unique_ptr<Animal>> animals;
    animals.push_back(std::make_unique<Dog>("Rex"));
    animals.push_back(std::make_unique<Cat>("Whiskers"));
    
    for (const auto& animal : animals) {
        animal->speak();
        animal->move();
    }
    // Can't call dog-specific fetch() without casting
    
    std::cout << "\n=== Composition (has-a) ===" << std::endl;
    Car myCar("Tesla", 500, 4);
    myCar.start();
    myCar.drive();
    
    // Can easily swap components
    myCar.setEngine(std::make_unique<Engine>(600));
    myCar.start();
    
    std::cout << "\n=== Composition over Inheritance ===" << std::endl;
    Bird sparrow(std::make_unique<FlyWithWings>());
    Bird penguin(std::make_unique<FlyNoWay>());
    
    sparrow.performFly();
    penguin.performFly();
    
    // Can change behavior at runtime
    penguin.setFlyBehavior(std::make_unique<FlyWithWings>());  // Magic penguin?
    penguin.performFly();
}
```

**Guidelines:**

| Use Inheritance When | Use Composition When |
| :--- | :--- |
| Clear "is-a" relationship | "Has-a" or "uses-a" relationship |
| Need polymorphic behavior | Need flexibility to change behavior |
| Base class provides core functionality | Want to reuse functionality across hierarchies |
| Liskov substitution holds | Need to avoid deep inheritance hierarchies |
| Framework requires inheritance | Want to encapsulate complex behavior |

**Rule of thumb:** Prefer composition over inheritance unless there's a compelling reason for inheritance.

---

### 70. What is interface segregation principle (ISP)?

The **Interface Segregation Principle** is the "I" in SOLID. It states that **no client should be forced to depend on interfaces it does not use**. Instead of one large interface, create smaller, focused interfaces.

```cpp
#include <iostream>
#include <memory>
#include <vector>

// ==================== BAD: Fat interface ====================

class IWorker {
public:
    virtual ~IWorker() = default;
    
    virtual void work() = 0;
    virtual void eat() = 0;
    virtual void sleep() = 0;
    virtual void code() = 0;
    virtual void design() = 0;
    virtual void manage() = 0;
};

// Human worker needs all methods
class HumanDeveloper : public IWorker {
public:
    void work() override { std::cout << "Human working" << std::endl; }
    void eat() override { std::cout << "Human eating" << std::endl; }
    void sleep() override { std::cout << "Human sleeping" << std::endl; }
    void code() override { std::cout << "Human coding" << std::endl; }
    void design() override { std::cout << "Human designing" << std::endl; }
    void manage() override { std::cout << "Human managing" << std::endl; }
};

// Robot doesn't need eat/sleep, but forced to implement them
class RobotWorker : public IWorker {
public:
    void work() override { std::cout << "Robot working" << std::endl; }
    void eat() override { throw std::logic_error("Robots don't eat!"); }
    void sleep() override { throw std::logic_error("Robots don't sleep!"); }
    void code() override { std::cout << "Robot coding" << std::endl; }
    void design() override { std::cout << "Robot designing" << std::endl; }
    void manage() override { throw std::logic_error("Robots don't manage!"); }
};

// ==================== GOOD: Segregated interfaces ====================

// Basic work interface
class IWorkable {
public:
    virtual ~IWorkable() = default;
    virtual void work() = 0;
};

// Human-specific needs
class IEatable {
public:
    virtual ~IEatable() = default;
    virtual void eat() = 0;
};

class ISleepable {
public:
    virtual ~ISleepable() = default;
    virtual void sleep() = 0;
};

// Role-specific interfaces
class ICodeable {
public:
    virtual ~ICodeable() = default;
    virtual void code() = 0;
};

class IDesignable {
public:
    virtual ~IDesignable() = default;
    virtual void design() = 0;
};

class IManageable {
public:
    virtual ~IManageable() = default;
    virtual void manage() = 0;
};

// Now humans implement only what they need
class HumanDeveloper2 : public IWorkable, public IEatable, 
                        public ISleepable, public ICodeable, 
                        public IDesignable, public IManageable {
public:
    void work() override { std::cout << "Human working" << std::endl; }
    void eat() override { std::cout << "Human eating" << std::endl; }
    void sleep() override { std::cout << "Human sleeping" << std::endl; }
    void code() override { std::cout << "Human coding" << std::endl; }
    void design() override { std::cout << "Human designing" << std::endl; }
    void manage() override { std::cout << "Human managing" << std::endl; }
};

// Robots implement only what they need
class RobotWorker2 : public IWorkable, public ICodeable, public IDesignable {
public:
    void work() override { std::cout << "Robot working" << std::endl; }
    void code() override { std::cout << "Robot coding" << std::endl; }
    void design() override { std::cout << "Robot designing" << std::endl; }
    // No eat/sleep methods forced!
};

// ==================== Another Example: Printer System ====================

// BAD: Monolithic interface
class IMachine {
public:
    virtual ~IMachine() = default;
    virtual void print() = 0;
    virtual void scan() = 0;
    virtual void fax() = 0;
    virtual void staple() = 0;
    virtual void copy() = 0;
};

// Simple printer forced to implement unnecessary methods
class SimplePrinter : public IMachine {
public:
    void print() override { std::cout << "Printing..." << std::endl; }
    void scan() override { throw std::logic_error("Cannot scan!"); }
    void fax() override { throw std::logic_error("Cannot fax!"); }
    void staple() override { throw std::logic_error("Cannot staple!"); }
    void copy() override { throw std::logic_error("Cannot copy!"); }
};

// GOOD: Segregated printer interfaces
class IPrinter {
public:
    virtual ~IPrinter() = default;
    virtual void print() = 0;
};

class IScanner {
public:
    virtual ~IScanner() = default;
    virtual void scan() = 0;
};

class IFax {
public:
    virtual ~IFax() = default;
    virtual void fax() = 0;
};

class IStapler {
public:
    virtual ~IStapler() = default;
    virtual void staple() = 0;
};

class ICopier : public IPrinter, public IScanner {};

// Simple printer implements only what it needs
class SimplePrinter2 : public IPrinter {
public:
    void print() override { std::cout << "Printing..." << std::endl; }
};

// Multifunction printer implements multiple interfaces
class MultiFunctionPrinter : public IPrinter, public IScanner, public IFax {
public:
    void print() override { std::cout << "Printing..." << std::endl; }
    void scan() override { std::cout << "Scanning..." << std::endl; }
    void fax() override { std::cout << "Faxing..." << std::endl; }
};

// ==================== Client Code That Benefits ====================

// Client only depends on what it needs
void printDocument(IPrinter& printer, const std::string& doc) {
    std::cout << "Printing document: " << doc << std::endl;
    printer.print();
}

void scanDocument(IScanner& scanner) {
    std::cout << "Scanning: ";
    scanner.scan();
}

// Can combine as needed
void copyDocument(ICopier& copier) {
    copier.scan();
    copier.print();
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Bad Design (Fat Interface) ===" << std::endl;
    try {
        RobotWorker robot;
        robot.work();
        robot.code();
        // robot.eat();  // Would throw exception!
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << std::endl;
    }
    
    std::cout << "\n=== Good Design (Segregated Interfaces) ===" << std::endl;
    
    RobotWorker2 robot2;
    robot2.work();
    robot2.code();
    robot2.design();
    // No eat/sleep methods available - can't call by mistake!
    
    std::cout << "\n=== Printer Example ===" << std::endl;
    
    SimplePrinter2 printer;
    MultiFunctionPrinter mfp;
    
    // Clients only depend on what they need
    printDocument(printer, "Report.pdf");
    printDocument(mfp, "Letter.doc");
    
    scanDocument(mfp);
    // scanDocument(printer);  // ERROR: printer doesn't implement IScanner
    
    // Can't accidentally call wrong methods
    // mfp.staple();  // ERROR: doesn't exist - good!
    
    std::cout << "\n=== Benefits of ISP ===" << std::endl;
    std::cout << "1. No empty or throwing method implementations" << std::endl;
    std::cout << "2. Clients depend only on what they use" << std::endl;
    std::cout << "3. Easier to mock for testing" << std::endl;
    std::cout << "4. More flexible and maintainable" << std::endl;
    
    return 0;
}
```

**Benefits of ISP:**
- **Reduces coupling** - clients only depend on methods they use
- **Increases cohesion** - interfaces are focused on specific roles
- **Improves readability** - smaller interfaces are easier to understand
- **Easier to implement** - classes only need to implement relevant methods
- **Better testability** - can mock only needed methods

**Signs of ISP violation:**
- Classes have empty method implementations
- Methods throw "not implemented" exceptions
- Clients pass `null` for unused parameters
- Interfaces have many methods but most clients use only a subset