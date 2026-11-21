# Singleton Design Pattern – Complete Revision Notes

**Category:** Creational Design Pattern  
**Language:** Java  
**Purpose:** Interview preparation, Java development and system design revision.

---

# 1. What is Singleton Design Pattern?

Singleton is a creational design pattern that ensures:

1. A class has only one instance.
2. That instance is accessible through a common access point.

**In simple words:** Instead of creating multiple objects of a class, we ensure that everyone uses the same shared instance.

### Real-world example

Imagine an application with a centralized Configuration Manager.

Instead of every service creating its own configuration manager, all services access the same instance.

### Architecture

```text
       Service A
           |
       Service B
           |
       Service C
           |
           v
   Singleton Instance
```

All services share the same object.

---

# 2. Why do we need Singleton?

Imagine creating multiple instances of a DatabaseConnectionManager.

```java
DatabaseConnectionManager db1 =
    new DatabaseConnectionManager();

DatabaseConnectionManager db2 =
    new DatabaseConnectionManager();
```

Problems:

- Unnecessary object creation.
- Duplicate shared configuration.
- Difficult to manage shared resources.
- Potentially inconsistent application state.

Singleton provides a common shared instance.

**Important:** Singleton is not a replacement for database connection pooling. Production applications generally use connection pools such as HikariCP.

---

# 3. Basic Singleton Implementation

There are three essential steps:

1. Make the constructor private.
2. Create a static instance variable.
3. Provide a public static `getInstance()` method.

### Java implementation

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {
        System.out.println("Object Created");
    }

    public static Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

### Client code

```java
public class Main {

    public static void main(String[] args) {

        Singleton obj1 = Singleton.getInstance();

        Singleton obj2 = Singleton.getInstance();

        Singleton obj3 = Singleton.getInstance();

        System.out.println(obj1 == obj2);
        System.out.println(obj2 == obj3);
    }
}
```

Output:

```text
Object Created
true
true
```

All references point to the same object.

### Important concepts

**Private constructor**

Prevents external classes from directly creating instances using `new`.

**Static instance**

Belongs to the class rather than individual objects.

**Static getInstance()**

Provides a common access point to the shared instance.

**Limitation:** This basic implementation is not thread-safe.

---

# 4. Why is Thread Safety Important?

Imagine two threads accessing `getInstance()` simultaneously.

Both threads might observe that `instance == null`.

As a result, both might create separate objects.

| Thread 1 | Thread 2 |
|---|---|
| Checks instance | |
| Finds null | |
| | Checks instance |
| | Finds null |
| Creates object A | |
| | Creates object B |

Now two instances exist!

We need a thread-safe implementation.

---

# 5. Different Ways to Implement Singleton

## Approach 1: Eager Initialization

The instance is created when the class is initialized.

```java
public class Singleton {

    private static final Singleton INSTANCE =
        new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### How does it work?

- Instance is created during class initialization.
- The instance is available whenever requested.
- Java class initialization guarantees thread safety.

**Advantages:**
- Simple implementation.
- Thread-safe.
- No synchronization overhead during access.

**Disadvantage:**
- Object is created even if it is never used.

**Remember:** Eager = Create the instance early.

---

## Approach 2: Lazy Initialization with synchronized

The object is created only when requested.

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

### How does it work?

- First request creates the object.
- Subsequent requests return the existing object.
- `synchronized` allows only one thread at a time to execute the method.

**Advantages:**
- Lazy initialization.
- Thread-safe.

**Disadvantage:**
- Every call requires synchronization, even after the instance is initialized.

**Remember:** synchronized = Only one thread executes the critical section at a time.

---

## Approach 3: Double-Checked Locking

This approach improves lazy initialization by avoiding synchronization after the object is created.

```java
public class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {

            synchronized (Singleton.class) {

                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
```

### Why do we check instance twice?

**First check:**

```java
if (instance == null)
```

Avoids acquiring the lock if the instance already exists.

**Second check:**

```java
synchronized (Singleton.class) {

    if (instance == null) {
        instance = new Singleton();
    }
}
```

Suppose two threads pass the first check.

- Thread 1 acquires the lock.
- Thread 2 waits.
- Thread 1 creates the object and releases the lock.
- Thread 2 acquires the lock.
- Thread 2 checks again and sees that the instance already exists.

Therefore, another object is not created.

### Why volatile?

```java
private static volatile Singleton instance;
```

`volatile` ensures:

- Visibility of the updated instance reference across threads.
- Appropriate memory-ordering guarantees.
- Prevention of problematic instruction reordering during initialization.

**Important:** Double-checked locking requires `volatile` for correctness in modern Java.

**Remember:** Double-Checked Locking = Two null checks + synchronized + volatile.

---

## Approach 4: Bill Pugh Singleton (Static Inner Class)

A clean and commonly recommended Java implementation.

```java
public class Singleton {

    private Singleton() {
    }

    private static class Holder {

        private static final Singleton INSTANCE =
            new Singleton();
    }

    public static Singleton getInstance() {

        return Holder.INSTANCE;
    }
}
```

### How does it work?

1. The outer Singleton class can load without creating the instance.
2. The inner Holder class is initialized only when `getInstance()` accesses it.
3. Java guarantees thread-safe class initialization.
4. The instance is created only once.

**Advantages:**
- Lazy initialization.
- Thread-safe.
- No explicit synchronization.
- No volatile required.

**Remember:** Bill Pugh = Static inner class + lazy initialization + thread safety.

---

## Approach 5: Enum Singleton

Java provides another simple approach.

```java
public enum Singleton {

    INSTANCE;

    public void showMessage() {
        System.out.println("Singleton Instance");
    }
}
```

Usage:

```java
Singleton.INSTANCE.showMessage();
```

### Why use Enum?

Java enum provides strong protection against several traditional Singleton problems:

- Thread-safe initialization.
- Serialization safety.
- Protection against reflective creation of additional enum instances.

**Advantages:**
- Very simple implementation.
- Thread-safe.
- Serialization-safe.

**Limitation:**
- Must be suitable for an enum-based design.
- Does not provide the same lazy access behavior as the Holder approach.

**Remember:** Enum Singleton = Simple and robust Singleton implementation.

---

# 6. Comparison of All Implementations

| Approach | Lazy Initialization | Thread-safe |
|---|---|---|
| Basic Lazy | Yes | No |
| Eager | No | Yes |
| Synchronized Method | Yes | Yes |
| Double-Checked Locking | Yes | Yes |
| Bill Pugh | Yes | Yes |
| Enum | On enum initialization | Yes |

### Which approach should you use?

- Eager initialization: When the instance is inexpensive and always needed.
- Synchronized method: When simplicity is more important than access overhead.
- Double-checked locking: When you specifically need lazy initialization with optimized access.
- Bill Pugh: A clean general-purpose lazy Singleton in Java.
- Enum: When an enum-based Singleton fits the requirements and serialization/reflection protection is important.

---

# 7. Singleton in Spring Boot

Spring beans have singleton scope by default.

Example:

```java
@Service
public class FlightService {

    public void searchFlights() {
        System.out.println("Searching flights");
    }
}
```

When Spring manages this bean, it normally creates one shared instance per bean definition per application context.

Other components can inject and use that instance.

### Important distinction

Traditional Java Singleton:

- Implemented using a private constructor.
- Uses a static instance.
- Provides a static `getInstance()` method.

Spring Singleton:

- Managed by the Spring IoC container.
- Uses dependency injection.
- One shared bean instance per bean definition per application context.

**Spring Singleton is not exactly the same as traditional Java Singleton.**

Different application contexts can have different instances.

---

# 8. Where is Singleton Used?

Common examples include:

| Use case | Purpose |
|---|---|
| Configuration Manager | Shared application configuration |
| Logger | Common logging component |
| Cache Manager | Shared cache management |
| Spring Service Bean | Shared application service |
| Connection Pool Manager | Shared pool management |

These are examples of potential Singleton use cases, not a requirement to implement every component using a manual Java Singleton.

---

# 9. Important Mistakes to Avoid

### 1. Singleton does not mean one instance across all servers.

Suppose an application runs on three separate JVM servers.

```text
Server 1 -> Singleton Instance A

Server 2 -> Singleton Instance B

Server 3 -> Singleton Instance C
```

Each JVM can have its own Singleton instance.

Traditional Java Singleton does not guarantee one globally shared instance across a distributed system.

### 2. Singleton does not automatically provide thread safety.

Even if only one object exists, multiple threads can modify its mutable fields simultaneously.

You must manage concurrent access to shared mutable state.

### 3. Avoid excessive Singleton usage.

Too many Singletons can introduce:

- Global state.
- Tight coupling.
- Difficult unit testing.
- Concurrency issues.

### 4. Singleton and connection pooling are different.

A connection pool manages multiple reusable database connections. A Singleton may manage access to the pool, but it should not force all database operations through one physical connection.

---

# 10. Interview Questions

**Q1. What is Singleton Design Pattern?**

A creational pattern that ensures only one instance of a class exists and provides a common access point to it.

**Q2. Why should the constructor be private?**

To prevent external classes from creating objects directly.

**Q3. Why do we use static for the instance variable?**

Because the instance belongs to the class rather than an individual object.

**Q4. Why is basic lazy initialization not thread-safe?**

Two threads can simultaneously observe a null instance and create separate objects.

**Q5. What is the purpose of synchronized?**

It ensures mutual exclusion, allowing only one thread at a time to execute the synchronized method or block.

**Q6. What is the purpose of volatile in Double-Checked Locking?**

It ensures visibility and correct memory ordering for the shared instance reference.

**Q7. Why do we check instance twice in Double-Checked Locking?**

The second check prevents another thread from creating a duplicate object after waiting for the lock.

**Q8. What is Bill Pugh Singleton?**

A lazy, thread-safe Singleton implementation using a static nested Holder class.

**Q9. Is Spring's default Singleton scope identical to Java Singleton?**

No. Spring manages one instance per bean definition per application context.

**Q10. Does Singleton guarantee one instance across multiple JVMs?**

No. Each JVM can have its own instance.

---

# 11. Quick Revision – All Five Creational Patterns

| Pattern | Main Responsibility |
|---|---|
| Factory | Select and create an object |
| Abstract Factory | Create families of related objects |
| Builder | Construct objects step by step |
| Prototype | Create objects by copying existing objects |
| Singleton | Ensure one shared instance |

### Memory Trick

- Factory → Selection
- Abstract Factory → Families
- Builder → Construction
- Prototype → Copying
- Singleton → Single instance

---

# 12. Final Takeaway

Singleton is useful when an application needs a shared instance of a class.

The most important Java concepts associated with Singleton are:

1. Private constructors.
2. Static variables and methods.
3. Lazy vs eager initialization.
4. Thread safety.
5. `synchronized` and `volatile`.
6. Double-checked locking.
7. Bill Pugh implementation.
8. Enum Singleton.
9. Spring Singleton scope.

**Recommended implementation to remember:** Bill Pugh Singleton for a clean, lazy, thread-safe Java implementation.

