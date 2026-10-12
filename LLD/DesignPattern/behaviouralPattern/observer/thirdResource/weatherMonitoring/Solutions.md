# Observer Design Pattern – Weather Monitoring (Simple Java)

## Question
Observer Design Pattern kya hai, aur ye kin scenarios mein use hota hai?
Use-case: Weather app jahan multiple display screens (current, forecast, statistics) weather data change hone pe automatically update ho jati hain.

---

## Observer Pattern kya hai?
Ek **behavioral design pattern** jisme:
- Ek object (**Subject**) ki state change hoti hai
- To usse jude saare objects (**Observers**) ko **automatically notification** mil jata hai
- Subject ko observers ke andar ki details nahi pata, wo sirf common interface jaanta hai (**loose coupling**)

**One-to-many relationship:** ek Subject, bahut saare Observers.

Real-life example: YouTube channel (Subject) pe naya video aaya, saare subscribers (Observers) ko notification mil gayi.

---

## Roles

| Role | Is example mein | Kaam |
|---|---|---|
| **Subject** | `WeatherData` | Observers ki list rakhta hai, data badalne pe sabko notify karta hai |
| **Observer** (interface) | `Observer` | `update()` method define karta hai |
| **Concrete Observers** | `CurrentConditionsDisplay`, `ForecastDisplay`, `StatisticsDisplay` | Notification milne pe apni screen update karte hain |

---

```java
import java.util.ArrayList;
import java.util.List;

// 1. Observer interface
interface Observer {
    void update(double temperature, double humidity, double pressure);
}

// 2. Subject interface
interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}

// 3. Concrete Subject
class WeatherData implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    private double temperature;
    private double humidity;
    private double pressure;

    public void registerObserver(Observer o) {
        observers.add(o);
    }

    public void removeObserver(Observer o) {
        observers.remove(o);
    }

    public void notifyObservers() {
        for (Observer o : observers) {
            o.update(temperature, humidity, pressure);
        }
    }

    // Naya data aaya -> sabko notify karo
    public void setMeasurements(double temperature, double humidity, double pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        notifyObservers();
    }
}

// 4. Concrete Observers

class CurrentConditionsDisplay implements Observer {
    public void update(double temperature, double humidity, double pressure) {
        System.out.printf("Current : %.1f C, %.1f%% humidity%n", temperature, humidity);
    }
}

class ForecastDisplay implements Observer {
    private double lastPressure = 0;

    public void update(double temperature, double humidity, double pressure) {
        if (lastPressure == 0) {
            System.out.println("Forecast: Collecting data...");
        } else if (pressure > lastPressure) {
            System.out.println("Forecast: Improving weather on the way!");
        } else if (pressure < lastPressure) {
            System.out.println("Forecast: Cooler, rainy weather ahead");
        } else {
            System.out.println("Forecast: More of the same");
        }
        lastPressure = pressure;
    }
}

class StatisticsDisplay implements Observer {
    private double sum = 0;
    private int count = 0;
    private double max = Double.NEGATIVE_INFINITY;
    private double min = Double.POSITIVE_INFINITY;

    public void update(double temperature, double humidity, double pressure) {
        sum += temperature;
        count++;
        max = Math.max(max, temperature);
        min = Math.min(min, temperature);
        System.out.printf("Stats   : Avg %.1f / Max %.1f / Min %.1f%n", sum / count, max, min);
    }
}

// 5. Client
public class WeatherStation {
    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();

        Observer current  = new CurrentConditionsDisplay();
        Observer forecast = new ForecastDisplay();
        Observer stats    = new StatisticsDisplay();

        // Observers ko register karo
        weatherData.registerObserver(current);
        weatherData.registerObserver(forecast);
        weatherData.registerObserver(stats);

        System.out.println("--- Update 1 ---");
        weatherData.setMeasurements(28, 65, 1013);

        System.out.println("--- Update 2 ---");
        weatherData.setMeasurements(30, 70, 1015);

        // Runtime pe ek observer hata diya
        weatherData.removeObserver(forecast);

        System.out.println("--- Update 3 (forecast removed) ---");
        weatherData.setMeasurements(25, 80, 1008);
    }
}
```

**Output**
```
--- Update 1 ---
Current : 28.0 C, 65.0% humidity
Forecast: Collecting data...
Stats   : Avg 28.0 / Max 28.0 / Min 28.0
--- Update 2 ---
Current : 30.0 C, 70.0% humidity
Forecast: Improving weather on the way!
Stats   : Avg 29.0 / Max 30.0 / Min 28.0
--- Update 3 (forecast removed) ---
Current : 25.0 C, 80.0% humidity
Stats   : Avg 27.7 / Max 30.0 / Min 25.0
```

---

## Naya observer add karna (e.g. Alert screen)

```java
class AlertDisplay implements Observer {
    public void update(double temperature, double humidity, double pressure) {
        if (temperature > 40) {
            System.out.println("ALERT   : Heat wave warning!");
        }
    }
}

// use:
weatherData.registerObserver(new AlertDisplay());
```

`WeatherData` ya baaki displays mein koi change nahi karna padta.

---

## Kin scenarios mein use hota hai?

| Scenario | Subject | Observers |
|---|---|---|
| Weather app | Weather data | Current / Forecast / Stats screens |
| Stock market app | Stock price | Charts, alerts, portfolio views |
| Social media / YouTube | Channel / user | Subscribers / followers |
| UI event handling | Button | Click listeners (`ActionListener`) |
| Payment system | Transaction status change | SMS service, Email service, Ledger, Audit |
| Messaging / events | Kafka topic | Consumer services (distributed version, pub/sub) |
| MVC architecture | Model | Views |

---

## Interview mein kya bolna hai

1. **Definition:** Observer Pattern ek one-to-many dependency define karta hai. Jab Subject ki state badalti hai, saare dependents ko automatically notify aur update kar diya jata hai.
2. **Without Observer:** Observers ko baar baar data poochna padega (polling), ya Subject ko har display class ka naam pata hona chahiye (tight coupling).
3. **Benefits:**
   - **Loose coupling:** Subject ko sirf `Observer` interface pata hai
   - Runtime pe observers add/remove kar sakte hain
   - Naya observer = naya class, Subject untouched (Open/Closed)
4. **Drawbacks:**
   - Notification ka **order guarantee nahi** hota
   - Observer ko unregister karna bhool gaye to **memory leak** (lapsed listener problem)
   - Cascading updates ho sakte hain, debug karna mushkil
5. **Push vs Pull model:**
   - **Push:** Subject saara data `update()` mein bhej deta hai (jaise upar code mein)
   - **Pull:** Subject sirf "kuch badla hai" bolta hai, observer jo chahiye wo getter se khud le leta hai
6. **Java mein built-in:** `java.util.Observable` / `Observer` **Java 9 se deprecated** hain. Ab `PropertyChangeListener`, ya apna interface use karte hain, ya `java.util.concurrent.Flow` / reactive libraries.

## Observer vs Pub/Sub (agar puche)
- **Observer:** Subject ke paas observers ke direct references hote hain. Same process mein, usually synchronous.
- **Pub/Sub:** Publisher aur subscriber ke beech **broker/topic** hota hai (Kafka, RabbitMQ). Dono ek dusre ko nahi jaante, aur usually async aur distributed hota hai.

## One-liner
> Subject ki state badli, saare observers ko automatically khabar mil gayi. Subject ko observers ke andar ka kuch pata nahi.