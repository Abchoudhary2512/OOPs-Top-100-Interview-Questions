### 71. What are templates in C++?

**Templates** are a powerful feature in C++ that enable **generic programming** - writing code that works with any data type. They allow you to define functions and classes that operate on types as parameters, **generating type-safe code at compile-time**.

```cpp
#include <iostream>
#include <string>
#include <vector>

// ==================== Basic Template Concepts ====================

// Without templates - need separate functions for each type
int maxInt(int a, int b) { return (a > b) ? a : b; }
double maxDouble(double a, double b) { return (a > b) ? a : b; }
std::string maxString(std::string a, std::string b) { return (a > b) ? a : b; }

// With templates - one definition works for any type
template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

// ==================== How Templates Work ====================

// Template definition - blueprint
template <typename T>
class Box {
private:
    T content;
    
public:
    Box(const T& item) : content(item) {}
    
    T getValue() const { return content; }
    
    void setValue(const T& item) { content = item; }
    
    void display() const {
        std::cout << "Box contains: " << content << std::endl;
    }
};

// ==================== Multiple Template Parameters ====================

template <typename T, typename U>
class Pair {
private:
    T first;
    U second;
    
public:
    Pair(const T& f, const U& s) : first(f), second(s) {}
    
    T getFirst() const { return first; }
    U getSecond() const { return second; }
    
    void display() const {
        std::cout << "Pair: (" << first << ", " << second << ")" << std::endl;
    }
};

// ==================== Template Type Deduction ====================

template <typename T>
void printType(const T& value) {
    std::cout << "Value: " << value << " (type: " << typeid(T).name() << ")" << std::endl;
}

// ==================== Non-type Template Parameters ====================

template <typename T, int Size>
class Array {
private:
    T data[Size];
    
public:
    T& operator[](int index) { return data[index]; }
    const T& operator[](int index) const { return data[index]; }
    
    int size() const { return Size; }
    
    void fill(const T& value) {
        for (int i = 0; i < Size; ++i) {
            data[i] = value;
        }
    }
    
    void display() const {
        std::cout << "Array<" << typeid(T).name() << ", " << Size << ">: ";
        for (int i = 0; i < Size; ++i) {
            std::cout << data[i] << " ";
        }
        std::cout << std::endl;
    }
};

// ==================== Template Template Parameters ====================

template <template <typename> class Container, typename T>
class Wrapper {
private:
    Container<T> container;
    
public:
    void add(const T& item) { container.push_back(item); }
    // ... other methods
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Templates ===" << std::endl;
    
    // Compiler generates code for each used type
    std::cout << "max(10, 20): " << max(10, 20) << std::endl;
    std::cout << "max(3.14, 2.72): " << max(3.14, 2.72) << std::endl;
    std::cout << "max('A', 'B'): " << max('A', 'B') << std::endl;
    
    std::cout << "\n=== Class Templates ===" << std::endl;
    
    Box<int> intBox(42);
    intBox.display();
    
    Box<std::string> stringBox("Hello Templates");
    stringBox.display();
    
    std::cout << "\n=== Multiple Type Parameters ===" << std::endl;
    
    Pair<int, std::string> person(25, "John");
    person.display();
    
    Pair<double, double> coordinates(3.14, 2.718);
    coordinates.display();
    
    std::cout << "\n=== Type Deduction (C++17) ===" << std::endl;
    
    printType(42);
    printType(3.14159);
    printType("Hello");
    
    std::cout << "\n=== Non-type Template Parameters ===" << std::endl;
    
    Array<int, 5> intArray;
    intArray.fill(10);
    intArray.display();
    
    Array<double, 3> doubleArray;
    doubleArray[0] = 1.1;
    doubleArray[1] = 2.2;
    doubleArray[2] = 3.3;
    doubleArray.display();
    
    std::cout << "\n=== Benefits of Templates ===" << std::endl;
    std::cout << "1. Code reuse - write once, use with any type" << std::endl;
    std::cout << "2. Type safety - compile-time checking" << std::endl;
    std::cout << "3. Performance - no runtime overhead" << std::endl;
    std::cout << "4. Flexibility - work with user-defined types" << std::endl;
    
    return 0;
}
```

**Template Instantiation:** When you use a template with specific types, the compiler generates actual code (instantiates the template) for those types.

---

### 72. What is function template?

A **function template** is a blueprint for creating functions that work with different data types without rewriting the function for each type.

```cpp
#include <iostream>
#include <string>
#include <vector>

// ==================== Basic Function Template ====================

// Single type parameter
template <typename T>
T maximum(T a, T b) {
    return (a > b) ? a : b;
}

// Multiple type parameters
template <typename T, typename U>
auto multiply(T a, U b) -> decltype(a * b) {
    return a * b;
}

// ==================== Function Template Overloading ====================

// Generic version
template <typename T>
void print(const T& value) {
    std::cout << "Generic: " << value << std::endl;
}

// Specialized for vectors
template <typename T>
void print(const std::vector<T>& vec) {
    std::cout << "Vector: [";
    for (size_t i = 0; i < vec.size(); ++i) {
        if (i > 0) std::cout << ", ";
        std::cout << vec[i];
    }
    std::cout << "]" << std::endl;
}

// Non-template overload
void print(const std::string& s) {
    std::cout << "String: \"" << s << "\"" << std::endl;
}

// ==================== Template with Multiple Parameters ====================

template <typename T, typename U>
bool areEqual(const T& a, const U& b) {
    return a == b;
}

// ==================== Template with Default Types ====================

template <typename T = int>
T getDefault() {
    return T();
}

// ==================== Function Template Specialization ====================

// Generic swap
template <typename T>
void mySwap(T& a, T& b) {
    std::cout << "Generic swap" << std::endl;
    T temp = a;
    a = b;
    b = temp;
}

// Specialization for arrays - just for demonstration
template <typename T, size_t N>
void mySwap(T (&a)[N], T (&b)[N]) {
    std::cout << "Array swap" << std::endl;
    for (size_t i = 0; i < N; ++i) {
        T temp = a[i];
        a[i] = b[i];
        b[i] = temp;
    }
}

// ==================== Advanced: SFINAE with Function Templates ====================

// Enable only for integral types
template <typename T>
typename std::enable_if<std::is_integral<T>::value, bool>::type
isEven(T value) {
    return value % 2 == 0;
}

// Enable only for floating point types
template <typename T>
typename std::enable_if<std::is_floating_point<T>::value, bool>::type
isEven(T value) {
    return static_cast<int>(value) % 2 == 0;
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Function Templates ===" << std::endl;
    
    std::cout << "maximum(10, 20): " << maximum(10, 20) << std::endl;
    std::cout << "maximum(3.14, 2.72): " << maximum(3.14, 2.72) << std::endl;
    std::cout << "maximum('A', 'B'): " << maximum('A', 'B') << std::endl;
    
    // Explicit type specification
    std::cout << "maximum<double>(10, 20.5): " 
              << maximum<double>(10, 20.5) << std::endl;
    
    std::cout << "\n=== Multiple Type Parameters ===" << std::endl;
    
    std::cout << "multiply(5, 3.14): " << multiply(5, 3.14) << std::endl;
    std::cout << "multiply(2.5, 4): " << multiply(2.5, 4) << std::endl;
    
    std::cout << "\n=== Function Overloading ===" << std::endl;
    
    print(42);
    print(3.14159);
    print("Hello");  // const char* uses generic version
    
    std::vector<int> vec = {1, 2, 3, 4, 5};
    print(vec);
    
    std::string str = "Hello World";
    print(str);  // Calls non-template overload
    
    std::cout << "\n=== Template Specialization ===" << std::endl;
    
    int x = 10, y = 20;
    std::cout << "Before swap: x=" << x << ", y=" << y << std::endl;
    mySwap(x, y);
    std::cout << "After swap: x=" << x << ", y=" << y << std::endl;
    
    int arr1[] = {1, 2, 3};
    int arr2[] = {4, 5, 6};
    std::cout << "Before array swap: arr1[0]=" << arr1[0] << ", arr2[0]=" << arr2[0] << std::endl;
    mySwap(arr1, arr2);  // Calls array specialization
    std::cout << "After array swap: arr1[0]=" << arr1[0] << ", arr2[0]=" << arr2[0] << std::endl;
    
    std::cout << "\n=== SFINAE with Function Templates ===" << std::endl;
    
    std::cout << "isEven(42): " << std::boolalpha << isEven(42) << std::endl;
    std::cout << "isEven(3.14): " << isEven(3.14) << std::endl;
    // isEven("Hello");  // Error: no matching function
    
    return 0;
}
```

---

### 73. What is class template?

A **class template** is a blueprint for creating classes that work with different data types. It allows you to define generic classes where the types of data members and member functions can be parameterized.

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <stdexcept>

// ==================== Basic Class Template ====================

template <typename T>
class Stack {
private:
    std::vector<T> elements;
    
public:
    void push(const T& value) {
        elements.push_back(value);
    }
    
    void pop() {
        if (isEmpty()) {
            throw std::out_of_range("Stack::pop(): empty stack");
        }
        elements.pop_back();
    }
    
    T top() const {
        if (isEmpty()) {
            throw std::out_of_range("Stack::top(): empty stack");
        }
        return elements.back();
    }
    
    bool isEmpty() const {
        return elements.empty();
    }
    
    size_t size() const {
        return elements.size();
    }
    
    void display() const {
        std::cout << "Stack (size " << size() << "): [";
        for (size_t i = 0; i < elements.size(); ++i) {
            if (i > 0) std::cout << ", ";
            std::cout << elements[i];
        }
        std::cout << "]" << std::endl;
    }
};

// ==================== Class Template with Multiple Parameters ====================

template <typename K, typename V, int Capacity = 100>
class Cache {
private:
    std::pair<K, V> items[Capacity];
    int count;
    
public:
    Cache() : count(0) {}
    
    void put(const K& key, const V& value) {
        if (count < Capacity) {
            items[count++] = {key, value};
        }
    }
    
    V get(const K& key) const {
        for (int i = 0; i < count; ++i) {
            if (items[i].first == key) {
                return items[i].second;
            }
        }
        throw std::out_of_range("Key not found");
    }
    
    int getCount() const { return count; }
    int getCapacity() const { return Capacity; }
};

// ==================== Template Member Functions ====================

template <typename T>
class Calculator {
public:
    T add(T a, T b) { return a + b; }
    T subtract(T a, T b) { return a - b; }
    T multiply(T a, T b) { return a * b; }
    T divide(T a, T b) {
        if (b == 0) throw std::runtime_error("Division by zero");
        return a / b;
    }
};

// ==================== Class Template with Default Types ====================

template <typename T = int, typename Allocator = std::allocator<T>>
class DefaultContainer {
private:
    std::vector<T, Allocator> data;
    
public:
    void add(const T& item) { data.push_back(item); }
    // ... other methods
};

// ==================== Template Template Parameters ====================

template <template <typename, typename> class Container,
          typename T,
          typename Allocator = std::allocator<T>>
class ContainerWrapper {
private:
    Container<T, Allocator> container;
    
public:
    void add(const T& item) { container.push_back(item); }
    auto begin() { return container.begin(); }
    auto end() { return container.end(); }
};

// ==================== Friend Functions in Class Templates ====================

template <typename T>
class Point;

template <typename T>
std::ostream& operator<<(std::ostream& os, const Point<T>& p);

template <typename T>
class Point {
private:
    T x, y;
    
public:
    Point(T xVal, T yVal) : x(xVal), y(yVal) {}
    
    // Friend function
    friend std::ostream& operator<< <>(std::ostream& os, const Point<T>& p);
};

template <typename T>
std::ostream& operator<<(std::ostream& os, const Point<T>& p) {
    os << "Point(" << p.x << ", " << p.y << ")";
    return os;
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Class Template ===" << std::endl;
    
    Stack<int> intStack;
    intStack.push(10);
    intStack.push(20);
    intStack.push(30);
    intStack.display();
    std::cout << "Top: " << intStack.top() << std::endl;
    intStack.pop();
    intStack.display();
    
    Stack<std::string> stringStack;
    stringStack.push("Hello");
    stringStack.push("World");
    stringStack.display();
    
    std::cout << "\n=== Multiple Template Parameters ===" << std::endl;
    
    Cache<int, std::string, 5> cache;
    cache.put(1, "One");
    cache.put(2, "Two");
    cache.put(3, "Three");
    
    std::cout << "Cache count: " << cache.getCount() << std::endl;
    std::cout << "Cache capacity: " << cache.getCapacity() << std::endl;
    std::cout << "Key 2: " << cache.get(2) << std::endl;
    
    std::cout << "\n=== Template Member Functions ===" << std::endl;
    
    Calculator<int> intCalc;
    std::cout << "10 + 5 = " << intCalc.add(10, 5) << std::endl;
    
    Calculator<double> doubleCalc;
    std::cout << "3.14 * 2 = " << doubleCalc.multiply(3.14, 2.0) << std::endl;
    
    std::cout << "\n=== Friend Functions ===" << std::endl;
    
    Point<int> p1(3, 4);
    Point<double> p2(1.5, 2.5);
    std::cout << p1 << std::endl;
    std::cout << p2 << std::endl;
    
    return 0;
}
```

---

### 74. What is template specialization?

**Template specialization** allows you to provide **different implementations** of a template for specific types. This includes full specialization (all parameters specified) and partial specialization (some parameters specified).

```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <vector>

// ==================== Full Specialization ====================

// Generic template
template <typename T>
class Processor {
public:
    static void process(const T& value) {
        std::cout << "Generic processor: " << value << std::endl;
    }
};

// Full specialization for int
template <>
class Processor<int> {
public:
    static void process(const int& value) {
        std::cout << "Integer processor (optimized): " << value * 2 << std::endl;
    }
};

// Full specialization for std::string
template <>
class Processor<std::string> {
public:
    static void process(const std::string& value) {
        std::cout << "String processor: " << value << " (length: " 
                  << value.length() << ")" << std::endl;
    }
};

// ==================== Partial Specialization ====================

// Generic template for pointers
template <typename T>
class PointerHandler {
public:
    static void handle(T* ptr) {
        std::cout << "Generic pointer: " << *ptr << std::endl;
    }
};

// Partial specialization for const pointers
template <typename T>
class PointerHandler<const T> {
public:
    static void handle(const T* ptr) {
        std::cout << "Const pointer (read-only): " << *ptr << std::endl;
    }
};

// Partial specialization for arrays
template <typename T, size_t N>
class ArrayHandler {
public:
    static void handle(T (&arr)[N]) {
        std::cout << "Array of size " << N << ": ";
        for (size_t i = 0; i < N; ++i) {
            std::cout << arr[i] << " ";
        }
        std::cout << std::endl;
    }
};

// ==================== Specialization for Template Classes ====================

// Generic pair
template <typename T, typename U>
class Pair {
public:
    T first;
    U second;
    
    Pair(const T& f, const U& s) : first(f), second(s) {}
    
    void print() const {
        std::cout << "Generic Pair: (" << first << ", " << second << ")" << std::endl;
    }
};

// Partial specialization when both types are the same
template <typename T>
class Pair<T, T> {
public:
    T first;
    T second;
    
    Pair(const T& f, const T& s) : first(f), second(s) {}
    
    void print() const {
        std::cout << "Same-type Pair: (" << first << ", " << second << ")" << std::endl;
    }
    
    T max() const { return (first > second) ? first : second; }
};

// Full specialization for int, std::string
template <>
class Pair<int, std::string> {
public:
    int id;
    std::string name;
    
    Pair(int i, const std::string& n) : id(i), name(n) {}
    
    void print() const {
        std::cout << "ID-Pair: " << id << " -> \"" << name << "\"" << std::endl;
    }
};

// ==================== Specialization for Member Functions ====================

template <typename T>
class Container {
private:
    T value;
    
public:
    Container(const T& v) : value(v) {}
    
    void print() const {
        std::cout << "Container: " << value << std::endl;
    }
};

// Specialize just the print function for bool
template <>
void Container<bool>::print() const {
    std::cout << "Boolean Container: " << (value ? "true" : "false") << std::endl;
}

// ==================== Practical Example: Type Traits ====================

template <typename T>
struct IsPointer {
    static const bool value = false;
};

template <typename T>
struct IsPointer<T*> {
    static const bool value = true;
};

template <typename T>
struct IsPointer<const T*> {
    static const bool value = true;
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Full Specialization ===" << std::endl;
    
    Processor<double>::process(3.14159);
    Processor<int>::process(42);
    Processor<std::string>::process("Hello Templates");
    
    std::cout << "\n=== Partial Specialization ===" << std::endl;
    
    int x = 100;
    PointerHandler<int>::handle(&x);
    
    const int y = 200;
    PointerHandler<const int>::handle(&y);
    
    int arr[] = {1, 2, 3, 4, 5};
    ArrayHandler<int, 5>::handle(arr);
    
    std::cout << "\n=== Template Class Specialization ===" << std::endl;
    
    Pair<int, double> p1(10, 3.14);
    p1.print();
    
    Pair<int, int> p2(5, 10);
    p2.print();
    std::cout << "Max: " << p2.max() << std::endl;
    
    Pair<int, std::string> p3(1001, "John Doe");
    p3.print();
    
    std::cout << "\n=== Member Function Specialization ===" << std::endl;
    
    Container<int> intCont(42);
    intCont.print();
    
    Container<bool> boolCont(true);
    boolCont.print();
    
    std::cout << "\n=== Type Traits via Specialization ===" << std::endl;
    
    std::cout << "IsPointer<int>::value = " << IsPointer<int>::value << std::endl;
    std::cout << "IsPointer<int*>::value = " << IsPointer<int*>::value << std::endl;
    std::cout << "IsPointer<const double*>::value = " 
              << IsPointer<const double*>::value << std::endl;
    
    return 0;
}
```

---

### 75. What are variadic templates?

**Variadic templates** (C++11) are templates that can accept **any number of template arguments**. They enable type-safe functions like `printf` that work with arbitrary numbers of arguments.

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <tuple>

// ==================== Basic Variadic Template ====================

// Base case: no arguments
void print() {
    std::cout << std::endl;
}

// Recursive variadic template
template <typename T, typename... Args>
void print(T first, Args... rest) {
    std::cout << first;
    if (sizeof...(rest) > 0) {
        std::cout << ", ";
    }
    print(rest...);  // Recursive call
}

// ==================== Variadic Template with Different Separator ====================

template <typename... Args>
void printWithSeparator(const std::string& separator, Args... args) {
    printWithSeparatorImpl(separator, args...);
}

// Helper implementation
template <typename T, typename... Args>
void printWithSeparatorImpl(const std::string& sep, T first, Args... rest) {
    std::cout << first;
    if (sizeof...(rest) > 0) {
        std::cout << sep;
        printWithSeparatorImpl(sep, rest...);
    }
}

// Base case
void printWithSeparatorImpl(const std::string&) {
    std::cout << std::endl;
}

// ==================== Sum with Variadic Templates ====================

// Base case
template <typename T>
T sum(T value) {
    return value;
}

// Recursive case
template <typename T, typename... Args>
T sum(T first, Args... rest) {
    return first + sum(rest...);
}

// C++17 fold expression version (simpler)
template <typename... Args>
auto sumFold(Args... args) {
    return (args + ...);  // Unary right fold
}

// ==================== Tuple-like Operations ====================

// Apply function to each argument
template <typename Func, typename... Args>
void forEach(Func func, Args... args) {
    (func(args), ...);  // C++17 fold expression
}

// ==================== Variadic Class Template ====================

template <typename... Types>
class VariadicContainer {
private:
    std::tuple<Types...> data;
    
public:
    VariadicContainer(Types... args) : data(args...) {}
    
    void print() const {
        std::apply([](const auto&... args) {
            ((std::cout << args << " "), ...);
        }, data);
        std::cout << std::endl;
    }
    
    size_t size() const {
        return sizeof...(Types);
    }
};

// ==================== Type Traits with Variadic Templates ====================

// Check if all types are the same
template <typename T, typename... Rest>
struct AllSame {
    static const bool value = (std::is_same_v<T, Rest> && ...);
};

// Check if a type is in a parameter pack
template <typename T, typename... Pack>
struct Contains {
    static const bool value = (std::is_same_v<T, Pack> || ...);
};

// ==================== Advanced: printf-like functionality ====================

void variadicPrintf(const char* format) {
    std::cout << format;
}

template <typename T, typename... Args>
void variadicPrintf(const char* format, T value, Args... args) {
    while (*format) {
        if (*format == '%' && *(format + 1) == 'd') {
            std::cout << value;
            variadicPrintf(format + 2, args...);
            return;
        }
        std::cout << *format++;
    }
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Variadic Templates ===" << std::endl;
    
    print(1);
    print(1, 2, 3);
    print(1, 2.5, "hello", 'A');
    
    std::cout << "\n=== With Separator ===" << std::endl;
    
    printWithSeparatorImpl(" - ", 1, 2, 3);
    printWithSeparatorImpl(" | ", "apple", "banana", "cherry");
    
    std::cout << "\n=== Sum Functions ===" << std::endl;
    
    std::cout << "sum(1, 2, 3) = " << sum(1, 2, 3) << std::endl;
    std::cout << "sum(1.1, 2.2, 3.3) = " << sum(1.1, 2.2, 3.3) << std::endl;
    std::cout << "sumFold(1, 2, 3, 4) = " << sumFold(1, 2, 3, 4) << std::endl;
    
    std::cout << "\n=== forEach with Fold ===" << std::endl;
    
    auto printer = [](const auto& x) { std::cout << x << " "; };
    forEach(printer, 1, 2, 3, "hello", 4.5);
    std::cout << std::endl;
    
    std::cout << "\n=== Variadic Class Template ===" << std::endl;
    
    VariadicContainer<int, double, std::string> container(42, 3.14, "Hello");
    container.print();
    std::cout << "Size: " << container.size() << std::endl;
    
    std::cout << "\n=== Type Traits ===" << std::endl;
    
    std::cout << "AllSame<int, int, int>: " 
              << AllSame<int, int, int>::value << std::endl;
    std::cout << "AllSame<int, double, int>: " 
              << AllSame<int, double, int>::value << std::endl;
    
    std::cout << "Contains<int, double, char, int, float>: " 
              << Contains<int, double, char, int, float>::value << std::endl;
    std::cout << "Contains<std::string, int, double>: " 
              << Contains<std::string, int, double>::value << std::endl;
    
    std::cout << "\n=== Custom printf ===" << std::endl;
    
    variadicPrintf("Value = %d\n", 42);
    variadicPrintf("%d + %d = %d\n", 5, 3, 8);
    
    return 0;
}
```

---

### 76. What are template metaprogramming basics?

**Template Metaprogramming (TMP)** is a technique where templates are used to perform **compile-time computations**. It treats templates as a functional language embedded in C++.

```cpp
#include <iostream>
#include <type_traits>

// ==================== Compile-time Factorial ====================

// Recursive template
template <int N>
struct Factorial {
    static const int value = N * Factorial<N - 1>::value;
};

// Base case specialization
template <>
struct Factorial<0> {
    static const int value = 1;
};

// ==================== Compile-time Fibonacci ====================

template <int N>
struct Fibonacci {
    static const int value = Fibonacci<N - 1>::value + Fibonacci<N - 2>::value;
};

template <>
struct Fibonacci<0> {
    static const int value = 0;
};

template <>
struct Fibonacci<1> {
    static const int value = 1;
};

// ==================== Compile-time Power ====================

template <int Base, int Exponent>
struct Power {
    static const int value = Base * Power<Base, Exponent - 1>::value;
};

template <int Base>
struct Power<Base, 0> {
    static const int value = 1;
};

// ==================== Type Traits ====================

// Primary template
template <typename T>
struct IsPointer {
    static const bool value = false;
};

// Specialization for pointers
template <typename T>
struct IsPointer<T*> {
    static const bool value = true;
};

// ==================== Conditional Type Selection ====================

template <bool Condition, typename TrueType, typename FalseType>
struct IfThenElse {
    using type = TrueType;
};

template <typename TrueType, typename FalseType>
struct IfThenElse<false, TrueType, FalseType> {
    using type = FalseType;
};

// ==================== Compile-time List ====================

template <int... Values>
struct IntList;

template <int Head, int... Tail>
struct IntList<Head, Tail...> {
    static const int head = Head;
    using tail = IntList<Tail...>;
    static const int size = 1 + sizeof...(Tail);
};

template <>
struct IntList<> {
    static const int size = 0;
};

// Compile-time operations on lists
template <typename List>
struct Sum {
    static const int value = List::head + Sum<typename List::tail>::value;
};

template <>
struct Sum<IntList<>> {
    static const int value = 0;
};

// ==================== Compile-time Loop Unrolling ====================

template <int N>
struct Loop {
    template <typename Func>
    static void execute(Func func) {
        Loop<N - 1>::execute(func);
        func(N);
    }
};

template <>
struct Loop<0> {
    template <typename Func>
    static void execute(Func) {}
};

// ==================== Enable_if Metaprogramming ====================

template <typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
process(T value) {
    std::cout << "Processing integral: " << value << std::endl;
    return value * 2;
}

template <typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
process(T value) {
    std::cout << "Processing floating point: " << value << std::endl;
    return value * 1.5;
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Compile-time Computations ===" << std::endl;
    
    std::cout << "Factorial<5>::value = " << Factorial<5>::value << std::endl;
    std::cout << "Factorial<10>::value = " << Factorial<10>::value << std::endl;
    
    std::cout << "Fibonacci<10>::value = " << Fibonacci<10>::value << std::endl;
    
    std::cout << "Power<2, 10>::value = " << Power<2, 10>::value << std::endl;
    std::cout << "Power<3, 4>::value = " << Power<3, 4>::value << std::endl;
    
    std::cout << "\n=== Type Traits ===" << std::endl;
    
    std::cout << "IsPointer<int>::value = " << IsPointer<int>::value << std::endl;
    std::cout << "IsPointer<int*>::value = " << IsPointer<int*>::value << std::endl;
    std::cout << "IsPointer<const char*>::value = " 
              << IsPointer<const char*>::value << std::endl;
    
    std::cout << "\n=== Conditional Type Selection ===" << std::endl;
    
    using IntType = IfThenElse<true, int, double>::type;
    using DoubleType = IfThenElse<false, int, double>::type;
    
    std::cout << "IntType size: " << sizeof(IntType) << std::endl;
    std::cout << "DoubleType size: " << sizeof(DoubleType) << std::endl;
    
    std::cout << "\n=== Compile-time List ===" << std::endl;
    
    using MyList = IntList<1, 2, 3, 4, 5>;
    std::cout << "List size: " << MyList::size << std::endl;
    std::cout << "First element: " << MyList::head << std::endl;
    std::cout << "Sum of list: " << Sum<MyList>::value << std::endl;
    
    std::cout << "\n=== Compile-time Loop Unrolling ===" << std::endl;
    
    Loop<5>::execute([](int i) {
        std::cout << "Iteration " << i << std::endl;
    });
    
    std::cout << "\n=== enable_if Metaprogramming ===" << std::endl;
    
    process(42);
    process(3.14159);
    // process("Hello");  // Error: no matching function
    
    return 0;
}
```

---

### 77. What is std::is_base_of?

`std::is_base_of` is a **type trait** that checks at compile-time whether one type is derived from another. It's part of the `<type_traits>` header.

```cpp
#include <iostream>
#include <type_traits>

// ==================== Basic Inheritance Hierarchy ====================

class Animal {
public:
    virtual ~Animal() = default;
};

class Mammal : public Animal {
};

class Dog : public Mammal {
};

class Cat : public Mammal {
};

class Robot {
};

// ==================== Template Functions with is_base_of ====================

// Function that only accepts types derived from Animal
template <typename T>
typename std::enable_if<std::is_base_of<Animal, T>::value>::type
processAnimal(const T& animal) {
    std::cout << "Processing an animal" << std::endl;
}

// Overload for non-animals
template <typename T>
typename std::enable_if<!std::is_base_of<Animal, T>::value>::type
processAnimal(const T& notAnimal) {
    std::cout << "Not an animal" << std::endl;
}

// ==================== Checking Relationships ====================

template <typename Base, typename Derived>
void checkInheritance() {
    std::cout << std::boolalpha;
    std::cout << typeid(Base).name() << " is base of " 
              << typeid(Derived).name() << ": "
              << std::is_base_of<Base, Derived>::value << std::endl;
}

// ==================== Real-world Example: Smart Pointer Casting ====================

template <typename Derived, typename Base>
std::unique_ptr<Derived> dynamic_unique_ptr_cast(std::unique_ptr<Base>&& ptr) {
    static_assert(std::is_base_of<Base, Derived>::value, 
                  "Derived must be derived from Base");
    
    if (Derived* result = dynamic_cast<Derived*>(ptr.get())) {
        ptr.release();
        return std::unique_ptr<Derived>(result);
    }
    return nullptr;
}

// ==================== SFINAE with is_base_of ====================

// Only enable if T is derived from Animal
template <typename T>
auto makeSound(const T& creature) -> decltype(creature.sound(), void()) {
    std::cout << "Creature says: ";
    creature.sound();
}

// Base case for non-animals
void makeSound(...) {
    std::cout << "Unknown creature" << std::endl;
}

class SpeakingDog : public Animal {
public:
    void sound() const { std::cout << "Woof!" << std::endl; }
};

class NonSpeakingDog : public Animal {
    // No sound method
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic is_base_of Checks ===" << std::endl;
    
    std::cout << std::boolalpha;
    std::cout << "Animal is base of Mammal: " 
              << std::is_base_of<Animal, Mammal>::value << std::endl;
    std::cout << "Animal is base of Dog: " 
              << std::is_base_of<Animal, Dog>::value << std::endl;
    std::cout << "Mammal is base of Dog: " 
              << std::is_base_of<Mammal, Dog>::value << std::endl;
    std::cout << "Dog is base of Animal: " 
              << std::is_base_of<Dog, Animal>::value << std::endl;
    std::cout << "Animal is base of Robot: " 
              << std::is_base_of<Animal, Robot>::value << std::endl;
    
    std::cout << "\n=== Template Specialization with is_base_of ===" << std::endl;
    
    Dog dog;
    Robot robot;
    
    processAnimal(dog);
    processAnimal(robot);
    
    std::cout << "\n=== Dynamic Cast with is_base_of ===" << std::endl;
    
    std::unique_ptr<Animal> animalPtr = std::make_unique<Dog>();
    
    // Safe cast using is_base_of guarantee
    auto dogPtr = dynamic_unique_ptr_cast<Dog>(std::move(animalPtr));
    if (dogPtr) {
        std::cout << "Successfully cast to Dog" << std::endl;
    }
    
    std::cout << "\n=== Compile-time Assertions ===" << std::endl;
    
    // This would fail at compile time if uncommented
    // static_assert(std::is_base_of<Dog, Animal>::value, "Dog is not base of Animal");
    static_assert(std::is_base_of<Animal, Dog>::value, "Dog must be derived from Animal");
    
    std::cout << "Static assertions passed!" << std::endl;
    
    std::cout << "\n=== Practical Usage ===" << std::endl;
    
    SpeakingDog speakingDog;
    NonSpeakingDog nonSpeakingDog;
    
    makeSound(speakingDog);     // Uses SFINAE version
    makeSound(nonSpeakingDog);   // Uses fallback
    
    return 0;
}
```

---

### 78. How do templates relate to OOP?

Templates and OOP are **complementary** paradigms in C++. While OOP provides runtime polymorphism (virtual functions), templates provide compile-time polymorphism (generics).

```cpp
#include <iostream>
#include <memory>
#include <vector>

// ==================== OOP vs Templates Comparison ====================

// OOP Approach - Runtime Polymorphism
class Drawable {
public:
    virtual void draw() const = 0;
    virtual ~Drawable() = default;
};

class Circle : public Drawable {
public:
    void draw() const override {
        std::cout << "Drawing Circle (OOP)" << std::endl;
    }
};

class Square : public Drawable {
public:
    void draw() const override {
        std::cout << "Drawing Square (OOP)" << std::endl;
    }
};

// Template Approach - Compile-time Polymorphism
template <typename T>
void draw(const T& shape) {
    shape.draw();
}

class TemplateCircle {
public:
    void draw() const {
        std::cout << "Drawing Circle (Template)" << std::endl;
    }
};

class TemplateSquare {
public:
    void draw() const {
        std::cout << "Drawing Square (Template)" << std::endl;
    }
};

// ==================== Template + Inheritance ====================

// Templates can work with inheritance hierarchies
template <typename T>
class Container {
private:
    T value;
    
public:
    Container(const T& v) : value(v) {}
    
    void process() {
        value.operation();
    }
};

class Base {
public:
    virtual void operation() {
        std::cout << "Base operation" << std::endl;
    }
};

class Derived : public Base {
public:
    void operation() override {
        std::cout << "Derived operation" << std::endl;
    }
};

// ==================== CRTP (Curiously Recurring Template Pattern) ====================
// Combines templates and inheritance

template <typename Derived>
class ShapeBase {
public:
    void draw() const {
        static_cast<const Derived*>(this)->drawImpl();
    }
    
    double area() const {
        return static_cast<const Derived*>(this)->areaImpl();
    }
};

class CRTPCircle : public ShapeBase<CRTPCircle> {
private:
    double radius;
    
public:
    CRTPCircle(double r) : radius(r) {}
    
    void drawImpl() const {
        std::cout << "Drawing Circle (CRTP)" << std::endl;
    }
    
    double areaImpl() const {
        return 3.14159 * radius * radius;
    }
};

class CRTPRectangle : public ShapeBase<CRTPRectangle> {
private:
    double width, height;
    
public:
    CRTPRectangle(double w, double h) : width(w), height(h) {}
    
    void drawImpl() const {
        std::cout << "Drawing Rectangle (CRTP)" << std::endl;
    }
    
    double areaImpl() const {
        return width * height;
    }
};

// ==================== Policy-Based Design ====================
// Templates enable policy-based design

struct FastAllocPolicy {
    static void* allocate(size_t size) {
        std::cout << "Fast allocation" << std::endl;
        return malloc(size);
    }
    
    static void deallocate(void* ptr) {
        std::cout << "Fast deallocation" << std::endl;
        free(ptr);
    }
};

struct SafeAllocPolicy {
    static void* allocate(size_t size) {
        std::cout << "Safe allocation with bounds checking" << std::endl;
        void* ptr = malloc(size + sizeof(size_t));
        if (ptr) {
            *static_cast<size_t*>(ptr) = size;
            return static_cast<char*>(ptr) + sizeof(size_t);
        }
        return nullptr;
    }
    
    static void deallocate(void* ptr) {
        if (ptr) {
            void* original = static_cast<char*>(ptr) - sizeof(size_t);
            std::cout << "Safe deallocation, size was: " 
                      << *static_cast<size_t*>(original) << std::endl;
            free(original);
        }
    }
};

template <typename AllocPolicy = FastAllocPolicy>
class ManagedObject : private AllocPolicy {
private:
    void* data;
    
public:
    ManagedObject(size_t size) {
        data = AllocPolicy::allocate(size);
    }
    
    ~ManagedObject() {
        AllocPolicy::deallocate(data);
    }
};

// ==================== Type Erasure (Templates + OOP) ====================

class AnyDrawable {
private:
    struct Concept {
        virtual ~Concept() = default;
        virtual void draw() const = 0;
        virtual std::unique_ptr<Concept> clone() const = 0;
    };
    
    template <typename T>
    struct Model : Concept {
        T object;
        
        Model(T obj) : object(std::move(obj)) {}
        
        void draw() const override {
            object.draw();
        }
        
        std::unique_ptr<Concept> clone() const override {
            return std::make_unique<Model<T>>(object);
        }
    };
    
    std::unique_ptr<Concept> pimpl;
    
public:
    template <typename T>
    AnyDrawable(T obj) : pimpl(std::make_unique<Model<T>>(std::move(obj))) {}
    
    AnyDrawable(const AnyDrawable& other) : pimpl(other.pimpl->clone()) {}
    
    void draw() const {
        pimpl->draw();
    }
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== OOP vs Templates ===" << std::endl;
    
    // OOP: runtime polymorphism
    std::vector<std::unique_ptr<Drawable>> oopShapes;
    oopShapes.push_back(std::make_unique<Circle>());
    oopShapes.push_back(std::make_unique<Square>());
    
    std::cout << "OOP (runtime): ";
    for (const auto& shape : oopShapes) {
        shape->draw();
    }
    
    // Templates: compile-time polymorphism
    TemplateCircle tCircle;
    TemplateSquare tSquare;
    
    std::cout << "\nTemplates (compile-time): ";
    draw(tCircle);
    draw(tSquare);
    
    std::cout << "\n=== CRTP: Templates + Inheritance ===" << std::endl;
    
    CRTPCircle crtpCircle(5.0);
    CRTPRectangle crtpRect(4.0, 6.0);
    
    crtpCircle.draw();
    std::cout << "Area: " << crtpCircle.area() << std::endl;
    crtpRect.draw();
    std::cout << "Area: " << crtpRect.area() << std::endl;
    
    std::cout << "\n=== Policy-Based Design ===" << std::endl;
    
    ManagedObject<FastAllocPolicy> fastObj(100);
    ManagedObject<SafeAllocPolicy> safeObj(200);
    
    std::cout << "\n=== Type Erasure: Best of Both Worlds ===" << std::endl;
    
    std::vector<AnyDrawable> anyShapes;
    anyShapes.emplace_back(TemplateCircle());
    anyShapes.emplace_back(TemplateSquare());
    anyShapes.emplace_back(crtpCircle);  // Different types in same container!
    
    for (const auto& shape : anyShapes) {
        shape.draw();
    }
    
    std::cout << "\n=== Comparison Summary ===" << std::endl;
    std::cout << "OOP: Runtime flexibility, virtual function overhead\n";
    std::cout << "Templates: Compile-time binding, no overhead, more restrictive\n";
    std::cout << "CRTP: Static polymorphism, code reuse without virtual\n";
    std::cout << "Type Erasure: Runtime flexibility with template support\n";
    
    return 0;
}
```

---

### 79. What are concepts (C++20)?

**Concepts** (C++20) are a way to specify **constraints on template parameters**. They improve error messages and make templates safer and more expressive by defining requirements that template arguments must satisfy.

```cpp
#include <iostream>
#include <concepts>
#include <vector>
#include <string>
#include <type_traits>

// ==================== Basic Concepts ====================

// Define a simple concept
template <typename T>
concept Incrementable = requires(T x) {
    ++x;
    x++;
    { x += 1 } -> std::convertible_to<T>;
};

// Use concept to constrain template
template <Incrementable T>
T increment(T value) {
    return ++value;
}

// ==================== Built-in Concepts ====================

// Using standard concepts
template <std::integral T>
T doubleValue(T value) {
    return value * 2;
}

template <std::floating_point T>
T tripleValue(T value) {
    return value * 3;
}

// ==================== Compound Concepts ====================

template <typename T>
concept Printable = requires(std::ostream& os, T x) {
    { os << x } -> std::convertible_to<std::ostream&>;
};

template <typename T>
concept Comparable = requires(T a, T b) {
    { a == b } -> std::convertible_to<bool>;
    { a != b } -> std::convertible_to<bool>;
    { a < b } -> std::convertible_to<bool>;
};

template <typename T>
concept Container = requires(T c) {
    typename T::value_type;
    typename T::iterator;
    { c.begin() } -> std::same_as<typename T::iterator>;
    { c.end() } -> std::same_as<typename T::iterator>;
    { c.size() } -> std::convertible_to<size_t>;
};

// ==================== Combining Concepts ====================

template <typename T>
concept SortableContainer = Container<T> && requires(T c) {
    std::sort(c.begin(), c.end());
};

// ==================== Auto with Concepts ====================

void printPrintable(const Printable auto& value) {
    std::cout << value << std::endl;
}

// ==================== Requires Clause ====================

template <typename T>
    requires std::integral<T> || std::floating_point<T>
T add(T a, T b) {
    return a + b;
}

// ==================== Concept-based Overloading ====================

template <std::integral T>
std::string process(T value) {
    return "Integral: " + std::to_string(value);
}

template <std::floating_point T>
std::string process(T value) {
    return "Floating point: " + std::to_string(value);
}

template <Printable T>
std::string process(const T& value) {
    std::ostringstream oss;
    oss << value;
    return "Printable: " + oss.str();
}

// ==================== Custom Container Example ====================

template <typename T>
concept Hashable = requires(T a) {
    { std::hash<T>{}(a) } -> std::convertible_to<size_t>;
};

template <typename T>
concept KeyType = Hashable<T> && std::equality_comparable<T>;

template <KeyType Key, typename Value>
class SimpleMap {
    // Implementation
};

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Basic Concepts ===" << std::endl;
    
    std::cout << "increment(5): " << increment(5) << std::endl;
    // increment("hello");  // Error: string doesn't satisfy Incrementable
    
    std::cout << "\n=== Built-in Concepts ===" << std::endl;
    
    std::cout << "doubleValue(10): " << doubleValue(10) << std::endl;
    std::cout << "tripleValue(3.14): " << tripleValue(3.14) << std::endl;
    // doubleValue(3.14);  // Error: double not integral
    
    std::cout << "\n=== Auto with Concepts ===" << std::endl;
    
    printPrintable(42);
    printPrintable(3.14159);
    printPrintable("Hello World");
    // printPrintable(std::vector<int>{});  // Error: vector not printable
    
    std::cout << "\n=== Concept-based Overloading ===" << std::endl;
    
    std::cout << process(42) << std::endl;
    std::cout << process(3.14159) << std::endl;
    std::cout << process(std::string("Hello")) << std::endl;
    
    std::cout << "\n=== Container Concept ===" << std::endl;
    
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // Check if vector satisfies our concepts
    static_assert(Container<decltype(vec)>);
    static_assert(SortableContainer<decltype(vec)>);
    
    std::cout << "Vector satisfies Container concept!" << std::endl;
    
    std::cout << "\n=== Benefits of Concepts ===" << std::endl;
    std::cout << "1. Clearer error messages" << std::endl;
    std::cout << "2. Self-documenting code" << std::endl;
    std::cout << "3. Overloading based on type properties" << std::endl;
    std::cout << "4. Compile-time checking" << std::endl;
    
    return 0;
}
```

---

### 80. How to constrain templates?

Template constraints ensure that templates are only instantiated with types that meet specific requirements. C++20 concepts provide the most elegant way, but there are several techniques available.

```cpp
#include <iostream>
#include <type_traits>
#include <concepts>
#include <vector>
#include <list>

// ==================== Method 1: C++20 Concepts (Best) ====================

template <typename T>
concept Numeric = std::is_arithmetic_v<T>;

template <Numeric T>
T add_concept(T a, T b) {
    return a + b;
}

// ==================== Method 2: SFINAE with enable_if ====================

// Enable for integral types only
template <typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
add_enable_if(T a, T b) {
    return a + b;
}

// Alternative syntax (C++14)
template <typename T>
std::enable_if_t<std::is_integral_v<T>, T>
add_enable_if_v2(T a, T b) {
    return a + b;
}

// ==================== Method 3: static_assert (Eager) ====================

template <typename T>
T add_static_assert(T a, T b) {
    static_assert(std::is_arithmetic_v<T>, "T must be arithmetic type");
    return a + b;
}

// ==================== Method 4: Requires Clause (C++20) ====================

template <typename T>
    requires std::is_arithmetic_v<T>
T add_requires(T a, T b) {
    return a + b;
}

// ==================== Method 5: Tag Dispatching ====================

// Helper tags
struct integral_tag {};
struct floating_tag {};

// Overloads based on type traits
template <typename T>
T add_dispatch_impl(T a, T b, integral_tag) {
    std::cout << "Using integer addition" << std::endl;
    return a + b;
}

template <typename T>
T add_dispatch_impl(T a, T b, floating_tag) {
    std::cout << "Using floating point addition" << std::endl;
    return a + b;
}

// Dispatcher
template <typename T>
T add_dispatch(T a, T b) {
    using tag = typename std::conditional<
        std::is_integral<T>::value,
        integral_tag,
        floating_tag
    >::type;
    return add_dispatch_impl(a, b, tag{});
}

// ==================== Method 6: Void_t SFINAE (C++17) ====================

template <typename T, typename = void>
struct has_size : std::false_type {};

template <typename T>
struct has_size<T, std::void_t<decltype(std::declval<T>().size())>> 
    : std::true_type {};

template <typename T>
typename std::enable_if<has_size<T>::value, size_t>::type
get_size(const T& container) {
    return container.size();
}

// ==================== Method 7: Multiple Constraints ====================

template <typename T>
concept Sortable = requires(T& c) {
    std::sort(c.begin(), c.end());
    typename T::value_type;
    { c.begin() } -> std::same_as<typename T::iterator>;
};

template <typename T>
    requires Sortable<T> && std::copyable<typename T::value_type>
void sort_and_print(T& container) {
    std::sort(container.begin(), container.end());
    for (const auto& elem : container) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;
}

// ==================== Method 8: Concept with Multiple Requirements ====================

template <typename T>
concept AdvancedNumeric = requires(T a, T b) {
    requires std::is_arithmetic_v<T>;
    { a + b } -> std::convertible_to<T>;
    { a - b } -> std::convertible_to<T>;
    { a * b } -> std::convertible_to<T>;
    { a / b } -> std::convertible_to<T>;
};

template <AdvancedNumeric T>
T advanced_operation(T a, T b, T c) {
    return (a + b) * c;
}

// ==================== Main Demonstration ====================

int main() {
    std::cout << "=== Method 1: C++20 Concepts ===" << std::endl;
    std::cout << "add_concept(5, 3): " << add_concept(5, 3) << std::endl;
    std::cout << "add_concept(3.14, 2.86): " << add_concept(3.14, 2.86) << std::endl;
    // add_concept("hello", "world");  // Error: doesn't satisfy Numeric
    
    std::cout << "\n=== Method 2: enable_if ===" << std::endl;
    std::cout << "add_enable_if(10, 20): " << add_enable_if(10, 20) << std::endl;
    // add_enable_if(3.14, 2.86);  // Error: no matching function
    
    std::cout << "\n=== Method 3: static_assert ===" << std::endl;
    std::cout << "add_static_assert(7, 8): " << add_static_assert(7, 8) << std::endl;
    // add_static_assert("a", "b");  // Compile error with clear message
    
    std::cout << "\n=== Method 4: requires clause ===" << std::endl;
    std::cout << "add_requires(100, 200): " << add_requires(100, 200) << std::endl;
    
    std::cout << "\n=== Method 5: Tag dispatching ===" << std::endl;
    std::cout << "add_dispatch(15, 25): " << add_dispatch(15, 25) << std::endl;
    std::cout << "add_dispatch(1.5, 2.5): " << add_dispatch(1.5, 2.5) << std::endl;
    
    std::cout << "\n=== Method 6: void_t SFINAE ===" << std::endl;
    std::vector<int> vec = {1, 2, 3};
    std::list<int> lst = {4, 5, 6};
    
    std::cout << "get_size(vec): " << get_size(vec) << std::endl;
    // std::cout << "get_size(lst): " << get_size(lst) << std::endl;  // Error: list has no size()
    
    std::cout << "\n=== Method 7: Multiple constraints ===" << std::endl;
    std::vector<int> sort_vec = {5, 2, 8, 1, 9};
    sort_and_print(sort_vec);
    
    std::cout << "\n=== Method 8: Advanced concepts ===" << std::endl;
    std::cout << "advanced_operation(2, 3, 4): " << advanced_operation(2, 3, 4) << std::endl;
    
    std::cout << "\n=== Comparison of Methods ===" << std::endl;
    std::cout << "1. Concepts (C++20): Cleanest, best error messages\n";
    std::cout << "2. enable_if: Works in older C++, verbose\n";
    std::cout << "3. static_assert: Simple but eager (can't overload)\n";
    std::cout << "4. requires clause: Concepts alternative\n";
    std::cout << "5. Tag dispatching: Complex but powerful\n";
    
    return 0;
}
```

**Summary of Template Constraint Methods:**

| Method | C++ Version | Pros | Cons |
| :--- | :--- | :--- | :--- |
| **Concepts** | C++20 | Clean syntax, good errors | Requires C++20 |
| **enable_if** | C++11 | Widely available | Verbose, hard to read |
| **static_assert** | C++11 | Simple, clear errors | Can't overload, eager |
| **requires clause** | C++20 | Flexible | Requires C++20 |
| **Tag dispatching** | C++98 | Works in all versions | Complex, boilerplate |