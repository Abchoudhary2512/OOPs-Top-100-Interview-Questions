### 89. What are virtual table (vtable) and vptr?

The **virtual table (vtable)** and **vptr (virtual pointer)** are the **internal implementation mechanisms** that C++ uses to support **runtime polymorphism** (virtual functions).

```cpp
#include <iostream>
#include <vector>

// ==================== Conceptual Explanation ====================

/*
 * vtable (Virtual Table):
 * - A static array (per class) containing function pointers to virtual functions
 * - One vtable per class with virtual functions
 * - Created at compile time
 * 
 * vptr (Virtual Pointer):
 * - A hidden pointer in each object pointing to its class's vtable
 * - Added automatically when class has virtual functions
 * - Set during object construction
 */

// ==================== Example Demonstrating vtable/vptr ====================

class Base {
public:
    // Virtual function - triggers vtable creation
    virtual void func1() {
        std::cout << "Base::func1" << std::endl;
    }
    
    virtual void func2() {
        std::cout << "Base::func2" << std::endl;
    }
    
    // Non-virtual function - not in vtable
    void func3() {
        std::cout << "Base::func3" << std::endl;
    }
    
    virtual ~Base() {}  // Virtual destructor
};

class Derived : public Base {
public:
    // Override func1
    void func1() override {
        std::cout << "Derived::func1" << std::endl;
    }
    
    // Add new virtual function
    virtual void func4() {
        std::cout << "Derived::func4" << std::endl;
    }
};

// ==================== Visualizing vtable Structure ====================

/*
 * Conceptual vtable layout:
 * 
 * Base vtable:
 * [0] -> Base::func1
 * [1] -> Base::func2
 * [2] -> Base::~Base
 * 
 * Derived vtable:
 * [0] -> Derived::func1  (overridden)
 * [1] -> Base::func2      (inherited)
 * [2] -> Base::~Base      (inherited)
 * [3] -> Derived::func4   (new)
 */

// ==================== Memory Layout Inspection ====================

#include <cstdint>

void inspectVptr(const Base& obj) {
    // Get the address of the object
    const uint8_t* objPtr = reinterpret_cast<const uint8_t*>(&obj);
    
    // First sizeof(void*) bytes are typically the vptr
    const void* vptr = *reinterpret_cast<const void* const*>(&obj);
    
    std::cout << "Object address: " << static_cast<const void*>(&obj) << std::endl;
    std::cout << "vptr address: " << static_cast<const void*>(&vptr) << std::endl;
    std::cout << "vptr points to: " << vptr << std::endl;
}

// ==================== Multiple Inheritance vtable ====================

class Base1 {
public:
    virtual void f1() { std::cout << "Base1::f1" << std::endl; }
    virtual ~Base1() {}
};

class Base2 {
public:
    virtual void f2() { std::cout << "Base2::f2" << std::endl; }
    virtual ~Base2() {}
};

class MultipleDerived : public Base1, public Base2 {
public:
    void f1() override { std::cout << "MultipleDerived::f1" << std::endl; }
    void f2() override { std::cout << "MultipleDerived::f2" << std::endl; }
};

// ==================== vtable Access Demonstration ====================

typedef void (*FuncPtr)();

void callViaVtable(Base* obj, int index) {
    // This is unsafe and compiler-dependent - for illustration only
    void** vtable = *(void***)obj;
    FuncPtr func = (FuncPtr)vtable[index];
    func();
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== vtable and vptr Demonstration ===" << std::endl;
    
    std::cout << "\nSize of objects:" << std::endl;
    std::cout << "sizeof(Base): " << sizeof(Base) << " bytes" << std::endl;
    std::cout << "sizeof(Derived): " << sizeof(Derived) << " bytes" << std::endl;
    std::cout << "sizeof(void*): " << sizeof(void*) << " bytes" << std::endl;
    
    std::cout << "\nPolymorphic behavior:" << std::endl;
    Base* basePtr = new Derived();
    
    // These calls use vtable indirection
    basePtr->func1();  // Calls Derived::func1
    basePtr->func2();  // Calls Base::func2
    basePtr->func3();  // Non-virtual - direct call
    
    std::cout << "\nvptr inspection:" << std::endl;
    Base b;
    Derived d;
    
    std::cout << "Base object: ";
    inspectVptr(b);
    std::cout << "Derived object: ";
    inspectVptr(d);
    
    std::cout << "\nMultiple inheritance vtable:" << std::endl;
    MultipleDerived md;
    std::cout << "sizeof(MultipleDerived): " << sizeof(MultipleDerived) << " bytes" << std::endl;
    // Has two vptrs (one for each base class)
    
    delete basePtr;
    
    std::cout << "\n=== vtable Implementation Details ===" << std::endl;
    std::cout << "1. vtable created per class (static)" << std::endl;
    std::cout << "2. vptr stored in each object (dynamic)" << std::endl;
    std::cout << "3. vptr initialized in constructor" << std::endl;
    std::cout << "4. Multiple inheritance creates multiple vptrs" << std::endl;
    
    return 0;
}
```

**Key Points:**

| Component | Description | Storage |
| :--- | :--- | :--- |
| **vtable** | Array of function pointers for virtual functions | One per class (static) |
| **vptr** | Pointer to vtable | One per object (dynamic) |
| **Override** | Derived class replaces vtable entry | New function pointer |
| **Multiple inheritance** | Multiple vptrs per object | One for each base class |

---

### 90. Can destructor throw exceptions?

**Destructors should never throw exceptions** in C++. If a destructor throws an exception during stack unwinding (when another exception is already active), the program **terminates**.

```cpp
#include <iostream>
#include <exception>
#include <vector>
#include <memory>

// ==================== BAD: Destructor Throwing Exceptions ====================

class BadResource {
private:
    std::string name;
    
public:
    BadResource(const std::string& n) : name(n) {
        std::cout << name << " acquired" << std::endl;
    }
    
    ~BadResource() {
        std::cout << name << " destructor starts" << std::endl;
        // BAD: Throwing in destructor
        if (name == "Resource2") {
            std::cout << "  Throwing exception in destructor!" << std::endl;
            throw std::runtime_error("Error in destructor");
        }
        std::cout << name << " destructor ends" << std::endl;
    }
};

// ==================== GOOD: Destructor Never Throws ====================

class GoodResource {
private:
    std::string name;
    bool throwOnCleanup;  // For demonstration only
    
public:
    GoodResource(const std::string& n, bool throws = false) 
        : name(n), throwOnCleanup(throws) {
        std::cout << name << " acquired" << std::endl;
    }
    
    ~GoodResource() noexcept {  // noexcept guarantees no throw
        std::cout << name << " destructor starts" << std::endl;
        try {
            // Cleanup code that might throw
            if (throwOnCleanup) {
                std::cout << "  Cleanup failed but caught!" << std::endl;
                // Handle error without throwing
            }
        } catch (...) {
            // Swallow all exceptions - destructor must not throw
            std::cout << "  Exception caught and swallowed" << std::endl;
        }
        std::cout << name << " destructor ends" << std::endl;
    }
    
    // Separate method for operations that might throw
    void cleanup() {
        if (throwOnCleanup) {
            throw std::runtime_error("Cleanup failed");
        }
    }
};

// ==================== RAII with Proper Exception Safety ====================

class DatabaseConnection {
private:
    bool connected;
    
public:
    DatabaseConnection() : connected(true) {
        std::cout << "Database connected" << std::endl;
    }
    
    // Destructor should not throw
    ~DatabaseConnection() noexcept {
        try {
            disconnect();
        } catch (...) {
            // Log error but don't throw
            std::cout << "Error during disconnect (logged)" << std::endl;
        }
    }
    
    void disconnect() {
        if (connected) {
            connected = false;
            std::cout << "Database disconnected" << std::endl;
        }
    }
    
    void query(const std::string& q) {
        if (!connected) throw std::runtime_error("Not connected");
        std::cout << "Executing query: " << q << std::endl;
    }
};

// ==================== Demonstrating Problems ====================

void demonstrateBadDestructor() {
    std::cout << "\n=== BAD: Destructor Throwing ===" << std::endl;
    
    try {
        BadResource r1("Resource1");
        BadResource r2("Resource2");  // This will throw in destructor
        BadResource r3("Resource3");
        
        throw std::runtime_error("Exception in try block");
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }
    // Program may terminate or have undefined behavior
}

void demonstrateGoodDestructor() {
    std::cout << "\n=== GOOD: Destructor Noexcept ===" << std::endl;
    
    try {
        GoodResource r1("Resource1");
        GoodResource r2("Resource2", true);  // Destructor catches exception
        GoodResource r3("Resource3");
        
        throw std::runtime_error("Exception in try block");
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }
    std::cout << "Program continues safely" << std::endl;
}

// ==================== Container Behavior ====================

void containerWithExceptions() {
    std::cout << "\n=== Container Behavior ===" << std::endl;
    
    std::vector<GoodResource> resources;
    resources.emplace_back("Vector Resource 1");
    resources.emplace_back("Vector Resource 2", true);  // Destructor won't throw
    resources.emplace_back("Vector Resource 3");
    
    // Vector will destroy all elements safely
    std::cout << "Vector going out of scope..." << std::endl;
}

// ==================== Rule of Thumb ====================

class SafeClass {
private:
    std::unique_ptr<int[]> data;
    DatabaseConnection db;
    
public:
    SafeClass() : data(std::make_unique<int[]>(100)) {}
    
    // Destructor is noexcept by default with smart pointers
    ~SafeClass() = default;  // noexcept
    
    // If custom cleanup might throw, provide separate method
    void close() {
        db.disconnect();  // May throw
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Destructor Exception Rules ===" << std::endl;
    
    // Demonstrate bad practice
    // demonstrateBadDestructor();  // Uncommenting may cause termination
    
    // Demonstrate good practice
    demonstrateGoodDestructor();
    
    containerWithExceptions();
    
    std::cout << "\n=== Best Practices ===" << std::endl;
    std::cout << "1. Mark destructors noexcept" << std::endl;
    std::cout << "2. Catch and handle all exceptions in destructors" << std::endl;
    std::cout << "3. Use RAII and smart pointers" << std::endl;
    std::cout << "4. Provide separate cleanup methods for operations that may throw" << std::endl;
    
    return 0;
}
```

**Key Rules:**

| Scenario | Behavior | Consequence |
| :--- | :--- | :--- |
| **Normal destruction** | Destructor throws | Exception propagates |
| **Stack unwinding** | Destructor throws | `std::terminate` called |
| **Container destruction** | Destructor throws | Undefined behavior |
| **Noexcept destructor** | Safe | Resources properly released |

---

### 91. What is multiple dispatch?

**Multiple dispatch** (also called **multimethods**) is a feature where the function called depends on the **runtime types of multiple objects**, not just one. C++ supports only **single dispatch** (virtual functions on one object) natively.

```cpp
#include <iostream>
#include <memory>
#include <typeinfo>
#include <map>
#include <functional>

// ==================== Single Dispatch (C++ Native) ====================

class Shape {
public:
    virtual ~Shape() = default;
    virtual std::string type() const = 0;
};

class Circle : public Shape {
public:
    std::string type() const override { return "Circle"; }
};

class Rectangle : public Shape {
public:
    std::string type() const override { return "Rectangle"; }
};

class Triangle : public Shape {
public:
    std::string type() const override { return "Triangle"; }
};

// ==================== Problem: Need Double Dispatch ====================

class CollisionDetector {
public:
    // Single dispatch only uses the type of 'this'
    virtual bool collide(const Shape& other) const {
        std::cout << "Generic collision" << std::endl;
        return false;
    }
};

// ==================== Solution 1: Visitor Pattern (Double Dispatch) ====================

class CircleCollisionDetector;
class RectangleCollisionDetector;
class TriangleCollisionDetector;

class CollisionVisitor {
public:
    virtual ~CollisionVisitor() = default;
    virtual bool visit(const Circle& c, const CircleCollisionDetector& d) const = 0;
    virtual bool visit(const Rectangle& r, const RectangleCollisionDetector& d) const = 0;
    virtual bool visit(const Triangle& t, const TriangleCollisionDetector& d) const = 0;
};

// Base detector with accept method
class CollisionDetector2 {
public:
    virtual ~CollisionDetector2() = default;
    virtual bool accept(const Shape& s, const CollisionVisitor& visitor) const = 0;
};

// Circle detector
class CircleCollisionDetector : public CollisionDetector2 {
public:
    bool accept(const Shape& s, const CollisionVisitor& visitor) const override {
        // Double dispatch: dispatch on both this and s
        if (auto* c = dynamic_cast<const Circle*>(&s)) {
            return visitor.visit(*c, *this);
        }
        if (auto* r = dynamic_cast<const Rectangle*>(&s)) {
            return visitor.visit(*r, *this);
        }
        if (auto* t = dynamic_cast<const Triangle*>(&s)) {
            return visitor.visit(*t, *this);
        }
        return false;
    }
    
    // Circle-specific collision logic
    bool collideWithCircle(const Circle& other) const {
        std::cout << "Circle-Circle collision" << std::endl;
        return true;
    }
    
    bool collideWithRectangle(const Rectangle& other) const {
        std::cout << "Circle-Rectangle collision" << std::endl;
        return true;
    }
    
    bool collideWithTriangle(const Triangle& other) const {
        std::cout << "Circle-Triangle collision" << std::endl;
        return true;
    }
};

// Similar for other detectors...
class RectangleCollisionDetector : public CollisionDetector2 {
public:
    bool accept(const Shape& s, const CollisionVisitor& visitor) const override {
        if (auto* c = dynamic_cast<const Circle*>(&s)) {
            return visitor.visit(*c, *this);
        }
        if (auto* r = dynamic_cast<const Rectangle*>(&s)) {
            return visitor.visit(*r, *this);
        }
        if (auto* t = dynamic_cast<const Triangle*>(&s)) {
            return visitor.visit(*t, *this);
        }
        return false;
    }
    
    bool collideWithCircle(const Circle& other) const {
        std::cout << "Rectangle-Circle collision" << std::endl;
        return true;
    }
    
    bool collideWithRectangle(const Rectangle& other) const {
        std::cout << "Rectangle-Rectangle collision" << std::endl;
        return true;
    }
    
    bool collideWithTriangle(const Triangle& other) const {
        std::cout << "Rectangle-Triangle collision" << std::endl;
        return true;
    }
};

// Visitor implementation
class CollisionVisitorImpl : public CollisionVisitor {
public:
    bool visit(const Circle& c, const CircleCollisionDetector& d) const override {
        return d.collideWithCircle(c);
    }
    
    bool visit(const Rectangle& r, const CircleCollisionDetector& d) const override {
        return d.collideWithRectangle(r);
    }
    
    bool visit(const Triangle& t, const CircleCollisionDetector& d) const override {
        return d.collideWithTriangle(t);
    }
    
    bool visit(const Circle& c, const RectangleCollisionDetector& d) const override {
        return d.collideWithCircle(c);
    }
    
    bool visit(const Rectangle& r, const RectangleCollisionDetector& d) const override {
        return d.collideWithRectangle(r);
    }
    
    bool visit(const Triangle& t, const RectangleCollisionDetector& d) const override {
        return d.collideWithTriangle(t);
    }
};

// ==================== Solution 2: Dynamic Casting (Runtime Type Identification) ====================

class CollisionSystem {
public:
    static bool collide(const Shape& a, const Shape& b) {
        if (auto* ca = dynamic_cast<const Circle*>(&a)) {
            if (auto* cb = dynamic_cast<const Circle*>(&b)) {
                std::cout << "Circle-Circle collision" << std::endl;
                return true;
            }
            if (auto* rb = dynamic_cast<const Rectangle*>(&b)) {
                std::cout << "Circle-Rectangle collision" << std::endl;
                return true;
            }
            if (auto* tb = dynamic_cast<const Triangle*>(&b)) {
                std::cout << "Circle-Triangle collision" << std::endl;
                return true;
            }
        }
        if (auto* ra = dynamic_cast<const Rectangle*>(&a)) {
            if (auto* cb = dynamic_cast<const Circle*>(&b)) {
                std::cout << "Rectangle-Circle collision" << std::endl;
                return true;
            }
            if (auto* rb = dynamic_cast<const Rectangle*>(&b)) {
                std::cout << "Rectangle-Rectangle collision" << std::endl;
                return true;
            }
            if (auto* tb = dynamic_cast<const Triangle*>(&b)) {
                std::cout << "Rectangle-Triangle collision" << std::endl;
                return true;
            }
        }
        if (auto* ta = dynamic_cast<const Triangle*>(&a)) {
            if (auto* cb = dynamic_cast<const Circle*>(&b)) {
                std::cout << "Triangle-Circle collision" << std::endl;
                return true;
            }
            if (auto* rb = dynamic_cast<const Rectangle*>(&b)) {
                std::cout << "Triangle-Rectangle collision" << std::endl;
                return true;
            }
            if (auto* tb = dynamic_cast<const Triangle*>(&b)) {
                std::cout << "Triangle-Triangle collision" << std::endl;
                return true;
            }
        }
        return false;
    }
};

// ==================== Solution 3: Function Table (Multiple Dispatch Simulation) ====================

using CollisionFunc = std::function<bool(const Shape&, const Shape&)>;

class MultiDispatchCollision {
private:
    using TypePair = std::pair<size_t, size_t>;
    std::map<TypePair, CollisionFunc> dispatchTable;
    
    size_t getTypeId(const Shape& s) const {
        if (dynamic_cast<const Circle*>(&s)) return 1;
        if (dynamic_cast<const Rectangle*>(&s)) return 2;
        if (dynamic_cast<const Triangle*>(&s)) return 3;
        return 0;
    }
    
public:
    MultiDispatchCollision() {
        // Register all combinations
        dispatchTable[{1,1}] = [](const Shape& a, const Shape& b) {
            std::cout << "Circle-Circle collision" << std::endl; return true;
        };
        dispatchTable[{1,2}] = [](const Shape& a, const Shape& b) {
            std::cout << "Circle-Rectangle collision" << std::endl; return true;
        };
        dispatchTable[{1,3}] = [](const Shape& a, const Shape& b) {
            std::cout << "Circle-Triangle collision" << std::endl; return true;
        };
        dispatchTable[{2,1}] = [](const Shape& a, const Shape& b) {
            std::cout << "Rectangle-Circle collision" << std::endl; return true;
        };
        dispatchTable[{2,2}] = [](const Shape& a, const Shape& b) {
            std::cout << "Rectangle-Rectangle collision" << std::endl; return true;
        };
        dispatchTable[{2,3}] = [](const Shape& a, const Shape& b) {
            std::cout << "Rectangle-Triangle collision" << std::endl; return true;
        };
        dispatchTable[{3,1}] = [](const Shape& a, const Shape& b) {
            std::cout << "Triangle-Circle collision" << std::endl; return true;
        };
        dispatchTable[{3,2}] = [](const Shape& a, const Shape& b) {
            std::cout << "Triangle-Rectangle collision" << std::endl; return true;
        };
        dispatchTable[{3,3}] = [](const Shape& a, const Shape& b) {
            std::cout << "Triangle-Triangle collision" << std::endl; return true;
        };
    }
    
    bool collide(const Shape& a, const Shape& b) const {
        auto it = dispatchTable.find({getTypeId(a), getTypeId(b)});
        if (it != dispatchTable.end()) {
            return it->second(a, b);
        }
        return false;
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Multiple Dispatch Demonstration ===" << std::endl;
    
    Circle c;
    Rectangle r;
    Triangle t;
    
    std::cout << "\n=== Method 1: Visitor Pattern ===" << std::endl;
    CircleCollisionDetector cd;
    RectangleCollisionDetector rd;
    CollisionVisitorImpl visitor;
    
    cd.accept(c, visitor);  // Circle-Circle
    cd.accept(r, visitor);  // Circle-Rectangle
    cd.accept(t, visitor);  // Circle-Triangle
    
    std::cout << "\n=== Method 2: Dynamic Cast ===" << std::endl;
    CollisionSystem::collide(c, r);
    CollisionSystem::collide(r, t);
    CollisionSystem::collide(t, c);
    
    std::cout << "\n=== Method 3: Dispatch Table ===" << std::endl;
    MultiDispatchCollision multi;
    multi.collide(c, c);
    multi.collide(c, r);
    multi.collide(r, t);
    
    return 0;
}
```

---

### 92. What is dynamic_cast?

`dynamic_cast` is a C++ operator used for **safe downcasting** in polymorphic hierarchies. It performs **runtime type checking** and returns null or throws `bad_cast` if the cast is invalid.

```cpp
#include <iostream>
#include <typeinfo>
#include <vector>
#include <memory>

// ==================== Base Hierarchy ====================

class Animal {
public:
    virtual ~Animal() = default;
    virtual void speak() const {
        std::cout << "Animal speaks" << std::endl;
    }
};

class Dog : public Animal {
public:
    void speak() const override {
        std::cout << "Dog barks: Woof!" << std::endl;
    }
    
    void wagTail() const {
        std::cout << "Dog wags tail" << std::endl;
    }
};

class Cat : public Animal {
public:
    void speak() const override {
        std::cout << "Cat meows: Meow!" << std::endl;
    }
    
    void purr() const {
        std::cout << "Cat purrs" << std::endl;
    }
};

class Bird : public Animal {
public:
    void speak() const override {
        std::cout << "Bird chirps: Tweet!" << std::endl;
    }
    
    void fly() const {
        std::cout << "Bird flies away" << std::endl;
    }
};

// ==================== dynamic_cast for Pointers ====================

void processAnimalPointer(Animal* animal) {
    std::cout << "\nProcessing animal pointer..." << std::endl;
    
    // Try to cast to Dog*
    if (Dog* dog = dynamic_cast<Dog*>(animal)) {
        std::cout << "It's a dog! ";
        dog->speak();
        dog->wagTail();
    }
    // Try to cast to Cat*
    else if (Cat* cat = dynamic_cast<Cat*>(animal)) {
        std::cout << "It's a cat! ";
        cat->speak();
        cat->purr();
    }
    // Try to cast to Bird*
    else if (Bird* bird = dynamic_cast<Bird*>(animal)) {
        std::cout << "It's a bird! ";
        bird->speak();
        bird->fly();
    }
    else {
        std::cout << "Unknown animal: ";
        animal->speak();
    }
}

// ==================== dynamic_cast for References ====================

void processAnimalReference(Animal& animal) {
    std::cout << "\nProcessing animal reference..." << std::endl;
    
    try {
        // dynamic_cast on references throws bad_cast on failure
        Dog& dog = dynamic_cast<Dog&>(animal);
        std::cout << "It's a dog! ";
        dog.speak();
        dog.wagTail();
    } catch (const std::bad_cast& e) {
        try {
            Cat& cat = dynamic_cast<Cat&>(animal);
            std::cout << "It's a cat! ";
            cat.speak();
            cat.purr();
        } catch (const std::bad_cast& e) {
            try {
                Bird& bird = dynamic_cast<Bird&>(animal);
                std::cout << "It's a bird! ";
                bird.speak();
                bird.fly();
            } catch (const std::bad_cast& e) {
                std::cout << "Unknown animal: ";
                animal.speak();
            }
        }
    }
}

// ==================== Cross Casting ====================

class Flyable {
public:
    virtual ~Flyable() = default;
    virtual void fly() = 0;
};

class Swimmable {
public:
    virtual ~Swimmable() = default;
    virtual void swim() = 0;
};

class Duck : public Animal, public Flyable, public Swimmable {
public:
    void speak() const override {
        std::cout << "Duck quacks: Quack!" << std::endl;
    }
    
    void fly() override {
        std::cout << "Duck flies" << std::endl;
    }
    
    void swim() override {
        std::cout << "Duck swims" << std::endl;
    }
};

void crossCastExample() {
    std::cout << "\n=== Cross Casting ===" << std::endl;
    
    Duck duck;
    Animal* animal = &duck;
    
    // Cross-cast from Animal* to Flyable*
    if (Flyable* flyable = dynamic_cast<Flyable*>(animal)) {
        std::cout << "Animal can fly: ";
        flyable->fly();
    }
    
    // Cross-cast from Animal* to Swimmable*
    if (Swimmable* swimmable = dynamic_cast<Swimmable*>(animal)) {
        std::cout << "Animal can swim: ";
        swimmable->swim();
    }
}

// ==================== dynamic_cast with void* ====================

void voidPointerCast() {
    std::cout << "\n=== dynamic_cast with void* ===" << std::endl;
    
    Dog dog;
    void* voidPtr = &dog;
    
    // Cannot dynamic_cast from void* directly
    // Dog* dogPtr = dynamic_cast<Dog*>(voidPtr);  // Error
    
    // Must cast back to original type first
    Animal* animalPtr = static_cast<Animal*>(voidPtr);
    if (Dog* dogPtr = dynamic_cast<Dog*>(animalPtr)) {
        std::cout << "Successfully cast back to Dog" << std::endl;
        dogPtr->wagTail();
    }
}

// ==================== Performance Considerations ====================

void performanceExample() {
    std::cout << "\n=== Performance Considerations ===" << std::endl;
    
    std::vector<Animal*> zoo;
    zoo.push_back(new Dog());
    zoo.push_back(new Cat());
    zoo.push_back(new Bird());
    zoo.push_back(new Duck());
    
    // dynamic_cast has runtime overhead (RTTI lookup)
    for (auto animal : zoo) {
        if (Dog* dog = dynamic_cast<Dog*>(animal)) {
            dog->wagTail();
        } else if (Duck* duck = dynamic_cast<Duck*>(animal)) {
            duck->fly();
            duck->swim();
        } else {
            animal->speak();
        }
    }
    
    // Cleanup
    for (auto animal : zoo) delete animal;
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== dynamic_cast Demonstration ===" << std::endl;
    
    Dog dog;
    Cat cat;
    Bird bird;
    Animal genericAnimal;
    
    std::cout << "\n=== Pointer dynamic_cast ===" << std::endl;
    processAnimalPointer(&dog);
    processAnimalPointer(&cat);
    processAnimalPointer(&bird);
    processAnimalPointer(&genericAnimal);
    
    std::cout << "\n=== Reference dynamic_cast ===" << std::endl;
    processAnimalReference(dog);
    processAnimalReference(cat);
    processAnimalReference(bird);
    processAnimalReference(genericAnimal);
    
    crossCastExample();
    voidPointerCast();
    performanceExample();
    
    std::cout << "\n=== Summary ===" << std::endl;
    std::cout << "1. Requires polymorphic base class (virtual functions)" << std::endl;
    std::cout << "2. Returns nullptr for pointer casts on failure" << std::endl;
    std::cout << "3. Throws std::bad_cast for reference casts on failure" << std::endl;
    std::cout << "4. Supports cross-casting in multiple inheritance" << std::endl;
    std::cout << "5. Has runtime overhead (RTTI)" << std::endl;
    
    return 0;
}
```

---

### 93. What is typeid and RTTI?

**RTTI (Runtime Type Information)** is a C++ feature that allows **type identification at runtime**. `typeid` is the operator used to access this information.

```cpp
#include <iostream>
#include <typeinfo>
#include <vector>
#include <memory>
#include <string>

// ==================== Basic typeid Usage ====================

class Base {
public:
    virtual ~Base() = default;  // Required for RTTI
};

class Derived : public Base {};

class AnotherClass {};

// ==================== typeid with Built-in Types ====================

void builtinTypeid() {
    std::cout << "\n=== Built-in Types ===" << std::endl;
    
    int i = 42;
    double d = 3.14;
    std::string s = "Hello";
    
    std::cout << "typeid(int).name(): " << typeid(int).name() << std::endl;
    std::cout << "typeid(i).name(): " << typeid(i).name() << std::endl;
    std::cout << "typeid(double).name(): " << typeid(double).name() << std::endl;
    std::cout << "typeid(d).name(): " << typeid(d).name() << std::endl;
    std::cout << "typeid(std::string).name(): " << typeid(std::string).name() << std::endl;
    std::cout << "typeid(s).name(): " << typeid(s).name() << std::endl;
}

// ==================== typeid with Polymorphic Classes ====================

void polymorphicTypeid() {
    std::cout << "\n=== Polymorphic Classes ===" << std::endl;
    
    Base base;
    Derived derived;
    Base* basePtr = &derived;
    
    std::cout << "typeid(Base).name(): " << typeid(Base).name() << std::endl;
    std::cout << "typeid(base).name(): " << typeid(base).name() << std::endl;
    std::cout << "typeid(Derived).name(): " << typeid(Derived).name() << std::endl;
    std::cout << "typeid(derived).name(): " << typeid(derived).name() << std::endl;
    
    // With polymorphism - shows dynamic type
    std::cout << "typeid(*basePtr).name(): " << typeid(*basePtr).name() << std::endl;
    
    // Static type of pointer
    std::cout << "typeid(basePtr).name(): " << typeid(basePtr).name() << std::endl;
}

// ==================== Comparing types ====================

void typeComparison() {
    std::cout << "\n=== Type Comparison ===" << std::endl;
    
    Base base;
    Derived derived;
    Base* basePtr1 = &base;
    Base* basePtr2 = &derived;
    
    // Compare types
    if (typeid(base) == typeid(Derived)) {
        std::cout << "base is Derived" << std::endl;
    } else {
        std::cout << "base is not Derived" << std::endl;
    }
    
    if (typeid(*basePtr1) == typeid(Base)) {
        std::cout << "*basePtr1 is Base" << std::endl;
    }
    
    if (typeid(*basePtr2) == typeid(Derived)) {
        std::cout << "*basePtr2 is Derived" << std::endl;
    }
}

// ==================== type_info Methods ====================

void typeInfoMethods() {
    std::cout << "\n=== type_info Methods ===" << std::endl;
    
    const std::type_info& intInfo = typeid(int);
    const std::type_info& doubleInfo = typeid(double);
    const std::type_info& stringInfo = typeid(std::string);
    
    // name() - implementation-defined type name
    std::cout << "int name: " << intInfo.name() << std::endl;
    std::cout << "double name: " << doubleInfo.name() << std::endl;
    std::cout << "string name: " << stringInfo.name() << std::endl;
    
    // hash_code() - unique hash value (C++11)
    std::cout << "int hash: " << intInfo.hash_code() << std::endl;
    std::cout << "double hash: " << doubleInfo.hash_code() << std::endl;
    
    // before() - ordering (implementation-defined)
    std::cout << "int before double? " << intInfo.before(doubleInfo) << std::endl;
    
    // operator== and operator!=
    std::cout << "int == double? " << (intInfo == doubleInfo) << std::endl;
}

// ==================== Practical Example: Type-based Factory ====================

class Shape {
public:
    virtual ~Shape() = default;
    virtual void draw() const = 0;
    virtual std::string getType() const = 0;
};

class Circle : public Shape {
public:
    void draw() const override { std::cout << "Drawing Circle" << std::endl; }
    std::string getType() const override { return "Circle"; }
};

class Rectangle : public Shape {
public:
    void draw() const override { std::cout << "Drawing Rectangle" << std::endl; }
    std::string getType() const override { return "Rectangle"; }
};

class ShapeFactory {
public:
    static std::unique_ptr<Shape> createShape(const std::type_info& type) {
        if (type == typeid(Circle)) {
            return std::make_unique<Circle>();
        } else if (type == typeid(Rectangle)) {
            return std::make_unique<Rectangle>();
        }
        return nullptr;
    }
};

// ==================== Type Identification in Templates ====================

template <typename T>
void templateTypeInfo(const T& value) {
    std::cout << "Template type: " << typeid(T).name() << std::endl;
    std::cout << "Value type: " << typeid(value).name() << std::endl;
}

// ==================== Bad Cast Example ====================

void badCastExample() {
    std::cout << "\n=== Bad Cast Handling ===" << std::endl;
    
    Base base;
    
    try {
        // This will throw std::bad_cast
        Derived& derived = dynamic_cast<Derived&>(base);
    } catch (const std::bad_cast& e) {
        std::cout << "Bad cast caught: " << e.what() << std::endl;
    }
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== RTTI and typeid Demonstration ===" << std::endl;
    
    builtinTypeid();
    polymorphicTypeid();
    typeComparison();
    typeInfoMethods();
    
    std::cout << "\n=== Practical Example ===" << std::endl;
    
    Circle circle;
    Rectangle rect;
    
    const std::type_info& circleType = typeid(circle);
    const std::type_info& rectType = typeid(rect);
    
    auto shape1 = ShapeFactory::createShape(circleType);
    auto shape2 = ShapeFactory::createShape(rectType);
    
    if (shape1) shape1->draw();
    if (shape2) shape2->draw();
    
    std::cout << "\n=== Template Type Info ===" << std::endl;
    templateTypeInfo(42);
    templateTypeInfo(3.14);
    templateTypeInfo(circle);
    
    badCastExample();
    
    std::cout << "\n=== Important Notes ===" << std::endl;
    std::cout << "1. RTTI requires virtual functions" << std::endl;
    std::cout << "2. typeid on null pointer throws bad_typeid" << std::endl;
    std::cout << "3. type_info::name() returns implementation-defined string" << std::endl;
    std::cout << "4. RTTI can be disabled with -fno-rtti" << std::endl;
    
    return 0;
}
```

**RTTI Components:**

| Component | Purpose | When to Use |
| :--- | :--- | :--- |
| **typeid** | Get type information | Type identification |
| **dynamic_cast** | Safe downcasting | Polymorphic type conversion |
| **type_info** | Type comparison and info | Runtime type queries |

---

### 94. What is friend class use case?

A **friend class** is granted access to all private and protected members of another class. It's used when two classes are **tightly coupled** and need to share implementation details.

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <map>

// ==================== Use Case 1: Iterator Pattern ====================

template <typename T>
class Container {
private:
    std::vector<T> items;
    
public:
    void add(const T& item) {
        items.push_back(item);
    }
    
    // Iterator class needs access to private data
    class Iterator {
    private:
        Container<T>& container;
        size_t index;
        
    public:
        Iterator(Container<T>& cont) : container(cont), index(0) {}
        
        bool hasNext() const {
            return index < container.items.size();  // Access private vector
        }
        
        T& next() {
            return container.items[index++];
        }
    };
    
    // Container is friend of Iterator by default (inner class)
    Iterator getIterator() {
        return Iterator(*this);
    }
};

// ==================== Use Case 2: Factory Pattern ====================

class Product {
private:
    std::string name;
    double price;
    
    // Private constructor - only factory can create
    Product(const std::string& n, double p) : name(n), price(p) {}
    
public:
    void display() const {
        std::cout << "Product: " << name << ", $" << price << std::endl;
    }
    
    // Factory class needs access to private constructor
    friend class ProductFactory;
};

class ProductFactory {
public:
    static Product createDefault(const std::string& name) {
        return Product(name, 0.0);  // Can access private constructor
    }
    
    static Product createWithPrice(const std::string& name, double price) {
        return Product(name, price);
    }
};

// ==================== Use Case 3: Graph Algorithms ====================

class Graph {
private:
    struct Node {
        int id;
        std::vector<Node*> neighbors;
        Node(int i) : id(i) {}
    };
    
    std::vector<Node*> nodes;
    
public:
    Graph() = default;
    ~Graph() {
        for (auto node : nodes) delete node;
    }
    
    void addNode(int id) {
        nodes.push_back(new Node(id));
    }
    
    void addEdge(int from, int to) {
        if (from < nodes.size() && to < nodes.size()) {
            nodes[from]->neighbors.push_back(nodes[to]);
        }
    }
    
    // GraphAlgorithm needs access to Node internals
    friend class GraphAlgorithm;
};

class GraphAlgorithm {
public:
    static bool hasCycle(Graph& graph) {
        std::map<Graph::Node*, bool> visited;
        std::map<Graph::Node*, bool> inStack;
        
        for (auto node : graph.nodes) {
            if (detectCycle(node, visited, inStack)) {
                return true;
            }
        }
        return false;
    }
    
private:
    static bool detectCycle(Graph::Node* node, 
                           std::map<Graph::Node*, bool>& visited,
                           std::map<Graph::Node*, bool>& inStack) {
        if (!node) return false;
        
        visited[node] = true;
        inStack[node] = true;
        
        for (auto neighbor : node->neighbors) {  // Access private neighbors
            if (!visited[neighbor]) {
                if (detectCycle(neighbor, visited, inStack)) {
                    return true;
                }
            } else if (inStack[neighbor]) {
                return true;
            }
        }
        
        inStack[node] = false;
        return false;
    }
};

// ==================== Use Case 4: Coupled Classes ====================

class Engine {
private:
    int horsepower;
    bool running;
    
public:
    Engine(int hp) : horsepower(hp), running(false) {}
    
    void start() {
        running = true;
        std::cout << "Engine started" << std::endl;
    }
    
    // Car needs to access engine internals
    friend class Car;
};

class Car {
private:
    Engine engine;
    int speed;
    
public:
    Car(int hp) : engine(hp), speed(0) {}
    
    void accelerate() {
        if (!engine.running) {  // Access private Engine member
            engine.start();
        }
        speed += 10;
        std::cout << "Speed: " << speed << " km/h" << std::endl;
    }
    
    int getHorsepower() const {
        return engine.horsepower;  // Access private Engine member
    }
};

// ==================== Use Case 5: Builder Pattern ====================

class House {
private:
    std::string walls;
    std::string roof;
    std::string windows;
    bool hasGarage;
    bool hasGarden;
    
    // Private constructor - only builder can create
    House() : hasGarage(false), hasGarden(false) {}
    
public:
    void describe() const {
        std::cout << "House with " << walls << " walls, " << roof << " roof";
        if (hasGarage) std::cout << ", with garage";
        if (hasGarden) std::cout << ", with garden";
        std::cout << std::endl;
    }
    
    friend class HouseBuilder;
};

class HouseBuilder {
private:
    House house;
    
public:
    HouseBuilder& setWalls(const std::string& type) {
        house.walls = type;  // Access private House members
        return *this;
    }
    
    HouseBuilder& setRoof(const std::string& type) {
        house.roof = type;
        return *this;
    }
    
    HouseBuilder& setWindows(const std::string& type) {
        house.windows = type;
        return *this;
    }
    
    HouseBuilder& addGarage() {
        house.hasGarage = true;
        return *this;
    }
    
    HouseBuilder& addGarden() {
        house.hasGarden = true;
        return *this;
    }
    
    House build() {
        return house;
    }
};

// ==================== Use Case 6: Unit Testing ====================

class ProductionCode {
private:
    int secretValue;
    std::string internalState;
    
    void complexInternalFunction() {
        // Complex logic
    }
    
public:
    ProductionCode() : secretValue(42), internalState("initial") {}
    
    int publicAPI() {
        complexInternalFunction();
        return secretValue * 2;
    }
    
    // Test fixture needs access for white-box testing
    friend class ProductionCodeTest;
};

class ProductionCodeTest {
public:
    static void runTests() {
        ProductionCode code;
        
        // Access private members for testing
        if (code.secretValue == 42) {
            std::cout << "Test 1 passed" << std::endl;
        }
        
        code.complexInternalFunction();  // Test private method
        
        code.internalState = "test";  // Modify private state
        // ... verify behavior
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Friend Class Use Cases ===" << std::endl;
    
    std::cout << "\n1. Iterator Pattern:" << std::endl;
    Container<int> container;
    container.add(10);
    container.add(20);
    container.add(30);
    
    auto it = container.getIterator();
    while (it.hasNext()) {
        std::cout << it.next() << " ";
    }
    std::cout << std::endl;
    
    std::cout << "\n2. Factory Pattern:" << std::endl;
    auto p1 = ProductFactory::createDefault("Widget");
    auto p2 = ProductFactory::createWithPrice("Gadget", 19.99);
    p1.display();
    p2.display();
    
    std::cout << "\n3. Coupled Classes:" << std::endl;
    Car car(200);
    car.accelerate();
    std::cout << "Horsepower: " << car.getHorsepower() << std::endl;
    
    std::cout << "\n4. Builder Pattern:" << std::endl;
    HouseBuilder builder;
    House house = builder.setWalls("brick")
                        .setRoof("tile")
                        .addGarage()
                        .addGarden()
                        .build();
    house.describe();
    
    std::cout << "\n5. Unit Testing:" << std::endl;
    ProductionCodeTest::runTests();
    
    return 0;
}
```

**When to Use Friend Classes:**

| Use Case | Why Friend? |
| :--- | :--- |
| **Iterators** | Need access to container internals |
| **Factories** | Need to call private constructors |
| **Graph algorithms** | Need access to node internals |
| **Tightly coupled classes** | Engine/Car relationship |
| **Builder pattern** | Need to set private members |
| **Unit testing** | White-box testing access |

---

### 95. What is the difference between composition and inheritance?

**Composition** and **inheritance** are two fundamental techniques for code reuse in OOP. Here's a comprehensive comparison:

```cpp
#include <iostream>
#include <string>
#include <memory>
#include <vector>

// ==================== Inheritance (IS-A) ====================

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

class Dog : public Animal {  // Dog IS-A Animal
private:
    std::string breed;
    
public:
    Dog(const std::string& n, const std::string& b) 
        : Animal(n), breed(b) {}
    
    void speak() const override {
        std::cout << name << " the " << breed << " barks: Woof!" << std::endl;
    }
    
    void wagTail() const {
        std::cout << name << " wags tail" << std::endl;
    }
};

// ==================== Composition (HAS-A) ====================

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

class Car {  // Car HAS-A Engine, HAS-A Wheels
private:
    std::unique_ptr<Engine> engine;
    std::unique_ptr<Wheels> wheels;
    std::string model;
    
public:
    Car(const std::string& m, int hp, int wheelCount)
        : engine(std::make_unique<Engine>(hp)),
          wheels(std::make_unique<Wheels>(wheelCount)),
          model(m) {}
    
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

// ==================== When Both Are Appropriate ====================

// Inheritance approach
class Employee {
protected:
    std::string name;
    double salary;
    
public:
    Employee(const std::string& n, double s) : name(n), salary(s) {}
    virtual void work() const {
        std::cout << name << " works" << std::endl;
    }
};

class Manager : public Employee {
private:
    int teamSize;
    
public:
    Manager(const std::string& n, double s, int team) 
        : Employee(n, s), teamSize(team) {}
    
    void work() const override {
        std::cout << name << " manages a team of " << teamSize << std::endl;
    }
    
    void holdMeeting() const {
        std::cout << name << " holds a meeting" << std::endl;
    }
};

// Composition approach
class Logger {
public:
    void log(const std::string& message) const {
        std::cout << "LOG: " << message << std::endl;
    }
};

class ReportGenerator {
private:
    std::unique_ptr<Logger> logger;
    std::string title;
    
public:
    ReportGenerator(const std::string& t) 
        : logger(std::make_unique<Logger>()), title(t) {}
    
    void generate() {
        logger->log("Generating report: " + title);
        // Generate report...
        logger->log("Report generated");
    }
    
    // Can easily change logging behavior
    void setLogger(std::unique_ptr<Logger> newLogger) {
        logger = std::move(newLogger);
    }
};

// ==================== Comparison Table ====================

void demonstrateDifferences() {
    std::cout << "\n=== Inheritance (IS-A) ===" << std::endl;
    Dog dog("Rex", "Golden Retriever");
    dog.speak();
    dog.move();
    dog.wagTail();
    
    std::cout << "\n=== Composition (HAS-A) ===" << std::endl;
    Car car("Tesla", 500, 4);
    car.start();
    car.drive();
    
    // Can't change inherited behavior at runtime
    std::cout << "\n=== Runtime Flexibility ===" << std::endl;
    auto electricEngine = std::make_unique<Engine>(600);
    car.setEngine(std::move(electricEngine));  // Easy with composition
    car.start();
}

// ==================== When to Use Each ====================

// BAD: Using inheritance when composition is better
class Database {
public:
    void connect() { std::cout << "Connecting to DB" << std::endl; }
    void query() { std::cout << "Running query" << std::endl; }
};

class UserService : public Database {  // Wrong! UserService IS NOT a Database
public:
    void createUser() {
        connect();
        query();
        std::cout << "User created" << std::endl;
    }
};

// GOOD: Using composition
class UserService2 {
private:
    std::unique_ptr<Database> db;  // HAS-A Database
    
public:
    UserService2() : db(std::make_unique<Database>()) {}
    
    void createUser() {
        db->connect();
        db->query();
        std::cout << "User created" << std::endl;
    }
};

// ==================== Comparison in Design ====================

// Inheritance for polymorphism
void processAnimal(const Animal& animal) {
    animal.speak();
    animal.move();
}

// Composition for strategy pattern
class TextProcessor {
private:
    class Strategy {
    public:
        virtual ~Strategy() = default;
        virtual std::string process(const std::string&) = 0;
    };
    
    class UpperCaseStrategy : public Strategy {
        std::string process(const std::string& s) override {
            std::string result;
            for (char c : s) result += toupper(c);
            return result;
        }
    };
    
    class LowerCaseStrategy : public Strategy {
        std::string process(const std::string& s) override {
            std::string result;
            for (char c : s) result += tolower(c);
            return result;
        }
    };
    
    std::unique_ptr<Strategy> strategy;
    
public:
    TextProcessor() : strategy(std::make_unique<UpperCaseStrategy>()) {}
    
    void setToUpper() { strategy = std::make_unique<UpperCaseStrategy>(); }
    void setToLower() { strategy = std::make_unique<LowerCaseStrategy>(); }
    
    std::string process(const std::string& input) {
        return strategy->process(input);
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Composition vs Inheritance ===" << std::endl;
    
    demonstrateDifferences();
    
    std::cout << "\n=== Key Differences ===" << std::endl;
    std::cout << "┌──────────────────────────┬──────────────────────────┐" << std::endl;
    std::cout << "│ Inheritance              │ Composition              │" << std::endl;
    std::cout << "├──────────────────────────┼──────────────────────────┤" << std::endl;
    std::cout << "│ IS-A relationship        │ HAS-A relationship       │" << std::endl;
    std::cout << "│ Static binding           │ Dynamic binding possible │" << std::endl;
    std::cout << "│ White-box reuse          │ Black-box reuse          │" << std::endl;
    std::cout << "│ Can't change at runtime  │ Can swap components      │" << std::endl;
    std::cout << "│ Deep hierarchies         │ Shallow hierarchies      │" << std::endl;
    std::cout << "│ Exposes internals        │ Encapsulates internals   │" << std::endl;
    std::cout << "└──────────────────────────┴──────────────────────────┘" << std::endl;
    
    std::cout << "\n=== When to Use ===" << std::endl;
    std::cout << "Use Inheritance when:" << std::endl;
    std::cout << "  - Clear IS-A relationship" << std::endl;
    std::cout << "  - Need polymorphic behavior" << std::endl;
    std::cout << "  - Base class provides core functionality" << std::endl;
    
    std::cout << "\nUse Composition when:" << std::endl;
    std::cout << "  - Clear HAS-A relationship" << std::endl;
    std::cout << "  - Need runtime flexibility" << std::endl;
    std::cout << "  - Want to hide implementation" << std::endl;
    std::cout << "  - Following 'prefer composition over inheritance'" << std::endl;
    
    return 0;
}
```

**Summary Table:**

| Aspect | Inheritance | Composition |
| :--- | :--- | :--- |
| **Relationship** | IS-A | HAS-A |
| **Binding time** | Compile-time | Runtime (can change) |
| **Reuse** | White-box (exposes parent) | Black-box (hides components) |
| **Flexibility** | Limited (static) | High (can swap) |
| **Coupling** | Tight (depends on parent) | Loose (depends on interface) |
| **Testing** | Harder (parent dependencies) | Easier (can mock) |
| **Change impact** | Affects hierarchy | Local to class |

---

### 96. Can C++ support OOP without classes?

**No, C++ cannot support OOP without classes** because classes are fundamental to OOP in C++. However, C++ is a multi-paradigm language and can support **object-based programming** using structs, which are essentially classes with default public access.

```cpp
#include <iostream>
#include <string>
#include <vector>

// ==================== OOP Requires Classes ====================

// This IS using classes (struct is a class in C++)
struct Point {
    double x, y;
    
    // Constructor
    Point(double x = 0, double y = 0) : x(x), y(y) {}
    
    // Method
    double distance() const {
        return sqrt(x*x + y*y);
    }
};

// ==================== Object-Based Programming with Structs ====================

// Struct with member functions (still a class in C++)
struct Rectangle {
    double width;
    double height;
    
    // Constructor
    Rectangle(double w, double h) : width(w), height(h) {}
    
    // Member function
    double area() const {
        return width * height;
    }
    
    double perimeter() const {
        return 2 * (width + height);
    }
};

// ==================== Without Classes (C-style) ====================

// C-style struct (no member functions)
struct CPoint {
    double x;
    double y;
};

// Free functions operating on the struct
double point_distance(const CPoint* p) {
    return sqrt(p->x * p->x + p->y * p->y);
}

void point_init(CPoint* p, double x, double y) {
    p->x = x;
    p->y = y;
}

// ==================== Without Any Structs (Pure Procedural) ====================

// Using parallel arrays
double points_x[100];
double points_y[100];
int point_count = 0;

void add_point(double x, double y) {
    points_x[point_count] = x;
    points_y[point_count] = y;
    point_count++;
}

double get_point_distance(int index) {
    return sqrt(points_x[index] * points_x[index] + 
                points_y[index] * points_y[index]);
}

// ==================== Simulating OOP with Function Pointers ====================

// Manual vtable simulation
struct ShapeVTable {
    double (*area)(void*);
    void (*draw)(void*);
};

struct Shape {
    ShapeVTable* vtable;
};

struct Circle {
    Shape base;
    double radius;
};

double circle_area(void* obj) {
    Circle* c = static_cast<Circle*>(obj);
    return 3.14159 * c->radius * c->radius;
}

void circle_draw(void* obj) {
    Circle* c = static_cast<Circle*>(obj);
    std::cout << "Drawing circle with radius " << c->radius << std::endl;
}

ShapeVTable circle_vtable = {circle_area, circle_draw};

Circle* create_circle(double r) {
    Circle* c = new Circle;
    c->base.vtable = &circle_vtable;
    c->radius = r;
    return c;
}

void process_shape(Shape* shape) {
    std::cout << "Area: " << shape->vtable->area(shape) << std::endl;
    shape->vtable->draw(shape);
}

// ==================== Template-Based Generic Programming ====================

template <typename T>
double generic_area(const T& shape) {
    return shape.area();
}

template <typename T>
void generic_draw(const T& shape) {
    shape.draw();
}

class ModernCircle {
private:
    double radius;
    
public:
    ModernCircle(double r) : radius(r) {}
    
    double area() const {
        return 3.14159 * radius * radius;
    }
    
    void draw() const {
        std::cout << "Drawing modern circle" << std::endl;
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== C++ Requires Classes/Structs for OOP ===" << std::endl;
    
    std::cout << "\n1. Using proper C++ class:" << std::endl;
    Point p(3, 4);
    std::cout << "Point distance: " << p.distance() << std::endl;
    
    std::cout << "\n2. Using struct (still a class in C++):" << std::endl;
    Rectangle rect(5, 3);
    std::cout << "Rectangle area: " << rect.area() << std::endl;
    
    std::cout << "\n3. C-style struct with free functions:" << std::endl;
    CPoint cp;
    point_init(&cp, 3, 4);
    std::cout << "C-style distance: " << point_distance(&cp) << std::endl;
    
    std::cout << "\n4. Pure procedural (parallel arrays):" << std::endl;
    add_point(3, 4);
    add_point(5, 12);
    std::cout << "Point 0 distance: " << get_point_distance(0) << std::endl;
    std::cout << "Point 1 distance: " << get_point_distance(1) << std::endl;
    
    std::cout << "\n5. Simulating OOP with function pointers:" << std::endl;
    Circle* c = create_circle(5);
    process_shape(reinterpret_cast<Shape*>(c));
    delete c;
    
    std::cout << "\n6. Template-based generic programming:" << std::endl;
    ModernCircle mc(3);
    std::cout << "Template area: " << generic_area(mc) << std::endl;
    generic_draw(mc);
    
    std::cout << "\n=== Conclusion ===" << std::endl;
    std::cout << "C++ cannot support true OOP without classes, but offers:" << std::endl;
    std::cout << "- Classes (with full OOP features)" << std::endl;
    std::cout << "- Structs (essentially classes with public default)" << std::endl;
    std::cout << "- Procedural programming (C-style)" << std::endl;
    std::cout << "- Generic programming (templates)" << std::endl;
    std::cout << "- Functional programming (C++11+)" << std::endl;
    
    return 0;
}
```

**Key Points:**

| Approach | OOP Support | Features |
| :--- | :--- | :--- |
| **Classes** | Full OOP | Encapsulation, inheritance, polymorphism |
| **Structs** | Full OOP (syntactic difference) | Same as classes |
| **C-style structs** | Object-based (no methods) | Data only, free functions |
| **Parallel arrays** | No OOP | Pure procedural |
| **Function pointers** | Simulated OOP | Manual vtable implementation |
| **Templates** | Generic programming | Compile-time polymorphism |

---

### 97. What are access specifiers default for template class?

The **default access specifier for template classes** follows the same rules as regular classes in C++ - **private by default**. This applies to both the template parameters and the class members.

```cpp
#include <iostream>
#include <type_traits>

// ==================== Default Access in Template Classes ====================

template <typename T>
class TemplateClass {
    // Everything here is PRIVATE by default
    T data;  // private
    void privateFunction() {}  // private
    
public:
    void publicFunction() {}  // explicitly public
};

template <typename T>
struct TemplateStruct {
    // Everything here is PUBLIC by default
    T data;  // public
    void publicFunction() {}  // public
};

// ==================== Demonstration ====================

template <typename T>
class Container {
    // Private by default
    T* elements;
    size_t size;
    
    void privateResize() {
        std::cout << "Private resize called" << std::endl;
    }
    
public:
    Container(size_t s) : size(s), elements(new T[s]) {}
    
    ~Container() {
        delete[] elements;
    }
    
    T& operator[](size_t index) {
        return elements[index];
    }
};

// ==================== Template Parameter Access ====================

template <typename T>
class Wrapper {
    T value;  // private by default
    
public:
    Wrapper(const T& v) : value(v) {}
    
    T& getValue() { return value; }
};

// ==================== Access in Template Specialization ====================

template <typename T>
class SpecializedAccess {
    T data;  // private
    
public:
    SpecializedAccess(const T& d) : data(d) {}
    T getData() const { return data; }
};

// Specialization - access specifiers can change
template <>
class SpecializedAccess<int> {
    int data;  // still private
    int extra;  // private
    
public:
    SpecializedAccess(int d) : data(d), extra(d * 2) {}
    int getData() const { return data; }
    int getExtra() const { return extra; }  // public
};

// ==================== Access in Template Inheritance ====================

template <typename T>
class Base {
    T baseData;  // private
    
protected:
    T protectedData;  // protected
    
public:
    Base(const T& d) : baseData(d), protectedData(d) {}
    T getBaseData() const { return baseData; }
};

template <typename T>
class Derived : public Base<T> {
    T derivedData;  // private
    
public:
    Derived(const T& d) : Base<T>(d), derivedData(d) {}
    
    void access() {
        // derivedData is accessible (private here)
        // baseData is NOT accessible (private in base)
        // protectedData IS accessible (protected in base)
        T val = this->protectedData;  // OK
        std::cout << "Protected data: " << val << std::endl;
    }
};

// ==================== Access in Template Friends ====================

template <typename T>
class FriendDemo {
    T secret;  // private
    
    friend class FriendClass;
    template <typename U>
    friend void friendFunction(FriendDemo<U>& demo, U value);
    
public:
    FriendDemo() : secret(T()) {}
};

class FriendClass {
public:
    template <typename T>
    void modify(FriendDemo<T>& demo, T value) {
        demo.secret = value;  // Access private member
    }
};

template <typename U>
void friendFunction(FriendDemo<U>& demo, U value) {
    demo.secret = value;  // Access private member
}

// ==================== SFINAE and Access ====================

template <typename T>
class HasPrivate {
private:
    using yes = char;
    using no = char[2];
    
    template <typename U>
    static yes test(decltype(U::privateMethod)*);
    
    template <typename U>
    static no test(...);
    
public:
    static constexpr bool value = sizeof(test<T>(0)) == sizeof(yes);
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Default Access in Template Classes ===" << std::endl;
    
    std::cout << "\n1. Template class (private by default):" << std::endl;
    TemplateClass<int> tc;
    // tc.data = 10;  // ERROR: private
    tc.publicFunction();  // OK: public
    
    std::cout << "\n2. Template struct (public by default):" << std::endl;
    TemplateStruct<int> ts;
    ts.data = 10;  // OK: public
    std::cout << "ts.data = " << ts.data << std::endl;
    
    std::cout << "\n3. Container with private members:" << std::endl;
    Container<int> cont(5);
    cont[0] = 42;  // OK: operator[] is public
    // cont.privateResize();  // ERROR: private
    
    std::cout << "\n4. Template inheritance access:" << std::endl;
    Derived<int> derived(100);
    derived.access();
    // std::cout << derived.protectedData;  // ERROR: protected
    
    std::cout << "\n5. Friend access in templates:" << std::endl;
    FriendDemo<int> fd;
    FriendClass fc;
    fc.modify(fd, 42);
    friendFunction(fd, 100);
    
    std::cout << "\n=== Summary ===" << std::endl;
    std::cout << "- Template classes: private by default" << std::endl;
    std::cout << "- Template structs: public by default" << std::endl;
    std::cout << "- Same access rules as non-template classes" << std::endl;
    std::cout << "- Access specifiers apply to all instantiations" << std::endl;
    
    return 0;
}
```

**Key Points:**

| Element | Default Access | Notes |
| :--- | :--- | :--- |
| **template class** | private | Same as regular class |
| **template struct** | public | Same as regular struct |
| **Template parameters** | N/A | Type parameters have no access |
| **Specializations** | Follows declared access | Can change in specialization |
| **Inheritance** | Base class access | Protected/private inheritance possible |

---

### 98. Explain pointer to member function.

A **pointer to member function** is a special pointer that stores the address of a class member function. It must be used with an object to call the function.

```cpp
#include <iostream>
#include <vector>
#include <functional>
#include <map>

// ==================== Basic Member Function Pointers ====================

class Calculator {
private:
    double memory;
    
public:
    Calculator() : memory(0) {}
    
    // Member functions
    double add(double a, double b) {
        memory = a + b;
        return memory;
    }
    
    double subtract(double a, double b) {
        memory = a - b;
        return memory;
    }
    
    double multiply(double a, double b) {
        memory = a * b;
        return memory;
    }
    
    double divide(double a, double b) {
        if (b != 0) {
            memory = a / b;
        }
        return memory;
    }
    
    double getMemory() const {
        return memory;
    }
    
    // Const member function
    void display() const {
        std::cout << "Memory: " << memory << std::endl;
    }
    
    // Static member function
    static void showInfo() {
        std::cout << "Calculator class" << std::endl;
    }
};

// ==================== Declaring and Using Member Function Pointers ====================

void basicMemberPointerDemo() {
    std::cout << "=== Basic Member Function Pointers ===" << std::endl;
    
    // Declare pointer to member function
    double (Calculator::*op)(double, double);
    
    Calculator calc;
    
    // Point to add function
    op = &Calculator::add;
    double result = (calc.*op)(10, 5);
    std::cout << "10 + 5 = " << result << std::endl;
    
    // Point to subtract function
    op = &Calculator::subtract;
    result = (calc.*op)(10, 5);
    std::cout << "10 - 5 = " << result << std::endl;
    
    // Using with pointer to object
    Calculator* calcPtr = &calc;
    op = &Calculator::multiply;
    result = (calcPtr->*op)(10, 5);
    std::cout << "10 * 5 = " << result << std::endl;
}

// ==================== Array of Member Function Pointers ====================

void arrayOfPointersDemo() {
    std::cout << "\n=== Array of Member Function Pointers ===" << std::endl;
    
    // Array of operation pointers
    double (Calculator::*operations[])(double, double) = {
        &Calculator::add,
        &Calculator::subtract,
        &Calculator::multiply,
        &Calculator::divide
    };
    
    const char* opNames[] = {"Add", "Subtract", "Multiply", "Divide"};
    
    Calculator calc;
    
    for (int i = 0; i < 4; ++i) {
        double result = (calc.*operations[i])(20, 4);
        std::cout << opNames[i] << ": " << result << std::endl;
    }
}

// ==================== Pointer to Const Member Function ====================

void constMemberPointerDemo() {
    std::cout << "\n=== Pointer to Const Member Function ===" << std::endl;
    
    // Pointer to const member function
    void (Calculator::*displayPtr)() const = &Calculator::display;
    
    Calculator calc;
    calc.add(10, 5);
    
    (calc.*displayPtr)();
}

// ==================== Pointer to Static Member Function ====================

void staticMemberPointerDemo() {
    std::cout << "\n=== Pointer to Static Member Function ===" << std::endl;
    
    // Pointer to static member function (regular function pointer)
    void (*staticPtr)() = &Calculator::showInfo;
    
    staticPtr();  // Called without object
}

// ==================== Using with std::function and bind ====================

void modernApproachDemo() {
    std::cout << "\n=== Modern C++ Approach ===" << std::endl;
    
    Calculator calc;
    
    // Using std::function with std::bind
    std::function<double(double, double)> addFunc = 
        std::bind(&Calculator::add, &calc, std::placeholders::_1, std::placeholders::_2);
    
    std::function<double(double, double)> multFunc = 
        [&calc](double a, double b) { return calc.multiply(a, b); };
    
    std::cout << "Add via bind: " << addFunc(15, 5) << std::endl;
    std::cout << "Multiply via lambda: " << multFunc(15, 5) << std::endl;
    
    // Using std::mem_fn (C++11)
    auto memAdd = std::mem_fn(&Calculator::add);
    std::cout << "Add via mem_fn: " << memAdd(calc, 20, 4) << std::endl;
}

// ==================== Practical Example: Command Pattern ====================

class Command {
private:
    Calculator& receiver;
    double (Calculator::*action)(double, double);
    double a, b;
    
public:
    Command(Calculator& calc, double (Calculator::*act)(double, double), 
            double x, double y) 
        : receiver(calc), action(act), a(x), b(y) {}
    
    double execute() {
        return (receiver.*action)(a, b);
    }
};

void commandPatternDemo() {
    std::cout << "\n=== Command Pattern with Member Pointers ===" << std::endl;
    
    Calculator calc;
    
    std::vector<Command> commands;
    commands.emplace_back(calc, &Calculator::add, 10, 5);
    commands.emplace_back(calc, &Calculator::subtract, 10, 5);
    commands.emplace_back(calc, &Calculator::multiply, 10, 5);
    
    for (auto& cmd : commands) {
        std::cout << "Result: " << cmd.execute() << std::endl;
    }
}

// ==================== Practical Example: Dispatch Table ====================

class MenuSystem {
private:
    struct MenuItem {
        std::string name;
        void (Calculator::*action)(double, double);
        double param1;
        double param2;
    };
    
    std::vector<MenuItem> items;
    Calculator& calc;
    
public:
    MenuSystem(Calculator& c) : calc(c) {}
    
    void addItem(const std::string& name, 
                 void (Calculator::*act)(double, double),
                 double p1, double p2) {
        items.push_back({name, act, p1, p2});
    }
    
    void run(int index) {
        if (index < items.size()) {
            auto& item = items[index];
            std::cout << "Executing " << item.name << ": ";
            (calc.*item.action)(item.param1, item.param2);
            calc.display();
        }
    }
};

// ==================== Syntax Summary ====================

void syntaxSummary() {
    std::cout << "\n=== Syntax Summary ===" << std::endl;
    
    std::cout << "1. Declaration:  return_type (ClassName::*ptr_name)(params)" << std::endl;
    std::cout << "2. Assignment:   ptr = &ClassName::function" << std::endl;
    std::cout << "3. Call (object): (object.*ptr)(args)" << std::endl;
    std::cout << "4. Call (pointer): (ptr->*ptr)(args)" << std::endl;
    std::cout << "5. Const member: return_type (ClassName::*ptr)(params) const" << std::endl;
    std::cout << "6. Static member: return_type (*ptr)(params)" << std::endl;
}

// ==================== Main Demonstration ====================

int main() {
    basicMemberPointerDemo();
    arrayOfPointersDemo();
    constMemberPointerDemo();
    staticMemberPointerDemo();
    modernApproachDemo();
    commandPatternDemo();
    
    Calculator calc;
    MenuSystem menu(calc);
    menu.addItem("Add", &Calculator::add, 10, 5);
    menu.addItem("Multiply", &Calculator::multiply, 10, 5);
    menu.run(0);
    menu.run(1);
    
    syntaxSummary();
    
    return 0;
}
```

**Member Function Pointer Syntax:**

| Operation | Syntax |
| :--- | :--- |
| **Declaration** | `return_type (Class::*ptr)(params)` |
| **Assignment** | `ptr = &Class::function` |
| **Call with object** | `(object.*ptr)(args)` |
| **Call with pointer** | `(ptr->*ptr)(args)` |
| **Const member** | `return_type (Class::*ptr)(params) const` |
| **Static member** | `return_type (*ptr)(params)` |

---

### 99. How are pure virtual methods implemented under the hood?

**Pure virtual methods** are implemented using the **vtable mechanism** but with entries that point to a special function that triggers **pure virtual call error**.

```cpp
#include <iostream>
#include <cstdio>  // For printf (to avoid iostream in low-level demonstration)

// ==================== Conceptual Implementation ====================

/*
 * Under the hood:
 * 
 * 1. Class with pure virtual functions has a vtable
 * 2. Pure virtual entries point to a special function: __pure_virtual_called()
 * 3. This function typically calls std::terminate()
 * 4. Derived classes override these entries with their implementations
 */

// ==================== Low-Level Simulation ====================

// Simulating what the compiler might generate
extern "C" void __pure_virtual_called() {
    printf("Pure virtual function called! Terminating...\n");
    std::terminate();
}

// Vtable entry type
typedef void (*VirtualFunc)();

// ==================== Abstract Base Class ====================

class AbstractBase {
private:
    // Hidden vptr (simulated)
    static const VirtualFunc vtable[2];  // 2 virtual functions
    
public:
    // Constructor sets vptr
    AbstractBase() {
        // Set vptr to this class's vtable
        *reinterpret_cast<const VirtualFunc**>(this) = const_cast<VirtualFunc**>(&vtable);
        printf("AbstractBase constructed, vtable set\n");
    }
    
    virtual ~AbstractBase() {
        printf("AbstractBase destructed\n");
    }
    
    // Pure virtual functions
    virtual void pureVirtual1() const = 0;
    virtual void pureVirtual2() const = 0;
    
    // Non-pure virtual
    virtual void implemented() const {
        printf("AbstractBase::implemented()\n");
    }
    
    // For demonstration - access vtable
    void showVtable() const {
        const VirtualFunc* vt = *reinterpret_cast<const VirtualFunc* const*>(this);
        printf("vtable address: %p\n", (void*)vt);
    }
};

// Initialize vtable with pure virtual call entries
const VirtualFunc AbstractBase::vtable[2] = {
    reinterpret_cast<VirtualFunc>(__pure_virtual_called),  // pureVirtual1
    reinterpret_cast<VirtualFunc>(__pure_virtual_called)   // pureVirtual2
};

// ==================== Derived Class Implementation ====================

class ConcreteDerived : public AbstractBase {
private:
    static const VirtualFunc vtable[2];  // Overridden vtable
    
public:
    ConcreteDerived() : AbstractBase() {
        // Override vptr to point to derived vtable
        *reinterpret_cast<const VirtualFunc**>(this) = const_cast<VirtualFunc**>(&vtable);
        printf("ConcreteDerived constructed, vtable updated\n");
    }
    
    void pureVirtual1() const override {
        printf("ConcreteDerived::pureVirtual1()\n");
    }
    
    void pureVirtual2() const override {
        printf("ConcreteDerived::pureVirtual2()\n");
    }
    
    ~ConcreteDerived() override {
        printf("ConcreteDerived destructed\n");
    }
};

// Derived vtable with actual implementations
namespace {
    void derivedPureVirtual1Thunk(const ConcreteDerived* obj) {
        obj->pureVirtual1();
    }
    
    void derivedPureVirtual2Thunk(const ConcreteDerived* obj) {
        obj->pureVirtual2();
    }
}

const VirtualFunc ConcreteDerived::vtable[2] = {
    reinterpret_cast<VirtualFunc>(derivedPureVirtual1Thunk),
    reinterpret_cast<VirtualFunc>(derivedPureVirtual2Thunk)
};

// ==================== What Happens in Constructor/Destructor ====================

class TracingAbstract {
public:
    TracingAbstract() {
        printf("TracingAbstract constructor\n");
        callPureVirtual();  // DANGER: pure virtual call in constructor!
    }
    
    virtual ~TracingAbstract() {
        printf("TracingAbstract destructor\n");
        callPureVirtual();  // DANGER: pure virtual call in destructor!
    }
    
    virtual void pureVirtual() = 0;
    
    void callPureVirtual() {
        // During constructor/destructor, vptr points to base class vtable
        // This would call __pure_virtual_called and terminate
        pureVirtual();  // UNDEFINED BEHAVIOR in constructor/destructor
    }
};

// ==================== Assembly-Level View (Conceptual) ====================

/*
 * Typical compiler-generated assembly for virtual call:
 * 
 * ; Call pureVirtual1() through vtable
 * mov rax, [this]           ; load vptr
 * mov rax, [rax]            ; load vtable
 * call [rax]                 ; call function through vtable[0]
 * 
 * For pure virtual: vtable[0] points to __pure_virtual_called
 * For implemented: vtable[0] points to actual function
 */

// ==================== Multiple Inheritance Case ====================

struct Interface1 {
    virtual void if1Func() = 0;
};

struct Interface2 {
    virtual void if2Func() = 0;
};

class MultiImpl : public Interface1, public Interface2 {
private:
    // Two vptrs (one for each base)
    
public:
    void if1Func() override {
        printf("MultiImpl::if1Func()\n");
    }
    
    void if2Func() override {
        printf("MultiImpl::if2Func()\n");
    }
};

// ==================== Size Information ====================

void sizeInfo() {
    printf("\n=== Size Information ===\n");
    printf("Sizeof(AbstractBase): %zu bytes (includes vptr)\n", sizeof(AbstractBase));
    printf("Sizeof(ConcreteDerived): %zu bytes (inherits vptr)\n", sizeof(ConcreteDerived));
    printf("Sizeof(void*): %zu bytes\n", sizeof(void*));
}

// ==================== Main Demonstration ====================

int main() {
    printf("=== Pure Virtual Function Implementation ===\n");
    
    sizeInfo();
    
    printf("\n=== Creating Concrete Object ===\n");
    ConcreteDerived derived;
    derived.showVtable();
    
    printf("\n=== Calling Virtual Functions ===\n");
    AbstractBase* base = &derived;
    base->pureVirtual1();
    base->pureVirtual2();
    base->implemented();
    
    printf("\n=== What NOT to do: Pure Virtual Call in Constructor ===\n");
    printf("(This would terminate - commented out for safety)\n");
    // TracingAbstract bad;  // Would call pure virtual in constructor
    
    printf("\n=== Multiple Inheritance Case ===\n");
    MultiImpl multi;
    Interface1* i1 = &multi;
    Interface2* i2 = &multi;
    i1->if1Func();
    i2->if2Func();
    printf("Sizeof(MultiImpl): %zu bytes (two vptrs)\n", sizeof(MultiImpl));
    
    printf("\n=== Implementation Summary ===\n");
    printf("1. Pure virtual functions get vtable entries\n");
    printf("2. Entries point to pure virtual call handler\n");
    printf("3. Derived classes override entries\n");
    printf("4. Constructor/destructor calls to pure virtual = termination\n");
    printf("5. Multiple inheritance = multiple vptrs\n");
    
    return 0;
}
```

**Pure Virtual Implementation Details:**

| Aspect | Implementation |
| :--- | :--- |
| **vtable entry** | Points to pure virtual call handler |
| **Handler function** | Usually calls `std::terminate()` |
| **Constructor** | vptr points to base vtable (pure entries) |
| **Destructor** | vptr reverts to base vtable |
| **Override** | Derived replaces entry with actual function |
| **Multiple inheritance** | Multiple vptrs, each with its own vtable |

---

### 100. How do you debug OOP design issues in large systems?

Debugging OOP design issues requires a **systematic approach** combining static analysis, runtime observation, and design review techniques.

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <map>
#include <set>
#include <chrono>
#include <thread>
#include <sstream>
#include <fstream>

// ==================== 1. Static Analysis Tools ====================

// Example of design issues that can be detected statically

class BadDesign {
    // God class - too many responsibilities
    void handleDatabase() {}
    void handleNetwork() {}
    void handleUI() {}
    void handleBusinessLogic() {}
    void handleFileIO() {}
    void handleLogging() {}
    // 50 more methods...
};

// Tool: Use static analyzers (Clang-Tidy, Cppcheck)
// Command: clang-tidy -checks='*' file.cpp -- -std=c++17

// ==================== 2. Runtime Instrumentation ====================

// Simple instrumentation for tracking object creation/destruction
class TracedObject {
private:
    static std::map<const TracedObject*, std::string> liveObjects;
    static std::mutex mtx;
    std::string objectId;
    
public:
    TracedObject(const std::string& id) : objectId(id) {
        std::lock_guard<std::mutex> lock(mtx);
        liveObjects[this] = id;
        std::cout << "Created: " << id << " (Total: " << liveObjects.size() << ")" << std::endl;
    }
    
    virtual ~TracedObject() {
        std::lock_guard<std::mutex> lock(mtx);
        liveObjects.erase(this);
        std::cout << "Destroyed: " << objectId << " (Remaining: " << liveObjects.size() << ")" << std::endl;
    }
    
    static void dumpLiveObjects() {
        std::lock_guard<std::mutex> lock(mtx);
        std::cout << "=== Live Objects (" << liveObjects.size() << ") ===" << std::endl;
        for (const auto& [ptr, id] : liveObjects) {
            std::cout << "  " << id << " at " << ptr << std::endl;
        }
    }
    
    virtual void operation() const {
        std::cout << "TracedObject::operation()" << std::endl;
    }
};

std::map<const TracedObject*, std::string> TracedObject::liveObjects;
std::mutex TracedObject::mtx;

// ==================== 3. Design Pattern Detection ====================

class DesignPatternDetector {
private:
    std::map<std::string, int> patternScores;
    
public:
    // Detect potential patterns in code structure
    void analyzeClass(const std::string& className, 
                      int virtualCount, 
                      int pureVirtualCount,
                      int friendCount,
                      int templateParams) {
        std::cout << "\nAnalyzing: " << className << std::endl;
        
        // Detect Abstract Base Class
        if (pureVirtualCount > 0 && virtualCount > pureVirtualCount) {
            patternScores["Abstract Base Class"]++;
            std::cout << "  - Possible Abstract Base Class" << std::endl;
        }
        
        // Detect Interface
        if (pureVirtualCount > 0 && virtualCount == pureVirtualCount) {
            patternScores["Interface"]++;
            std::cout << "  - Possible Interface" << std::endl;
        }
        
        // Detect God Class (anti-pattern)
        if (virtualCount > 20) {
            patternScores["God Class (anti-pattern)"]++;
            std::cout << "  - WARNING: Possible God Class" << std::endl;
        }
        
        // Detect Template Pattern
        if (virtualCount > 0 && templateParams > 0) {
            patternScores["Template Method"]++;
            std::cout << "  - Possible Template Method pattern" << std::endl;
        }
    }
    
    void printReport() {
        std::cout << "\n=== Design Pattern Detection Report ===" << std::endl;
        for (const auto& [pattern, count] : patternScores) {
            std::cout << pattern << ": " << count << " occurrences" << std::endl;
        }
    }
};

// ==================== 4. Coupling Analysis ====================

class CouplingAnalyzer {
private:
    std::map<std::string, std::set<std::string>> dependencies;
    
public:
    void addDependency(const std::string& from, const std::string& to) {
        dependencies[from].insert(to);
    }
    
    void analyzeCoupling() {
        std::cout << "\n=== Coupling Analysis ===" << std::endl;
        
        for (const auto& [className, deps] : dependencies) {
            std::cout << className << " depends on:" << std::endl;
            for (const auto& dep : deps) {
                std::cout << "  - " << dep << std::endl;
            }
            
            // Detect high coupling
            if (deps.size() > 5) {
                std::cout << "  WARNING: High coupling (" << deps.size() << " dependencies)" << std::endl;
            }
        }
        
        // Detect cyclic dependencies
        detectCycles();
    }
    
private:
    void detectCycles() {
        std::cout << "\nChecking for cycles..." << std::endl;
        // Simplified cycle detection
        for (const auto& [classA, depsA] : dependencies) {
            for (const auto& classB : depsA) {
                if (dependencies[classB].count(classA)) {
                    std::cout << "  CYCLE DETECTED: " << classA << " <-> " << classB << std::endl;
                }
            }
        }
    }
};

// ==================== 5. Performance Profiling ====================

class PerformanceProfiler {
private:
    using Clock = std::chrono::high_resolution_clock;
    std::map<std::string, Clock::time_point> startTimes;
    std::map<std::string, long long> totalTimes;
    std::map<std::string, int> callCounts;
    
public:
    void start(const std::string& operation) {
        startTimes[operation] = Clock::now();
    }
    
    void stop(const std::string& operation) {
        auto end = Clock::now();
        auto start = startTimes[operation];
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
        
        totalTimes[operation] += duration;
        callCounts[operation]++;
    }
    
    void printReport() {
        std::cout << "\n=== Performance Profile ===" << std::endl;
        for (const auto& [op, total] : totalTimes) {
            int count = callCounts[op];
            double avg = static_cast<double>(total) / count;
            std::cout << op << ": " << count << " calls, "
                      << "total " << total << " µs, "
                      << "avg " << avg << " µs" << std::endl;
            
            if (avg > 1000) {  // 1ms threshold
                std::cout << "  WARNING: Slow operation!" << std::endl;
            }
        }
    }
};

// ==================== 6. Memory Leak Detection ====================

class MemoryTracker {
private:
    struct AllocationInfo {
        size_t size;
        std::string file;
        int line;
    };
    
    static std::map<void*, AllocationInfo> allocations;
    static size_t totalAllocated;
    static size_t totalFreed;
    
public:
    static void* trackAllocate(size_t size, const char* file, int line) {
        void* ptr = malloc(size);
        if (ptr) {
            allocations[ptr] = {size, file, line};
            totalAllocated += size;
        }
        return ptr;
    }
    
    static void trackDeallocate(void* ptr) {
        auto it = allocations.find(ptr);
        if (it != allocations.end()) {
            totalFreed += it->second.size;
            allocations.erase(it);
        }
        free(ptr);
    }
    
    static void reportLeaks() {
        std::cout << "\n=== Memory Leak Report ===" << std::endl;
        std::cout << "Total allocated: " << totalAllocated << " bytes" << std::endl;
        std::cout << "Total freed: " << totalFreed << " bytes" << std::endl;
        std::cout << "Current leaks: " << (totalAllocated - totalFreed) << " bytes" << std::endl;
        
        if (!allocations.empty()) {
            std::cout << "Leaked allocations:" << std::endl;
            for (const auto& [ptr, info] : allocations) {
                std::cout << "  " << info.size << " bytes at " << ptr 
                          << " (allocated in " << info.file << ":" << info.line << ")" << std::endl;
            }
        }
    }
};

std::map<void*, MemoryTracker::AllocationInfo> MemoryTracker::allocations;
size_t MemoryTracker::totalAllocated = 0;
size_t MemoryTracker::totalFreed = 0;

// Override new/delete operators for tracking
void* operator new(size_t size, const char* file, int line) {
    return MemoryTracker::trackAllocate(size, file, line);
}

void operator delete(void* ptr) noexcept {
    MemoryTracker::trackDeallocate(ptr);
}

#define new new(__FILE__, __LINE__)

// ==================== 7. Visualization ====================

void generateUML(const std::string& className, 
                 const std::vector<std::string>& parents,
                 const std::vector<std::string>& children) {
    std::cout << "\n=== UML Representation ===" << std::endl;
    std::cout << "class " << className << " {" << std::endl;
    if (!parents.empty()) {
        std::cout << "  extends: ";
        for (const auto& p : parents) std::cout << p << " ";
        std::cout << std::endl;
    }
    std::cout << "  children: ";
    for (const auto& c : children) std::cout << c << " ";
    std::cout << "\n}" << std::endl;
    
    // In real tool, generate actual UML diagram
    std::cout << "Run: doxygen -g && doxygen" << std::endl;
}

// ==================== 8. Testing Strategy ====================

class DesignTester {
public:
    static void testInheritanceDepth(const std::string& className, int depth) {
        std::cout << "\nTesting " << className << " (depth: " << depth << ")" << std::endl;
        
        if (depth > 5) {
            std::cout << "  WARNING: Deep inheritance hierarchy (" << depth << " levels)" << std::endl;
            std::cout << "  Suggestion: Consider composition over inheritance" << std::endl;
        }
    }
    
    static void testClassCohesion(const std::string& className, 
                                   int methods, int fields) {
        std::cout << "\nCohesion test for " << className << std::endl;
        double cohesion = static_cast<double>(methods) / fields;
        
        if (cohesion < 0.5) {
            std::cout << "  LOW COHESION: " << methods << " methods, " 
                      << fields << " fields" << std::endl;
            std::cout << "  Suggestion: Consider splitting the class" << std::endl;
        }
    }
};

// ==================== Example Usage ====================

class TestClass : public TracedObject {
private:
    std::vector<int> data;
    
public:
    TestClass(const std::string& id) : TracedObject(id) {
        data.resize(1000);
    }
    
    void operation() const override {
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
        std::cout << "TestClass::operation()" << std::endl;
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Debugging OOP Design Issues ===" << std::endl;
    
    // 1. Static Analysis
    std::cout << "\n1. STATIC ANALYSIS" << std::endl;
    DesignPatternDetector detector;
    detector.analyzeClass("BadDesign", 50, 0, 10, 0);
    detector.analyzeClass("AbstractBase", 3, 2, 0, 0);
    detector.analyzeClass("Interface", 2, 2, 0, 0);
    detector.printReport();
    
    // 2. Coupling Analysis
    std::cout << "\n2. COUPLING ANALYSIS" << std::endl;
    CouplingAnalyzer analyzer;
    analyzer.addDependency("Controller", "Service");
    analyzer.addDependency("Controller", "Repository");
    analyzer.addDependency("Controller", "View");
    analyzer.addDependency("Service", "Repository");
    analyzer.addDependency("Repository", "Database");
    analyzer.addDependency("Service", "Controller");  // Creates cycle
    analyzer.analyzeCoupling();
    
    // 3. Performance Profiling
    std::cout << "\n3. PERFORMANCE PROFILING" << std::endl;
    PerformanceProfiler profiler;
    
    profiler.start("TestClass::operation");
    TestClass testObj("test");
    testObj.operation();
    profiler.stop("TestClass::operation");
    
    profiler.printReport();
    
    // 4. Memory Tracking
    std::cout << "\n4. MEMORY TRACKING" << std::endl;
    {
        TestClass* leak1 = new TestClass("leak1");
        TestClass* leak2 = new TestClass("leak2");
        TestClass* ok = new TestClass("ok");
        delete ok;
        // leak1 and leak2 are leaked
    }
    MemoryTracker::reportLeaks();
    
    // 5. Visualization
    std::cout << "\n5. VISUALIZATION" << std::endl;
    generateUML("Animal", {}, {"Dog", "Cat"});
    generateUML("Dog", {"Animal"}, {});
    
    // 6. Design Testing
    std::cout << "\n6. DESIGN TESTING" << std::endl;
    DesignTester::testInheritanceDepth("DeepClass", 7);
    DesignTester::testClassCohesion("GodClass", 5, 20);
    
    // 7. Runtime Object Tracking
    std::cout << "\n7. RUNTIME OBJECT TRACKING" << std::endl;
    {
        TestClass obj1("runtime1");
        TestClass obj2("runtime2");
        TracedObject::dumpLiveObjects();
    }
    TracedObject::dumpLiveObjects();
    
    std::cout << "\n=== Debugging Checklist ===" << std::endl;
    std::cout << "□ Static analysis tools (Clang-Tidy, Cppcheck)" << std::endl;
    std::cout << "□ Runtime instrumentation" << std::endl;
    std::cout << "□ Design pattern detection" << std::endl;
    std::cout << "□ Coupling analysis" << std::endl;
    std::cout << "□ Performance profiling" << std::endl;
    std::cout << "□ Memory leak detection" << std::endl;
    std::cout << "□ UML visualization" << std::endl;
    std::cout << "□ Unit testing coverage" << std::endl;
    std::cout << "□ Code review checklist" << std::endl;
    
    return 0;
}
```

**Debugging Tools and Techniques:**

| Category | Tools/Methods | What to Look For |
| :--- | :--- | :--- |
| **Static Analysis** | Clang-Tidy, Cppcheck | Design anti-patterns, complexity |
| **Runtime** | Instrumentation, logging | Object lifetimes, call patterns |
| **Coupling** | Dependency graphs | High coupling, cycles |
| **Performance** | Profilers | Bottlenecks, slow operations |
| **Memory** | Valgrind, sanitizers | Leaks, double frees |
| **Visualization** | Doxygen, UML tools | Hierarchy issues |
| **Testing** | Unit tests, coverage | Missing tests, edge cases |
| **Review** | Code walkthroughs | Logic errors, design flaws |

