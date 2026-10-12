# Observer Pattern – Order Status (Quick Look)

## Idea
- **Subject:** `Order` (status badalta hai)
- **Observers:** `Customer`, `Restaurant`, `CallCenter` (status badalte hi automatically notify hote hain)
- Order ko sirf `Observer` interface pata hai, kaun kaun sun raha hai ye nahi. Observer `attach` / `detach` kar sakte hain.

## Code (ek file, seedha run hoga)

```java
import java.util.ArrayList;
import java.util.List;

// Observer
interface Observer {
    void update(Order order);
}

// Concrete Observers
class Customer implements Observer {
    private final String name;
    Customer(String name) { this.name = name; }

    public void update(Order order) {
        System.out.println("Hello, " + name + "! Order #" + order.getId() + " is now " + order.getStatus());
    }
}

class Restaurant implements Observer {
    private final String name;
    Restaurant(String name) { this.name = name; }

    public void update(Order order) {
        System.out.println("Restaurant " + name + ": Order #" + order.getId() + " is now " + order.getStatus());
    }
}

class CallCenter implements Observer {
    public void update(Order order) {
        System.out.println("Call center: Order #" + order.getId() + " is now " + order.getStatus());
    }
}

// Subject
class Order {
    private final int id;
    private String status = "Order Placed";
    private final List<Observer> observers = new ArrayList<>();

    Order(int id) { this.id = id; }

    int getId() { return id; }
    String getStatus() { return status; }

    void attach(Observer o) { observers.add(o); }
    void detach(Observer o) { observers.remove(o); }

    void setStatus(String status) {
        this.status = status;
        notifyObservers();          // status badla -> sabko khabar
    }

    private void notifyObservers() {
        for (Observer o : observers) {
            o.update(this);
        }
    }
}

// Client
public class OrderStatus {
    public static void main(String[] args) {
        Order order = new Order(123);
        CallCenter callCenter = new CallCenter();

        order.attach(new Customer("Aman"));
        order.attach(new Restaurant("Pizza Hub"));
        order.attach(callCenter);

        order.setStatus("Out for Delivery");

        order.detach(callCenter);        // call center ko ab updates nahi chahiye

        order.setStatus("Delivered");
    }
}
```

## Output
```
Hello, Aman! Order #123 is now Out for Delivery
Restaurant Pizza Hub: Order #123 is now Out for Delivery
Call center: Order #123 is now Out for Delivery
Hello, Aman! Order #123 is now Delivered
Restaurant Pizza Hub: Order #123 is now Delivered
```

Dusre update mein Call center ka message nahi aaya, kyunki wo detach ho chuka tha.

## Tumhare code mein kya simplify kiya
- `DeliveryDriver` hata diya (same pattern repeat ho raha tha, 3 observers kaafi hain)
- Fields `final` kiye jo change nahi hote
- `java.util.*` ki jagah sirf zaroori imports
- Comments aur extra lines kam kiye

## Interview ke liye 3 quick points
1. **Flow:** `setStatus()` -> `notifyObservers()` -> har observer ka `update()` call.
2. **Benefit:** naya observer (e.g. `SmsService`) = naya class + ek `attach()`. `Order` mein koi change nahi.
3. **Improvement bolna ho to:** status `String` ki jagah `enum` (`PLACED, OUT_FOR_DELIVERY, DELIVERED`) use karo, typo se bachne ke liye.

## One-liner
> Order ka status badla, saare attached observers ko automatically khabar mil gayi.