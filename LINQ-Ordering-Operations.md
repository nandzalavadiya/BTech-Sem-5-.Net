# LINQ Ordering Operations — Student Management System

## 📊 Visualize First: Same Student List, 5 Real Diffrent Way

![Student Management System — Ordering Views](<./spms-ordering-mockup%20(2).svg>)

## The Model We'll Use

```csharp
public class Student
{
    public int RollNo { get; set; }
    public string Name { get; set; }
    public string ClassName { get; set; }
    public int Marks { get; set; }
}

List<Student> students = new List<Student>
{
    new Student { RollNo = 3, Name = "Riya",  ClassName = "10-A", Marks = 88 },
    new Student { RollNo = 1, Name = "Aman",  ClassName = "10-B", Marks = 91 },
    new Student { RollNo = 5, Name = "Kabir", ClassName = "10-A", Marks = 91 },
    new Student { RollNo = 2, Name = "Sneha", ClassName = "10-B", Marks = 76 },
    new Student { RollNo = 4, Name = "Zara",  ClassName = "10-A", Marks = 65 },
};
```

---

## 1️⃣ OrderBy — Ascending, Single Key

Sorts students by **Roll No.**, smallest first.

```csharp
var sorted = students.OrderBy(s => s.RollNo);
// Result: RollNo 1, 2, 3, 4, 5
```

**Use case:** Displaying the Student Data Roll No wise in Asceding Order.

---

## 2️⃣ OrderByDescending — Descending, Single Key

Sorts students Data by **Marks**, highest first — a leaderboard.

```csharp
var topScorers = students.OrderByDescending(s => s.Marks);
// Result: 91, 91, 88, 76, 65
```

**Use case:** Showing "Top Rank Students Data First" on a dashboard.

---

## 3️⃣ ThenBy — Secondary Ascending Key

`OrderBy`/`OrderByDescending` Sorts data by the secondary (next) key when the primary key values are the same.

```csharp
var byClassThenName = students
    .OrderBy(s => s.ClassName)   // 1st key
    .ThenBy(s => s.Name);        // 2nd key
// Groups by Class, then alphabetical within each class
```

**Use case:** Class-wise attendance sheet, names alphabetically ordered inside each class.

---

## 4️⃣ ThenByDescending — Secondary Descending Key

Same idea, but the goes highest → lowest.

```csharp
var byClassThenTopMarks = students
    .OrderBy(s => s.ClassName)         // group by class
    .ThenByDescending(s => s.Marks);   // best performer first, per class
```

**Use case:** Class-wise merit list — highest marks shown first _within_ each class.

---

## 5️⃣ Reverse — Flip, Don't Re-sort

`Reverse()` doesn't compare values — it just reverses the **current order** of the sequence.

```csharp
var recentFirst = students.OrderBy(s => s.RollNo).Reverse();
// Was: 1,2,3,4,5 → Now: 5,4,3,2,1
```

**Use case:** Showing the most recently admitted student (highest Roll No.) at the top, without writing a new sort.

---

