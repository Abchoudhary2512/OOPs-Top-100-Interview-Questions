### 51. What is RAII? (Resource Acquisition Is Initialization)

**RAII (Resource Acquisition Is Initialization)** is a core C++ programming technique where **resource management is tied to object lifetime**. Resources are acquired during object construction and released during object destruction.

**Core Principle:** The constructor acquires resources and sets up object state, while the destructor automatically releases those resources when the object goes out of scope.

```cpp
#include <iostream>
#include <fstream>
#include <stdexcept>

// Bad: Manual resource management (C-style)
void badExample() {
    int* ptr = new int[100];  // Acquire resource
    // ... some code ...
    if (someError) {
        // Forgot to delete! Memory leak!
        return;
    }
    delete[] ptr;  // Release resource
}

// Good: RAII approach
class RAIIWrapper {
private:
    int* data;
    std::ofstream file;
    
public:
    // Constructor acquires resources
    RAIIWrapper(const char* filename, size_t size) 
        : data(new int[size]), file(filename) {
        std::cout << "Resources acquired" << std::endl;
        if (!file.is_open()) {
            delete[] data;
            throw std::runtime_error("Cannot open file");
        }
    }
    
    // Destructor automatically releases resources
    ~RAIIWrapper() {
        delete[] data;
        if (file.is_open()) {
            file.close();
        }
        std::cout << "Resources released" << std::endl;
    }
    
    // Prevent copying (or implement properly)
    RAIIWrapper(const RAIIWrapper&) = delete;
    RAIIWrapper& operator=(const RAIIWrapper&) = delete;
};

// Even better: Use standard library RAII classes
void modernExample() {
    std::ifstream file("example.txt");  // RAII - file closes automatically
    std::vector<int> vec(100);          // RAII - memory freed automatically
    std::unique_ptr<int[]> ptr(new int[100]); // RAII - memory freed automatically
    
    if (!file.is_open()) {
        return;  // No leak! All destructors called automatically
    }
    // Use resources...
}  // All resources automatically released here!

int main() {
    {
        RAIIWrapper wrapper("test.txt", 100);
        // Use wrapper...
    } // Destructor called automatically - resources released
    
    modernExample(); // All resources managed automatically
    
    return 0;
}
```

**RAII Advantages:**
- **Exception safety**: Resources are released even if exceptions occur
- **No manual cleanup**: Destructors are called automatically
- **Race condition prevention**: Resources are properly initialized before use
- **Standard in C++**: Used by standard containers, smart pointers, locks

---

### 52. What is new and delete?

**`new`** and **`delete`** are C++ operators for **dynamic memory allocation** and **deallocation**. They are more sophisticated than C's malloc/free as they also **call constructors and destructors**.

#### `new` operator:
1. Allocates memory from the heap
2. Calls the constructor to initialize the object
3. Returns a pointer to the created object

#### `delete` operator:
1. Calls the destructor to clean up the object
2. Deallocates the memory back to the heap

```cpp
#include <iostream>

class Person {
private:
    std::string name;
    int age;
    
public:
    Person(const std::string& n, int a) : name(n), age(a) {
        std::cout << "Person " << name << " created" << std::endl;
    }
    
    ~Person() {
        std::cout << "Person " << name << " destroyed" << std::endl;
    }
    
    void introduce() const {
        std::cout << "Hi, I'm " << name << ", " << age << " years old" << std::endl;
    }
};

int main() {
    // Single object allocation
    Person* p1 = new Person("Alice", 25);  // Allocates memory + calls constructor
    p1->introduce();
    delete p1;  // Calls destructor + deallocates memory
    
    // Array allocation
    Person* people = new Person[3] {  // Calls constructor for each element
        {"Bob", 30},
        {"Charlie", 35},
        {"Diana", 40}
    };
    
    for (int i = 0; i < 3; i++) {
        people[i].introduce();
    }
    
    delete[] people;  // Calls destructor for each element + deallocates
    
    // Different forms of new
    int* p2 = new int;           // Default-initialized (garbage value)
    int* p3 = new int();         // Value-initialized (0)
    int* p4 = new int(42);       // Direct-initialized (42)
    int* p5 = new int[10];       // Array of 10 ints
    int* p6 = new int[10]();     // Array of 10 ints, zero-initialized
    
    delete p2;   delete p3;   delete p4;
    delete[] p5; delete[] p6;
    
    // C++17: aligned allocation
    alignas(16) double* p7 = new double;
    delete p7;
    
    return 0;
}
```

**Important notes:**
- Always match `new` with `delete`, and `new[]` with `delete[]`
- Never mix `malloc`/`free` with `new`/`delete`
- In modern C++, prefer smart pointers over raw `new`/`delete`

---

### 53. What is difference between malloc/free and new/delete?

| Feature | `malloc`/`free` | `new`/`delete` |
| :--- | :--- | :--- |
| **Language** | C functions | C++ operators |
| **Constructor/Destructor** | ❌ Not called | ✅ Called automatically |
| **Return type** | `void*` (needs casting) | Typed pointer |
| **Failure handling** | Returns `NULL` | Throws `std::bad_alloc` (or returns nullptr with `nothrow` version) |
| **Size calculation** | Manual (`sizeof`) | Automatic |
| **Memory allocation** | Allocates raw memory | Allocates memory + constructs object |
| **Overridable** | ❌ No | ✅ Yes (operator new/delete can be overloaded) |
| **Array allocation** | Manual calculation needed | `new[]` and `delete[]` syntax |

```cpp
#include <iostream>
#include <cstdlib>  // for malloc/free

class Test {
public:
    int data;
    
    Test() : data(42) {
        std::cout << "Constructor called" << std::endl;
    }
    
    ~Test() {
        std::cout << "Destructor called" << std::endl;
    }
};

int main() {
    // C-style: malloc/free
    std::cout << "=== malloc/free ===" << std::endl;
    Test* t1 = (Test*)malloc(sizeof(Test));  // Raw memory, no constructor
    // t1->data = 100;  // Object not constructed! Undefined behavior!
    free(t1);  // No destructor called
    
    // C++ style: new/delete
    std::cout << "=== new/delete ===" << std::endl;
    Test* t2 = new Test();  // Allocates memory AND calls constructor
    std::cout << "Data: " << t2->data << std::endl;
    delete t2;  // Calls destructor AND deallocates memory
    
    // Array differences
    std::cout << "=== Arrays ===" << std::endl;
    
    // malloc for array
    int* arr1 = (int*)malloc(10 * sizeof(int));  // Need manual size calc
    
    // new for array
    int* arr2 = new int[10];  // Automatic size calculation
    
    free(arr1);
    delete[] arr2;
    
    // Failure handling
    int* p1 = (int*)malloc(1000000000000);  // Returns NULL on failure
    if (p1 == NULL) {
        std::cout << "malloc failed" << std::endl;
    }
    
    try {
        int* p2 = new int[1000000000000];  // Throws exception on failure
    } catch (const std::bad_alloc& e) {
        std::cout << "new failed: " << e.what() << std::endl;
    }
    
    // nothrow version of new
    int* p3 = new (std::nothrow) int[1000000000000];
    if (p3 == nullptr) {
        std::cout << "nothrow new failed" << std::endl;
    }
    
    return 0;
}
```

---

### 54. What is placement new?

**Placement new** is a variant of the `new` operator that **constructs an object at a specific memory location** instead of allocating new memory. It allows you to separate memory allocation from object construction.

**Syntax:** `new (address) Type(parameters)`

```cpp
#include <iostream>
#include <cstdlib>  // for aligned_alloc (C++17)

class Widget {
public:
    int id;
    std::string name;
    
    Widget(int i, const std::string& n) : id(i), name(n) {
        std::cout << "Widget " << id << " constructed at " << this << std::endl;
    }
    
    ~Widget() {
        std::cout << "Widget " << id << " destroyed" << std::endl;
    }
};

int main() {
    // Example 1: Basic placement new
    std::cout << "=== Basic placement new ===" << std::endl;
    
    // Allocate raw memory buffer
    char buffer[sizeof(Widget) * 3];
    std::cout << "Buffer at: " << (void*)buffer << std::endl;
    
    // Construct object in the buffer
    Widget* w1 = new (buffer) Widget(1, "First");
    std::cout << "w1 at: " << w1 << std::endl;
    
    // Construct another object at offset
    Widget* w2 = new (buffer + sizeof(Widget)) Widget(2, "Second");
    std::cout << "w2 at: " << w2 << std::endl;
    
    // Must manually call destructors
    w1->~Widget();
    w2->~Widget();
    
    // Example 2: Placement new with alignment (C++17)
    std::cout << "\n=== Aligned placement new ===" << std::endl;
    
    // Get aligned memory
    void* aligned_mem = std::aligned_alloc(alignof(Widget), sizeof(Widget) * 2);
    std::cout << "Aligned memory at: " << aligned_mem << std::endl;
    
    Widget* w3 = new (aligned_mem) Widget(3, "Third");
    Widget* w4 = new (static_cast<char*>(aligned_mem) + sizeof(Widget)) Widget(4, "Fourth");
    
    w3->~Widget();
    w4->~Widget();
    std::free(aligned_mem);
    
    // Example 3: Placement new in vector-like container
    std::cout << "\n=== Custom vector simulation ===" << std::endl;
    
    class SimpleVector {
    private:
        char* data;
        size_t capacity;
        size_t size;
        
    public:
        SimpleVector(size_t cap) : capacity(cap), size(0) {
            data = static_cast<char*>(operator new(cap * sizeof(Widget)));
        }
        
        void push_back(int id, const std::string& name) {
            if (size < capacity) {
                new (data + size * sizeof(Widget)) Widget(id, name);
                ++size;
            }
        }
        
        void pop_back() {
            if (size > 0) {
                --size;
                reinterpret_cast<Widget*>(data + size * sizeof(Widget))->~Widget();
            }
        }
        
        Widget* operator[](size_t idx) {
            return reinterpret_cast<Widget*>(data + idx * sizeof(Widget));
        }
        
        ~SimpleVector() {
            // Destroy in reverse order
            for (size_t i = size; i > 0; --i) {
                reinterpret_cast<Widget*>(data + (i-1) * sizeof(Widget))->~Widget();
            }
            operator delete(data);
        }
    };
    
    SimpleVector vec(3);
    vec.push_back(5, "Five");
    vec.push_back(6, "Six");
    vec.push_back(7, "Seven");
    
    vec.pop_back();  // Destroys "Seven"
    
    return 0;
}
```

**Use cases:**
- Custom memory pools/allocators
- Embedded systems with fixed memory
- Implementing containers (like `std::vector`)
- Avoiding fragmentation in real-time systems

---

### 55. What are smart pointers?

**Smart pointers** are RAII-based wrapper classes that **automatically manage the lifetime of dynamically allocated objects**. They ensure proper memory deallocation even in case of exceptions, preventing memory leaks.

**Types of smart pointers (C++11):**

| Smart Pointer | Description | Ownership |
| :--- | :--- | :--- |
| `std::unique_ptr` | Exclusive ownership | Single owner |
| `std::shared_ptr` | Shared ownership | Multiple owners (reference counting) |
| `std::weak_ptr` | Non-owning observer | Breaks circular references |

```cpp
#include <iostream>
#include <memory>  // For smart pointers
#include <vector>

class Resource {
public:
    int id;
    
    Resource(int i) : id(i) {
        std::cout << "Resource " << id << " acquired" << std::endl;
    }
    
    ~Resource() {
        std::cout << "Resource " << id << " released" << std::endl;
    }
    
    void use() const {
        std::cout << "Using resource " << id << std::endl;
    }
};

// Without smart pointers (BAD)
void badExample() {
    Resource* res = new Resource(1);
    res->use();
    if (someCondition) {
        // Oops! Forgot to delete - memory leak!
        return;
    }
    delete res;
}

// With smart pointers (GOOD)
void goodExample() {
    // Automatically deleted when going out of scope
    std::unique_ptr<Resource> res = std::make_unique<Resource>(2);
    res->use();
    
    if (someCondition) {
        return;  // No leak! unique_ptr destructor called
    }
}  // Automatically deleted here too

int main() {
    // Example 1: Basic unique_ptr
    std::cout << "=== unique_ptr ===" << std::endl;
    {
        std::unique_ptr<Resource> ptr1 = std::make_unique<Resource>(10);
        ptr1->use();
        
        // std::unique_ptr<Resource> ptr2 = ptr1;  // ERROR: can't copy
        std::unique_ptr<Resource> ptr2 = std::move(ptr1);  // Transfer ownership
        // ptr1 is now nullptr
        ptr2->use();
    }  // Resource automatically deleted
    
    // Example 2: shared_ptr
    std::cout << "\n=== shared_ptr ===" << std::endl;
    {
        std::shared_ptr<Resource> ptr1 = std::make_shared<Resource>(20);
        {
            std::shared_ptr<Resource> ptr2 = ptr1;  // Reference count = 2
            std::shared_ptr<Resource> ptr3 = ptr1;  // Reference count = 3
            ptr2->use();
            std::cout << "Use count: " << ptr1.use_count() << std::endl;
        }  // ptr2, ptr3 destroyed - count back to 1
        std::cout << "Use count: " << ptr1.use_count() << std::endl;
    }  // Last reference destroyed - Resource deleted
    
    // Example 3: weak_ptr (breaks cycles)
    std::cout << "\n=== weak_ptr ===" << std::endl;
    {
        std::shared_ptr<Resource> shared = std::make_shared<Resource>(30);
        std::weak_ptr<Resource> weak = shared;  // Non-owning observer
        
        std::cout << "Shared use count: " << shared.use_count() << std::endl;
        
        if (auto locked = weak.lock()) {  // Try to get shared_ptr
            std::cout << "Resource still alive: ";
            locked->use();
        }
        
        shared.reset();  // Release ownership
        
        if (auto locked = weak.lock()) {
            std::cout << "Resource still alive" << std::endl;
        } else {
            std::cout << "Resource has been destroyed" << std::endl;
        }
    }
    
    return 0;
}
```

---

### 56. Explain unique_ptr vs shared_ptr vs weak_ptr

#### **`std::unique_ptr`**
- **Exclusive ownership** - only one `unique_ptr` can own an object at a time
- Cannot be copied, only moved
- Lightweight (no reference counting overhead)
- Automatically deletes object when destroyed or reset
- Can have custom deleter

```cpp
std::unique_ptr<int> ptr1 = std::make_unique<int>(42);
// std::unique_ptr<int> ptr2 = ptr1;  // ERROR: can't copy
std::unique_ptr<int> ptr2 = std::move(ptr1);  // OK: transfer ownership
// ptr1 is now nullptr

// With custom deleter
auto deleter = [](FILE* f) { if (f) fclose(f); };
std::unique_ptr<FILE, decltype(deleter)> filePtr(fopen("test.txt", "r"), deleter);
```

#### **`std::shared_ptr`**
- **Shared ownership** - multiple `shared_ptr` can own the same object
- Uses **reference counting** - object deleted when count reaches 0
- Copying increments reference count
- Slightly heavier due to control block overhead
- Thread-safe reference counting (but object access needs separate synchronization)

```cpp
std::shared_ptr<int> ptr1 = std::make_shared<int>(42);
std::shared_ptr<int> ptr2 = ptr1;  // Reference count = 2
std::shared_ptr<int> ptr3 = ptr1;  // Reference count = 3

ptr2.reset();  // Count = 2
ptr3.reset();  // Count = 1
ptr1.reset();  // Count = 0 -> object deleted
```

#### **`std::weak_ptr`**
- **Non-owning observer** - doesn't affect reference count
- Created from `shared_ptr`
- Used to **break circular references** that would cause memory leaks
- Must be **locked** to access the object (creates temporary `shared_ptr`)
- Expires automatically when object is deleted

```cpp
std::shared_ptr<int> shared = std::make_shared<int>(42);
std::weak_ptr<int> weak = shared;

// Access requires locking
if (auto locked = weak.lock()) {
    std::cout << "Value: " << *locked << std::endl;
} else {
    std::cout << "Object no longer exists" << std::endl;
}

// Breaking circular references
struct Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;  // Use weak_ptr to avoid cycle
    ~Node() { std::cout << "Node destroyed" << std::endl; }
};

auto n1 = std::make_shared<Node>();
auto n2 = std::make_shared<Node>();
n1->next = n2;
n2->prev = n1;  // Weak reference doesn't prevent destruction
```

| Feature | `unique_ptr` | `shared_ptr` | `weak_ptr` |
| :--- | :--- | :--- | :--- |
| **Ownership** | Exclusive | Shared | None (observer) |
| **Copyable** | ❌ No (only move) | ✅ Yes | ✅ Yes |
| **Reference counting** | ❌ No | ✅ Yes | ❌ No (but observes count) |
| **Overhead** | Minimal | Control block | From shared_ptr |
| **Use case** | RAII, factory functions | Shared resources | Cache, breaking cycles |

---

### 57. What is memory leak and how to avoid it?

A **memory leak** occurs when a program **fails to release memory** that is no longer needed, causing the program to consume more and more memory over time, eventually leading to performance degradation or crashes.

```cpp
#include <iostream>
#include <memory>
#include <vector>

// BAD: Memory leak examples
void leakExamples() {
    // Leak 1: Forgot to delete
    int* ptr1 = new int(42);
    // No delete - memory leak!
    
    // Leak 2: Lost pointer
    int* ptr2 = new int[100];
    ptr2 = new int[50];  // Original memory lost - can't delete it now!
    
    // Leak 3: Exception before delete
    int* ptr3 = new int(100);
    throw std::runtime_error("Oops!");  // Function exits, delete never called
    delete ptr3;  // Never reached
    
    // Leak 4: Improper array deletion
    int* ptr4 = new int[100];
    delete ptr4;  // Should be delete[] - undefined behavior, potential leak
}

// GOOD: Memory leak prevention
class SafeResource {
private:
    std::unique_ptr<int[]> data;
    
public:
    SafeResource(size_t size) : data(std::make_unique<int[]>(size)) {}
    // No destructor needed - unique_ptr handles cleanup
};

void leakFreeExamples() {
    // Solution 1: Use smart pointers
    std::unique_ptr<int> ptr1 = std::make_unique<int>(42);
    // Automatically deleted
    
    // Solution 2: RAII containers
    std::vector<int> vec(100);  // Memory managed automatically
    
    // Solution 3: Scope-based allocation
    {
        int* ptr2 = new int(100);
        std::unique_ptr<int> smartPtr(ptr2);  // Wrap raw pointer
    }  // Automatically deleted
    
    // Solution 4: Handle exceptions
    try {
        auto ptr3 = std::make_unique<int[]>(1000);
        throw std::runtime_error("Error!");
        // unique_ptr still cleans up on stack unwinding
    } catch (...) {
        std::cout << "Exception caught, no leak!" << std::endl;
    }
}

// Leak detection tools
int main() {
    // To detect leaks:
    // 1. Valgrind (Linux): valgrind --leak-check=full ./program
    // 2. Visual Studio: _CrtDumpMemoryLeaks();
    // 3. Address Sanitizer: compile with -fsanitize=address
    
    leakFreeExamples();
    
    #ifdef _MSC_VER
    _CrtDumpMemoryLeaks();  // Reports leaks in debug mode
    #endif
    
    return 0;
}
```

**How to avoid memory leaks:**

| Technique | Description |
| :--- | :--- |
| **Smart pointers** | Use `unique_ptr`, `shared_ptr` instead of raw pointers |
| **RAII** | Tie resource lifetime to object lifetime |
| **STL containers** | Use `vector`, `string`, etc. instead of manual arrays |
| **Consistent pairing** | Always match `new` with `delete`, `new[]` with `delete[]` |
| **Exception safety** | Ensure cleanup happens even with exceptions |
| **Tools** | Use Valgrind, sanitizers, static analysis |

---

### 58. What are dangling pointers?

A **dangling pointer** is a pointer that **points to memory that has already been freed or deallocated**. Using a dangling pointer leads to **undefined behavior** - crashes, data corruption, or security vulnerabilities.

```cpp
#include <iostream>
#include <memory>

int* createDanglingPointer() {
    int localVar = 42;
    return &localVar;  // BAD: pointer to local variable (stack memory)
}  // localVar destroyed here - pointer now dangling

int main() {
    // Example 1: Pointer to deallocated memory
    std::cout << "=== Dangling from delete ===" << std::endl;
    int* ptr1 = new int(100);
    std::cout << "Value: " << *ptr1 << std::endl;
    
    delete ptr1;  // Memory freed
    // ptr1 is now dangling!
    
    // Dangerous: using dangling pointer
    // *ptr1 = 200;  // UNDEFINED BEHAVIOR - crash or corruption
    // std::cout << *ptr1 << std::endl;  // UNDEFINED BEHAVIOR
    
    // Example 2: Multiple pointers to same memory
    std::cout << "\n=== Multiple pointers ===" << std::endl;
    int* ptr2 = new int(50);
    int* ptr3 = ptr2;  // Both point to same memory
    
    delete ptr2;  // Memory freed
    ptr2 = nullptr;  // Good: set to null
    // ptr3 is now dangling! Still points to freed memory
    
    // Example 3: Return pointer to local (stack)
    std::cout << "\n=== Local variable ===" << std::endl;
    int* ptr4 = createDanglingPointer();  // ptr4 is dangling immediately
    // *ptr4 = 10;  // UNDEFINED BEHAVIOR - stack memory already reused
    
    // Example 4: Pointer to out-of-scope object
    std::cout << "\n=== Scope issue ===" << std::endl;
    int* ptr5;
    {
        int temp = 30;
        ptr5 = &temp;  // Points to local variable
    }  // temp destroyed, ptr5 now dangling
    
    // How to prevent dangling pointers
    std::cout << "\n=== Prevention ===" << std::endl;
    
    // Solution 1: Set to nullptr after delete
    int* safePtr1 = new int(75);
    delete safePtr1;
    safePtr1 = nullptr;  // Now safe - checking for nullptr prevents use
    
    // Solution 2: Use smart pointers
    std::unique_ptr<int> safePtr2 = std::make_unique<int>(100);
    // Automatically managed, no dangling
    
    // Solution 3: Scope limitation
    {
        std::unique_ptr<int> scopedPtr = std::make_unique<int>(200);
        // Use scopedPtr...
    }  // Automatically destroyed - no dangling
    
    // Solution 4: Weak pointers for observers
    std::shared_ptr<int> shared = std::make_shared<int>(300);
    std::weak_ptr<int> weak = shared;  // Non-owning observer
    
    shared.reset();  // Object destroyed
    
    if (auto locked = weak.lock()) {
        // Won't enter - object gone
        std::cout << "Value: " << *locked << std::endl;
    } else {
        std::cout << "Object no longer exists - safe handling" << std::endl;
    }
    
    return 0;
}
```

**Types of dangling pointers:**

| Type | Cause | Example |
| :--- | :--- | :--- |
| **Heap dangling** | Memory freed with `delete` | Pointer to deleted heap memory |
| **Stack dangling** | Local variable out of scope | Pointer to function local |
| **Multiple pointers** | One pointer freed, others still exist | Shared raw pointers |

**Prevention:**
- Set pointers to `nullptr` after `delete`
- Use smart pointers (`unique_ptr`, `shared_ptr`)
- Prefer references when non-null guaranteed
- Use `weak_ptr` for non-owning observation

---

### 59. What is object slicing?

**Object slicing** occurs when a **derived class object is assigned to a base class object** (by value), causing the derived-specific parts to be "sliced off". Only the base class portion remains, leading to loss of data and polymorphic behavior.

```cpp
#include <iostream>
#include <vector>
#include <memory>

class Animal {
protected:
    std::string name;
    
public:
    Animal(const std::string& n) : name(n) {}
    
    virtual void speak() const {
        std::cout << name << " makes a sound" << std::endl;
    }
    
    virtual std::string getType() const {
        return "Animal";
    }
    
    virtual ~Animal() = default;
};

class Dog : public Animal {
private:
    std::string breed;  // Derived-specific member
    
public:
    Dog(const std::string& n, const std::string& b) 
        : Animal(n), breed(b) {}
    
    void speak() const override {
        std::cout << name << " the " << breed << " says: Woof!" << std::endl;
    }
    
    std::string getType() const override {
        return "Dog (" + breed + ")";
    }
    
    void wagTail() const {  // Derived-specific function
        std::cout << name << " wags tail" << std::endl;
    }
};

int main() {
    // Example 1: Basic slicing
    std::cout << "=== Basic Slicing ===" << std::endl;
    
    Dog dog("Buddy", "Golden Retriever");
    Animal animal = dog;  // SLICING! dog's derived parts are lost
    
    dog.speak();      // Output: Buddy the Golden Retriever says: Woof!
    animal.speak();   // Output: Buddy makes a sound (polymorphism lost!)
    
    // Can't call wagTail() on animal - breed information lost
    
    // Example 2: Container slicing
    std::cout << "\n=== Container Slicing ===" << std::endl;
    
    std::vector<Animal> animals;  // Stores Animal objects, not references
    animals.push_back(Dog("Rex", "German Shepherd"));
    animals.push_back(Animal("Generic"));
    
    for (const auto& a : animals) {
        std::cout << "Type: " << a.getType() << std::endl;  // Always "Animal"
        a.speak();  // Always Animal::speak()
    }
    
    // Example 3: Function parameter slicing
    std::cout << "\n=== Function Parameter Slicing ===" << std::endl;
    
    auto processByValue = [](Animal a) {  // Takes by value - slicing!
        std::cout << "Processing: ";
        a.speak();
    };
    
    auto processByReference = [](const Animal& a) {  // Takes by reference - no slicing
        std::cout << "Processing (ref): ";
        a.speak();
    };
    
    Dog dog2("Max", "Beagle");
    
    processByValue(dog2);      // Slicing occurs
    processByReference(dog2);  // No slicing - polymorphic
    
    // Example 4: Return type slicing
    std::cout << "\n=== Return Type Slicing ===" << std::endl;
    
    auto getAnimal = []() -> Animal {  // Returns Animal - slicing!
        return Dog("Lucy", "Poodle");
    };
    
    Animal sliced = getAnimal();  // Slicing
    sliced.speak();  // Animal version
    
    // Correct way: Use pointers/references
    std::cout << "\n=== Correct Approaches ===" << std::endl;
    
    // Using pointers
    Animal* animalPtr = new Dog("Charlie", "Labrador");
    animalPtr->speak();  // Polymorphic behavior preserved
    delete animalPtr;
    
    // Using references
    Dog dog3("Daisy", "Pug");
    Animal& animalRef = dog3;  // No slicing
    animalRef.speak();  // Dog::speak() called
    
    // Using smart pointers in containers
    std::vector<std::unique_ptr<Animal>> animalVec;
    animalVec.push_back(std::make_unique<Dog>("Rocky", "Boxer"));
    animalVec.push_back(std::make_unique<Animal>("Generic"));
    
    for (const auto& aPtr : animalVec) {
        aPtr->speak();  // Polymorphic behavior preserved
    }
    
    return 0;
}
```

**How to avoid slicing:**

| Technique | Description |
| :--- | :--- |
| **Use pointers/references** | Pass/return by pointer or reference, not by value |
| **Virtual functions** | Ensure polymorphic behavior via virtual functions |
| **Smart pointers in containers** | Store `unique_ptr<Base>` or `shared_ptr<Base>` |
| **Prevent copying** | Make base class copy operations protected or deleted |
| **Clone function** | Implement virtual `clone()` for polymorphic copying |

---

### 60. How does C++ handle object lifetime?

C++ manages object lifetime through **deterministic destruction** - objects are created at specific points and destroyed at specific points (when they go out of scope or are explicitly deleted).

```cpp
#include <iostream>
#include <memory>
#include <vector>

class TraceObject {
private:
    static int counter;
    int id;
    std::string name;
    
public:
    TraceObject(const std::string& n) : id(++counter), name(n) {
        std::cout << "[" << id << "] " << name << " constructed" << std::endl;
    }
    
    ~TraceObject() {
        std::cout << "[" << id << "] " << name << " destroyed" << std::endl;
    }
    
    TraceObject(const TraceObject& other) : id(++counter), name(other.name + " (copy)") {
        std::cout << "[" << id << "] " << name << " copy-constructed from [" 
                  << other.id << "]" << std::endl;
    }
    
    TraceObject(TraceObject&& other) : id(++counter), name(std::move(other.name) + " (moved)") {
        std::cout << "[" << id << "] " << name << " move-constructed from [" 
                  << other.id << "]" << std::endl;
    }
    
    void use() const {
        std::cout << "[" << id << "] " << name << " in use" << std::endl;
    }
};

int TraceObject::counter = 0;

// Example 1: Automatic (stack) lifetime
void automaticLifetime() {
    std::cout << "\n=== Automatic (Stack) Lifetime ===" << std::endl;
    
    TraceObject obj1("stack1");  // Constructed here
    {
        TraceObject obj2("stack2");  // Nested scope
        obj2.use();
    }  // obj2 destroyed here
    
    obj1.use();
    // obj1 destroyed when function returns
}

// Example 2: Static lifetime
TraceObject staticObj("static");  // Constructed before main()

void staticLifetime() {
    std::cout << "\n=== Static Lifetime ===" << std::endl;
    
    static TraceObject funcStatic("function-static");  // Constructed first time function called
    
    staticObj.use();
    funcStatic.use();
}  // Not destroyed here - lives until program end

// Example 3: Dynamic lifetime
void dynamicLifetime() {
    std::cout << "\n=== Dynamic Lifetime ===" << std::endl;
    
    // Raw new/delete
    TraceObject* rawPtr = new TraceObject("raw-dynamic");
    rawPtr->use();
    delete rawPtr;  // Must explicitly delete
    
    // Smart pointers
    std::unique_ptr<TraceObject> uniquePtr = 
        std::make_unique<TraceObject>("unique-dynamic");
    uniquePtr->use();
    // Automatically destroyed when uniquePtr goes out of scope
    
    std::shared_ptr<TraceObject> sharedPtr1 = 
        std::make_shared<TraceObject>("shared-dynamic");
    {
        std::shared_ptr<TraceObject> sharedPtr2 = sharedPtr1;  // Share ownership
        std::cout << "Use count: " << sharedPtr1.use_count() << std::endl;
    }  // sharedPtr2 destroyed, count decreases
    std::cout << "Use count: " << sharedPtr1.use_count() << std::endl;
    // Destroyed when last shared_ptr (sharedPtr1) goes out of scope
}

// Example 4: Member object lifetime
class Container {
private:
    TraceObject member1;
    TraceObject member2;
    std::unique_ptr<TraceObject> memberPtr;
    
public:
    Container() : member1("container-member1"), member2("container-member2"),
                  memberPtr(std::make_unique<TraceObject>("container-ptr")) {
        std::cout << "Container constructed" << std::endl;
    }
    
    ~Container() {
        std::cout << "Container destructed" << std::endl;
        // Members destroyed in reverse order: memberPtr, member2, member1
    }
};

// Example 5: Temporary object lifetime
TraceObject createTemp() {
    return TraceObject("temporary");  // Return value optimization may apply
}

int main() {
    std::cout << "=== Program Start ===" << std::endl;
    
    automaticLifetime();
    
    staticLifetime();
    
    dynamicLifetime();
    
    std::cout << "\n=== Container Example ===" << std::endl;
    {
        Container container;
    }  // Container and its members destroyed here
    
    std::cout << "\n=== Temporary Objects ===" << std::endl;
    const TraceObject& ref = createTemp();  // Lifetime of temporary extended to match ref
    ref.use();
    // Temporary destroyed after ref goes out of scope
    
    std::cout << "\n=== Program End ===" << std::endl;
    // static objects destroyed after main() in reverse order of construction
    return 0;
}
```

**Object Lifetime Categories:**

| Storage Duration | Creation | Destruction | Example |
| :--- | :--- | :--- | :--- |
| **Automatic (stack)** | At declaration | End of scope | Local variables |
| **Static** | Program start (or first use) | Program end | Global, static locals |
| **Thread-local** | Thread start | Thread end | `thread_local` variables |
| **Dynamic (heap)** | `new`, `make_unique`, etc. | `delete` or smart pointer | Allocated memory |
| **Temporary** | When created | End of full expression | Return values, casts |

**Key Lifetime Rules:**
- Constructors run from base to derived
- Destructors run from derived to base
- Members constructed in declaration order, destroyed in reverse order
- Temporary objects destroyed at end of full expression (except when bound to const reference)
- Static objects destroyed in reverse order of construction