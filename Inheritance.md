### 21. What is single inheritance?

**Single inheritance** is the simplest form of inheritance where a **derived class inherits from only one base class**. This creates a linear parent-child relationship.

```
    Base Class
        ↑
        |
    Derived Class
```

**Example:**
```cpp
class Animal {          // Base class
public:
    void eat() {
        std::cout << "Eating..." << std::endl;
    }
};

class Dog : public Animal {  // Derived class inherits from ONE base class
public:
    void bark() {
        std::cout << "Barking..." << std::endl;
    }
};

int main() {
    Dog dog;
    dog.eat();  // Inherited from Animal
    dog.bark(); // Own function
    return 0;
}
```

---

### 22. What is multiple inheritance?

**Multiple inheritance** is a feature where a **derived class inherits from more than one base class**. The derived class combines the members of all its base classes.

```
    Base1      Base2
        \      /
         \    /
        Derived Class
```

**Example:**
```cpp
class Printer {
public:
    void print() {
        std::cout << "Printing..." << std::endl;
    }
};

class Scanner {
public:
    void scan() {
        std::cout << "Scanning..." << std::endl;
    }
};

class AllInOne : public Printer, public Scanner {  // Inherits from two classes
public:
    void doBoth() {
        print();
        scan();
    }
};

int main() {
    AllInOne device;
    device.print();  // From Printer
    device.scan();   // From Scanner
    device.doBoth(); // Own function
    return 0;
}
```

---

### 23. What is multilevel inheritance?

**Multilevel inheritance** refers to a chain of inheritance where a class is derived from another derived class. It creates a hierarchy with multiple levels.

```
    Grandparent
        ↑
        |
      Parent
        ↑
        |
      Child
```

**Example:**
```cpp
class Animal {              // Level 1 (Grandparent)
public:
    void breathe() {
        std::cout << "Breathing..." << std::endl;
    }
};

class Mammal : public Animal {  // Level 2 (Parent)
public:
    void giveBirth() {
        std::cout << "Giving birth..." << std::endl;
    }
};

class Dog : public Mammal {     // Level 3 (Child)
public:
    void bark() {
        std::cout << "Barking..." << std::endl;
    }
};

int main() {
    Dog dog;
    dog.breathe();   // From Animal
    dog.giveBirth(); // From Mammal
    dog.bark();      // From Dog
    return 0;
}
```

---

### 24. What is hierarchical inheritance?

**Hierarchical inheritance** is when **multiple derived classes inherit from a single base class**. It creates a tree-like structure where one base class serves as the common source for several derived classes.

```
          Base
        /  |  \
       /   |   \
    Der1  Der2  Der3
```

**Example:**
```cpp
class Shape {           // Base class
public:
    void draw() {
        std::cout << "Drawing a shape" << std::endl;
    }
};

class Circle : public Shape {  // Derived class 1
public:
    void calculateArea() {
        std::cout << "Area = π * r²" << std::endl;
    }
};

class Rectangle : public Shape {  // Derived class 2
public:
    void calculateArea() {
        std::cout << "Area = length * width" << std::endl;
    }
};

class Triangle : public Shape {  // Derived class 3
public:
    void calculateArea() {
        std::cout << "Area = 0.5 * base * height" << std::endl;
    }
};
```

---

### 25. What is hybrid inheritance?

**Hybrid inheritance** is a **combination of two or more types of inheritance** (single, multiple, multilevel, hierarchical). It often leads to complexity and potential ambiguities, such as the diamond problem.

```
      Base
     /    \
    /      \
Derived1  Derived2
    \      /
     \    /
   Derived3
```

**Example:** The diagram above combines hierarchical (Base → Derived1, Base → Derived2) and multiple inheritance (Derived3 ← Derived1, Derived2).

```cpp
class Base {
public:
    void show() {
        std::cout << "Base" << std::endl;
    }
};

class Derived1 : public Base {
    // ...
};

class Derived2 : public Base {
    // ...
};

// Hybrid: Multiple inheritance + Hierarchical
class Derived3 : public Derived1, public Derived2 {
    // This creates the diamond problem!
};
```

---

### 26. What is the diamond problem and how to solve it?

The **diamond problem** is an **ambiguity that arises in hybrid inheritance** (specifically multiple inheritance) when a derived class inherits from two classes that both inherit from a common base class. This creates two copies of the base class members in the final derived class, leading to ambiguity when accessing those members.

```
      Base
     /    \
    /      \
Derived1  Derived2
    \      /
     \    /
   Derived3
```

**The Problem:**
```cpp
class Base {
public:
    void show() { std::cout << "Base" << std::endl; }
};

class Derived1 : public Base { };
class Derived2 : public Base { };

class Derived3 : public Derived1, public Derived2 { };

int main() {
    Derived3 obj;
    obj.show();  // ERROR: Ambiguous - which show()? From Derived1 or Derived2?
    return 0;
}
```

**Solution: Virtual Inheritance**
Use **virtual inheritance** to ensure only one copy of the base class is inherited.

```cpp
class Base {
public:
    void show() { std::cout << "Base" << std::endl; }
};

// Add 'virtual' keyword
class Derived1 : virtual public Base { };
class Derived2 : virtual public Base { };

class Derived3 : public Derived1, public Derived2 { };
// Now only ONE copy of Base exists in Derived3

int main() {
    Derived3 obj;
    obj.show();  // OK! No ambiguity
    return 0;
}
```

---

### 27. What is virtual inheritance?

**Virtual inheritance** is a technique in C++ used to solve the diamond problem. When a base class is inherited as `virtual`, the compiler ensures that **only one copy of that base class's members** are inherited by any further derived classes, regardless of how many inheritance paths lead to it.

**Key points:**
- It's specified using the `virtual` keyword in the inheritance list.
- It adds a level of indirection (usually through virtual base pointers/tables) to locate the shared base class subobject.
- It ensures that the most derived class is responsible for initializing the virtual base class.

```cpp
class Base {
    int data;
public:
    Base(int d = 0) : data(d) {}
};

// Virtual inheritance
class Intermediate1 : virtual public Base {
    // ...
};

class Intermediate2 : virtual public Base {
    // ...
};

// Only ONE copy of Base exists in Derived
class Derived : public Intermediate1, public Intermediate2 {
public:
    // Derived must initialize Base directly
    Derived(int d) : Base(d) {}
};
```

---

### 28. What does "protected" inheritance do?

**Protected inheritance** is an inheritance access specifier where `public` and `protected` members of the base class become `protected` members in the derived class.

**Access changes with protected inheritance:**
| Base Class Member | Access in Derived Class |
| :--- | :--- |
| `public` | `protected` |
| `protected` | `protected` |
| `private` | Not accessible (but still inherited) |

**What this means:**
- Members that were `public` in the base become `protected` in the derived class.
- They are accessible within the derived class and its further descendants.
- They are **NOT accessible** from outside the class hierarchy (by non-member functions or unrelated classes).

```cpp
class Base {
public:
    int publicVar;
protected:
    int protectedVar;
private:
    int privateVar;
};

// Protected inheritance
class Derived : protected Base {
    // publicVar becomes protected
    // protectedVar remains protected
    // privateVar is not accessible
    
    void func() {
        publicVar = 10;    // OK (protected inside Derived)
        protectedVar = 20; // OK (protected inside Derived)
        // privateVar = 30; // ERROR: not accessible
    }
};

class GrandChild : public Derived {
    void func() {
        publicVar = 10;    // OK (still protected in GrandChild)
        protectedVar = 20; // OK (still protected)
    }
};

int main() {
    Derived d;
    // d.publicVar = 10;  // ERROR: publicVar is now protected
    return 0;
}
```

---

### 29. How do you call base class constructor?

The base class constructor is called using the **member initializer list** in the derived class's constructor. This ensures the base class part is properly initialized before the derived class's own members are initialized.

**Syntax:**
```cpp
DerivedConstructor(parameters) : BaseConstructor(parametersForBase), member1(val1), ... {
    // body of derived constructor
}
```

**Example:**
```cpp
class Base {
private:
    int baseValue;
public:
    Base(int x) : baseValue(x) {
        std::cout << "Base constructor called with " << x << std::endl;
    }
};

class Derived : public Base {
private:
    int derivedValue;
public:
    // Call Base constructor explicitly in initializer list
    Derived(int b, int d) : Base(b), derivedValue(d) {
        std::cout << "Derived constructor called" << std::endl;
    }
};

int main() {
    Derived obj(10, 20);  
    // Output:
    // Base constructor called with 10
    // Derived constructor called
    return 0;
}
```

**Important notes:**
- If you don't explicitly call a base constructor, the **default constructor** of the base class is called automatically.
- If the base class doesn't have a default constructor, you **must** explicitly call one of its parameterized constructors.

---

### 30. Can private members be inherited?

**Yes and no** — this requires careful explanation:

- **Are they inherited?** **YES**, private members of a base class **are physically present** in the derived class object. The memory for them is allocated when a derived object is created.

- **Are they accessible?** **NO**, private members **cannot be accessed directly** by the derived class's member functions. They are hidden from the derived class.

**Analogy:** Imagine you inherit a house. The house contains a safe (private member of the base class). The safe is physically in the house (it's inherited), but you don't have the combination to open it (you can't access it directly).

```cpp
class Base {
private:
    int secretNumber;  // Private member
public:
    Base(int s) : secretNumber(s) {}
    
    // Public method to access private member
    int getSecret() const {
        return secretNumber;
    }
};

class Derived : public Base {
public:
    Derived(int s) : Base(s) {}
    
    void tryToAccess() {
        // secretNumber = 100;  // ERROR: cannot access private member directly
        
        // But can access through public/protected base class functions
        int val = getSecret();  // OK: using public member function
        std::cout << "Secret is: " << val << std::endl;
    }
};
```

**How to make base members accessible to derived classes but not outside?**
Use `protected` access specifier instead of `private`.