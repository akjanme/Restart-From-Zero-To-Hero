# Week 2 — OOP & SOLID in Simple English
### Classes, Objects, Interfaces, Abstract, Inheritance vs Composition, and the 5 SOLID principles

> How to use this: read one section, then say the idea out loud in your own words. If you can explain it to a friend simply, you know it. The goal is not to memorise — it's to be able to talk about it calmly in an interview. Last week was about how data is stored (value vs reference). This week is about how we **organise** code into objects, and how to keep it clean as it grows.

---

## 1. What is OOP, really?

**The simplest way to think about it:**

OOP (Object-Oriented Programming) is just a way of organising your code so it matches how we think about the real world. The real world is full of *things* (objects) that have some **information** about them and can **do** some actions.

- A **dog** has information (name, age, colour) and can do things (bark, eat, sleep).
- A **bank account** has information (balance, owner) and can do things (deposit, withdraw).

OOP says: instead of having loose data floating around and separate functions far away, **keep the data and the actions that belong to it together in one box.** That box is called an **object**.

**Why interviews test this:**
Almost all C#/.NET code is written this way. They want to see that you can model a problem as a few well-named objects with clear responsibilities — not one giant messy file. Good OOP is really about **keeping related things together and unrelated things apart.**

---

## 2. Class vs Object (the blueprint and the house)

**The simplest way to think about it:**

- A **class** is a **blueprint**. It is a drawing of a house. You cannot live in a drawing.
- An **object** is the **actual house** built from that blueprint. You can build many houses from one drawing, and each one is separate.

So you write the class once, then create as many objects as you want from it. Creating an object is called **instantiation** (just a fancy word for "build one from the blueprint").

**See it in code:**

```csharp
// The CLASS — the blueprint (written once)
public class Dog
{
    public string Name;      // information the dog has
    public int Age;

    public void Bark()       // an action the dog can do
    {
        Console.WriteLine($"{Name} says woof!");
    }
}

// OBJECTS — actual dogs built from the blueprint
Dog d1 = new Dog();   // build one house
d1.Name = "Rex";
d1.Age = 3;

Dog d2 = new Dog();   // build a totally separate one
d2.Name = "Bella";

d1.Bark();   // "Rex says woof!"
d2.Bark();   // "Bella says woof!"
```

Remember from Week 1: a class is a **reference type**. So `d1` and `d2` each point to their own object on the heap.

---

## 3. The 4 pillars of OOP (the four big ideas)

People always ask about these four in interviews. Here they are in plain words.

### Pillar 1 — Encapsulation ("hide the messy insides")

**Picture it:** a TV remote. You press "volume up." You do **not** open the remote and touch the circuit board. The complicated stuff is hidden inside; you only get a few safe buttons.

**Definition:** keep an object's data private, and only let the outside world touch it through safe, controlled methods or properties. This stops other code from putting your object into a broken state.

```csharp
public class BankAccount
{
    private decimal _balance;            // hidden — nobody can set it directly

    public decimal Balance => _balance;  // they can READ it

    public void Deposit(decimal amount)  // but can only CHANGE it through safe rules
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        _balance += amount;
    }
}
```

Without encapsulation, anyone could write `account._balance = -9999;` and break everything. With it, the only way in is through `Deposit`, which checks the rules.

**Properties** are C#'s clean way to do this:

```csharp
public string Name { get; set; }          // auto property
public string Id   { get; private set; }  // outside can read, only this class can set
```

### Pillar 2 — Inheritance ("is-a", reuse from a parent)

**Picture it:** a `Cat` **is an** `Animal`. So a cat automatically gets everything an animal has (it can eat and sleep) without rewriting it, and then adds its own special thing (meow).

**Definition:** a child class can inherit the data and behaviour of a parent class, then add to it or change it. The child gets the parent's stuff for free.

```csharp
public class Animal
{
    public string Name { get; set; }
    public void Eat() => Console.WriteLine($"{Name} is eating");
}

public class Cat : Animal      // Cat IS AN Animal
{
    public void Meow() => Console.WriteLine("Meow");
}

var c = new Cat { Name = "Tom" };
c.Eat();   // inherited for free
c.Meow();  // its own
```

**Warning (interviewers love this):** inheritance is easy to overuse. Only use it for a real "is-a" relationship. If it's more of a "has-a" or "uses-a", prefer **composition** (Section 5).

### Pillar 3 — Polymorphism ("many forms" — same call, different behaviour)

**Picture it:** you tell three different animals "make your sound." The dog barks, the cat meows, the cow moos. **One command, different results** depending on what the thing actually is.

**Definition:** you can call the same method on different objects and each one does its own version. In C# you do this with `virtual` (parent says "this can be overridden") and `override` (child says "here's my version").

```csharp
public class Animal
{
    public virtual void Speak() => Console.WriteLine("Some sound");
}

public class Dog : Animal
{
    public override void Speak() => Console.WriteLine("Woof");
}

public class Cat : Animal
{
    public override void Speak() => Console.WriteLine("Meow");
}

// The magic: treat them all as Animal, but each behaves as itself
List<Animal> animals = new() { new Dog(), new Cat() };
foreach (Animal a in animals)
    a.Speak();   // Woof, then Meow — each picks its own version
```

This is the most powerful idea in OOP. It lets you write code that works with a general type (`Animal`) and still gets the correct specific behaviour at run time.

### Pillar 4 — Abstraction ("show only what matters, hide the rest")

**Picture it:** driving a car. You use the steering wheel and pedals. You don't think about how the engine burns fuel. The car gives you a **simple surface** and hides the complex machine.

**Definition:** expose a simple, clear set of things an object can do, and hide all the complicated details behind it. In C# you do this with **interfaces** and **abstract classes** (next sections).

*(Quick note on the difference: encapsulation hides **data** inside one object; abstraction hides **complexity** behind a simple design. They're cousins — related but not the same.)*

---

## 4. Interface vs Abstract class (the two ways to make a "contract")

This is one of the most common C# interview questions. Here is the simple version.

### Interface — "a promise / a contract"

**Picture it:** a power socket on the wall. Any device with the right plug can connect. The socket doesn't care if it's a lamp, a charger, or a TV — it only requires that the device "can plug in." The plug shape is the **contract**.

**Definition:** an interface is a list of things a class **promises to provide**, with no code inside (traditionally). If a class says it implements an interface, it **must** provide all those members. A class can implement **many** interfaces.

```csharp
public interface IPayable
{
    void Pay(decimal amount);   // just the promise, no body
}

public class CreditCard : IPayable
{
    public void Pay(decimal amount) => Console.WriteLine($"Paid {amount} by card");
}

public class PayPal : IPayable
{
    public void Pay(decimal amount) => Console.WriteLine($"Paid {amount} by PayPal");
}

// Now any code can accept "anything payable" and not care which one:
void Checkout(IPayable method, decimal total) => method.Pay(total);
```

### Abstract class — "a half-built parent"

**Picture it:** a half-built house with the foundation and walls already done, but some rooms left for you to finish. You can't live in it as-is (it's not complete), but it gives you a big head start.

**Definition:** an abstract class is a parent you **cannot create directly**. It can have real code (shared by children) **and** some empty `abstract` methods that each child must fill in. A class can inherit only **one** abstract class.

```csharp
public abstract class Shape
{
    public string Name { get; set; }
    public abstract double Area();         // no body — each shape must give its own

    public void Describe()                 // real shared code
        => Console.WriteLine($"{Name} has area {Area()}");
}

public class Circle : Shape
{
    public double Radius { get; set; }
    public override double Area() => Math.PI * Radius * Radius;
}
```

### The cheat-sheet you can say in an interview:

| Question | Interface | Abstract class |
|----------|-----------|----------------|
| Can it have real code? | Mostly no (modern C# allows some default methods, but avoid it) | Yes, plenty |
| Can it have private data/fields? | No | Yes |
| How many can a class use? | **Many** | **Only one** |
| Relationship it models | "**can do** X" (a capability) | "**is a** kind of" X (a family) |
| When to pick it | Unrelated classes share an ability (e.g. `IDisposable`, `IComparable`) | Related classes share code **and** a base identity |

**The one-liner to remember:** *use an interface to describe a capability many different things can have; use an abstract class to share real code among a family of related things.*

---

## 5. Inheritance vs Composition ("is-a" vs "has-a")

This is the most important design lesson of the week.

- **Inheritance = "is-a".** A `Car` **is a** `Vehicle`. The car gets vehicle stuff by being a kind of vehicle.
- **Composition = "has-a".** A `Car` **has an** `Engine`. The car *contains* an engine and uses it, instead of *being* one.

**See the difference in code:**

```csharp
// COMPOSITION — Car HAS AN Engine (it holds one and uses it)
public class Engine
{
    public void Start() => Console.WriteLine("Engine started");
}

public class Car
{
    private readonly Engine _engine = new Engine();   // a part it owns
    public void Drive()
    {
        _engine.Start();   // delegate the work to the part
        Console.WriteLine("Driving");
    }
}
```

**Why interviewers push "favour composition over inheritance":**

Inheritance locks you into one parent forever and drags along everything the parent has — even things you didn't want. Deep inheritance chains (A → B → C → D) become rigid and confusing; a change at the top can secretly break the bottom. This is sometimes called the "fragile base class" problem.

Composition is more flexible: you snap parts together like LEGO. You can swap a part out (especially if the part is an interface), test parts on their own, and you're not forced into one rigid family tree.

**Simple rule:** start with composition. Only reach for inheritance when there's a genuine, stable "is-a" relationship **and** you actually want to reuse/override the parent's behaviour.

This rule leads us straight into SOLID.

---

# The 5 SOLID Principles

SOLID is five guidelines for writing code that is easy to change without breaking. The name is just the first letters: **S O L I D**. Don't memorise definitions — understand the *pain each one removes*.

> The big idea behind all five: **software changes constantly. Write it so that a change in one place doesn't force changes everywhere else.**

---

## S — Single Responsibility Principle (SRP)

**Plain words:** a class should do **one job**, so it has only **one reason to change**.

**Picture it:** a Swiss Army knife that is *also* trying to be your phone, your wallet, and your car keys. When one part needs fixing, you risk breaking all the others. Better to have separate, focused tools.

**The smell:** a class called `UserManager` that validates input, saves to the database, sends emails, AND writes log files. Four jobs = four different reasons it might need to change. A change to email logic could accidentally break saving.

```csharp
// BAD — one class doing everything
public class UserService
{
    public void Register(User u)
    {
        // validate...
        // save to database...
        // send welcome email...
        // write to log file...
    }
}

// GOOD — each class has ONE job
public class UserValidator   { public bool IsValid(User u) { /*...*/ return true; } }
public class UserRepository  { public void Save(User u)    { /* db */ } }
public class EmailSender     { public void SendWelcome(User u) { /* email */ } }

public class UserService
{
    private readonly UserValidator _validator;
    private readonly UserRepository _repo;
    private readonly EmailSender _email;
    // ...constructor sets them...

    public void Register(User u)
    {
        if (!_validator.IsValid(u)) return;
        _repo.Save(u);
        _email.SendWelcome(u);
    }
}
```

Now if email rules change, you only touch `EmailSender`. Everything else is safe.

---

## O — Open/Closed Principle (OCP)

**Plain words:** code should be **open to extension, closed to modification.** You should be able to add new behaviour **without editing existing, working code.**

**Picture it:** a power strip. To add a new device you plug it in — you don't rewire the strip. The strip is "closed" (you don't open it up) but "open" (you can keep adding devices).

**The smell:** a giant `if`/`switch` that you have to edit every single time a new case appears.

```csharp
// BAD — every new shape forces you to edit this method
public double Area(object shape)
{
    if (shape is Circle c)    return Math.PI * c.Radius * c.Radius;
    if (shape is Square s)    return s.Side * s.Side;
    // add a Triangle? you must come back and edit this again...
    return 0;
}

// GOOD — add new shapes WITHOUT touching old code
public abstract class Shape { public abstract double Area(); }
public class Circle   : Shape { public double Radius; public override double Area() => Math.PI * Radius * Radius; }
public class Square   : Shape { public double Side;   public override double Area() => Side * Side; }
public class Triangle : Shape { public double B, H;   public override double Area() => 0.5 * B * H; }  // just add a new class!

double TotalArea(IEnumerable<Shape> shapes) => shapes.Sum(s => s.Area());
```

Adding `Triangle` required **zero changes** to the existing classes or `TotalArea`. That's OCP. Polymorphism (Section 3) is the main tool that makes it possible.

---

## L — Liskov Substitution Principle (LSP)

**Plain words:** anywhere you use a parent type, you should be able to drop in any child **and nothing breaks.** A child must honestly behave like its parent promises.

**Picture it:** you order "a coffee." Whether you get an espresso or a latte, it should still be drinkable coffee. If the "coffee" you're handed is actually a glass of orange juice, the substitution failed — it broke the promise.

**The classic broken example — Square inheriting from Rectangle:**

```csharp
public class Rectangle
{
    public virtual int Width  { get; set; }
    public virtual int Height { get; set; }
    public int Area => Width * Height;
}

public class Square : Rectangle
{
    // A square forces width and height to stay equal — sounds reasonable...
    public override int Width  { set { base.Width = base.Height = value; } }
    public override int Height { set { base.Width = base.Height = value; } }
}

// Code that works for ANY Rectangle:
void Test(Rectangle r)
{
    r.Width = 5;
    r.Height = 4;
    // Anyone reading this expects Area == 20
    Console.WriteLine(r.Area);   // for a Square: prints 16, not 20!  BROKEN
}
```

A `Square` is *not* a behavioural substitute for a `Rectangle`, even though "a square is a rectangle" in maths. It breaks the caller's expectations → LSP violation. The fix is usually to **not** force that inheritance (use composition or a common interface instead).

**The takeaway:** a subclass that overrides a method to throw "not supported", or to ignore inputs, or to behave surprisingly, is violating LSP. Children should strengthen, never betray, the parent's contract.

---

## I — Interface Segregation Principle (ISP)

**Plain words:** don't force a class to implement methods it doesn't need. Prefer **several small interfaces** over one giant one.

**Picture it:** a job description that says you must be a chef, a pilot, AND a surgeon. Almost nobody can fill it. Three separate, focused job descriptions are far more useful.

**The smell:** one fat interface that forces classes to write empty or "throw" methods.

```csharp
// BAD — a fat interface
public interface IWorker
{
    void Work();
    void Eat();
}

public class Robot : IWorker
{
    public void Work() { /* fine */ }
    public void Eat()  { throw new NotSupportedException(); }  // robots don't eat! red flag
}

// GOOD — split into focused interfaces
public interface IWorkable { void Work(); }
public interface IFeedable { void Eat();  }

public class Human : IWorkable, IFeedable { public void Work(){} public void Eat(){} }
public class Robot : IWorkable             { public void Work(){} }   // only takes what it needs
```

Real .NET examples done right: `IDisposable` (just `Dispose`), `IComparable` (just `CompareTo`), `IEnumerable` (just iterate). Each is tiny and focused. That's ISP in the framework itself.

---

## D — Dependency Inversion Principle (DIP)

**Plain words:** depend on **abstractions (interfaces)**, not on **concrete details.** High-level code (your business rules) shouldn't be glued directly to low-level details (a specific database, a specific email service).

**Picture it:** a lamp plugs into the **wall socket** (a standard interface), not soldered directly to the power station. You can move the lamp to any socket in any building. The socket is the abstraction in the middle.

**The smell:** a high-level class that does `new SqlDatabase()` inside itself — now it's permanently married to SQL Server and you can't test it without a real database.

```csharp
// BAD — OrderService is hard-wired to a specific class
public class OrderService
{
    private readonly SqlDatabase _db = new SqlDatabase();   // glued to SQL forever
    public void Place(Order o) => _db.Save(o);
}

// GOOD — depend on an interface; the real one is handed in from outside
public interface IOrderRepository { void Save(Order o); }

public class SqlOrderRepository : IOrderRepository { public void Save(Order o) { /* SQL */ } }

public class OrderService
{
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo)   // it asks for "anything that can save"
    {
        _repo = repo;
    }
    public void Place(Order o) => _repo.Save(o);
}
```

Now `OrderService` works with SQL, with an in-memory fake (great for tests), or with a future MongoDB version — **without changing a line.** Handing the dependency in from outside like this is called **Dependency Injection**, and it's exactly what ASP.NET Core's built-in container does for you. (You'll go deep on DI in Week 6.)

**Don't confuse the two terms:**
- **Dependency Inversion** = the *principle* (depend on interfaces).
- **Dependency Injection** = the *technique* (pass the dependency in via the constructor) that achieves it.

---

# Interview Questions (with simple answers)

**Q1. What are the four pillars of OOP?**
Encapsulation (hide the insides, expose safe buttons), Inheritance (a child reuses a parent via "is-a"), Polymorphism (same method call, different behaviour per type), and Abstraction (show a simple surface, hide complexity).

**Q2. Difference between an interface and an abstract class?**
An interface is a pure contract describing a *capability* ("can do X"), has no fields, and a class can implement many. An abstract class is a half-built parent that can hold real shared code and fields, models an "is-a" family, and a class can inherit only one. Use interfaces for capabilities across unrelated types; abstract classes to share code among related types.

**Q3. What is the difference between overriding and overloading?**
**Overriding** is a child replacing a parent's `virtual` method with its own version (run-time polymorphism). **Overloading** is having several methods with the *same name but different parameters* in the same class (decided at compile time). Different ideas that sound similar.

**Q4. Inheritance vs composition — which should I prefer and why?**
Favour composition ("has-a"). Inheritance ("is-a") is rigid: it locks you to one parent and drags its whole baggage along, and deep chains get fragile. Composition snaps parts together flexibly, is easier to test and swap, and avoids the fragile-base-class problem. Use inheritance only for a true, stable "is-a" with real reuse.

**Q5. Explain SOLID in one sentence each.**
S — one class, one job. O — add new behaviour by adding code, not editing old code. L — any child must be usable wherever its parent is, without surprises. I — many small focused interfaces beat one fat one. D — depend on interfaces, not concrete classes.

**Q6. What's the difference between Dependency Inversion and Dependency Injection?**
Dependency Inversion is the *principle*: high-level code depends on abstractions, not details. Dependency Injection is the *technique* that delivers it: pass the needed dependency in (usually through the constructor) instead of `new`-ing it inside.

**Q7. Give a real example of breaking the Liskov Substitution Principle.**
The Square/Rectangle case: a `Square` overriding width/height to stay equal breaks code that sets them independently and expects `Area == width × height`. The child betrayed the parent's contract. Also: any subclass that throws "not supported" for an inherited method.

**Q8. What does encapsulation actually protect you from?**
From other code putting your object into an invalid state. By keeping data private and only allowing changes through methods that enforce rules (like "deposit must be positive"), you guarantee the object is always valid.

**Q9. Can you have a class implement two interfaces with the same method name?**
Yes. You can implement it once for both, or use **explicit interface implementation** (`void IFoo.Method()`) to give each interface its own version, chosen by which interface type you call through.

**Q10. Why is "favour composition over inheritance" such common advice?**
Because inheritance is the tightest coupling in OOP — a child is permanently bound to a parent's internals and can break when the parent changes. Composition keeps pieces independent and swappable, which makes large codebases far easier to change safely.

---

# Scenario Questions (guess the answer, then check)

### S1 — Polymorphism in action
```csharp
public class Animal { public virtual string Sound() => "..."; }
public class Dog : Animal { public override string Sound() => "Woof"; }

Animal a = new Dog();
Console.WriteLine(a.Sound());
```
**Answer:** `Woof`. Even though the variable's type is `Animal`, the object is really a `Dog`, so the overridden version runs. That's run-time polymorphism.

### S2 — virtual missing
```csharp
public class Animal { public string Sound() => "..."; }     // NOT virtual
public class Dog : Animal { public new string Sound() => "Woof"; }  // 'new' hides it

Animal a = new Dog();
Console.WriteLine(a.Sound());
```
**Answer:** `...`. Without `virtual`/`override`, using `new` only *hides* the method. When called through an `Animal` variable, you get the `Animal` version. This is a classic trap — always use `virtual`/`override` for real polymorphism.

### S3 — Encapsulation saves you
```csharp
var acc = new BankAccount();
acc.Deposit(100);
acc.Deposit(-50);   // what happens?
```
**Answer:** the second call throws `ArgumentException` (our rule rejected the negative amount), so the balance stays 100. Because the field is private, there's no other way to corrupt it.

### S4 — Which SOLID letter is broken?
```csharp
public interface IPrinter
{
    void Print();
    void Scan();
    void Fax();
}
public class OldPrinter : IPrinter
{
    public void Print() { /* ok */ }
    public void Scan()  { throw new NotSupportedException(); }
    public void Fax()   { throw new NotSupportedException(); }
}
```
**Answer:** **I** (Interface Segregation). A simple printer is forced to implement Scan and Fax it doesn't have. Fix: split into `IPrint`, `IScan`, `IFax`.

### S5 — Which SOLID letter is broken?
```csharp
public class ReportService
{
    public void Generate()
    {
        var db = new SqlConnection("...");   // creates its own concrete dependency
        // read data, build report, also write the PDF file here...
    }
}
```
**Answer:** **D** (Dependency Inversion — it `new`s a concrete SQL connection instead of depending on an interface). Arguably **S** too, since it both fetches data *and* builds the PDF (two jobs). Both fixes: inject an `IDataSource`, and move PDF creation to its own class.

### S6 — Abstract class can't be created
```csharp
public abstract class Shape { public abstract double Area(); }
var s = new Shape();
```
**Answer:** compile error. You cannot instantiate an abstract class directly — it's a half-built blueprint. You must create a concrete child like `new Circle()`.

### S7 — Composition swap
```csharp
public interface IEngine { void Start(); }
public class PetrolEngine : IEngine { public void Start() => Console.WriteLine("Vroom"); }
public class ElectricEngine : IEngine { public void Start() => Console.WriteLine("Hum"); }

public class Car
{
    private readonly IEngine _engine;
    public Car(IEngine engine) => _engine = engine;
    public void Go() => _engine.Start();
}

new Car(new ElectricEngine()).Go();
```
**Answer:** `Hum`. Because `Car` is *composed* with an `IEngine` passed in, you can swap petrol for electric without touching `Car` at all. This is composition + DIP working together.

---

# Hands-on Exercises (try them this week)

These build toward your roadmap task: **a Library console app that applies all 5 SOLID principles.**

**E1.** Make a `BankAccount` class with a private `_balance`, a public read-only `Balance`, and `Deposit`/`Withdraw` methods that reject invalid amounts (negative, or withdrawing more than the balance). Prove from `Main` that you cannot put it into a bad state. *(Encapsulation)*

**E2.** Make an abstract `Shape` with an abstract `Area()` and a real `Describe()`. Create `Circle`, `Rectangle`, `Triangle`. Loop over a `List<Shape>` and print each area. Then add a `Pentagon` and notice you changed **no existing code**. *(Polymorphism + Open/Closed)*

**E3.** Take a deliberately bad `OrderProcessor` class that validates, saves, and emails all in one method. Split it into `IOrderValidator`, `IOrderRepository`, and `IEmailSender`, and have `OrderProcessor` receive all three through its constructor. *(SRP + DIP)*

**E4.** Build the Library app skeleton: a `Book` (encapsulated fields), an `IBookRepository` interface with an in-memory implementation, an `ILoanService` that borrows/returns books, and a `INotifier` interface (console version now, email version later). Wire them together in `Main` by hand (manual dependency injection). Write one sentence per class naming which SOLID letter it demonstrates.

**E5.** Reproduce the broken Square/Rectangle example and watch the test print 16 instead of 20. Then redesign it so the bug is impossible — e.g. make both implement a common `IShape { int Area(); }` instead of `Square : Rectangle`. *(Liskov)*

---

## Quick self-check before Week 3
You are ready when you can explain, in your own simple words:
1. The difference between a class and an object (blueprint vs house).
2. All four OOP pillars with a real-life example for each.
3. When to use an interface vs an abstract class.
4. Why "favour composition over inheritance" is good advice.
5. Each SOLID letter — the pain it removes, not just the definition.
6. The difference between Dependency Inversion (principle) and Dependency Injection (technique).

When all six feel easy, you've got the foundation that the rest of .NET (especially ASP.NET Core's DI) is built on. Next week: Generics, Collections & LINQ.
