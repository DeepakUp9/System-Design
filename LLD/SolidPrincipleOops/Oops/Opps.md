# Object-Oriented Programming (OOP) Concepts

OOP concepts enable developers to create modular, organized, maintainable, reusable, and extensible software systems. Below is the complete picture — core concepts, abstract classes vs interfaces, and the extra pillars that usually come up in senior-level discussions (association/aggregation/composition, SOLID).

---

## 1. Class

A class is a blueprint for creating objects. It defines the structure (attributes) and behavior (methods) that instances of the class will possess.

```java
class Order {
    private String orderId;
    private double amount;

    public Order(String orderId, double amount) {
        this.orderId = orderId;
        this.amount = amount;
    }

    public double getAmount() {
        return amount;
    }
}
```

## 2. Object

An object is an instance of a class. It represents a real-world entity and holds actual data.  
we can say that objects have state(s) and behavior(s).



```java
Order order1 = new Order("ORD-101", 499.99);
```

`Order` is the class; `order1` is the object.

---

## 3. Encapsulation

Bundling data and the methods that operate on it into a single unit (a class), while restricting direct access to internal state. Access happens only through defined methods (getters/setters, or more meaningful domain methods), which hides implementation details and protects invariants.

```java
class BankAccount {
    private double balance; // hidden from outside

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

Nobody can do `account.balance = -500;` — the class controls how its state changes.

## 4. Abstraction

Simplifying complex reality by exposing only essential properties/behaviors and hiding non-essential details. Focus on **what** an object does, not **how** it does it.

```java
interface PaymentProcessor {
    void pay(double amount);
}
```

A caller just calls `pay()` — whether it's UPI, card, or wallet internally is irrelevant to them.

> **Encapsulation vs Abstraction (common interview trap):** Encapsulation is about *hiding data/state* via access control. Abstraction is about *hiding implementation complexity* via simplified interfaces. Encapsulation is a technique; abstraction is a design goal.

## 5. Inheritance 

**Definition:** Creating a new class from an existing one. The child inherits the parent's public attributes and methods and specializes them.

**When to use:** Only when an **IS-A** relationship exists.
- `Square IS-A Shape`, `Dog IS-A Animal`, `Car IS-A Vehicle` ✅
- If it's HAS-A (`Car HAS-A Engine`) → use composition, not inheritance ❌

---

## Modes (access modifiers)

Modifiers define who can access a member directly.

| Modifier | Class | Package | Subclass (other pkg) | World |
|---|:-:|:-:|:-:|:-:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| default | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

---

## 5 types

| Type | Meaning | Example |
|---|---|---|
| **Single** | One child, one parent | `FuelCar → Vehicle` |
| **Multiple** | One child, many parents | `HybridCar → FuelCar + ElectricCar` |
| **Multi-level** | Chain of inheritance | `GasolineCar → FuelCar → Vehicle` |
| **Hierarchical** | Many children, one parent | `FuelCar`, `ElectricCar` → `Vehicle` |
| **Hybrid** | Mix of the above | Vehicle → Fuel/Electric → Hybrid |

> Java, C#, JavaScript don't support multiple inheritance via classes — use **interfaces** (avoids the diamond problem).

---

## Code

```java
class Vehicle {
    protected String model;
    Vehicle(String model) { this.model = model; }
    void start() { System.out.println("Starting..."); }
}

class FuelCar extends Vehicle {          // single
    FuelCar(String m) { super(m); }
    @Override void start() { System.out.println("Igniting engine"); }
    void refuel() { System.out.println("Refueling"); }
}

class GasolineCar extends FuelCar {      // multi-level
    GasolineCar(String m) { super(m); }
}

// multiple inheritance via interfaces
interface Chargeable { void charge(); }
class HybridCar extends FuelCar implements Chargeable {
    HybridCar(String m) { super(m); }
    public void charge() { System.out.println("Charging"); }
}
```

---

## Advantages

- **Reusability** — no duplicate code in child classes
- **Code modification** — changes stay localized, no inconsistencies
- **Extensibility** — upgrade parts without touching the core
- **Data hiding** — base class keeps data private (encapsulation)

---

## Interview points

- Favour **composition over inheritance** — inheritance is the tightest coupling.
- **LSP:** a subclass must be usable anywhere the parent is expected.
- **Override** = same signature, runtime dispatch. **Overload** = different params, compile-time.
- Constructors aren't inherited; subclass calls `super()` first.
- `private` members are inherited but not accessible.
- Fields use compile-time type; methods use runtime type.
- `final` blocks extension; `sealed` (Java 17) restricts who can extend.



# Generalization — Quick Notes

**Definition:** Extracting common properties and behaviors from multiple classes into a single parent class. Subclasses inherit the shared stuff and specialize further.

---

## Why it matters

| Benefit | What it gives you |
|---|---|
| **Reusability** | Subclasses reuse parent code, no duplication |
| **Extensibility** | Add new classes by extending, without touching existing code |
| **Polymorphism** | Parent reference can point to any child object |
| **Maintainability** | Shared behavior changes in one place only |

---

## Generalization vs Specialization

- **Generalization** — find common traits across classes → abstract them *up* into a parent.
- **Specialization** — create subclasses with more specific behavior/attributes *down* the tree.

`Bird` (general) → `Sparrow`, `Eagle` (specialized).

---

## Implemented via inheritance

```java
class Animal {
    void sleep() { System.out.println("Sleeping"); }
    void eat()   { System.out.println("Eating"); }
}

class Dog extends Animal {
    void bark() { System.out.println("Woof"); }
}

class Cat extends Animal {
    void meow() { System.out.println("Meow"); }
}
```

`sleep()` and `eat()` live once in `Animal` instead of being repeated in both children.

---

## Generalization → Polymorphism

```java
class Shape { void draw() { } }
class Circle extends Shape { void draw() { System.out.println("Circle"); } }
class Rectangle extends Shape { void draw() { System.out.println("Rectangle"); } }

Shape myShape = new Circle();     // draws Circle
myShape = new Rectangle();        // same reference, draws Rectangle
```

One reference type, many runtime types → less type checking, more flexible code.

---

## Real-world: vehicle management system

```
                 Vehicle  (registrationNumber, capacity, move())
                ▲                    ▲
         LandVehicle            AirVehicle
         ▲         ▲            ▲         ▲
      Truck       Car     Helicopter   CargoPlane
   (loadType) (passenger  (rotorCount) (maxAltitude,
               Count)                   cargoVolume)
```

---

## 6. Polymorphism

**"Many forms"** — the same interface behaves differently depending on context.

### Compile-time Polymorphism (Method Overloading)
Multiple methods, same name, different parameters. Resolved at compile time.

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
}
```

### Runtime Polymorphism (Method Overriding)
A subclass provides its own implementation of a method already defined in its superclass. Resolved at runtime based on the actual object type.

```java
class Vehicle {
    void start() { System.out.println("Generic start"); }
}

class ElectricCar extends Vehicle {
    @Override
    void start() { System.out.println("Silent electric start"); }
}

Vehicle v = new ElectricCar();
v.start(); // "Silent electric start" — decided at runtime
```


# Polymorphism — Short Notes

**Definition:** "Poly" (many) + "morph" (forms) — an object/method can take many forms. E.g. `Animal.makeNoise()` behaves differently for `Lion`, `Dog`, `Crocodile`.

**Two types:** Static (compile-time) and Dynamic (runtime).

---

## Dynamic polymorphism (runtime)

Achieved via **method overriding** — subclass redefines a method with same name, return type, and parameters as the parent.

```java
class Animal {
    void printAnimal() { System.out.println("I am an animal"); }
}
class Lion extends Animal {
    @Override
    void printAnimal() { System.out.println("I am a lion"); }
}
```

Call resolved at runtime based on actual object type.

---

## Static polymorphism (compile-time)

Achieved via **method overloading** or **operator overloading**.

**Method overloading** — same method name, different parameter count/type:
```java
void add(int a, int b) { }
void add(int a, int b, int c) { }
```

**Operator overloading** — same operator, different behavior per type (e.g. `+` adds ints, concatenates strings, or adds complex numbers via custom logic).
> Java and JavaScript do **not** support operator overloading.

---

## Static vs Dynamic — comparison

| | Static | Dynamic |
|---|---|---|
| Resolved | Compile-time | Runtime |
| Mechanism | Overloading | Overriding |
| Purpose | Readability | Separate implementation per subclass |
| Arguments | Must differ | Must be same |
| Return type | Doesn't matter | Must match |
| Private/sealed methods | Can overload | Cannot override |
| Performance | Better (compile-time binding) | Worse (runtime binding) |

---

## Abstract Classes vs Interfaces

### Abstract Classes
- A blueprint/template for subclasses
- Cannot be instantiated directly
- Can mix method declarations **and** actual implementations
- Subclasses must implement the abstract methods

```java
abstract class Shape {
    abstract double area();          // must be implemented by subclass

    void describe() {                 // shared implementation
        System.out.println("This shape has area: " + area());
    }
}

class Circle extends Shape {
    double radius;
    Circle(double radius) { this.radius = radius; }

    double area() { return Math.PI * radius * radius; }
}
```

### Interfaces
- A pure contract — a set of method signatures a class must adhere to
- No implementation (until `default` methods, which are the exception, not the norm)
- Lets unrelated classes share common behavior
- A class can implement multiple interfaces (Java doesn't allow multiple class inheritance, but does allow this)

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    public void fly()  { System.out.println("Duck flying"); }
    public void swim() { System.out.println("Duck swimming"); }
}
```

| | Abstract Class | Interface |
|---|---|---|
| Instantiable | No | No |
| Implementation | Can have some | None (except `default`) |
| Multiple inheritance | Not supported (single) | A class can implement many |
| Fields | Can have instance state | Only `public static final` constants |
| Use when | Sharing common code among closely related classes | Defining a capability/contract across unrelated classes |

---

## Bonus: Relationships Between Objects (often asked alongside inheritance)

- **Association** — a general relationship where objects know about each other (e.g., a `Teacher` and a `Student`).
- **Aggregation** — a "has-a" relationship where the child can exist independently of the parent (e.g., a `Department` has `Employees`, but employees still exist if the department is dissolved).
- **Composition** — a stronger "has-a" relationship where the child's lifecycle depends on the parent (e.g., a `House` has `Rooms`; destroy the house, the rooms are gone too).

```java
class Engine {}                     // Composition: Engine can't meaningfully exist without a Car
class Car {
    private final Engine engine = new Engine();
}
```

---

## Quick Recap

| Concept | One-liner |
|---|---|
| Class | Blueprint |
| Object | Instance of a class |
| Encapsulation | Hide state, expose behavior |
| Abstraction | Hide complexity, expose essentials |
| Inheritance | Reuse via "is-a" |
| Polymorphism | One interface, many forms |
| Abstract Class | Partial blueprint, single inheritance |
| Interface | Pure contract, multiple implementation |