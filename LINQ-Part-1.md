# LINQ – Part 1

## 1. What is LINQ?

**LINQ** stands for **L**anguage **In**tegrated **Q**uery.

It is a feature in C# that lets you **query data** — from lists, arrays, databases, or XML — using one common, readable syntax, instead of writing loops and `if` conditions by hand.

Think of LINQ as giving C# a built-in "search and shape" toolkit for any collection of data.

```text
┌─────────────┐      ┌────────────┐      ┌────────────┐
│ Data Source │ ---> │ LINQ Query │ ---> │   Result   │
│ (List, DB…) │      │ (Where,    │      │ (filtered/ │
│             │      │  Select…)  │      │  shaped)   │
└─────────────┘      └────────────┘      └────────────┘
```

---

## 2. Why LINQ?

Without LINQ, filtering a list needs a manual loop:

```csharp
List<int> numbers = new List<int>() { 1, 2, 3, 4, 5, 6 };
List<int> result = new List<int>();

foreach (var n in numbers)
{
    if (n > 3)
    {
        result.Add(n);
    }
}
```

With LINQ, the same logic becomes **one line**:

```csharp
var result = numbers.Where(n => n > 3);
```

**Output (both cases):**

```text
4
5
6
```

LINQ code is shorter, easier to read, and less likely to contain bugs like off-by-one errors.

---

## 3. When to Use LINQ

Use LINQ whenever you need to:

- **Filter** a collection (only CE branch students, only passing students, etc.)
- **Select/shape** specific fields out of objects
- **Check conditions** (does any student have CPI > 9? do all students pass?)
- **Remove duplicates**, **take/skip** records
- Query a **database** through Entity Framework using the same syntax you use for lists

---

## 4. Method Syntax vs Query Syntax

LINQ can be written in two styles that produce the **same result**.

### Method Syntax (most common)

```csharp
var result = students
                .Where(s => s.Branch == "CE")
                .Select(s => s.Name);
```

### Query Syntax (looks like SQL)

```csharp
var result = from s in students
             where s.Branch == "CE"
             select s.Name;
```

**Output (both):**

```text
Aarav
Raj
Pooja
Vivek
```

### Comparison Table

| Aspect                         | Method Syntax                             | Query Syntax                                             |
| ------------------------------ | ----------------------------------------- | -------------------------------------------------------- |
| Style                          | Chained methods (`.Where()`, `.Select()`) | SQL-like keywords (`from`, `where`, `select`)            |
| Readability for SQL background | Less familiar                             | Very familiar                                            |
| Operator coverage              | Supports **all** LINQ operators           | Only supports a subset (`Any`, `Skip`, etc. are missing) |
| Used in real projects          | ✅ Very common                            | Occasionally, for simple                                 |

---

## 5. Lambda Expressions

A **lambda expression** is a short, unnamed function written inline.

```text
input => output
```

Read the `=>` as **"goes to"**. So `x => x * 2` means _"x goes to x times 2"_.

LINQ methods like `Where`, `Select`, and `Any` expect a small function to run on every item — a lambda is the quickest way to supply it.

### 5.1 Zero Parameters

Used when the function needs no input:

```csharp
() => Console.WriteLine("Hello!");

```

**Output:**

```text
Hello!
```

### 5.2 One Parameter

Parentheses are optional with a single parameter:

```csharp
students.Where(s => s.CPI > 8);
// same as:
students.Where((s) => s.CPI > 8);
```

### 5.3 Multiple Parameters

Parentheses are **required** with more than one parameter:

```csharp
(x, y) => x + y;

```

### 5.4 Multi-Statement Lambda

If the logic needs more than one line, use `{ }` and `return`:

```csharp
students.Select(s =>
{
    var grade = s.CPI >= 8 ? "A" : "B";
    return grade;
});
```

### Common Lambda Examples

| Goal                      | Lambda                  |
| ------------------------- | ----------------------- |
| Select student names      | `s => s.Name`           |
| Filter CE branch students | `s => s.Branch == "CE"` |
| Filter CPI greater than 8 | `s => s.CPI > 8`        |
| Check semester equals 5   | `s => s.Sem == 5`       |

---

## 6. IEnumerable vs IQueryable

```text
IEnumerable:  DB/List -> Fetch ALL records -> Filter IN MEMORY -> Result
IQueryable :  DB      -> Build query -> Filter IN THE DATABASE -> Fetch only needed rows -> Result
```

| Feature                        | `IEnumerable`                   | `IQueryable`                      |
| ------------------------------ | ------------------------------- | --------------------------------- |
| Works best with                | In-memory data (List, Array)    | Databases (Entity Framework)      |
| Namespace                      | `System.Collections.Generic`    | `System.Linq`                     |
| Performance on large DB tables | Slower (loads everything first) | Faster (loads only what's needed) |
| Typical use                    | `List<T>`, `T[]`                | `DbSet<T>` in EF Core             |

```csharp
// IEnumerable: all rows come to memory FIRST, then filter
IEnumerable<Student> students = context.Students.ToList();
var result = students.Where(s => s.Branch == "CE");

// IQueryable: filter is translated into SQL and runs on the server
IQueryable<Student> students = context.Students;
var result = students.Where(s => s.Branch == "CE");
```

---

## 7. Our Sample Data

Every example in this document uses this **same** `Student` model and list.

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Branch { get; set; }
    public int Sem { get; set; }
    public double CPI { get; set; }
    public List<string> Courses { get; set; }
}
```

```csharp
List<Student> students = new List<Student>()
{
    new Student { Id = 1,  Name = "Aarav",  Branch = "CE", Sem = 3, CPI = 8.5, Courses = new List<string> { "C#", "DBMS" } },
    new Student { Id = 2,  Name = "Neha",   Branch = "IT", Sem = 4, CPI = 9.1, Courses = new List<string> { "Java", "AI" } },
    new Student { Id = 3,  Name = "Raj",    Branch = "CE", Sem = 3, CPI = 7.8, Courses = new List<string> { "C#", "Math" } },
    new Student { Id = 4,  Name = "Priya",  Branch = "IT", Sem = 5, CPI = 8.9, Courses = new List<string> { "Python", "DBMS" } },
    new Student { Id = 5,  Name = "Kiran",  Branch = "ME", Sem = 2, CPI = 7.2, Courses = new List<string> { "CAD", "Physics" } },
    new Student { Id = 6,  Name = "Pooja",  Branch = "CE", Sem = 4, CPI = 8.3, Courses = new List<string> { "C#", "Data Structures" } },
    new Student { Id = 7,  Name = "Rahul",  Branch = "EC", Sem = 6, CPI = 7.9, Courses = new List<string> { "Signals", "IoT" } },
    new Student { Id = 8,  Name = "Sneha",  Branch = "IT", Sem = 3, CPI = 8.7, Courses = new List<string> { "Python", "Web" } },
    new Student { Id = 9,  Name = "Vivek",  Branch = "CE", Sem = 5, CPI = 6.9, Courses = new List<string> { "JavaScript", "DBMS" } },
    new Student { Id = 10, Name = "Anjali", Branch = "ME", Sem = 4, CPI = 8.1, Courses = new List<string> { "CAD", "Thermodynamics" } },
};
```

| Id  | Name   | Branch | Sem | CPI | Courses             |
| --- | ------ | ------ | --- | --- | ------------------- |
| 1   | Aarav  | CE     | 3   | 8.5 | C#, DBMS            |
| 2   | Neha   | IT     | 4   | 9.1 | Java, AI            |
| 3   | Raj    | CE     | 3   | 7.8 | C#, Math            |
| 4   | Priya  | IT     | 5   | 8.9 | Python, DBMS        |
| 5   | Kiran  | ME     | 2   | 7.2 | CAD, Physics        |
| 6   | Pooja  | CE     | 4   | 8.3 | C#, Data Structures |
| 7   | Rahul  | EC     | 6   | 7.9 | Signals, IoT        |
| 8   | Sneha  | IT     | 3   | 8.7 | Python, Web         |
| 9   | Vivek  | CE     | 5   | 6.9 | JavaScript, DBMS    |
| 10  | Anjali | ME     | 4   | 8.1 | CAD, Thermodynamics |

---

## 8. Projection Operators

Projection operators **reshape** each item into something new (a field, a new object, or a flattened list).

### 8.1 Select

**-** Transforms each item into a new shape (one field, or a new object).

**Syntax:**

```csharp
collection.Select(item => newShape);
```

**Example 1 — select only names:**

```csharp
var result = students.Select(s => s.Name);
```

**Output:**

```text
Aarav
Neha
Raj
Priya
Kiran
Pooja
Rahul
Sneha
Vivek
Anjali
```

**Example 2 — select a new anonymous object:**

```csharp
var result = students.Select(s => new { s.Name, s.CPI });
```

**Output:**

```text
Aarav - 8.5
Neha - 9.1
Raj - 7.8
...
```

---

### 8.2 SelectMany

**Syntax:**

```csharp
collection.SelectMany(item => item.InnerCollection);
```

**Example — get every course from every student:**

```csharp
var result = students.SelectMany(s => s.Courses);
```

**Output:**

```text
C#
DBMS
Java
AI
C#
Math
Python
DBMS
CAD
Physics
...
```

**How it works:** For each student, `SelectMany` takes their `Courses` list and **merges** it into one single sequence.

### Select vs SelectMany

```text
Select:                          SelectMany:

[C#, DBMS]                        C#
[Java, AI]         --->           DBMS
[C#, Math]                        Java
                                   AI
(list of lists)                   C#
                                   Math
                                (one flat list)
```

| Aspect       | `Select`                             | `SelectMany`                  |
| ------------ | ------------------------------------ | ----------------------------- |
| Output shape | One output per input (can be a list) | One flat, merged list         |
| Best for     | Picking/transforming fields          | Flattening nested collections |

---

## 9. Filtering Operators

Filtering operators reduce a collection down to only the items you need.

| Operator    | Purpose                                                  |
| ----------- | -------------------------------------------------------- |
| `Where`     | Keep items matching a condition                          |
| `OfType`    | Keep items of a specific type                            |
| `Take`      | Keep the first N items                                   |
| `TakeWhile` | Keep items while a condition is true, then stop          |
| `Skip`      | Skip the first N items                                   |
| `SkipWhile` | Skip items while a condition is true, then keep the rest |
| `Distinct`  | Remove duplicate values                                  |

### 9.1 Where

**-** Filters items by a Sepcific condition.

**Syntax:** `collection.Where(item => condition);`

```csharp
var result = students.Where(s => s.Branch == "CE");
```

**Output:**

```text
Aarav
Raj
Pooja
Vivek
```

**How it works:** `Where` checks the condition for **every** item, one by one, and keeps only the ones where the condition is `true`.

---

### 9.2 OfType

**-** Only Retrive a Specific Type of Data From a Collection

**Syntax:** `collection.OfType<Type>();`

```csharp
ArrayList values = new ArrayList() { "Aarav", 10, "Neha", 20, "Raj" };
var result = values.OfType<string>();
```

**Output:**

```text
Aarav
Neha
Raj
```

---

### 9.3 Take

**-** Returns the first N items.
**When:** You need a limited number of items from the start.

**Syntax:** `collection.Take(n);`

```csharp
var result = students.Take(3);
```

**Output:**

```text
Aarav
Neha
Raj
```

---

### 9.4 TakeWhile

**-** Returns items **while** a condition is true, and stops at the first failure.

**Syntax:** `collection.TakeWhile(item => condition);`

```csharp
var result = students.TakeWhile(s => s.CPI >= 8);
```

**Output:**

```text
Aarav
Neha
```

## **How it works:** Aarav (8.5) and Neha (9.1) pass, but Raj (7.8) fails the condition — so `TakeWhile`

### 9.5 Skip

**-** Ignores the first N items and returns the rest.

**Syntax:** `collection.Skip(n);`

```csharp
var result = students.Skip(8);
```

**Output:**

```text
Vivek
Anjali
```

---

### 9.6 SkipWhile

**-** Skips items **while** a condition is true, then returns everything from the first failure onward.

**Syntax:** `collection.SkipWhile(item => condition);`

```csharp
var result = students.SkipWhile(s => s.CPI >= 8);
```

**Output:**

```text
Raj
Priya
Kiran
Pooja
Rahul
Sneha
Vivek
Anjali
```

---

### 9.7 Distinct

**-** Removes duplicate values from a collection.

**Syntax:** `collection.Distinct();`

```csharp
var result = students.Select(s => s.Branch).Distinct();
```

**Output:**

```text
CE
IT
ME
EC
```

### Where vs TakeWhile vs SkipWhile

| Aspect                           | `Where`                                  | `TakeWhile`                 | `SkipWhile`                                           |
| -------------------------------- | ---------------------------------------- | --------------------------- | ----------------------------------------------------- |
| Checks every item?               | Yes, all items                           | No — stops at first `false` | No — stops checking at first `false`                  |
| Order-sensitive?                 | No                                       | Yes                         | Yes                                                   |
| Example on our data (`CPI >= 8`) | Aarav, Neha, Priya, Pooja, Sneha, Anjali | Aarav, Neha                 | Raj, Priya, Kiran, Pooja, Rahul, Sneha, Vivek, Anjali |

---

## 10. Element Operators

These return a **single element** from a sequence.

### 10.1 First

**-** Returns the first element (or first match).

```csharp
var result = students.First(s => s.Branch == "IT");
```

**Output:** `Neha`

If no match exists, `First` **throws an exception**: `InvalidOperationException: Sequence contains no matching element`.

### 10.2 FirstOrDefault

**-** Returns the first element (or match), or a default value (`null` for objects) if none exists.

```csharp
var result = students.FirstOrDefault(s => s.Branch == "ME2");
```

**Output:** `null`

### 10.3 Single

**-** Returns the **one and only** matching element.

```csharp
var result = students.Single(s => s.Id == 2);
```

**Output:** `Neha`

```csharp
// Unsafe: multiple CE students exist!
var result = students.Single(s => s.Branch == "CE");
```

**Output:** `Exception: Sequence contains more than one matching element`

### 10.4 SingleOrDefault

**-** Same as `Single`, but returns a default value instead of throwing when there's **no** match. Still throws if there's **more than one** match.

```csharp
var result = students.SingleOrDefault(s => s.Id == 50);
```

**Output:** `null`

### 🔑 Comparison Table: First vs FirstOrDefault vs Single vs SingleOrDefault

| Behavior                   | `First`                                | `FirstOrDefault`                       | `Single`                          | `SingleOrDefault`                 |
| -------------------------- | -------------------------------------- | -------------------------------------- | --------------------------------- | --------------------------------- |
| **Returns**                | First matching element                 | First matching element                 | The one and only matching element | The one and only matching element |
| **No match found**         | ❌ Throws exception                    | ✅ Returns `default` (`null`)          | ❌ Throws exception               | ✅ Returns `default` (`null`)     |
| **Multiple matches found** | ✅ Returns the first one, ignores rest | ✅ Returns the first one, ignores rest | ❌ Throws exception               | ❌ Throws exception               |

---

## 11. Quantifiers

Quantifiers return a **boolean** (`true`/`false`) describing the collection.

### 11.1 Any

**-** Checks whether **at least one** element exists (optionally matching a condition).

```csharp
bool result = students.Any(s => s.CPI > 9);
```

**Output:** `True`

### 11.2 All

**-** Checks whether **every** element satisfies a condition.

```csharp
bool result = students.All(s => s.CPI >= 6);
```

**Output:** `True`

```csharp
bool result = students.All(s => s.Branch == "CE");
```

**Output:** `False`

### 11.3 Contains

**-** Checks whether a specific value exists in a collection.

```csharp
List<string> branches = students.Select(s => s.Branch).Distinct().ToList();
bool result = branches.Contains("IT");
```

**Output:** `True`

---

## 12. Deferred Execution

LINQ queries are **not executed immediately** when you write them — they run only when you **iterate** the result (with `foreach`, `.ToList()`, `.Count()`, etc.).

```csharp
var query = students.Where(s => s.CPI > 8);   // NOT executed yet — just a "plan"

foreach (var s in query)                       // NOW it executes, sees the new student too
{
    Console.WriteLine(s.Name);
}
```

---

## 13. Practice Questions

Use the `students` list from all questions.

1. Get the names of all students in branch `"IT"`.
2. Get all students with `CPI > 8.5`.
3. Get the names of the first 4 students.
4. Skip the first 5 students and return the rest.
5. Get all unique branches in the list.
6. Get all courses of all students combined into one flat list (no duplicates).
7. Select each student's `Name` and `Sem` as a new anonymous object.
8. Find the first student whose branch is `"ME"`.
9. Find the first student whose branch is `"XY"` — return `null` instead of throwing.
10. Find the single student with `Id == 5`.
11. Try to use `Single` to find a student with `Branch == "CE"` — what happens, and why?
12. Check whether **any** student has `CPI < 7`.
13. Check whether **all** students have `Sem <= 6`.
14. Check whether the branch list contains `"EC"`.
15. Get students while their `CPI >= 8`, starting from the top of the list (use `TakeWhile`).
16. Skip students while `CPI >= 8`, then return the rest (use `SkipWhile`).
17. Get the number of students in branch `"CE"` (hint: combine `Where` with `Count()`).
18. Get students sorted is not required — just filter students in Semester `3` or `4` using `Where`.
19. Using `OfType<string>`, filter only string values from this list: `object[] mixed = { "A", 1, "B", 2.5, "C" };`

---
