# Week 1 — C# Basics in Simple English
### Value vs Reference, Memory, Boxing, Structs, Enums, Strings, var/const

> How to use this: read one section, then say the idea out loud in your own words. If you can explain it to a friend simply, you know it. The goal is not to memorise — it's to be able to talk about it calmly in an interview.

---

## 1. Value types vs Reference types

**The simplest way to think about it:**

- A **value type** is like writing a number on a piece of paper. If your friend wants it, you write a **fresh copy** on a new paper and give it to them. Now there are two papers. If they change theirs, yours stays the same.

- A **reference type** is like a shared Google Doc. You give your friend the **link**, not a copy. You both open the *same* document. If they edit it, you see the change too, because it's the same document.

**Detailed definition:**

A value type variable **holds the actual data inside itself**. When you copy it, the whole value is duplicated. The two copies are now totally separate.

A reference type variable **does not hold the data**. It holds an *address* that points to where the real data lives. When you copy the variable, you only copy the address — both variables still point to the same one object.

**See it in code:**

```csharp
// VALUE type (int) — copying makes a separate copy
int a = 5;
int b = a;     // b is a fresh copy of 5
b = 10;        // changing b does NOT touch a
// a is still 5

// REFERENCE type (array) — copying shares the same thing
int[] x = { 1, 2, 3 };
int[] y = x;   // y points to the SAME array as x (just the address copied)
y[0] = 99;     // we changed the shared array
// x[0] is now also 99
```

**Which is which?**

- Value types: `int`, `double`, `bool`, `char`, `decimal`, `DateTime`, `Guid`, and anything made with `struct` or `enum`.
- Reference types: `class`, `string`, arrays, `interface`, `delegate`, `object`.

**One more important point — passing to a method.**
By default C# passes a **copy**. So if you give a method a number, the method works on a copy and your original is safe. If you give it a reference type (like a list), it gets a copy of the *address*, so it can still change the shared object inside.

You can change this behaviour with keywords:
- `ref` — let the method read **and** change your original variable.
- `out` — the method must give your variable a value before it finishes (used to "return" extra results).
- `in` — the method can read your variable but not change it (used for speed with big structs).

---

## 2. Stack vs Heap (where data is kept in memory)

Think of two storage areas in your computer's memory:

- **The Stack** is like a **stack of plates** on a counter. It is fast and tidy. Every time a method (function) runs, it puts its local things on top. When the method finishes, those plates are removed instantly. It mostly holds small, short-lived things.

- **The Heap** is like a **big warehouse**. Bigger things (objects created with `new`) are stored here. A helper called the **Garbage Collector (GC)** walks around the warehouse and throws away things nobody is using anymore, so you don't have to clean up by hand.

**Detailed definition:**
The stack stores the local variables of a running method and is cleaned up automatically the moment the method returns. The heap stores objects (reference types) and is cleaned up later by the Garbage Collector when nothing points to them anymore.

**The trick interviewers love (please remember this one):**
People often say "value types always live on the stack." **This is not always true.** Where a value lives depends on the situation:
- A normal number inside a method → stack.
- A number that is a **field inside a class object** → it lives in the **heap**, inside that object.

So the safe thing to say in an interview is:
> "Whether something is on the stack or heap is an internal detail. What really matters is the copy rule: value types copy the whole value, reference types copy only the address."

Saying that makes you sound senior, because it is exactly right.

---

## 3. Boxing and Unboxing

**Simple picture:** Boxing is putting a small value (like the number 42) inside a **gift box** so it can sit on the "objects" shelf (the heap). Unboxing is opening that box to take the number back out.

**Detailed definition:**
- **Boxing** happens when you put a value type (like `int`) into an `object`. C# secretly creates a small object on the heap and copies the value into it. This is *automatic* (you don't ask for it).
- **Unboxing** is taking the value back out with a cast. You must say the exact original type, or it crashes.

```csharp
int n = 42;
object o = n;        // BOXING: a box is made on the heap, 42 copied in
int back = (int)o;   // UNBOXING: take 42 back out
```

**Why should you care?**
Making a box uses memory and gives the Garbage Collector extra work. If this happens millions of times (inside a big loop), your program gets slow. So boxing is something to **avoid in hot code**.

**The good news:** modern collections like `List<int>` do **not** box. Only old ones (like `ArrayList`) and putting values into `object` do. This is one big reason "generics" (`List<T>`) were invented.

---

## 4. Structs

**Simple picture:** A `struct` is a small value type you design yourself, used for small things that feel like "one value" — a point on a map (X, Y), a sum of money, a date range.

**Detailed definition:**
A `struct` is like a `class` but it is a **value type**. That means it follows the copy rule: when you assign it or pass it to a method, the **whole thing is copied**. It is best for small, simple data that does not change after it is created.

```csharp
public readonly struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }
    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
}
```

**Easy rules to remember:**
- Use a struct when the data is **small** and **should not change** after creation.
- Add the word `readonly` to promise it never changes — this is safer and a little faster.
- **Do not make structs that change after creation.** Because structs copy themselves so easily, you often end up changing a *copy* by mistake and wonder why nothing happened. This is a very common bug. Keep structs unchangeable.
- If your data is big or changes a lot, use a `class` instead.

---

## 5. Enums

**Simple picture:** An `enum` is a short list of allowed named choices. Like a traffic light: Red, Yellow, Green. Instead of using numbers (0, 1, 2) that mean nothing on their own, you give them names.

**Detailed definition:**
An `enum` lets you give friendly names to a fixed set of options. Behind the scenes each name is just a number (the first is 0 by default). Enums are value types. They make code easier to read and harder to get wrong.

```csharp
public enum Status { Pending, Active, Closed }   // Pending=0, Active=1, Closed=2
```

**Flags — choosing more than one option at once:**
Sometimes you want to combine choices (like file permissions: read AND write). For that, mark the enum with `[Flags]` and give each item a power-of-two number:

```csharp
[Flags]
public enum Permissions { None = 0, Read = 1, Write = 2, Delete = 4 }

var p = Permissions.Read | Permissions.Write;   // means "Read and Write"
bool canWrite = p.HasFlag(Permissions.Write);   // true
```

**Two things to watch out for:**
- An enum is not bullet-proof. Someone can force a wrong number in, like `(Status)99`, and it will not match any name. When data comes from outside (a file, a web request), check it with `Enum.IsDefined`.
- The number 0 is the default. So make the 0 item a sensible "nothing" choice, like `None` or `Unknown`.

---

## 6. Strings (text)

**Simple picture:** A `string` is text, like `"hello"`. The surprising rule: **a string can never be changed after it is made.** If you "change" it, C# quietly throws the old one away and makes a brand new one.

**Detailed definition:**
`string` is technically a reference type (it lives on the heap), but it *behaves* like a value because it is **immutable** (unchangeable) and because `==` compares the **text content**, not the address. So two strings with the same letters are seen as equal.

```csharp
string s = "a";
s += "b";   // this does NOT edit "a". It makes a NEW string "ab".
```

**Why this matters — the most common beginner mistake:**
If you build text inside a big loop using `+`, C# creates a new string every single time. With thousands of loops this becomes very slow. Use a `StringBuilder` instead — think of it as a notepad you keep adding to, instead of rewriting the whole page each time.

```csharp
var sb = new System.Text.StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);
string result = sb.ToString();   // build once at the end
```

**One handy tip:** when comparing text that is not human language (like IDs, file names, codes), use `string.Equals(a, b, StringComparison.Ordinal)`. It is faster and avoids weird bugs in different countries' language settings.

---

## 7. var vs const (and readonly)

These three words confuse people. Here is the simple version.

**`var` — "you figure out the type for me."**
You use `var` when the type is obvious from the right side. The compiler still picks a real, fixed type — `var` is just less typing. It is **not** slower and it is **not** flexible/dynamic.

```csharp
var name = "Anil";   // compiler knows this is a string
var age = 30;        // compiler knows this is an int
```

**`const` — "this value is fixed forever, decided right now."**
Use `const` for things that never change and are known while writing the code, like Pi. The value gets stamped directly into the code that uses it.

```csharp
const double Pi = 3.14159;
```
*Watch out:* if a `const` lives in a shared library and other programs use it, they each keep their **own copy** of the value. If you later change it, they won't see the new value until they are rebuilt. So for shared values that might change, prefer `readonly` (below).

**`readonly` — "set once, then locked."**
Use `readonly` for a value you set when the object is created and never again. Unlike `const`, it can be a value worked out at run time (like the current date).

```csharp
public static readonly DateTime StartedAt = DateTime.Now;  // decided when the app runs
```

**Quick rule:**
- `var` = let the compiler name the type for you.
- `const` = truly fixed, simple value known while coding (Pi, a fixed limit).
- `readonly` = set once when the program runs, then never changes.

---

# Interview Questions (with simple answers)

**Q1. Is `string` a value type or a reference type? Why does it feel like both?**
It is a reference type. It *feels* like a value type because it can never be changed (immutable) and because `==` compares the actual text, not the memory address. So copies act independently.

**Q2. Do value types always live on the stack?**
No. A value type that sits *inside* a class object lives on the heap with that object. The real rule is the copy rule (value copies the whole value, reference copies the address). Stack vs heap is an internal detail.

**Q3. What is boxing and why can it be bad?**
Boxing is putting a value type (like `int`) into an `object`, which secretly makes a small object on the heap. It is bad in large loops because it uses memory and slows things down. `List<int>` avoids it; old `ArrayList` causes it.

**Q4. Difference between `const` and `readonly`?**
`const` is fixed while writing code and copied into everyone who uses it. `readonly` is set once when the program runs (can be a calculated value) and is safer for values that might change later.

**Q5. When would you use a `struct` instead of a `class`?**
For small data that behaves like one simple value and does not change — like a point or money amount. For bigger data, or data that changes a lot, use a class.

**Q6. Why are changeable (mutable) structs dangerous?**
Because structs copy themselves so easily, you often edit a copy by accident, and your real data does not change. This causes confusing bugs. Keep structs `readonly`.

**Q7. What does `[Flags]` do?**
It lets one enum value hold several choices at once (like Read + Write). You give each item a power-of-two number and combine them with `|`.

**Q8. Is `var` the same as dynamic? Is it slow?**
No and no. `var` is just shorthand — the compiler still picks one fixed type. It runs exactly the same as writing the type yourself. (`dynamic` is the flexible-at-runtime one, and it is different.)

**Q9. How does `==` behave for value types, strings, and other objects?**
Value types compare their content. `string` compares its text. Other reference types compare the address (are they the exact same object?) unless the class is written to compare differently.

**Q10. Why use `StringComparison.Ordinal`?**
It compares text letter-by-letter quickly without language rules, which is correct and safe for IDs, codes, and file paths. Use language-aware comparing only for text shown to users.

---

# Scenario Questions (guess the answer, then check)

### S1 — Shared list
```csharp
var list1 = new List<int> { 1, 2, 3 };
var list2 = list1;
list2.Add(4);
Console.WriteLine(list1.Count);
```
**Answer:** `4`. `list2` points to the same list (reference copy). To make a real separate copy: `var list2 = new List<int>(list1);`

### S2 — Struct copy
```csharp
struct P { public int X; }
var a = new P { X = 1 };
var b = a;
b.X = 99;
Console.WriteLine(a.X);
```
**Answer:** `1`. A struct copies the whole value, so `b` is separate. (If `P` were a `class`, the answer would be `99`.)

### S3 — Two boxes are not equal
```csharp
int n = 5;
object o1 = n;   // box
object o2 = n;   // another box
Console.WriteLine(o1 == o2);
```
**Answer:** `False`. They are two different boxes (compared by address). `o1.Equals(o2)` would be `True`.

### S4 — String does not change itself
```csharp
string s = "hello";
s.ToUpper();
Console.WriteLine(s);
```
**Answer:** `hello`. `ToUpper()` returns a NEW string and leaves the old one alone. Fix: `s = s.ToUpper();`

### S5 — Slow text building
```csharp
string result = "";
for (int i = 0; i < 100000; i++)
    result += i + ",";
```
**Problem:** a new string is made every loop → very slow. **Fix:** use `StringBuilder`.

### S6 — Enum from bad data
```csharp
enum Status { Pending, Active, Closed }
var s = (Status)42;
Console.WriteLine(s);                                   // 42 (no name)
Console.WriteLine(Enum.IsDefined(typeof(Status), s));   // False
```
**Lesson:** always check enums that come from outside with `Enum.IsDefined`.

### S7 — Default struct
```csharp
struct Point { public int X; public int Y; }
Point p = default;
Console.WriteLine($"{p.X},{p.Y}");
```
**Answer:** `0,0`. A struct's fields start at zero automatically.

### S8 — const surprise
> A shared library has `public const int MaxRetries = 3;`. Program B was built when it was 3. You change the library to 5 but do not rebuild B. What does B see?
**Answer:** still `3`, because the value was copied into B when it was built. Use `static readonly` if the value may change later.

---

# Hands-on Exercises (try them this week)

Try to guess the output first, then run the code to check.

**E1.** Make a `class Person { public string Name; }` and a `struct PersonStruct { public string Name; }`. Write a method that changes `.Name` for each. Show that the class change is seen by the caller but the struct change is not. Then add a `ref` version for the struct and show it now is seen.

**E2.** Add a million numbers to an old `ArrayList`, then to a `List<int>`. Time both with `Stopwatch`. Write one sentence saying why `ArrayList` is slower (boxing).

**E3.** Join the numbers 0 to 50,000 into text two ways: with `+=` and with `StringBuilder`. Time both. Write down how many times faster `StringBuilder` was.

**E4.** Make a `[Flags] enum FileAccess { None=0, Read=1, Write=2, Execute=4 }`. Combine Read and Write, print it, check `HasFlag(Write)`, then add Execute and remove Read.

**E5.** Make a class with a `const Pi`, a `static readonly DateTime StartedAt = DateTime.Now`, and a method that uses `var` for three different types. In a comment, explain why `StartedAt` cannot be a `const`.

---

## Quick self-check before Week 2
You are ready when you can explain, in your own simple words:
1. The difference between value (copy) and reference (shared link).
2. Why "value types always live on the stack" is not fully true.
3. Why a `string` feels like a value even though it is a reference type.
4. What boxing is and why it can slow a program.
5. When to use a `struct`, and why changeable structs are risky.
6. The difference between `var`, `const`, and `readonly`.
