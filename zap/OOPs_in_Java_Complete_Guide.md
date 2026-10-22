# OOPs in Java — Complete Guide with Real-World Examples

OOPs stands for **Object-Oriented Programming System** (commonly called Object-Oriented Programming, or OOP). It is a programming approach in which we organize code around **objects and classes** rather than writing everything as independent functions.

Java is primarily an object-oriented programming language, and its OOP concepts help us build applications that are reusable, maintainable, extensible, and easier to test.

For a Java developer or Senior SDET, understanding OOP is essential for designing test automation frameworks, API clients, reusable utilities, and maintainable application code.

## 1. The Four Pillars of OOP

### 1. Encapsulation — Protect the data

Bundle data and methods inside a class and control access to the data using access modifiers.

Example: A bank account protects its balance and allows changes only through authorized methods.

### 2. Inheritance — Reuse existing behavior

A child class inherits accessible fields and methods from a parent class.

Example: `Car` and `Bike` inherit common behavior from `Vehicle`.

### 3. Polymorphism — One interface, different behaviors

The same method call can produce different results depending on the object or the arguments.

Example: `pay()` behaves differently for credit cards and UPI payments.

### 4. Abstraction — Hide implementation details

Expose what an object can do while hiding how it performs the operation.

Example: You call `driver.get(url)` without needing to know how the browser communicates internally.

## 2. Class and Object — The Foundation of OOP

Before learning the four pillars, understand these two terms.

- **Class:** A blueprint that defines data (fields) and behavior (methods).
- **Object:** An instance of a class created at runtime.

Real-world analogy: A house design is a class; each house built from that design is an object.

```java
class Car {
    String brand;
    String color;

    void start() {
        System.out.println(brand + " is starting");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car1 = new Car();
        car1.brand = "Toyota";
        car1.color = "Black";

        Car car2 = new Car();
        car2.brand = "Honda";
        car2.color = "White";

        car1.start(); // Toyota is starting
        car2.start(); // Honda is starting
    }
}
```

What happens here?

- `Car` is the class.
- `car1` and `car2` are reference variables.
- `new Car()` creates a new object.
- `brand` and `color` are instance fields.
- `start()` is an instance method.

Both objects follow the same blueprint, but each stores its own state.

## 3. Pillar 1: Encapsulation

**Definition:** Encapsulation is the process of combining data and methods within a class and restricting direct access to the internal data.

Java commonly implements encapsulation using `private` fields and `public` methods.

### Without encapsulation

```java
class BankAccount {
    public double balance;
}

// Anyone can change the balance directly
BankAccount account = new BankAccount();
account.balance = -5000;
```

The account allows an invalid balance because the field is publicly accessible.

### With encapsulation

```java
class BankAccount {
    private double balance;

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();

        account.deposit(1000);
        account.deposit(-500);

        System.out.println(account.getBalance()); // 1000.0
    }
}
```

Here, the `balance` field cannot be accessed directly outside the class. The `deposit()` method controls how the balance changes.

**Important concepts:**

- `private`: Accessible only within the declaring class.
- `public`: Accessible from anywhere the class is accessible.
- `protected`: Accessible within the same package and by subclasses, subject to Java's access rules.
- Default access (no modifier): Accessible within the same package.

### Where is encapsulation useful in test automation?

```java
class LoginPage {
    private String usernameField = "#username";
    private String passwordField = "#password";
    private String loginButton = "#login";

    public void login(String username, String password) {
        // Interact with these locators through the page automation API
    }
}
```

The test calls `login()` without needing to manipulate the page's internal locator details directly. In a real framework, the fields would be used by Playwright, Selenium, or another browser automation library.

## 4. Pillar 2: Inheritance

**Definition:** Inheritance allows one class to acquire accessible fields and methods from another class.

Java uses the `extends` keyword for class inheritance.

```text
        Vehicle
       /       \
     Car       Bike
```

### Java example

```java
class Vehicle {
    void start() {
        System.out.println("Vehicle started");
    }

    void stop() {
        System.out.println("Vehicle stopped");
    }
}

class Car extends Vehicle {
    void drive() {
        System.out.println("Car is driving");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();

        car.start(); // Inherited from Vehicle
        car.drive(); // Defined in Car
        car.stop();  // Inherited from Vehicle
    }
}
```

The `Car` class reuses the behavior defined in `Vehicle` instead of duplicating it.

### Types of inheritance in Java

| Type | Meaning | Supported with classes? |
|---|---|---|
| Single | One child, one parent | Yes |
| Multilevel | A class extends a class that extends another class | Yes |
| Hierarchical | Multiple child classes share one parent | Yes |
| Multiple | One child inherits from multiple parents | No, not with classes |

Java supports multiple inheritance of *type* through interfaces. A class can implement multiple interfaces.

### Test automation example

```java
class BaseTest {
    void initialize() {
        System.out.println("Browser setup");
    }

    void cleanup() {
        System.out.println("Browser cleanup");
    }
}

class LoginTest extends BaseTest {
    void testLogin() {
        System.out.println("Executing login test");
    }
}
```

`LoginTest` inherits `initialize()` and `cleanup()` from `BaseTest`.

**Senior SDET tip:** Inheritance is useful for genuinely shared behavior, but avoid creating deep inheritance hierarchies. In test frameworks, composition and fixtures are often more flexible than relying on a large `BaseTest` class.

## 5. Pillar 3: Polymorphism

**Definition:** Polymorphism means "many forms." The same method name or contract can represent different behaviors.

Java has two commonly discussed forms:

- Compile-time polymorphism — method overloading.
- Runtime polymorphism — method overriding.

### A. Method overloading

Multiple methods share the same name but have different parameter lists.

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }

    double add(double a, double b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();

        System.out.println(calc.add(10, 20));       // 30
        System.out.println(calc.add(10, 20, 30));   // 60
        System.out.println(calc.add(1.5, 2.5));     // 4.0
    }
}
```

The compiler selects the appropriate overload based on the arguments and method resolution rules.

**Rule:** Changing only the return type is not enough to overload a method.

### B. Method overriding

A child class provides its own implementation of an inherited instance method.

```java
class Payment {
    void pay() {
        System.out.println("Generic payment");
    }
}

class CreditCardPayment extends Payment {
    @Override
    void pay() {
        System.out.println("Paid using Credit Card");
    }
}

class UpiPayment extends Payment {
    @Override
    void pay() {
        System.out.println("Paid using UPI");
    }
}

public class Main {
    public static void main(String[] args) {
        Payment p1 = new CreditCardPayment();
        Payment p2 = new UpiPayment();

        p1.pay(); // Paid using Credit Card
        p2.pay(); // Paid using UPI
    }
}
```

The variable type is `Payment` in both cases, but the actual object's class determines which overridden `pay()` method runs.

This is **runtime polymorphism**.

### Why does this matter in an automation framework?

Imagine supporting different authentication providers.

```java
interface AuthProvider {
    void login();
}

class OktaAuth implements AuthProvider {
    public void login() {
        System.out.println("Login through Okta");
    }
}

class KeycloakAuth implements AuthProvider {
    public void login() {
        System.out.println("Login through Keycloak");
    }
}

class LoginService {
    void authenticate(AuthProvider provider) {
        provider.login();
    }
}
```

Now the framework can use the same `authenticate()` method with different providers:

```java
LoginService service = new LoginService();

service.authenticate(new OktaAuth());
service.authenticate(new KeycloakAuth());
```

You can add another provider without changing `LoginService`, provided it implements `AuthProvider`.

This design is particularly useful for provider-agnostic frameworks.

## 6. Pillar 4: Abstraction

**Definition:** Abstraction exposes essential operations while hiding implementation details.

Java supports abstraction mainly through:

1. Abstract classes
2. Interfaces

### A. Abstract class

An abstract class cannot be instantiated directly. It can contain both abstract methods (without an implementation) and concrete methods (with an implementation).

```java
abstract class Browser {
    abstract void launch();

    void close() {
        System.out.println("Browser closed");
    }
}

class ChromeBrowser extends Browser {
    @Override
    void launch() {
        System.out.println("Chrome launched");
    }
}

class FirefoxBrowser extends Browser {
    @Override
    void launch() {
        System.out.println("Firefox launched");
    }
}

public class Main {
    public static void main(String[] args) {
        Browser browser = new ChromeBrowser();
        browser.launch();
        browser.close();
    }
}
```

Output:

```text
Chrome launched
Browser closed
```

The abstract `Browser` class defines what a browser must do, while each concrete browser supplies its launch implementation.

### B. Interface

An interface defines a contract that implementing classes agree to fulfill. Interfaces can also contain `default`, `static`, and (since Java 9) private methods, subject to Java's rules.

```java
interface NotificationService {
    void send(String message);
}

class EmailNotification implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SmsNotification implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

Both implementations satisfy the same contract but perform the operation differently.

### Abstract class vs interface

| Feature | Abstract class | Interface |
|---|---|---|
| Keyword | `abstract class` | `interface` |
| Inheritance | A class can extend only one class | A class can implement multiple interfaces |
| Instance fields | Can have instance state | Fields are implicitly `public static final` constants |
| Constructors | Can have constructors | Cannot have ordinary instance constructors |
| Methods | Abstract and concrete methods | Abstract methods and supported default, static, and private methods |
| Best suited for | Shared state and closely related behavior | Contracts and interchangeable implementations |

**How to decide?**

- Use an **abstract class** when related classes share state or common implementation.
- Use an **interface** when different classes should follow the same contract, even if they are unrelated.

For an enterprise test automation framework, interfaces are often useful for interchangeable authentication providers, reporting adapters, API clients, or other pluggable components.

## 7. Other Important OOP Concepts in Java

The four pillars are only part of the picture. These additional concepts appear frequently in interviews.

### Constructor

A constructor initializes an object when it is created. It has the same name as its class and no return type.

```java
class Employee {
    String name;
    int experience;

    Employee(String name, int experience) {
        this.name = name;
        this.experience = experience;
    }
}

Employee employee = new Employee("Ravi", 10);

System.out.println(employee.name);       // Ravi
System.out.println(employee.experience); // 10
```

`this.name` refers to the object's field, while `name` refers to the constructor parameter.

### `this` vs `super`

| Keyword | Purpose |
|---|---|
| `this` | Refers to the current object; can also invoke another constructor using `this(...)` |
| `super` | Refers to superclass members; can also invoke a superclass constructor using `super(...)` |

```java
class Animal {
    Animal() {
        System.out.println("Animal constructor");
    }
}

class Dog extends Animal {
    Dog() {
        super();
        System.out.println("Dog constructor");
    }
}
```

Creating `new Dog()` prints the parent constructor message first, then the child constructor message.

### Association, aggregation, and composition

These describe relationships between objects.

- **Association — Uses or interacts with:** Example: A `Teacher` teaches a `Student`. Both can exist independently.
- **Aggregation — Has-a, with independent lifecycles:** Example: A `Team` has `Player` objects, but players can exist independently of a particular team.
- **Composition — Strong ownership:** Example: An `Order` owns its `OrderLine` objects in a model where order lines are managed as integral parts of the order.

These relationships are design concepts; Java does not enforce aggregation or composition through a special keyword.

### Static vs instance members

```java
class Employee {
    static String company = "ABC";
    String name;
}
```

- `company` belongs to the class and is shared across instances.
- `name` belongs to each object.

Access a static field with `Employee.company`, rather than relying on an object reference.

### Access modifiers

| Modifier | Visibility |
|---|---|
| `private` | Declaring class only |
| No modifier | Same package |
| `protected` | Same package and permitted subclasses |
| `public` | Wherever the declaring type/member is accessible |

Access modifiers are fundamental to encapsulation and API design.

## 8. How the Four Pillars Work Together

Consider a test automation framework that supports multiple browsers.

- **Browser interface — Abstraction:** Defines the contract `launch()` and `close()`.
- **ChromeBrowser and FirefoxBrowser:** Provide different implementations.
- **Polymorphism:** The framework calls `launch()` through the `Browser` interface, and the correct implementation runs.
- **Encapsulation:** Each browser implementation can hide its internal configuration and setup details.
- **Inheritance:** A shared abstract base class could provide common lifecycle behavior if appropriate.

Here is a complete, simplified Java example combining these concepts:

```java
interface Browser {
    void launch();
    void close();
}

class ChromeBrowser implements Browser {
    @Override
    public void launch() {
        System.out.println("Launching Chrome");
    }

    @Override
    public void close() {
        System.out.println("Closing Chrome");
    }
}

class FirefoxBrowser implements Browser {
    @Override
    public void launch() {
        System.out.println("Launching Firefox");
    }

    @Override
    public void close() {
        System.out.println("Closing Firefox");
    }
}

public class Main {
    public static void main(String[] args) {
        Browser browser = new ChromeBrowser();

        browser.launch();
        browser.close();

        browser = new FirefoxBrowser();

        browser.launch();
        browser.close();
    }
}
```

Output:

```text
Launching Chrome
Closing Chrome
Launching Firefox
Closing Firefox
```

The client code works with the `Browser` contract rather than depending on one specific implementation.

## 9. OOP Interview Questions and Answers

### 1. What is the difference between a class and an object?

A class defines the structure and behavior; an object is a particular instance of that class.

### 2. What is the difference between abstraction and encapsulation?

Abstraction hides implementation complexity and exposes essential operations. Encapsulation bundles state and behavior and controls access to internal data.

### 3. Can Java support multiple inheritance?

A Java class cannot extend multiple classes. It can implement multiple interfaces.

### 4. Can a constructor be overridden?

No. Constructors are not inherited, so they cannot be overridden. A class can have multiple overloaded constructors.

### 5. Can static methods be overridden?

No, not in the runtime-polymorphism sense. A static method in a subclass can hide an accessible static method with a compatible signature.

### 6. What is the difference between overloading and overriding?

Overloading uses the same method name with different parameter lists. Overriding supplies a subclass implementation of an inherited instance method.

### 7. What is runtime polymorphism?

When an overridden instance method is selected based on the actual object's runtime class.

### 8. Can an abstract class have a constructor?

Yes. Its constructor runs as part of constructing a concrete subclass instance.

### 9. Why use an interface instead of directly creating a concrete class?

An interface reduces dependency on a particular implementation and makes it easier to replace implementations and test components independently.

### 10. What is composition over inheritance?

A design preference for building functionality by combining objects rather than extending a class hierarchy whenever possible. It often improves flexibility.

## 10. Your OOP Learning Roadmap

For interview preparation and practical framework design, learn in this order:

1. **Classes, objects, constructors, and methods** — build a small Employee or BankAccount program.
2. **Encapsulation and access modifiers** — protect fields and expose controlled operations.
3. **Inheritance and composition** — understand code reuse and object relationships.
4. **Polymorphism** — implement interchangeable authentication providers or browser implementations.
5. **Abstraction using interfaces and abstract classes** — define contracts for pluggable framework components.
6. **SOLID principles and design patterns** — learn how to apply OOP in larger automation frameworks.

**The key takeaway:** Don't memorize only the definitions. Be able to explain the problem each OOP principle solves, implement a small Java example, and describe when you would use it in a production application or a test automation framework.
