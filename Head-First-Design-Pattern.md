# Head First Design Patterns

**Authors:** Eric Freeman, Elisabeth Freeman, with Kathy Sierra and Bert Bates  
**Publisher:** O'Reilly Media  
**Edition:** First Edition (2004)

---

## Table of Contents

1. [Chapter 1 — Welcome to Design Patterns: The Strategy Pattern](#chapter-1--welcome-to-design-patterns-the-strategy-pattern)
2. [Chapter 2 — Keeping your Objects in the Know: The Observer Pattern](#chapter-2--keeping-your-objects-in-the-know-the-observer-pattern)
3. [Chapter 3 — Decorating Objects: The Decorator Pattern](#chapter-3--decorating-objects-the-decorator-pattern)
4. [Chapter 4 — Baking with OO Goodness: The Factory Pattern](#chapter-4--baking-with-oo-goodness-the-factory-pattern)
5. [Chapter 5 — One of a Kind Objects: The Singleton Pattern](#chapter-5--one-of-a-kind-objects-the-singleton-pattern)
6. [Chapter 6 — Encapsulating Invocation: The Command Pattern](#chapter-6--encapsulating-invocation-the-command-pattern)
7. [Chapter 7 — Being Adaptive: The Adapter and Facade Patterns](#chapter-7--being-adaptive-the-adapter-and-facade-patterns)
8. [Chapter 8 — Encapsulating Algorithms: The Template Method Pattern](#chapter-8--encapsulating-algorithms-the-template-method-pattern)
9. [Chapter 9 — Well-Managed Collections: The Iterator and Composite Patterns](#chapter-9--well-managed-collections-the-iterator-and-composite-patterns)
10. [Chapter 10 — The State of Things: The State Pattern](#chapter-10--the-state-of-things-the-state-pattern)
11. [Chapter 11 — Controlling Object Access: The Proxy Pattern](#chapter-11--controlling-object-access-the-proxy-pattern)
12. [Chapter 12 — Patterns of Patterns: Compound Patterns (MVC)](#chapter-12--patterns-of-patterns-compound-patterns-mvc)
13. [Chapter 13 — Patterns in the Real World: Better Living with Patterns](#chapter-13--patterns-in-the-real-world-better-living-with-patterns)

---

## Preface: What Are Design Patterns?

Design patterns are **proven, reusable solutions to recurring design problems** in object-oriented software. Rather than providing code you can directly copy, they provide a *template* — a general approach you can adapt to your specific context. Patterns capture the collective experience of expert OO designers and encode that experience in a form that is shareable, communicable, and reusable.

The book uses the phrase **"experience reuse"** to contrast with code reuse: with patterns, you are reusing the design wisdom of developers who have faced the same class of problem before.

---

## Chapter 1 — Welcome to Design Patterns: The Strategy Pattern

### Summary

Chapter 1 introduces design patterns through a relatable problem: a duck simulation game (`SimUDuck`) that grows in complexity as new requirements are added. The journey illustrates the dangers of overusing inheritance and shows how the **Strategy Pattern** solves the problem by separating what varies from what stays the same, and then encapsulating the varying parts.

### Detailed Explanation

#### The SimUDuck Problem

The original system had a `Duck` superclass with `quack()`, `swim()`, and `display()` methods, all inherited by concrete duck types like `MallardDuck` and `RedheadDuck`.

When the requirement to *fly* was added, a naive developer added `fly()` directly to `Duck`. This caused unintended consequences — rubber ducks and decoy ducks, which shouldn't fly, suddenly inherited flying behavior. Fixing by overriding `fly()` in every non-flying subclass leads to **maintenance nightmares** as the class hierarchy grows.

Trying to use interfaces (`Flyable`, `Quackable`) seems better, but Java interfaces have no implementation — so every change to flying behavior must be duplicated in every class that implements it. This destroys code reuse.

#### The Design Principle: Separate What Varies

> **Design Principle:** Identify the aspects of your application that vary and separate them from what stays the same.

This is the cornerstone principle behind nearly every design pattern. By pulling out what varies (`fly` and `quack` behaviors), we protect the rest of the code from change.

#### The Design Principle: Program to an Interface, Not an Implementation

> **Design Principle:** Program to an interface, not an implementation.

Instead of coding duck behavior directly into duck classes (concrete implementation), the book introduces two interfaces — `FlyBehavior` and `QuackBehavior` — each with multiple implementations:

```java
// FlyBehavior interface
public interface FlyBehavior {
    public void fly();
}

// Implementations
public class FlyWithWings implements FlyBehavior {
    public void fly() {
        System.out.println("I'm flying!");
    }
}

public class FlyNoWay implements FlyBehavior {
    public void fly() {
        System.out.println("I can't fly!");
    }
}
```

The `Duck` class then holds a *reference* to these interfaces and delegates behavior to them:

```java
public abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    public void performFly() {
        flyBehavior.fly();
    }

    public void performQuack() {
        quackBehavior.quack();
    }

    public void setFlyBehavior(FlyBehavior fb) {
        flyBehavior = fb;
    }
}
```

#### The Design Principle: Favor Composition Over Inheritance

> **Design Principle:** Favor composition over inheritance.

By *composing* behavior objects into `Duck`, rather than inheriting behavior, we gain the ability to change behaviors at runtime:

```java
duck.setFlyBehavior(new FlyRocketPowered());
```

This is far more flexible than inheritance — behavior can be swapped without touching the `Duck` hierarchy.

### Key Concepts

- **Encapsulate what varies:** Pull out the parts of your code that change frequently and isolate them.
- **Program to an interface:** Use abstract types (interfaces or abstract classes) rather than concrete implementations.
- **Favor composition over inheritance:** "HAS-A" relationships (object holding a reference to a behavior) can be better than "IS-A" relationships (subclassing).
- **Delegation:** Duck *delegates* its flying and quacking behavior to the composed behavior objects.
- **Runtime flexibility:** Because behavior is assigned via composition, it can be swapped at runtime.

### Important Definitions

| Term | Definition |
|---|---|
| **Design Pattern** | A proven solution to a recurring design problem in a given context. |
| **Strategy Pattern** | Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from the clients that use it. |
| **Composition** | Building classes using references to other objects rather than inheriting from them. |
| **Interface (OO)** | An abstract type that defines a contract without implementation, allowing polymorphic use. |
| **HAS-A relationship** | A class that contains a reference to another class (composition). |
| **IS-A relationship** | Inheritance — one class extending another. |

### Example (from the book)

```java
public class MallardDuck extends Duck {
    public MallardDuck() {
        quackBehavior = new Quack();     // real quack
        flyBehavior = new FlyWithWings(); // flies normally
    }

    public void display() {
        System.out.println("I'm a real Mallard duck");
    }
}
```

A `RubberDuck` would instead use `FlyNoWay` and `Squeak`, without inheriting any unwanted behavior.

### Real-World Applications

- **Sorting algorithms:** A sorting utility can hold a `SortStrategy` (bubble sort, quicksort, merge sort) and switch strategies based on data size.
- **Payment processing:** An e-commerce app can hold a `PaymentStrategy` (credit card, PayPal, crypto) and swap it based on user selection at runtime.
- **Logging frameworks:** Different log destinations (file, console, remote server) can be swapped via a strategy without changing the core logging logic.

### Chapter Summary

The Strategy Pattern solves the brittleness of inheritance by encapsulating varying algorithms/behaviors into separate classes and composing them into the objects that need them. The three key OO design principles introduced — encapsulate what varies, program to an interface, and favor composition over inheritance — form the philosophical backbone of most design patterns throughout the book.

---

## Chapter 2 — Keeping your Objects in the Know: The Observer Pattern

### Summary

Chapter 2 introduces the **Observer Pattern** through a weather monitoring application. The problem is how to keep multiple display elements (current conditions, statistics, forecast) synchronized with a single source of weather data without tightly coupling them together. The Observer Pattern elegantly solves this with a publisher-subscriber model.

### Detailed Explanation

#### The Weather Station Problem

`WeatherData` reads temperature, humidity, and pressure from sensors. Multiple displays need to update whenever the data changes. A naive approach calls each display's `update()` method directly from `WeatherData` — tightly coupling the weather object to every display type and making it impossible to add new displays without modifying `WeatherData`.

#### The Observer Pattern Structure

The pattern uses two key roles:

- **Subject (Publisher):** Maintains a list of observers, notifies them of state changes.
- **Observer (Subscriber):** Registers with a subject and receives update notifications.

```java
public interface Subject {
    public void registerObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObservers();
}

public interface Observer {
    public void update(float temp, float humidity, float pressure);
}
```

`WeatherData` implements `Subject`. Each display implements `Observer` and `DisplayElement`:

```java
public class WeatherData implements Subject {
    private ArrayList observers;
    private float temperature, humidity, pressure;

    public void notifyObservers() {
        for (Observer o : observers) {
            o.update(temperature, humidity, pressure);
        }
    }

    public void measurementsChanged() {
        notifyObservers();
    }
}
```

#### Loose Coupling

> **Design Principle:** Strive for loosely coupled designs between objects that interact.

The Subject only knows that observers implement the `Observer` interface. It doesn't know anything about their concrete types. Observers can be added, removed, or replaced at any time without modifying the Subject.

#### Java's Built-in Observer Support

Java provides `java.util.Observable` (the Subject) and `java.util.Observer` (the Observer interface). Subjects must call `setChanged()` before `notifyObservers()` to trigger updates. However, `Observable` is a class (not an interface), which limits flexibility through single-inheritance restrictions.

### Key Concepts

- **One-to-many dependency:** One subject, many observers — changes propagate automatically.
- **Loose coupling:** Subject and observers are decoupled; they interact only through the `Observer` interface.
- **Push vs. Pull:** In *push* mode the subject sends data to observers; in *pull* mode observers retrieve data from the subject after being notified.
- **Dynamic subscription:** Observers can register and deregister at runtime.

### Important Definitions

| Term | Definition |
|---|---|
| **Observer Pattern** | Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically. |
| **Subject** | The object that holds state and notifies observers of changes. Also called Publisher. |
| **Observer** | An object that wants to be notified of state changes in the subject. Also called Subscriber. |
| **Loose Coupling** | A design where interacting objects have minimal knowledge of each other. |

### Example (from the book)

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature, humidity;
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    public void display() {
        System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
    }
}
```

### Real-World Applications

- **GUI event listeners:** Button clicks trigger all registered listeners (e.g., Java's `ActionListener`).
- **Event buses in frameworks:** React/Redux, Angular's EventEmitter, and Android's LiveData all use Observer-based patterns.
- **Stock tickers:** A stock price service notifies all subscribing dashboards and alert systems.
- **Social media feeds:** Followers (observers) are updated when someone they follow (subject) posts.

### Chapter Summary

The Observer Pattern creates a one-to-many dependency so that a single state-holding subject automatically propagates changes to all registered observers. It achieves loose coupling — the subject and its observers know very little about each other. This pattern is one of the most widely used in software development, appearing throughout GUIs, event systems, and reactive frameworks.

---

## Chapter 3 — Decorating Objects: The Decorator Pattern

### Summary

Chapter 3 uses Starbuzz Coffee as its narrative. The problem is representing beverages and condiments (milk, mocha, soy, whip) in a class hierarchy. A naive combinatorial approach leads to a class explosion. The **Decorator Pattern** provides an elegant alternative: wrapping objects with decorator objects that add behavior dynamically, without subclassing.

### Detailed Explanation

#### The Class Explosion Problem

If every combination of beverage + condiment were a separate class, you'd need: `HouseBlendWithMilk`, `HouseBlendWithMilkAndMocha`, `EspressoWithSoy`, and dozens more. This is unmanageable.

An alternative — adding boolean fields for each condiment in the base `Beverage` class — violates the Open-Closed Principle and creates a single bloated class with methods irrelevant to most beverages.

#### The Open-Closed Principle

> **Design Principle:** Classes should be open for extension but closed for modification.

This principle is the heart of the Decorator Pattern: you can extend behavior without modifying existing code.

#### How Decorators Work

Decorators share the same supertype as the objects they decorate. They *wrap* the object and add behavior either before or after delegating to the wrapped object.

```java
// Abstract component
public abstract class Beverage {
    String description = "Unknown Beverage";
    public String getDescription() { return description; }
    public abstract double cost();
}

// Abstract decorator
public abstract class CondimentDecorator extends Beverage {
    public abstract String getDescription();
}

// Concrete decorator
public class Mocha extends CondimentDecorator {
    Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }

    public double cost() {
        return .20 + beverage.cost();
    }
}
```

Usage at runtime:

```java
Beverage beverage = new Espresso();
beverage = new Mocha(beverage);       // wrap with Mocha
beverage = new Mocha(beverage);       // double Mocha
beverage = new Whip(beverage);        // wrap with Whip
System.out.println(beverage.getDescription() + " $" + beverage.cost());
// Espresso, Mocha, Mocha, Whip $1.49
```

#### Java I/O and the Decorator Pattern

Java's `java.io` package is a real-world application of this exact pattern:
- `InputStream` is the abstract component.
- `FilterInputStream` is the abstract decorator.
- `BufferedInputStream`, `DataInputStream`, `ZipInputStream` are concrete decorators.

### Key Concepts

- **Wrapping:** A decorator holds a reference to the component it decorates and adds behavior around it.
- **Same type:** Decorators and components share the same interface, so decorators can replace components transparently.
- **Dynamic composition:** Behavior is added at runtime by wrapping, not at compile time through inheritance.
- **Unlimited nesting:** Decorators can be stacked — a decorator can wrap another decorator.

### Important Definitions

| Term | Definition |
|---|---|
| **Decorator Pattern** | Attaches additional responsibilities to an object dynamically. Provides a flexible alternative to subclassing for extending functionality. |
| **Component** | The base object being wrapped, defined by an abstract class or interface. |
| **Concrete Component** | The actual object that is being decorated (e.g., `Espresso`). |
| **Decorator** | Wraps a component and adds behavior before or after delegating to the component. |
| **Open-Closed Principle** | Classes should be open for extension but closed for modification. |

### Real-World Applications

- **Java I/O streams:** `new BufferedReader(new FileReader("file.txt"))` layers buffering on top of file reading.
- **Logging middleware:** Each middleware layer in a web server (logging, authentication, caching) wraps the next handler.
- **UI components:** GUI libraries wrap widgets with border, scroll, or shadow decorators without changing the widget class.

### Chapter Summary

The Decorator Pattern lets you dynamically add behavior to objects by wrapping them in decorator objects that share the same interface. This avoids a class explosion caused by inheritance and respects the Open-Closed Principle. The trade-off is added complexity — many small classes — but the flexibility gained is often worth it. Java's own I/O library is a demonstration of this pattern at scale.

---

## Chapter 4 — Baking with OO Goodness: The Factory Pattern

### Summary

Chapter 4 addresses a common OO problem: using `new` to create concrete objects directly leads to tight coupling. The chapter explores three approaches — the Simple Factory (an idiom, not a true pattern), the **Factory Method Pattern**, and the **Abstract Factory Pattern** — each progressively more powerful for managing object creation.

### Detailed Explanation

#### The Problem with `new`

Every time you write `new ConcreteClass()`, you depend on that concrete type. If requirements change and you need a different class, you must modify calling code everywhere. This violates "program to interfaces, not implementations."

```java
// Tightly coupled — bad
Duck duck;
if (picnic) {
    duck = new MallardDuck();
} else if (hunting) {
    duck = new DecoyDuck();
}
```

#### Simple Factory (Not a True Pattern)

A Simple Factory moves object creation into a single dedicated class:

```java
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        if (type.equals("cheese")) pizza = new CheesePizza();
        else if (type.equals("pepperoni")) pizza = new PepperoniPizza();
        return pizza;
    }
}
```

This centralizes creation but is not a true pattern — it doesn't provide the flexibility of the full patterns below.

#### Factory Method Pattern

The Factory Method Pattern defines an abstract `createPizza()` method in the creator class, deferring the actual instantiation to subclasses:

```java
public abstract class PizzaStore {
    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);  // delegates to subclass
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }

    // Factory Method — subclasses decide what to create
    abstract Pizza createPizza(String type);
}

public class NYPizzaStore extends PizzaStore {
    Pizza createPizza(String type) {
        if (type.equals("cheese")) return new NYStyleCheesePizza();
        // etc.
    }
}
```

> **Factory Method Pattern:** Defines an interface for creating an object, but lets subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

#### The Dependency Inversion Principle

> **Design Principle:** Depend upon abstractions. Do not depend upon concrete classes.

High-level components (like `PizzaStore`) should not depend on low-level concrete components (like `NYStyleCheesePizza`). Both should depend on abstractions. The Factory Method achieves this by inserting an abstraction layer between creator and product.

#### Abstract Factory Pattern

When you need to create *families* of related objects (e.g., all NY-style ingredients vs. all Chicago-style ingredients), the Abstract Factory provides an interface for creating the entire family:

```java
public interface PizzaIngredientFactory {
    public Dough createDough();
    public Sauce createSauce();
    public Cheese createCheese();
    public Veggies[] createVeggies();
    public Pepperoni createPepperoni();
    public Clams createClam();
}

public class NYPizzaIngredientFactory implements PizzaIngredientFactory {
    public Dough createDough() { return new ThinCrustDough(); }
    public Sauce createSauce() { return new MarinaraSauce(); }
    // etc.
}
```

> **Abstract Factory Pattern:** Provides an interface for creating families of related or dependent objects without specifying their concrete classes.

### Key Concepts

- **Encapsulate object creation:** Move `new` calls out of application code into factories.
- **Factory Method vs. Abstract Factory:** Factory Method uses inheritance (subclasses override a factory method); Abstract Factory uses composition (an object holds a factory interface).
- **Dependency Inversion:** High-level modules should not depend on low-level modules; both should depend on abstractions.
- **Product families:** Abstract Factory creates groups of related objects consistently.

### Important Definitions

| Term | Definition |
|---|---|
| **Simple Factory** | A class with a static or instance method that creates objects. Not a formal pattern, but a common idiom. |
| **Factory Method Pattern** | Defines a method for creating objects in an abstract class; subclasses override it to instantiate the concrete type. |
| **Abstract Factory Pattern** | Provides an interface for creating families of related or dependent objects without specifying concrete classes. |
| **Creator** | The class that declares the factory method (abstract or with a default implementation). |
| **Product** | The object returned by the factory method. |
| **Dependency Inversion Principle** | Depend on abstractions, not concretions. High-level and low-level components should both depend on abstractions. |

### Real-World Applications

- **UI toolkits:** A `WidgetFactory` can return platform-specific buttons, scrollbars, and dialogs for Windows or macOS without the application knowing which platform it runs on.
- **Database drivers:** A connection factory returns MySQL, PostgreSQL, or SQLite connections — the application code only uses the abstract `Connection` interface.
- **Test mocking:** Test factories substitute mock objects for real dependencies during unit testing.

### Chapter Summary

Factory patterns encapsulate object creation to reduce coupling between creators and products. The Simple Factory is a useful idiom; the Factory Method Pattern uses inheritance to let subclasses decide what to create; the Abstract Factory Pattern uses composition to create families of related objects. All three help programs depend on abstractions rather than concrete types.

---

## Chapter 5 — One of a Kind Objects: The Singleton Pattern

### Summary

Chapter 5 covers the **Singleton Pattern** — ensuring that a class has at most one instance and providing a global access point to it. While the concept is simple, correct implementation (especially in multi-threaded environments) requires careful thought.

### Detailed Explanation

#### Why Singleton?

Some objects — like thread pools, logging systems, dialog boxes, device driver objects, registry settings — should have exactly one instance. Creating multiple instances could lead to:
- Incorrect program behavior (conflicting state)
- Resource overuse
- Inconsistent results

Relying on conventions or global variables doesn't guarantee uniqueness. The Singleton Pattern *mechanically* enforces it.

#### Basic Implementation

```java
public class Singleton {
    private static Singleton uniqueInstance;

    private Singleton() {}  // private constructor prevents external instantiation

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();  // lazy instantiation
        }
        return uniqueInstance;
    }
}
```

- The constructor is `private` — no one outside can call `new Singleton()`.
- `getInstance()` is the only way to get the instance.
- *Lazy instantiation*: the instance is created only when first needed.

#### Multithreading Problem

In a multi-threaded environment, two threads could simultaneously call `getInstance()`, both find `uniqueInstance == null`, and each create a separate instance — violating the Singleton guarantee.

#### Solutions for Thread Safety

**Option 1 — Synchronize `getInstance()`:**

```java
public static synchronized Singleton getInstance() {
    if (uniqueInstance == null) {
        uniqueInstance = new Singleton();
    }
    return uniqueInstance;
}
```

Simple and correct, but synchronization on every call adds overhead. Use when performance is not critical.

**Option 2 — Eagerly create the instance:**

```java
private static Singleton uniqueInstance = new Singleton();

public static Singleton getInstance() {
    return uniqueInstance;
}
```

The JVM creates the instance when the class is loaded — thread safe by default. Use when the instance is always needed and creation is not expensive.

**Option 3 — Double-checked locking:**

```java
private volatile static Singleton uniqueInstance;

public static Singleton getInstance() {
    if (uniqueInstance == null) {
        synchronized (Singleton.class) {
            if (uniqueInstance == null) {
                uniqueInstance = new Singleton();
            }
        }
    }
    return uniqueInstance;
}
```

Synchronizes only the first creation; `volatile` ensures the JVM does not reorder writes. Use when performance matters and lazy instantiation is required.

### Key Concepts

- **Private constructor:** Prevents any other class from instantiating the Singleton directly.
- **Static method `getInstance()`:** The global access point, creating the instance only once.
- **Lazy vs. eager instantiation:** Lazy creates on first use; eager creates at class load time.
- **Thread safety:** Naive implementation is not thread-safe; solutions include synchronization, eager initialization, or double-checked locking.

### Important Definitions

| Term | Definition |
|---|---|
| **Singleton Pattern** | Ensures a class has only one instance, and provides a global point of access to it. |
| **Lazy instantiation** | Delaying object creation until it is first needed. |
| **Eager instantiation** | Creating the object at class load time, before it is requested. |
| **Double-checked locking** | A technique that checks the instance twice (once without synchronization, once with) to minimize synchronization overhead. |
| **`volatile` keyword** | In Java, ensures that a variable's value is always read from and written to main memory, not a thread-local cache. |

### Real-World Applications

- **Logging systems:** A single `Logger` instance shared across all classes ensures log messages from all parts of the application go to one coordinated output.
- **Configuration managers:** One config object reads settings once and provides them globally.
- **Connection pools:** A single pool manager controls all database connections to prevent resource exhaustion.
- **Device drivers:** A single driver object coordinates access to hardware like a printer or GPU.

### Chapter Summary

The Singleton Pattern guarantees a single instance of a class with a global access point. The basic implementation is straightforward, but multi-threaded environments require careful handling. The three main approaches are: synchronizing `getInstance()`, eager initialization, or double-checked locking with `volatile`. Despite its simplicity on a class diagram (just one class!), getting Singleton right in production code takes attention to detail.

---

## Chapter 6 — Encapsulating Invocation: The Command Pattern

### Summary

Chapter 6 introduces the **Command Pattern** through the design of a programmable home automation remote control. The pattern encapsulates a request as an object, which allows requests to be parameterized, queued, logged, and undone. The chapter builds from a restaurant order metaphor to a full remote control implementation.

### Detailed Explanation

#### The Diner Metaphor

The chapter uses a diner to explain the pattern intuitively:
- **Customer** = Client (places the order)
- **Waitress** = Invoker (takes the order and calls `orderUp()`)
- **Order Slip** = Command object (encapsulates what is being requested)
- **Cook** = Receiver (actually performs the work)
- `orderUp()` = `execute()` method on the Command

The Waitress doesn't need to know what the Cook does — she just invokes `orderUp()` on the slip.

#### Core Command Pattern Structure

```java
// Command interface
public interface Command {
    public void execute();
}

// Concrete command
public class LightOnCommand implements Command {
    Light light;  // the receiver

    public LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.on();  // delegates to the receiver
    }
}

// Invoker
public class SimpleRemoteControl {
    Command slot;

    public void setCommand(Command command) {
        this.slot = command;
    }

    public void buttonWasPressed() {
        slot.execute();
    }
}
```

#### The Remote Control

The full remote has 7 on/off slots. Each slot holds a Command. The remote doesn't know anything about lights, fans, or stereos — it only calls `execute()` on whatever Command is stored.

#### Undo Support

Commands can implement an `undo()` method that reverses the effect of `execute()`. The invoker stores the last command and calls `undo()` when the undo button is pressed:

```java
public interface Command {
    public void execute();
    public void undo();
}

public class LightOnCommand implements Command {
    public void execute() { light.on(); }
    public void undo()    { light.off(); }
}
```

#### Macro Commands

A `MacroCommand` holds an array of Commands and calls `execute()` on each — implementing the ability to run several actions with a single button press.

#### Queuing and Logging

Since Command objects encapsulate requests, they can be:
- **Queued:** Stored and executed later (e.g., job queues, thread pools).
- **Logged:** Serialized to disk so crashed operations can be replayed (e.g., transactional systems).

### Key Concepts

- **Encapsulate invocation:** A command object packages a request (method call + receiver + arguments) into a single object.
- **Decoupling:** The invoker (remote) is decoupled from the receiver (light, fan) — it knows only the `Command` interface.
- **Undo/redo:** Storing commands enables reversible operations.
- **Parameterization:** You can pass different Commands to the same invoker, changing its behavior without changing its code.
- **Null Object Pattern:** A `NoCommand` object (does nothing) fills empty slots, avoiding null checks.

### Important Definitions

| Term | Definition |
|---|---|
| **Command Pattern** | Encapsulates a request as an object, letting you parameterize other objects with different requests, queue or log requests, and support undoable operations. |
| **Invoker** | The object that holds and calls a Command (e.g., remote control, button). |
| **Receiver** | The object that knows how to perform the action (e.g., Light, Stereo). |
| **Command** | An object encapsulating a request and its receiver, exposing only `execute()`. |
| **Macro Command** | A Command that contains and sequentially invokes multiple Commands. |
| **Null Object** | A Command that does nothing, used to avoid null checks. |

### Real-World Applications

- **GUI buttons and menu items:** Each menu item holds a Command object (`CopyCommand`, `PasteCommand`), decoupling menus from application logic.
- **Transaction systems:** Database transactions log commands that can be replayed after a crash.
- **Thread pools / job queues:** Workers pull Command objects from a queue and execute them without knowing what they do.
- **Macro recorders:** Applications record user actions as command sequences that can be replayed.

### Chapter Summary

The Command Pattern encapsulates method calls as objects, providing powerful decoupling between invokers and receivers. This makes it easy to support parameterized requests, undo/redo, logging, queuing, and macro operations. It is widely applied in GUI toolkits, transaction systems, and automation frameworks.

---

## Chapter 7 — Being Adaptive: The Adapter and Facade Patterns

### Summary

Chapter 7 covers two structural patterns. The **Adapter Pattern** converts one interface into another that a client expects — like an AC power adapter for travel. The **Facade Pattern** provides a simplified, unified interface to a complex subsystem — like a single "watch movie" button that coordinates a projector, amp, lights, and DVD player.

### Detailed Explanation

#### The Adapter Pattern

When you have existing code that uses a `Turkey` interface, but your client expects a `Duck`, you create an adapter:

```java
public class TurkeyAdapter implements Duck {
    Turkey turkey;

    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }

    public void quack() {
        turkey.gobble();  // adapts turkey's gobble to duck's quack
    }

    public void fly() {
        // turkeys fly short distances; call fly multiple times
        for (int i = 0; i < 5; i++) {
            turkey.fly();
        }
    }
}
```

The client uses `Duck`. The `TurkeyAdapter` wraps a `Turkey` and translates calls. The client never knows a turkey is involved.

> **Adapter Pattern:** Converts the interface of a class into another interface the clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

**Object Adapter vs. Class Adapter:**
- **Object Adapter** (shown above) uses composition — holds a reference to the adaptee. Works in both Java and C++.
- **Class Adapter** uses multiple inheritance to subclass both target and adaptee. Only possible in languages supporting multiple inheritance (not standard Java).

#### The Facade Pattern

A home theater system has many components: DVD player, projector, amplifier, tuner, screen, lights, popcorn maker. Watching a movie requires calling many methods in sequence. A Facade simplifies this:

```java
public class HomeTheaterFacade {
    Amplifier amp;
    Tuner tuner;
    DvdPlayer dvd;
    CdPlayer cd;
    Projector projector;
    TheaterLights lights;
    Screen screen;

    public void watchMovie(String movie) {
        System.out.println("Get ready to watch a movie...");
        popper.on();
        popper.pop();
        lights.dim(10);
        screen.down();
        projector.on();
        projector.wideScreenMode();
        amp.on();
        amp.setDvd(dvd);
        amp.setSurroundSound();
        amp.setVolume(5);
        dvd.on();
        dvd.play(movie);
    }

    public void endMovie() {
        // reverse all the above
    }
}
```

> **Facade Pattern:** Provides a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

#### Principle of Least Knowledge (Law of Demeter)

> **Design Principle:** Talk only to your immediate friends.

A method should only call methods on: the object itself, objects passed as parameters, objects it creates, and objects held in instance variables. This reduces the number of classes a component depends on, limiting ripple effects of changes.

### Key Concepts

- **Adapter:** Converts an existing incompatible interface into the one the client expects. Enables reuse of otherwise unusable classes.
- **Facade:** Provides a high-level, simplified interface to a complex subsystem. Does NOT restrict access to the subsystem — the components remain accessible.
- **Object vs. Class Adapter:** Object adapter uses composition (preferred in Java); class adapter uses multiple inheritance.
- **Principle of Least Knowledge:** Minimize the number of objects a class interacts with.

### Important Definitions

| Term | Definition |
|---|---|
| **Adapter Pattern** | Converts the interface of a class into another interface clients expect, enabling classes with incompatible interfaces to work together. |
| **Facade Pattern** | Provides a simplified, unified interface to a complex set of subsystem classes. |
| **Adaptee** | The class with the incompatible interface that the adapter wraps. |
| **Target** | The interface the client expects. |
| **Facade** | The simplified interface class that coordinates subsystem components. |
| **Principle of Least Knowledge** | A class should only communicate with closely related objects to reduce coupling. |

### Real-World Applications

- **Legacy system integration:** An Adapter wraps an old system's API to match a new application's expected interface.
- **Third-party library integration:** When a library uses a different interface than your code expects, an adapter bridges the gap.
- **Operating system APIs:** Facades wrap complex OS calls (file I/O, network) into simpler, higher-level APIs.
- **SLF4J logging:** The Simple Logging Facade for Java provides a uniform API over different logging backends (Log4j, Logback, JUL).

### Chapter Summary

The Adapter Pattern and Facade Pattern both wrap other classes, but for different reasons. Adapter converts one interface to another so incompatible classes can collaborate. Facade simplifies a complex subsystem behind a clean, easy-to-use interface. The Principle of Least Knowledge (Law of Demeter) underpins the Facade by encouraging minimal coupling between objects.

---

## Chapter 8 — Encapsulating Algorithms: The Template Method Pattern

### Summary

Chapter 8 compares making coffee and tea to reveal a shared algorithmic structure. The **Template Method Pattern** defines the skeleton of an algorithm in a base class, letting subclasses provide implementations for specific steps without changing the overall structure. The chapter also introduces the Hollywood Principle and *hooks*.

### Detailed Explanation

#### Coffee and Tea Comparison

Making coffee: boil water → brew coffee → pour into cup → add sugar and milk.  
Making tea: boil water → steep tea → pour into cup → add lemon.

The steps differ only in *what* is brewed and *what* is added. The skeleton is the same.

```java
public abstract class CaffeineBeverage {

    // The template method — defines the algorithm skeleton
    final void prepareRecipe() {
        boilWater();
        brew();              // abstract — subclass implements
        pourInCup();
        addCondiments();     // abstract — subclass implements
    }

    abstract void brew();
    abstract void addCondiments();

    void boilWater()  { System.out.println("Boiling water"); }
    void pourInCup()  { System.out.println("Pouring into cup"); }
}

public class Coffee extends CaffeineBeverage {
    void brew()          { System.out.println("Dripping Coffee through filter"); }
    void addCondiments() { System.out.println("Adding Sugar and Milk"); }
}

public class Tea extends CaffeineBeverage {
    void brew()          { System.out.println("Steeping the tea"); }
    void addCondiments() { System.out.println("Adding Lemon"); }
}
```

> **Template Method Pattern:** Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

#### Hooks

A *hook* is a method declared in the abstract class with a default (often empty) implementation. Subclasses may override it to hook into the algorithm at specific points:

```java
void customerWantsCondiments() {
    return true;  // subclasses can override to customize
}

final void prepareRecipe() {
    boilWater();
    brew();
    pourInCup();
    if (customerWantsCondiments()) {
        addCondiments();
    }
}
```

Hooks give subclasses the ability to *optionally* influence the algorithm's flow.

#### The Hollywood Principle

> **Design Principle:** Don't call us, we'll call you.

High-level components (the abstract class) call low-level components (subclasses), not the other way around. Subclasses never call the template method directly — they just implement the steps the high-level algorithm calls on them. This prevents "dependency rot" where low-level components trigger high-level components in complex chains.

#### Template Method in the JDK

- `Arrays.sort()` uses a template method — callers provide a `Comparator` that fills in the comparison step.
- Java's `AbstractList` uses template methods for `add()`, `get()`, `size()`.
- Applets used `init()`, `start()`, `paint()`, `stop()`, `destroy()` — a template method lifecycle.

### Key Concepts

- **Algorithm skeleton:** The template method defines the sequence of steps; subclasses fill in specifics.
- **`final` keyword:** The template method is declared `final` to prevent subclasses from changing the algorithm's structure.
- **Abstract methods:** Must be implemented by subclasses (mandatory steps).
- **Hooks:** Optional override points with default implementations.
- **Hollywood Principle:** High-level components call low-level ones, not the reverse.

### Important Definitions

| Term | Definition |
|---|---|
| **Template Method Pattern** | Defines the skeleton of an algorithm in a method, deferring some steps to subclasses, without changing the algorithm's structure. |
| **Template Method** | The method (usually `final`) that defines the algorithm's steps. |
| **Hook** | A method with a default implementation in the abstract class that subclasses may optionally override. |
| **Hollywood Principle** | High-level components decide when and how to call low-level components ("Don't call us, we'll call you"). |

### Real-World Applications

- **Frameworks:** Web frameworks define request-handling lifecycle methods (`beforeFilter`, `handle`, `afterFilter`); applications override specific steps.
- **Game engines:** A game loop template (`initialize → update → render → cleanup`) lets subclasses implement game-specific logic.
- **Data processing pipelines:** A template defines the pipeline (`read → parse → validate → store`); subclasses provide format-specific implementations.
- **JUnit test lifecycle:** `setUp()`, test method, `tearDown()` follow a template method structure.

### Chapter Summary

The Template Method Pattern is ideal when multiple classes share the same algorithmic structure but differ in specific steps. The base class defines and controls the algorithm's flow; subclasses customize individual steps. Hooks provide optional extension points. The Hollywood Principle prevents low-level subclasses from driving the application, keeping control at the higher-level template.

---

## Chapter 9 — Well-Managed Collections: The Iterator and Composite Patterns

### Summary

Chapter 9 tackles two related patterns. The **Iterator Pattern** provides a uniform way to traverse the elements of any collection without exposing its internal structure. The **Composite Pattern** allows individual objects and groups of objects to be treated uniformly, organizing them into tree structures representing part-whole hierarchies.

### Detailed Explanation

#### The Iterator Pattern

Two restaurants merge: one stores its menu in an `ArrayList`, the other in an `Array`. A `Waitress` wants to print all menu items, but can't — the data structures are different. The solution: each menu provides an `Iterator`.

```java
public interface Iterator {
    boolean hasNext();
    Object next();
}

// For the Diner's array-based menu
public class DinerMenuIterator implements Iterator {
    MenuItem[] items;
    int position = 0;

    public DinerMenuIterator(MenuItem[] items) {
        this.items = items;
    }

    public Object next()    { return items[position++]; }
    public boolean hasNext() { return position < items.length && items[position] != null; }
}
```

The `Waitress` now uses `Iterator` regardless of whether the underlying collection is an `Array`, `ArrayList`, or `Hashtable`:

```java
public void printMenu(Iterator iterator) {
    while (iterator.hasNext()) {
        MenuItem menuItem = (MenuItem) iterator.next();
        System.out.println(menuItem.getName());
    }
}
```

> **Iterator Pattern:** Provides a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

**Java's built-in Iterator:** `java.util.Iterator` provides `hasNext()`, `next()`, and `remove()`. Collections like `ArrayList` and `Hashtable` implement `java.lang.Iterable`, which returns a `java.util.Iterator`.

#### The Single Responsibility Principle

> **Design Principle:** A class should have only one reason to change.

The Iterator Pattern applies this: by separating the traversal logic (Iterator) from the collection (Menu), neither needs to change when the other does.

#### The Composite Pattern

After iterators are in place, a new requirement emerges: nested menus (a dessert submenu inside the dinner menu). A flat list of `MenuItem` objects can't represent hierarchical structure.

The Composite Pattern uses a common interface (`MenuComponent`) for both leaves (`MenuItem`) and composites (`Menu`):

```java
public abstract class MenuComponent {
    // Operations for composites
    public void add(MenuComponent menuComponent)    { throw new UnsupportedOperationException(); }
    public void remove(MenuComponent menuComponent) { throw new UnsupportedOperationException(); }
    public MenuComponent getChild(int i)            { throw new UnsupportedOperationException(); }

    // Operations for leaves
    public String getName()        { throw new UnsupportedOperationException(); }
    public String getDescription() { throw new UnsupportedOperationException(); }
    public double getPrice()       { throw new UnsupportedOperationException(); }
    public boolean isVegetarian()  { throw new UnsupportedOperationException(); }
    public void print()            { throw new UnsupportedOperationException(); }
}
```

`MenuItem` (leaf) and `Menu` (composite holding a list of `MenuComponent`) both extend `MenuComponent`. Calling `print()` on a `Menu` recursively prints all its items, including nested menus.

> **Composite Pattern:** Allows you to compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.

**Trade-off:** The common interface means some operations are not meaningful on all types (a `MenuItem` can't `add()` children). Methods throw `UnsupportedOperationException` by default. This trades some type safety for transparency.

### Key Concepts

- **Uniform traversal:** Iterator decouples traversal from collection representation.
- **Single Responsibility:** Splitting traversal logic from collection management makes each simpler.
- **Tree structures:** Composite organizes objects into hierarchical, recursive structures.
- **Uniform treatment:** Clients treat leaves and composites identically via the common interface.
- **Recursive composition:** Operations on a composite automatically propagate down to all children.

### Important Definitions

| Term | Definition |
|---|---|
| **Iterator Pattern** | Provides a way to access elements of an aggregate object sequentially without exposing its internal representation. |
| **Iterator** | An object that knows how to traverse a specific collection type. |
| **Aggregate** | The collection that creates an Iterator. |
| **Composite Pattern** | Composes objects into tree structures to represent part-whole hierarchies, allowing clients to treat leaves and composites uniformly. |
| **Leaf** | A node with no children in a composite tree (e.g., `MenuItem`). |
| **Composite (node)** | A node that can hold children (e.g., `Menu`). |
| **Single Responsibility Principle** | A class should have only one reason to change. |

### Real-World Applications

- **File systems:** Directories (composites) contain files (leaves) and other directories, all treated as `FileSystemItem`.
- **UI component trees:** A `Window` contains `Panel`s which contain `Button`s — all implement `Component` so painting, resizing, and event handling can be applied uniformly.
- **XML/HTML DOM:** Elements can contain other elements (composites) or text nodes (leaves), traversed with iterators.
- **Organization charts:** Departments (composites) contain employees (leaves) or sub-departments.

### Chapter Summary

The Iterator Pattern provides a clean, uniform way to traverse any collection without knowing its internal structure, and applies the Single Responsibility Principle by separating traversal from the collection. The Composite Pattern organizes objects into tree structures where leaves and composites are treated identically, making recursive operations natural. Together they handle the complexity of heterogeneous, hierarchical data.

---

## Chapter 10 — The State of Things: The State Pattern

### Summary

Chapter 10 uses a Gumball Machine to illustrate the **State Pattern**. The problem is managing behavior that changes depending on an object's internal state. A naive approach with conditionals becomes unmaintainable as states multiply. The State Pattern encapsulates each state's behavior in a separate class, making state transitions explicit and the system easy to extend.

### Detailed Explanation

#### The Gumball Machine Problem

A gumball machine has states: `NO_QUARTER`, `HAS_QUARTER`, `SOLD`, `SOLD_OUT`. Actions include `insertQuarter()`, `ejectQuarter()`, `turnCrank()`, `dispense()`. 

A naive implementation uses `if/else` or `switch` in every method, checking the current state. Adding a "10% chance of a free gumball" Winner state requires modifying every method — fragile and error-prone.

#### State Pattern Implementation

Each state becomes its own class implementing a `State` interface:

```java
public interface State {
    public void insertQuarter();
    public void ejectQuarter();
    public void turnCrank();
    public void dispense();
}

public class NoQuarterState implements State {
    GumballMachine gumballMachine;

    public NoQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    public void insertQuarter() {
        System.out.println("You inserted a quarter");
        gumballMachine.setState(gumballMachine.getHasQuarterState());
    }

    public void ejectQuarter() {
        System.out.println("You haven't inserted a quarter");
    }

    public void turnCrank()  { System.out.println("You turned, but there's no quarter"); }
    public void dispense()   { System.out.println("You need to pay first"); }
}
```

The `GumballMachine` (the *Context*) holds a reference to the current state and delegates all action calls to it:

```java
public class GumballMachine {
    State soldOutState, noQuarterState, hasQuarterState, soldState;
    State state;
    int count = 0;

    public GumballMachine(int numberGumballs) {
        // initialize state objects
        state = soldOutState;
        count = numberGumballs;
        if (numberGumballs > 0) state = noQuarterState;
    }

    public void insertQuarter() { state.insertQuarter(); }
    public void ejectQuarter()  { state.ejectQuarter();  }
    public void turnCrank()     { state.turnCrank(); state.dispense(); }

    void setState(State state)  { this.state = state; }
}
```

> **State Pattern:** Allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

#### State vs. Strategy Pattern

State and Strategy look structurally similar, but differ in intent:
- **Strategy:** The client chooses and sets an algorithm explicitly. Algorithms are interchangeable and unaware of each other.
- **State:** States know about each other (transitions) and drive themselves. The client rarely changes states directly.

### Key Concepts

- **Encapsulate state:** Each state's behaviors are encapsulated in a single class, not scattered in conditionals.
- **Context:** The object (`GumballMachine`) that delegates to the current state.
- **State transition:** State objects change the context's current state by calling `context.setState()`.
- **Open for extension:** Adding a new state (e.g., `WinnerState`) requires a new class and minimal changes to existing states — no touching every `if/else`.

### Important Definitions

| Term | Definition |
|---|---|
| **State Pattern** | Allows an object to alter its behavior when its internal state changes; the object appears to change its class. |
| **Context** | The class that holds a reference to the current state and delegates behavior to it (e.g., `GumballMachine`). |
| **State** | An interface or abstract class defining the behavior for a particular state. |
| **Concrete State** | A class that implements all behaviors relevant to one specific state of the context. |

### Real-World Applications

- **Vending machines:** Idle, Selection, Dispensing, and Out-of-Stock states each handle button presses differently.
- **Traffic light controllers:** Each light phase (Red, Green, Yellow) is a state with its own timing and transition logic.
- **Media players:** States like Playing, Paused, Stopped, Buffering, each handle play/pause/stop commands differently.
- **Order management:** Draft, Submitted, Approved, Shipped, Delivered states with distinct allowed transitions and behaviors.

### Chapter Summary

The State Pattern eliminates sprawling conditionals by giving each state its own class. The context object delegates all behavior to the current state object, which handles the behavior and drives transitions to the next state. This makes it easy to add new states without changing existing ones, following the Open-Closed Principle. Though it looks like Strategy, State is about letting an object control its own behavior as its internal state changes.

---

## Chapter 11 — Controlling Object Access: The Proxy Pattern

### Summary

Chapter 11 introduces the **Proxy Pattern** — providing a surrogate or placeholder that controls access to another object. The chapter explores Remote Proxies (Java RMI) and Virtual Proxies (lazy loading), then surveys other proxy variants including Protection Proxies, Firewall Proxies, and Smart Reference Proxies.

### Detailed Explanation

#### The Gumball Machine Monitoring Scenario

The CEO of Mighty Gumball wants to monitor all gumball machines remotely — see inventory and state without visiting each machine. The solution is a Remote Proxy that acts as a local representative for a remote object.

#### Remote Proxy with Java RMI

Java RMI (Remote Method Invocation) makes a remote object look local:
- The **stub** (proxy) lives on the client, forwarding calls over the network.
- The **skeleton** lives on the server, receiving calls and invoking the real object.

```java
// Remote interface
public interface GumballMachineRemote extends Remote {
    public int getCount() throws RemoteException;
    public String getLocation() throws RemoteException;
    public State getState() throws RemoteException;
}

// Server side — real object
public class GumballMachine extends UnicastRemoteObject implements GumballMachineRemote {
    // implementation
}

// Client uses the stub as if it were local
GumballMachineRemote machine = (GumballMachineRemote) Naming.lookup("rmi://location/gumballmachine");
System.out.println(machine.getCount());  // transparently calls remote object
```

> **Proxy Pattern:** Provides a surrogate or placeholder for another object to control access to it.

#### Virtual Proxy

A Virtual Proxy delays creation of an expensive object until it is truly needed. For example, loading a large image: the proxy shows a placeholder while the image loads in a background thread, then hands off to the real image once loaded.

```java
public class ImageProxy implements Icon {
    ImageIcon realImage;
    URL imageURL;

    public void paintIcon(Component c, Graphics g, int x, int y) {
        if (realImage != null) {
            realImage.paintIcon(c, g, x, y);
        } else {
            g.drawString("Loading album cover, please wait...", x+300, y+190);
            // load image in background thread
        }
    }
}
```

#### Protection Proxy (Java Dynamic Proxies)

Java's `java.lang.reflect.Proxy` can create a proxy dynamically at runtime for any interface. A Protection Proxy uses this to control access to methods based on the caller's role — e.g., preventing a person from calling methods on their own dating profile that only others should call.

#### Proxy Variants

| Proxy Type | Purpose |
|---|---|
| **Remote Proxy** | Represents an object in a different address space (another machine or JVM). |
| **Virtual Proxy** | Delays creation of an expensive object until needed. |
| **Protection Proxy** | Controls access to the real object based on access rights. |
| **Firewall Proxy** | Controls access to network resources, protecting local clients. |
| **Smart Reference Proxy** | Provides additional actions (reference counting, locking) when the object is accessed. |
| **Caching Proxy** | Caches results of expensive operations, returning cached results to multiple clients. |
| **Synchronization Proxy** | Provides safe access to a real object from multiple threads. |

### Key Concepts

- **Surrogate:** The proxy stands in for the real object and controls access to it.
- **Same interface:** Proxy and real object implement the same interface, so clients are unaware of the proxy.
- **Transparency:** Clients code against the interface; whether they talk to the proxy or real object is invisible.
- **Decorator vs. Proxy:** Decorators add behavior; proxies control access. Both wrap objects and use the same interface.
- **Java Dynamic Proxy:** The JVM creates a proxy class at runtime using `Proxy.newProxyInstance()` and an `InvocationHandler`.

### Important Definitions

| Term | Definition |
|---|---|
| **Proxy Pattern** | Provides a surrogate or placeholder for another object to control access to it. |
| **Remote Proxy** | A local representative for a remote object; handles network communication transparently. |
| **Virtual Proxy** | Delays expensive object creation until the object is actually needed. |
| **Protection Proxy** | Controls access to the real object based on permissions or roles. |
| **Dynamic Proxy** | A proxy created at runtime by the JVM for a given interface, using `java.lang.reflect.Proxy`. |
| **InvocationHandler** | The object that receives all method calls on a dynamic proxy and decides what to do with them. |

### Real-World Applications

- **Web service clients:** SOAP/REST client stubs act as remote proxies, hiding network communication.
- **Hibernate/JPA lazy loading:** Entity relationships are virtual proxies — related objects are loaded only when accessed.
- **Spring AOP:** Method interception for logging, security, and transactions uses dynamic proxy-based aspect-oriented programming.
- **CDN / caching layers:** A caching proxy stores expensive content and serves it without hitting the origin server.

### Chapter Summary

The Proxy Pattern provides a surrogate that controls access to another object. Remote proxies handle network distribution; virtual proxies delay expensive creation; protection proxies enforce access control. All proxy types share the same interface as the real object, making them transparent to clients. Java's dynamic proxy facility allows creating proxies at runtime without writing proxy classes manually.

---

## Chapter 12 — Patterns of Patterns: Compound Patterns (MVC)

### Summary

Chapter 12 explores *compound patterns* — solutions that combine multiple design patterns to solve a recurring, larger design problem. After demonstrating how patterns naturally emerge and cooperate through the duck simulator revisit, the chapter focuses on the quintessential compound pattern: **Model-View-Controller (MVC)**.

### Detailed Explanation

#### What is a Compound Pattern?

> A compound pattern combines two or more patterns into a solution that solves a recurring or general problem.

Simply using multiple patterns in a design does *not* make it a compound pattern. The combination must be reusable across many problems and constitute an established solution.

#### Duck Simulator — Patterns Working Together

The book rebuilds the SimUDuck simulator from scratch using multiple patterns cooperating:

1. **Interface + Adapter:** `Goose` doesn't implement `Quackable`, so a `GooseAdapter` wraps it.
2. **Decorator:** `QuackCounter` decorates any `Quackable` and counts how many times `quack()` is called.
3. **Abstract Factory:** `AbstractDuckFactory` and `CountingDuckFactory` create ducks with or without counting decorators.
4. **Composite:** `Flock` is a composite of `Quackable` objects; calling `quack()` on a flock calls it on all members.
5. **Observer:** `Quackologist` observes ducks through an `Observable` interface, notified whenever a duck quacks.

This demonstrates that patterns are not isolated tools — they naturally complement each other.

#### Model-View-Controller (MVC)

MVC is the most celebrated compound pattern in software. It separates an application into three roles:

- **Model:** Contains all the state, data, and application logic. It is unaware of views and controllers. Notifies observers (views) when state changes.
- **View:** The visual representation of the model. Renders the current state and hands user interactions to the controller.
- **Controller:** Acts as the intermediary. Receives user input from the view, interprets it, and calls appropriate methods on the model.

```
User → View → Controller → Model → (notifies) → View
```

**MVC through the lens of Design Patterns:**

| MVC Component | Pattern(s) Used |
|---|---|
| Model notifies views | **Observer Pattern** — view registers as observer of model |
| View displays state | **Composite Pattern** — GUI components nested in a tree |
| Controller translates input | **Strategy Pattern** — controller is a strategy plugged into the view |

**The MP3 Player example:**
- **Model:** Manages the song list, playback state, and metadata.
- **View:** Displays current track, time, and controls.
- **Controller:** Handles "play", "pause", "skip" button presses, updates model accordingly.

```java
// Model (simplified)
public class BeatModel implements BeatModelInterface {
    List observers = new ArrayList();
    // state: bpm, beat count

    public void registerObserver(BeatObserver o) { observers.add(o); }
    public void notifyObservers() {
        for (BeatObserver o : observers) o.updateBeat();
    }
    public void setBPM(int bpm) {
        this.bpm = bpm;
        notifyObservers();
    }
}

// Controller (simplified)
public class BeatController implements ControllerInterface {
    BeatModelInterface model;
    DJView view;

    public void increaseBPM() { model.setBPM(model.getBPM() + 1); }
    public void decreaseBPM() { model.setBPM(model.getBPM() - 1); }
}
```

#### Why MVC is Powerful

- **Separation of concerns:** Model, View, and Controller each have a single, well-defined responsibility.
- **Multiple views:** The same model can support many views simultaneously (e.g., graphical + text interface).
- **Pluggable controllers:** Different controllers can be swapped in to change how the view interacts with the model (Strategy).
- **Model independence:** The model knows nothing about the UI; it can be tested independently.

### Key Concepts

- **Compound pattern:** Two or more patterns combined to form a reusable solution to a general problem.
- **MVC roles:** Model (data + logic), View (presentation), Controller (input + coordination).
- **Observer in MVC:** Model notifies views of changes.
- **Strategy in MVC:** Controller is a strategy for the view; different controllers change behavior without changing the view.
- **Composite in MVC:** Views are typically composed of nested GUI components.

### Important Definitions

| Term | Definition |
|---|---|
| **Compound Pattern** | A combination of two or more design patterns that together solve a recurring, general problem. |
| **MVC (Model-View-Controller)** | An architectural compound pattern separating data/logic (Model), presentation (View), and input handling (Controller). |
| **Model** | Holds application state and logic; notifies observers when state changes. |
| **View** | Renders the model's state; forwards user input to the controller. |
| **Controller** | Interprets user input, manipulates the model accordingly. |

### Real-World Applications

- **Web frameworks:** Spring MVC, Ruby on Rails, Django, and ASP.NET all implement MVC.
- **Mobile development:** iOS (UIKit) and Android use MVC/MVP/MVVM variants derived from MVC.
- **Desktop GUIs:** Swing, JavaFX, and WPF use MVC-inspired architecture.
- **Game development:** Game state (model), rendering engine (view), and input handler (controller) map to MVC.

### Chapter Summary

Compound patterns combine multiple patterns into cohesive, reusable solutions. MVC is the most significant compound pattern — combining Observer, Strategy, and Composite — to separate model, view, and controller responsibilities. Understanding MVC *through* its constituent patterns reveals why it is so flexible: observers decouple view from model; strategy makes controllers pluggable; composite unifies GUI component hierarchies. This chapter is the capstone of the book's pattern coverage, showing how individual patterns combine into larger architectural solutions.

---

## Chapter 13 — Patterns in the Real World: Better Living with Patterns

### Summary

Chapter 13 is a practical guide to living with design patterns beyond the classroom. It covers: the formal definition of a design pattern, how patterns are classified, how to think in patterns (when to use them, when not to), how patterns relate to each other, and the broader pattern community.

### Detailed Explanation

#### What is a Design Pattern? (Formal Definition)

> A **pattern** is a solution to a problem in a context.

Three elements:
- **Context:** The situation in which the pattern applies.
- **Problem:** The goal you want to achieve (including constraints and forces at play).
- **Solution:** A general design that resolves the problem in the context.

Patterns capture design knowledge in a form that can be communicated and applied by others.

#### The Pattern Catalog and the Gang of Four

The canonical reference for design patterns is *Design Patterns: Elements of Reusable Object-Oriented Software* by Gamma, Helm, Johnson, and Vlissides — known as the **Gang of Four (GoF)**. They documented 23 classic patterns classified into three categories:

| Category | Description | Examples |
|---|---|---|
| **Creational** | Concern the process of object creation | Singleton, Factory Method, Abstract Factory |
| **Structural** | Deal with the composition of classes and objects | Adapter, Decorator, Facade, Proxy, Composite |
| **Behavioral** | Characterize the way classes and objects interact | Observer, Strategy, Command, Iterator, State, Template Method |

#### Thinking in Patterns: When to Use Them

The book offers practical guidance:

1. **Keep it simple (KISS):** Always start with the simplest design. Do not reach for patterns to prove sophistication.
2. **Use a pattern when you see a need:** Introduce patterns when a recurring problem clearly matches a pattern's intent — not speculatively.
3. **Let patterns emerge:** Begin designing from principles. Patterns emerge naturally as designs evolve; don't impose them from the start.
4. **Refactoring time is pattern time:** When revisiting code, look for spots where a pattern could improve structure without changing behavior.
5. **Know when to remove a pattern:** If complexity has grown and flexibility isn't being used, a simpler solution may serve better.

#### Pattern Anti-Patterns

The book warns against **pattern abuse**:
- Applying patterns to problems they aren't suited for, just to use them.
- Over-engineering simple systems with unnecessary layers of abstraction.
- Treating patterns as a silver bullet rather than as one tool among many.

#### How Patterns Relate

Patterns don't exist in isolation. Common relationships:
- **Decorator** and **Composite** both use recursive composition, but for different purposes.
- **Factory Method** is often used inside **Abstract Factory**.
- **State** and **Strategy** are structurally similar but differ in intent.
- **Proxy** and **Decorator** both wrap objects; proxy controls access, decorator adds behavior.
- **Observer** is the basis of MVC's model-view communication.

#### The Pattern Community

Design patterns are a living discipline. Beyond the GoF book:
- The *Portland Pattern Repository* (wiki) catalogs patterns.
- Pattern languages (like Christopher Alexander's original architectural patterns) extend the concept beyond software.
- AntiPatterns document commonly recurring bad solutions.
- Domain-specific catalogs exist for enterprise, concurrent, and distributed systems.

### Key Concepts

- **Pattern definition:** A solution to a problem in a context.
- **GoF categories:** Creational, Structural, Behavioral.
- **KISS principle:** Simplicity first — use patterns only when needed.
- **Emergent patterns:** Let patterns arise from your design naturally, driven by principles.
- **Pattern relationships:** Understanding how patterns relate helps combine them effectively.
- **Anti-patterns:** Overusing patterns or applying them to the wrong problems.

### Important Definitions

| Term | Definition |
|---|---|
| **Gang of Four (GoF)** | Gamma, Helm, Johnson, and Vlissides — authors of the foundational Design Patterns book documenting 23 classic patterns. |
| **Creational Patterns** | Patterns that deal with object creation mechanisms. |
| **Structural Patterns** | Patterns that describe how classes and objects are composed to form larger structures. |
| **Behavioral Patterns** | Patterns that characterize how classes and objects communicate and distribute responsibility. |
| **Pattern Language** | A structured collection of interrelated patterns that together address a design domain. |
| **Anti-Pattern** | A commonly occurring solution that appears correct but causes more problems than it solves. |
| **Refactoring** | Improving the internal structure of code without changing its external behavior. |

### Real-World Applications

- **Framework design:** Spring, Rails, and Angular are built on layers of design patterns — recognizing them helps you use and extend frameworks effectively.
- **Code reviews:** Pattern vocabulary makes design discussions precise and efficient among team members.
- **Architecture documentation:** Pattern names serve as a shared language for describing system structure.
- **Mentoring:** Patterns are a proven way to transmit OO design wisdom from experienced to junior developers.

### Chapter Summary

Chapter 13 zooms out from individual patterns to the bigger picture. It defines what a pattern is formally, explains how the GoF organized patterns into creational, structural, and behavioral categories, and provides practical wisdom about when to introduce patterns, when to hold back, and when to remove them. The central message: patterns are a powerful tool, but tools that should be used thoughtfully — *let them emerge from your design, don't impose them upon it.* Simplicity guided by OO principles is always the goal; patterns are a means to that end, not the end itself.

---

## OO Design Principles Summary

The following design principles are woven throughout the book and underpin all the patterns:

| Principle | Description |
|---|---|
| **Encapsulate what varies** | Identify the parts of your code that change and separate them from what stays the same. |
| **Program to an interface, not an implementation** | Depend on abstract types, not concrete classes. |
| **Favor composition over inheritance** | Use "HAS-A" (composition) relationships more than "IS-A" (inheritance) relationships. |
| **Strive for loosely coupled designs** | Objects that interact should know as little as possible about each other. |
| **Open-Closed Principle** | Classes should be open for extension but closed for modification. |
| **Dependency Inversion Principle** | Depend on abstractions; high-level and low-level modules should both depend on abstractions. |
| **Principle of Least Knowledge** | Talk only to your immediate friends; minimize the chain of method calls to distant objects. |
| **Hollywood Principle** | Don't call us, we'll call you — high-level components call low-level components, not the reverse. |
| **Single Responsibility Principle** | A class should have only one reason to change. |

---

## Quick Reference: All Patterns Covered

| Pattern | Category | One-Line Definition |
|---|---|---|
| **Strategy** | Behavioral | Defines a family of algorithms, encapsulates each, and makes them interchangeable. |
| **Observer** | Behavioral | Defines one-to-many dependency; when one object changes state, all dependents are notified automatically. |
| **Decorator** | Structural | Attaches additional responsibilities to an object dynamically; flexible alternative to subclassing. |
| **Factory Method** | Creational | Defines an interface for creating an object, but lets subclasses decide which class to instantiate. |
| **Abstract Factory** | Creational | Provides an interface for creating families of related or dependent objects without specifying concrete classes. |
| **Singleton** | Creational | Ensures a class has only one instance and provides a global point of access to it. |
| **Command** | Behavioral | Encapsulates a request as an object, allowing parameterization, queuing, logging, and undo. |
| **Adapter** | Structural | Converts the interface of a class into another interface clients expect. |
| **Facade** | Structural | Provides a unified interface to a set of interfaces in a subsystem. |
| **Template Method** | Behavioral | Defines the skeleton of an algorithm, deferring some steps to subclasses. |
| **Iterator** | Behavioral | Provides a way to access elements of an aggregate object sequentially without exposing its representation. |
| **Composite** | Structural | Composes objects into tree structures to represent part-whole hierarchies; treats leaves and composites uniformly. |
| **State** | Behavioral | Allows an object to alter its behavior when its internal state changes. |
| **Proxy** | Structural | Provides a surrogate or placeholder for another object to control access to it. |
| **Compound (MVC)** | Architectural | Combines Observer, Strategy, and Composite to separate Model, View, and Controller concerns. |
