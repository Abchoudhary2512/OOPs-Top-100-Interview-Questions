
### 11. Difference between a class and a struct in C++.

In C++, **classes** and **structs** are very similar. They both can contain data members and member functions, support inheritance, and are used to create blueprints for objects. The primary difference lies in their **default access levels** and the **default type of inheritance**.

A common convention (though not enforced by the compiler) is to use `struct` for simple data containers (Plain Old Data - POD) and `class` for more complex objects that require encapsulation and invariants.

---

### 12. Explain default access modifiers for class vs struct.

This is the core technical difference between them:

- **`struct`**: By default, all members (variables and functions) and base classes are **`public`**.
- **`class`**: By default, all members and base classes are **`private`**.

```cpp
// Struct Example - Default access is PUBLIC
struct MyStruct {
    int x;  // This is public by default
    void func() {} // This is public by default
};

// Class Example - Default access is PRIVATE
class MyClass {
    int y;  // This is private by default
    void func() {} // This is private by default
public:
    // Must explicitly specify public to make members accessible from outside
    int z; // This is now public
};

// Inheritance example
struct A {};
struct B : A {}; // Default inheritance is PUBLIC (B publically inherits from A)

class C {};
class D : C {};   // Default inheritance is PRIVATE (D privately inherits from C)
```

---

### 13. What is a constructor?

A **constructor** is a special member function of a class that is **automatically called when an object of that class is created (instantiated)**. Its main purpose is to **initialize the object's data members** and establish any class invariants (ensuring the object starts in a valid state).

**Key characteristics:**
- It has the **same name as the class**.
- It **does not have a return type**, not even `void`.
- It can be overloaded (multiple constructors with different parameters).
- It is automatically invoked when an object is defined.

```cpp
class Rectangle {
private:
    double width, height;
public:
    // Constructor
    Rectangle(double w, double h) {
        width = w;
        height = h;
        std::cout << "A rectangle is created!" << std::endl;
    }
};

int main() {
    Rectangle rect(5.0, 3.0); // Constructor is called here
    return 0;
}
```

---

### 14. What are types of constructors?

There are several types of constructors in C++:

1.  **Default Constructor:** A constructor that takes no arguments. If you don't define any constructors, the compiler will automatically generate one (which does nothing). You can also define your own.
    ```cpp
    class Point {
    public:
        Point() { x = 0; y = 0; } // User-defined default constructor
        private: int x, y;
    };
    ```

2.  **Parameterized Constructor:** A constructor that takes arguments. This allows you to initialize an object with specific values.
    ```cpp
    class Point {
    public:
        Point(int x1, int y1) { x = x1; y = y1; } // Parameterized
        private: int x, y;
    };
    // Usage: Point p1(10, 20);
    ```

3.  **Copy Constructor:** A constructor that initializes an object using another object of the same class. It takes a reference to the class as a parameter (usually `const`).
    ```cpp
    class Point {
    public:
        Point(const Point &p) { x = p.x; y = p.y; } // Copy constructor
        private: int x, y;
    };
    // Usage: Point p2(p1); or Point p3 = p1;
    ```

4.  **Move Constructor (C++11 onwards):** A constructor that "steals" the resources from a temporary object, avoiding expensive deep copies. It takes an rvalue reference (`&&`).
    ```cpp
    class Buffer {
    public:
        Buffer(Buffer&& other) { // Move constructor
            data = other.data; // Steal the pointer
            other.data = nullptr; // Leave the other in a valid but empty state
        }
    private: int* data;
    };
    ```

---

### 15. What is destructor?

A **destructor** is a special member function that is **automatically called when an object goes out of scope or is explicitly deleted**. Its main purpose is to **release resources** acquired by the object during its lifetime (like dynamic memory, file handles, network connections) to prevent memory leaks and resource leaks.

**Key characteristics:**
- It has the **same name as the class**, preceded by a tilde (`~`).
- It **cannot take arguments** (no overloading is possible).
- It **does not have a return type**.
- There is **exactly one destructor** per class.

```cpp
class MyArray {
private:
    int* arr;
    int size;
public:
    MyArray(int s) { // Constructor
        size = s;
        arr = new int[size]; // Dynamically allocate memory
        std::cout << "Array allocated." << std::endl;
    }

    ~MyArray() { // Destructor
        delete[] arr; // Release the dynamically allocated memory
        std::cout << "Array deallocated." << std::endl;
    }
};

int main() {
    MyArray myArray(10); // Constructor called
    // ... use the array ...
    return 0; // Destructor called automatically for myArray here
}
```

---

### 16. What is the “this” pointer in C++?

The **`this` pointer** is an implicit pointer available inside all non-static member functions of a class. It points to the **current object** for which the member function is called.

**Key uses:**
- **To distinguish between local parameters and member variables** (especially when they have the same name).
- **To return the current object from a member function** (to enable method chaining).

```cpp
class Example {
private:
    int value;
public:
    Example(int value) {
        // 'value' on the left is the member variable.
        // 'value' on the right is the constructor parameter.
        this->value = value;
    }

    Example& setValue(int value) {
        this->value = value;
        return *this; // Returning the current object
    }

    void print() {
        std::cout << "Value: " << this->value << std::endl; // 'this->' is optional here
    }
};

int main() {
    Example obj(10);
    obj.setValue(20).print(); // Method chaining works because setValue returns *this
    return 0;
}
```

---

### 17. Can constructors be virtual?

**No, constructors cannot be virtual in C++.**

**Why?** The concept of `virtual` relies on the object's **vtable (virtual table)** to determine which function to call at runtime. However, when a constructor is called, the object **does not exist yet**. The constructor's job is to create the object and, as part of that, to set up the vtable. If a constructor were virtual, there would be no vtable to resolve the call, creating a chicken-and-egg problem. The constructor must be statically bound to the exact type being created.

---

### 18. What is a copy constructor?

A **copy constructor** is a specific type of constructor used to create a new object as a copy of an existing object. It is called whenever a new object is created from an existing object.

**When is it called?**
1.  When an object is passed **by value** to a function.
2.  When an object is returned **by value** from a function.
3.  When an object is initialized using another object: `MyClass obj2 = obj1;` or `MyClass obj2(obj1);`

The compiler provides a default copy constructor that performs a member-wise copy (which leads to shallow copy problems if you have pointers). You can also define your own to perform a deep copy.

```cpp
class MyString {
private:
    char* data;
public:
    // Copy Constructor
    MyString(const MyString &source) {
        // Allocate new memory and copy the content from source.data
        data = new char[strlen(source.data) + 1];
        strcpy(data, source.data);
        std::cout << "Copy constructor called!" << std::endl;
    }
    // ... other members (constructors, destructor)
};
```

---

### 19. Difference between shallow and deep copy?

This distinction is crucial when a class manages dynamic memory (pointers).

- **Shallow Copy:** A default, member-wise copy where only the pointer values are copied, **not the data they point to**. After a shallow copy, two objects will have pointers pointing to the **same memory location**. This leads to problems like double deletion in destructors and data corruption.

- **Deep Copy:** A programmer-defined copy where, along with the pointer, the **data it points to is also copied** to a newly allocated memory location. Each object has its **own separate copy of the data**, making them independent.

| Feature | Shallow Copy | Deep Copy |
| :--- | :--- | :--- |
| **What is copied?** | The values of the members, including pointers (the memory address). | The values of the members. For pointers, the actual data being pointed to is also copied. |
| **Memory for pointed data** | Shared between the original and the copy. | Separate for the original and the copy. |
| **Pointer after copy** | Both objects point to the same memory address. | Each object points to its own distinct memory block. |
| **Default behavior?** | Provided by the compiler-generated copy constructor and assignment operator. | Must be implemented by the programmer. |
| **Risk** | Dangling pointers, double deletion, data corruption. | No risk of interference between objects. |

**Visual Example:**
- **Shallow Copy:** Object A (ptr -> [Data]) --- Copy to Object B ---> Object B (ptr -> [Data]) *(same block)*
- **Deep Copy:** Object A (ptr -> [Data1]) --- Copy to Object B ---> Object B (ptr -> [Data2]) *(Data2 is a copy of Data1)*

---

### 20. What is operator overloading?

**Operator overloading** is a form of compile-time polymorphism in C++ that allows you to redefine or give special meanings to C++ operators (like `+`, `-`, `=`, `[]`, `<<`, etc.) when they are used with user-defined data types (objects of a class).

It allows you to write code that is more intuitive and natural. For example, you can define the `+` operator for a `Complex` class so that you can add two complex numbers just like you would with built-in types.

```cpp
class Complex {
private:
    double real, imag;
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // Overloading the '+' operator as a member function
    Complex operator + (const Complex &other) const {
        return Complex(real + other.real, imag + other.imag);
    }

    // Overloading the '<<' operator (often as a friend function)
    friend std::ostream& operator << (std::ostream &out, const Complex &c) {
        out << c.real << " + " << c.imag << "i";
        return out;
    }
};

int main() {
    Complex c1(3.0, 4.0);
    Complex c2(1.0, 2.0);

    Complex c3 = c1 + c2; // Uses the overloaded '+' operator

    std::cout << c3 << std::endl; // Uses the overloaded '<<' operator. Output: 4 + 6i
    return 0;
}
```