# State Design Pattern – Revision Notes

## 1. What is the State Design Pattern?

The **State Pattern** is a behavioral design pattern that allows an object to change its behavior when its internal state changes.

It makes an object appear as if it has changed its class by delegating behavior to different state objects.

**In simple terms:** The same method can behave differently depending on the object's current state.

### Real-world example: Order Management

Consider an e-commerce order:

- PLACED → Order has been created.
- CONFIRMED → Order has been accepted.
- SHIPPED → Order has been dispatched.
- DELIVERED → Order has reached the customer.
- CANCELLED → Order has been cancelled.

The operations allowed on an order depend on its current state.

For example:
- A PLACED order can be confirmed or cancelled.
- A CONFIRMED order can be shipped or cancelled.
- A SHIPPED order can be delivered.
- A DELIVERED order cannot be cancelled.

This is a good use case for the State Pattern.

---

## 2. Problem Without the State Pattern

We might initially implement the order logic using multiple `if-else` statements.

```java
public void processOrder(String status) {

    if (status.equals("PLACED")) {
        // Confirm order
    } 
    else if (status.equals("CONFIRMED")) {
        // Ship order
    } 
    else if (status.equals("SHIPPED")) {
        // Deliver order
    } 
    else if (status.equals("DELIVERED")) {
        // Order completed
    }
}
```

### Problems

1. Too many conditional statements.
2. Adding new states makes the code harder to maintain.
3. State-specific logic becomes mixed together.
4. Violates the Open/Closed Principle when every new state requires modifying existing conditional logic.
5. Testing individual state behaviors becomes difficult.

**Solution:** Encapsulate each state and its behavior in a separate class.

---

## 3. Structure and Components

The State Pattern consists of four main components:

| Component | Responsibility |
|---|---|
| State Interface | Defines common operations for all states. |
| Concrete States | Implement behavior specific to each state. |
| Context | Maintains the current state and delegates operations to it. |
| Client | Interacts with the Context. |

### Class relationship

```text
             OrderState
             (Interface)
                  |
       -------------------------
       |           |           |
   PlacedState ConfirmedState ShippedState
       |           |           |
       -------------------------
                  |
                Order
               (Context)
                  |
          Current OrderState
```

The `Order` context delegates operations to its current `OrderState`.

---

## 4. Java Implementation

### Step 1: Create the State Interface

```java
public interface OrderState {

    void confirm(Order order);

    void ship(Order order);

    void deliver(Order order);

    void cancel(Order order);
}
```

This interface defines the operations that different order states can handle.

### Step 2: Create Concrete State Classes

**PlacedState**

```java
public class PlacedState implements OrderState {

    @Override
    public void confirm(Order order) {
        System.out.println("Order confirmed");
        order.setState(new ConfirmedState());
    }

    @Override
    public void ship(Order order) {
        System.out.println("Cannot ship an unconfirmed order");
    }

    @Override
    public void deliver(Order order) {
        System.out.println("Cannot deliver an unconfirmed order");
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Order cancelled");
        order.setState(new CancelledState());
    }
}
```

**ConfirmedState**

```java
public class ConfirmedState implements OrderState {

    @Override
    public void confirm(Order order) {
        System.out.println("Order is already confirmed");
    }

    @Override
    public void ship(Order order) {
        System.out.println("Order shipped");
        order.setState(new ShippedState());
    }

    @Override
    public void deliver(Order order) {
        System.out.println("Cannot deliver before shipping");
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Order cancelled");
        order.setState(new CancelledState());
    }
}
```

**ShippedState**

```java
public class ShippedState implements OrderState {

    @Override
    public void confirm(Order order) {
        System.out.println("Order is already confirmed");
    }

    @Override
    public void ship(Order order) {
        System.out.println("Order is already shipped");
    }

    @Override
    public void deliver(Order order) {
        System.out.println("Order delivered");
        order.setState(new DeliveredState());
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Cannot cancel a shipped order");
    }
}
```

**DeliveredState**

```java
public class DeliveredState implements OrderState {

    @Override
    public void confirm(Order order) {
        System.out.println("Order already delivered");
    }

    @Override
    public void ship(Order order) {
        System.out.println("Order already delivered");
    }

    @Override
    public void deliver(Order order) {
        System.out.println("Order already delivered");
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Cannot cancel a delivered order");
    }
}
```

**CancelledState**

```java
public class CancelledState implements OrderState {

    @Override
    public void confirm(Order order) {
        System.out.println("Cancelled order cannot be confirmed");
    }

    @Override
    public void ship(Order order) {
        System.out.println("Cancelled order cannot be shipped");
    }

    @Override
    public void deliver(Order order) {
        System.out.println("Cancelled order cannot be delivered");
    }

    @Override
    public void cancel(Order order) {
        System.out.println("Order is already cancelled");
    }
}
```

### Step 3: Create the Context Class

```java
public class Order {

    private OrderState currentState;

    public Order() {
        this.currentState = new PlacedState();
    }

    public void setState(OrderState state) {
        this.currentState = state;
    }

    public void confirm() {
        currentState.confirm(this);
    }

    public void ship() {
        currentState.ship(this);
    }

    public void deliver() {
        currentState.deliver(this);
    }

    public void cancel() {
        currentState.cancel(this);
    }
}
```

**Important:** The Context does not implement the state-specific business logic. It delegates each operation to the current state.

### Step 4: Client Code

```java
public class Main {

    public static void main(String[] args) {

        Order order = new Order();

        order.confirm();

        order.ship();

        order.deliver();

        order.cancel();
    }
}
```

Output:

```text
Order confirmed
Order shipped
Order delivered
Cannot cancel a delivered order
```

---

## 5. How Does the State Transition Work?

Consider this execution:

```java
Order order = new Order();
```

1. The Order object is created.
2. Its initial state is `PlacedState`.

Next:

```java
order.confirm();
```

3. Order delegates the operation to `PlacedState`.
4. `PlacedState` executes the confirmation logic.
5. It changes the current state to `ConfirmedState`.

Next:

```java
order.ship();
```

6. Order delegates the operation to `ConfirmedState`.
7. `ConfirmedState` executes shipping logic.
8. It changes the current state to `ShippedState`.

This continues until the order reaches its final state.

### Key concept: Runtime Polymorphism

The Context maintains a reference of type `OrderState`.

```java
private OrderState currentState;
```

At runtime, this reference can point to different implementations:

```java
currentState = new PlacedState();

currentState = new ConfirmedState();

currentState = new ShippedState();
```

When we invoke:

```java
currentState.confirm(this);
```

Java executes the implementation belonging to the actual object referenced by `currentState`.

This is runtime polymorphism.

---

## 6. State Pattern in Spring Boot

Imagine a Spring Boot order-management application.

We can create separate state implementations as Spring beans.

```java
public interface OrderState {
    void process(Order order);
}
```

Example:

```java
@Component("placed")
public class PlacedState implements OrderState {

    @Override
    public void process(Order order) {
        System.out.println("Processing placed order");
    }
}
```

```java
@Component("shipped")
public class ShippedState implements OrderState {

    @Override
    public void process(Order order) {
        System.out.println("Processing shipped order");
    }
}
```

A service can inject the available implementations:

```java
@Service
public class OrderService {

    private final Map<String, OrderState> states;

    public OrderService(Map<String, OrderState> states) {
        this.states = states;
    }

    public void processOrder(String status, Order order) {

        OrderState state = states.get(status);

        if (state == null) {
            throw new IllegalArgumentException("Invalid order state");
        }

        state.process(order);
    }
}
```

This is one possible Spring implementation. In a production application, state transitions should also validate allowed transitions, persist the new state, and account for concurrent updates.

---

## 7. Advantages

1. **Single Responsibility Principle:** Each state class handles its own behavior.
2. **Open/Closed Principle:** New states can often be introduced without modifying existing state classes.
3. **Reduced conditional logic:** Avoids large `if-else` or `switch` blocks.
4. **Better maintainability:** State-specific logic is isolated.
5. **Clear transitions:** State changes are explicitly represented.
6. **Testability:** Each state can be tested independently.

## 8. Disadvantages

1. More classes are introduced.
2. Can be excessive for objects with only a few simple states.
3. State transitions can become difficult to manage if the workflow is very complex.
4. State persistence and concurrent transitions require additional design in real-world applications.

---

## 9. State Pattern vs Strategy Pattern

Both are behavioral design patterns and use composition and polymorphism, but their intentions differ.

| Strategy Pattern | State Pattern |
|---|---|
| Encapsulates interchangeable algorithms. | Encapsulates state-specific behavior. |
| Client usually chooses the strategy. | Current state determines the behavior. |
| Focuses on how an operation is performed. | Focuses on how behavior changes with state. |
| Strategy generally does not manage its own lifecycle. | State objects often trigger transitions to other states. |
| Example: Payment methods. | Example: Order lifecycle. |

**Remember:**
- Strategy = Choose an algorithm.
- State = Behavior depends on the current state.

---

## 10. When Should You Use the State Pattern?

Use it when:

- An object has multiple states.
- Its behavior changes depending on its current state.
- Many conditional statements are checking the same state variable.
- State transitions need to be explicitly managed.

### Real-world use cases

- E-commerce order lifecycle.
- ATM machine (Idle, CardInserted, PINVerified, CashDispensing).
- Traffic light (Red, Yellow, Green).
- Media player (Playing, Paused, Stopped).
- Document workflow (Draft, Review, Approved, Published).
- Support ticket lifecycle (Open, Assigned, Resolved, Closed).

---

## 11. Important Interview Questions

**Q1. What is the State Design Pattern?**

It is a behavioral pattern that allows an object to change its behavior when its internal state changes.

**Q2. Why do we use the State Pattern?**

To encapsulate state-specific behavior, reduce conditional logic, and make state transitions easier to maintain.

**Q3. What is the role of the Context?**

The Context maintains the current state and delegates operations to that state.

**Q4. How does the State Pattern use polymorphism?**

The Context holds a reference to the State interface. At runtime, Java invokes the implementation of the concrete state currently referenced.

**Q5. What is the difference between State and Strategy?**

Strategy encapsulates interchangeable algorithms, whereas State encapsulates behavior that changes according to an object's internal state.

**Q6. Can a State object change the Context's state?**

Yes. A concrete state can request a transition by calling a state-changing method on the Context.

**Q7. Is the State Pattern always better than if-else?**

No. For simple objects with only a few states, ordinary conditional logic may be easier to maintain. The State Pattern becomes useful when state-dependent behavior grows in complexity.

---

## 12. Quick Revision

| Concept | Remember |
|---|---|
| Category | Behavioral Design Pattern |
| Main purpose | Change behavior based on internal state |
| State Interface | Defines common operations |
| Concrete State | Implements state-specific behavior |
| Context | Maintains current state |
| Client | Interacts with Context |
| Main concept | Runtime polymorphism |
| Key benefit | Removes complex state-based conditionals |
| Example | E-commerce order lifecycle |

### One-line takeaway

**State Pattern = Encapsulate each state into a separate class and let the Context delegate behavior to its current state.**