# Builder Pattern – Interview Notes

## 1. What is it?
A **creational design pattern** that constructs a complex object **step by step** and returns the final object with the desired attribute values. Construction logic is separated from the object's representation.

---

## 2. Problem it solves
When an object has many attributes, creating it directly or via a factory gets painful:

1. **Too many constructor arguments** – hard to read, easy to mix up the order (the "telescoping constructor" problem).
2. **Some parameters are optional** – you end up with many overloaded constructors or lots of `null`s.
3. **A factory takes all responsibility for creation** – if the object is heavy, all the complexity ends up inside the factory class.

**Builder's answer:** build the object step by step, then call `build()` to get the final object. It also helps create **immutable objects**.

---

## 3. When to use
- A class has **multiple optional parameters**.
- Object needs **complex initialization**.
- You want object creation to be **readable and maintainable** (fluent chaining).
- You want **immutable objects**.

---

## 4. Core principles

### 4.1 `CarBuilder` should not have getters
- The builder's only purpose is to **build** the object, not to expose values.
- Once built, read values from the constructed object (`Car`), not the builder.

### 4.2 `Car` should not have setters
- Builder encourages **immutability**: once created, the object should not change.
- Setters break immutability and can introduce inconsistent state.

---

## 5. Structure

| Component | Role |
|---|---|
| **Product** (`Car`) | The complex object being built; immutable, private constructor |
| **Builder** (`Car.CarBuilder`) | Holds the fields, has chained setter-like methods, and a `build()` method |
| **Director** *(optional, classic GoF)* | Defines the order of build steps for common configurations |

---

## 6. Java example (static nested builder)

```java
public final class Car {
    // required
    private final String engine;
    private final int seats;
    // optional
    private final boolean sunroof;
    private final boolean gps;
    private final String color;

    private Car(CarBuilder b) {
        this.engine  = b.engine;
        this.seats   = b.seats;
        this.sunroof = b.sunroof;
        this.gps     = b.gps;
        this.color   = b.color;
    }

    // only getters, NO setters
    public String getEngine()  { return engine; }
    public int getSeats()      { return seats; }
    public boolean hasSunroof(){ return sunroof; }
    public boolean hasGps()    { return gps; }
    public String getColor()   { return color; }

    public static class CarBuilder {
        // required
        private final String engine;
        private final int seats;
        // optional with defaults
        private boolean sunroof = false;
        private boolean gps = false;
        private String color = "white";

        public CarBuilder(String engine, int seats) {   // required params in constructor
            this.engine = engine;
            this.seats = seats;
        }

        public CarBuilder sunroof(boolean sunroof) { this.sunroof = sunroof; return this; }
        public CarBuilder gps(boolean gps)         { this.gps = gps;         return this; }
        public CarBuilder color(String color)      { this.color = color;     return this; }

        public Car build() {
            // validation lives here
            if (seats <= 0) throw new IllegalStateException("seats must be > 0");
            return new Car(this);
        }
    }
}
```

**Usage**

```java
Car car = new Car.CarBuilder("V8", 4)
        .sunroof(true)
        .color("red")
        .build();
```

---

## 7. Benefits
- Readable, self-documenting object creation (fluent API)
- No telescoping constructors / no long argument lists
- Optional params handled cleanly with defaults
- Produces **immutable**, thread-safe objects
- Validation can be centralized in `build()`
- Same construction process can create different representations

## 8. Drawbacks
- More code (extra builder class per product)
- Builder and product must be kept in sync when fields change
- Overkill for simple objects with 2–3 fields

---

## 9. Builder vs Factory

| | Factory | Builder |
|---|---|---|
| Focus | **Which** object/type to create | **How** to construct a complex object |
| Creation | Single step | Step by step |
| Complexity | Hidden entirely inside factory | Spread across the builder's steps |
| Optional params | Awkward | Natural |
| Typical use | Returning different subclasses | Objects with many attributes |

---

## 10. Real-world Java examples
- `StringBuilder` / `StringBuffer`: step-by-step construction using `append()`. Note: the builder itself is **mutable**; the `String` it produces is the immutable result.
- `Stream.Builder`
- `Locale.Builder`
- `HttpRequest.newBuilder()` (Java 11+ HTTP client)
- `Lombok @Builder`
- Spring: `UriComponentsBuilder`, `ResponseEntity.ok().header(...).body(...)`

---

## 11. Quick interview Q&A

**Q: Why not just use setters?**
Setters leave the object in a partially built, mutable, possibly inconsistent state, and aren't thread-safe. Builder gives a fully valid, immutable object at the end.

**Q: Why is the Car constructor private?**
To force creation through the builder so no one can bypass validation or build half-initialized objects.

**Q: Where should validation go?**
Inside `build()`, so an invalid object can never be created.

**Q: How do you make the object truly immutable?**
`final` class, `final` fields, no setters, private constructor, and defensive copies for mutable fields (e.g. `List`, `Date`).

**Q: Is Builder thread-safe?**
Typically no, a builder is meant to be used by a single thread. The **built** object is thread-safe because it's immutable.

**Q: Builder vs Telescoping Constructor vs JavaBeans?**
- Telescoping: unreadable with many params.
- JavaBeans (no-arg constructor + setters): inconsistent state, not immutable.
- Builder: readable, safe, immutable.

**Q: How does Lombok help?**
`@Builder` generates the builder class automatically; pair it with `@Value` for immutability.

---

## 12. One-liner to remember
> *Builder = construct a complex object step by step, keep it immutable, and make the creation code readable.*