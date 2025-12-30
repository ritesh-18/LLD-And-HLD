
# Mastering Low Level Design in C++
## A 45-Day Comprehensive Guide for Backend Engineers

**Author: Senior Staff Engineer**  
**Version: 1.0**

---

## Table of Contents

### Phase 1: C++ for LLD (Days 1-7)
1. C++ OOP Basics for LLD
2. Inheritance, Polymorphism, and Virtual Functions
3. Composition vs Inheritance
4. Memory Ownership (RAII, Smart Pointers)
5. Interfaces Using Abstract Classes
6. Mini Design: Logger System

### Phase 2: SOLID Principles (Days 8-14)
7. Single Responsibility Principle (SRP)
8. Open-Closed Principle (OCP)
9. Liskov Substitution Principle (LSP)
10. Interface Segregation Principle (ISP)
11. Dependency Inversion Principle (DIP)
12. Hands-on SOLID Refactoring

### Phase 3: Design Patterns (Days 15-25)
13. Creational Patterns
14. Structural Patterns
15. Behavioral Patterns

### Phase 4: LLD Case Studies (Days 26-35)
16. Parking Lot System
17. Elevator System
18. Notification System
19. Rate Limiter
20. Splitwise System

### Phase 5: LLD for Microservices (Days 36-45)
21. Service Boundaries and Shared Libraries
22. Idempotency and Retries
23. Circuit Breaker and Backoff Design
24. Event-Driven Architecture
25. Saga and Outbox Patterns
26. Designing for Testability
27. Final Capstone System Design

### Appendix
28. 50 LLD Interview Questions with Answers

---

# PHASE 1: C++ for LLD (Days 1-7)

## Chapter 1: C++ OOP Basics for LLD

### Introduction

Object-Oriented Programming (OOP) is the foundation of Low Level Design. In this chapter, we'll explore the core concepts that make C++ a powerful language for designing scalable systems.

### 1.1 Classes and Objects

A **class** is a blueprint for creating objects. It encapsulates data and behavior together.

```cpp
#include <iostream>
#include <string>

// Basic class definition
class User {
private:
    int userId;
    std::string username;
    std::string email;

public:
    // Constructor
    User(int id, const std::string& name, const std::string& mail) 
        : userId(id), username(name), email(mail) {
        std::cout << "User created: " << username << std::endl;
    }

    // Destructor
    ~User() {
        std::cout << "User destroyed: " << username << std::endl;
    }

    // Getter methods
    int getUserId() const { return userId; }
    std::string getUsername() const { return username; }
    std::string getEmail() const { return email; }

    // Setter methods
    void setEmail(const std::string& newEmail) {
        email = newEmail;
    }

    // Business logic method
    void displayInfo() const {
        std::cout << "User ID: " << userId << std::endl;
        std::cout << "Username: " << username << std::endl;
        std::cout << "Email: " << email << std::endl;
    }
};

int main() {
    User user1(1, "john_doe", "john@example.com");
    user1.displayInfo();
    
    user1.setEmail("john.doe@example.com");
    std::cout << "Updated email: " << user1.getEmail() << std::endl;
    
    return 0;
}
```

**Explanation:**
- **Private members**: Data is hidden from external access (encapsulation)
- **Public interface**: Methods provide controlled access to data
- **Constructor**: Initializes object state using member initializer list (efficient)
- **Destructor**: Cleans up resources when object is destroyed
- **const methods**: Indicate methods that don't modify object state

### 1.2 Access Modifiers

Access modifiers control the visibility of class members:

```cpp
class BankAccount {
private:
    // Only accessible within this class
    double balance;
    std::string accountNumber;
    
    void validateTransaction(double amount) {
        if (amount <= 0) {
            throw std::invalid_argument("Amount must be positive");
        }
    }

protected:
    // Accessible in this class and derived classes
    std::string accountType;
    
    void logTransaction(const std::string& type, double amount) {
        std::cout << "[" << type << "] Amount: $" << amount << std::endl;
    }

public:
    // Accessible from anywhere
    BankAccount(const std::string& accNum, const std::string& type)
        : accountNumber(accNum), accountType(type), balance(0.0) {}

    void deposit(double amount) {
        validateTransaction(amount);
        balance += amount;
        logTransaction("DEPOSIT", amount);
    }

    void withdraw(double amount) {
        validateTransaction(amount);
        if (balance < amount) {
            throw std::runtime_error("Insufficient funds");
        }
        balance -= amount;
        logTransaction("WITHDRAW", amount);
    }

    double getBalance() const {
        return balance;
    }
};

// Derived class demonstrating protected access
class SavingsAccount : public BankAccount {
private:
    double interestRate;

public:
    SavingsAccount(const std::string& accNum, double rate)
        : BankAccount(accNum, "SAVINGS"), interestRate(rate) {}

    void applyInterest() {
        double interest = getBalance() * interestRate / 100.0;
        // Can access protected members from base class
        logTransaction("INTEREST", interest);
    }
};
```

**Key Points:**
- **private**: Strongest encapsulation, implementation details hidden
- **protected**: Allows inheritance while maintaining encapsulation
- **public**: API exposed to clients

### 1.3 Constructors Deep Dive

```cpp
#include <vector>
#include <memory>

class Order {
private:
    int orderId;
    std::string customerName;
    std::vector<std::string> items;
    double totalAmount;

public:
    // Default constructor
    Order() : orderId(0), customerName(""), totalAmount(0.0) {
        std::cout << "Default constructor called" << std::endl;
    }

    // Parameterized constructor
    Order(int id, const std::string& customer)
        : orderId(id), customerName(customer), totalAmount(0.0) {
        std::cout << "Parameterized constructor called" << std::endl;
    }

    // Constructor with all parameters
    Order(int id, const std::string& customer, 
          const std::vector<std::string>& orderItems, double amount)
        : orderId(id), customerName(customer), 
          items(orderItems), totalAmount(amount) {
        std::cout << "Full constructor called" << std::endl;
    }

    // Copy constructor
    Order(const Order& other)
        : orderId(other.orderId), customerName(other.customerName),
          items(other.items), totalAmount(other.totalAmount) {
        std::cout << "Copy constructor called" << std::endl;
    }

    // Move constructor (C++11)
    Order(Order&& other) noexcept
        : orderId(other.orderId), customerName(std::move(other.customerName)),
          items(std::move(other.items)), totalAmount(other.totalAmount) {
        std::cout << "Move constructor called" << std::endl;
        other.orderId = 0;
        other.totalAmount = 0.0;
    }

    // Copy assignment operator
    Order& operator=(const Order& other) {
        std::cout << "Copy assignment operator called" << std::endl;
        if (this != &other) {
            orderId = other.orderId;
            customerName = other.customerName;
            items = other.items;
            totalAmount = other.totalAmount;
        }
        return *this;
    }

    // Move assignment operator
    Order& operator=(Order&& other) noexcept {
        std::cout << "Move assignment operator called" << std::endl;
        if (this != &other) {
            orderId = other.orderId;
            customerName = std::move(other.customerName);
            items = std::move(other.items);
            totalAmount = other.totalAmount;
            
            other.orderId = 0;
            other.totalAmount = 0.0;
        }
        return *this;
    }

    void display() const {
        std::cout << "Order ID: " << orderId << ", Customer: " 
                  << customerName << ", Total: $" << totalAmount << std::endl;
    }
};

// Demonstration
void demonstrateConstructors() {
    Order o1;  // Default constructor
    Order o2(101, "Alice");  // Parameterized constructor
    
    std::vector<std::string> items = {"Laptop", "Mouse"};
    Order o3(102, "Bob", items, 1500.0);  // Full constructor
    
    Order o4 = o3;  // Copy constructor
    Order o5(std::move(o2));  // Move constructor
    
    o1 = o3;  // Copy assignment
    o1 = std::move(o5);  // Move assignment
}
```

**Rule of Five:**
When you define any of these, consider defining all:
1. Destructor
2. Copy constructor
3. Move constructor
4. Copy assignment operator
5. Move assignment operator

---

## Chapter 2: Inheritance, Polymorphism, and Virtual Functions

### 2.1 Inheritance Fundamentals

Inheritance enables code reuse and establishes "is-a" relationships.

```cpp
#include <iostream>
#include <string>
#include <memory>

// Base class
class Vehicle {
protected:
    std::string registrationNumber;
    std::string manufacturer;
    int yearOfManufacture;

public:
    Vehicle(const std::string& regNum, const std::string& mfr, int year)
        : registrationNumber(regNum), manufacturer(mfr), 
          yearOfManufacture(year) {
        std::cout << "Vehicle constructor called" << std::endl;
    }

    virtual ~Vehicle() {
        std::cout << "Vehicle destructor called" << std::endl;
    }

    // Virtual function - can be overridden
    virtual void start() {
        std::cout << "Vehicle starting..." << std::endl;
    }

    virtual void stop() {
        std::cout << "Vehicle stopping..." << std::endl;
    }

    // Pure virtual function - must be implemented by derived classes
    virtual void displayType() const = 0;

    // Non-virtual function
    void displayInfo() const {
        std::cout << "Registration: " << registrationNumber << std::endl;
        std::cout << "Manufacturer: " << manufacturer << std::endl;
        std::cout << "Year: " << yearOfManufacture << std::endl;
    }
};

// Derived class: Car
class Car : public Vehicle {
private:
    int numberOfDoors;
    std::string fuelType;

public:
    Car(const std::string& regNum, const std::string& mfr, 
        int year, int doors, const std::string& fuel)
        : Vehicle(regNum, mfr, year), numberOfDoors(doors), fuelType(fuel) {
        std::cout << "Car constructor called" << std::endl;
    }

    ~Car() override {
        std::cout << "Car destructor called" << std::endl;
    }

    // Override virtual functions
    void start() override {
        std::cout << "Car engine starting with " << fuelType << "..." << std::endl;
    }

    void displayType() const override {
        std::cout << "Type: Car (" << numberOfDoors << " doors)" << std::endl;
    }

    // Car-specific method
    void openTrunk() {
        std::cout << "Trunk opened" << std::endl;
    }
};

// Derived class: Motorcycle
class Motorcycle : public Vehicle {
private:
    bool hasSidecar;

public:
    Motorcycle(const std::string& regNum, const std::string& mfr, 
               int year, bool sidecar)
        : Vehicle(regNum, mfr, year), hasSidecar(sidecar) {
        std::cout << "Motorcycle constructor called" << std::endl;
    }

    ~Motorcycle() override {
        std::cout << "Motorcycle destructor called" << std::endl;
    }

    void start() override {
        std::cout << "Motorcycle engine roaring to life..." << std::endl;
    }

    void displayType() const override {
        std::cout << "Type: Motorcycle" 
                  << (hasSidecar ? " (with sidecar)" : "") << std::endl;
    }

    void wheelie() {
        if (!hasSidecar) {
            std::cout << "Performing wheelie!" << std::endl;
        } else {
            std::cout << "Cannot wheelie with sidecar" << std::endl;
        }
    }
};
```

### 2.2 Polymorphism in Action

```cpp
// Demonstrating runtime polymorphism
class VehicleManager {
private:
    std::vector<std::unique_ptr<Vehicle>> vehicles;

public:
    void addVehicle(std::unique_ptr<Vehicle> vehicle) {
        vehicles.push_back(std::move(vehicle));
    }

    void startAll() {
        std::cout << "\n=== Starting all vehicles ===" << std::endl;
        for (const auto& vehicle : vehicles) {
            vehicle->start();  // Runtime polymorphism
            vehicle->displayType();
            std::cout << "---" << std::endl;
        }
    }

    void stopAll() {
        std::cout << "\n=== Stopping all vehicles ===" << std::endl;
        for (const auto& vehicle : vehicles) {
            vehicle->stop();
        }
    }
};

// Usage example
void polymorphismDemo() {
    VehicleManager manager;

    // Add different types of vehicles
    manager.addVehicle(
        std::make_unique<Car>("CAR-001", "Toyota", 2022, 4, "Petrol")
    );
    manager.addVehicle(
        std::make_unique<Car>("CAR-002", "Tesla", 2023, 2, "Electric")
    );
    manager.addVehicle(
        std::make_unique<Motorcycle>("BIKE-001", "Harley", 2021, false)
    );

    // Polymorphic behavior
    manager.startAll();
    manager.stopAll();
}
```

**Key Concepts:**
- **virtual**: Enables runtime polymorphism through vtable
- **override**: Explicitly marks function as overriding base class
- **Pure virtual (= 0)**: Makes class abstract, forces implementation
- **Virtual destructor**: Critical for proper cleanup in inheritance hierarchies

### 2.3 Virtual Function Table (VTable) Internals

```cpp
// Understanding how virtual functions work internally

class Base {
public:
    virtual void func1() { 
        std::cout << "Base::func1" << std::endl; 
    }
    virtual void func2() { 
        std::cout << "Base::func2" << std::endl; 
    }
    void func3() {  // Non-virtual
        std::cout << "Base::func3" << std::endl; 
    }
};

class Derived : public Base {
public:
    void func1() override { 
        std::cout << "Derived::func1" << std::endl; 
    }
    // func2 not overridden - will use Base's implementation
    void func3() {  // Hides Base::func3, doesn't override
        std::cout << "Derived::func3" << std::endl; 
    }
};

void vtableDemo() {
    Base* ptr1 = new Base();
    Base* ptr2 = new Derived();  // Base pointer to Derived object
    Derived* ptr3 = new Derived();

    std::cout << "=== Virtual function calls ===" << std::endl;
    ptr1->func1();  // Base::func1
    ptr2->func1();  // Derived::func1 (runtime polymorphism)
    ptr3->func1();  // Derived::func1

    std::cout << "\n=== Non-virtual function calls ===" << std::endl;
    ptr1->func3();  // Base::func3
    ptr2->func3();  // Base::func3 (static binding)
    ptr3->func3();  // Derived::func3

    delete ptr1;
    delete ptr2;
    delete ptr3;
}
```

---

## Chapter 3: Composition vs Inheritance

### 3.1 Understanding Composition

Composition represents "has-a" relationships and is often preferred over inheritance for flexibility.

```cpp
#include <iostream>
#include <string>
#include <memory>
#include <vector>

// Component classes
class Engine {
private:
    int horsepower;
    std::string type;

public:
    Engine(int hp, const std::string& engineType)
        : horsepower(hp), type(engineType) {}

    void start() {
        std::cout << type << " engine (" << horsepower 
                  << " HP) starting..." << std::endl;
    }

    void stop() {
        std::cout << "Engine stopping..." << std::endl;
    }

    int getHorsepower() const { return horsepower; }
};

class Transmission {
private:
    std::string type;  // "Automatic" or "Manual"
    int numberOfGears;

public:
    Transmission(const std::string& transType, int gears)
        : type(transType), numberOfGears(gears) {}

    void shiftGear(int gear) {
        if (gear >= 1 && gear <= numberOfGears) {
            std::cout << "Shifting to gear " << gear << std::endl;
        }
    }

    std::string getType() const { return type; }
};

class FuelTank {
private:
    double capacity;
    double currentLevel;

public:
    FuelTank(double cap) : capacity(cap), currentLevel(cap) {}

    bool consumeFuel(double amount) {
        if (currentLevel >= amount) {
            currentLevel -= amount;
            return true;
        }
        std::cout << "Not enough fuel!" << std::endl;
        return false;
    }

    void refuel(double amount) {
        currentLevel = std::min(capacity, currentLevel + amount);
        std::cout << "Refueled. Current level: " << currentLevel 
                  << "/" << capacity << std::endl;
    }

    double getFuelLevel() const { return currentLevel; }
};

// Car using composition
class ComposedCar {
private:
    std::string model;
    std::unique_ptr<Engine> engine;
    std::unique_ptr<Transmission> transmission;
    std::unique_ptr<FuelTank> fuelTank;

public:
    ComposedCar(const std::string& carModel, 
                std::unique_ptr<Engine> eng,
                std::unique_ptr<Transmission> trans,
                std::unique_ptr<FuelTank> tank)
        : model(carModel), 
          engine(std::move(eng)),
          transmission(std::move(trans)),
          fuelTank(std::move(tank)) {}

    void start() {
        std::cout << "Starting " << model << std::endl;
        if (fuelTank->getFuelLevel() > 0) {
            engine->start();
        } else {
            std::cout << "Cannot start: No fuel!" << std::endl;
        }
    }

    void drive(int gear, double fuelConsumption) {
        if (fuelTank->consumeFuel(fuelConsumption)) {
            transmission->shiftGear(gear);
            std::cout << model << " is driving..." << std::endl;
        }
    }

    void stop() {
        engine->stop();
        std::cout << model << " stopped." << std::endl;
    }

    void displaySpecs() const {
        std::cout << "\n=== " << model << " Specifications ===" << std::endl;
        std::cout << "Engine: " << engine->getHorsepower() << " HP" << std::endl;
        std::cout << "Transmission: " << transmission->getType() << std::endl;
        std::cout << "Fuel Level: " << fuelTank->getFuelLevel() << std::endl;
    }
};
```

### 3.2 Composition vs Inheritance: A Comparison

```cpp
// INHERITANCE APPROACH (tightly coupled)
class VehicleBase {
protected:
    Engine engine;
    Transmission transmission;
public:
    VehicleBase(const Engine& eng, const Transmission& trans)
        : engine(eng), transmission(trans) {}
    
    virtual void operate() = 0;
};

class SportsCar : public VehicleBase {
public:
    SportsCar() : VehicleBase(Engine(500, "V8"), 
                               Transmission("Manual", 6)) {}
    
    void operate() override {
        // Tightly coupled to parent implementation
    }
};

// COMPOSITION APPROACH (loosely coupled, flexible)
class FlexibleCar {
private:
    std::unique_ptr<Engine> engine;
    std::unique_ptr<Transmission> transmission;

public:
    // Can inject any engine and transmission at runtime
    void setEngine(std::unique_ptr<Engine> newEngine) {
        engine = std::move(newEngine);
    }

    void setTransmission(std::unique_ptr<Transmission> newTrans) {
        transmission = std::move(newTrans);
    }

    // Easy to swap components without changing class hierarchy
};
```

### 3.3 Real-World Example: Notification System

```cpp
// Bad: Using inheritance
class Notification {
public:
    virtual void send(const std::string& message) = 0;
};

class EmailNotification : public Notification {
    // Stuck with this implementation
};

class SMSNotification : public Notification {
    // Cannot combine with email easily
};

// Good: Using composition
class NotificationChannel {
public:
    virtual void send(const std::string& message) = 0;
    virtual ~NotificationChannel() = default;
};

class EmailChannel : public NotificationChannel {
public:
    void send(const std::string& message) override {
        std::cout << "Sending email: " << message << std::endl;
    }
};

class SMSChannel : public NotificationChannel {
public:
    void send(const std::string& message) override {
        std::cout << "Sending SMS: " << message << std::endl;
    }
};

class PushChannel : public NotificationChannel {
public:
    void send(const std::string& message) override {
        std::cout << "Sending push notification: " << message << std::endl;
    }
};

// Flexible notification service using composition
class NotificationService {
private:
    std::vector<std::unique_ptr<NotificationChannel>> channels;

public:
    void addChannel(std::unique_ptr<NotificationChannel> channel) {
        channels.push_back(std::move(channel));
    }

    void removeChannel(size_t index) {
        if (index < channels.size()) {
            channels.erase(channels.begin() + index);
        }
    }

    void sendNotification(const std::string& message) {
        std::cout << "\n=== Sending notification ===" << std::endl;
        for (const auto& channel : channels) {
            channel->send(message);
        }
    }
};

// Usage
void compositionDemo() {
    NotificationService service;
    
    // Dynamically compose notification channels
    service.addChannel(std::make_unique<EmailChannel>());
    service.addChannel(std::make_unique<SMSChannel>());
    service.addChannel(std::make_unique<PushChannel>());
    
    service.sendNotification("Your order has been shipped!");
    
    // Easy to add/remove channels at runtime
    service.removeChannel(1);  // Remove SMS
    service.sendNotification("Payment received");
}
```

**When to Use:**
- **Inheritance**: "is-a" relationship, shared interface needed
- **Composition**: "has-a" relationship, flexibility, runtime behavior change

---

## Chapter 4: Memory Ownership (RAII, Smart Pointers)

### 4.1 RAII (Resource Acquisition Is Initialization)

RAII is a fundamental C++ idiom that ties resource management to object lifetime.

```cpp
#include <iostream>
#include <fstream>
#include <mutex>
#include <memory>

// Bad: Manual resource management
class FileHandlerBad {
private:
    std::FILE* file;

public:
    FileHandlerBad(const std::string& filename) {
        file = std::fopen(filename.c_str(), "w");
        if (!file) {
            throw std::runtime_error("Failed to open file");
        }
    }

    void write(const std::string& data) {
        std::fputs(data.c_str(), file);
    }

    // Must remember to call this!
    void close() {
        if (file) {
            std::fclose(file);
            file = nullptr;
        }
    }
    
    // If exception occurs before close(), resource leaks!
};

// Good: RAII-based resource management
class FileHandlerRAII {
private:
    std::FILE* file;

public:
    FileHandlerRAII(const std::string& filename) {
        file = std::fopen(filename.c_str(), "w");
        if (!file) {
            throw std::runtime_error("Failed to open file");
        }
        std::cout << "File opened: " << filename << std::endl;
    }

    // Destructor automatically releases resource
    ~FileHandlerRAII() {
        if (file) {
            std::fclose(file);
            std::cout << "File closed automatically" << std::endl;
        }
    }

    // Delete copy operations (file handle shouldn't be copied)
    FileHandlerRAII(const FileHandlerRAII&) = delete;
    FileHandlerRAII& operator=(const FileHandlerRAII&) = delete;

    // Allow move operations
    FileHandlerRAII(FileHandlerRAII&& other) noexcept : file(other.file) {
        other.file = nullptr;
    }

    FileHandlerRAII& operator=(FileHandlerRAII&& other) noexcept {
        if (this != &other) {
            if (file) std::fclose(file);
            file = other.file;
            other.file = nullptr;
        }
        return *this;
    }

    void write(const std::string& data) {
        if (file) {
            std::fputs(data.c_str(), file);
        }
    }
};

// RAII with std::lock_guard
class ThreadSafeCounter {
private:
    int count;
    mutable std::mutex mtx;

public:
    ThreadSafeCounter() : count(0) {}

    void increment() {
        std::lock_guard<std::mutex> lock(mtx);  // RAII lock
        ++count;
        // Mutex automatically released when lock goes out of scope
    }

    int getCount() const {
        std::lock_guard<std::mutex> lock(mtx);
        return count;
    }
};
```

### 4.2 Smart Pointers: unique_ptr

`unique_ptr` represents exclusive ownership of a resource.

```cpp
#include <memory>
#include <vector>

class DatabaseConnection {
private:
    std::string connectionString;
    bool connected;

public:
    DatabaseConnection(const std::string& connStr) 
        : connectionString(connStr), connected(false) {
        std::cout << "Connecting to database: " << connStr << std::endl;
        connected = true;
    }

    ~DatabaseConnection() {
        if (connected) {
            std::cout << "Disconnecting from database" << std::endl;
            connected = false;
        }
    }

    void executeQuery(const std::string& query) {
        if (connected) {
            std::cout << "Executing: " << query << std::endl;
        }
    }
};

// Factory pattern with unique_ptr
class ConnectionFactory {
public:
    static std::unique_ptr<DatabaseConnection> createConnection(
        const std::string& connStr) {
        return std::make_unique<DatabaseConnection>(connStr);
    }
};

// Using unique_ptr for exclusive ownership
void uniquePtrDemo() {
    // Create unique_ptr
    std::unique_ptr<DatabaseConnection> conn1 = 
        ConnectionFactory::createConnection("localhost:5432");
    
    conn1->executeQuery("SELECT * FROM users");

    // Transfer ownership (move semantics)
    std::unique_ptr<DatabaseConnection> conn2 = std::move(conn1);
    
    // conn1 is now nullptr
    if (!conn1) {
        std::cout << "conn1 is nullptr after move" << std::endl;
    }

    conn2->executeQuery("INSERT INTO users VALUES (1, 'John')");

    // Automatic cleanup when conn2 goes out of scope
}

// Storing unique_ptr in containers
class ConnectionPool {
private:
    std::vector<std::unique_ptr<DatabaseConnection>> connections;

public:
    void addConnection(std::unique_ptr<DatabaseConnection> conn) {
        connections.push_back(std::move(conn));
    }

    DatabaseConnection* getConnection(size_t index) {
        if (index < connections.size()) {
            return connections[index].get();  // Get raw pointer
        }
        return nullptr;
    }

    size_t size() const {
        return connections.size();
    }
};
```

### 4.3 Smart Pointers: shared_ptr and weak_ptr

`shared_ptr` enables shared ownership with reference counting.

```cpp
#include <memory>
#include <iostream>

class Document {
private:
    std::string title;
    std::string content;

public:
    Document(const std::string& t, const std::string& c)
        : title(t), content(c) {
        std::cout << "Document created: " << title << std::endl;
    }

    ~Document() {
        std::cout << "Document destroyed: " << title << std::endl;
    }

    std::string getTitle() const { return title; }
    std::string getContent() const { return content; }
};

class User {
private:
    std::string name;
    std::shared_ptr<Document> currentDocument;

public:
    User(const std::string& n) : name(n) {}

    void openDocument(std::shared_ptr<Document> doc) {
        currentDocument = doc;
        std::cout << name << " opened document: " 
                  << doc->getTitle() << std::endl;
    }

    void closeDocument() {
        if (currentDocument) {
            std::cout << name << " closed document: " 
                      << currentDocument->getTitle() << std::endl;
            currentDocument.reset();
        }
    }

    std::shared_ptr<Document> getDocument() const {
        return currentDocument;
    }
};

void sharedPtrDemo() {
    // Create shared document
    auto doc = std::make_shared<Document>("Report.docx", "Annual report...");
    
    std::cout << "Reference count: " << doc.use_count() << std::endl;  // 1

    {
        User user1


# Complete Low-Level Design Learning Guide (45 Days)

## Phase 1: OOP Fundamentals (Days 1-7)

### Day 1-2: Interfaces Using Abstract Classes

**Concept**: Abstract classes provide partial implementation and define contracts for subclasses.

**Key Differences**:
- **Abstract Class**: Can have both abstract and concrete methods, fields, constructors
- **Interface**: Pure contracts (Java 8+ allows default methods)

**Example**:
```java
// Abstract class with partial implementation
abstract class PaymentProcessor {
    protected String merchantId;
    
    public PaymentProcessor(String merchantId) {
        this.merchantId = merchantId;
    }
    
    // Concrete method
    public void logTransaction(String transactionId) {
        System.out.println("Transaction " + transactionId + " logged");
    }
    
    // Abstract methods - must be implemented
    abstract boolean processPayment(double amount);
    abstract void refund(String transactionId);
}

class CreditCardProcessor extends PaymentProcessor {
    public CreditCardProcessor(String merchantId) {
        super(merchantId);
    }
    
    @Override
    boolean processPayment(double amount) {
        // Credit card specific logic
        return true;
    }
    
    @Override
    void refund(String transactionId) {
        // Refund logic
    }
}
```

**When to Use**:
- Abstract Class: When you have common implementation to share
- Interface: When defining pure contracts, supporting multiple inheritance

---

### Day 3-4: Mini Design - Logger System

**Requirements**:
1. Support multiple log levels (DEBUG, INFO, WARN, ERROR)
2. Multiple output destinations (Console, File, Database)
3. Configurable log formatting
4. Thread-safe operation

**Basic Implementation**:
```java
// Log level enum
enum LogLevel {
    DEBUG(1), INFO(2), WARN(3), ERROR(4);
    
    private int priority;
    
    LogLevel(int priority) {
        this.priority = priority;
    }
    
    public boolean shouldLog(LogLevel configuredLevel) {
        return this.priority >= configuredLevel.priority;
    }
}

// Abstract log destination
abstract class LogDestination {
    abstract void write(String message);
}

class ConsoleDestination extends LogDestination {
    @Override
    void write(String message) {
        System.out.println(message);
    }
}

class FileDestination extends LogDestination {
    private String fileName;
    
    public FileDestination(String fileName) {
        this.fileName = fileName;
    }
    
    @Override
    void write(String message) {
        // Write to file
    }
}

// Logger class (Singleton pattern)
class Logger {
    private static Logger instance;
    private LogLevel minLevel;
    private List<LogDestination> destinations;
    
    private Logger() {
        minLevel = LogLevel.INFO;
        destinations = new ArrayList<>();
    }
    
    public static synchronized Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
    
    public void addDestination(LogDestination destination) {
        destinations.add(destination);
    }
    
    public void setMinLevel(LogLevel level) {
        this.minLevel = level;
    }
    
    public void log(LogLevel level, String message) {
        if (!level.shouldLog(minLevel)) {
            return;
        }
        
        String formattedMessage = String.format("[%s] %s: %s",
            LocalDateTime.now(), level, message);
        
        for (LogDestination dest : destinations) {
            dest.write(formattedMessage);
        }
    }
    
    public void debug(String message) { log(LogLevel.DEBUG, message); }
    public void info(String message) { log(LogLevel.INFO, message); }
    public void warn(String message) { log(LogLevel.WARN, message); }
    public void error(String message) { log(LogLevel.ERROR, message); }
}
```

---

## Phase 2: SOLID Principles (Days 8-14)

### Day 8-9: Single Responsibility Principle (SRP)

**Principle**: A class should have only one reason to change.

**Bad Example**:
```java
class User {
    private String name;
    private String email;
    
    // Violates SRP - multiple responsibilities
    public void save() {
        // Database logic
    }
    
    public void sendEmail(String message) {
        // Email logic
    }
    
    public String generateReport() {
        // Report generation logic
        return "";
    }
}
```

**Good Example**:
```java
// Single responsibility: User data
class User {
    private String name;
    private String email;
    
    // Getters and setters only
}

// Single responsibility: Database operations
class UserRepository {
    public void save(User user) {
        // Database logic
    }
    
    public User findById(String id) {
        return null;
    }
}

// Single responsibility: Email operations
class EmailService {
    public void sendEmail(User user, String message) {
        // Email logic
    }
}

// Single responsibility: Report generation
class UserReportGenerator {
    public String generateReport(User user) {
        // Report logic
        return "";
    }
}
```

---

### Day 10-11: Open-Closed Principle (OCP)

**Principle**: Classes should be open for extension but closed for modification.

**Bad Example**:
```java
class PaymentProcessor {
    public void processPayment(String type, double amount) {
        if (type.equals("credit")) {
            // Credit card logic
        } else if (type.equals("debit")) {
            // Debit card logic
        } else if (type.equals("paypal")) {
            // PayPal logic
        }
        // Need to modify this method for each new payment type
    }
}
```

**Good Example**:
```java
interface Payment {
    boolean process(double amount);
}

class CreditCardPayment implements Payment {
    @Override
    public boolean process(double amount) {
        // Credit card logic
        return true;
    }
}

class PayPalPayment implements Payment {
    @Override
    public boolean process(double amount) {
        // PayPal logic
        return true;
    }
}

class PaymentProcessor {
    public boolean processPayment(Payment payment, double amount) {
        return payment.process(amount);
    }
}

// Adding new payment types doesn't require modifying existing code
class CryptoPayment implements Payment {
    @Override
    public boolean process(double amount) {
        // Crypto logic
        return true;
    }
}
```

---

### Day 12: Liskov Substitution Principle (LSP)

**Principle**: Objects of a superclass should be replaceable with objects of subclasses without breaking the application.

**Bad Example**:
```java
class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // Violates LSP
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height; // Violates LSP
    }
}

// This breaks
void testRectangle(Rectangle r) {
    r.setWidth(5);
    r.setHeight(4);
    assert r.getArea() == 20; // Fails for Square
}
```

**Good Example**:
```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public int getArea() {
        return width * height;
    }
}

class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    @Override
    public int getArea() {
        return side * side;
    }
}
```

---

### Day 13: Interface Segregation Principle (ISP)

**Principle**: Clients should not be forced to depend on interfaces they don't use.

**Bad Example**:
```java
interface Worker {
    void work();
    void eat();
    void sleep();
}

class HumanWorker implements Worker {
    public void work() { }
    public void eat() { }
    public void sleep() { }
}

class RobotWorker implements Worker {
    public void work() { }
    public void eat() { } // Robots don't eat!
    public void sleep() { } // Robots don't sleep!
}
```

**Good Example**:
```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

class HumanWorker implements Workable, Eatable, Sleepable {
    public void work() { }
    public void eat() { }
    public void sleep() { }
}

class RobotWorker implements Workable {
    public void work() { }
    // Only implements what it needs
}
```

---

### Day 14: Dependency Inversion Principle (DIP)

**Principle**: High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Bad Example**:
```java
class MySQLDatabase {
    public void save(String data) {
        // MySQL specific code
    }
}

class UserService {
    private MySQLDatabase database; // Tight coupling
    
    public UserService() {
        this.database = new MySQLDatabase();
    }
    
    public void saveUser(String user) {
        database.save(user);
    }
}
```

**Good Example**:
```java
interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    public void save(String data) {
        // MySQL specific code
    }
}

class MongoDatabase implements Database {
    public void save(String data) {
        // MongoDB specific code
    }
}

class UserService {
    private Database database; // Depends on abstraction
    
    public UserService(Database database) {
        this.database = database;
    }
    
    public void saveUser(String user) {
        database.save(user);
    }
}

// Usage
Database db = new MySQLDatabase();
UserService service = new UserService(db);
```

---

## Phase 3: Design Patterns (Days 15-25)

### Day 15-17: Creational Patterns

#### 1. Singleton Pattern
**Use Case**: Ensure only one instance exists (Logger, Configuration Manager)

```java
class DatabaseConnection {
    private static DatabaseConnection instance;
    private Connection connection;
    
    private DatabaseConnection() {
        // Initialize connection
    }
    
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
    
    // Thread-safe lazy initialization
    public static DatabaseConnection getInstanceDoubleChecked() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

#### 2. Factory Pattern
**Use Case**: Create objects without specifying exact class

```java
interface Vehicle {
    void drive();
}

class Car implements Vehicle {
    public void drive() {
        System.out.println("Driving a car");
    }
}

class Bike implements Vehicle {
    public void drive() {
        System.out.println("Riding a bike");
    }
}

class VehicleFactory {
    public static Vehicle createVehicle(String type) {
        switch (type.toLowerCase()) {
            case "car":
                return new Car();
            case "bike":
                return new Bike();
            default:
                throw new IllegalArgumentException("Unknown vehicle type");
        }
    }
}
```

#### 3. Builder Pattern
**Use Case**: Construct complex objects step by step

```java
class Pizza {
    private String size;
    private boolean cheese;
    private boolean pepperoni;
    private boolean mushrooms;
    
    private Pizza(Builder builder) {
        this.size = builder.size;
        this.cheese = builder.cheese;
        this.pepperoni = builder.pepperoni;
        this.mushrooms = builder.mushrooms;
    }
    
    static class Builder {
        private String size;
        private boolean cheese = false;
        private boolean pepperoni = false;
        private boolean mushrooms = false;
        
        public Builder(String size) {
            this.size = size;
        }
        
        public Builder cheese(boolean value) {
            cheese = value;
            return this;
        }
        
        public Builder pepperoni(boolean value) {
            pepperoni = value;
            return this;
        }
        
        public Builder mushrooms(boolean value) {
            mushrooms = value;
            return this;
        }
        
        public Pizza build() {
            return new Pizza(this);
        }
    }
}

// Usage
Pizza pizza = new Pizza.Builder("Large")
    .cheese(true)
    .pepperoni(true)
    .build();
```

#### 4. Prototype Pattern
**Use Case**: Clone objects instead of creating new ones

```java
abstract class Shape implements Cloneable {
    private String id;
    protected String type;
    
    abstract void draw();
    
    public Object clone() {
        Object clone = null;
        try {
            clone = super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

class Circle extends Shape {
    public Circle() {
        type = "Circle";
    }
    
    void draw() {
        System.out.println("Drawing Circle");
    }
}
```

---

### Day 18-20: Structural Patterns

#### 1. Adapter Pattern
**Use Case**: Make incompatible interfaces work together

```java
// Legacy interface
interface LegacyPrinter {
    void printOldWay(String text);
}

// Modern interface
interface ModernPrinter {
    void print(String text);
}

class OldPrinter implements LegacyPrinter {
    public void printOldWay(String text) {
        System.out.println("Old way: " + text);
    }
}

class PrinterAdapter implements ModernPrinter {
    private LegacyPrinter legacyPrinter;
    
    public PrinterAdapter(LegacyPrinter legacyPrinter) {
        this.legacyPrinter = legacyPrinter;
    }
    
    public void print(String text) {
        legacyPrinter.printOldWay(text);
    }
}
```

#### 2. Decorator Pattern
**Use Case**: Add functionality to objects dynamically

```java
interface Coffee {
    double getCost();
    String getDescription();
}

class SimpleCoffee implements Coffee {
    public double getCost() {
        return 2.0;
    }
    
    public String getDescription() {
        return "Simple Coffee";
    }
}

abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public double getCost() {
        return coffee.getCost() + 0.3;
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
}

// Usage
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription() + " costs $" + coffee.getCost());
```

#### 3. Facade Pattern
**Use Case**: Provide simplified interface to complex subsystem

```java
class CPU {
    public void freeze() { }
    public void jump(long position) { }
    public void execute() { }
}

class Memory {
    public void load(long position, byte[] data) { }
}

class HardDrive {
    public byte[] read(long lba, int size) {
        return new byte[size];
    }
}

class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}
```

#### 4. Proxy Pattern
**Use Case**: Control access to an object

```java
interface Image {
    void display();
}

class RealImage implements Image {
    private String fileName;
    
    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + fileName);
    }
    
    public void display() {
        System.out.println("Displaying " + fileName);
    }
}

class ProxyImage implements Image {
    private RealImage realImage;
    private String fileName;
    
    public ProxyImage(String fileName) {
        this.fileName = fileName;
    }
    
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}
```

---

### Day 21-25: Behavioral Patterns

#### 1. Observer Pattern
**Use Case**: Notify multiple objects about state changes

```java
interface Observer {
    void update(String message);
}

interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

class NewsChannel implements Observer {
    private String name;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}
```

#### 2. Strategy Pattern
**Use Case**: Select algorithm at runtime

```java
interface PaymentStrategy {
    void pay(int amount);
}

class CreditCardStrategy implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardStrategy(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

class PayPalStrategy implements PaymentStrategy {
    private String email;
    
    public PayPalStrategy(String email) {
        this.email = email;
    }
    
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}

class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}
```

#### 3. Command Pattern
**Use Case**: Encapsulate requests as objects

```java
interface Command {
    void execute();
    void undo();
}

class Light {
    public void on() {
        System.out.println("Light is on");
    }
    
    public void off() {
        System.out.println("Light is off");
    }
}

class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    public void execute() {
        light.on();
    }
    
    public void undo() {
        light.off();
    }
}

class RemoteControl {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
    }
    
    public void pressUndo() {
        command.undo();
    }
}
```

#### 4. State Pattern
**Use Case**: Change behavior based on state

```java
interface State {
    void doAction(Context context);
}

class StartState implements State {
    public void doAction(Context context) {
        System.out.println("Starting");
        context.setState(this);
    }
    
    public String toString() {
        return "Start State";
    }
}

class StopState implements State {
    public void doAction(Context context) {
        System.out.println("Stopping");
        context.setState(this);
    }
    
    public String toString() {
        return "Stop State";
    }
}

class Context {
    private State state;
    
    public void setState(State state) {
        this.state = state;
    }
    
    public State getState() {
        return state;
    }
}
```

---

## Phase 4: LLD Case Studies (Days 26-35)

### Day 26-27: Parking Lot System

**Requirements**:
1. Multiple floors with parking spots
2. Different vehicle types (Car, Bike, Truck)
3. Different spot types (Compact, Large, Handicapped)
4. Entry/Exit gates with ticket generation
5. Parking fee calculation

**Key Classes**:
```java
enum VehicleType {
    CAR, BIKE, TRUCK
}

enum SpotType {
    COMPACT, LARGE, HANDICAPPED, MOTORBIKE
}

class Vehicle {
    private String licensePlate;
    private VehicleType type;
    
    public Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }
    
    public VehicleType getType() {
        return type;
    }
}

class ParkingSpot {
    private String id;
    private SpotType type;
    private boolean isAvailable;
    private Vehicle currentVehicle;
    
    public ParkingSpot(String id, SpotType type) {
        this.id = id;
        this.type = type;
        this.isAvailable = true;
    }
    
    public boolean canFitVehicle(Vehicle vehicle) {
        // Logic to check if vehicle fits spot type
        return isAvailable;
    }
    
    public void parkVehicle(Vehicle vehicle) {
        this.currentVehicle = vehicle;
        this.isAvailable = false;
    }
    
    public void removeVehicle() {
        this.currentVehicle = null;
        this.isAvailable = true;
    }
}

class ParkingFloor {
    private int floorNumber;
    private List<ParkingSpot> spots;
    
    public ParkingFloor(int floorNumber) {
        this.floorNumber = floorNumber;
        this.spots = new ArrayList<>();
    }
    
    public ParkingSpot findAvailableSpot(Vehicle vehicle) {
        for (ParkingSpot spot : spots) {
            if (spot.canFitVehicle(vehicle)) {
                return spot;
            }
        }
        return null;
    }
}

class ParkingTicket {
    private String ticketId;
    private Vehicle vehicle;
    private ParkingSpot spot;
    private LocalDateTime entryTime;
    private LocalDateTime exitTime;
    
    public ParkingTicket(String ticketId, Vehicle vehicle, ParkingSpot spot) {
        this.ticketId = ticketId;
        this.vehicle = vehicle;
        this.spot = spot;
        this.entryTime = LocalDateTime.now();
    }
    
    public double calculateFee() {
        Duration duration = Duration.between(entryTime, exitTime);
        long hours = duration.toHours();
        return hours * 10; // $10 per hour
    }
}

class ParkingLot {
    private String name;
    private List<ParkingFloor> floors;
    private Map<String, ParkingTicket> activeTickets;
    
    private static ParkingLot instance;
    
    private ParkingLot(String name) {
        this.name = name;
        this.floors = new ArrayList<>();
        this.activeTickets = new HashMap<>();
    }
    
    public static synchronized ParkingLot getInstance(String name) {
        if (instance == null) {
            instance = new ParkingLot(name);
        }
        return instance;
    }
    
    public ParkingTicket parkVehicle(Vehicle vehicle) {
        for (ParkingFloor floor : floors) {
            ParkingSpot spot = floor.findAvailableSpot(vehicle);
            if (spot != null) {
                spot.parkVehicle(vehicle);
                String ticketId = UUID.randomUUID().toString();
                ParkingTicket ticket = new ParkingTicket(ticketId, vehicle, spot);
                activeTickets.put(ticketId, ticket);
                return ticket;
            }
        }
        return null; // No spot available
    }
    
    public double unparkVehicle(String ticketId) {
        ParkingTicket ticket = activeTickets.get(ticketId);
        if (ticket != null) {
            ticket.exitTime = LocalDateTime.now();
            double fee = ticket.calculateFee();
            ticket.spot.removeVehicle();
            activeTickets.remove(ticketId);
            return fee;
        }
        return 0;
    }
}
```

---

### Day 28-29: Elevator System

**Requirements**:
1. Multiple elevators in a building
2. Request handling (up/down buttons on floors)
3. Optimal elevator selection
4. Door control and safety mechanisms

**Key Classes**:
```java
enum Direction {
    UP, DOWN, IDLE
}

enum ElevatorState {
    MOVING, IDLE, MAINTENANCE
}

class Request {
    private int floor;
    private Direction direction;
    
    public Request(int floor, Direction direction) {
        this.floor = floor;
        this.direction = direction;
    }
    
    public int getFloor() { return floor; }
    public Direction getDirection() { return direction; }
}

class Elevator {
    private int id;
    private int currentFloor;
    private Direction direction;
    private ElevatorState state;
    private Set<Integer> destinationFloors;
    
    public Elevator(int id) {
        this.id = id;
        this.currentFloor = 0;
        this.direction = Direction.IDLE;
        this.state = ElevatorState.IDLE;
        this.destinationFloors = new TreeSet<>();
    }
    
    public void addDestination(int floor) {
        destinationFloors.add(floor);
        if (state == ElevatorState.IDLE) {
            state = ElevatorState.MOVING;
            determineDirection();
        }
    }
    
    private void determineDirection() {
        if (!destinationFloors.isEmpty()) {
            int nextFloor = destinationFloors.iterator().next();
            direction = nextFloor > currentFloor ? Direction.UP : Direction.DOWN;
        }
    }
    
    public void move() {
        if (destinationFloors.isEmpty()) {
            state = ElevatorState.IDLE;
            direction = Direction.IDLE;
            return;
        }
        
        if (direction == Direction.UP) {
            currentFloor++;
        } else if (direction == Direction.DOWN) {
            currentFloor--;
        }
        
        if (destinationFloors.contains(currentFloor)) {
            openDoors();
            destinationFloors.remove(currentFloor);
            closeDoors();
        }
        
        if (destinationFloors.isEmpty()) {
            state = ElevatorState.IDLE;
            direction = Direction.IDLE;
        } else {
            determineDirection();
        }
    }
    
    private void openDoors() {
        System.out.println("Elevator " + id + " opening doors at floor " + currentFloor);
    }
    
    private void closeDoors() {
        System.out.println("Elevator " + id + " closing doors");
    }
    
    public int getCurrentFloor() { return currentFloor; }
    public Direction getDirection() { return direction; }
    public ElevatorState getState() { return state; }
}

class ElevatorController {
    private List<Elevator> elevators;
    private Queue<Request> pendingRequests;
    
    public ElevatorController(int numElevators) {
        elevators = new ArrayList<>();
        for (int i = 0; i < numElevators; i++) {
            elevators.add(new Elevator(i));
        }
        pendingRequests = new LinkedList<>();
    }
    
    public void requestElevator(int floor, Direction direction) {
        Request request = new Request(floor, direction);
        Elevator bestElevator = findBestElevator(request);
        
        if (bestElevator != null) {
            bestElevator.addDestination(floor);
        } else {
            pendingRequests.offer(request);
        }
    }
    
    private Elevator findBestElevator(Request request) {
        Elevator bestElevator = null;
        int minDistance = Integer.MAX_VALUE;
        
        for (Elevator elevator : elevators) {
            if (elevator.getState() == ElevatorState.MAINTENANCE) {
                continue;
            }
            
            int distance = Math.abs(elevator.getCurrentFloor() - request.getFloor());
            
            // Prefer elevators moving in the same direction
            if (elevator.getDirection() == request.getDirection() ||
                elevator.getDirection() == Direction.IDLE) {
                if (distance < minDistance) {
                    minDistance = distance;
                    bestElevator = elevator;
                }
            }
        }
        
        return bestElevator;
    }
    
    public void step()
