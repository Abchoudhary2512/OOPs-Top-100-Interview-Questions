### What is Object-Oriented Programming (OOP)?

Object-Oriented Programming (OOP) is a programming paradigm, or a style of programming, centered around the concept of "objects" rather than just functions and logic. An object is a self-contained entity that contains both **data** (attributes or properties) and **code** (methods or behaviors) that operates on that data.

Think of it as a way to model real-world things in code. For example, a `Car` object would have data like `color`, `model`, and `speed`, and behaviors like `accelerate()`, `brake()`, and `turn()`. The main goal of OOP is to bind together the data and the functions that operate on them so that no other part of the code can access this data except through defined functions. This makes code more modular, reusable, and easier to manage, especially in large and complex software projects.

---

### What are the four pillars of OOP?

The four fundamental concepts that form the foundation of OOP are:

1.  **Encapsulation**
2.  **Abstraction**
3.  **Inheritance**
4.  **Polymorphism**

These are often referred to as the four pillars of OOP.

---

### Define a class in C++.

A **class** in C++ is a user-defined data type that serves as a blueprint or a template for creating objects. It defines the structure and behavior that all objects of that type will have. It bundles data members (variables) and member functions (methods) into a single unit.

```cpp
#include <iostream>
#include <string>

// Class definition (the blueprint)
class Car {
// By default, members are private, but we'll use access specifiers for clarity.
private:
    // Data members (attributes)
    std::string color;
    int speed;

public:
    // Constructor (special member function to initialize objects)
    Car(std::string c, int s) : color(c), speed(s) {}

    // Member function (method) to display car info
    void displayInfo() {
        std::cout << "Car Color: " << color << ", Speed: " << speed << " km/h" << std::endl;
    }

    // Member function to accelerate
    void accelerate(int amount) {
        speed += amount;
        std::cout << "Accelerating. New speed: " << speed << " km/h" << std::endl;
    }
};
```

### What is an object?

An **object** is an instance of a class. When a class is defined, no memory is allocated until an object of that class type is created. You can think of the class as the architectural blueprint for a house, and the object as the actual house built from that blueprint. You can create many objects from a single class, each with its own unique set of data.

```cpp
int main() {
    // Creating objects (instances) of the Car class
    Car myCar("Red", 0);   // 'myCar' is an object
    Car anotherCar("Blue", 50); // 'anotherCar' is another, independent object

    // Using the objects' member functions
    myCar.displayInfo();     // Output: Car Color: Red, Speed: 0 km/h
    myCar.accelerate(30);    // Output: Accelerating. New speed: 30 km/h

    anotherCar.displayInfo(); // Output: Car Color: Blue, Speed: 50 km/h

    return 0;
}
```

### What is encapsulation?

**Encapsulation** is the bundling of data (attributes) and the methods that operate on that data within a single unit, i.e., a class. It also involves **restricting direct access to some of an object's internal components**. This is often achieved through access specifiers like `private` and `protected`. Data hiding is a key aspect of encapsulation.

**Why it's useful:** It protects the integrity of the data. The internal state of an object can only be changed through well-defined, controlled interfaces (public member functions). This prevents accidental or unauthorized modification of data.

In the `Car` example above, the `color` and `speed` are `private`. You cannot directly change `myCar.speed = 100;` from outside the class. You must use the public `accelerate()` method, which could include checks (e.g., preventing negative acceleration).

### What is data abstraction?

**Data Abstraction** is the concept of hiding complex implementation details and showing only the essential features of an object to the outside world. It's about separating *what* an object does from *how* it does it.

In C++, abstraction is achieved through:
1.  **Classes:** A class defines the public interface (the *what*) while keeping its internal workings (the *how*) private.
2.  **Header Files:** Standard library functions like `std::sort()` are a perfect example. You know *what* they do (sort a collection) and how to use them (their interface), but you don't need to see the complex code *how* they implement the sorting algorithm.

**Example:** When you drive a car, you interact with an abstracted interface: the steering wheel, pedals, and gear shift. You don't need to understand the intricate details of the engine, fuel injection, and transmission to drive it. The `Car` class abstracts away the internal mechanics of how `accelerate()` changes the speed, only presenting the method itself.

### What is inheritance?

**Inheritance** is a mechanism that allows you to define a new class (called a **derived** or **child** class) based on an existing class (called a **base** or **parent** class). The derived class inherits (reuses) the members (data and functions) of the base class and can also add its own new members or modify the inherited ones. This promotes code reusability and establishes a hierarchical relationship.

```cpp
// Base class (Parent)
class Vehicle {
protected: // Accessible by derived classes, but not from outside
    std::string brand;
public:
    Vehicle(std::string b) : brand(b) {}
    void honk() {
        std::cout << "Beep beep!" << std::endl;
    }
};

// Derived class (Child) inheriting from Vehicle
class Car : public Vehicle {
private:
    int numberOfDoors;
public:
    // Call base class constructor to initialize inherited members
    Car(std::string b, int doors) : Vehicle(b), numberOfDoors(doors) {}

    void displayInfo() {
        std::cout << "Brand: " << brand << ", Doors: " << numberOfDoors << std::endl;
    }
};

int main() {
    Car myCar("Toyota", 4);
    myCar.honk();        // Inherited from Vehicle
    myCar.displayInfo(); // Defined in Car
    return 0;
}
```
Here, `Car` inherits the `brand` member and the `honk()` function from `Vehicle`.

### What is polymorphism?

**Polymorphism** means "many forms." In OOP, it's the ability of an object or a function to take on multiple forms. It allows you to write code that works on a base class level, but the actual function called is determined by the type of the derived object at runtime. The most common use is when a parent class reference is used to refer to a child class object.

**Virtual functions** are the key to achieving polymorphism in C++.

```cpp
#include <iostream>

class Animal {
public:
    // 'virtual' tells the compiler to support polymorphism for this function
    virtual void makeSound() {
        std::cout << "The animal makes a sound." << std::endl;
    }
};

class Dog : public Animal {
public:
    // Override the base class function (C++11 'override' keyword is recommended)
    void makeSound() override {
        std::cout << "Woof! Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {
        std::cout << "Meow." << std::endl;
    }
};

// This function works polymorphically
void playSound(Animal& animal) {
    animal.makeSound(); // The correct sound (Dog or Cat) will be called!
}

int main() {
    Dog myDog;
    Cat myCat;

    playSound(myDog); // Output: Woof! Woof!
    playSound(myCat); // Output: Meow.

    return 0;
}
```

### What is dynamic binding?

**Dynamic binding** (also known as **late binding**) is the mechanism that makes polymorphism work. It means that the code (the function body) associated with a given function call is not determined until **runtime**.

When you call a virtual function through a base class pointer or reference, the compiler adds extra code behind the scenes to look up which function to execute based on the actual type of the object. This lookup happens when the program is running. In the example above, the call to `animal.makeSound()` is dynamically bound. The program doesn't know whether to call `Animal::makeSound()`, `Dog::makeSound()`, or `Cat::makeSound()` until it knows the actual type of `animal` at runtime.

### What is static binding?

**Static binding** (also known as **early binding**) is the default binding mechanism in C++. It means that the function call is resolved at **compile-time**. The compiler knows exactly which function to call based on the type of the object or reference at the time of compilation.

This happens with:
- **Non-virtual functions**
- **Function overloading**
- **Function calls on objects (not pointers/references)**

```cpp
#include <iostream>

class Math {
public:
    // Overloaded functions - resolved at compile-time (static binding)
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
};

int main() {
    Math math;
    int sum1 = math.add(5, 3);       // Calls the int version - static binding
    double sum2 = math.add(2.5, 1.3); // Calls the double version - static binding
    std::cout << sum1 << ", " << sum2 << std::endl;
    return 0;
}
```
Static binding is faster than dynamic binding because there's no runtime lookup, but it's less flexible.