# Strategy Pattern – Payment Demo (Interview Version)

## Idea (2 line mein)
- Har payment method (Credit Card, PayPal, Crypto) ka apna class, sab `PaymentStrategy` interface implement karte hain.
- `PaymentProcessor` bas `strategy.processPayment()` call karta hai. Kaunsa method chal raha hai, use pata nahi. Runtime pe badal sakte hain.

---

```java
// 1. Strategy interface
interface PaymentStrategy {
    void processPayment(double amount);
}

// 2. Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
    }
}

class PayPalPayment implements PaymentStrategy {
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment of $" + amount);
    }
}

class CryptocurrencyPayment implements PaymentStrategy {
    public void processPayment(double amount) {
        System.out.println("Processing cryptocurrency payment of $" + amount);
    }
}

// 3. Factory: user ke choice (String) se sahi strategy banata hai
class PaymentStrategyFactory {
    public static PaymentStrategy create(String method) {
        switch (method) {
            case "CreditCard":     return new CreditCardPayment();
            case "PayPal":         return new PayPalPayment();
            case "Cryptocurrency": return new CryptocurrencyPayment();
            default:
                throw new IllegalArgumentException("Unsupported payment method: " + method);
        }
    }
}

// 4. Context
class PaymentProcessor {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void processPayment(double amount) {
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment strategy not set");
        }
        paymentStrategy.processPayment(amount);
    }
}

// 5. Client
public class PaymentDemo {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();

        // Strategy runtime pe set karo
        processor.setPaymentStrategy(new CreditCardPayment());
        processor.processPayment(100.0);

        // Same processor, strategy badal di
        processor.setPaymentStrategy(new PayPalPayment());
        processor.processPayment(50.0);

        // User ke input (String) se strategy lena ho to factory use karo
        processor.setPaymentStrategy(PaymentStrategyFactory.create("Cryptocurrency"));
        processor.processPayment(75.0);
    }
}
```

**Output**
```
Processing credit card payment of $100.0
Processing PayPal payment of $50.0
Processing cryptocurrency payment of $75.0
```

---

## Tumhare code mein kya hata diya (aur kyu)

| Tumhara code | Problem | Clean version |
|---|---|---|
| `finalize()` method | Java 9 se **deprecated**, bekaar hai. Interview mein red flag ban sakta hai | Hata diya |
| `paymentStrategy = null` "cleanup" | C++ wali aadat hai. Java mein GC khud memory handle karta hai, null karne ki zaroorat nahi | Seedha `this.paymentStrategy = strategy` |
| Constructor mein `paymentStrategy = null` | Field by default `null` hi hota hai | Constructor hata diya |
| Unknown method pe silently `CreditCard` return | Galat paise ka galat method se payment ho sakta hai. Payment mein silent default dangerous hai | `IllegalArgumentException` throw kiya |
| Strategy set nahi hai to `System.err.println` | Error chhup jata hai, program chalta rehta hai | `IllegalStateException` throw kiya |
| Do alag files (Strategy aur Strategy+Factory) | Dono ek hi cheez hain | Ek file, factory optional part ki tarah |

---

## Naya payment method add karna (e.g. UPI)

```java
// 1. Naya class
class UpiPayment implements PaymentStrategy {
    public void processPayment(double amount) {
        System.out.println("Processing UPI payment of $" + amount);
    }
}

// 2. Factory ke switch mein ek case
case "UPI": return new UpiPayment();
```

`PaymentProcessor` aur baaki strategies untouched rehti hain.

---

## Interview mein kya bolna hai

1. **Definition:** Strategy Pattern algorithms ki family ko alag classes mein daalta hai aur unhe interchangeable banata hai, runtime pe.
2. **Without Strategy:** `if (type == "card") ... else if (type == "paypal") ...` har naye method pe edit karna padta hai (Open/Closed Principle todta hai).
3. **Roles:**
   - **Strategy:** `PaymentStrategy` (interface)
   - **Concrete Strategies:** `CreditCardPayment`, `PayPalPayment`, `CryptocurrencyPayment`
   - **Context:** `PaymentProcessor`
   - **Client:** `main()`
4. **Benefits:** naya method = naya class, `if/else` nahi, har strategy alag test hoti hai, runtime pe swap.
5. **Drawback:** classes zyada ho jati hain.
6. **Factory kyu add kiya?** Strategy batati hai *kaise* kaam karna hai. Factory batati hai *kaunsi* strategy banani hai (String input se). Dono saath mein achhe chalte hain.

## Agar puche: "Factory mein bhi switch hai, to Open/Closed kaise follow hua?"
Sach bolo: *"Factory mein ek line add karni padti hai, par business logic (`PaymentProcessor` aur existing strategies) untouched rehta hai. Aur zyada flexible chahiye to Spring mein har strategy pe `@Component` laga ke `Map<String, PaymentStrategy>` inject kar sakte hain, to switch bhi hat jata hai."*

## One-liner
> Ek interface, har payment method ka ek class, processor sirf `processPayment()` call karta hai. Method badalna = strategy badalna.