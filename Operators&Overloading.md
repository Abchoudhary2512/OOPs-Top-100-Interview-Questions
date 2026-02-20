### 81. What is operator overloading?

**Operator overloading** is a form of compile-time polymorphism that allows you to **redefine the behavior of C++ operators** for user-defined types (classes). It enables objects to be used with intuitive syntax similar to built-in types.

```cpp
#include <iostream>
#include <string>
#include <vector>

// ==================== Complex Number Class with Operator Overloading ====================

class Complex {
private:
    double real;
    double imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // Overloaded + operator
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    // Overloaded - operator
    Complex operator-(const Complex& other) const {
        return Complex(real - other.real, imag - other.imag);
    }
    
    // Overloaded * operator
    Complex operator*(const Complex& other) const {
        return Complex(
            real * other.real - imag * other.imag,
            real * other.imag + imag * other.real
        );
    }
    
    // Overloaded += operator
    Complex& operator+=(const Complex& other) {
        real += other.real;
        imag += other.imag;
        return *this;
    }
    
    // Overloaded == operator
    bool operator==(const Complex& other) const {
        return (real == other.real) && (imag == other.imag);
    }
    
    // Overloaded != operator
    bool operator!=(const Complex& other) const {
        return !(*this == other);
    }
    
    // Overloaded << operator (as friend)
    friend std::ostream& operator<<(std::ostream& os, const Complex& c);
    
    // Overloaded >> operator (as friend)
    friend std::istream& operator>>(std::istream& is, Complex& c);
    
    // Unary - operator
    Complex operator-() const {
        return Complex(-real, -imag);
    }
    
    // Prefix ++ operator
    Complex& operator++() {
        ++real;
        return *this;
    }
    
    // Postfix ++ operator
    Complex operator++(int) {
        Complex temp = *this;
        ++real;
        return temp;
    }
    
    // Conversion operator to double (returns magnitude)
    operator double() const {
        return std::sqrt(real * real + imag * imag);
    }
};

// Overloaded stream insertion operator
std::ostream& operator<<(std::ostream& os, const Complex& c) {
    os << c.real;
    if (c.imag >= 0) {
        os << " + " << c.imag << "i";
    } else {
        os << " - " << -c.imag << "i";
    }
    return os;
}

// Overloaded stream extraction operator
std::istream& operator>>(std::istream& is, Complex& c) {
    std::cout << "Enter real and imaginary parts: ";
    is >> c.real >> c.imag;
    return is;
}

// ==================== Another Example: Vector3D ====================

class Vector3D {
private:
    double x, y, z;
    
public:
    Vector3D(double x = 0, double y = 0, double z = 0) : x(x), y(y), z(z) {}
    
    // Dot product
    double operator*(const Vector3D& other) const {
        return x * other.x + y * other.y + z * other.z;
    }
    
    // Cross product
    Vector3D operator^(const Vector3D& other) const {
        return Vector3D(
            y * other.z - z * other.y,
            z * other.x - x * other.z,
            x * other.y - y * other.x
        );
    }
    
    // Scalar multiplication
    Vector3D operator*(double scalar) const {
        return Vector3D(x * scalar, y * scalar, z * scalar);
    }
    
    // Vector addition
    Vector3D operator+(const Vector3D& other) const {
        return Vector3D(x + other.x, y + other.y, z + other.z);
    }
    
    // Subscript operator
    double& operator[](int index) {
        if (index == 0) return x;
        if (index == 1) return y;
        return z;
    }
    
    const double& operator[](int index) const {
        if (index == 0) return x;
        if (index == 1) return y;
        return z;
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Vector3D& v) {
        os << "(" << v.x << ", " << v.y << ", " << v.z << ")";
        return os;
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Complex Number Operations ===" << std::endl;
    
    Complex c1(3, 4);
    Complex c2(1, 2);
    
    std::cout << "c1 = " << c1 << std::endl;
    std::cout << "c2 = " << c2 << std::endl;
    
    Complex c3 = c1 + c2;
    std::cout << "c1 + c2 = " << c3 << std::endl;
    
    Complex c4 = c1 * c2;
    std::cout << "c1 * c2 = " << c4 << std::endl;
    
    c1 += c2;
    std::cout << "c1 += c2 = " << c1 << std::endl;
    
    std::cout << "c1 == c2? " << (c1 == c2 ? "Yes" : "No") << std::endl;
    
    Complex c5 = -c2;
    std::cout << "-c2 = " << c5 << std::endl;
    
    std::cout << "Magnitude of c1: " << static_cast<double>(c1) << std::endl;
    
    std::cout << "\n=== Vector3D Operations ===" << std::endl;
    
    Vector3D v1(1, 2, 3);
    Vector3D v2(4, 5, 6);
    
    std::cout << "v1 = " << v1 << std::endl;
    std::cout << "v2 = " << v2 << std::endl;
    
    double dot = v1 * v2;
    std::cout << "v1 · v2 = " << dot << std::endl;
    
    Vector3D cross = v1 ^ v2;
    std::cout << "v1 × v2 = " << cross << std::endl;
    
    Vector3D v3 = v1 * 2.5;
    std::cout << "v1 * 2.5 = " << v3 << std::endl;
    
    std::cout << "v1[1] = " << v1[1] << std::endl;
    
    return 0;
}
```

---

### 82. Which operators cannot be overloaded?

Most C++ operators can be overloaded, but there are **five operators that cannot be overloaded**:

```cpp
#include <iostream>

// ==================== Operators That CANNOT Be Overloaded ====================

/*
 * 1. ::  - Scope resolution operator
 * 2. .   - Member access operator
 * 3. .*  - Pointer-to-member operator
 * 4. ?:  - Ternary conditional operator
 * 5. sizeof - Sizeof operator
 * 6. typeid - Type identification operator (C++ RTTI)
 * 7. alignof - Alignment operator (C++11)
 */

// ==================== Demonstration of What CAN Be Overloaded ====================

class Demo {
private:
    int value;
    
public:
    Demo(int v = 0) : value(v) {}
    
    // Most operators CAN be overloaded:
    
    // Arithmetic operators
    Demo operator+(const Demo& other) const { return Demo(value + other.value); }
    Demo operator-(const Demo& other) const { return Demo(value - other.value); }
    Demo operator*(const Demo& other) const { return Demo(value * other.value); }
    Demo operator/(const Demo& other) const { return Demo(value / other.value); }
    Demo operator%(const Demo& other) const { return Demo(value % other.value); }
    
    // Bitwise operators
    Demo operator&(const Demo& other) const { return Demo(value & other.value); }
    Demo operator|(const Demo& other) const { return Demo(value | other.value); }
    Demo operator^(const Demo& other) const { return Demo(value ^ other.value); }
    Demo operator~() const { return Demo(~value); }
    Demo operator<<(int shift) const { return Demo(value << shift); }
    Demo operator>>(int shift) const { return Demo(value >> shift); }
    
    // Assignment operators
    Demo& operator=(const Demo& other) { value = other.value; return *this; }
    Demo& operator+=(const Demo& other) { value += other.value; return *this; }
    Demo& operator-=(const Demo& other) { value -= other.value; return *this; }
    
    // Comparison operators
    bool operator==(const Demo& other) const { return value == other.value; }
    bool operator!=(const Demo& other) const { return value != other.value; }
    bool operator<(const Demo& other) const { return value < other.value; }
    bool operator>(const Demo& other) const { return value > other.value; }
    bool operator<=(const Demo& other) const { return value <= other.value; }
    bool operator>=(const Demo& other) const { return value >= other.value; }
    
    // Increment/Decrement
    Demo& operator++() { ++value; return *this; }        // Prefix
    Demo operator++(int) { Demo temp = *this; ++value; return temp; } // Postfix
    
    // Subscript
    int& operator[](int index) { return value; }  // Not meaningful here
    
    // Function call
    void operator()(const std::string& msg) { 
        std::cout << "Demo says: " << msg << " (value=" << value << ")" << std::endl;
    }
    
    // Member access (->)
    Demo* operator->() { return this; }
    
    // Pointer dereference (*)
    Demo& operator*() { return *this; }
    
    // Comma operator
    Demo& operator,(const Demo& other) { 
        std::cout << "Comma operator called" << std::endl;
        return *this;
    }
    
    // New and Delete
    static void* operator new(size_t size) {
        std::cout << "Custom new for Demo" << std::endl;
        return ::operator new(size);
    }
    
    static void operator delete(void* ptr) {
        std::cout << "Custom delete for Demo" << std::endl;
        ::operator delete(ptr);
    }
    
    // Conversion operators
    operator int() const { return value; }
    operator bool() const { return value != 0; }
    
    // Stream operators (as friends)
    friend std::ostream& operator<<(std::ostream& os, const Demo& d) {
        os << "Demo(" << d.value << ")";
        return os;
    }
    
    friend std::istream& operator>>(std::istream& is, Demo& d) {
        is >> d.value;
        return is;
    }
};

// ==================== What Cannot Be Done ====================

/*
// These would cause compilation errors:
class Illegal {
public:
    // void operator::() {}     // ERROR: cannot overload ::
    // void operator.() {}      // ERROR: cannot overload .
    // void operator.*() {}     // ERROR: cannot overload .*
    // void operator?:(int) {}  // ERROR: cannot overload ?:
};
*/

int main() {
    std::cout << "=== Operators That CAN Be Overloaded ===" << std::endl;
    
    Demo d1(10), d2(20);
    
    auto sum = d1 + d2;
    std::cout << "d1 + d2 = " << sum << std::endl;
    
    d1("Hello from function call operator");
    
    std::cout << "\n=== Operators That CANNOT Be Overloaded ===" << std::endl;
    std::cout << "1. :: - Scope resolution\n";
    std::cout << "2. .  - Member access\n";
    std::cout << "3. .* - Pointer-to-member\n";
    std::cout << "4. ?: - Ternary conditional\n";
    std::cout << "5. sizeof - Sizeof operator\n";
    std::cout << "6. typeid - Type identification\n";
    std::cout << "7. alignof - Alignment operator\n";
    
    std::cout << "\n=== Why Some Operators Can't Be Overloaded ===" << std::endl;
    std::cout << "- Prevent ambiguity and maintain expected behavior\n";
    std::cout << "- Some operators work on names, not values (::, .)\n";
    std::cout << "- Short-circuit behavior of ?: would be lost\n";
    std::cout << "- sizeof requires compile-time evaluation\n";
    
    return 0;
}
```

**Summary of Non-Overloadable Operators:**

| Operator | Name | Reason Not Overloadable |
| :--- | :--- | :--- |
| `::` | Scope resolution | Operates on names, not values |
| `.` | Member access | Would break class member access semantics |
| `.*` | Pointer-to-member | Complex semantics, would be ambiguous |
| `?:` | Ternary conditional | Short-circuit evaluation would be lost |
| `sizeof` | Size of | Requires compile-time evaluation |
| `typeid` | Type identification | Part of RTTI system |
| `alignof` | Alignment | Compile-time requirement |

---

### 83. How to overload assignment operator?

The **assignment operator** (`=`) must be overloaded as a **member function**. It's crucial for proper resource management, especially when a class manages dynamic memory (Rule of Three/Five).

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

// ==================== Basic Assignment Operator ====================

class SimpleString {
private:
    char* data;
    size_t length;
    
public:
    // Constructor
    SimpleString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        std::cout << "Constructed: " << data << std::endl;
    }
    
    // Copy constructor
    SimpleString(const SimpleString& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        std::cout << "Copy constructed: " << data << std::endl;
    }
    
    // Assignment operator - CORRECT IMPLEMENTATION
    SimpleString& operator=(const SimpleString& other) {
        std::cout << "Assignment operator: " << other.data << std::endl;
        
        // 1. Self-assignment check
        if (this == &other) {
            return *this;
        }
        
        // 2. Release existing resources
        delete[] data;
        
        // 3. Allocate new resources and copy
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        
        // 4. Return *this to allow chaining
        return *this;
    }
    
    // Destructor
    ~SimpleString() {
        std::cout << "Destructed: " << (data ? data : "null") << std::endl;
        delete[] data;
    }
    
    void print() const {
        std::cout << "String: " << (data ? data : "empty") << std::endl;
    }
};

// ==================== Modern Copy-and-Swap Idiom ====================

class ModernString {
private:
    char* data;
    size_t length;
    
public:
    ModernString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
    }
    
    // Copy constructor
    ModernString(const ModernString& other) 
        : data(new char[other.length + 1]), length(other.length) {
        strcpy(data, other.data);
    }
    
    // Destructor
    ~ModernString() {
        delete[] data;
    }
    
    // Swap function
    void swap(ModernString& other) noexcept {
        std::swap(data, other.data);
        std::swap(length, other.length);
    }
    
    // Assignment operator using copy-and-swap
    ModernString& operator=(ModernString other) {  // Note: pass by value
        swap(other);  // Swap with the copy
        return *this;
    }
    
    void print() const {
        std::cout << "ModernString: " << data << std::endl;
    }
};

// ==================== Move Assignment Operator (C++11) ====================

class MoveEnabledString {
private:
    char* data;
    size_t length;
    
public:
    MoveEnabledString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        std::cout << "Constructed: " << data << std::endl;
    }
    
    // Copy constructor
    MoveEnabledString(const MoveEnabledString& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        std::cout << "Copy constructed: " << data << std::endl;
    }
    
    // Move constructor
    MoveEnabledString(MoveEnabledString&& other) noexcept
        : data(other.data), length(other.length) {
        other.data = nullptr;
        other.length = 0;
        std::cout << "Move constructed" << std::endl;
    }
    
    // Copy assignment
    MoveEnabledString& operator=(const MoveEnabledString& other) {
        std::cout << "Copy assignment" << std::endl;
        if (this != &other) {
            delete[] data;
            length = other.length;
            data = new char[length + 1];
            strcpy(data, other.data);
        }
        return *this;
    }
    
    // Move assignment
    MoveEnabledString& operator=(MoveEnabledString&& other) noexcept {
        std::cout << "Move assignment" << std::endl;
        if (this != &other) {
            delete[] data;
            data = other.data;
            length = other.length;
            other.data = nullptr;
            other.length = 0;
        }
        return *this;
    }
    
    ~MoveEnabledString() {
        std::cout << "Destructed: " << (data ? data : "null") << std::endl;
        delete[] data;
    }
    
    void print() const {
        std::cout << "String: " << (data ? data : "empty") << std::endl;
    }
};

// ==================== Assignment Operator for Template Class ====================

template <typename T>
class SmartArray {
private:
    T* data;
    size_t size;
    
public:
    SmartArray(size_t n) : size(n), data(new T[n]) {
        std::cout << "SmartArray constructed with size " << size << std::endl;
    }
    
    // Template assignment operator
    template <typename U>
    SmartArray& operator=(const SmartArray<U>& other) {
        if (static_cast<const void*>(this) != static_cast<const void*>(&other)) {
            delete[] data;
            size = other.size();
            data = new T[size];
            for (size_t i = 0; i < size; ++i) {
                data[i] = static_cast<T>(other[i]);
            }
        }
        return *this;
    }
    
    size_t getSize() const { return size; }
    
    T& operator[](size_t index) { return data[index]; }
    const T& operator[](size_t index) const { return data[index]; }
    
    ~SmartArray() {
        delete[] data;
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Assignment Operator ===" << std::endl;
    
    SimpleString s1("Hello");
    SimpleString s2("World");
    SimpleString s3("C++");
    
    std::cout << "\nBefore assignment:" << std::endl;
    s1.print();
    s2.print();
    
    s2 = s1;  // Assignment
    std::cout << "\nAfter s2 = s1:" << std::endl;
    s1.print();
    s2.print();
    
    s3 = s2 = s1;  // Chaining
    std::cout << "\nAfter chaining:" << std::endl;
    s1.print();
    s2.print();
    s3.print();
    
    std::cout << "\n=== Copy-and-Swap Idiom ===" << std::endl;
    
    ModernString ms1("Modern");
    ModernString ms2("String");
    
    ms2 = ms1;  // Clean, exception-safe assignment
    
    std::cout << "\n=== Move Assignment ===" << std::endl;
    
    MoveEnabledString m1("Move");
    MoveEnabledString m2("Assignment");
    
    m2 = std::move(m1);  // Move assignment
    std::cout << "After move:" << std::endl;
    m2.print();
    // m1 is now in valid but unspecified state
    
    return 0;
}
```

**Key Points for Assignment Operator:**

| Rule | Why |
| :--- | :--- |
| **Check self-assignment** | Prevent deleting resources before copying |
| **Return *this** | Enable chaining (a = b = c) |
| **Release old resources** | Avoid memory leaks |
| **Copy all members** | Ensure complete object state |
| **Exception safety** | Provide strong guarantee if possible |

---

### 84. What is conversion operator?

A **conversion operator** (also called **type-cast operator**) allows a class to define **implicit or explicit conversions** to other types. It's defined using `operator type()` syntax.

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <cmath>

// ==================== Basic Conversion Operators ====================

class Integer {
private:
    int value;
    
public:
    Integer(int v = 0) : value(v) {}
    
    // Conversion to int (implicit)
    operator int() const {
        std::cout << "Converting Integer to int: " << value << std::endl;
        return value;
    }
    
    // Conversion to double (implicit)
    operator double() const {
        std::cout << "Converting Integer to double: " << static_cast<double>(value) << std::endl;
        return static_cast<double>(value);
    }
    
    // Conversion to bool (implicit)
    operator bool() const {
        std::cout << "Converting Integer to bool: " << (value != 0) << std::endl;
        return value != 0;
    }
    
    // Conversion to string (explicit - C++11)
    explicit operator std::string() const {
        return std::to_string(value);
    }
};

// ==================== Multiple Conversion Operators ====================

class Rational {
private:
    int numerator;
    int denominator;
    
public:
    Rational(int num = 0, int den = 1) : numerator(num), denominator(den) {
        if (denominator == 0) denominator = 1;
    }
    
    // Conversion to double
    operator double() const {
        return static_cast<double>(numerator) / denominator;
    }
    
    // Conversion to int (truncates)
    operator int() const {
        return numerator / denominator;
    }
    
    // Conversion to std::string
    operator std::string() const {
        return std::to_string(numerator) + "/" + std::to_string(denominator);
    }
    
    // Conversion to bool (true if non-zero)
    operator bool() const {
        return numerator != 0;
    }
};

// ==================== Safe Bool Idiom (pre-C++11) ====================

class SafeBool {
private:
    int value;
    
public:
    SafeBool(int v) : value(v) {}
    
    // Safe conversion to bool (pre-C++11)
    typedef void (SafeBool::*BoolType)() const;
    void trueCondition() const {}
    
    operator BoolType() const {
        return value != 0 ? &SafeBool::trueCondition : nullptr;
    }
};

// ==================== Conversion with Error Handling ====================

class Percentage {
private:
    double value;
    
public:
    Percentage(double v) : value(v) {
        if (v < 0 || v > 100) {
            throw std::out_of_range("Percentage must be between 0 and 100");
        }
    }
    
    // Convert to double (the percentage value)
    operator double() const {
        return value;
    }
    
    // Convert to decimal (0.0 to 1.0)
    operator float() const {
        return static_cast<float>(value / 100.0);
    }
    
    // Explicit conversion to string
    explicit operator std::string() const {
        return std::to_string(value) + "%";
    }
};

// ==================== Conversion Operators in Templates ====================

template <typename T>
class Wrapper {
private:
    T value;
    
public:
    Wrapper(const T& v) : value(v) {}
    
    // Conversion to wrapped type
    operator T() const {
        return value;
    }
    
    // Conversion to other types if possible
    template <typename U>
    operator U() const {
        return static_cast<U>(value);
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Conversion Operators ===" << std::endl;
    
    Integer num(42);
    
    int i = num;                 // implicit conversion to int
    double d = num;              // implicit conversion to double
    bool b = num;                // implicit conversion to bool
    
    std::cout << "i = " << i << std::endl;
    std::cout << "d = " << d << std::endl;
    std::cout << "b = " << b << std::endl;
    
    // Explicit conversion required
    std::string s = static_cast<std::string>(num);
    std::cout << "s = " << s << std::endl;
    
    std::cout << "\n=== Rational Number Conversions ===" << std::endl;
    
    Rational r(3, 4);
    
    double d2 = r;                // 0.75
    int i2 = r;                   // 0 (truncated)
    std::string s2 = r;           // "3/4"
    bool b2 = r;                  // true (3 != 0)
    
    std::cout << "Rational(3/4) as double: " << d2 << std::endl;
    std::cout << "Rational(3/4) as int: " << i2 << std::endl;
    std::cout << "Rational(3/4) as string: " << s2 << std::endl;
    std::cout << "Rational(3/4) as bool: " << b2 << std::endl;
    
    std::cout << "\n=== Percentage Conversions ===" << std::endl;
    
    Percentage pct(75);
    
    double val = pct;             // 75.0
    float dec = pct;              // 0.75f
    std::string pctStr = static_cast<std::string>(pct);  // "75.000000%"
    
    std::cout << "75% as double: " << val << std::endl;
    std::cout << "75% as float: " << dec << std::endl;
    std::cout << "75% as string: " << pctStr << std::endl;
    
    std::cout << "\n=== Template Wrapper ===" << std::endl;
    
    Wrapper<int> w(42);
    
    int wi = w;                   // OK
    double wd = w;                // Converts int to double
    // std::string ws = w;        // Error: can't convert int to string
    
    std::cout << "wi = " << wi << std::endl;
    std::cout << "wd = " << wd << std::endl;
    
    std::cout << "\n=== Implicit vs Explicit Conversions ===" << std::endl;
    
    Integer int1(5);
    Integer int2(0);
    
    // Implicit conversions
    if (int1) {  // bool conversion
        std::cout << "int1 is true" << std::endl;
    }
    
    if (!int2) {  // bool conversion
        std::cout << "int2 is false" << std::endl;
    }
    
    int arr[10];
    int index = int1;  // Convert to int for array indexing
    arr[index] = 100;
    
    // Explicit conversion required for string
    // std::string str = int1;  // Error: conversion is explicit
    
    return 0;
}
```

**Types of Conversion Operators:**

| Type | Syntax | Typical Use |
| :--- | :--- | :--- |
| **To numeric** | `operator int()` | Convert to arithmetic types |
| **To bool** | `operator bool()` | Enable conditional checks |
| **To string** | `operator std::string()` | String representation |
| **To custom types** | `operator MyType()` | Cross-type conversions |
| **Explicit** | `explicit operator T()` | Prevent implicit conversions |

---

### 85. What is function call operator overloading?

The **function call operator** `()` allows objects to be used **as if they were functions**. Such objects are called **functors** or **function objects**. They're particularly useful in STL algorithms and for creating callable objects with state.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>

// ==================== Basic Functor ====================

class Multiplier {
private:
    int factor;
    
public:
    Multiplier(int f) : factor(f) {}
    
    // Overloaded function call operator
    int operator()(int x) const {
        return x * factor;
    }
};

// ==================== Functor with State ====================

class Counter {
private:
    int count;
    
public:
    Counter() : count(0) {}
    
    int operator()() {
        return ++count;
    }
    
    int getCount() const { return count; }
    void reset() { count = 0; }
};

// ==================== Functor for STL Algorithms ====================

class Adder {
private:
    int total;
    
public:
    Adder() : total(0) {}
    
    void operator()(int x) {
        total += x;
    }
    
    int getTotal() const { return total; }
};

// ==================== Parameterized Functor ====================

class Between {
private:
    int low, high;
    
public:
    Between(int l, int h) : low(l), high(h) {}
    
    bool operator()(int x) const {
        return x >= low && x <= high;
    }
};

// ==================== Functor Template ====================

template <typename T>
class Comparator {
private:
    bool ascending;
    
public:
    Comparator(bool asc = true) : ascending(asc) {}
    
    bool operator()(const T& a, const T& b) const {
        return ascending ? (a < b) : (a > b);
    }
};

// ==================== Recursive Functor ====================

class Factorial {
public:
    int operator()(int n) const {
        if (n <= 1) return 1;
        return n * (*this)(n - 1);  // Recursive call
    }
};

// ==================== Functor with Multiple Parameters ====================

class Power {
public:
    double operator()(double base, int exponent) const {
        double result = 1.0;
        for (int i = 0; i < exponent; ++i) {
            result *= base;
        }
        return result;
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Functor ===" << std::endl;
    
    Multiplier times2(2);
    Multiplier times3(3);
    
    std::cout << "times2(5) = " << times2(5) << std::endl;
    std::cout << "times3(5) = " << times3(5) << std::endl;
    
    std::cout << "\n=== Functor with State ===" << std::endl;
    
    Counter counter;
    std::cout << "counter() = " << counter() << std::endl;
    std::cout << "counter() = " << counter() << std::endl;
    std::cout << "counter() = " << counter() << std::endl;
    std::cout << "Total calls: " << counter.getCount() << std::endl;
    
    std::cout << "\n=== Functor with STL Algorithms ===" << std::endl;
    
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // Use functor with for_each
    Adder adder;
    std::for_each(numbers.begin(), numbers.end(), std::ref(adder));
    std::cout << "Sum of numbers: " << adder.getTotal() << std::endl;
    
    // Use functor with count_if
    Between between3and7(3, 7);
    int count = std::count_if(numbers.begin(), numbers.end(), between3and7);
    std::cout << "Numbers between 3 and 7: " << count << std::endl;
    
    std::cout << "\n=== Functor Template ===" << std::endl;
    
    std::vector<int> unsorted = {5, 2, 8, 1, 9, 3, 7, 4, 6};
    
    // Sort ascending
    Comparator<int> ascending(true);
    std::sort(unsorted.begin(), unsorted.end(), ascending);
    std::cout << "Ascending: ";
    for (int x : unsorted) std::cout << x << " ";
    std::cout << std::endl;
    
    // Sort descending
    Comparator<int> descending(false);
    std::sort(unsorted.begin(), unsorted.end(), descending);
    std::cout << "Descending: ";
    for (int x : unsorted) std::cout << x << " ";
    std::cout << std::endl;
    
    std::cout << "\n=== Recursive Functor ===" << std::endl;
    
    Factorial fact;
    for (int i = 1; i <= 10; ++i) {
        std::cout << i << "! = " << fact(i) << std::endl;
    }
    
    std::cout << "\n=== Multi-parameter Functor ===" << std::endl;
    
    Power power;
    std::cout << "2^10 = " << power(2, 10) << std::endl;
    std::cout << "3^4 = " << power(3, 4) << std::endl;
    
    std::cout << "\n=== Lambda vs Functor Comparison ===" << std::endl;
    
    // Lambda (C++11)
    auto lambda = [factor = 2](int x) { return x * factor; };
    
    // Functor
    Multiplier functor(2);
    
    std::cout << "lambda(10) = " << lambda(10) << std::endl;
    std::cout << "functor(10) = " << functor(10) << std::endl;
    
    std::cout << "\n=== Benefits of Functors ===" << std::endl;
    std::cout << "1. Can maintain state between calls" << std::endl;
    std::cout << "2. Can have multiple instances with different states" << std::endl;
    std::cout << "3. Can be passed to STL algorithms" << std::endl;
    std::cout << "4. Better performance than function pointers in some cases" << std::endl;
    
    return 0;
}
```

**Benefits of Functors over Function Pointers:**

| Feature | Functors | Function Pointers |
| :--- | :--- | :--- |
| **State** | Can maintain state | Stateless |
| **Inline expansion** | Compiler can inline | Harder to inline |
| **Template compatibility** | Works well | Limited |
| **Multiple instances** | Different states per instance | Same function always |

---

### 86. What is stream operator overloading?

**Stream operator overloading** allows you to define how objects are **input from** (`>>`) and **output to** (`<<`) streams. This is essential for making classes work with `std::cout`, `std::cin`, file streams, and string streams.

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <vector>
#include <iomanip>

// ==================== Basic Stream Operator Overloading ====================

class Point {
private:
    double x, y;
    
public:
    Point(double x = 0, double y = 0) : x(x), y(y) {}
    
    // Getters for serialization
    double getX() const { return x; }
    double getY() const { return y; }
    
    // Friend functions for stream operators
    friend std::ostream& operator<<(std::ostream& os, const Point& p);
    friend std::istream& operator>>(std::istream& is, Point& p);
};

// Output operator
std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << "(" << p.x << ", " << p.y << ")";
    return os;
}

// Input operator
std::istream& operator>>(std::istream& is, Point& p) {
    char comma, paren;
    // Expected format: (x, y)
    is >> paren >> p.x >> comma >> p.y >> paren;
    if (comma != ',') {
        is.setstate(std::ios::failbit);
    }
    return is;
}

// ==================== Complex Example with Formatting ====================

class Student {
private:
    std::string name;
    int id;
    std::vector<double> grades;
    
public:
    Student(const std::string& n = "", int i = 0) : name(n), id(i) {}
    
    void addGrade(double grade) {
        grades.push_back(grade);
    }
    
    double getAverage() const {
        if (grades.empty()) return 0;
        double sum = 0;
        for (double g : grades) sum += g;
        return sum / grades.size();
    }
    
    // Friend declaration for stream operators
    friend std::ostream& operator<<(std::ostream& os, const Student& s);
    friend std::istream& operator>>(std::istream& is, Student& s);
};

// Formatted output
std::ostream& operator<<(std::ostream& os, const Student& s) {
    os << "Student: " << s.name << " (ID: " << s.id << ")\n";
    os << "Grades: ";
    for (double g : s.grades) {
        os << std::fixed << std::setprecision(1) << g << " ";
    }
    os << "\nAverage: " << std::fixed << std::setprecision(2) 
       << s.getAverage();
    return os;
}

// Formatted input
std::istream& operator>>(std::istream& is, Student& s) {
    std::cout << "Enter name: ";
    is >> s.name;
    std::cout << "Enter ID: ";
    is >> s.id;
    
    std::cout << "Enter grades (enter -1 to stop): ";
    s.grades.clear();
    double grade;
    while (is >> grade && grade >= 0) {
        s.grades.push_back(grade);
    }
    is.clear();  // Clear fail state from -1
    return is;
}

// ==================== XML-like Formatting ====================

class Book {
private:
    std::string title;
    std::string author;
    int year;
    double price;
    
public:
    Book(const std::string& t = "", const std::string& a = "", 
         int y = 0, double p = 0) : title(t), author(a), year(y), price(p) {}
    
    friend std::ostream& operator<<(std::ostream& os, const Book& b);
    friend std::istream& operator>>(std::istream& is, Book& b);
};

// XML-style output
std::ostream& operator<<(std::ostream& os, const Book& b) {
    os << "<book>\n"
       << "  <title>" << b.title << "</title>\n"
       << "  <author>" << b.author << "</author>\n"
       << "  <year>" << b.year << "</year>\n"
       << "  <price>" << std::fixed << std::setprecision(2) 
       << b.price << "</price>\n"
       << "</book>";
    return os;
}

// XML-style input
std::istream& operator>>(std::istream& is, Book& b) {
    std::string line;
    // Very simplified parsing - in reality would use proper XML parser
    while (std::getline(is, line)) {
        if (line.find("<title>") != std::string::npos) {
            b.title = line.substr(line.find(">") + 1, 
                                  line.find("</") - line.find(">") - 1);
        } else if (line.find("<author>") != std::string::npos) {
            b.author = line.substr(line.find(">") + 1, 
                                   line.find("</") - line.find(">") - 1);
        } else if (line.find("<year>") != std::string::npos) {
            std::string yearStr = line.substr(line.find(">") + 1, 
                                              line.find("</") - line.find(">") - 1);
            b.year = std::stoi(yearStr);
        } else if (line.find("<price>") != std::string::npos) {
            std::string priceStr = line.substr(line.find(">") + 1, 
                                               line.find("</") - line.find(">") - 1);
            b.price = std::stod(priceStr);
        }
    }
    return is;
}

// ==================== File Stream Example ====================

void fileStreamExample() {
    std::cout << "\n=== File Stream Example ===" << std::endl;
    
    // Write to file
    std::ofstream outFile("points.txt");
    std::vector<Point> points = {Point(1,2), Point(3,4), Point(5,6)};
    
    for (const auto& p : points) {
        outFile << p << std::endl;
    }
    outFile.close();
    
    // Read from file
    std::ifstream inFile("points.txt");
    Point p;
    while (inFile >> p) {
        std::cout << "Read from file: " << p << std::endl;
    }
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Stream Operators ===" << std::endl;
    
    Point p1(3.14, 2.718);
    Point p2;
    
    std::cout << "p1 = " << p1 << std::endl;
    
    std::cout << "Enter a point (format: (x, y)): ";
    std::cin >> p2;
    std::cout << "You entered: " << p2 << std::endl;
    
    std::cout << "\n=== Student Example ===" << std::endl;
    
    Student student("Alice", 12345);
    student.addGrade(85.5);
    student.addGrade(92.0);
    student.addGrade(78.5);
    
    std::cout << student << std::endl;
    
    std::cout << "\n=== Book Example (XML format) ===" << std::endl;
    
    Book book("The C++ Programming Language", "Bjarne Stroustrup", 2013, 64.99);
    std::cout << book << std::endl;
    
    // String stream example
    std::cout << "\n=== String Stream Example ===" << std::endl;
    
    std::stringstream ss;
    ss << p1 << " " << student << " " << book;
    
    std::cout << "String stream contains: " << ss.str() << std::endl;
    
    fileStreamExample();
    
    std::cout << "\n=== Important Notes ===" << std::endl;
    std::cout << "1. Return reference to stream for chaining" << std::endl;
    std::cout << "2. Usually implemented as friend functions" << std::endl;
    std::cout << "3. Can't be member functions (first parameter would be this)" << std::endl;
    std::cout << "4. Respect stream formatting flags (setprecision, width, etc.)" << std::endl;
    
    return 0;
}
```

**Key Points for Stream Operators:**

| Aspect | Output Operator `<<` | Input Operator `>>` |
| :--- | :--- | :--- |
| **Signature** | `std::ostream& operator<<(std::ostream&, const T&)` | `std::istream& operator>>(std::istream&, T&)` |
| **Return** | Reference to stream (for chaining) | Reference to stream |
| **Const correctness** | Object is const | Object is non-const |
| **Friendship** | Usually needed | Usually needed |
| **Error handling** | Rarely fails | Set failbit on error |

---

### 87. Rules for operator overloading?

Operator overloading in C++ follows specific **rules and guidelines** to maintain code clarity and prevent unexpected behavior.

```cpp
#include <iostream>
#include <memory>

// ==================== Rule 1: Cannot Create New Operators ====================

class Demo {
private:
    int value;
    
public:
    Demo(int v = 0) : value(v) {}
    
    // Can't create new operators like ** or <>
    // Can only overload existing C++ operators
};

// ==================== Rule 2: Arity Cannot Be Changed ====================

class ArityDemo {
private:
    int data;
    
public:
    ArityDemo(int d = 0) : data(d) {}
    
    // Unary operators must remain unary
    ArityDemo& operator++() { ++data; return *this; }  // OK: unary prefix
    ArityDemo operator++(int) { ArityDemo tmp = *this; ++data; return tmp; } // OK: postfix
    
    // Binary operators must remain binary
    ArityDemo operator+(const ArityDemo& other) const {  // OK: binary
        return ArityDemo(data + other.data);
    }
    
    // Error: Would change arity
    // ArityDemo operator+(const ArityDemo& a, const ArityDemo& b, const ArityDemo& c) const;
};

// ==================== Rule 3: Precedence and Associativity Preserved ====================

void precedenceExample() {
    std::cout << "\n=== Rule 3: Precedence Preserved ===" << std::endl;
    
    Demo a(5), b(3), c(2);
    
    // Operator precedence remains the same
    // Multiplication happens before addition
    // Demo result = a + b * c;  // Same precedence rules apply
}

// ==================== Rule 4: Member vs Non-member ====================

class MemberRules {
private:
    int value;
    
public:
    MemberRules(int v = 0) : value(v) {}
    
    // Member operators (this is first operand)
    MemberRules& operator+=(const MemberRules& other) {
        value += other.value;
        return *this;
    }
    
    // Non-member operators (as friend when needed)
    friend MemberRules operator+(const MemberRules& a, const MemberRules& b) {
        return MemberRules(a.value + b.value);
    }
    
    // Some operators MUST be members:
    // =, [], (), ->
    MemberRules& operator=(const MemberRules& other) = default;  // Must be member
    
    int& operator[](int index) { return value; }  // Must be member
    
    void operator()(const std::string& msg) {  // Must be member
        std::cout << msg << ": " << value << std::endl;
    }
    
    MemberRules* operator->() { return this; }  // Must be member
};

// ==================== Rule 5: Short-circuit Operators ====================

class ShortCircuit {
private:
    bool flag;
    
public:
    ShortCircuit(bool f) : flag(f) {}
    
    // && and || operators lose short-circuit evaluation when overloaded
    ShortCircuit operator&&(const ShortCircuit& other) const {
        std::cout << "&& overloaded (no short-circuit!)" << std::endl;
        return ShortCircuit(flag && other.flag);
    }
    
    ShortCircuit operator||(const ShortCircuit& other) const {
        std::cout << "|| overloaded (no short-circuit!)" << std::endl;
        return ShortCircuit(flag || other.flag);
    }
    
    operator bool() const { return flag; }
};

// ==================== Rule 6: Return Type Guidelines ====================

class ReturnTypeRules {
private:
    int value;
    
public:
    ReturnTypeRules(int v = 0) : value(v) {}
    
    // Arithmetic operators: return by value
    ReturnTypeRules operator+(const ReturnTypeRules& other) const {
        return ReturnTypeRules(value + other.value);
    }
    
    // Assignment operators: return by reference
    ReturnTypeRules& operator+=(const ReturnTypeRules& other) {
        value += other.value;
        return *this;
    }
    
    // Comparison operators: return bool
    bool operator==(const ReturnTypeRules& other) const {
        return value == other.value;
    }
    
    // Stream operators: return stream reference
    friend std::ostream& operator<<(std::ostream& os, const ReturnTypeRules& r) {
        os << r.value;
        return os;
    }
    
    // Subscript: return reference
    int& operator[](int) { return value; }
    
    // Function call: can return anything
    int operator()() const { return value; }
};

// ==================== Rule 7: Consistency ====================

class Consistent {
private:
    int value;
    
public:
    Consistent(int v = 0) : value(v) {}
    
    // If you overload +, also overload +=
    Consistent operator+(const Consistent& other) const {
        return Consistent(value + other.value);
    }
    
    Consistent& operator+=(const Consistent& other) {
        value += other.value;
        return *this;
    }
    
    // If you overload ==, also overload !=
    bool operator==(const Consistent& other) const {
        return value == other.value;
    }
    
    bool operator!=(const Consistent& other) const {
        return !(*this == other);
    }
    
    // If you overload <, also overload >, <=, >=
    bool operator<(const Consistent& other) const {
        return value < other.value;
    }
    
    bool operator>(const Consistent& other) const {
        return other < *this;
    }
    
    bool operator<=(const Consistent& other) const {
        return !(other < *this);
    }
    
    bool operator>=(const Consistent& other) const {
        return !(*this < other);
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Operator Overloading Rules ===" << std::endl;
    
    std::cout << "\n1. Cannot create new operators" << std::endl;
    std::cout << "2. Cannot change arity (number of operands)" << std::endl;
    std::cout << "3. Cannot change precedence or associativity" << std::endl;
    std::cout << "4. Some operators must be members (=, [], (), ->)" << std::endl;
    
    std::cout << "\n=== Rule 5: Short-circuit Loss ===" << std::endl;
    
    ShortCircuit a(true);
    ShortCircuit b(false);
    
    // Short-circuit would normally skip evaluating second operand
    auto result1 = a && b;  // Both operands evaluated!
    auto result2 = a || b;  // Both operands evaluated!
    
    std::cout << "\n=== Rule 6: Return Type Guidelines ===" << std::endl;
    
    ReturnTypeRules r1(10), r2(20);
    auto r3 = r1 + r2;           // Returns by value
    auto r4 = (r1 += r2);        // Returns reference
    bool eq = (r1 == r2);         // Returns bool
    
    std::cout << "r1 + r2 = " << r3 << std::endl;
    std::cout << "r1 == r2? " << eq << std::endl;
    
    std::cout << "\n=== Rule 7: Consistency ===" << std::endl;
    
    Consistent c1(5), c2(10);
    
    std::cout << "c1 < c2: " << (c1 < c2) << std::endl;
    std::cout << "c1 > c2: " << (c1 > c2) << std::endl;
    std::cout << "c1 <= c2: " << (c1 <= c2) << std::endl;
    std::cout << "c1 >= c2: " << (c1 >= c2) << std::endl;
    
    return 0;
}
```

**Summary of Rules:**

| Rule | Description |
| :--- | :--- |
| **No new operators** | Can only overload existing C++ operators |
| **Arity fixed** | Number of operands cannot change |
| **Precedence preserved** | Operator precedence remains the same |
| **Associativity fixed** | Left-to-right or right-to-left unchanged |
| **Some must be members** | `=`, `[]`, `()`, `->` must be member functions |
| **Short-circuit lost** | `&&` and `||` lose short-circuit evaluation |
| **Consistency** | Overload related operators together |
| **Return types** | Follow conventions (reference for assignment, value for arithmetic) |

---

### 88. What is friend operator function?

A **friend operator function** is a non-member operator overload that has access to **private and protected members** of a class. This is commonly used for binary operators where the left operand is not of the class type.

```cpp
#include <iostream>
#include <cmath>

// ==================== Basic Friend Operator ====================

class Complex {
private:
    double real;
    double imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // Member operator (left operand must be Complex)
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    // Friend operator allows non-Complex left operand
    friend Complex operator+(double lhs, const Complex& rhs) {
        return Complex(lhs + rhs.real, rhs.imag);
    }
    
    // Friend for stream output
    friend std::ostream& operator<<(std::ostream& os, const Complex& c) {
        os << c.real;
        if (c.imag >= 0) os << " + " << c.imag << "i";
        else os << " - " << -c.imag << "i";
        return os;
    }
    
    // Friend for multiplication by scalar
    friend Complex operator*(double lhs, const Complex& rhs) {
        return Complex(lhs * rhs.real, lhs * rhs.imag);
    }
    
    // Friend for equality with double
    friend bool operator==(double lhs, const Complex& rhs) {
        return (lhs == rhs.real) && (rhs.imag == 0);
    }
};

// ==================== Why Friend Operators are Needed ====================

class Integer {
private:
    int value;
    
public:
    Integer(int v = 0) : value(v) {}
    
    // Member operator: works when Integer is left operand
    Integer operator+(const Integer& other) const {
        return Integer(value + other.value);
    }
    
    // This wouldn't work: 5 + intObj
    // Because 5 is not an Integer
    
    // Friend operator allows int + Integer
    friend Integer operator+(int lhs, const Integer& rhs) {
        return Integer(lhs + rhs.value);
    }
    
    // Also need friend for int == Integer
    friend bool operator==(int lhs, const Integer& rhs) {
        return lhs == rhs.value;
    }
    
    int getValue() const { return value; }
};

// ==================== Symmetric Operators Need Friends ====================

class Vector2D {
private:
    double x, y;
    
public:
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}
    
    // Dot product (symmetric operation)
    friend double dot(const Vector2D& a, const Vector2D& b) {
        return a.x * b.x + a.y * b.y;
    }
    
    // Equality (should be symmetric)
    friend bool operator==(const Vector2D& a, const Vector2D& b) {
        return (a.x == b.x) && (a.y == b.y);
    }
    
    // Multiplication by scalar (commutative)
    friend Vector2D operator*(const Vector2D& v, double s) {
        return Vector2D(v.x * s, v.y * s);
    }
    
    friend Vector2D operator*(double s, const Vector2D& v) {
        return Vector2D(v.x * s, v.y * s);  // Same implementation
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        os << "(" << v.x << ", " << v.y << ")";
        return os;
    }
};

// ==================== Matrix Example with Multiple Friends ====================

class Matrix {
private:
    double data[2][2];
    
public:
    Matrix(double a11 = 0, double a12 = 0, double a21 = 0, double a22 = 0) {
        data[0][0] = a11; data[0][1] = a12;
        data[1][0] = a21; data[1][1] = a22;
    }
    
    // Matrix * Vector
    friend class Vector;  // Forward declaration needed
    
    // Matrix * scalar
    friend Matrix operator*(const Matrix& m, double s) {
        Matrix result;
        for (int i = 0; i < 2; ++i)
            for (int j = 0; j < 2; ++j)
                result.data[i][j] = m.data[i][j] * s;
        return result;
    }
    
    // scalar * Matrix (commutative)
    friend Matrix operator*(double s, const Matrix& m) {
        return m * s;  // Reuse implementation
    }
    
    // Matrix multiplication
    friend Matrix operator*(const Matrix& a, const Matrix& b) {
        Matrix result;
        for (int i = 0; i < 2; ++i)
            for (int j = 0; j < 2; ++j)
                for (int k = 0; k < 2; ++k)
                    result.data[i][j] += a.data[i][k] * b.data[k][j];
        return result;
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Matrix& m) {
        os << "[ " << m.data[0][0] << " " << m.data[0][1] << " ; "
           << m.data[1][0] << " " << m.data[1][1] << " ]";
        return os;
    }
};

class Vector {
private:
    double comp[2];
    
public:
    Vector(double x = 0, double y = 0) {
        comp[0] = x; comp[1] = y;
    }
    
    // Matrix * Vector (needs access to both)
    friend Vector operator*(const Matrix& m, const Vector& v) {
        Vector result;
        result.comp[0] = m.data[0][0] * v.comp[0] + m.data[0][1] * v.comp[1];
        result.comp[1] = m.data[1][0] * v.comp[0] + m.data[1][1] * v.comp[1];
        return result;
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Vector& v) {
        os << "[" << v.comp[0] << ", " << v.comp[1] << "]";
        return os;
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Friend Operator Functions ===" << std::endl;
    
    Complex c1(3, 4);
    std::cout << "c1 = " << c1 << std::endl;
    
    // Member operator
    Complex c2 = c1 + Complex(1, 2);
    std::cout << "c1 + Complex(1,2) = " << c2 << std::endl;
    
    // Friend operator (int + Complex)
    Complex c3 = 5 + c1;
    std::cout << "5 + c1 = " << c3 << std::endl;
    
    // Friend operator (double * Complex)
    Complex c4 = 2.5 * c1;
    std::cout << "2.5 * c1 = " << c4 << std::endl;
    
    std::cout << "\n=== Integer Example ===" << std::endl;
    
    Integer i1(10);
    Integer i2 = 5 + i1;  // Works due to friend
    std::cout << "5 + Integer(10) = " << i2.getValue() << std::endl;
    
    bool eq = (5 == i1);  // Works due to friend
    std::cout << "5 == Integer(10)? " << eq << std::endl;
    
    std::cout << "\n=== Vector Example ===" << std::endl;
    
    Vector2D v1(3, 4);
    Vector2D v2 = 2 * v1;  // Uses friend operator
    Vector2D v3 = v1 * 3;   // Uses friend operator
    
    std::cout << "v1 = " << v1 << std::endl;
    std::cout << "2 * v1 = " << v2 << std::endl;
    std::cout << "v1 * 3 = " << v3 << std::endl;
    
    double d = dot(v1, v2);  // Friend function
    std::cout << "Dot product: " << d << std::endl;
    
    std::cout << "\n=== Matrix-Vector Example ===" << std::endl;
    
    Matrix m(1, 2, 3, 4);
    Vector vec(5, 6);
    
    std::cout << "Matrix m = " << m << std::endl;
    std::cout << "Vector v = " << vec << std::endl;
    
    Vector result = m * vec;  // Uses friend operator
    std::cout << "m * v = " << result << std::endl;
    
    std::cout << "\n=== When to Use Friend Operators ===" << std::endl;
    std::cout << "1. When left operand is not of class type (e.g., int + MyClass)\n";
    std::cout << "2. For symmetric operations (equality, addition)\n";
    std::cout << "3. For stream operators (<< and >>)\n";
    std::cout << "4. When operator needs access to private members\n";
    
    return 0;
}
```

**When to Use Friend Operator Functions:**

| Scenario | Example | Why Friend? |
| :--- | :--- | :--- |
| **Left operand not class type** | `5 + Complex` | Can't be member (5 has no member function) |
| **Symmetric operators** | `==`, `+` (commutative) | Both sides treated equally |
| **Stream operators** | `<<`, `>>` | Left operand is stream, not class |
| **Binary operators needing private access** | Matrix multiplication | Need access to both operands' internals |