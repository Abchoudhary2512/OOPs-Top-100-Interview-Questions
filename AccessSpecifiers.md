### 41. What are public, private, and protected access specifiers?

Access specifiers define the **visibility and accessibility** of class members (data members and member functions). They control who can access what parts of a class.

**1. `public`:**
- Members declared as `public` are **accessible from anywhere** in the program
- Accessible inside the class, derived classes, and by objects/outside code

**2. `private`:**
- Members declared as `private` are **only accessible within the same class**
- Not accessible in derived classes or outside the class
- **Default** access specifier for class members

**3. `protected`:**
- Members declared as `protected` are accessible **within the same class** and **in derived classes**
- Not accessible outside the class hierarchy (by non-member functions or objects)

```cpp
class MyClass {
private:
    int privateVar;      // Accessible only inside MyClass
    
protected:
    int protectedVar;    // Accessible in MyClass and derived classes
    
public:
    int publicVar;       // Accessible everywhere
    
    void accessTest() {
        privateVar = 10;   // OK: inside same class
        protectedVar = 20; // OK: inside same class
        publicVar = 30;    // OK: inside same class
    }
};

class Derived : public MyClass {
public:
    void derivedAccess() {
        // privateVar = 10;  // ERROR: not accessible in derived class
        protectedVar = 20;   // OK: accessible in derived class
        publicVar = 30;      // OK: accessible in derived class
    }
};

int main() {
    MyClass obj;
    // obj.privateVar = 10;   // ERROR: private
    // obj.protectedVar = 20; // ERROR: protected
    obj.publicVar = 30;        // OK: public
    return 0;
}
```

---

### 42. What's the difference between private and protected?

The key difference lies in **accessibility from derived classes**:

| Feature | `private` | `protected` |
| :--- | :--- | :--- |
| **Access within same class** | ✅ Yes | ✅ Yes |
| **Access in derived classes** | ❌ No | ✅ Yes |
| **Access by objects/outside** | ❌ No | ❌ No |
| **Default in class** | ✅ Yes (default) | ❌ No |
| **Level of encapsulation** | Highest | Medium |

```cpp
class Base {
private:
    int privateMember = 1;
protected:
    int protectedMember = 2;
public:
    int publicMember = 3;
};

class Derived : public Base {
public:
    void demonstrate() {
        // privateMember = 10;  // ERROR: cannot access private member
        protectedMember = 20;    // OK: can access protected member
        publicMember = 30;       // OK: can access public member
    }
};

int main() {
    Base base;
    // base.privateMember = 1;   // ERROR
    // base.protectedMember = 2; // ERROR
    base.publicMember = 3;        // OK
    
    Derived derived;
    // derived.protectedMember = 2; // ERROR: not accessible via object
    derived.publicMember = 3;       // OK
    
    return 0;
}
```

**Rule of thumb:**
- Use `private` for implementation details that shouldn't be exposed to anyone
- Use `protected` for things that derived classes might need but still hide from outside

---

### 43. Can private members be accessed by derived classes?

**No, private members cannot be accessed directly by derived classes.**

This is a fundamental rule of encapsulation in C++. Private members are **only accessible within the class that defines them**, not in any derived classes, regardless of the inheritance type.

However, derived classes can access private members **indirectly** through public or protected member functions of the base class.

```cpp
class Base {
private:
    int secret = 42;  // Private member
    
protected:
    int getSecret() const {  // Protected getter
        return secret;
    }
    
public:
    void showSecret() const {  // Public getter
        std::cout << "Secret: " << secret << std::endl;
    }
};

class Derived : public Base {
public:
    void tryAccess() {
        // int x = secret;      // ERROR: cannot access private member directly
        
        int x = getSecret();     // OK: through protected member function
        std::cout << "Accessed through getter: " << x << std::endl;
        
        showSecret();            // OK: through public member function
    }
};

int main() {
    Derived d;
    d.tryAccess();  // Works through the getter methods
    return 0;
}
```

**Why?** This design enforces encapsulation. The base class maintains control over how its private data is accessed or modified, even by derived classes.

---

### 44. What is friend function and friend class?

**Friend functions** and **friend classes** are mechanisms in C++ that allow **non-member functions or other classes to access private and protected members** of a class.

#### Friend Function:
A function that is not a member of the class but is granted access to all its private and protected members.

```cpp
#include <iostream>

class BankAccount {
private:
    double balance;
    
public:
    BankAccount(double b) : balance(b) {}
    
    // Declare an external function as a friend
    friend void displayBalance(const BankAccount& account);
};

// Friend function definition - can access private members
void displayBalance(const BankAccount& account) {
    // Direct access to private member
    std::cout << "Balance: $" << account.balance << std::endl;
}

int main() {
    BankAccount myAccount(1000.50);
    displayBalance(myAccount);  // Friend function called
    return 0;
}
```

#### Friend Class:
A class where all member functions have access to the private and protected members of another class.

```cpp
#include <iostream>

class Engine;  // Forward declaration

class Car {
private:
    int speed;
    
public:
    Car() : speed(0) {}
    
    // Declare Engine class as a friend
    friend class Engine;
    
    void showSpeed() {
        std::cout << "Car speed: " << speed << " km/h" << std::endl;
    }
};

class Engine {
public:
    void accelerate(Car& car, int amount) {
        // Accessing Car's private member
        car.speed += amount;
        std::cout << "Engine accelerating by " << amount << " km/h" << std::endl;
    }
};

int main() {
    Car myCar;
    Engine engine;
    
    myCar.showSpeed();    // Speed: 0
    engine.accelerate(myCar, 30);
    myCar.showSpeed();    // Speed: 30
    return 0;
}
```

**Important notes:**
- Friendship is **not reciprocal** (if A is friend of B, B is not automatically friend of A)
- Friendship is **not transitive** (if A is friend of B and B is friend of C, A is not friend of C)
- Friendship is **granted, not taken** (the class decides who its friends are)
- Use friendship sparingly - it breaks encapsulation

---

### 45. What is data hiding?

**Data hiding** is a fundamental principle of OOP that involves **restricting direct access to the internal data of an object**. It's a key aspect of **encapsulation** where the internal state of an object is kept private and can only be modified through well-defined public interfaces.

**Key aspects:**
1. **Private/Protected members**: Data is declared as private or protected
2. **Controlled access**: Public member functions (getters/setters) provide controlled access
3. **Implementation independence**: Internal implementation can change without affecting external code

```cpp
#include <iostream>
#include <string>

class Student {
private:
    // Hidden data (implementation details)
    std::string name;
    int age;
    double gpa;
    std::string studentId;
    
    // Validation logic hidden from outside
    bool isValidAge(int a) {
        return a >= 5 && a <= 100;
    }
    
    bool isValidGPA(double g) {
        return g >= 0.0 && g <= 4.0;
    }
    
public:
    // Public interface - controlled access to hidden data
    Student(std::string n, int a, std::string id) 
        : name(n), studentId(id) {
        setAge(a);  // Use setter for validation
        gpa = 0.0;
    }
    
    // Setter with validation
    void setAge(int a) {
        if (isValidAge(a)) {
            age = a;
        } else {
            std::cout << "Invalid age!" << std::endl;
        }
    }
    
    // Getter provides read-only access
    int getAge() const {
        return age;
    }
    
    void setGPA(double g) {
        if (isValidGPA(g)) {
            gpa = g;
        }
    }
    
    double getGPA() const {
        return gpa;
    }
    
    void displayInfo() const {
        std::cout << "Name: " << name << ", ID: " << studentId 
                  << ", Age: " << age << ", GPA: " << gpa << std::endl;
    }
};

int main() {
    Student s("Alice", 20, "S12345");
    
    // s.age = 25;        // ERROR: private
    // s.gpa = 3.5;       // ERROR: private
    
    s.setAge(25);         // OK: using public interface
    s.setGPA(3.8);        // OK: using public interface
    
    s.displayInfo();      // Access through public method
    
    int age = s.getAge(); // OK: controlled access
    return 0;
}
```

**Benefits of data hiding:**
- **Protection**: Prevents accidental or unauthorized modification of data
- **Validation**: Data can be validated before changes
- **Flexibility**: Internal implementation can change without breaking external code
- **Maintainability**: Easier to debug and maintain

---

### 46. What are inline functions?

**Inline functions** are functions that are **expanded in place at the point of call** rather than being called through the normal function call mechanism. The `inline` keyword is a **request** to the compiler to replace the function call with the actual function code.

**Purpose:** To eliminate function call overhead for small, frequently used functions.

```cpp
#include <iostream>

// Inline function definition
inline int square(int x) {
    return x * x;
}

class MathUtils {
public:
    // Inline member function (defined inside class - automatically inline)
    int add(int a, int b) {
        return a + b;
    }
    
    // Inline function defined outside class
    inline int multiply(int a, int b);
};

// Definition of inline function outside class
inline int MathUtils::multiply(int a, int b) {
    return a * b;
}

int main() {
    MathUtils math;
    
    // The compiler may replace these calls with the actual code
    int result1 = square(5);           // May become: int result1 = 5 * 5;
    int result2 = math.add(10, 20);    // May become: int result2 = 10 + 20;
    int result3 = math.multiply(4, 6); // May become: int result3 = 4 * 6;
    
    std::cout << "Square: " << result1 << std::endl;
    std::cout << "Add: " << result2 << std::endl;
    std::cout << "Multiply: " << result3 << std::endl;
    
    return 0;
}
```

**Characteristics of inline functions:**

| Aspect | Description |
| :--- | :--- |
| **Performance** | Eliminates function call overhead (push/pop stack) |
| **Code size** | Can increase binary size if used excessively (code bloat) |
| **Definition** | Should be in header files (need definition at compile time) |
| **Compiler discretion** | Compiler may ignore inline request (e.g., for large functions) |
| **Best for** | Small, frequently called functions (getters, setters) |

**Modern C++ note:** Modern compilers are good at optimizing and may inline functions even without the `inline` keyword.

---

### 47. What is static member in class?

**Static members** (both data members and member functions) belong to the **class itself** rather than to any specific object. All objects of the class share the **same static member**.

#### Static Data Members:
- Only one copy exists for the entire class
- Must be defined outside the class (usually in a .cpp file)
- Exist even when no objects are created

#### Static Member Functions:
- Can be called without creating an object
- Can only access static members (no `this` pointer)
- Cannot be `const` or `virtual`

```cpp
#include <iostream>

class Counter {
private:
    static int totalCount;      // Static data member (declaration)
    int instanceId;
    
public:
    Counter() {
        totalCount++;           // Increment when object created
        instanceId = totalCount;
    }
    
    ~Counter() {
        totalCount--;           // Decrement when object destroyed
    }
    
    static int getTotalCount() {  // Static member function
        return totalCount;
    }
    
    void display() const {
        std::cout << "Instance " << instanceId 
                  << " of " << totalCount << " total" << std::endl;
    }
    
    // Static function can be called without object
    static void showClassInfo() {
        std::cout << "Counter class - Total objects: " << totalCount << std::endl;
        // std::cout << instanceId; // ERROR: cannot access non-static member
    }
};

// Definition of static member (outside class)
int Counter::totalCount = 0;

int main() {
    // Call static function without object
    std::cout << "Initial count: " << Counter::getTotalCount() << std::endl;
    
    Counter obj1;
    Counter obj2;
    Counter obj3;
    
    obj1.display();        // Instance 1 of 3 total
    obj2.display();        // Instance 2 of 3 total
    obj3.display();        // Instance 3 of 3 total
    
    std::cout << "Final count: " << Counter::getTotalCount() << std::endl;
    
    Counter::showClassInfo();  // Call static member function
    
    return 0;
}
```

**Common uses of static members:**
- Keeping track of object count
- Implementing singleton pattern
- Sharing configuration or constant data across objects
- Utility functions that don't need object state

---

### 48. What is mutable keyword?

The **`mutable`** keyword allows a member of a **const object** or a **const member function** to be modified. It overrides the const-ness of an object for that specific member.

**Why needed?** Sometimes you have data members that are not logically part of the object's state (like caches, reference counts, mutexes) that need to be modified even in const contexts.

```cpp
#include <iostream>
#include <string>

class DataProcessor {
private:
    int data;
    mutable int cacheResult;      // Cache that can be modified even in const
    mutable bool cacheValid;      // Cache flag
    
public:
    DataProcessor(int d) : data(d), cacheResult(0), cacheValid(false) {}
    
    // Const member function - promises not to modify object logically
    int expensiveComputation() const {
        // Even though this is const, we can modify mutable members
        if (!cacheValid) {
            std::cout << "Computing and caching result..." << std::endl;
            cacheResult = data * data * data;  // Modify mutable member
            cacheValid = true;                   // Modify mutable member
        } else {
            std::cout << "Using cached result..." << std::endl;
        }
        return cacheResult;
    }
    
    // This can still modify non-mutable members
    void setData(int d) {
        data = d;
        cacheValid = false;  // Invalidate cache
    }
    
    int getData() const {
        return data;  // OK: just reading
    }
};

int main() {
    const DataProcessor proc(5);  // const object
    
    // proc.setData(10);  // ERROR: can't call non-const function on const object
    
    std::cout << "Result: " << proc.expensiveComputation() << std::endl;
    std::cout << "Result: " << proc.expensiveComputation() << std::endl;
    
    // Even though proc is const, the mutable cache was updated!
    return 0;
}
```

**Another example - Reference counting:**
```cpp
class SharedData {
private:
    std::string name;
    mutable int refCount;  // Reference count needs to be modified even in const
    
public:
    SharedData(const std::string& n) : name(n), refCount(0) {}
    
    void addReference() const {  // const function modifies mutable member
        ++refCount;
    }
    
    void releaseReference() const {  // const function modifies mutable member
        --refCount;
    }
    
    int getRefCount() const {
        return refCount;
    }
};
```

---

### 49. Explain "final" keyword (since C++11)

The **`final`** keyword in C++11 is used to **prevent further inheritance or overriding**. It can be applied to:

1. **Virtual functions**: Prevents the function from being overridden in derived classes
2. **Classes**: Prevents the class from being used as a base class (cannot be inherited)

```cpp
#include <iostream>

// Example 1: Final class (cannot be inherited)
class BaseClass final {
public:
    void show() {
        std::cout << "BaseClass" << std::endl;
    }
};

// ERROR: Cannot inherit from final class
// class DerivedClass : public BaseClass { };  // Compilation error!

// Example 2: Class with final virtual function
class Shape {
public:
    virtual void draw() {
        std::cout << "Drawing Shape" << std::endl;
    }
    
    virtual void resize() final {  // This function cannot be overridden
        std::cout << "Resizing Shape" << std::endl;
    }
};

class Circle : public Shape {
public:
    void draw() override {  // OK: draw is not final
        std::cout << "Drawing Circle" << std::endl;
    }
    
    // ERROR: Cannot override final function
    // void resize() override {  // Compilation error!
    //     std::cout << "Resizing Circle" << std::endl;
    // }
};

// Example 3: Combining override and final
class AdvancedShape : public Shape {
public:
    void draw() override final {  // Override and make it final
        std::cout << "Drawing AdvancedShape" << std::endl;
    }
};

class MoreAdvanced : public AdvancedShape {
public:
    // ERROR: Cannot override final function from AdvancedShape
    // void draw() override {  // Compilation error!
    //     std::cout << "Drawing MoreAdvanced" << std::endl;
    // }
};

int main() {
    Circle c;
    c.draw();     // Output: Drawing Circle
    c.resize();   // Output: Resizing Shape (cannot be changed)
    
    AdvancedShape adv;
    adv.draw();   // Output: Drawing AdvancedShape
    
    return 0;
}
```

**When to use `final`:**
- When you've designed a class to be used as-is and shouldn't be extended
- When a specific function's behavior should be fixed and not changed in derived classes
- As a design decision to enforce architectural boundaries

---

### 50. How does access control help security?

Access control in C++ (public, private, protected) provides **security at the design/development level** rather than runtime security. It helps create **robust, maintainable, and bug-resistant code**.

#### Key security benefits:

**1. Prevents Accidental Modification:**
```cpp
class BankAccount {
private:
    double balance;  // Hidden from direct access
    
public:
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
};

int main() {
    BankAccount account;
    // account.balance = 1000000;  // ERROR: private - can't accidentally set
    
    account.deposit(500);    // OK: controlled access with validation
    account.withdraw(100);   // OK: controlled access with validation
    return 0;
}
```

**2. Enforces Valid State (Invariants):**
```cpp
class Person {
private:
    int age;  // Hidden to enforce validation
    
public:
    void setAge(int newAge) {
        if (newAge >= 0 && newAge <= 150) {  // Validation
            age = newAge;
        } else {
            // Handle error - prevents invalid state
        }
    }
};
```

**3. Implementation Hiding (Security Through Obscurity is NOT security):**
```cpp
class Encryption {
private:
    // Implementation details hidden - not for security but for abstraction
    std::string key;
    std::vector<uint8_t> sBox;
    
    void generateSBox() { /* Complex implementation */ }
    
public:
    std::vector<uint8_t> encrypt(const std::vector<uint8_t>& data) {
        // Public interface - users don't need to know internals
        generateSBox();
        // ... encryption logic
    }
};
```

**4. Controlled Interface (Principle of Least Privilege):**
```cpp
class DatabaseConnection {
private:
    std::string connectionString;
    void* connectionHandle;  // Low-level handle
    
    // Internal helper - not exposed
    void reconnect() { /* ... */ }
    
public:
    // Only expose what's necessary
    bool executeQuery(const std::string& query) { /* ... */ }
    void close() { /* ... */ }
    
protected:
    // For derived classes that might need more access
    void setConnectionTimeout(int seconds) { /* ... */ }
};
```

**5. Encapsulation of Sensitive Data:**
```cpp
class User {
private:
    std::string passwordHash;  // Never expose directly
    std::string salt;
    
public:
    bool authenticate(const std::string& password) const {
        // Only expose authentication result, not the hash
        std::string hash = calculateHash(password, salt);
        return hash == passwordHash;
    }
};
```

#### Summary: How Access Control Enhances Security

| Access Level | Security Benefit |
| :--- | :--- |
| **private** | Prevents external code from tampering with internal state |
| **protected** | Limits access to trusted derived classes only |
| **public** | Provides a well-defined, controlled interface |

**Important Note:** Access control is about **program correctness and robustness**, not about preventing malicious attacks at runtime. A determined attacker with memory access can still bypass these protections. It's about creating reliable, maintainable code that behaves as intended.