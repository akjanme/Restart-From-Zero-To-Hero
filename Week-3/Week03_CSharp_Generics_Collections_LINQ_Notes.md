# Week 3 — Generics, Collections & LINQ in Simple English
### List/Dictionary/HashSet, IEnumerable vs IQueryable, and LINQ deep dive (deferred execution, joins, grouping)

> How to use this: read one section, then say the idea out loud in your own words. Week 1 was about how data sits in memory. Week 2 was about organising code into objects. This week is about **storing groups of data well** and **querying them like a database, in code** — this is the single most-used skill in day-to-day C# work.

---

## 1. What are Generics, really?

**The simplest way to think about it:**

A generic is a **container with a label you fill in later**. Imagine a box that says "Box of ___". You decide at the moment you use it: "Box of books", "Box of shoes", "Box of numbers". The box works the same way no matter what's inside — it just needs to know the type up front so it can keep things safe.

**Why they exist — the problem before generics:**

Before generics, people used `ArrayList`, which stores everything as `object`. That caused two pains:
1. **No type safety** — you could put a string and an int in the same list by accident, and only find out at run time (crash).
2. **Boxing/unboxing cost** (remember Week 1) — every value type going into an `object` list gets boxed onto the heap, which is slow.

Generics fix both: you say the type once, the compiler enforces it everywhere, and value types don't need boxing.

```csharp
// OLD WAY — ArrayList, not type-safe, boxes value types
ArrayList list = new ArrayList();
list.Add(5);          // boxed int
list.Add("oops");     // compiles fine — but wrong!  Bug found only at run time

// GENERIC WAY — List<T>, type-safe, no boxing
List<int> numbers = new List<int>();
numbers.Add(5);
numbers.Add("oops");  // COMPILE ERROR — caught immediately
```

**Writing your own generic (you'll be asked this in interviews):**

```csharp
// T is a placeholder — "the type will be decided when someone uses this class"
public class Box<T>
{
    private T _item;
    public void Put(T item) => _item = item;
    public T Get() => _item;
}

Box<string> b1 = new Box<string>();
b1.Put("hello");

Box<int> b2 = new Box<int>();
b2.Put(42);
```

**Why interviews test this:** almost every collection you use daily (`List<T>`, `Dictionary<TKey,TValue>`) is generic. Understanding *why* they exist (type safety + performance) shows you understand the framework, not just the syntax.

---

## 2. The main collections — pick the right box for the job

Each collection is shaped for a different question you want to ask fast. Picture them like this:

### `List<T>` — a numbered row of lockers
**Picture it:** a row of lockers, numbered 0, 1, 2, 3... You can jump straight to locker #5. You can also add a new locker at the end easily.

**Use it when:** you need an **ordered** sequence and mostly add to the end / read by position.

```csharp
List<string> names = new List<string> { "Amit", "Bella" };
names.Add("Cara");        // add to end — fast
string first = names[0];  // jump to index — fast, O(1)
names.Remove("Bella");    // removing from the middle — slow, O(n), shifts everything
```

### `Dictionary<TKey, TValue>` — a phone book
**Picture it:** a phone book. You don't scan every page for "Bella" — you jump straight to the "B" section using the **key**. Every entry is a `key → value` pair.

**Use it when:** you need instant lookup by a unique key. (This is exactly the HashSet/Dictionary trick from Week 1 — same idea, but here you store a *value* alongside each key.)

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>();
ages["Amit"] = 30;
ages["Bella"] = 25;

if (ages.TryGetValue("Amit", out int age))   // safe lookup, no exception if missing
    Console.WriteLine(age);                  // 30 — instant, O(1) average
```

### `HashSet<T>` — a guest list where nobody can sign in twice
**Picture it:** a guest list at the door. The bouncer only cares "is this name already on the list?" — no duplicates allowed, and no particular order.

**Use it when:** you need **uniqueness** and fast "have I seen this?" checks (exactly last week's `RemoveDuplicates`-style pattern, and Week 1's Two Sum).

```csharp
HashSet<int> seen = new HashSet<int>();
seen.Add(5);
seen.Add(5);              // ignored — already there
Console.WriteLine(seen.Contains(5));  // true, O(1) average
```

### `Queue<T>` and `Stack<T>` — the two disciplines of a line
**Picture it — Queue:** a line at a coffee shop. First person in line is first served: **FIFO** (first in, first out).
**Picture it — Stack:** a stack of plates. You take the top plate off first: **LIFO** (last in, first out).

```csharp
Queue<string> line = new Queue<string>();
line.Enqueue("Amit"); line.Enqueue("Bella");
Console.WriteLine(line.Dequeue());   // "Amit" — first one in, first one out

Stack<string> plates = new Stack<string>();
plates.Push("Plate1"); plates.Push("Plate2");
Console.WriteLine(plates.Pop());     // "Plate2" — last one in, first one out
```

### The cheat-sheet you can say in an interview:

| Need | Collection | Why |
|------|-----------|-----|
| Ordered list, access by position | `List<T>` | Index lookup is O(1) |
| Fast lookup by a unique key, with a value attached | `Dictionary<TKey,TValue>` | Hashing gives O(1) average lookup |
| Fast "have I seen this?" / no duplicates | `HashSet<T>` | Hashing, no values needed, just presence |
| Process items in the order they arrived | `Queue<T>` | FIFO |
| Undo/redo, "last thing wins first" | `Stack<T>` | LIFO |

---

## 3. `IEnumerable` vs `IQueryable` vs `ICollection` vs `IList` (a favourite interview trap)

**The simplest way to think about it:** these are **contracts describing how much you're allowed to do** with a sequence of data (remember interfaces from Week 2 — this is that idea applied to collections).

- **`IEnumerable<T>`** — "I promise you can walk through me, one item at a time, forward only." That's it. The most basic contract. Every collection implements this.
- **`ICollection<T>`** — everything `IEnumerable<T>` has, **plus** `Count`, `Add`, `Remove`, `Contains`. "I know how many items I have and you can change me."
- **`IList<T>`** — everything `ICollection<T>` has, **plus** access by index (`list[i]`). "I'm ordered and you can jump to any position."
- **`IQueryable<T>`** — looks like `IEnumerable<T>` but is very different underneath: instead of running the query in memory right away, it **builds up an expression tree** and only sends the final translated query (e.g. as SQL) to the data source when you actually enumerate it.

**Picture the `IEnumerable` vs `IQueryable` difference:**

`IEnumerable` — you already brought the entire fruit basket home, and now you're picking out the apples one by one, in memory, in your kitchen.

`IQueryable` — you call the fruit shop and say "send me only the apples." The filtering happens **at the shop** (the database), and only the apples travel to you.

```csharp
// IEnumerable — filtering happens in C#, AFTER all rows are pulled into memory
IEnumerable<User> users = dbContext.Users.ToList();     // pulls EVERY row first
var admins = users.Where(u => u.Role == "Admin");       // then filters in memory — wasteful

// IQueryable — filtering happens IN THE DATABASE, only matching rows come over the wire
IQueryable<User> query = dbContext.Users;                 // nothing runs yet
var admins2 = query.Where(u => u.Role == "Admin");        // still nothing runs — just builds a plan
var result = admins2.ToList();                            // NOW it runs — translated to SQL: SELECT * WHERE Role = 'Admin'
```

**The one-liner to remember:** *`IEnumerable` = "filter in memory, in C#." `IQueryable` = "build a query, let the database (or source) do the filtering, and only bring back what you need."* Using `IQueryable` incorrectly — like calling `.ToList()` too early — silently turns off this optimisation. This is a very common real-world Entity Framework bug.

---

## 4. LINQ — asking questions about your data, in C#

**The simplest way to think about it:**

LINQ (Language Integrated Query) lets you write **database-style questions directly in C#** — "give me all users over 18, sorted by name" — instead of writing manual `for` loops with `if` statements. It works the same way whether the data is an in-memory `List`, a database (via `IQueryable`), an XML file, or almost anything else.

**Two ways to write the same LINQ query:**

```csharp
List<int> nums = new List<int> { 5, 2, 8, 1, 9, 3 };

// Query syntax — reads like SQL
var query1 = from n in nums
             where n > 3
             orderby n
             select n;

// Method syntax — chained methods (far more common in real code)
var query2 = nums.Where(n => n > 3).OrderBy(n => n);

// Both give: 5, 8, 9
```

Most C# developers use **method syntax** day to day. Query syntax is good to recognise but you'll write method syntax far more often.

---

## 5. Deferred execution — the most important LINQ concept for interviews

**Picture it:** ordering food at a restaurant. Writing `.Where(...)` is like **giving the waiter your order**. The food doesn't appear yet — nothing has been cooked. The kitchen only starts cooking (the query actually runs) when you **ask for the plate** — by looping over the result, or calling `.ToList()`, `.Count()`, `.First()`, etc.

**Definition:** most LINQ methods (`Where`, `Select`, `OrderBy`, `GroupBy`...) don't run immediately. They just **build up a description of the work**. The work only happens when the sequence is actually enumerated (a `foreach`, or a method like `.ToList()`, `.Count()`, `.Sum()`, `.First()`).

```csharp
List<int> nums = new List<int> { 1, 2, 3 };

var query = nums.Where(n =>
{
    Console.WriteLine($"checking {n}");
    return n > 1;
});

Console.WriteLine("query created — nothing printed above this line yet!");

foreach (var n in query)          // THIS is when the filtering actually runs
    Console.WriteLine($"got {n}");
```

**Output:**
```
query created — nothing printed above this line yet!
checking 1
checking 2
got 2
checking 3
got 3
```

**Why this bites people:** if the underlying data changes *between* creating the query and enumerating it, the query sees the **new** data — because it wasn't run yet!

```csharp
List<int> nums = new List<int> { 1, 2, 3 };
var query = nums.Where(n => n > 1);
nums.Add(10);                      // change the list AFTER building the query
foreach (var n in query)
    Console.WriteLine(n);          // prints 2, 3, 10 — the new item is included!
```

**Force immediate execution** with `.ToList()`, `.ToArray()`, `.ToDictionary()`, `.Count()`, `.Sum()`, `.First()`, etc. These are called **execution methods** — the moment you call one, the whole chain runs right then, and the result is locked in.

```csharp
var snapshot = nums.Where(n => n > 1).ToList();   // runs NOW, snapshot is frozen
nums.Add(99);                                     // does NOT affect 'snapshot' anymore
```

**The one-liner to remember:** *building a LINQ query is just writing a recipe. The recipe only cooks when you enumerate it (`foreach`, `.ToList()`, `.Count()`...). Until then, nothing has actually run.*

---

## 6. The LINQ operators you'll use constantly

Think of these as your everyday toolbox. Learn what each one is *for*, not just the syntax.

```csharp
List<int> nums = new List<int> { 5, 2, 8, 1, 9, 3, 8 };

nums.Where(n => n > 3);              // FILTER — keep only items matching a condition
nums.Select(n => n * 2);             // TRANSFORM — turn each item into something else
nums.OrderBy(n => n);                // SORT ascending
nums.OrderByDescending(n => n);      // SORT descending
nums.Distinct();                     // remove duplicates
nums.Skip(2);                        // skip the first 2 items
nums.Take(3);                        // keep only the first 3 items
nums.First();                        // first item — THROWS if the sequence is empty
nums.FirstOrDefault();               // first item, or default (0/null) if empty — SAFER
nums.Single(n => n == 9);            // exactly ONE match expected — throws if 0 or 2+ matches
nums.Any(n => n > 100);              // true if AT LEAST ONE matches (stops early — fast)
nums.All(n => n > 0);                // true if EVERY item matches
nums.Count(n => n > 3);              // how many match
nums.Sum();  nums.Max();  nums.Min(); nums.Average();   // aggregate math
nums.Aggregate((a, b) => a + b);     // roll every item into one result, your own rule
```

**`First` vs `FirstOrDefault` vs `Single` — a classic interview question:**

| Method | 0 matches | 1 match | 2+ matches |
|--------|-----------|---------|-----------|
| `First()` | throws | returns it | returns the **first** one |
| `FirstOrDefault()` | returns `default` (e.g. `0`, `null`) | returns it | returns the **first** one |
| `Single()` | throws | returns it | **throws** (it's ambiguous!) |
| `SingleOrDefault()` | returns `default` | returns it | **throws** |

**Rule of thumb:** use `Single`/`SingleOrDefault` when you're saying "there should be exactly one" (like looking up a user by unique ID) — it protects you by throwing if your assumption was wrong.

---

## 7. Grouping and Joining — the two that feel like real database work

### `GroupBy` — sort items into labelled buckets

**Picture it:** a big pile of laundry, and you're sorting it into baskets by colour. `GroupBy` doesn't merge anything — it just **buckets** items by a key you choose.

```csharp
List<string> words = new List<string> { "apple", "ant", "banana", "berry", "cherry" };

var groups = words.GroupBy(w => w[0]);   // group by first letter

foreach (var group in groups)
{
    Console.WriteLine($"Letter {group.Key}:");
    foreach (var word in group)
        Console.WriteLine($"  {word}");
}
// Letter a: apple, ant
// Letter b: banana, berry
// Letter c: cherry
```

Each `group` is itself a mini sequence (`IGrouping<TKey, T>`) — it has a `.Key` and you can loop through its members, or call `.Count()`, `.Select()`, etc. on it.

### `Join` — matching two lists by a shared key

**Picture it:** matching students to their classrooms using a shared `ClassId` — like lining up two spreadsheets on a common column, exactly like a SQL `INNER JOIN`.

```csharp
record Student(int Id, string Name, int ClassId);
record Classroom(int ClassId, string Room);

List<Student> students = new()
{
    new Student(1, "Amit", 10),
    new Student(2, "Bella", 20),
    new Student(3, "Cara", 10),
};
List<Classroom> classes = new()
{
    new Classroom(10, "Room A"),
    new Classroom(20, "Room B"),
};

var result = students.Join(
    classes,                       // the other list
    s => s.ClassId,                // key from students
    c => c.ClassId,                // key from classes
    (s, c) => $"{s.Name} is in {c.Room}"   // how to combine a match
);

foreach (var line in result) Console.WriteLine(line);
// Amit is in Room A
// Bella is in Room B
// Cara is in Room A
```

Only students whose `ClassId` matches a classroom appear — same behaviour as a SQL `INNER JOIN`. (For a "keep unmatched items too" version, look up `GroupJoin` with `DefaultIfEmpty` — the LINQ equivalent of a `LEFT JOIN`.)

---

# Interview Questions (with simple answers)

**Q1. Why use generics instead of `object`?**
Type safety (mistakes caught at compile time, not run time) and performance (value types stored in a generic collection don't need boxing, unlike storing them as `object`).

**Q2. When would you use a `Dictionary` vs a `List`?**
`List` when you need order and mostly access by position or add to the end. `Dictionary` when you need fast lookup by a unique key — trading a bit of memory for near-instant `O(1)` average lookups instead of scanning.

**Q3. What's the real difference between `IEnumerable` and `IQueryable`?**
`IEnumerable` runs its filtering **in memory, in C#**, on data you've already pulled in. `IQueryable` builds an **expression tree** and defers execution until enumerated, letting the underlying provider (e.g. a database) translate it and do the filtering at the source — usually far more efficient for large data sets.

**Q4. What is deferred execution and why does it matter?**
Most LINQ operators don't run immediately — they build a description of the work, which only executes when you enumerate the result (`foreach`, `.ToList()`, `.Count()`, etc). It matters because the query can see data that changed after it was defined, and because chaining many `Where`/`Select` calls costs nothing until you actually pull the results.

**Q5. Difference between `First()`, `FirstOrDefault()`, and `Single()`?**
`First()` throws if nothing matches, otherwise returns the first match even if there are many. `FirstOrDefault()` returns a default value instead of throwing when nothing matches. `Single()` expects **exactly one** match and throws if there are zero **or** more than one — use it to enforce a uniqueness assumption.

**Q6. What does `GroupBy` actually return?**
A sequence of `IGrouping<TKey, TElement>` — each item is itself a mini-sequence with a `.Key` property plus the members that share that key. It buckets, it doesn't merge or sum anything by itself.

**Q7. How is LINQ's `Join` similar to a SQL join?**
It matches items from two sequences based on equal keys and produces a combined result for every matching pair — the same behaviour as a SQL `INNER JOIN`. Unmatched items on either side are dropped, just like an inner join.

**Q8. Why can calling `.ToList()` too early on an `IQueryable` hurt performance?**
It forces the query to run and pull **all** rows into memory immediately, before further filtering (like a later `.Where()`) can be pushed down to the database. Any operations chained after that point run in memory in C# instead of being translated into SQL — you lose the database's ability to filter, sort, or page before sending data over the wire.

**Q9. What's the difference between `Where` and `Select`?**
`Where` **filters** — it keeps items and drops others, but every item that survives keeps its original shape. `Select` **transforms** — it keeps every item (same count in, same count out) but turns each one into something new (a different type, a projection, a calculated value).

**Q10. Explain boxing and why generic collections avoid it.**
Boxing is wrapping a value type (like `int`, which lives on the stack) inside an `object` on the heap, so it can be stored somewhere that expects a reference type — this costs a heap allocation. `List<int>` (generic) stores actual `int`s directly with no boxing, while the old `ArrayList` (non-generic, stores `object`) boxes every value type you add. This is a direct callback to Week 1's stack/heap lesson.

---

# Scenario Questions (guess the answer, then check)

### S1 — Deferred execution surprise
```csharp
List<int> nums = new List<int> { 1, 2, 3 };
var query = nums.Select(n => n * 10);
nums.Add(4);
foreach (var n in query) Console.Write(n + " ");
```
**Answer:** `10 20 30 40`. The query wasn't run when it was defined — only when the `foreach` enumerates it, by which point `4` was already added. This is deferred execution.

### S2 — Forcing immediate execution
```csharp
List<int> nums = new List<int> { 1, 2, 3 };
var snapshot = nums.Select(n => n * 10).ToList();
nums.Add(4);
foreach (var n in snapshot) Console.Write(n + " ");
```
**Answer:** `10 20 30`. `.ToList()` executed the query immediately and froze the result, so the later `Add(4)` has no effect on `snapshot`.

### S3 — `Single` throws
```csharp
List<int> nums = new List<int> { 1, 2, 2, 3 };
var result = nums.Single(n => n == 2);
```
**Answer:** throws `InvalidOperationException` — there are **two** `2`s, and `Single` requires exactly one match. If you only wanted "the first one, don't care if there are more," you should have used `First(n => n == 2)`.

### S4 — Dictionary key not found
```csharp
Dictionary<string, int> ages = new() { ["Amit"] = 30 };
int age = ages["Bella"];
```
**Answer:** throws `KeyNotFoundException` — direct indexing (`ages["Bella"]`) throws if the key is missing. The safe version is `ages.TryGetValue("Bella", out int age)`, which returns `false` instead of throwing.

### S5 — HashSet stops duplicates
```csharp
HashSet<int> ids = new HashSet<int>();
bool r1 = ids.Add(5);
bool r2 = ids.Add(5);
Console.WriteLine($"{r1} {r2} {ids.Count}");
```
**Answer:** `True False 1`. `Add` returns `true` only if the item was actually new. The second `Add(5)` is ignored, so `Count` stays `1`.

### S6 — IQueryable vs IEnumerable behaviour
```csharp
IQueryable<User> q = dbContext.Users.Where(u => u.Age > 18);
var list = q.ToList();
```
**Answer:** the `Where` filter is translated to SQL (`WHERE Age > 18`) and only matching rows are pulled from the database when `.ToList()` runs. If `dbContext.Users` had instead been `.ToList()`'d **before** the `Where`, every row would be pulled first and the filter would run in memory in C# — much slower for a big table.

### S7 — GroupBy then aggregate
```csharp
List<(string Name, string Dept)> people = new()
{
    ("Amit", "Eng"), ("Bella", "Eng"), ("Cara", "Sales")
};
var counts = people.GroupBy(p => p.Dept)
                    .Select(g => new { Dept = g.Key, Count = g.Count() });
foreach (var c in counts) Console.WriteLine($"{c.Dept}: {c.Count}");
```
**Answer:**
```
Eng: 2
Sales: 1
```
`GroupBy` buckets by department, then `Select` turns each bucket into a small summary object — a very common real-world pattern (like a `GROUP BY ... COUNT(*)` in SQL).

---

# Hands-on Exercises (try them this week)

These build toward your roadmap task: **refactor last week's Library app to use LINQ, plus 15 LINQ katas.**

**E1.** Take your Week 2 Library app's in-memory `List<Book>` repository. Replace any manual `foreach`/`if` search loops with LINQ (`Where`, `FirstOrDefault`, `OrderBy`). Confirm the behaviour is identical.

**E2.** Write a generic `Repository<T>` class with `Add(T item)`, `GetAll()`, and `Find(Func<T, bool> predicate)` using LINQ inside `Find`. Use it for both `Book` and a new `Member` type — prove the *same* class works for two unrelated types.

**E3.** Given a `List<Order>` with `CustomerName`, `Amount`, and `Date`, write LINQ queries to: total revenue per customer (`GroupBy` + `Sum`), the single highest order (`OrderByDescending` + `First`), and all orders in the last 30 days (`Where`).

**E4.** Reproduce the deferred-execution trap from S1 above on purpose. Then fix it with `.ToList()` and prove the behaviour changes. Write one sentence explaining why, in your own words.

**E5. 15 LINQ katas** — small, one-line-answer drills to build muscle memory. For a `List<int>` and a `List<Person> { Name, Age, City }`:
1. Sum of all even numbers.
2. Count of numbers greater than the average.
3. The 3 largest numbers, descending.
4. All distinct cities.
5. People grouped by city, with counts.
6. The oldest person's name (`OrderByDescending().First()`, then `Single` version if you assume no ties).
7. Average age per city.
8. All people whose name starts with a given letter.
9. A `Dictionary<string, List<Person>>` mapping city → people (`.ToDictionary` / `GroupBy` + `.ToDictionary`).
10. Numbers reversed, then take the first 5 (`Reverse().Take(5)`).
11. Check if **any** person is under 18 (`Any`).
12. Check if **all** people are adults (`All`).
13. Join people to a `List<(string City, string Country)>` to print "Name lives in Country."
14. Flatten a `List<List<int>>` into one `List<int>` (`SelectMany`).
15. Numbers converted to their string form, joined with commas (`Select` + `string.Join`).

---

## Quick self-check before Week 4
You are ready when you can explain, in your own simple words:
1. Why generics give you type safety and better performance than using `object`.
2. Which collection (`List`, `Dictionary`, `HashSet`, `Queue`, `Stack`) fits which situation, and why.
3. The real difference between `IEnumerable` and `IQueryable`, with an example of when picking the wrong one hurts performance.
4. What deferred execution means, and how to force immediate execution.
5. When to reach for `First` vs `FirstOrDefault` vs `Single`.
6. How `GroupBy` and `Join` work, and how they map to SQL `GROUP BY` and `INNER JOIN`.

When all six feel easy, you can query any in-memory data (or database, through EF Core) with confidence — this is one of the most-used everyday skills in real C# jobs. Next week: **Async/Await & Concurrency** (Task vs Thread, deadlocks, cancellation).
