# Behavioral Design Patterns – Interview Notes

## 1. What are they?
Behavioral patterns deal with **how objects communicate and share responsibility**. They focus on algorithms, workflows, and the assignment of responsibilities between objects, while keeping them **loosely coupled**.

> Creational = *how objects are created*
> Structural = *how objects are composed*
> **Behavioral = how objects interact and distribute responsibility**

## 2. The 11 GoF behavioral patterns (cheat sheet)

| Pattern | One-line intent | Java / real-world example |
|---|---|---|
| **Strategy** | Swap algorithms at runtime via a common interface | `Comparator`, payment methods |
| **Observer** | One-to-many notification when state changes | `PropertyChangeListener`, Kafka consumers, event listeners |
| **Command** | Encapsulate a request as an object | `Runnable`, undo/redo, job queues |
| **State** | Object changes behavior when its internal state changes | Order status, payment txn lifecycle, vending machine |
| **Template Method** | Fixed algorithm skeleton, subclasses fill in steps | `AbstractList`, Spring `JdbcTemplate` |
| **Chain of Responsibility** | Pass request along handlers until one handles it | Servlet filters, Spring Security filter chain |
| **Iterator** | Sequential access without exposing internals | `java.util.Iterator`, `for-each` |
| **Mediator** | Central object coordinates communication between objects | Chat room, `ExecutorService`, Spring `ApplicationContext` |
| **Memento** | Capture and restore object state without breaking encapsulation | Undo, snapshots, editor history |
| **Visitor** | Add new operations to a class hierarchy without modifying it | `FileVisitor`, AST traversal |
| **Interpreter** | Define a grammar and interpret sentences in it | Regex, SpEL, SQL parsers |

**Most asked in interviews:** Strategy, Observer, Template Method, Command, State, Chain of Responsibility, Iterator.

---

## 3. Patterns in detail

### 3.1 Strategy
**Intent:** Define a family of algorithms, encapsulate each one, make them interchangeable at runtime.

**Use when:** many `if/else` or `switch` blocks pick between algorithms (payment, sorting, pricing, discount).

```java
interface PaymentStrategy { void pay(double amount); }

class UpiPayment implements PaymentStrategy {
    public void pay(double amount) { System.out.println("Paid " + amount + " via UPI"); }
}
class CardPayment implements PaymentStrategy {
    public void pay(double amount) { System.out.println("Paid " + amount + " via Card"); }
}

class Checkout {
    private PaymentStrategy strategy;
    public Checkout(PaymentStrategy strategy) { this.strategy = strategy; }
    public void setStrategy(PaymentStrategy s) { this.strategy = s; }
    public void process(double amt) { strategy.pay(amt); }
}
```

- **Pros:** removes conditionals, follows Open/Closed Principle, easy to test.
- **Cons:** client must know the strategies; more classes.
- **Java:** `Comparator`, Spring's `@Qualifier`-injected implementations, `Map<String, Strategy>` lookup.

---

### 3.2 Observer
**Intent:** When one object (Subject) changes state, all dependents (Observers) are notified automatically.

**Use when:** event systems, notifications, pub/sub, UI updates.

```java
interface Observer { void update(String event); }

class Subject {
    private final List<Observer> observers = new ArrayList<>();
    public void subscribe(Observer o)   { observers.add(o); }
    public void unsubscribe(Observer o) { observers.remove(o); }
    public void notifyAll(String event) { observers.forEach(o -> o.update(event)); }
}

class EmailService implements Observer {
    public void update(String event) { System.out.println("Email sent for " + event); }
}
class SmsService implements Observer {
    public void update(String event) { System.out.println("SMS sent for " + event); }
}
```

- **Pros:** loose coupling, dynamic subscribe/unsubscribe.
- **Cons:** unpredictable notification order, memory leaks if observers aren't removed (lapsed listener problem), hard to debug.
- **Related:** Pub/Sub (Kafka, RabbitMQ) is the distributed, broker-based version. Observer is in-process and direct, pub/sub has a broker between publisher and subscriber.

---

### 3.3 Command
**Intent:** Wrap a request as an object so it can be queued, logged, retried, or undone.

```java
interface Command { void execute(); void undo(); }

class Light { void on() {...} void off() {...} }

class LightOnCommand implements Command {
    private final Light light;
    LightOnCommand(Light l) { this.light = l; }
    public void execute() { light.on(); }
    public void undo()    { light.off(); }
}

class RemoteControl {                       // Invoker
    private final Deque<Command> history = new ArrayDeque<>();
    void press(Command c) { c.execute(); history.push(c); }
    void undo() { if (!history.isEmpty()) history.pop().undo(); }
}
```

- **Roles:** Command, Concrete Command, Receiver, Invoker, Client.
- **Pros:** decouples invoker from receiver, supports undo/redo, queueing, logging, retries.
- **Cons:** class explosion for many commands.
- **Java:** `Runnable`, `Callable`, job/task queues, transaction retry.

---

### 3.4 State
**Intent:** Let an object alter its behavior when its internal state changes, so it appears to change class.

**Use when:** large `switch(status)` blocks, lifecycle workflows (Order: `CREATED → PAID → SHIPPED → DELIVERED`, Transaction: `INITIATED → PENDING → SUCCESS/FAILED`).

```java
interface TxnState { void next(Transaction t); }

class Initiated implements TxnState {
    public void next(Transaction t) { t.setState(new Pending()); }
}
class Pending implements TxnState {
    public void next(Transaction t) { t.setState(new Success()); }
}
class Success implements TxnState {
    public void next(Transaction t) { /* terminal */ }
}

class Transaction {
    private TxnState state = new Initiated();
    void setState(TxnState s) { this.state = s; }
    void proceed() { state.next(this); }
}
```

- **State vs Strategy:** structurally the same. **Strategy** is chosen by the *client* and strategies usually don't know each other. **State** transitions are driven by the *object itself*, and states often know the next state.
- **Pros:** removes big conditionals, each state is isolated, easy to add states.
- **Cons:** more classes; overkill for 2–3 simple states.

---

### 3.5 Template Method
**Intent:** Define the skeleton of an algorithm in a base class, let subclasses override specific steps without changing the structure.

```java
abstract class DataProcessor {
    public final void process() {          // final -> skeleton can't be overridden
        readData();
        transform();
        save();
    }
    abstract void readData();
    abstract void transform();
    void save() { System.out.println("Saved to DB"); }   // default step (hook)
}

class CsvProcessor extends DataProcessor {
    void readData()  { System.out.println("Reading CSV"); }
    void transform() { System.out.println("Transforming CSV"); }
}
```

- **Uses inheritance** (Strategy uses composition).
- **Hook methods:** optional steps with default behavior that subclasses may override.
- **Java:** `JdbcTemplate`, `RestTemplate`, `AbstractList`, `HttpServlet.service()`.
- **Principle:** "Hollywood Principle" – *don't call us, we'll call you*.

---

### 3.6 Chain of Responsibility
**Intent:** Pass a request along a chain of handlers. Each handler decides to process it, pass it on, or both.

```java
abstract class Handler {
    protected Handler next;
    public Handler setNext(Handler n) { this.next = n; return n; }
    public abstract void handle(Request r);
}

class AuthHandler extends Handler {
    public void handle(Request r) {
        if (!r.isAuthenticated()) throw new SecurityException("Unauthorized");
        if (next != null) next.handle(r);
    }
}
class ValidationHandler extends Handler { /* validate, then next.handle(r) */ }
class LoggingHandler extends Handler    { /* log, then next.handle(r) */ }
```

- **Pros:** decouples sender from receivers, easy to add/reorder handlers, single responsibility per handler.
- **Cons:** no guarantee the request is handled, harder to debug.
- **Java:** Servlet `Filter` / `FilterChain`, Spring Security filter chain, Spring `HandlerInterceptor`, logging levels, approval workflows.

---

### 3.7 Iterator
**Intent:** Access elements of a collection sequentially without exposing its underlying representation.

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.isEmpty()) it.remove();   // safe removal
}
```

- **Fail-fast vs fail-safe:** `ArrayList`/`HashMap` iterators throw `ConcurrentModificationException` if modified during iteration (fail-fast). `CopyOnWriteArrayList`/`ConcurrentHashMap` iterators don't (fail-safe / weakly consistent).
- **`Iterator` vs `Iterable`:** `Iterable` produces an `Iterator`, and enables for-each.

---

### 3.8 Mediator
**Intent:** Centralize complex communication between objects so they don't refer to each other directly.

- Replaces many-to-many with many-to-one relationships.
- **Example:** chat room (users talk through the room, not to each other), air traffic control.
- **Con:** the mediator can become a "god object".
- **Mediator vs Observer:** Observer is one-to-many broadcast. Mediator is a central coordinator that manages bidirectional interaction logic.

---

### 3.9 Memento
**Intent:** Capture an object's internal state so it can be restored later, without violating encapsulation.

- **Roles:** **Originator** (creates/restores from memento), **Memento** (immutable snapshot), **Caretaker** (stores mementos, never inspects them).
- **Use:** undo/redo, checkpoints, rollback.
- **Con:** memory heavy if snapshots are large or frequent.
- Often paired with **Command** for undo.

---

### 3.10 Visitor
**Intent:** Add new operations to existing object structures without modifying their classes (double dispatch).

```java
interface Shape { void accept(ShapeVisitor v); }
class Circle implements Shape { public void accept(ShapeVisitor v) { v.visit(this); } }
class Square implements Shape { public void accept(ShapeVisitor v) { v.visit(this); } }

interface ShapeVisitor { void visit(Circle c); void visit(Square s); }
class AreaCalculator implements ShapeVisitor { /* implement both visit methods */ }
```

- **Pros:** easy to add new operations (Open/Closed for operations).
- **Cons:** hard to add new element types (every visitor must change); breaks encapsulation.
- **Java:** `FileVisitor` / `Files.walkFileTree`, compiler AST visitors.

---

### 3.11 Interpreter
**Intent:** Define a grammar for a language and an interpreter to evaluate sentences in it.

- **Use:** expression evaluators, rule engines, simple DSLs.
- **Java:** `java.util.regex.Pattern`, Spring Expression Language (SpEL).
- Rarely coded by hand; for complex grammars use parser generators (ANTLR).

---

## 4. Strategy vs State vs Template Method

| | Strategy | State | Template Method |
|---|---|---|---|
| Mechanism | Composition | Composition | **Inheritance** |
| Who changes behavior | Client | The object itself (transitions) | Fixed at compile time via subclass |
| Changes at runtime? | Yes | Yes | No |
| Intent | Interchangeable algorithms | Behavior depends on state | Fixed skeleton, variable steps |

## 5. Observer vs Mediator vs Chain of Responsibility

| | Observer | Mediator | Chain of Responsibility |
|---|---|---|---|
| Communication | One → many broadcast | Many ↔ many via a hub | Sequential, one handler at a time |
| Coupling | Subject doesn't know observer details | Colleagues only know the mediator | Handlers only know the next handler |

## 6. Where these show up in backend / payments work
- **Strategy:** choosing payment provider / routing rule / fee calculation per country or operator.
- **State:** transaction lifecycle (`INITIATED → PENDING → SUCCESS / FAILED / REVERSED`).
- **Observer / Pub-Sub:** publishing a transaction event to Kafka so notification, ledger and audit services react independently.
- **Chain of Responsibility:** request validation → fraud check → limit check → processing pipeline; servlet / security filters.
- **Template Method:** common processing flow (validate → process → persist → notify) with provider-specific steps.
- **Command:** retryable / queued payment jobs, compensating actions in sagas.

*(Good to mention in interviews: tie the pattern to a real system you've built.)*

## 7. Quick interview Q&A

**Q: Which pattern removes large if/else or switch blocks?**
Strategy (choose algorithm) or State (behavior depends on status).

**Q: Difference between Strategy and State?**
Strategy: client picks the algorithm, strategies are independent. State: the object transitions between states on its own, states often know each other.

**Q: Difference between Observer and Pub/Sub?**
Observer: subject directly holds observer references, in-process, synchronous. Pub/Sub: publisher and subscriber are decoupled through a broker/topic, usually async and distributed.

**Q: How does Template Method differ from Strategy?**
Template Method uses inheritance and fixes the algorithm skeleton at compile time. Strategy uses composition and swaps whole algorithms at runtime.

**Q: Which patterns support undo?**
Command (store inverse operation) and Memento (store state snapshot).

**Q: Where is Chain of Responsibility used in Spring / Java EE?**
Servlet filters, Spring Security filter chain, `HandlerInterceptor`.

**Q: Can you replace Strategy with lambdas in Java 8+?**
Yes. A single-method strategy interface is a functional interface, so `Comparator<String> c = (a, b) -> a.length() - b.length();` is a Strategy.

**Q: Downsides of Observer?**
Memory leaks from forgotten listeners, unpredictable order, cascading updates, hard debugging.

**Q: Which behavioral pattern uses double dispatch?**
Visitor.

**Q: Which patterns are most related?**
State ↔ Strategy (same structure), Command ↔ Memento (undo), Mediator ↔ Observer (communication), Template Method ↔ Strategy (alternative ways to vary algorithms).

---

## 8. One-liners to remember
- **Strategy** – swap the algorithm.
- **Observer** – notify subscribers on change.
- **Command** – turn a request into an object.
- **State** – behavior follows state.
- **Template Method** – fixed skeleton, custom steps.
- **Chain of Responsibility** – pass it down the line.
- **Iterator** – traverse without exposing internals.
- **Mediator** – one hub for communication.
- **Memento** – save and restore state.
- **Visitor** – add operations without touching classes.
- **Interpreter** – evaluate a grammar.