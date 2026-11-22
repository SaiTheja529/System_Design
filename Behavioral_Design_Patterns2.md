# Observer Design Pattern – Java Notes

## 1. Introduction

The **Observer Design Pattern** is a **Behavioral Design Pattern** that establishes a one-to-many dependency between objects.

When one object changes its state, all its registered dependent objects are automatically notified and updated.

In simple words:

**One object changes → Multiple interested objects get notified automatically.**

### Real-world example

Consider a YouTube channel:

- A channel uploads a new video.
- All its subscribers receive notifications.
- A subscriber can unsubscribe at any time.
- The channel doesn't need to know how each subscriber handles notifications.

This is the basic idea behind the Observer Pattern.

## 2. Problem Without Observer Pattern

Suppose we are developing an e-commerce application.

Whenever an order is placed, we need to:

1. Send an email notification.
2. Send an SMS notification.
3. Update the inventory.
4. Notify the delivery service.

A tightly coupled implementation might look like this:

```java
public class OrderService {

    public void placeOrder() {

        // Save order

        EmailService email = new EmailService();
        email.sendEmail();

        SMSService sms = new SMSService();
        sms.sendSMS();

        InventoryService inventory =
            new InventoryService();

        inventory.updateInventory();
    }
}
```

### Problems

1. `OrderService` directly depends on multiple services.
2. Adding a new notification mechanism requires modifying the existing class.
3. The class becomes difficult to maintain.
4. Testing becomes harder because multiple dependencies are involved.
5. The system becomes tightly coupled.

## 3. Solution – Observer Pattern

We introduce two major components:

- **Subject (Publisher):** Maintains a list of observers and notifies them when something changes.
- **Observer (Subscriber):** Receives notifications and performs its respective operation.

### Main components

| Component | Responsibility |
|---|---|
| Subject Interface | Defines methods for managing observers |
| Concrete Subject | Maintains observers and sends notifications |
| Observer Interface | Defines a common notification method |
| Concrete Observer | Implements the notification behavior |
| Client | Registers and removes observers |

### Architecture

```text
                 Client
                   |
                   v
             YouTubeChannel
             (Concrete Subject)
                   |
                   v
              Subject
              Interface
                   |
            Observer List
                   |
          -------------------
          |        |        |
          v        v        v
       Observer Observer Observer
          A        B        C
          |        |        |
          v        v        v
       Update() Update() Update()
```

## 4. Java Implementation

Let's implement a YouTube notification system.

### Step 1: Create the Observer Interface

```java
public interface Observer {

    void update(String videoName);

}
```

This interface defines a common contract for all subscribers.

Every subscriber must implement the `update()` method.

### Step 2: Create Concrete Observers

```java
public class Subscriber implements Observer {

    private String name;

    public Subscriber(String name) {
        this.name = name;
    }

    @Override
    public void update(String videoName) {

        System.out.println(
            name + " received notification: " +
            videoName + " uploaded!"
        );
    }
}
```

Each subscriber implements its own notification behavior.

We can create multiple subscriber objects using this class.

### Step 3: Create the Subject Interface

```java
public interface Subject {

    void subscribe(Observer observer);

    void unsubscribe(Observer observer);

    void notifyObservers(String videoName);

}
```

The Subject interface defines three important methods.

| Method | Responsibility |
|---|---|
| `subscribe()` | Registers a new observer |
| `unsubscribe()` | Removes an observer |
| `notifyObservers()` | Notifies all registered observers |

### Step 4: Create the Concrete Subject

```java
import java.util.ArrayList;
import java.util.List;

public class YouTubeChannel implements Subject {

    private List<Observer> subscribers =
            new ArrayList<>();

    @Override
    public void subscribe(Observer observer) {

        subscribers.add(observer);
    }

    @Override
    public void unsubscribe(Observer observer) {

        subscribers.remove(observer);
    }

    @Override
    public void notifyObservers(String videoName) {

        for (Observer observer : subscribers) {

            observer.update(videoName);
        }
    }

    public void uploadVideo(String videoName) {

        System.out.println(
            "New video uploaded: " + videoName
        );

        notifyObservers(videoName);
    }
}
```

**Important design decision:**

Notice that we maintain:

```java
List<Observer> subscribers;
```

Instead of:

```java
List<Subscriber> subscribers;
```

Why?

Because the Subject should depend on the Observer interface rather than a specific implementation.

This allows us to add different types of observers without modifying the Subject.

### Step 5: Client Code

```java
public class Main {

    public static void main(String[] args) {

        YouTubeChannel channel =
            new YouTubeChannel();

        Observer user1 =
            new Subscriber("Theja");

        Observer user2 =
            new Subscriber("Rahul");

        Observer user3 =
            new Subscriber("Priya");

        // Subscribe users

        channel.subscribe(user1);
        channel.subscribe(user2);
        channel.subscribe(user3);

        // Upload a video

        channel.uploadVideo("Java Design Patterns");

        // Rahul unsubscribes

        channel.unsubscribe(user2);

        // Upload another video

        channel.uploadVideo("Spring Boot Tutorial");
    }
}
```

### Output

```text
New video uploaded: Java Design Patterns

Theja received notification: Java Design Patterns uploaded!
Rahul received notification: Java Design Patterns uploaded!
Priya received notification: Java Design Patterns uploaded!

New video uploaded: Spring Boot Tutorial

Theja received notification: Spring Boot Tutorial uploaded!
Priya received notification: Spring Boot Tutorial uploaded!
```

## 5. How Does It Work Internally?

1. The client creates a Concrete Subject.
2. Subscribers register themselves using `subscribe()`.
3. The Subject maintains a list of registered observers.
4. An event occurs, such as uploading a new video.
5. The Subject calls `notifyObservers()`.
6. The notification method iterates through the observer list.
7. Each observer's `update()` method gets executed.

### Important line

```java
observer.update(videoName);
```

Here, `observer` is an interface reference.

Java runtime polymorphism executes the `update()` implementation of the actual observer object.

**Remember:** The Subject knows how to notify observers, but it does not need to know how each observer processes the notification.

## 6. Adding a New Observer

Suppose we want to introduce email notifications.

We don't need to modify the existing `YouTubeChannel` class.

We simply create another implementation.

```java
public class EmailSubscriber implements Observer {

    private String email;

    public EmailSubscriber(String email) {
        this.email = email;
    }

    @Override
    public void update(String videoName) {

        System.out.println(
            "Email sent to " + email +
            ": New video - " + videoName
        );
    }
}
```

Register it:

```java
channel.subscribe(
    new EmailSubscriber("user@gmail.com")
);
```

The new observer automatically starts receiving notifications.

This demonstrates the **Open/Closed Principle**.

We can extend the system with new observer implementations without modifying the existing Subject.

## 7. Observer Pattern in Spring Boot

Spring provides an event-publishing mechanism that follows the same general publish-subscribe concept.

Consider an e-commerce application.

Whenever an order is placed:

- Email service sends a confirmation.
- Inventory service updates stock.
- Other components can react to the same event.

### Step 1: Create an Event

```java
public record OrderPlacedEvent(Long orderId) {}
```

### Step 2: Publish the Event

```java
@Service
public class OrderService {

    private final ApplicationEventPublisher publisher;

    public OrderService(
        ApplicationEventPublisher publisher) {

        this.publisher = publisher;
    }

    public void placeOrder(Long orderId) {

        // Save order logic

        publisher.publishEvent(
            new OrderPlacedEvent(orderId)
        );
    }
}
```

### Step 3: Create Event Listeners

Email notification listener:

```java
@Component
public class EmailNotificationListener {

    @EventListener
    public void handle(OrderPlacedEvent event) {

        System.out.println(
            "Sending confirmation email for order: "
            + event.orderId()
        );
    }
}
```

Inventory listener:

```java
@Component
public class InventoryListener {

    @EventListener
    public void handle(OrderPlacedEvent event) {

        System.out.println(
            "Updating inventory for order: "
            + event.orderId()
        );
    }
}
```

### Execution Flow

```text
OrderService
     |
     | publishEvent()
     v
ApplicationEventPublisher
     |
     |----------------------|
     v                      v
Email Listener       Inventory Listener
     |                      |
     v                      v
Send Email             Update Stock
```

Spring manages event publishing and listener registration.

**Production note:**

- Spring application events are synchronous by default.
- `@Async` can be used for asynchronous listener execution with appropriate configuration.
- For communication across independent microservices, use a message broker such as Kafka or RabbitMQ.
- For reliable event delivery, consider transactional event publishing and the Outbox Pattern.

## 8. Advantages

1. **Loose Coupling:** Subject and observers communicate through interfaces.
2. **Open/Closed Principle:** New observers can be introduced without modifying the Subject.
3. **Dynamic Registration:** Observers can subscribe and unsubscribe at runtime.
4. **Reusability:** Different observers can react to the same event.
5. **Event-Driven Architecture:** Supports event-based communication between components.

## 9. Disadvantages

1. Too many observers can make the notification process complicated.
2. Notification execution order may not be predictable unless explicitly controlled.
3. Synchronous notification can slow down the Subject.
4. Observers that are not unsubscribed may cause memory leaks.
5. Debugging can become difficult when many components react to the same event.

## 10. Observer Pattern vs Strategy Pattern

Both are behavioral patterns, but they solve different problems.

| Strategy Pattern | Observer Pattern |
|---|---|
| Encapsulates interchangeable algorithms | Establishes notification relationships |
| Focuses on how an operation is performed | Focuses on how objects communicate |
| Usually selects a strategy for an operation | Notifies multiple registered observers |
| Example: Payment methods | Example: YouTube subscribers |

## 11. Real-World Use Cases

| Application | Subject | Observers |
|---|---|---|
| YouTube | Channel | Subscribers |
| Stock market | Stock price publisher | Trading applications |
| E-commerce | Order | Email, SMS, inventory services |
| Weather application | Weather station | Mobile apps and dashboards |
| UI applications | Button | Event listeners |
| Spring Boot | Application event publisher | Event listeners |

## 12. Interview Questions

**Q1. What is the Observer Design Pattern?**

It is a behavioral design pattern that establishes a one-to-many dependency between objects. Whenever the Subject changes its state, all registered observers are automatically notified.

**Q2. What are Subject and Observer?**

- Subject: Maintains observers and notifies them.
- Observer: Receives notifications from the Subject.

**Q3. Why does the Subject maintain a list of Observer interfaces instead of concrete classes?**

To reduce coupling and allow different observer implementations to be registered without modifying the Subject.

**Q4. Can observers subscribe and unsubscribe dynamically?**

Yes. The Subject provides registration and removal methods.

**Q5. Which SOLID principle does Observer Pattern support?**

Primarily the Open/Closed Principle and Dependency Inversion Principle.

**Q6. What is the difference between synchronous and asynchronous notification?**

- Synchronous: Subject waits for each observer's notification processing to finish.
- Asynchronous: Notifications are processed independently, without making the publisher wait for all processing to finish.

**Q7. Is Spring's ApplicationEventPublisher an implementation of the Observer Pattern?**

It follows the same general event-publishing and subscriber-listener concept, with Spring managing event dispatching and listener registration.

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Category | Behavioral Pattern |
| Main purpose | One-to-many notification |
| Publisher | Subject |
| Subscriber | Observer |
| Registration | `subscribe()` |
| Removal | `unsubscribe()` |
| Notification | `notifyObservers()` |
| Observer callback | `update()` |
| Key concept | Loose Coupling |
| SOLID principles | OCP and DIP |
| Common application | Event-driven systems |

### One-line Definition

The Observer Pattern defines a one-to-many dependency between objects so that when one object changes its state, all its registered dependent objects are automatically notified.

### Most Important Things to Remember

1. Subject maintains a list of observers.
2. Observers implement a common interface.
3. Subject notifies observers when an event or state change occurs.
4. Observers can be registered and removed dynamically.
5. Subject depends on an abstraction, not concrete observer implementations.
