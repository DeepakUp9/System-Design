# Strategy Pattern – Tax Calculator (Simple Java)

## Question

Question:
 You're building a tax calculation module that needs to support different tax policies based on region (India, US, Europe). 
 How would you use the Strategy Pattern to design this module for easy expansion and maintenance?
Example Use-Case:
 Each region has its own TaxStrategy implementation. 
 The system dynamically selects the correct strategy based on user region, 
 ensuring that adding a new country doesn’t require modifying existing logic — just plug in a new strategy.

 
Region ke hisaab se tax calculate karna hai (India, US, Europe). Strategy Pattern use karke aisa design karo ki naya country add karne pe purana code change na karna pade.



---

## Idea (2 line mein)
- Har region ka apna class, sab ek `TaxStrategy` interface implement karte hain.
- `TaxCalculator` region ke basis pe sahi strategy uthata hai aur `calculate()` call kar deta hai. `if/else` nahi.

---

## Full Code (ek file, copy-paste karke run karo)

```java
import java.util.HashMap;
import java.util.Map;

// 1. Strategy interface
interface TaxStrategy {
    double calculate(double amount);
}

// 2. Concrete strategies (har region ka apna class)
class IndiaTax implements TaxStrategy {
    public double calculate(double amount) {
        return amount * 0.18;   // GST 18%
    }
}

class USTax implements TaxStrategy {
    public double calculate(double amount) {
        return amount * 0.07;   // Sales tax 7%
    }
}

class EuropeTax implements TaxStrategy {
    public double calculate(double amount) {
        return amount * 0.20;   // VAT 20%
    }
}

// 3. Context: sahi strategy select karke use karta hai
class TaxCalculator {
    private final Map<String, TaxStrategy> strategies = new HashMap<>();

    public TaxCalculator() {
        strategies.put("INDIA", new IndiaTax());
        strategies.put("US", new USTax());
        strategies.put("EUROPE", new EuropeTax());
    }

    public double calculateTax(String region, double amount) {
        TaxStrategy strategy = strategies.get(region.toUpperCase());
        if (strategy == null) {
            throw new IllegalArgumentException("Unsupported region: " + region);
        }
        return strategy.calculate(amount);
    }
}

// 4. Main
public class Main {
    public static void main(String[] args) {
        TaxCalculator calculator = new TaxCalculator();

        System.out.println("India  : " + calculator.calculateTax("India", 1000));   // 180.0
        System.out.println("US     : " + calculator.calculateTax("US", 1000));      // 70.0
        System.out.println("Europe : " + calculator.calculateTax("Europe", 1000));  // 200.0
    }
}
```

---

## Naya country add karna (e.g. Japan)

Sirf ye karo:

```java
// 1. Naya class banao
class JapanTax implements TaxStrategy {
    public double calculate(double amount) {
        return amount * 0.10;
    }
}

// 2. TaxCalculator ke constructor mein ek line add karo
strategies.put("JAPAN", new JapanTax());
```

Existing strategies (`IndiaTax`, `USTax`, `EuropeTax`) ko touch nahi karna padta, aur calculation logic bhi same rehta hai.

---

## Interview mein kya bolna hai

1. **Problem:** Agar `if (region == "INDIA") ... else if (region == "US") ...` likhein to har naye country pe same method edit karna padega. Ye **Open/Closed Principle** todta hai.
2. **Solution:** Strategy Pattern. Algorithm (tax rule) ko alag class mein daal do, interface ke peeche.
3. **Benefit:**
   - Naya region = naya class (extension), purana code safe (no modification)
   - Har class ka ek hi kaam (Single Responsibility)
   - Testing easy, har strategy ko alag test kar sakte ho
   - Runtime pe strategy select hoti hai (map lookup)
4. **Bonus point:** Spring Boot mein har strategy pe `@Component` laga do aur `List<TaxStrategy>` inject karo, to register karne ki line bhi nahi likhni padti.

## Ek cheez dhyan rakhna
Interview mein bolna: *"Real money calculation mein `double` nahi, `BigDecimal` use karte hain, kyunki double mein precision issue aata hai."* Yahan simple rakhne ke liye `double` use kiya hai.

## One-liner
> Ek interface, har region ka ek class, ek map se lookup. Naya country = naya class.