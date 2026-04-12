# 📘 The Complete OOP & Low-Level Design (LLD) Masterclass
### From Zero to Interview-Ready SDE — A Classroom-Style Deep Dive

> **How to use this guide:** Read it top-to-bottom, like a course. Every section builds on the previous one. Code along as you go. Every ⚠️ is important, every 💡 is a tip, every 🔥 is an interview favourite.

---

# 📑 Table of Contents

- [Part 1: Programming Fundamentals](#part-1-programming-fundamentals)
- [Part 2: Core OOP Concepts](#part-2-core-oop-concepts)
- [Part 3: Advanced OOP Design](#part-3-advanced-oop-design)
- [Part 4: Low-Level Design (LLD)](#part-4-low-level-design)
- [Part 5: Design Patterns](#part-5-design-patterns)
- [Part 6: LLD Case Studies](#part-6-lld-case-studies)
- [Part 7: Advanced Topics](#part-7-advanced-topics)
- [Part 8: Interview Preparation](#part-8-interview-preparation)
- [Part 9: Practice Section](#part-9-practice-section)

---

# 📚 PART 1: Programming Fundamentals

## 1.1 What is Programming?

Programming is giving instructions to a computer to solve a problem. The computer executes these instructions literally — it does *exactly* what you say, nothing more.

**Three styles of programming:**

| Style | Idea | Example Language |
|---|---|---|
| Procedural | Step-by-step instructions, functions | C |
| Object-Oriented | Group data + behavior into objects | C++, Java, Python |
| Functional | Compute by applying pure functions | Haskell, parts of Python/JS |

### Why OOP Won (for large systems)

Procedural code works great for scripts. But when you have 10,000 lines of code across 50 files, you need structure. OOP gives you:
- **Modularity** — each class is a self-contained unit
- **Reusability** — inherit and extend without rewriting
- **Maintainability** — changing one class doesn't break everything else
- **Scalability** — add features by adding classes, not modifying old code

---

## 1.2 Memory Layout of a Program

Every running program has memory divided into regions:

```
High Address ┌─────────────────┐
             │   Stack         │  ← Local variables, function call frames
             │   (grows ↓)     │    Fast, auto-managed, limited size (~1–8MB)
             ├─────────────────┤
             │   (gap)         │
             ├─────────────────┤
             │   Heap          │  ← Dynamic allocation (new/malloc)
             │   (grows ↑)     │    Slow, manual management, large (GBs)
             ├─────────────────┤
             │   BSS Segment   │  ← Uninitialized global/static variables
             ├─────────────────┤
             │   Data Segment  │  ← Initialized global/static variables
             ├─────────────────┤
             │   Text Segment  │  ← Compiled code (read-only)
Low Address  └─────────────────┘
```

### Stack vs Heap

```cpp
void example() {
    int x = 10;           // Stack — auto-freed when function returns
    int* p = new int(20); // Heap — you must delete it!
    delete p;             // Manual cleanup
}
```

| Property | Stack | Heap |
|---|---|---|
| Allocation | Automatic | Manual (`new`/`delete`) |
| Speed | Very fast | Slower |
| Size | Limited (~MB) | Large (~GB) |
| Lifetime | Ends when scope ends | Until you `delete` |
| Fragmentation | No | Yes, over time |

---

## 1.3 Pointers and References

### Pointers

A pointer stores the **memory address** of another variable.

```cpp
int x = 42;
int* p = &x;      // p holds the address of x

cout << x;        // 42    — value of x
cout << &x;       // 0x7ffd... — address of x
cout << p;        // 0x7ffd... — same address (p stores it)
cout << *p;       // 42    — dereference: value at that address

*p = 100;         // Change x through the pointer
cout << x;        // 100
```

### References

A reference is an **alias** — another name for the same variable. It cannot be null or reassigned.

```cpp
int x = 42;
int& ref = x;    // ref IS x (same memory, different name)

ref = 100;
cout << x;       // 100 — x is changed through ref
```

### Pointer vs Reference

| | Pointer | Reference |
|---|---|---|
| Can be null | Yes (`int* p = nullptr`) | No — must bind to valid object |
| Can be reassigned | Yes | No — bound once, forever |
| Syntax | `*p`, `p->` | Same as regular variable |
| Use when | Optional ownership, nullable | Alias, function parameters |

---

## 1.4 Dynamic Memory Allocation

```cpp
// Single object
int* p = new int(42);
delete p;
p = nullptr;  // Good practice: null out after delete

// Array
int* arr = new int[10];
arr[0] = 5;
delete[] arr;  // Note: delete[] not delete for arrays!
arr = nullptr;
```

---

## 1.5 Common Memory Issues

### Memory Leak

You allocate memory but never free it. The program gradually eats up RAM.

```cpp
void leak() {
    int* p = new int(100);
    // Forgot to delete p!
    // When function returns, p is gone but the memory isn't freed
}
// Called 1 million times → 4MB+ leaked
```

### Dangling Pointer

A pointer that points to memory that has already been freed.

```cpp
int* p = new int(42);
delete p;      // Memory freed
cout << *p;    // ⚠️ UNDEFINED BEHAVIOR — memory was freed!
p = nullptr;   // Fix: null out after delete
```

### Double Free

Freeing the same memory twice — causes a crash.

```cpp
int* p = new int(42);
delete p;
delete p;  // ⚠️ CRASH — memory already freed
```

> 💡 **Modern C++ solution:** Use smart pointers (`unique_ptr`, `shared_ptr`) and these problems disappear. We'll cover them later.

---

# 📚 PART 2: Core OOP Concepts

## 2.1 Classes and Objects

### The Big Picture

A **class** is a blueprint. An **object** is a concrete instance built from that blueprint.

Think of a `Car` class as the engineering specifications. Every actual car on the road is an object created from those specs.

```cpp
class Car {
private:
    string brand;
    int speed;
    int fuel;

public:
    Car(string b, int s) : brand(b), speed(s), fuel(100) {}

    void accelerate(int amount) {
        speed += amount;
        fuel -= amount / 5;
    }

    void printStatus() {
        cout << brand << " | Speed: " << speed << " | Fuel: " << fuel << endl;
    }
};

int main() {
    Car tesla("Tesla", 0);
    Car bmw("BMW", 0);

    tesla.accelerate(60);
    bmw.accelerate(40);

    tesla.printStatus(); // Tesla | Speed: 60 | Fuel: 88
    bmw.printStatus();   // BMW | Speed: 40 | Fuel: 92
}
```

### Object Memory Layout

```
Stack:
  ┌──────────────┐
  │  tesla       │
  │  ┌─────────┐ │
  │  │ brand   │ │  ← string (8–32 bytes)
  │  │ speed   │ │  ← int (4 bytes)
  │  │ fuel    │ │  ← int (4 bytes)
  │  └─────────┘ │
  └──────────────┘

Member functions (accelerate, printStatus) are stored ONCE
in the code segment, shared by all objects.
```

> 🔥 **Interview:** "Where are member functions stored?" — In the code segment, shared by all objects. Not per-object.

### Static Members

Static members belong to the **class**, not to any individual object.

```cpp
class BankAccount {
private:
    double balance;
    static int totalAccounts;  // One copy for the entire class

public:
    BankAccount(double initial) : balance(initial) {
        totalAccounts++;
    }

    static int getTotalAccounts() {  // Static function: no 'this' pointer
        return totalAccounts;
    }
};

int BankAccount::totalAccounts = 0;  // Must define outside class

BankAccount a(1000), b(2000), c(500);
cout << BankAccount::getTotalAccounts(); // 3
```

### The `this` Pointer — Internal Working

Every non-static member function secretly receives a `this` pointer as a hidden first argument. It points to the object the function was called on.

```cpp
class Counter {
    int count;
public:
    Counter(int c) : count(c) {}

    // What you write:
    Counter& add(int n) {
        count += n;
        return *this;  // Return reference to current object
    }

    // What the compiler sees (roughly):
    // Counter& add(Counter* this, int n) {
    //     this->count += n;
    //     return *this;
    // }
};

Counter c(0);
c.add(5).add(3).add(2);  // Method chaining — returns *this each time
cout << c.count; // 10
```

### Const Objects and Const Member Functions

```cpp
class Circle {
    double radius;
public:
    Circle(double r) : radius(r) {}

    double area() const {         // 'const' — I promise not to modify any member
        return 3.14159 * radius * radius;
    }

    void setRadius(double r) {    // Non-const — modifies state
        radius = r;
    }
};

const Circle c(5);  // Const object — can only call const functions
c.area();           // ✅
c.setRadius(10);    // ❌ Error: calling non-const on const object
```

> 🔥 **Interview:** "Why mark a function const?" — It signals the function doesn't change state, allows calling on const objects, and enables compiler optimizations.

---

## 2.2 Constructors and Destructors

### The Full Lineup

```cpp
class String {
    char* data;
    int length;

public:
    // 1. Default constructor
    String() : data(nullptr), length(0) {
        cout << "Default constructor\n";
    }

    // 2. Parameterized constructor
    String(const char* s) {
        length = strlen(s);
        data = new char[length + 1];
        strcpy(data, s);
        cout << "Parameterized constructor: " << data << "\n";
    }

    // 3. Copy constructor (deep copy!)
    String(const String& other) {
        length = other.length;
        data = new char[length + 1];    // Allocate NEW memory
        strcpy(data, other.data);       // Copy content
        cout << "Copy constructor\n";
    }

    // 4. Destructor
    ~String() {
        delete[] data;
        cout << "Destructor for: " << (data ? data : "empty") << "\n";
    }
};
```

### Initialization List — Why It Matters

```cpp
class Person {
    string name;
    const int id;   // const member — MUST be initialized in init list
    int& ref;       // reference member — MUST be in init list

public:
    // ✅ Correct — uses initialization list
    Person(string n, int i, int& r) : name(n), id(i), ref(r) {}

    // ❌ Won't compile — can't assign to const/reference in body
    // Person(string n, int i, int& r) {
    //     name = n;   // This is assignment, not initialization
    //     id = i;     // Error: id is const
    // }
};
```

**Performance:** For class-type members, the init list calls the constructor directly. Without it, the default constructor runs first, then assignment happens — wasteful.

### Order of Construction and Destruction

```cpp
class A {
public:
    A() { cout << "A constructed\n"; }
    ~A() { cout << "A destroyed\n"; }
};

class B {
public:
    B() { cout << "B constructed\n"; }
    ~B() { cout << "B destroyed\n"; }
};

class C : public A {  // Inherits A
    B member;         // Has a B member
public:
    C() { cout << "C constructed\n"; }
    ~C() { cout << "C destroyed\n"; }
};

C obj;
// Output:
// A constructed    ← Base class first
// B constructed    ← Members in declaration order
// C constructed    ← Derived class last
// C destroyed      ← Destruction: REVERSE order
// B destroyed
// A destroyed
```

### Deep vs Shallow Copy — The Critical Difference

```cpp
class ShallowProblem {
    int* data;
public:
    ShallowProblem(int val) { data = new int(val); }

    // DEFAULT copy constructor does SHALLOW copy:
    // Just copies the pointer, not what it points to!
    // ShallowProblem(const ShallowProblem& other) : data(other.data) {}
    //                                                    ^^^^^^^^^^^
    //                                      Both objects now share the SAME int!
};

ShallowProblem a(42);
ShallowProblem b = a;   // b.data == a.data (SAME address)
// When a is destroyed, deletes the int
// When b is destroyed, tries to delete the SAME int → CRASH (double free)
```

```cpp
class DeepCorrect {
    int* data;
public:
    DeepCorrect(int val) { data = new int(val); }

    // Deep copy: allocate NEW memory and copy content
    DeepCorrect(const DeepCorrect& other) {
        data = new int(*other.data);  // New allocation, same value
    }

    ~DeepCorrect() { delete data; }
};

DeepCorrect a(42);
DeepCorrect b = a;   // b.data != a.data, but *b.data == *a.data = 42
// Each object has its own int — no sharing, no crashes
```

```
Shallow Copy:             Deep Copy:
┌───┐                    ┌───┐    ┌───┐
│ a │──┐                 │ a │───▶│42 │
└───┘  │  ┌────┐         └───┘    └───┘
       └─▶│ 42 │         ┌───┐    ┌───┐
┌───┐  ┌─▶└────┘         │ b │───▶│42 │  (different memory)
│ b │──┘                 └───┘    └───┘
└───┘
  Both point to same memory!
```

---

## 2.3 Encapsulation

### The Core Idea

Encapsulation = bundling data + methods + controlling who can touch what.

**Real-world:** Your phone's battery. You can't directly touch the cells — you plug in a charger (public interface). The battery management system (private logic) handles charging safely.

```cpp
class Temperature {
private:
    double celsius;   // Raw data — hidden

    bool isValid(double c) { return c >= -273.15; }  // Internal helper

public:
    // Controlled access
    void setCelsius(double c) {
        if (!isValid(c))
            throw invalid_argument("Temperature below absolute zero!");
        celsius = c;
    }

    double getCelsius() const { return celsius; }
    double getFahrenheit() const { return celsius * 9.0/5.0 + 32; }
    double getKelvin() const { return celsius + 273.15; }
};

Temperature t;
t.setCelsius(-300);    // ❌ Throws exception — prevented!
t.setCelsius(100);     // ✅
cout << t.getFahrenheit(); // 212
```

### Friend Functions and Classes

Sometimes an external function needs access to private members (e.g., operator overloading).

```cpp
class Vector {
    double x, y;
public:
    Vector(double x, double y) : x(x), y(y) {}

    // Friend function — can access private x, y
    friend ostream& operator<<(ostream& os, const Vector& v);
    friend double dotProduct(const Vector& a, const Vector& b);
};

ostream& operator<<(ostream& os, const Vector& v) {
    os << "(" << v.x << ", " << v.y << ")";  // Accesses private!
    return os;
}

double dotProduct(const Vector& a, const Vector& b) {
    return a.x * b.x + a.y * b.y;  // Accesses private!
}

Vector v1(3, 4), v2(1, 2);
cout << v1;                     // (3, 4)
cout << dotProduct(v1, v2);     // 11
```

> ⚠️ **Use friend sparingly.** It breaks encapsulation. Prefer using public methods. Use friend only for operators and closely related classes.

---

## 2.4 Abstraction

### What and Why

Abstraction = expose **what** an object does, hide **how** it does it.

When you call `sort(vec.begin(), vec.end())`, you don't know (or care) if it uses introsort, heapsort, or quicksort internally. That's abstraction.

### Abstract Classes (Pure Virtual Functions)

```cpp
// Abstract class — defines the CONTRACT
class PaymentProcessor {
public:
    virtual bool processPayment(double amount) = 0;  // Pure virtual
    virtual string getGatewayName() const = 0;
    virtual ~PaymentProcessor() {}  // Virtual destructor!

    // Can have concrete methods too
    void logTransaction(double amount) {
        cout << "Processing " << amount << " via " << getGatewayName() << "\n";
    }
};

// Concrete implementations
class StripeProcessor : public PaymentProcessor {
    string apiKey;
public:
    StripeProcessor(string key) : apiKey(key) {}
    bool processPayment(double amount) override {
        // Stripe-specific logic
        cout << "Stripe charging $" << amount << "\n";
        return true;
    }
    string getGatewayName() const override { return "Stripe"; }
};

class PayPalProcessor : public PaymentProcessor {
public:
    bool processPayment(double amount) override {
        cout << "PayPal charging $" << amount << "\n";
        return true;
    }
    string getGatewayName() const override { return "PayPal"; }
};

// User code doesn't know which processor is behind the pointer
void checkout(PaymentProcessor* processor, double amount) {
    processor->logTransaction(amount);
    processor->processPayment(amount);
}

PaymentProcessor* p = new StripeProcessor("sk_live_xxx");
checkout(p, 99.99);
```

### Interface vs Abstract Class

| Feature | Interface (Java) | Abstract Class (C++) |
|---|---|---|
| Data members | No (Java) | Yes |
| Constructor | No | Yes |
| Implementation | No (default in Java 8+) | Yes (partial OK) |
| Multiple inheritance | Yes | Careful (diamond problem) |
| Use when | Pure contract | Partial implementation + contract |

```java
// Java interface
interface Drawable {
    void draw();          // implicitly public abstract
    default void highlight() { /* can have default */ }
}

// Java abstract class
abstract class Shape {
    protected String color;  // Has state
    public abstract double area();  // Must implement
    public void setColor(String c) { color = c; } // Concrete method
}
```

---

## 2.5 Inheritance — In Depth

### Why Inheritance?

Without it, you'd copy-paste common code everywhere. Change something? Fix it in 10 places. With inheritance, fix it once in the base class.

### Access Control in Inheritance

```cpp
class Base {
public:    int pub;
protected: int prot;
private:   int priv;
};

// Public inheritance (most common) — IS-A relationship
class PubDerived : public Base {
    void test() {
        pub = 1;   // ✅ still public
        prot = 2;  // ✅ still protected
        priv = 3;  // ❌ Error: private in base
    }
};

// Protected inheritance — rarely used
class ProtDerived : protected Base {
    // pub and prot both become protected here
};

// Private inheritance — implementation inheritance (not IS-A)
class PrivDerived : private Base {
    // pub and prot both become private here
};
```

> 🔥 **Interview:** "What's the difference between public, protected, private inheritance?" — Public: IS-A (most common). Protected/Private: implementation detail, not a true IS-A relationship.

### Method Overriding

```cpp
class Animal {
public:
    virtual void speak() {
        cout << "Some animal sound\n";
    }

    virtual ~Animal() {}
};

class Dog : public Animal {
public:
    void speak() override {  // 'override' keyword — compiler checks you're actually overriding
        cout << "Woof!\n";
    }
};

class Cat : public Animal {
public:
    void speak() override {
        cout << "Meow!\n";
    }
};
```

### The Diamond Problem — Full Explanation

```
        Animal
       /      \
     Dog      Cat
       \      /
        DogCat
```

```cpp
class Animal {
public:
    int age = 5;
    void breathe() { cout << "Breathing\n"; }
};

class Dog : public Animal {};
class Cat : public Animal {};

class DogCat : public Dog, public Cat {
    // Has TWO Animal sub-objects!
    // Dog::Animal and Cat::Animal are separate
};

DogCat dc;
dc.breathe();     // ❌ AMBIGUOUS — Dog::breathe or Cat::breathe?
dc.Dog::breathe(); // ✅ Explicit resolution
dc.age;           // ❌ AMBIGUOUS — Dog::age or Cat::age?
```

**Solution: Virtual Inheritance**

```cpp
class Animal { public: int age = 5; };

class Dog : virtual public Animal {};  // 'virtual' keyword
class Cat : virtual public Animal {};  // 'virtual' keyword

class DogCat : public Dog, public Cat {
    // NOW there's only ONE Animal sub-object, shared between Dog and Cat
};

DogCat dc;
dc.age;      // ✅ Only one copy, no ambiguity
dc.breathe(); // ✅ Unambiguous
```

> ⚠️ Virtual inheritance adds a hidden pointer (vbptr) to track the shared base. Slight performance overhead. Use only when needed.

### Ambiguity Resolution

```cpp
class A { public: void foo() { cout << "A::foo\n"; } };
class B { public: void foo() { cout << "B::foo\n"; } };

class C : public A, public B {
public:
    void foo() {            // Override to resolve ambiguity
        A::foo();           // OR explicitly call one parent's version
    }
};

C c;
c.foo();       // C::foo (calls A::foo inside)
c.A::foo();    // Explicit
c.B::foo();    // Explicit
```

---

## 2.6 Polymorphism — Deep Dive

### Compile-Time: Function Overloading

The compiler picks the right function based on argument types at compile time.

```cpp
class Math {
public:
    int    max(int a, int b)       { return a > b ? a : b; }
    double max(double a, double b) { return a > b ? a : b; }
    int    max(int a, int b, int c){ return max(max(a,b), c); }
};

Math m;
m.max(3, 5);        // int version
m.max(3.14, 2.71);  // double version
m.max(1, 5, 3);     // 3-arg version
```

### Compile-Time: Operator Overloading

```cpp
class Matrix {
    int data[2][2];
public:
    Matrix(int a, int b, int c, int d) {
        data[0][0]=a; data[0][1]=b;
        data[1][0]=c; data[1][1]=d;
    }

    Matrix operator+(const Matrix& other) const {
        return Matrix(
            data[0][0]+other.data[0][0], data[0][1]+other.data[0][1],
            data[1][0]+other.data[1][0], data[1][1]+other.data[1][1]
        );
    }

    bool operator==(const Matrix& other) const {
        for(int i=0;i<2;i++)
            for(int j=0;j<2;j++)
                if(data[i][j] != other.data[i][j]) return false;
        return true;
    }

    friend ostream& operator<<(ostream& os, const Matrix& m) {
        os << m.data[0][0] << " " << m.data[0][1] << "\n"
           << m.data[1][0] << " " << m.data[1][1];
        return os;
    }
};

Matrix a(1,2,3,4), b(5,6,7,8);
Matrix c = a + b;
cout << c;
```

### Runtime Polymorphism — The vtable Explained

This is how C++ knows which `speak()` to call when you have an `Animal*` pointing to a `Dog`.

```
Memory layout of objects with virtual functions:

Animal object:                Dog object:
┌──────────────┐             ┌──────────────┐
│ vptr ────────┼──▶ Animal vtable:
│ (data...)    │    │ [0] speak → Animal::speak │
└──────────────┘    │ [1] ~Animal → Animal::~Animal │
                    └──────────────────────────────┘

                             │ vptr ────────┼──▶ Dog vtable:
                             │ (data...)    │    │ [0] speak → Dog::speak │  ← OVERRIDDEN
                             └──────────────┘    │ [1] ~Dog → Dog::~Dog   │
                                                 └──────────────────────────────┘
```

When you call `animal_ptr->speak()`:
1. Follow `vptr` to get the vtable
2. Look up index 0 (speak) in the vtable
3. Call whatever function is at that address

For a `Dog*`, index 0 points to `Dog::speak`. For an `Animal*`, it points to `Animal::speak`. **Same call, different behavior — that's runtime polymorphism.**

```cpp
Animal* zoo[3] = { new Animal(), new Dog(), new Cat() };

for (auto* a : zoo) {
    a->speak();   // Correct version called for each!
}
// Output: Some animal sound / Woof! / Meow!
```

### Early vs Late Binding

| | Early Binding | Late Binding |
|---|---|---|
| When decided | Compile time | Runtime |
| How | Function call resolved by compiler | Via vtable lookup |
| Performance | Faster | Slight overhead (one pointer dereference) |
| Flexibility | Less | More (polymorphism) |
| Keyword | (default) | `virtual` |

> ⚠️ **Always make base class destructors virtual** when using polymorphism. Otherwise, `delete animal_ptr` where `animal_ptr` points to a `Dog` won't call `Dog::~Dog()` — resource leak!

---

## 2.7 Rule of 3 / 5 / 0

### Rule of 3

If you define any of:
- Destructor
- Copy constructor
- Copy assignment operator

...you almost certainly need all three.

```cpp
class Buffer {
    char* data;
    size_t size;

public:
    // Constructor
    Buffer(size_t sz) : size(sz), data(new char[sz]) {}

    // RULE OF 3:

    // 1. Destructor
    ~Buffer() {
        delete[] data;
    }

    // 2. Copy constructor
    Buffer(const Buffer& other) : size(other.size), data(new char[other.size]) {
        memcpy(data, other.data, size);
    }

    // 3. Copy assignment operator
    Buffer& operator=(const Buffer& other) {
        if (this == &other) return *this;  // Self-assignment guard

        delete[] data;              // Free old resource
        size = other.size;
        data = new char[size];      // Allocate new
        memcpy(data, other.data, size); // Copy content

        return *this;
    }
};
```

### Rule of 5 (C++11 — Move Semantics)

Add:
- Move constructor
- Move assignment operator

Move = "steal" the resources instead of copying them. Used when the source object won't be needed anymore (temporaries, `std::move`).

```cpp
// 4. Move constructor — steal from 'other', leave it in valid empty state
Buffer(Buffer&& other) noexcept : size(other.size), data(other.data) {
    other.data = nullptr;   // 'other' is now empty (valid but unspecified)
    other.size = 0;
}

// 5. Move assignment operator
Buffer& operator=(Buffer&& other) noexcept {
    if (this == &other) return *this;

    delete[] data;          // Free what we have

    data = other.data;      // Steal other's resources
    size = other.size;

    other.data = nullptr;   // Empty 'other'
    other.size = 0;

    return *this;
}

// Usage:
Buffer b1(1024);
Buffer b2 = std::move(b1);  // Move — no allocation! b1 is now empty.
```

### Rule of 0

If you use RAII wrappers (smart pointers, containers), define NONE of the above. Let the compiler generate them correctly.

```cpp
class ModernBuffer {
    unique_ptr<char[]> data;
    size_t size;

public:
    ModernBuffer(size_t sz) : size(sz), data(make_unique<char[]>(sz)) {}

    // No destructor needed — unique_ptr handles it
    // No copy constructor needed (unique_ptr is non-copyable → ModernBuffer is non-copyable)
    // No move — unique_ptr is movable → ModernBuffer is movable
};

ModernBuffer b1(1024);
ModernBuffer b2 = std::move(b1);  // Works!
// ModernBuffer b3 = b1;          // ❌ Won't compile — intentionally non-copyable
```

---

# 📚 PART 3: Advanced OOP Design Concepts

## 3.1 Object Relationships

### Association

Objects know about each other but neither owns the other.

```
Teacher ─────── Student
(teaches)    (enrolls in)
Both can exist independently
```

```cpp
class Student;

class Teacher {
    vector<Student*> students;  // Knows about students, doesn't own them
public:
    void teach(Student* s) { students.push_back(s); }
};

class Student {
    string name;
public:
    Student(string n) : name(n) {}
};
```

### Aggregation — "Has-A" (Weak Ownership)

The parent has the child, but the child can exist independently.

```
Department ◇──── Professor
(has)         (can exist without department)
```

```cpp
class Professor {
    string name;
public:
    Professor(string n) : name(n) {}
};

class Department {
    vector<Professor*> profs;  // Pointer — department doesn't own professors
public:
    void addProfessor(Professor* p) { profs.push_back(p); }
    // When Department is destroyed, Professors still exist
};
```

### Composition — "Has-A" (Strong Ownership)

The parent owns the child. Child cannot exist without parent.

```
House ●──── Room
(owns)    (cannot exist without house)
```

```cpp
class Room {
    int number;
    double area;
public:
    Room(int n, double a) : number(n), area(a) {}
};

class House {
    vector<Room> rooms;  // By value — House owns the Rooms
public:
    void addRoom(int num, double area) {
        rooms.emplace_back(num, area);
    }
    // When House is destroyed, Rooms are destroyed with it
};
```

### Dependency

One class uses another temporarily (method parameter / local variable).

```cpp
class OrderProcessor {
public:
    void process(PaymentGateway& gateway, Order& order) {  // Temporary use
        gateway.charge(order.total());
    }
};
```

---

## 3.2 Coupling and Cohesion

### Coupling — How Tightly Connected Are Classes?

**Low coupling is GOOD.** Classes should know as little as possible about each other.

```cpp
// High coupling (BAD):
class UserService {
    MySQLDatabase db;    // Directly depends on concrete class
    SmtpMailer mailer;   // Can't swap either without modifying UserService
};

// Low coupling (GOOD):
class UserService {
    IDatabase& db;       // Depends on abstraction
    IMailer& mailer;     // Can swap implementation anytime
public:
    UserService(IDatabase& d, IMailer& m) : db(d), mailer(m) {}
};
```

### Cohesion — How Focused Is a Class?

**High cohesion is GOOD.** A class should have one clear purpose.

```cpp
// Low cohesion (BAD):
class God {
    void parseXML() {}
    void saveToDatabase() {}
    void sendEmail() {}
    void renderUI() {}
    // Does everything — change for any reason
};

// High cohesion (GOOD):
class XMLParser { void parse() {} };
class UserRepository { void save() {} };
class EmailService { void send() {} };
class UIRenderer { void render() {} };
```

---

## 3.3 SOLID Principles — Deep Dive

---

### S — Single Responsibility Principle (SRP)

> **A class should have only ONE reason to change.**

Every class should have one job. If a class changes for multiple reasons, it's doing too much.

**Bad Design ❌**
```cpp
class Employee {
public:
    string name;
    double salary;

    // Reason 1: Business logic changes
    double calculatePay() { return salary * 1.2; }

    // Reason 2: DB schema changes
    void saveToDatabase() { /* SQL logic */ }

    // Reason 3: Report format changes
    string generateReport() { return "Employee: " + name; }
};
```

**Good Design ✅**
```cpp
class Employee {
public:
    string name;
    double salary;
    double calculatePay() { return salary * 1.2; }
};

class EmployeeRepository {
public:
    void save(const Employee& e) { /* DB logic only */ }
    Employee findById(int id) { /* query */ }
};

class EmployeeReporter {
public:
    string generate(const Employee& e) { return "Employee: " + e.name; }
};
```

**Real-world analogy:** A chef cooks. A waiter serves. A cashier bills. One person, one responsibility.

---

### O — Open/Closed Principle (OCP)

> **Open for extension, closed for modification.**

You should be able to add new behavior WITHOUT changing existing code.

**Bad Design ❌**
```cpp
class NotificationService {
public:
    void send(string type, string message) {
        if (type == "email") {
            // send email
        } else if (type == "sms") {
            // send SMS
        } else if (type == "push") {
            // send push — had to MODIFY this class to add
        }
        // Every new channel requires modifying this!
    }
};
```

**Good Design ✅**
```cpp
class Notifier {
public:
    virtual void send(const string& message) = 0;
    virtual ~Notifier() {}
};

class EmailNotifier : public Notifier {
    void send(const string& msg) override { cout << "Email: " << msg; }
};

class SMSNotifier : public Notifier {
    void send(const string& msg) override { cout << "SMS: " << msg; }
};

// Add WhatsApp? Just add a new class!
class WhatsAppNotifier : public Notifier {
    void send(const string& msg) override { cout << "WhatsApp: " << msg; }
};

class NotificationService {
    vector<Notifier*> notifiers;
public:
    void addNotifier(Notifier* n) { notifiers.push_back(n); }
    void sendAll(const string& msg) {
        for (auto* n : notifiers) n->send(msg);
    }
};
```

**Real-world analogy:** USB ports. You can plug in new devices (open for extension) without rewiring the motherboard (closed for modification).

---

### L — Liskov Substitution Principle (LSP)

> **Subclasses must be substitutable for their parent class without breaking behavior.**

If you have code that works with a `Base*`, it must work correctly with `Derived*` too — without surprising behavior.

**Bad Design ❌**
```cpp
class Rectangle {
protected:
    int width, height;
public:
    virtual void setWidth(int w) { width = w; }
    virtual void setHeight(int h) { height = h; }
    int area() { return width * height; }
};

class Square : public Rectangle {
public:
    void setWidth(int w) override {
        width = height = w;   // Square must keep them equal
    }
    void setHeight(int h) override {
        width = height = h;   // Same here
    }
};

// Test that works for Rectangle breaks for Square:
void testRectangle(Rectangle* r) {
    r->setWidth(5);
    r->setHeight(10);
    assert(r->area() == 50);  // ✅ Rectangle passes, ❌ Square: 10*10=100!
}
```

**Good Design ✅**
```cpp
class Shape {
public:
    virtual int area() = 0;
    virtual ~Shape() {}
};

class Rectangle : public Shape {
    int width, height;
public:
    Rectangle(int w, int h) : width(w), height(h) {}
    int area() override { return width * height; }
};

class Square : public Shape {
    int side;
public:
    Square(int s) : side(s) {}
    int area() override { return side * side; }
};
// No inheritance between Rectangle and Square — they're separate shapes
```

**Real-world analogy:** A substitute teacher should be able to handle the class just like the regular teacher. If they instead set the room on fire, that violates LSP.

---

### I — Interface Segregation Principle (ISP)

> **Clients should not be forced to depend on interfaces they don't use.**

Don't create fat interfaces. Break them into smaller, focused ones.

**Bad Design ❌**
```cpp
class IWorker {
public:
    virtual void work() = 0;
    virtual void eat() = 0;
    virtual void sleep() = 0;
    virtual void takeCoffeeBreak() = 0;
};

class Robot : public IWorker {
    void work() override { /* OK */ }
    void eat() override { /* Robots don't eat! Empty/throw */ }
    void sleep() override { /* Robots don't sleep! Empty/throw */ }
    void takeCoffeeBreak() override { /* What? */ }
};
```

**Good Design ✅**
```cpp
class IWorkable   { public: virtual void work() = 0; };
class IEatable    { public: virtual void eat() = 0; };
class ISleepable  { public: virtual void sleep() = 0; };

class Human : public IWorkable, public IEatable, public ISleepable {
    void work() override { /* works */ }
    void eat() override { /* eats */ }
    void sleep() override { /* sleeps */ }
};

class Robot : public IWorkable {
    void work() override { /* works 24/7, no food, no sleep */ }
};
```

**Real-world analogy:** A TV remote has buttons for volume, channels, power. It doesn't have a button to control your AC — that's a separate remote. Focused interfaces.

---

### D — Dependency Inversion Principle (DIP)

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

**Bad Design ❌**
```cpp
// Low-level module
class MySQLDatabase {
public:
    void query(string sql) { /* MySQL specific */ }
};

// High-level module depends directly on low-level
class UserService {
    MySQLDatabase db;  // Tightly coupled!
public:
    User getUser(int id) {
        db.query("SELECT * FROM users WHERE id = " + to_string(id));
    }
};
// Want to switch to PostgreSQL? Rewrite UserService!
```

**Good Design ✅**
```cpp
// Abstraction (interface)
class IDatabase {
public:
    virtual void query(const string& sql) = 0;
    virtual ~IDatabase() {}
};

// Low-level modules implement the abstraction
class MySQLDatabase : public IDatabase {
    void query(const string& sql) override { /* MySQL */ }
};

class PostgreSQLDatabase : public IDatabase {
    void query(const string& sql) override { /* PostgreSQL */ }
};

// High-level module depends only on abstraction
class UserService {
    IDatabase& db;  // Depends on interface!
public:
    UserService(IDatabase& d) : db(d) {}  // Inject dependency
    User getUser(int id) {
        db.query("SELECT * FROM users WHERE id = " + to_string(id));
    }
};

// Switch database by changing injection, not UserService:
MySQLDatabase mysql;
UserService service(mysql);
// or:
PostgreSQLDatabase pg;
UserService service2(pg);
```

**Real-world analogy:** A lamp plugs into a standard wall socket (abstraction). The lamp doesn't care if the power comes from a coal plant or solar panels (low-level implementations).

---

# 📚 PART 4: Low-Level Design (LLD)

## 4.1 What is LLD?

**High-Level Design (HLD):** The big picture. What components exist, how they communicate, what databases and services are used. Think: "We'll have a User Service, an Order Service, a Payment Service connected via message queue."

**Low-Level Design (LLD):** The detailed blueprint. What classes, what methods, what relationships, what data structures. Think: "The `Order` class has fields `id`, `items[]`, `status`. It has methods `addItem()`, `removeItem()`, `calculateTotal()`."

```
HLD:                           LLD:
┌────────────────┐             ┌──────────────────────────────────┐
│  User Service  │             │  class User {                    │
│   (microservice)│            │    string id, name, email;       │
│       ↕        │             │    Address address;              │
│ Order Service  │             │    vector<Order*> orderHistory;  │
│   (microservice)│            │    void placeOrder(Cart& cart);  │
│       ↕        │             │  };                              │
│ Payment Service│             │                                  │
└────────────────┘             │  class Order { ... };            │
                               └──────────────────────────────────┘
```

## 4.2 Design Thinking Process

### Step 1: Understand Requirements

Ask clarifying questions. Don't jump to code.

- What are the **functional requirements**? (What must it do?)
- What are the **non-functional requirements**? (Scale, performance, reliability?)
- What are the **constraints**? (Read-heavy? Write-heavy?)

### Step 2: Identify Entities

Find the nouns in the requirements. Each major noun is probably a class.

> "Users can browse a **catalog** of **books**, add them to a **cart**, place **orders**, and make **payments**."

Entities: `User`, `Book`, `Catalog`, `Cart`, `Order`, `Payment`

### Step 3: Identify Relationships

- IS-A → inheritance
- HAS-A → composition/aggregation
- USES-A → association/dependency

### Step 4: Define Responsibilities

For each class: what data does it hold? What operations does it perform?

### Step 5: Define Interfaces

What's the public API of each class? What's hidden?

### Step 6: Iterate

First design is never perfect. Refine based on SOLID principles.

---

## 4.3 UML Basics

### Class Diagram Notation

```
┌─────────────────────────┐
│        ClassName         │  ← Class name
├─────────────────────────┤
│  - privateField: Type    │  ← '-' = private
│  # protectedField: Type  │  ← '#' = protected
│  + publicField: Type     │  ← '+' = public
├─────────────────────────┤
│  + method(): ReturnType  │  ← Methods
│  - helper(): void        │
└─────────────────────────┘

Relationships:
A ────────── B   Association (A uses B)
A ◇────────── B   Aggregation (A has B, B independent)
A ●────────── B   Composition (A owns B, B dies with A)
A ──────────▷ B   Inheritance (A extends B)
A - - - - -▷ B   Implements interface B
```

---

# 📚 PART 5: Design Patterns

Design patterns are **proven solutions to recurring design problems**. They're a shared vocabulary among developers.

> 💡 **Don't memorize patterns. Understand the problem each solves.**

---

## 5.1 Creational Patterns

### Singleton

**Problem:** You need exactly one instance of a class (config manager, logger, connection pool).

**Solution:** The class controls its own instantiation.

```
┌─────────────────────────┐
│       Singleton          │
├─────────────────────────┤
│ - instance: Singleton*   │
├─────────────────────────┤
│ - Singleton()            │ ← Private constructor
│ + getInstance(): Singleton* │
└─────────────────────────┘
```

```cpp
class Logger {
private:
    static Logger* instance;
    static mutex mtx;
    ofstream logFile;

    Logger() {
        logFile.open("app.log", ios::app);
    }

public:
    // Delete copy and move
    Logger(const Logger&) = delete;
    Logger& operator=(const Logger&) = delete;

    // Thread-safe getInstance (C++11 local static is thread-safe)
    static Logger& getInstance() {
        static Logger instance;  // Guaranteed thread-safe in C++11!
        return instance;
    }

    void log(const string& msg) {
        lock_guard<mutex> lock(mtx);
        logFile << "[LOG] " << msg << "\n";
    }

    ~Logger() { logFile.close(); }
};

// Usage:
Logger::getInstance().log("Application started");
Logger::getInstance().log("User logged in");
// Both calls use the SAME Logger instance
```

**Pros:** Controlled single instance, lazy initialization.
**Cons:** Global state (hard to test), hides dependencies.
**When NOT to use:** When you might need multiple instances later, or when testability matters.

> 🔥 **Interview:** "Make Singleton thread-safe" → Use `static` local variable (C++11 guarantee) or double-checked locking pattern.

---

### Factory Method

**Problem:** You need to create objects, but you don't know at compile time which concrete class to instantiate.

**Solution:** Define a factory method that subclasses can override.

```
┌─────────────┐        ┌───────────────┐
│  Creator     │        │    Product     │
│+ create()   │───────▶│  + operation()│
└──────┬──────┘        └───────▲───────┘
       │                       │
       │               ┌───────┴───────┐
┌──────▼──────┐    ┌───┴────┐    ┌─────┴──┐
│ConcreteCreator│  │ProductA│    │ProductB│
│+ create()   │  └────────┘    └────────┘
└─────────────┘
```

```cpp
// Product hierarchy
class Animal {
public:
    virtual void speak() = 0;
    virtual ~Animal() {}
};

class Dog : public Animal { void speak() override { cout << "Woof\n"; } };
class Cat : public Animal { void speak() override { cout << "Meow\n"; } };
class Duck : public Animal { void speak() override { cout << "Quack\n"; } };

// Factory
class AnimalFactory {
public:
    static unique_ptr<Animal> create(const string& type) {
        if (type == "dog") return make_unique<Dog>();
        if (type == "cat") return make_unique<Cat>();
        if (type == "duck") return make_unique<Duck>();
        throw invalid_argument("Unknown animal: " + type);
    }
};

// Usage — caller doesn't know which concrete class is created
string userChoice = "dog";
auto animal = AnimalFactory::create(userChoice);
animal->speak(); // Woof
```

**When to use:** When the type of object to create depends on runtime conditions (config, user input, DB value).

---

### Abstract Factory

**Problem:** You need families of related objects. You want to ensure they're compatible.

```
"Create me a UI toolkit — but it must be ALL Windows-style or ALL Mac-style, not mixed."
```

```cpp
// Abstract products
class Button { public: virtual void render() = 0; virtual ~Button() {} };
class Checkbox { public: virtual void render() = 0; virtual ~Checkbox() {} };

// Windows implementations
class WindowsButton : public Button { void render() override { cout << "[Windows Button]\n"; } };
class WindowsCheckbox : public Checkbox { void render() override { cout << "[Windows Checkbox]\n"; } };

// Mac implementations
class MacButton : public Button { void render() override { cout << "[Mac Button]\n"; } };
class MacCheckbox : public Checkbox { void render() override { cout << "[Mac Checkbox]\n"; } };

// Abstract factory
class UIFactory {
public:
    virtual unique_ptr<Button> createButton() = 0;
    virtual unique_ptr<Checkbox> createCheckbox() = 0;
    virtual ~UIFactory() {}
};

// Concrete factories
class WindowsFactory : public UIFactory {
    unique_ptr<Button> createButton() override { return make_unique<WindowsButton>(); }
    unique_ptr<Checkbox> createCheckbox() override { return make_unique<WindowsCheckbox>(); }
};

class MacFactory : public UIFactory {
    unique_ptr<Button> createButton() override { return make_unique<MacButton>(); }
    unique_ptr<Checkbox> createCheckbox() override { return make_unique<MacCheckbox>(); }
};

// Client code — doesn't know which OS it's running on
void buildUI(UIFactory& factory) {
    auto btn = factory.createButton();
    auto chk = factory.createCheckbox();
    btn->render();
    chk->render();
}

MacFactory mac;
buildUI(mac);
// [Mac Button] [Mac Checkbox] — always consistent!
```

---

### Builder

**Problem:** A complex object needs many optional parameters. Telescoping constructors are ugly.

```cpp
// BAD: Telescoping constructor
Pizza(string size, string crust, bool cheese, bool pepperoni, bool mushrooms, bool onions, bool peppers) {}
Pizza("large", "thin", true, true, false, true, false); // What's what??
```

**Solution:** Separate the construction from the representation.

```cpp
class Pizza {
    string size, crust;
    bool cheese, pepperoni, mushrooms, onions;

    Pizza() {}  // Private — only Builder creates it

public:
    class Builder {
        string size = "medium";
        string crust = "regular";
        bool cheese = false, pepperoni = false, mushrooms = false, onions = false;

    public:
        Builder& setSize(string s) { size = s; return *this; }
        Builder& setCrust(string c) { crust = c; return *this; }
        Builder& withCheese() { cheese = true; return *this; }
        Builder& withPepperoni() { pepperoni = true; return *this; }
        Builder& withMushrooms() { mushrooms = true; return *this; }
        Builder& withOnions() { onions = true; return *this; }

        Pizza build() {
            Pizza p;
            p.size = size; p.crust = crust;
            p.cheese = cheese; p.pepperoni = pepperoni;
            p.mushrooms = mushrooms; p.onions = onions;
            return p;
        }
    };

    void print() {
        cout << size << " pizza, " << crust << " crust"
             << (cheese ? ", cheese" : "")
             << (pepperoni ? ", pepperoni" : "")
             << (mushrooms ? ", mushrooms" : "")
             << (onions ? ", onions" : "") << "\n";
    }
};

// Beautiful, readable construction:
Pizza p = Pizza::Builder()
    .setSize("large")
    .setCrust("thin")
    .withCheese()
    .withPepperoni()
    .withOnions()
    .build();
p.print(); // large pizza, thin crust, cheese, pepperoni, onions
```

---

### Prototype

**Problem:** Creating an object is expensive (lots of computation or DB calls). You want to clone an existing one.

```cpp
class Enemy {
    string type;
    int health, damage;
    vector<string> abilities;  // Expensive to compute

public:
    Enemy(string t, int h, int d) : type(t), health(h), damage(d) {
        // Expensive initialization (AI setup, loading assets, etc.)
    }

    virtual Enemy* clone() const {
        return new Enemy(*this);  // Uses copy constructor
    }

    void setPosition(int x, int y) { /* position */ }
};

// Create the expensive prototype once:
Enemy* goblinPrototype = new Enemy("Goblin", 100, 15);

// Clone cheaply for each level:
Enemy* goblin1 = goblinPrototype->clone();
Enemy* goblin2 = goblinPrototype->clone();
goblin1->setPosition(100, 200);
goblin2->setPosition(300, 400);
```

---

## 5.2 Structural Patterns

### Adapter

**Problem:** You have two incompatible interfaces. You want them to work together.

**Real-world analogy:** A travel power adapter. Your laptop plug (one interface) → adapter → UK wall socket (different interface).

```cpp
// Existing interface your code expects
class MediaPlayer {
public:
    virtual void play(const string& filename) = 0;
};

// Incompatible third-party library
class VLCLibrary {
public:
    void vlcPlay(const string& filename) {
        cout << "VLC playing: " << filename << "\n";
    }
};

// Adapter: makes VLC look like MediaPlayer
class VLCAdapter : public MediaPlayer {
    VLCLibrary vlc;
public:
    void play(const string& filename) override {
        vlc.vlcPlay(filename);  // Translate the call
    }
};

// Your code uses MediaPlayer interface
void playMedia(MediaPlayer* player, string file) {
    player->play(file);
}

VLCAdapter adapter;
playMedia(&adapter, "movie.mp4");  // Works!
```

---

### Decorator

**Problem:** You want to add behavior to objects dynamically, without subclassing.

**Real-world analogy:** A coffee. Start with espresso, add milk → latte. Add whipped cream → fancy latte. Each addition "decorates" the previous.

```cpp
class Coffee {
public:
    virtual double cost() const = 0;
    virtual string description() const = 0;
    virtual ~Coffee() {}
};

class Espresso : public Coffee {
    double cost() const override { return 1.0; }
    string description() const override { return "Espresso"; }
};

// Decorator base
class CoffeeDecorator : public Coffee {
protected:
    Coffee* coffee;
public:
    CoffeeDecorator(Coffee* c) : coffee(c) {}
};

// Concrete decorators
class Milk : public CoffeeDecorator {
    double cost() const override { return coffee->cost() + 0.5; }
    string description() const override { return coffee->description() + ", Milk"; }
public:
    Milk(Coffee* c) : CoffeeDecorator(c) {}
};

class Sugar : public CoffeeDecorator {
    double cost() const override { return coffee->cost() + 0.2; }
    string description() const override { return coffee->description() + ", Sugar"; }
public:
    Sugar(Coffee* c) : CoffeeDecorator(c) {}
};

class WhippedCream : public CoffeeDecorator {
    double cost() const override { return coffee->cost() + 0.8; }
    string description() const override { return coffee->description() + ", Whipped Cream"; }
public:
    WhippedCream(Coffee* c) : CoffeeDecorator(c) {}
};

// Usage:
Coffee* myCoffee = new Espresso();
myCoffee = new Milk(myCoffee);
myCoffee = new Sugar(myCoffee);
myCoffee = new WhippedCream(myCoffee);

cout << myCoffee->description() << "\n"; // Espresso, Milk, Sugar, Whipped Cream
cout << myCoffee->cost() << "\n";        // 2.5
```

---

### Facade

**Problem:** A complex subsystem is hard to use. You want a simple unified interface.

**Real-world analogy:** A home theater remote. One button "Watch Movie" dims lights, turns on projector, starts surround sound, plays the movie. You don't control each separately.

```cpp
// Complex subsystems
class DVDPlayer { public: void on() {} void play(string movie) {} };
class Projector { public: void on() {} void widescreen() {} };
class SoundSystem { public: void on() {} void setVolume(int v) {} };
class Lights { public: void dim(int level) {} };

// Facade
class HomeTheaterFacade {
    DVDPlayer dvd;
    Projector projector;
    SoundSystem sound;
    Lights lights;

public:
    void watchMovie(const string& movie) {
        cout << "Setting up movie...\n";
        lights.dim(20);
        projector.on();
        projector.widescreen();
        sound.on();
        sound.setVolume(80);
        dvd.on();
        dvd.play(movie);
    }

    void endMovie() {
        cout << "Shutting down...\n";
        // Turn everything off
    }
};

// Simple client code:
HomeTheaterFacade theater;
theater.watchMovie("Inception");  // One call does everything
```

---

### Proxy

**Problem:** You want to control access to an object (lazy loading, access control, caching, logging).

```cpp
// Real service (expensive to create)
class ImageLoader {
    string filename;
    string imageData;
public:
    ImageLoader(const string& f) : filename(f) {
        cout << "Loading image from disk: " << filename << "\n";
        imageData = "[image data]"; // Expensive disk I/O
    }
    void display() { cout << "Displaying " << filename << "\n"; }
};

// Interface
class Image {
public:
    virtual void display() = 0;
    virtual ~Image() {}
};

// Real subject
class RealImage : public Image {
    ImageLoader loader;
public:
    RealImage(const string& f) : loader(f) {} // Loads immediately
    void display() override { loader.display(); }
};

// Proxy — lazy loading
class ProxyImage : public Image {
    string filename;
    RealImage* realImage = nullptr;
public:
    ProxyImage(const string& f) : filename(f) {}

    void display() override {
        if (!realImage) {
            realImage = new RealImage(filename); // Load only when needed!
        }
        realImage->display();
    }

    ~ProxyImage() { delete realImage; }
};

ProxyImage img("photo.jpg");
// Image NOT loaded yet
img.display(); // ← Loads NOW, then displays
img.display(); // ← Just displays, no reload
```

---

### Composite

**Problem:** You have a tree structure (file system, UI hierarchy) and want to treat leaves and branches uniformly.

```cpp
class FileSystemItem {
public:
    virtual void display(int indent = 0) = 0;
    virtual int size() = 0;
    virtual ~FileSystemItem() {}
};

// Leaf
class File : public FileSystemItem {
    string name;
    int fileSize;
public:
    File(string n, int s) : name(n), fileSize(s) {}
    void display(int indent = 0) override {
        cout << string(indent, ' ') << "📄 " << name << " (" << fileSize << "B)\n";
    }
    int size() override { return fileSize; }
};

// Composite
class Directory : public FileSystemItem {
    string name;
    vector<FileSystemItem*> children;
public:
    Directory(string n) : name(n) {}
    void add(FileSystemItem* item) { children.push_back(item); }

    void display(int indent = 0) override {
        cout << string(indent, ' ') << "📁 " << name << "\n";
        for (auto* c : children) c->display(indent + 2);
    }

    int size() override {
        int total = 0;
        for (auto* c : children) total += c->size();
        return total;
    }
};

// Build tree:
Directory* root = new Directory("root");
root->add(new File("readme.txt", 100));
Directory* src = new Directory("src");
src->add(new File("main.cpp", 500));
src->add(new File("utils.cpp", 300));
root->add(src);
root->display();
cout << "Total size: " << root->size() << "B\n";
```

---

## 5.3 Behavioral Patterns

### Observer

**Problem:** When one object changes state, many others need to be notified automatically.

**Real-world analogy:** YouTube subscriptions. When a creator uploads (subject changes), all subscribers (observers) get notified.

```cpp
class Observer {
public:
    virtual void update(const string& event) = 0;
    virtual ~Observer() {}
};

class Subject {
    vector<Observer*> observers;
public:
    void subscribe(Observer* o) { observers.push_back(o); }
    void unsubscribe(Observer* o) {
        observers.erase(remove(observers.begin(), observers.end(), o), observers.end());
    }
    void notify(const string& event) {
        for (auto* o : observers) o->update(event);
    }
};

class StockMarket : public Subject {
    string symbol;
    double price;
public:
    StockMarket(string sym, double p) : symbol(sym), price(p) {}

    void setPrice(double p) {
        price = p;
        notify(symbol + " price changed to $" + to_string(p));
    }
};

class Trader : public Observer {
    string name;
public:
    Trader(string n) : name(n) {}
    void update(const string& event) override {
        cout << name << " notified: " << event << "\n";
    }
};

StockMarket apple("AAPL", 150.0);
Trader alice("Alice"), bob("Bob");

apple.subscribe(&alice);
apple.subscribe(&bob);
apple.setPrice(155.0);  // Alice notified, Bob notified
apple.unsubscribe(&bob);
apple.setPrice(148.0);  // Only Alice notified
```

---

### Strategy

**Problem:** You have multiple algorithms for the same task. You want to swap them at runtime.

**Real-world analogy:** Navigation app. Same destination, different routes: fastest, shortest, avoid tolls. The routing strategy changes, the app doesn't.

```cpp
class SortStrategy {
public:
    virtual void sort(vector<int>& data) = 0;
    virtual string name() = 0;
    virtual ~SortStrategy() {}
};

class BubbleSort : public SortStrategy {
    void sort(vector<int>& data) override {
        // Bubble sort implementation
        for (int i = 0; i < data.size(); i++)
            for (int j = 0; j < data.size()-i-1; j++)
                if (data[j] > data[j+1]) swap(data[j], data[j+1]);
    }
    string name() override { return "Bubble Sort"; }
};

class QuickSort : public SortStrategy {
    void sort(vector<int>& data) override {
        std::sort(data.begin(), data.end()); // Simplified
    }
    string name() override { return "Quick Sort"; }
};

class Sorter {
    SortStrategy* strategy;
public:
    Sorter(SortStrategy* s) : strategy(s) {}

    void setStrategy(SortStrategy* s) { strategy = s; }

    void sort(vector<int>& data) {
        cout << "Sorting with: " << strategy->name() << "\n";
        strategy->sort(data);
    }
};

vector<int> data = {5, 2, 8, 1, 9};
Sorter sorter(new BubbleSort());
sorter.sort(data);                     // Uses Bubble Sort

sorter.setStrategy(new QuickSort());
sorter.sort(data);                     // Switched to Quick Sort
```

---

### Command

**Problem:** You want to encapsulate requests as objects (for undo/redo, queuing, logging).

**Real-world analogy:** A restaurant order. The waiter writes it on a slip (command object). The slip goes to the kitchen (executor). The waiter doesn't cook — they just manage the command.

```cpp
class Command {
public:
    virtual void execute() = 0;
    virtual void undo() = 0;
    virtual ~Command() {}
};

class TextEditor {
    string text;
public:
    void insertText(const string& s) { text += s; }
    void deleteText(int n) { if (n <= text.size()) text.erase(text.size()-n); }
    string getText() { return text; }
};

class InsertCommand : public Command {
    TextEditor& editor;
    string insertedText;
public:
    InsertCommand(TextEditor& e, const string& s) : editor(e), insertedText(s) {}
    void execute() override { editor.insertText(insertedText); }
    void undo() override { editor.deleteText(insertedText.size()); }
};

class CommandManager {
    stack<Command*> history;
public:
    void execute(Command* cmd) {
        cmd->execute();
        history.push(cmd);
    }
    void undo() {
        if (!history.empty()) {
            history.top()->undo();
            history.pop();
        }
    }
};

TextEditor editor;
CommandManager manager;

manager.execute(new InsertCommand(editor, "Hello "));
manager.execute(new InsertCommand(editor, "World"));
cout << editor.getText(); // "Hello World"
manager.undo();
cout << editor.getText(); // "Hello "
manager.undo();
cout << editor.getText(); // ""
```

---

### State

**Problem:** An object behaves differently based on its internal state. Use state objects instead of massive if/else chains.

**Real-world analogy:** A traffic light. It cycles through Red → Green → Yellow → Red. Its behavior (which color shows next) depends on current state.

```cpp
class TrafficLight;

class TrafficLightState {
public:
    virtual void handle(TrafficLight& light) = 0;
    virtual string color() = 0;
    virtual ~TrafficLightState() {}
};

class RedState : public TrafficLightState {
public:
    void handle(TrafficLight& light) override;
    string color() override { return "RED"; }
};

class GreenState : public TrafficLightState {
public:
    void handle(TrafficLight& light) override;
    string color() override { return "GREEN"; }
};

class YellowState : public TrafficLightState {
public:
    void handle(TrafficLight& light) override;
    string color() override { return "YELLOW"; }
};

class TrafficLight {
    TrafficLightState* state;
public:
    TrafficLight() : state(new RedState()) {}

    void setState(TrafficLightState* s) {
        delete state;
        state = s;
    }

    void change() {
        cout << "Current: " << state->color() << "\n";
        state->handle(*this);
    }
};

void RedState::handle(TrafficLight& l) { l.setState(new GreenState()); }
void GreenState::handle(TrafficLight& l) { l.setState(new YellowState()); }
void YellowState::handle(TrafficLight& l) { l.setState(new RedState()); }

TrafficLight light;
light.change(); // RED → GREEN
light.change(); // GREEN → YELLOW
light.change(); // YELLOW → RED
```

---

### Template Method

**Problem:** Multiple algorithms share the same skeleton but differ in specific steps.

```cpp
class DataProcessor {
public:
    // Template method — defines the algorithm skeleton
    void process() {
        readData();
        processData();  // Hook — subclasses implement
        writeData();
    }

protected:
    void readData() { cout << "Reading data\n"; }  // Common step
    virtual void processData() = 0;                 // Varies
    void writeData() { cout << "Writing data\n"; } // Common step
};

class CSVProcessor : public DataProcessor {
    void processData() override { cout << "Parsing CSV\n"; }
};

class JSONProcessor : public DataProcessor {
    void processData() override { cout << "Parsing JSON\n"; }
};

CSVProcessor csv;
csv.process();
// Reading data / Parsing CSV / Writing data
```

---

### Chain of Responsibility

**Problem:** A request should pass through a chain of handlers until one handles it.

**Real-world analogy:** Customer support escalation. Try automated bot → Level 1 support → Level 2 → Manager.

```cpp
class SupportHandler {
protected:
    SupportHandler* next = nullptr;
public:
    void setNext(SupportHandler* n) { next = n; }

    virtual void handle(int severity) {
        if (next) next->handle(severity);
        else cout << "Issue unresolved\n";
    }
};

class AutoBot : public SupportHandler {
    void handle(int severity) override {
        if (severity == 1) cout << "Bot resolved issue\n";
        else SupportHandler::handle(severity);
    }
};

class Level1 : public SupportHandler {
    void handle(int severity) override {
        if (severity <= 2) cout << "L1 agent resolved issue\n";
        else SupportHandler::handle(severity);
    }
};

class Manager : public SupportHandler {
    void handle(int severity) override {
        cout << "Manager handling severity " << severity << "\n";
    }
};

AutoBot bot;
Level1 l1;
Manager mgr;

bot.setNext(&l1);
l1.setNext(&mgr);

bot.handle(1); // Bot resolved
bot.handle(2); // L1 resolved
bot.handle(3); // Manager handling
```

---

# 📚 PART 6: LLD Case Studies

---

## 6.1 Parking Lot System

### Requirements

**Functional:**
- Support multiple vehicle types: Motorcycle, Car, Truck
- Multiple parking spots of different sizes: Small, Medium, Large
- Issue tickets on entry, calculate fees on exit
- Track available spots in real time
- Handle entrance/exit gates

**Non-Functional:**
- Handle concurrent entry/exit (thread safety)
- Supports up to 1000 spots

### Class Diagram

```
┌──────────────────┐      ┌─────────────────┐
│   ParkingLot     │      │  ParkingFloor   │
│─────────────────│      │─────────────────│
│ - floors[]       │1───*│ - spots[]       │
│ - gates[]        │      │ - floorNumber   │
│─────────────────│      │─────────────────│
│ + getTicket()    │      │ + getFreeSpot() │
│ + processExit()  │      │ + isFull()      │
└──────────────────┘      └────────┬────────┘
                                   │ 1
                                   │ *
                          ┌────────▼────────┐
                          │   ParkingSpot   │
                          │─────────────────│
                          │ - spotId        │
                          │ - size: SpotSize│
                          │ - isOccupied    │
                          │ - vehicle       │
                          │─────────────────│
                          │ + park()        │
                          │ + unpark()      │
                          └────────┬────────┘
                                   │ parks
┌──────────────────┐               │
│    Vehicle       │◀──────────────┘
│─────────────────│
│ - licensePlate   │
│ - type: VehicleType│
└────────┬─────────┘
         │
    ┌────┴────┐
 Motorcycle  Car  Truck

┌──────────────────┐      ┌─────────────────┐
│   ParkingTicket  │      │   PricingEngine │
│─────────────────│      │─────────────────│
│ - ticketId       │      │ + calculateFee()│
│ - entryTime      │      │ - ratePerHour[] │
│ - spot           │      └─────────────────┘
│ - vehicle        │
└──────────────────┘
```

### Code Skeleton

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <chrono>
#include <string>
using namespace std;
using namespace chrono;

// Enums
enum class VehicleType { MOTORCYCLE, CAR, TRUCK };
enum class SpotSize { SMALL, MEDIUM, LARGE };

// Vehicle
class Vehicle {
protected:
    string licensePlate;
    VehicleType type;
public:
    Vehicle(string plate, VehicleType t) : licensePlate(plate), type(t) {}
    VehicleType getType() const { return type; }
    string getPlate() const { return licensePlate; }
    virtual SpotSize requiredSize() const = 0;
    virtual ~Vehicle() {}
};

class Motorcycle : public Vehicle {
public:
    Motorcycle(string plate) : Vehicle(plate, VehicleType::MOTORCYCLE) {}
    SpotSize requiredSize() const override { return SpotSize::SMALL; }
};

class Car : public Vehicle {
public:
    Car(string plate) : Vehicle(plate, VehicleType::CAR) {}
    SpotSize requiredSize() const override { return SpotSize::MEDIUM; }
};

class Truck : public Vehicle {
public:
    Truck(string plate) : Vehicle(plate, VehicleType::TRUCK) {}
    SpotSize requiredSize() const override { return SpotSize::LARGE; }
};

// Parking Spot
class ParkingSpot {
    string spotId;
    SpotSize size;
    bool occupied;
    Vehicle* vehicle;

public:
    ParkingSpot(string id, SpotSize s)
        : spotId(id), size(s), occupied(false), vehicle(nullptr) {}

    bool canFit(const Vehicle& v) const {
        return !occupied && (int)size >= (int)v.requiredSize();
    }

    void park(Vehicle* v) {
        vehicle = v;
        occupied = true;
    }

    Vehicle* unpark() {
        Vehicle* v = vehicle;
        vehicle = nullptr;
        occupied = false;
        return v;
    }

    string getId() const { return spotId; }
    bool isOccupied() const { return occupied; }
    SpotSize getSize() const { return size; }
};

// Parking Ticket
class ParkingTicket {
    static int counter;
    string ticketId;
    system_clock::time_point entryTime;
    ParkingSpot* spot;
    Vehicle* vehicle;

public:
    ParkingTicket(ParkingSpot* s, Vehicle* v)
        : ticketId("TKT" + to_string(++counter)),
          entryTime(system_clock::now()),
          spot(s), vehicle(v) {}

    string getId() const { return ticketId; }
    ParkingSpot* getSpot() const { return spot; }
    system_clock::time_point getEntryTime() const { return entryTime; }
    Vehicle* getVehicle() const { return vehicle; }
};
int ParkingTicket::counter = 0;

// Pricing
class PricingEngine {
    map<SpotSize, double> ratePerHour = {
        {SpotSize::SMALL, 10.0},
        {SpotSize::MEDIUM, 20.0},
        {SpotSize::LARGE, 30.0}
    };

public:
    double calculateFee(const ParkingTicket& ticket) {
        auto now = system_clock::now();
        auto duration = duration_cast<hours>(now - ticket.getEntryTime()).count();
        if (duration == 0) duration = 1; // Minimum 1 hour

        SpotSize size = ticket.getSpot()->getSize();
        return duration * ratePerHour[size];
    }
};

// Parking Floor
class ParkingFloor {
    int floorNumber;
    vector<ParkingSpot*> spots;

public:
    ParkingFloor(int num) : floorNumber(num) {}

    void addSpot(ParkingSpot* spot) { spots.push_back(spot); }

    ParkingSpot* findFreeSpot(const Vehicle& v) {
        for (auto* spot : spots) {
            if (spot->canFit(v)) return spot;
        }
        return nullptr;
    }

    int getFloorNumber() const { return floorNumber; }
};

// Main Parking Lot
class ParkingLot {
    vector<ParkingFloor*> floors;
    map<string, ParkingTicket*> activeTickets;  // ticketId -> ticket
    PricingEngine pricing;
    static ParkingLot* instance;

    ParkingLot() {
        // Initialize floors and spots
        ParkingFloor* f1 = new ParkingFloor(1);
        f1->addSpot(new ParkingSpot("S1-1", SpotSize::SMALL));
        f1->addSpot(new ParkingSpot("S1-2", SpotSize::SMALL));
        f1->addSpot(new ParkingSpot("M1-1", SpotSize::MEDIUM));
        f1->addSpot(new ParkingSpot("M1-2", SpotSize::MEDIUM));
        f1->addSpot(new ParkingSpot("L1-1", SpotSize::LARGE));
        floors.push_back(f1);
    }

public:
    static ParkingLot& getInstance() {
        static ParkingLot lot;
        return lot;
    }

    ParkingTicket* parkVehicle(Vehicle* v) {
        for (auto* floor : floors) {
            ParkingSpot* spot = floor->findFreeSpot(*v);
            if (spot) {
                spot->park(v);
                auto* ticket = new ParkingTicket(spot, v);
                activeTickets[ticket->getId()] = ticket;
                cout << "Vehicle " << v->getPlate()
                     << " parked at spot " << spot->getId()
                     << ". Ticket: " << ticket->getId() << "\n";
                return ticket;
            }
        }
        cout << "No available spot for " << v->getPlate() << "\n";
        return nullptr;
    }

    double processExit(const string& ticketId) {
        auto it = activeTickets.find(ticketId);
        if (it == activeTickets.end()) {
            cout << "Invalid ticket\n";
            return -1;
        }

        ParkingTicket* ticket = it->second;
        double fee = pricing.calculateFee(*ticket);
        ticket->getSpot()->unpark();
        activeTickets.erase(it);

        cout << "Fee for ticket " << ticketId << ": Rs." << fee << "\n";
        delete ticket;
        return fee;
    }
};

// Demo
int main() {
    ParkingLot& lot = ParkingLot::getInstance();

    Car car1("KA01AB1234");
    Motorcycle bike("KA02CD5678");
    Truck truck("KA03EF9012");

    auto* t1 = lot.parkVehicle(&car1);
    auto* t2 = lot.parkVehicle(&bike);
    auto* t3 = lot.parkVehicle(&truck);

    if (t1) lot.processExit(t1->getId());
    if (t2) lot.processExit(t2->getId());
}
```

**Design Decisions:**
- Singleton for ParkingLot — only one real lot exists
- Strategy pattern for pricing — easy to add new pricing models
- Vehicle's `requiredSize()` follows OCP — new vehicle types, no change to ParkingSpot
- Thread safety: add `mutex` around `parkVehicle` and `processExit` for production

---

## 6.2 Library Management System

### Requirements

**Functional:**
- Maintain catalog of books (multiple copies)
- Members can borrow/return books
- Track due dates, calculate fines
- Search books by title/author/ISBN
- Manage reservations

### Class Diagram

```
┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│    Book      │1──*│  BookItem      │     │   Member     │
│─────────────│     │───────────────│     │─────────────│
│- isbn        │     │- barcode       │     │- memberId    │
│- title       │     │- isAvailable   │     │- name        │
│- author      │     │- dueDate       │     │- borrowed[]  │
│- genre       │     │- location      │     │─────────────│
└──────────────┘     └───────────────┘     │+ borrow()    │
                                           │+ return()    │
                                           └──────────────┘

┌──────────────────┐     ┌──────────────┐
│  BorrowRecord    │     │  Catalog     │
│─────────────────│     │─────────────│
│- bookItem        │     │- books[]     │
│- member          │     │─────────────│
│- issueDate       │     │+ search()   │
│- dueDate         │     │+ addBook()  │
└──────────────────┘     └──────────────┘
```

### Code Skeleton

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <optional>
#include <algorithm>
#include <chrono>
using namespace std;
using namespace chrono;

class Book {
    string isbn, title, author, genre;
public:
    Book(string i, string t, string a, string g)
        : isbn(i), title(t), author(a), genre(g) {}

    string getIsbn() const { return isbn; }
    string getTitle() const { return title; }
    string getAuthor() const { return author; }
};

class BookItem {
    string barcode;
    string isbn;
    bool available;
    system_clock::time_point dueDate;

public:
    BookItem(string bc, string isbn) : barcode(bc), isbn(isbn), available(true) {}

    bool isAvailable() const { return available; }
    string getBarcode() const { return barcode; }
    string getIsbn() const { return isbn; }

    void checkout(int days) {
        available = false;
        dueDate = system_clock::now() + hours(24 * days);
    }

    void returnItem() { available = true; }

    double calculateFine() const {
        if (available) return 0.0;
        auto now = system_clock::now();
        auto overdue = duration_cast<hours>(now - dueDate).count();
        return (overdue > 0) ? (overdue / 24.0) * 5.0 : 0.0; // Rs.5/day
    }
};

class Member {
    string memberId, name;
    vector<BookItem*> borrowed;
    static const int MAX_BOOKS = 5;

public:
    Member(string id, string n) : memberId(id), name(n) {}

    bool canBorrow() const { return borrowed.size() < MAX_BOOKS; }

    bool borrow(BookItem* item) {
        if (!canBorrow()) {
            cout << name << " has reached borrow limit\n";
            return false;
        }
        item->checkout(14); // 2 weeks
        borrowed.push_back(item);
        cout << name << " borrowed book: " << item->getBarcode() << "\n";
        return true;
    }

    bool returnBook(const string& barcode) {
        auto it = find_if(borrowed.begin(), borrowed.end(),
                          [&](BookItem* b) { return b->getBarcode() == barcode; });
        if (it == borrowed.end()) return false;

        double fine = (*it)->calculateFine();
        (*it)->returnItem();
        borrowed.erase(it);

        if (fine > 0)
            cout << name << " paid fine: Rs." << fine << "\n";
        else
            cout << name << " returned book: " << barcode << "\n";
        return true;
    }

    string getId() const { return memberId; }
};

class Catalog {
    map<string, Book*> books;          // isbn → Book
    map<string, vector<BookItem*>> items; // isbn → BookItems

public:
    void addBook(Book* book) {
        books[book->getIsbn()] = book;
    }

    void addBookItem(BookItem* item) {
        items[item->getIsbn()].push_back(item);
    }

    BookItem* findAvailable(const string& isbn) {
        if (items.find(isbn) == items.end()) return nullptr;
        for (auto* item : items[isbn]) {
            if (item->isAvailable()) return item;
        }
        return nullptr;
    }

    vector<Book*> searchByTitle(const string& title) {
        vector<Book*> result;
        for (auto& [isbn, book] : books) {
            if (book->getTitle().find(title) != string::npos)
                result.push_back(book);
        }
        return result;
    }
};

class Library {
    Catalog catalog;
    map<string, Member*> members;

public:
    void addMember(Member* m) { members[m->getId()] = m; }
    void addBook(Book* b) { catalog.addBook(b); }
    void addBookItem(BookItem* bi) { catalog.addBookItem(bi); }

    bool checkout(const string& memberId, const string& isbn) {
        auto* member = members.count(memberId) ? members[memberId] : nullptr;
        if (!member) { cout << "Member not found\n"; return false; }

        auto* item = catalog.findAvailable(isbn);
        if (!item) { cout << "No available copy\n"; return false; }

        return member->borrow(item);
    }

    bool returnBook(const string& memberId, const string& barcode) {
        auto* member = members.count(memberId) ? members[memberId] : nullptr;
        if (!member) return false;
        return member->returnBook(barcode);
    }
};

int main() {
    Library lib;

    Book* b1 = new Book("978-0-13-110362-7", "The C++ Programming Language", "Bjarne Stroustrup", "CS");
    BookItem* bi1 = new BookItem("BC001", "978-0-13-110362-7");
    BookItem* bi2 = new BookItem("BC002", "978-0-13-110362-7");
    lib.addBook(b1);
    lib.addBookItem(bi1);
    lib.addBookItem(bi2);

    Member* m1 = new Member("M001", "Alice");
    Member* m2 = new Member("M002", "Bob");
    lib.addMember(m1);
    lib.addMember(m2);

    lib.checkout("M001", "978-0-13-110362-7"); // Alice borrows
    lib.checkout("M002", "978-0-13-110362-7"); // Bob borrows (2nd copy)
    lib.returnBook("M001", "BC001");             // Alice returns
}
```

---

## 6.3 Elevator System

### Requirements

- Multiple elevators in a building
- Users request elevator from any floor (up/down)
- Elevator picks up passengers, takes them to destination
- Efficient dispatching algorithm

### Class Diagram

```
┌──────────────────┐     ┌────────────────────┐
│  ElevatorSystem  │1──*│     Elevator        │
│─────────────────│     │───────────────────│
│- elevators[]     │     │- id               │
│- dispatcher      │     │- currentFloor     │
│─────────────────│     │- direction        │
│+ requestElevator()│    │- state            │
└──────────────────┘     │- pendingRequests[]│
                         │───────────────────│
                         │+ addRequest()     │
                         │+ processNext()    │
                         └────────────────────┘

┌──────────────────┐     ┌────────────────────┐
│  ExternalRequest │     │  InternalRequest   │
│─────────────────│     │───────────────────│
│- floor           │     │- destinationFloor  │
│- direction       │     └────────────────────┘
└──────────────────┘
```

### Code Skeleton

```cpp
#include <iostream>
#include <queue>
#include <set>
#include <vector>
using namespace std;

enum class Direction { UP, DOWN, IDLE };
enum class ElevatorState { MOVING, IDLE, DOOR_OPEN };

class Elevator {
    int id, currentFloor, totalFloors;
    Direction direction;
    ElevatorState state;
    set<int> upQueue;    // Floors to stop while going up (sorted ascending)
    set<int, greater<int>> downQueue; // Floors to stop while going down (sorted descending)

public:
    Elevator(int id, int floors) : id(id), currentFloor(1), totalFloors(floors),
        direction(Direction::IDLE), state(ElevatorState::IDLE) {}

    void addRequest(int floor) {
        if (floor > currentFloor) upQueue.insert(floor);
        else if (floor < currentFloor) downQueue.insert(floor);
        else openDoor(); // Already here
        updateDirection();
    }

    void step() {
        if (state == ElevatorState::IDLE) return;

        if (direction == Direction::UP && !upQueue.empty()) {
            currentFloor++;
            cout << "Elevator " << id << " at floor " << currentFloor << " (↑)\n";
            if (upQueue.count(currentFloor)) {
                upQueue.erase(currentFloor);
                openDoor();
            }
        } else if (direction == Direction::DOWN && !downQueue.empty()) {
            currentFloor--;
            cout << "Elevator " << id << " at floor " << currentFloor << " (↓)\n";
            if (downQueue.count(currentFloor)) {
                downQueue.erase(currentFloor);
                openDoor();
            }
        }
        updateDirection();
    }

    void openDoor() {
        cout << "Elevator " << id << ": Door OPEN at floor " << currentFloor << "\n";
    }

    void updateDirection() {
        if (!upQueue.empty()) direction = Direction::UP;
        else if (!downQueue.empty()) direction = Direction::DOWN;
        else { direction = Direction::IDLE; state = ElevatorState::IDLE; }
        if (direction != Direction::IDLE) state = ElevatorState::MOVING;
    }

    int getCurrentFloor() const { return currentFloor; }
    Direction getDirection() const { return direction; }
    int getId() const { return id; }
    bool isIdle() const { return state == ElevatorState::IDLE; }

    int distanceTo(int floor) const { return abs(currentFloor - floor); }
};

class ElevatorDispatcher {
public:
    // Simple: find nearest idle or same-direction elevator
    Elevator* dispatch(vector<Elevator*>& elevators, int requestFloor, Direction dir) {
        Elevator* best = nullptr;
        int minDist = INT_MAX;

        for (auto* e : elevators) {
            if (e->isIdle()) {
                int dist = e->distanceTo(requestFloor);
                if (dist < minDist) { minDist = dist; best = e; }
            }
        }
        if (!best) best = elevators[0]; // Fallback: first elevator
        return best;
    }
};

class ElevatorSystem {
    vector<Elevator*> elevators;
    ElevatorDispatcher dispatcher;

public:
    ElevatorSystem(int numElevators, int floors) {
        for (int i = 1; i <= numElevators; i++)
            elevators.push_back(new Elevator(i, floors));
    }

    void requestElevator(int floor, Direction dir) {
        cout << "External request: Floor " << floor << "\n";
        Elevator* e = dispatcher.dispatch(elevators, floor, dir);
        e->addRequest(floor);
    }

    void pressButton(int elevatorId, int destinationFloor) {
        cout << "Internal request: Elevator " << elevatorId
             << " → Floor " << destinationFloor << "\n";
        elevators[elevatorId-1]->addRequest(destinationFloor);
    }

    void simulate(int steps) {
        for (int i = 0; i < steps; i++) {
            for (auto* e : elevators) e->step();
        }
    }
};

int main() {
    ElevatorSystem sys(2, 10);

    sys.requestElevator(5, Direction::UP);   // Someone on floor 5 wants to go up
    sys.pressButton(1, 8);                   // They pressed 8 inside elevator 1
    sys.requestElevator(3, Direction::DOWN); // Someone on floor 3 wants to go down

    sys.simulate(15); // Simulate 15 time steps
}
```

---

## 6.4 ATM System

### Requirements

- Card authentication, PIN verification
- Check balance, withdraw cash, deposit, transfer
- Print receipts
- Handle insufficient funds, expired cards, locked accounts

### Class Diagram

```
┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
│     ATM      │    │ CardReader   │    │    CashDispenser  │
│─────────────│    │─────────────│    │─────────────────│
│- cashHolder  │    │+ readCard()  │    │- cashAvailable    │
│- cardReader  │    └──────────────┘    │+ dispenseCash()   │
│- display     │                        └──────────────────┘
│─────────────│    ┌──────────────┐
│+ authenticate()│  │  BankAccount │
│+ withdraw()  │   │─────────────│
│+ deposit()   │   │- accountNo   │
│+ transfer()  │   │- balance     │
└──────────────┘   │- pin (hashed)│
                   │─────────────│
                   │+ validatePin()│
                   │+ debit()     │
                   │+ credit()    │
                   └──────────────┘
```

### Code Skeleton

```cpp
#include <iostream>
#include <string>
#include <map>
using namespace std;

enum class TransactionType { WITHDRAW, DEPOSIT, TRANSFER, BALANCE_CHECK };

class BankAccount {
    string accountNo;
    string cardNo;
    int pin;              // In real system: hashed
    double balance;
    bool locked;
    int failedAttempts;

public:
    BankAccount(string acNo, string card, int p, double bal)
        : accountNo(acNo), cardNo(card), pin(p), balance(bal),
          locked(false), failedAttempts(0) {}

    bool validatePin(int enteredPin) {
        if (locked) { cout << "Account locked\n"; return false; }
        if (enteredPin == pin) {
            failedAttempts = 0;
            return true;
        }
        failedAttempts++;
        if (failedAttempts >= 3) {
            locked = true;
            cout << "Account LOCKED after 3 failed attempts\n";
        }
        return false;
    }

    bool debit(double amount) {
        if (amount > balance) { cout << "Insufficient funds\n"; return false; }
        balance -= amount;
        return true;
    }

    void credit(double amount) { balance += amount; }
    double getBalance() const { return balance; }
    string getAccountNo() const { return accountNo; }
    string getCardNo() const { return cardNo; }
};

class Transaction {
    TransactionType type;
    double amount;
    string accountNo;
    bool success;
    string timestamp;

public:
    Transaction(TransactionType t, double a, string acc, bool s)
        : type(t), amount(a), accountNo(acc), success(s) {}

    void printReceipt() {
        cout << "==== RECEIPT ====\n";
        cout << "Account: " << accountNo << "\n";
        cout << "Amount: Rs." << amount << "\n";
        cout << "Status: " << (success ? "SUCCESS" : "FAILED") << "\n";
        cout << "=================\n";
    }
};

class Bank {
    map<string, BankAccount*> accounts; // cardNo → account
public:
    void addAccount(BankAccount* acc) {
        accounts[acc->getCardNo()] = acc;
    }

    BankAccount* findByCard(const string& cardNo) {
        auto it = accounts.find(cardNo);
        return (it != accounts.end()) ? it->second : nullptr;
    }
};

class ATM {
    double cashAvailable;
    Bank& bank;
    BankAccount* currentAccount = nullptr;

public:
    ATM(double cash, Bank& b) : cashAvailable(cash), bank(b) {}

    bool insertCard(const string& cardNo) {
        currentAccount = bank.findByCard(cardNo);
        if (!currentAccount) { cout << "Card not recognized\n"; return false; }
        cout << "Card accepted\n";
        return true;
    }

    bool enterPin(int pin) {
        if (!currentAccount) return false;
        bool valid = currentAccount->validatePin(pin);
        if (!valid) cout << "Invalid PIN\n";
        return valid;
    }

    void checkBalance() {
        if (!currentAccount) return;
        cout << "Balance: Rs." << currentAccount->getBalance() << "\n";
    }

    Transaction withdraw(double amount) {
        if (!currentAccount) return Transaction(TransactionType::WITHDRAW, amount, "", false);

        if (amount > cashAvailable) {
            cout << "ATM has insufficient cash\n";
            return Transaction(TransactionType::WITHDRAW, amount, "", false);
        }

        bool ok = currentAccount->debit(amount);
        if (ok) cashAvailable -= amount;

        Transaction t(TransactionType::WITHDRAW, amount, currentAccount->getAccountNo(), ok);
        t.printReceipt();
        return t;
    }

    Transaction deposit(double amount) {
        if (!currentAccount) return Transaction(TransactionType::DEPOSIT, amount, "", false);
        currentAccount->credit(amount);
        cashAvailable += amount;
        Transaction t(TransactionType::DEPOSIT, amount, currentAccount->getAccountNo(), true);
        t.printReceipt();
        return t;
    }

    void ejectCard() {
        currentAccount = nullptr;
        cout << "Please take your card. Goodbye!\n";
    }
};

int main() {
    Bank bank;
    BankAccount* acc = new BankAccount("ACC001", "4111111111111111", 1234, 50000);
    bank.addAccount(acc);

    ATM atm(100000, bank);

    atm.insertCard("4111111111111111");
    atm.enterPin(9999);  // Wrong
    atm.enterPin(1234);  // Correct
    atm.checkBalance();
    atm.withdraw(5000);
    atm.checkBalance();
    atm.ejectCard();
}
```

---

## 6.5 Food Delivery System (Swiggy/Zomato)

### Requirements

**Functional:**
- Browse restaurants and menus
- Place orders from a restaurant
- Real-time order tracking
- Multiple payment methods
- Delivery agent assignment

**Non-Functional:**
- Scalable to millions of orders
- Real-time updates

### Class Diagram

```
┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
│     User     │    │  Restaurant  │    │  DeliveryAgent   │
│─────────────│    │─────────────│    │─────────────────│
│- userId      │    │- restaurantId│    │- agentId         │
│- name        │    │- name        │    │- name            │
│- address     │    │- menu[]      │    │- location        │
│- cart        │    │- rating      │    │- isAvailable     │
└──────────────┘    └──────────────┘    └──────────────────┘
        │                  │                     │
        └──────────────────┼─────────────────────┘
                           │
                    ┌──────▼──────┐
                    │    Order    │
                    │─────────────│
                    │- orderId    │
                    │- items[]    │
                    │- status     │
                    │- total      │
                    └─────────────┘
```

### Code Skeleton

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>
using namespace std;

enum class OrderStatus {
    PLACED, ACCEPTED, PREPARING, READY,
    PICKED_UP, DELIVERED, CANCELLED
};

class MenuItem {
    string id, name;
    double price;
    bool available;

public:
    MenuItem(string i, string n, double p) : id(i), name(n), price(p), available(true) {}
    string getId() const { return id; }
    string getName() const { return name; }
    double getPrice() const { return price; }
    bool isAvailable() const { return available; }
};

class CartItem {
public:
    MenuItem* item;
    int quantity;
    CartItem(MenuItem* m, int q) : item(m), quantity(q) {}
    double subtotal() const { return item->getPrice() * quantity; }
};

class Restaurant {
    string id, name, cuisine;
    double rating;
    vector<MenuItem*> menu;
    bool isOpen;

public:
    Restaurant(string i, string n, string c, double r)
        : id(i), name(n), cuisine(c), rating(r), isOpen(true) {}

    void addMenuItem(MenuItem* item) { menu.push_back(item); }

    MenuItem* findItem(const string& itemId) {
        for (auto* m : menu)
            if (m->getId() == itemId) return m;
        return nullptr;
    }

    string getId() const { return id; }
    string getName() const { return name; }
    double getRating() const { return rating; }
    bool isAvailable() const { return isOpen; }
    void printMenu() {
        cout << "\n--- " << name << " Menu ---\n";
        for (auto* m : menu)
            cout << m->getId() << ": " << m->getName() << " - Rs." << m->getPrice() << "\n";
    }
};

class DeliveryAgent {
    string id, name;
    double lat, lng;
    bool available;

public:
    DeliveryAgent(string i, string n) : id(i), name(n), available(true) {}
    bool isAvailable() const { return available; }
    void assignOrder() { available = false; }
    void completeDelivery() { available = true; }
    string getId() const { return id; }
    string getName() const { return name; }
};

class Order {
    static int counter;
    string orderId;
    string userId;
    Restaurant* restaurant;
    vector<CartItem> items;
    OrderStatus status;
    DeliveryAgent* agent;
    double total;

public:
    Order(string uid, Restaurant* r, vector<CartItem> i)
        : orderId("ORD" + to_string(++counter)),
          userId(uid), restaurant(r), items(i),
          status(OrderStatus::PLACED), agent(nullptr) {
        total = 0;
        for (auto& ci : items) total += ci.subtotal();
    }

    void updateStatus(OrderStatus s) {
        status = s;
        cout << "Order " << orderId << " status: " << statusString() << "\n";
    }

    void assignAgent(DeliveryAgent* a) {
        agent = a;
        a->assignOrder();
        cout << "Order " << orderId << " assigned to " << a->getName() << "\n";
    }

    string statusString() const {
        switch (status) {
            case OrderStatus::PLACED:    return "PLACED";
            case OrderStatus::ACCEPTED:  return "ACCEPTED";
            case OrderStatus::PREPARING: return "PREPARING";
            case OrderStatus::READY:     return "READY";
            case OrderStatus::PICKED_UP: return "PICKED_UP";
            case OrderStatus::DELIVERED: return "DELIVERED";
            default: return "CANCELLED";
        }
    }

    string getId() const { return orderId; }
    double getTotal() const { return total; }
};
int Order::counter = 0;

class FoodDeliverySystem {
    map<string, Restaurant*> restaurants;
    map<string, DeliveryAgent*> agents;
    map<string, Order*> orders;

public:
    void addRestaurant(Restaurant* r) { restaurants[r->getId()] = r; }
    void addAgent(DeliveryAgent* a) { agents[a->getId()] = a; }

    void listRestaurants() {
        cout << "\n=== Restaurants ===\n";
        for (auto& [id, r] : restaurants)
            cout << r->getId() << ": " << r->getName()
                 << " ⭐" << r->getRating() << "\n";
    }

    Order* placeOrder(const string& userId, const string& restaurantId,
                      vector<pair<string, int>> itemRequests) {
        auto* r = restaurants.count(restaurantId) ? restaurants[restaurantId] : nullptr;
        if (!r) { cout << "Restaurant not found\n"; return nullptr; }

        vector<CartItem> items;
        for (auto& [itemId, qty] : itemRequests) {
            auto* m = r->findItem(itemId);
            if (m && m->isAvailable()) items.push_back(CartItem(m, qty));
        }

        if (items.empty()) { cout << "No valid items\n"; return nullptr; }

        auto* order = new Order(userId, r, items);
        orders[order->getId()] = order;

        cout << "Order placed! ID: " << order->getId()
             << " | Total: Rs." << order->getTotal() << "\n";

        // Auto-assign nearest available agent
        for (auto& [aid, agent] : agents) {
            if (agent->isAvailable()) {
                order->assignAgent(agent);
                break;
            }
        }

        return order;
    }

    void updateOrderStatus(const string& orderId, OrderStatus status) {
        if (orders.count(orderId)) {
            orders[orderId]->updateStatus(status);
        }
    }
};

int main() {
    FoodDeliverySystem app;

    // Setup
    Restaurant* r1 = new Restaurant("R1", "Burger King", "Fast Food", 4.2);
    r1->addMenuItem(new MenuItem("M1", "Whopper", 150));
    r1->addMenuItem(new MenuItem("M2", "Fries", 60));
    r1->addMenuItem(new MenuItem("M3", "Coke", 40));
    app.addRestaurant(r1);

    DeliveryAgent* a1 = new DeliveryAgent("A1", "Ravi");
    app.addAgent(a1);

    // User flow
    app.listRestaurants();
    r1->printMenu();

    auto* order = app.placeOrder("U001", "R1", {{"M1", 2}, {"M3", 1}});
    if (order) {
        app.updateOrderStatus(order->getId(), OrderStatus::ACCEPTED);
        app.updateOrderStatus(order->getId(), OrderStatus::PREPARING);
        app.updateOrderStatus(order->getId(), OrderStatus::PICKED_UP);
        app.updateOrderStatus(order->getId(), OrderStatus::DELIVERED);
    }
}
```

---

## 6.6 Ride Sharing System (Uber/Ola)

### Requirements

- User requests ride from source to destination
- System matches with nearest available driver
- Real-time tracking
- Fare calculation based on distance/time
- Rating system

### Class Diagram

```
┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
│    Rider     │    │    Driver    │    │    Location      │
│─────────────│    │─────────────│    │─────────────────│
│- riderId     │    │- driverId    │    │- latitude        │
│- name        │    │- name        │    │- longitude       │
│- rating      │    │- vehicleType │    └──────────────────┘
└──────────────┘    │- isAvailable │
                    │- location    │
                    └──────────────┘
         │                │
         └──────┬─────────┘
                │
         ┌──────▼──────┐
         │    Trip     │
         │─────────────│
         │- tripId     │
         │- rider      │
         │- driver     │
         │- pickup     │
         │- dropoff    │
         │- status     │
         │- fare       │
         └─────────────┘
```

### Code Skeleton

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <cmath>
using namespace std;

enum class TripStatus { REQUESTED, ACCEPTED, STARTED, COMPLETED, CANCELLED };
enum class VehicleType { BIKE, MINI, SEDAN, SUV };

struct Location {
    double lat, lng;
    Location(double la, double lo) : lat(la), lng(lo) {}
    double distanceTo(const Location& other) const {
        double dlat = lat - other.lat, dlng = lng - other.lng;
        return sqrt(dlat*dlat + dlng*dlng) * 111; // Approx km
    }
};

class Driver {
    string id, name;
    VehicleType vehicleType;
    double rating;
    bool available;
    Location location;

public:
    Driver(string i, string n, VehicleType vt, Location loc)
        : id(i), name(n), vehicleType(vt), rating(4.5),
          available(true), location(loc) {}

    bool isAvailable() const { return available; }
    void setAvailable(bool a) { available = a; }
    Location getLocation() const { return location; }
    void updateLocation(Location l) { location = l; }
    string getId() const { return id; }
    string getName() const { return name; }
    VehicleType getVehicleType() const { return vehicleType; }
    double getRating() const { return rating; }
    void updateRating(double r) { rating = (rating + r) / 2; }
};

class Rider {
    string id, name;
    double rating;

public:
    Rider(string i, string n) : id(i), name(n), rating(4.5) {}
    string getId() const { return id; }
    string getName() const { return name; }
};

class FareCalculator {
    map<VehicleType, double> baseRate = {
        {VehicleType::BIKE, 5.0},
        {VehicleType::MINI, 8.0},
        {VehicleType::SEDAN, 12.0},
        {VehicleType::SUV, 15.0}
    };
    map<VehicleType, double> perKmRate = {
        {VehicleType::BIKE, 4.0},
        {VehicleType::MINI, 7.0},
        {VehicleType::SEDAN, 10.0},
        {VehicleType::SUV, 14.0}
    };

public:
    double calculate(VehicleType vt, double distanceKm) {
        return baseRate[vt] + perKmRate[vt] * distanceKm;
    }
};

class Trip {
    static int counter;
    string tripId;
    Rider* rider;
    Driver* driver;
    Location pickup, dropoff;
    TripStatus status;
    double fare;
    double driverRating, riderRating;

public:
    Trip(Rider* r, Driver* d, Location pickup, Location dropoff, double fare)
        : tripId("TRIP" + to_string(++counter)),
          rider(r), driver(d), pickup(pickup), dropoff(dropoff),
          status(TripStatus::ACCEPTED), fare(fare),
          driverRating(0), riderRating(0) {}

    void start() {
        status = TripStatus::STARTED;
        cout << "Trip " << tripId << " started!\n";
    }

    void complete() {
        status = TripStatus::COMPLETED;
        driver->setAvailable(true);
        cout << "Trip " << tripId << " completed! Fare: Rs." << fare << "\n";
    }

    void rateDriver(double rating) {
        driverRating = rating;
        driver->updateRating(rating);
    }

    string getId() const { return tripId; }
    double getFare() const { return fare; }
    TripStatus getStatus() const { return status; }
};
int Trip::counter = 0;

class DriverMatcher {
public:
    Driver* findNearest(vector<Driver*>& drivers, const Location& pickup, VehicleType preferred) {
        Driver* best = nullptr;
        double minDist = 1e9;

        for (auto* d : drivers) {
            if (!d->isAvailable()) continue;
            if (preferred != VehicleType::BIKE && d->getVehicleType() != preferred) continue;

            double dist = d->getLocation().distanceTo(pickup);
            if (dist < minDist) { minDist = dist; best = d; }
        }
        return best;
    }
};

class RideSharingSystem {
    vector<Driver*> drivers;
    map<string, Rider*> riders;
    map<string, Trip*> trips;
    FareCalculator fareCalc;
    DriverMatcher matcher;

public:
    void registerDriver(Driver* d) { drivers.push_back(d); }
    void registerRider(Rider* r) { riders[r->getId()] = r; }

    Trip* requestRide(const string& riderId, Location pickup,
                      Location dropoff, VehicleType vtype) {
        auto* rider = riders.count(riderId) ? riders[riderId] : nullptr;
        if (!rider) { cout << "Rider not found\n"; return nullptr; }

        Driver* driver = matcher.findNearest(drivers, pickup, vtype);
        if (!driver) { cout << "No available drivers nearby\n"; return nullptr; }

        double distance = pickup.distanceTo(dropoff);
        double fare = fareCalc.calculate(driver->getVehicleType(), distance);

        auto* trip = new Trip(rider, driver, pickup, dropoff, fare);
        trips[trip->getId()] = trip;
        driver->setAvailable(false);

        cout << "Ride matched! Driver: " << driver->getName()
             << " | ETA: " << (int)(driver->getLocation().distanceTo(pickup) * 3) << " mins"
             << " | Est. Fare: Rs." << fare << "\n";

        return trip;
    }

    void startTrip(const string& tripId) {
        if (trips.count(tripId)) trips[tripId]->start();
    }

    void completeTrip(const string& tripId) {
        if (trips.count(tripId)) trips[tripId]->complete();
    }
};

int main() {
    RideSharingSystem app;

    Driver* d1 = new Driver("D1", "Suresh", VehicleType::SEDAN, Location(12.97, 77.59));
    Driver* d2 = new Driver("D2", "Rajan", VehicleType::MINI, Location(12.93, 77.61));
    app.registerDriver(d1);
    app.registerDriver(d2);

    Rider* r1 = new Rider("R1", "Priya");
    app.registerRider(r1);

    Location pickup(12.95, 77.60), dropoff(13.01, 77.55);
    auto* trip = app.requestRide("R1", pickup, dropoff, VehicleType::SEDAN);

    if (trip) {
        app.startTrip(trip->getId());
        app.completeTrip(trip->getId());
    }
}
```

---

## 6.7 Splitwise System

### Requirements

- Users share expenses
- Track who owes whom
- Settle debts
- Different split types: equal, exact, percentage

### Code Skeleton

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>
using namespace std;

class User {
    string id, name, email;
public:
    User(string i, string n, string e) : id(i), name(n), email(e) {}
    string getId() const { return id; }
    string getName() const { return name; }
};

enum class SplitType { EQUAL, EXACT, PERCENTAGE };

class Split {
public:
    User* user;
    double amount;
    Split(User* u, double a) : user(u), amount(a) {}
};

class Expense {
    static int counter;
    string expenseId;
    User* paidBy;
    double totalAmount;
    string description;
    vector<Split> splits;

public:
    Expense(User* paid, double amount, string desc, vector<Split> s)
        : expenseId("EXP" + to_string(++counter)),
          paidBy(paid), totalAmount(amount), description(desc), splits(s) {}

    User* getPaidBy() const { return paidBy; }
    const vector<Split>& getSplits() const { return splits; }
    double getTotal() const { return totalAmount; }
    string getId() const { return expenseId; }
};
int Expense::counter = 0;

class SplitCalculator {
public:
    static vector<Split> equalSplit(const vector<User*>& users, double total) {
        double each = total / users.size();
        vector<Split> splits;
        for (auto* u : users) splits.emplace_back(u, each);
        return splits;
    }

    static vector<Split> exactSplit(const vector<pair<User*, double>>& userAmounts) {
        vector<Split> splits;
        for (auto& [u, a] : userAmounts) splits.emplace_back(u, a);
        return splits;
    }

    static vector<Split> percentageSplit(const vector<pair<User*, double>>& userPercentages, double total) {
        vector<Split> splits;
        for (auto& [u, pct] : userPercentages)
            splits.emplace_back(u, total * pct / 100.0);
        return splits;
    }
};

class SplitwiseSystem {
    map<string, User*> users;
    vector<Expense*> expenses;
    // balances[A][B] = amount A owes B (can be negative)
    map<string, map<string, double>> balances;

public:
    void addUser(User* u) { users[u->getId()] = u; }

    void addExpense(User* paidBy, double total, string desc,
                    SplitType type, vector<User*> participants,
                    vector<double> amounts = {}) {
        vector<Split> splits;

        if (type == SplitType::EQUAL) {
            splits = SplitCalculator::equalSplit(participants, total);
        } else if (type == SplitType::EXACT) {
            vector<pair<User*, double>> exact;
            for (int i = 0; i < participants.size(); i++)
                exact.push_back({participants[i], amounts[i]});
            splits = SplitCalculator::exactSplit(exact);
        }

        auto* expense = new Expense(paidBy, total, desc, splits);
        expenses.push_back(expense);

        // Update balances
        for (auto& split : splits) {
            if (split.user->getId() != paidBy->getId()) {
                // split.user owes paidBy this amount
                balances[split.user->getId()][paidBy->getId()] += split.amount;
                balances[paidBy->getId()][split.user->getId()] -= split.amount;
            }
        }
        cout << "Expense added: " << desc << " Rs." << total << "\n";
    }

    void showBalances(User* u) {
        cout << "\n=== Balances for " << u->getName() << " ===\n";
        for (auto& [otherId, amount] : balances[u->getId()]) {
            if (abs(amount) < 0.01) continue;
            string other = users.count(otherId) ? users[otherId]->getName() : otherId;
            if (amount > 0) cout << u->getName() << " owes " << other << " Rs." << amount << "\n";
            else cout << other << " owes " << u->getName() << " Rs." << -amount << "\n";
        }
    }

    void settle(User* from, User* to, double amount) {
        balances[from->getId()][to->getId()] -= amount;
        balances[to->getId()][from->getId()] += amount;
        cout << from->getName() << " paid " << to->getName() << " Rs." << amount << "\n";
    }
};

int main() {
    SplitwiseSystem app;

    User* alice = new User("U1", "Alice", "alice@email.com");
    User* bob   = new User("U2", "Bob", "bob@email.com");
    User* carol = new User("U3", "Carol", "carol@email.com");
    app.addUser(alice); app.addUser(bob); app.addUser(carol);

    // Alice pays Rs.300 for dinner, split equally
    app.addExpense(alice, 300, "Dinner", SplitType::EQUAL, {alice, bob, carol});

    // Bob pays Rs.200 for movie, split equally
    app.addExpense(bob, 200, "Movie", SplitType::EQUAL, {alice, bob});

    app.showBalances(alice);
    app.showBalances(bob);

    // Bob settles with Alice
    app.settle(bob, alice, 100);
    app.showBalances(bob);
}
```

---

## 6.8 Online Shopping System (Amazon)

### Class Diagram

```
┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
│    User      │    │    Product   │    │  ShoppingCart    │
│─────────────│    │─────────────│    │─────────────────│
│- userId      │    │- productId   │    │- items[]         │
│- name        │    │- name        │    │─────────────────│
│- address[]   │    │- price       │    │+ addItem()       │
│- cart        │    │- inventory   │    │+ removeItem()    │
│- orders[]    │    │- category    │    │+ getTotal()      │
└──────────────┘    └──────────────┘    └──────────────────┘

┌──────────────────┐    ┌──────────────┐
│      Order       │    │  Payment     │
│─────────────────│    │─────────────│
│- orderId         │    │- paymentId   │
│- items[]         │    │- method      │
│- status          │    │- amount      │
│- shippingAddress │    │- status      │
│- payment         │    └──────────────┘
│- total           │
└──────────────────┘
```

### Code Skeleton

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>
using namespace std;

enum class OrderStatus { PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED };
enum class PaymentMethod { CREDIT_CARD, UPI, COD, WALLET };

class Address {
public:
    string street, city, state, zip;
    Address(string s, string c, string st, string z)
        : street(s), city(c), state(st), zip(z) {}
    string toString() const { return street + ", " + city + ", " + state + " " + zip; }
};

class Product {
    string id, name, category;
    double price;
    int inventory;
    double rating;

public:
    Product(string i, string n, string cat, double p, int inv)
        : id(i), name(n), category(cat), price(p), inventory(inv), rating(4.0) {}

    bool isAvailable(int qty = 1) const { return inventory >= qty; }
    bool reserve(int qty) {
        if (!isAvailable(qty)) return false;
        inventory -= qty;
        return true;
    }
    void restock(int qty) { inventory += qty; }

    string getId() const { return id; }
    string getName() const { return name; }
    double getPrice() const { return price; }
    int getInventory() const { return inventory; }
};

class CartItem {
public:
    Product* product;
    int quantity;
    CartItem(Product* p, int q) : product(p), quantity(q) {}
    double subtotal() const { return product->getPrice() * quantity; }
};

class ShoppingCart {
    vector<CartItem> items;

public:
    bool addItem(Product* p, int qty = 1) {
        if (!p->isAvailable(qty)) {
            cout << p->getName() << " not available in required quantity\n";
            return false;
        }
        for (auto& item : items) {
            if (item.product->getId() == p->getId()) {
                item.quantity += qty;
                return true;
            }
        }
        items.emplace_back(p, qty);
        cout << p->getName() << " added to cart\n";
        return true;
    }

    void removeItem(const string& productId) {
        items.erase(remove_if(items.begin(), items.end(),
            [&](const CartItem& ci) { return ci.product->getId() == productId; }),
            items.end());
    }

    double getTotal() const {
        double total = 0;
        for (auto& item : items) total += item.subtotal();
        return total;
    }

    const vector<CartItem>& getItems() const { return items; }
    void clear() { items.clear(); }
    bool isEmpty() const { return items.empty(); }

    void printCart() {
        cout << "\n=== Shopping Cart ===\n";
        for (auto& item : items)
            cout << item.product->getName() << " x" << item.quantity
                 << " = Rs." << item.subtotal() << "\n";
        cout << "Total: Rs." << getTotal() << "\n";
    }
};

class Payment {
    static int counter;
    string paymentId;
    PaymentMethod method;
    double amount;
    bool success;

public:
    Payment(PaymentMethod m, double a) : paymentId("PAY" + to_string(++counter)),
        method(m), amount(a), success(false) {}

    bool process() {
        // Simulate payment processing
        success = true; // In real system: call payment gateway
        cout << "Payment " << paymentId << " of Rs." << amount << " processed\n";
        return success;
    }

    bool isSuccessful() const { return success; }
    string getId() const { return paymentId; }
};
int Payment::counter = 0;

class Order {
    static int counter;
    string orderId;
    string userId;
    vector<CartItem> items;
    Address shippingAddress;
    OrderStatus status;
    Payment* payment;
    double total;

public:
    Order(string uid, vector<CartItem> i, Address addr, Payment* p)
        : orderId("ORD" + to_string(++counter)), userId(uid),
          items(i), shippingAddress(addr), status(OrderStatus::CONFIRMED),
          payment(p), total(0) {
        for (auto& item : items) total += item.subtotal();
    }

    void updateStatus(OrderStatus s) {
        status = s;
        cout << "Order " << orderId << " → " << statusToString() << "\n";
    }

    string statusToString() const {
        switch (status) {
            case OrderStatus::PENDING:    return "PENDING";
            case OrderStatus::CONFIRMED:  return "CONFIRMED";
            case OrderStatus::SHIPPED:    return "SHIPPED";
            case OrderStatus::DELIVERED:  return "DELIVERED";
            default: return "CANCELLED";
        }
    }

    void printReceipt() {
        cout << "\n====== ORDER RECEIPT ======\n";
        cout << "Order ID: " << orderId << "\n";
        cout << "Ship to: " << shippingAddress.toString() << "\n";
        for (auto& item : items)
            cout << "  " << item.product->getName() << " x" << item.quantity
                 << " = Rs." << item.subtotal() << "\n";
        cout << "Total: Rs." << total << "\n";
        cout << "Payment: " << payment->getId() << "\n";
        cout << "===========================\n";
    }

    string getId() const { return orderId; }
};
int Order::counter = 0;

class ProductCatalog {
    map<string, Product*> products;
    map<string, vector<Product*>> categoryIndex;

public:
    void addProduct(Product* p) {
        products[p->getId()] = p;
    }

    Product* findById(const string& id) {
        return products.count(id) ? products[id] : nullptr;
    }

    vector<Product*> search(const string& keyword) {
        vector<Product*> result;
        for (auto& [id, p] : products)
            if (p->getName().find(keyword) != string::npos)
                result.push_back(p);
        return result;
    }
};

class OnlineShoppingSystem {
    ProductCatalog catalog;
    map<string, ShoppingCart*> carts;   // userId → cart
    map<string, vector<Order*>> orders; // userId → orders

public:
    void addProduct(Product* p) { catalog.addProduct(p); }

    ShoppingCart* getCart(const string& userId) {
        if (!carts.count(userId)) carts[userId] = new ShoppingCart();
        return carts[userId];
    }

    Order* placeOrder(const string& userId, const string& addressStr, PaymentMethod pm) {
        auto* cart = getCart(userId);
        if (cart->isEmpty()) { cout << "Cart is empty\n"; return nullptr; }

        // Reserve inventory
        for (auto& item : cart->getItems()) {
            if (!item.product->reserve(item.quantity)) {
                cout << "Failed to reserve " << item.product->getName() << "\n";
                return nullptr;
            }
        }

        double total = cart->getTotal();
        auto* payment = new Payment(pm, total);
        if (!payment->process()) {
            cout << "Payment failed\n";
            return nullptr;
        }

        Address addr("123 MG Road", "Bangalore", "Karnataka", "560001");
        auto* order = new Order(userId, cart->getItems(), addr, payment);
        orders[userId].push_back(order);
        cart->clear();
        order->printReceipt();
        return order;
    }
};

int main() {
    OnlineShoppingSystem app;

    Product* phone = new Product("P1", "iPhone 15", "Electronics", 79999, 100);
    Product* cover = new Product("P2", "Phone Cover", "Accessories", 999, 500);
    app.addProduct(phone);
    app.addProduct(cover);

    auto* cart = app.getCart("U001");
    cart->addItem(phone, 1);
    cart->addItem(cover, 2);
    cart->printCart();

    auto* order = app.placeOrder("U001", "Bangalore", PaymentMethod::UPI);
    if (order) {
        order->updateStatus(OrderStatus::SHIPPED);
        order->updateStatus(OrderStatus::DELIVERED);
    }
}
```

---

# 📚 PART 7: Advanced Topics

## 7.1 Dependency Injection (DI)

Instead of a class creating its own dependencies, they are **injected from outside**.

```cpp
// Without DI (tightly coupled):
class UserService {
    MySQLDB db;        // Created internally — can't swap
    SMTPMailer mailer; // Created internally — can't swap
};

// With DI (loosely coupled):
class UserService {
    IDatabase& db;
    IMailer& mailer;
public:
    // Dependencies injected through constructor
    UserService(IDatabase& d, IMailer& m) : db(d), mailer(m) {}
};

// At composition root (main):
MySQLDB db;
SMTPMailer mailer;
UserService service(db, mailer); // Inject here
```

**Benefits:**
- Easy to test (inject mock dependencies)
- Easy to swap implementations
- Explicit dependencies (no hidden coupling)

---

## 7.2 Inversion of Control (IoC)

DI is one way to achieve IoC. The principle: "Don't call us, we'll call you."

Traditional flow: `Main → creates → Services → creates → Repositories`
IoC flow: `IoC Container → creates and wires → everything`

DI containers (Spring in Java, Boost.DI in C++) automate this wiring.

---

## 7.3 Interface vs Abstract Class

| | Interface | Abstract Class |
|---|---|---|
| State | No | Yes |
| Constructor | No | Yes |
| Full implementation | No (Java 8+: default methods) | Yes |
| Multiple inheritance | Yes | Careful |
| When to use | Pure contract (CAN-DO) | Partial implementation (IS-A) |

**Rule of thumb:** If something IS something, use abstract class. If something CAN DO something, use interface.

```
Dog IS-A Animal → abstract class/inheritance
Duck CAN-DO Flyable, Swimmable → interfaces
```

---

## 7.4 Composition Over Inheritance

**Why prefer composition?**

1. Inheritance is a compile-time relationship — you can't change it at runtime
2. Inheritance exposes internal details to subclasses (breaks encapsulation)
3. Composition is more flexible — swap behaviors at runtime

```cpp
// Inheritance — rigid
class Duck : public Bird, public Swimmable {
    // Behavior hardcoded at compile time
};

// Composition — flexible (Strategy pattern)
class FlyBehavior { public: virtual void fly() = 0; };
class SwimBehavior { public: virtual void swim() = 0; };

class Duck {
    FlyBehavior* flyBehavior;
    SwimBehavior* swimBehavior;
public:
    Duck(FlyBehavior* f, SwimBehavior* s) : flyBehavior(f), swimBehavior(s) {}

    void setFlyBehavior(FlyBehavior* f) { flyBehavior = f; } // Change at runtime!
    void fly() { flyBehavior->fly(); }
    void swim() { swimBehavior->swim(); }
};
```

---

## 7.5 Immutability

An immutable object's state cannot be changed after creation. Safer in multithreaded environments.

```cpp
class ImmutablePoint {
    const double x, y;  // const members

public:
    ImmutablePoint(double x, double y) : x(x), y(y) {}

    double getX() const { return x; }
    double getY() const { return y; }

    // Don't modify — return new object
    ImmutablePoint translate(double dx, double dy) const {
        return ImmutablePoint(x + dx, y + dy);
    }
};
```

**Benefits:**
- Thread-safe by default (no shared mutable state)
- Easy to reason about
- Safe to cache and share

---

## 7.6 Thread Safety Basics

```cpp
class ThreadSafeCounter {
    int count = 0;
    mutable mutex mtx;

public:
    void increment() {
        lock_guard<mutex> lock(mtx);  // Auto-released when out of scope
        count++;
    }

    int get() const {
        lock_guard<mutex> lock(mtx);
        return count;
    }
};
```

**For Singleton (thread-safe):**
```cpp
static Singleton& getInstance() {
    static Singleton instance;  // C++11: guaranteed thread-safe initialization
    return instance;
}
```

---

# 📚 PART 8: Interview Preparation

## 8.1 Common OOP Interview Questions

**Q: What are the four pillars of OOP?**
A: Encapsulation (bundling + access control), Abstraction (hiding complexity), Inheritance (IS-A reuse), Polymorphism (many forms — compile-time and runtime).

**Q: What is the difference between overloading and overriding?**
A: Overloading = same name, different parameters, resolved at COMPILE time. Overriding = same name, same signature, resolved at RUNTIME via virtual dispatch.

**Q: What is the diamond problem? How is it solved?**
A: When two parent classes share a common grandparent, the child has ambiguous access to grandparent members. Solved with virtual inheritance in C++.

**Q: Why is the destructor virtual in base classes?**
A: When deleting a derived class through a base pointer, without virtual destructor, only the base destructor runs — derived resources leak.

**Q: What is the Rule of 5?**
A: If a class manages resources, define: destructor, copy constructor, copy assignment, move constructor, move assignment. Use smart pointers (Rule of 0) to avoid this.

**Q: What's the difference between composition and aggregation?**
A: Composition = strong ownership (child dies with parent). Aggregation = weak ownership (child can exist independently).

---

## 8.2 How to Approach LLD Interview Problems

### The Framework (Use This Every Time)

```
1. CLARIFY (2-3 min)
   - Ask about scale
   - Ask about key features to focus on
   - Confirm what's in/out of scope

2. IDENTIFY ENTITIES (3-5 min)
   - Nouns from requirements = classes
   - Verbs = methods

3. RELATIONSHIPS (2-3 min)
   - IS-A / HAS-A / USES-A
   - Cardinality (1:1, 1:N, M:N)

4. CLASS DIAGRAM (5-7 min)
   - Draw it on whiteboard/paper
   - Show key attributes and methods
   - Show relationships

5. CODE (10-15 min)
   - Start with the most important class
   - Skeleton first, fill in logic
   - Show design patterns used

6. DISCUSS (5 min)
   - Trade-offs made
   - How to scale
   - What you'd add with more time
```

### Communication Tips

- **Think out loud.** Say "I'm considering two options: X and Y. I'll go with X because..."
- **Ask clarifying questions.** "Should I handle concurrent requests?" shows maturity.
- **Justify decisions.** "I'm using Singleton here because there should be only one instance of..."
- **Mention trade-offs.** "This uses more memory but reduces lookup time from O(n) to O(1)."
- **SOLID principles.** Mention when you apply them. Interviewers love this.

---

## 8.3 Common Mistakes to Avoid

| Mistake | Better Approach |
|---|---|
| Jumping straight to code | Clarify requirements first |
| One giant God class | SRP — small, focused classes |
| Using `if-else` for types | Polymorphism / Strategy pattern |
| Hardcoding concrete classes | Depend on interfaces |
| Forgetting edge cases | "What if the parking lot is full?" |
| Not thinking about concurrency | Mention threading concerns |
| Skipping destructor/cleanup | Always think about resource management |

---

## 8.4 LLD Interview Questions (Commonly Asked)

**Beginner:**
- Design a Stack / Queue with generics
- Design a Tic-Tac-Toe game
- Design a basic Calculator
- Design a Traffic Light System

**Intermediate:**
- Design a Parking Lot ← very common!
- Design a Library Management System
- Design an Elevator System
- Design a Chess Game
- Design a Hotel Booking System

**Advanced:**
- Design an ATM
- Design Splitwise
- Design Uber/Ola
- Design Swiggy/Zomato
- Design an Online Shopping System (Amazon)
- Design a Movie Ticket Booking System (BookMyShow)
- Design a Rate Limiter
- Design a Cache (LRU)

---

# 📚 PART 9: Practice Section

## Beginner Problems

1. **Library Book:** Design a `Book` class with title, author, ISBN. Add methods to display info and check if overdue.

2. **Simple Bank Account:** `BankAccount` with deposit, withdraw, transfer. Handle insufficient funds.

3. **Vehicle Hierarchy:** `Vehicle` → `Car`, `Motorcycle`, `Truck`. Override `fuelEfficiency()` method.

4. **Shape Area Calculator:** Abstract `Shape` with `area()`. Implement `Circle`, `Rectangle`, `Triangle`.

5. **Student Grade System:** Students have courses and grades. Calculate GPA.

## Intermediate Design Problems

6. **Design a Deck of Cards:** 52 cards, suits, ranks. Shuffle, deal, support Poker and BlackJack.

7. **Design a Restaurant Menu + Order System:** Menu with categories, items, prices. Place orders, calculate bill.

8. **Design a Movie Theater Booking System:** Shows, seats, bookings. Handle seat selection.

9. **Design a Chat Application:** Users, messages, groups. Send/receive messages.

10. **Design an LRU Cache:** Fixed-capacity cache that evicts least-recently-used items.

## Advanced LLD Problems

11. **Design a Rate Limiter:** Limit API calls per user per time window (token bucket, sliding window).

12. **Design a Notification System:** Multiple channels (email, SMS, push). Multiple event types. User preferences.

13. **Design a File System:** Directories, files, symlinks. Create, delete, move, search.

14. **Design a Chess Game:** Pieces, board, moves, check/checkmate detection.

15. **Design a Vending Machine:** Accept coins, select product, dispense, give change.

## Thought Exercises

- **"How would you redesign Instagram's feed if you had to build it from scratch?"** Think about: `Post`, `User`, `Feed`, `FeedStrategy` (chronological vs algorithmic).

- **"How would you add 'undo/redo' to any application?"** Think about the Command pattern.

- **"How would you make the Parking Lot system handle 100 parking lots across the city?"** Think about abstractions that scale.

- **"Design a system that supports multiple payment methods (credit card, UPI, crypto). How do you add a new payment method without changing existing code?"** Hint: OCP + Strategy.

---

# 📌 Final Cheat Sheet

```
OOP:
  Encapsulation  = data + methods + access control
  Abstraction    = hide HOW, show WHAT
  Inheritance    = IS-A, reuse code, virtual for polymorphism
  Polymorphism   = one interface, many forms (overload=compile, override=runtime)

SOLID:
  S = One reason to change (SRP)
  O = Add don't modify (OCP)
  L = Subclass must substitute safely (LSP)
  I = Small focused interfaces (ISP)
  D = Depend on abstractions (DIP)

Patterns:
  Creational: Singleton, Factory, Abstract Factory, Builder, Prototype
  Structural: Adapter, Decorator, Facade, Proxy, Composite
  Behavioral: Observer, Strategy, Command, State, Template Method, Chain of Responsibility

LLD Approach:
  1. Clarify → 2. Entities → 3. Relationships → 4. Class Diagram → 5. Code → 6. Trade-offs
```

---

> 🔥 **Final Tip:** The best way to prepare is to design 2-3 systems on paper every day for a month. Write the class diagram first, then code the skeleton. Review SOLID compliance. Improve. Repeat.

*Good luck with your SDE interviews! You've got this.* 🚀
