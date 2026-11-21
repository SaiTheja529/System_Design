# Strategy Design Pattern – Java Notes

## 1. Introduction

The **Strategy Design Pattern** is a **Behavioral Design Pattern** that allows us to define a family of algorithms, encapsulate each algorithm in a separate class, and make them interchangeable at runtime.

In simple words:

Instead of writing multiple algorithms inside a single class using `if-else` or `switch` statements, we create separate classes for each behavior and select the required behavior dynamically.

**Key idea:** Encapsulate what varies and make it interchangeable.

## 2. Problem Without Strategy Pattern

Consider an e-commerce application that supports multiple payment methods:

- Credit Card
- UPI
- PayPal

A traditional implementation might look like this:

```java
public class PaymentService {

    public void processPayment(String type, double amount) {

        if (type.equals("UPI")) {
            System.out.println("Processing UPI payment");
        }
        else if (type.equals("CARD")) {
            System.out.println("Processing Card payment");
        }
        else if (type.equals("PAYPAL")) {
            System.out.println("Processing PayPal payment");
        }
    }
}
```

### Problems

1. Every time a new payment method is introduced, we modify the existing class.
2. The class becomes difficult to maintain as conditions increase.
3. Payment logic is tightly coupled to the service.
4. Unit testing individual payment algorithms becomes harder.
5. It violates the Open/Closed Principle.

## 3. Solution – Strategy Pattern

We separate each payment algorithm into its own class.

### Main components

| Component | Responsibility |
|---|---|
| Strategy Interface | Defines a common algorithm contract |
| Concrete Strategies | Implement individual algorithms |
| Context | Maintains a strategy reference and delegates execution |
| Client | Selects and provides the required strategy |

### Architecture

```text
                  Client
                    |
                    v
              PaymentService
                 (Context)
                    |
                    v
             PaymentStrategy
                (Interface)
                    |
          ---------------------
          |         |         |
          v         v         v
       Credit     UPI      PayPal
       Card     Payment    Payment
       Payment   Strategy   Strategy
```

## 4. Java Implementation

### Step 1: Create the Strategy Interface

```java
public interface PaymentStrategy {

    void pay(double amount);

}
```

This interface defines a common contract that all payment strategies must follow.

### Step 2: Create Concrete Strategies

**CreditCardPayment.java**

```java
public class CreditCardPayment implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid " + amount + " using Credit Card"
        );
    }
}
```

**UPIPayment.java**

```java
public class UPIPayment implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid " + amount + " using UPI"
        );
    }
}
```

**PayPalPayment.java**

```java
public class PayPalPayment implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println(
            "Paid " + amount + " using PayPal"
        );
    }
}
```

Each class provides its own implementation of the `pay()` method.

### Step 3: Create the Context Class

```java
public class PaymentService {

    private PaymentStrategy paymentStrategy;

    public PaymentService(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void processPayment(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

**Important:** The `PaymentService` does not know how individual payment methods work. It only knows about the `PaymentStrategy` interface.

It delegates the actual payment operation to the selected strategy.

### Step 4: Client Code

```java
public class Main {

    public static void main(String[] args) {

        PaymentStrategy strategy =
            new UPIPayment();

        PaymentService service =
            new PaymentService(strategy);

        service.processPayment(1000);

    }
}
```

Output:

```text
Paid 1000.0 using UPI
```

We can change the strategy without modifying `PaymentService`:

```java
PaymentStrategy strategy =
    new CreditCardPayment();

PaymentService service =
    new PaymentService(strategy);

service.processPayment(2000);
```

## 5. How Does It Work Internally?

1. The client creates the required concrete strategy.
2. The client passes that strategy to the Context (`PaymentService`).
3. The Context stores the strategy through an interface reference.
4. When `processPayment()` is called, the Context invokes `paymentStrategy.pay()`.
5. Java runtime polymorphism executes the implementation belonging to the actual strategy object.

**Remember:** The Context delegates behavior; the concrete strategy performs the actual work.

## 6. Runtime Strategy Switching

We can also provide a setter to change the strategy dynamically.

```java
public class PaymentService {

    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(
            PaymentStrategy paymentStrategy) {

        this.paymentStrategy = paymentStrategy;
    }

    public void processPayment(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

Usage:

```java
PaymentService service = new PaymentService();

service.setPaymentStrategy(new UPIPayment());
service.processPayment(1000);

service.setPaymentStrategy(new CreditCardPayment());
service.processPayment(2000);
```

This demonstrates that the behavior can change at runtime.

In production, ensure a valid strategy is always configured before calling `processPayment()`.

## 7. Strategy Pattern in Spring Boot

Spring can manage concrete strategies as beans.

Example:

```java
public interface DiscountStrategy {
    double calculateDiscount(double amount);
}
```

Concrete implementations:

```java
@Component("regular")
public class RegularDiscount implements DiscountStrategy {

    public double calculateDiscount(double amount) {
        return amount * 0.05;
    }
}
```

```java
@Component("premium")
public class PremiumDiscount implements DiscountStrategy {

    public double calculateDiscount(double amount) {
        return amount * 0.15;
    }
}
```

Context/service:

```java
@Service
public class DiscountService {

    private final Map<String, DiscountStrategy> strategies;

    public DiscountService(
        Map<String, DiscountStrategy> strategies) {
        this.strategies = strategies;
    }

    public double calculate(
        String customerType, double amount) {

        DiscountStrategy strategy =
            strategies.get(customerType);

        if (strategy == null) {
            throw new IllegalArgumentException(
                "Invalid customer type"
            );
        }

        return strategy.calculateDiscount(amount);
    }
}
```

Spring automatically injects the available `DiscountStrategy` beans into the map, using their bean names as keys.

For real financial applications, prefer `BigDecimal` instead of `double` for monetary calculations.

## 8. Advantages

1. **Open/Closed Principle:** Add new strategies without modifying existing strategy implementations or the Context.
2. **Loose Coupling:** The Context depends on an interface rather than concrete classes.
3. **Maintainability:** Each algorithm has its own class.
4. **Reusability:** Strategies can be reused in different parts of the application.
5. **Testability:** Individual strategies can be unit-tested independently.
6. **Runtime Flexibility:** Behavior can be changed by supplying a different strategy.

## 9. Disadvantages

1. Increases the number of classes in the application.
2. The client or another component must decide which strategy to use.
3. May introduce unnecessary complexity when there are only a few simple algorithms that rarely change.

## 10. Strategy Pattern vs Factory Pattern

These patterns are often used together, but their responsibilities are different.

| Factory Pattern | Strategy Pattern |
|---|---|
| Focuses on object creation | Focuses on interchangeable behavior |
| Creates or selects an object | Executes a selected algorithm |
| Hides object creation logic | Encapsulates algorithm implementation |
| Example: Creating a payment object | Example: Executing a payment algorithm |

**Can they be used together?**

Yes!

A Factory can select and create the appropriate payment strategy, and the Strategy Pattern can execute the payment behavior.

## 11. Real-World Use Cases

- Payment processing: UPI, Credit Card, PayPal.
- Discount calculation: Regular, Premium, Festival.
- Route calculation: Fastest, Shortest, Cheapest.
- Authentication: OTP, Password, Biometric.
- File compression: ZIP, GZIP, RAR.
- Sorting: Different sorting algorithms.

## 12. Interview Questions

**Q1. What is the Strategy Design Pattern?**

A behavioral design pattern that encapsulates a family of algorithms into separate classes and makes them interchangeable.

**Q2. Why is Strategy Pattern preferred over multiple if-else conditions?**

It separates algorithms, reduces coupling, improves maintainability, and allows new strategies to be introduced without modifying the Context.

**Q3. What is the role of the Context class?**

It maintains a reference to a strategy and delegates the execution of the algorithm to it.

**Q4. Can we change a strategy at runtime?**

Yes. We can pass a different strategy object or expose a setter to replace the current strategy.

**Q5. Does Strategy Pattern completely eliminate if-else conditions?**

No. We may still need conditional logic or a Factory to select the appropriate strategy. The pattern primarily removes algorithm-specific conditional logic from the Context.

**Q6. Which SOLID principle does Strategy Pattern support?**

Primarily the Open/Closed Principle and Dependency Inversion Principle.

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Category | Behavioral Pattern |
| Main purpose | Encapsulate interchangeable algorithms |
| Main interface | Strategy |
| Implementations | Concrete Strategies |
| Delegating class | Context |
| Selection | Client or Factory |
| Key concept | Runtime Polymorphism |
| SOLID principles | OCP and DIP |

**One-line definition to remember:**

Strategy Pattern defines a family of algorithms, encapsulates each one separately, and allows them to be interchangeable without changing the client that uses them.
