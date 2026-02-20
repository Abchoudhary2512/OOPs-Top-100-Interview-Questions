### 31. What is compile-time polymorphism?

**Compile-time polymorphism** (also called **static polymorphism** or **early binding**) is a type of polymorphism where the function to be called is determined at **compile time**. The compiler knows exactly which function will be executed based on the function signatures and the types of arguments.

**How it's achieved in C++:**
1. **Function overloading** (multiple functions with same name but different parameters)
2. **Operator overloading** (redefining operators for user-defined types)
3. **Templates** (function and class templates)

**Key characteristics:**
- Faster execution (no runtime decision overhead)
- Function calls are resolved by the compiler
- Also called **static binding** or **early binding**

```cpp
#include <iostream>

// Compile-time polymorphism through function overloading
class Calculator {
public:
    // Same function name, different parameters
    int add(int a, int b) {
        return a + b;
    }
    
    double add(double a, double b) {
        return a + b;
    }
    
    int add(int a, int b, int c) {
        return a + b + c;
    }
};

int main() {
    Calculator calc;
    
    std::cout << calc.add(5, 10) << std::endl;       // Calls int version
    std::cout << calc.add(3.5, 2.7) << std::endl;   // Calls double version
    std::cout << calc.add(1, 2, 3) << std::endl;     // Calls three-param version
    
    // All resolved at compile time!
    return 0;
}
```

---

### 32. What is runtime polymorphism?

**Runtime polymorphism** (also called **dynamic polymorphism** or **late binding**) is a type of polymorphism where the function to be called is determined at **runtime** based on the actual type of the object. It allows a base class pointer or reference to call derived class functions.

**How it's achieved in C++:**
- **Virtual functions** and **function overriding**
- Requires inheritance hierarchy

**Key characteristics:**
- Slightly slower due to runtime decision (vtable lookup)
- Provides flexibility and extensibility
- Also called **dynamic binding** or **late binding**

```cpp
#include <iostream>

class Animal {
public:
    // Virtual function enables runtime polymorphism
    virtual void makeSound() {
        std::cout << "Animal makes a sound" << std::endl;
    }
};

class Dog : public Animal {
public:
    // Override the virtual function
    void makeSound() override {
        std::cout << "Dog barks: Woof! Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {
        std::cout << "Cat meows: Meow!" << std::endl;
    }
};

int main() {
    Animal* ptr;  // Base class pointer
    
    Dog dog;
    Cat cat;
    
    ptr = &dog;
    ptr->makeSound();  // Output: Dog barks: Woof! Woof! (decided at runtime)
    
    ptr = &cat;
    ptr->makeSound();  // Output: Cat meows: Meow! (decided at runtime)
    
    return 0;
}
```

---

### 33. What is function overloading?

**Function overloading** is a feature that allows multiple functions with the **same name** but **different parameters** (different number, types, or order of parameters) within the same scope. The compiler determines which function to call based on the arguments provided.

**Rules for function overloading:**
1. Functions must differ in their parameter list (number, type, or order)
2. Return type alone **cannot** be used to distinguish overloaded functions
3. Default arguments can create ambiguity if not careful

```cpp
#include <iostream>

class Display {
public:
    // Same function name, different parameters
    
    void show(int i) {
        std::cout << "Integer: " << i << std::endl;
    }
    
    void show(double d) {
        std::cout << "Double: " << d << std::endl;
    }
    
    void show(const char* str) {
        std::cout << "String: " << str << std::endl;
    }
    
    void show(int i, double d) {
        std::cout << "Int and Double: " << i << ", " << d << std::endl;
    }
    
    // This would NOT work (only return type differs)
    // int show(int i) { return i; } // ERROR: already defined
};

int main() {
    Display disp;
    
    disp.show(10);           // Calls show(int)
    disp.show(3.14);         // Calls show(double)
    disp.show("Hello");      // Calls show(const char*)
    disp.show(5, 2.5);       // Calls show(int, double)
    
    return 0;
}
```

---

### 34. What is function overriding?

**Function overriding** is a feature that allows a derived class to provide a **different implementation** of a function that is already defined in its base class. The function in the derived class must have the **same signature** (name, return type, and parameters) as the base class function.

**Key points:**
- Requires inheritance
- Used to achieve runtime polymorphism (with virtual functions)
- The base class function is said to be "overridden"
- In C++11+, use the `override` keyword for clarity and compiler checks

```cpp
#include <iostream>

class Base {
public:
    virtual void display() {  // Must be virtual to be overridden
        std::cout << "Display from Base" << std::endl;
    }
    
    void nonVirtual() {
        std::cout << "Non-virtual from Base" << std::endl;
    }
};

class Derived : public Base {
public:
    // Overriding the virtual function
    void display() override {  // 'override' keyword is optional but recommended
        std::cout << "Display from Derived" << std::endl;
    }
    
    // This is NOT overriding (just hiding) - no 'virtual' keyword
    void nonVirtual() {
        std::cout << "Non-virtual from Derived" << std::endl;
    }
};

int main() {
    Base* basePtr = new Derived();
    
    basePtr->display();     // Output: Display from Derived (overridden)
    basePtr->nonVirtual();  // Output: Non-virtual from Base (not overridden)
    
    delete basePtr;
    return 0;
}
```

**Key difference from overloading:**
| Feature | Function Overloading | Function Overriding |
| :--- | :--- | :--- |
| Scope | Same class | Base and derived classes |
| Parameters | Must differ | Must be identical |
| Return type | Can differ | Must be identical |
| Virtual keyword | Not needed | Required for polymorphism |

---

### 35. What is a virtual function?

A **virtual function** is a member function declared with the `virtual` keyword in the base class that can be **overridden in derived classes**. It enables **runtime polymorphism** by allowing function calls to be resolved based on the actual type of the object, not the type of the pointer/reference.

**How it works:**
- Each class with virtual functions has a **vtable (virtual table)** – an array of function pointers
- Each object has a hidden **vptr (virtual pointer)** pointing to its class's vtable
- When a virtual function is called, the program follows the vptr to the vtable and calls the appropriate function

```cpp
#include <iostream>

class Shape {
public:
    // Virtual function
    virtual double area() {
        std::cout << "Shape area" << std::endl;
        return 0;
    }
    
    // Non-virtual function
    void display() {
        std::cout << "This is a shape" << std::endl;
    }
};

class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    
    // Override virtual function
    double area() override {
        std::cout << "Circle area: ";
        return 3.14 * radius * radius;
    }
};

int main() {
    Shape* shapePtr;
    Circle circle(5);
    
    shapePtr = &circle;
    shapePtr->display();  // Calls Shape::display() (static binding)
    std::cout << shapePtr->area() << std::endl;  // Calls Circle::area() (dynamic binding)
    
    return 0;
}
```

---

### 36. What is a pure virtual function?

A **pure virtual function** is a virtual function that has **no implementation** in the base class and is declared by assigning `= 0`. It forces derived classes to provide their own implementation (unless they want to remain abstract).

**Syntax:**
```cpp
virtual returnType functionName(parameters) = 0;
```

**Purpose:**
- To define an interface that derived classes must implement
- To create abstract base classes

```cpp
#include <iostream>

class Shape {
public:
    // Pure virtual function - no implementation in base class
    virtual double area() = 0;
    
    // Regular member function (can still have implementation)
    void display() {
        std::cout << "This is a shape" << std::endl;
    }
    
    // Virtual destructor (important for abstract classes)
    virtual ~Shape() {}
};

class Rectangle : public Shape {
private:
    double length, width;
public:
    Rectangle(double l, double w) : length(l), width(w) {}
    
    // Must implement area() or remain abstract
    double area() override {
        return length * width;
    }
};

int main() {
    // Shape s;  // ERROR: Cannot instantiate abstract class
    
    Rectangle rect(5, 3);
    rect.display();
    std::cout << "Area: " << rect.area() << std::endl;
    
    Shape* shapePtr = &rect;
    std::cout << "Area via pointer: " << shapePtr->area() << std::endl;
    
    return 0;
}
```

---

### 37. What is an abstract class?

An **abstract class** is a class that **cannot be instantiated** (you cannot create objects of that class). It's designed to serve as a **base class** (interface) for other classes. A class becomes abstract if it contains **at least one pure virtual function**.

**Characteristics:**
- Cannot create objects of an abstract class
- Can have pointers and references to abstract classes (for polymorphism)
- Can have regular member functions and data members
- Derived classes must implement all pure virtual functions to become concrete

```cpp
#include <iostream>

// Abstract class
class Animal {
protected:
    std::string name;
    
public:
    Animal(const std::string& n) : name(n) {}
    
    // Pure virtual functions (make class abstract)
    virtual void makeSound() = 0;
    virtual void move() = 0;
    
    // Concrete method (can be used by all derived classes)
    void sleep() {
        std::cout << name << " is sleeping" << std::endl;
    }
    
    virtual ~Animal() {}
};

// Concrete class
class Dog : public Animal {
public:
    Dog(const std::string& n) : Animal(n) {}
    
    // Must implement all pure virtual functions
    void makeSound() override {
        std::cout << name << " says: Woof! Woof!" << std::endl;
    }
    
    void move() override {
        std::cout << name << " runs on four legs" << std::endl;
    }
};

// Another concrete class
class Bird : public Animal {
public:
    Bird(const std::string& n) : Animal(n) {}
    
    void makeSound() override {
        std::cout << name << " says: Chirp! Chirp!" << std::endl;
    }
    
    void move() override {
        std::cout << name << " flies in the sky" << std::endl;
    }
};

int main() {
    // Animal a;  // ERROR: Cannot instantiate abstract class
    
    Dog dog("Buddy");
    Bird bird("Tweety");
    
    dog.makeSound();
    dog.move();
    dog.sleep();
    
    bird.makeSound();
    bird.move();
    
    // Polymorphic behavior with abstract class pointer
    Animal* animals[] = {&dog, &bird};
    for (auto animal : animals) {
        animal->makeSound();
    }
    
    return 0;
}
```

---

### 38. Why do you need a virtual destructor?

A **virtual destructor** is needed when you intend to **delete a derived class object through a base class pointer**. Without a virtual destructor, only the base class destructor would be called, leading to **resource leaks** (derived class resources not freed).

**The problem (without virtual destructor):**
```cpp
class Base {
public:
    ~Base() {  // Non-virtual destructor
        std::cout << "Base destructor" << std::endl;
    }
};

class Derived : public Base {
private:
    int* data;
public:
    Derived() { data = new int[100]; }
    ~Derived() {  // This won't be called if deleted via Base pointer
        delete[] data;
        std::cout << "Derived destructor" << std::endl;
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr;  // Only Base destructor called! Memory leak!
    return 0;
}
```

**The solution (virtual destructor):**
```cpp
class Base {
public:
    virtual ~Base() {  // Virtual destructor
        std::cout << "Base destructor" << std::endl;
    }
};

class Derived : public Base {
private:
    int* data;
public:
    Derived() { data = new int[100]; }
    ~Derived() override {
        delete[] data;
        std::cout << "Derived destructor" << std::endl;
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr;  // Both Derived and Base destructors called!
    return 0;
}
```

**Rule of thumb:** If a class has any virtual functions, it should have a virtual destructor.

---

### 39. What is late binding?

**Late binding** (also called **dynamic binding** or **runtime binding**) is the process of linking a function call to its definition **at runtime** rather than at compile time. In C++, this is achieved through **virtual functions**.

**Key points:**
- The actual function to be called is determined at runtime
- Based on the type of the object being pointed to, not the pointer type
- Requires virtual functions and inheritance
- Implemented using vtables (virtual tables)

```cpp
#include <iostream>

class Base {
public:
    virtual void whoAmI() {  // Virtual function enables late binding
        std::cout << "I am Base" << std::endl;
    }
};

class Derived1 : public Base {
public:
    void whoAmI() override {
        std::cout << "I am Derived1" << std::endl;
    }
};

class Derived2 : public Base {
public:
    void whoAmI() override {
        std::cout << "I am Derived2" << std::endl;
    }
};

void identify(Base& obj) {
    // Which function is called? Decided at runtime (late binding)
    obj.whoAmI();
}

int main() {
    Derived1 d1;
    Derived2 d2;
    Base b;
    
    identify(b);   // Output: I am Base
    identify(d1);  // Output: I am Derived1
    identify(d2);  // Output: I am Derived2
    
    // The same function 'identify' behaves differently at runtime
    return 0;
}
```

---

### 40. What is early binding?

**Early binding** (also called **static binding** or **compile-time binding**) is the process of linking a function call to its definition **at compile time**. The compiler determines exactly which function will be called based on the type of the pointer/variable.

**Key points:**
- Function call is resolved during compilation
- More efficient (no runtime overhead)
- Used for non-virtual functions, overloaded functions, and templates
- Default behavior in C++

```cpp
#include <iostream>

class Base {
public:
    void whoAmI() {  // Non-virtual function - early binding
        std::cout << "I am Base" << std::endl;
    }
};

class Derived : public Base {
public:
    void whoAmI() {  // Not overriding (just hiding) because not virtual
        std::cout << "I am Derived" << std::endl;
    }
};

void identify(Base obj) {  // Pass by value (object slicing occurs!)
    obj.whoAmI();  // Always calls Base::whoAmI() - early binding
}

int main() {
    Derived d;
    Base b;
    
    identify(b);   // Output: I am Base
    identify(d);   // Output: I am Base (Derived part is sliced off!)
    
    // With pointer/reference, still early binding for non-virtual
    Base* ptr = &d;
    ptr->whoAmI(); // Output: I am Base (early binding)
    
    return 0;
}
```

---

### Comparison: Early Binding vs Late Binding

| Feature | Early Binding (Static) | Late Binding (Dynamic) |
| :--- | :--- | :--- |
| **When resolved** | Compile time | Runtime |
| **Functions** | Non-virtual functions | Virtual functions |
| **Performance** | Faster (no overhead) | Slightly slower (vtable lookup) |
| **Flexibility** | Less flexible | More flexible (polymorphism) |
| **Memory** | No extra memory | Requires vtable per class, vptr per object |
| **Determination** | Based on pointer/reference type | Based on actual object type |