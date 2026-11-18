# Creational Design Patterns – Complete Revision Notes

**Topics:** Factory, Abstract Factory, Builder and Prototype  
**Language:** Java  
**Purpose:** Quick revision, interview preparation and practical understanding.

---

# 1. Introduction to Creational Design Patterns

Creational Design Patterns deal with **how objects are created in an application**.

Instead of creating objects randomly using `new`, these patterns provide structured approaches to object creation.

We have covered four patterns:

| Pattern | Main Responsibility |
|---|---|
| Factory | Create an object based on a requirement |
| Abstract Factory | Create families of related objects |
| Builder | Construct complex objects step by step |
| Prototype | Create objects by copying existing objects |

# 2. Factory Design Pattern

### What is it?

Factory Pattern delegates object creation to a separate factory class.

The client requests an object without knowing how that object is created.

### Real-world scenario

Imagine a notification application supporting:

- Email
- SMS
- WhatsApp

Instead of creating notification objects inside the service, we delegate creation to `NotificationFactory`.

### Architecture

```text
       NotificationService
               |
               v
       NotificationFactory
               |
        Select implementation
               |
       ---------------------
       |         |         |
     Email      SMS     WhatsApp
```

### Java implementation

**1. Common interface**

```java
public interface Notification {
    void send(String message);
}
```

**2. Implementations**

```java
public class EmailNotification implements Notification {

    public void send(String message) {
        System.out.println("Sending Email");
    }
}
```

```java
public class SmsNotification implements Notification {

    public void send(String message) {
        System.out.println("Sending SMS");
    }
}
```

**3. Factory class**

```java
public class NotificationFactory {

    public static Notification getNotification(String type) {

        if (type.equalsIgnoreCase("EMAIL")) {
            return new EmailNotification();
        }

        if (type.equalsIgnoreCase("SMS")) {
            return new SmsNotification();
        }

        throw new IllegalArgumentException("Invalid type");
    }
}
```

**4. Client**

```java
Notification notification =
    NotificationFactory.getNotification("EMAIL");

notification.send("Hello!");
```

### Important points

- Separates object creation from business logic.
- Reduces direct dependencies on concrete implementations.
- Centralizes object creation logic.
- Client works with an interface instead of concrete classes.

### When should you use it?

When you have multiple implementations of an interface and need to select one based on a condition.

**Remember:** Factory = Which object should I create?

---

# 3. Abstract Factory Design Pattern

### What is it?

Abstract Factory provides an interface for creating families of related objects without exposing their concrete implementations.

### Real-world scenario

Imagine an application supporting Windows and Mac operating systems.

Each operating system has:

- Button
- Checkbox

You want Windows users to receive Windows components and Mac users to receive Mac components.

### Architecture

```text
             UIFactory
                 |
        -------------------
        |                 |
   WindowsFactory      MacFactory
        |                 |
   ------------       ------------
   |          |       |          |
 Windows   Windows   Mac        Mac
 Button    Checkbox  Button    Checkbox
```

### Java implementation

**1. Product interfaces**

```java
public interface Button {
    void render();
}

public interface Checkbox {
    void render();
}
```

**2. Concrete implementations**

```java
public class WindowsButton implements Button {
    public void render() {
        System.out.println("Windows Button");
    }
}

public class MacButton implements Button {
    public void render() {
        System.out.println("Mac Button");
    }
}
```

```java
public class WindowsCheckbox implements Checkbox {
    public void render() {
        System.out.println("Windows Checkbox");
    }
}

public class MacCheckbox implements Checkbox {
    public void render() {
        System.out.println("Mac Checkbox");
    }
}
```

**3. Abstract Factory**

```java
public interface UIFactory {

    Button createButton();

    Checkbox createCheckbox();
}
```

**4. Concrete factories**

```java
public class WindowsFactory implements UIFactory {

    public Button createButton() {
        return new WindowsButton();
    }

    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}
```

```java
public class MacFactory implements UIFactory {

    public Button createButton() {
        return new MacButton();
    }

    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}
```

**5. Client**

```java
UIFactory factory = new WindowsFactory();

Button button = factory.createButton();
Checkbox checkbox = factory.createCheckbox();

button.render();
checkbox.render();
```

To switch to Mac, simply change:

```java
UIFactory factory = new MacFactory();
```

The client code remains unchanged.

### Important points

- Creates families of related objects.
- Uses multiple product interfaces.
- Uses an abstract factory interface.
- Concrete factories provide compatible implementations.
- Helps maintain consistency between related objects.

### When should you use it?

When your application supports multiple product families, such as AWS/Azure implementations or Windows/Mac UI components.

**Remember:** Abstract Factory = Which family of objects should I create?

---

# 4. Builder Design Pattern

### What is it?

Builder Pattern constructs a complex object step by step.

It avoids constructors containing too many parameters, especially when many fields are optional.

### Real-world scenario

Imagine an airline booking application.

A booking object contains:

- Passenger name
- Source
- Destination
- Travel date
- Seat number
- Meal preference
- Extra baggage
- Insurance

Passing all these values through one constructor makes the code difficult to understand.

### Without Builder

```java
FlightBooking booking = new FlightBooking(
    "Theja",
    "Bangalore",
    "Delhi",
    "2026-12-20",
    "A12",
    "VEG",
    true,
    false
);
```

### With Builder

```java
FlightBooking booking = new FlightBooking.Builder()
    .passengerName("Theja")
    .source("Bangalore")
    .destination("Delhi")
    .travelDate("2026-12-20")
    .seatNumber("A12")
    .mealPreference("VEG")
    .extraBaggage(true)
    .insurance(false)
    .build();
```

### Architecture

```text
      Client
        |
        v
  Create Builder
        |
        v
  Set properties
        |
        v
     build()
        |
        v
 FlightBooking Object
```

### Important Java implementation concepts

**1. Private constructor**

```java
private FlightBooking(Builder builder) {
    this.passengerName = builder.passengerName;
}
```

Prevents direct object creation through that constructor.

**2. Static nested Builder class**

```java
public static class Builder {

    private String passengerName;

    public Builder passengerName(String name) {
        this.passengerName = name;
        return this;
    }
}
```

**3. build() method**

```java
public FlightBooking build() {
    return new FlightBooking(this);
}
```

**4. Method chaining**

```java
builder.source("Bangalore")
       .destination("Delhi")
       .seatNumber("A12");
```

Every method returns `this`, allowing multiple methods to be chained.

### Important points

- Avoids telescoping constructors.
- Makes object creation readable.
- Supports optional parameters.
- Allows validation before object creation.
- Can help create immutable objects.

### When should you use it?

When an object has many fields, especially optional fields, and requires a controlled construction process.

**Remember:** Builder = How should I construct this complex object?

---

# 5. Prototype Design Pattern

### What is it?

Prototype Pattern creates new objects by copying an existing object.

The existing object acts as a template.

### Real-world scenario

Imagine a game containing hundreds of similar enemy characters.

Instead of initializing every enemy from scratch, you can clone an existing enemy template and customize its properties.

### Architecture

```text
       Prototype Interface
               |
               v
       Concrete Prototype
               |
               v
          Original Object
               |
         clone() method
               |
       -----------------
       |               |
     Copy 1          Copy 2
```

### Java implementation

**1. Prototype interface**

```java
public interface Prototype<T> {
    T clone();
}
```

**2. Concrete prototype**

```java
public class FlightBooking
        implements Prototype<FlightBooking> {

    private String passengerName;
    private String source;
    private String destination;

    public FlightBooking(
            String passengerName,
            String source,
            String destination) {

        this.passengerName = passengerName;
        this.source = source;
        this.destination = destination;
    }

    @Override
    public FlightBooking clone() {

        return new FlightBooking(
            this.passengerName,
            this.source,
            this.destination
        );
    }
}
```

**3. Client**

```java
FlightBooking original =
    new FlightBooking("Theja", "Bangalore", "Delhi");

FlightBooking copy1 = original.clone();

FlightBooking copy2 = original.clone();
```

Now we have three independent outer objects.

### Shallow Copy vs Deep Copy

This is a very important Prototype concept.

**Shallow Copy**

- Creates a new outer object.
- Copies field values.
- Nested object references are shared.

Example:

```text
Original Booking ----\
                      ---> PassengerDetails
Cloned Booking ------/
```

Both objects point to the same nested object.

**Deep Copy**

- Creates a new outer object.
- Creates independent copies of nested mutable objects.

Example:

```text
Original Booking ---> PassengerDetails A

Cloned Booking ----> PassengerDetails B
```

Changes to one nested object do not affect the other.

### Important points

- Uses an existing object as a template.
- Avoids repeatedly reconstructing similar objects.
- Useful when object initialization is expensive.
- Requires careful handling of nested mutable objects.
- Java's `Object.clone()` performs a shallow copy by default.

A copy constructor or an explicit copy method is often easier to maintain than Java's `Cloneable` mechanism.

### When should you use it?

When you need many similar objects based on an existing object and copying is more convenient than constructing them from scratch.

**Remember:** Prototype = Can I create a new object by copying an existing one?

---

# 6. Comparison of all four patterns

| Feature | Factory | Abstract Factory | Builder | Prototype |
|---|---|---|---|---|
| Category | Creational | Creational | Creational | Creational |
| Main purpose | Create an object | Create object families | Step-by-step construction | Copy existing object |
| Main mechanism | Factory method | Factory interface | Builder methods | Clone/copy |
| Object creation | Centralized | Family-based | Step-by-step | Copy-based |
| Main advantage | Loose coupling | Consistent families | Readability | Reuse existing state |
| Example | Notification | OS UI components | Flight booking | Game characters |

# 7. One real-world scenario connecting all four

Imagine developing an airline booking platform.

| Requirement | Pattern | Example |
|---|---|---|
| Select Email or SMS notification | Factory | NotificationFactory |
| Select AWS or Azure services | Abstract Factory | CloudFactory |
| Construct a booking with optional details | Builder | FlightBooking.Builder |
| Duplicate an existing booking template | Prototype | FlightBooking.clone() |

This is how different design patterns can solve different object-creation problems within the same application.

# 8. Interview Quick Revision

**Q1. What is a creational design pattern?**

A pattern that provides a structured approach to object creation.

**Q2. What is the difference between Factory and Abstract Factory?**

Factory creates an object based on a requirement, whereas Abstract Factory creates families of related objects.

**Q3. What problem does Builder solve?**

It solves the telescoping constructor problem and simplifies the creation of complex objects.

**Q4. Why is Builder useful for immutable objects?**

It allows fields to be initialized during construction while avoiding public setters afterward.

**Q5. What is Prototype?**

It creates new objects by copying existing objects.

**Q6. What is the difference between shallow copy and deep copy?**

Shallow copy shares nested object references, whereas deep copy creates independent copies of nested mutable objects.

**Q7. Does Java's Object.clone() perform deep copying?**

No. It performs shallow copying by default.

**Q8. Can Abstract Factory use Factory Method internally?**

Yes. Concrete factories commonly implement object-creation methods that act like factory methods.

---

# 9. Final Memory Trick

Remember these four questions:

| Pattern | Ask yourself |
|---|---|
| Factory | Which object should I create? |
| Abstract Factory | Which family of objects should I create? |
| Builder | How should I construct this object? |
| Prototype | Can I copy an existing object? |

**The ultimate takeaway:**

All four patterns solve object-creation problems, but they approach object creation differently.

Don't memorize the Java code alone. Understand the problem each pattern solves. When you understand the problem, identifying the appropriate design pattern becomes much easier.
