# FluentValidation — Part 2 (Advanced)

**Covers:** `DependentRules()`, `InclusiveBetween()`, `ExclusiveBetween()`, `CreditCard()`, `IsInEnum()`, `Matches()`, `EmailAddress()`, `RuleForEach()` (lists/arrays), `MustAsync()`

---

## 1.Student Model

```csharp
namespace EFCoreCrudDemo.Dtos;

public class StudentDto
{
    public int StudentId { get; set; }
    public string StudentName { get; set; } = string.Empty;
    public string StudentEmail { get; set; } = string.Empty;
    public int DepartmentId { get; set; }
    public int Age { get; set; }

    public int RoleId { get; set; }              // for IsInEnum()
    public string CardNumber { get; set; } = string.Empty; // for CreditCard()
    public List<string> Courses { get; set; } = new();      // for RuleForEach()
}

public enum UserRole
{
    Admin = 1,
    Faculty = 2,
    Student = 3
}
```

---

## 2. `InclusiveBetween()` / `ExclusiveBetween()`

Both check that a number falls within a range — the difference is whether the **boundary values** are accepted.

### `InclusiveBetween(min, max)` — boundaries **included**

```csharp
RuleFor(student => student.Age)
    .InclusiveBetween(18, 25)
    .WithMessage("Age must be between 18 and 25.");
```

`18` ✅ `25` ✅ `26` ❌

### `ExclusiveBetween(min, max)` — boundaries **excluded**

```csharp
RuleFor(student => student.Age)
    .ExclusiveBetween(18, 25)
    .WithMessage("Age must be strictly between 18 and 25.");
```

`18` ❌ `20` ✅ `25` ❌

---

## 3. `Matches()` — Regex Validation

Checks a string against a regular expression pattern.

```csharp
RuleFor(student => student.StudentName)
    .Matches(@"^[A-Z]")
    .WithMessage("Student name must start with a capital letter.");
```

| Input    | Result     |
| -------- | ---------- |
| `"Nand"` | ✅ Valid   |
| `"nand"` | ❌ Invalid |

You can use `Matches()` for things like phone numbers, postal codes, or usernames — any pattern-based rule.

---

## 4. `EmailAddress()`

Checks that a string is a valid email format.

```csharp
RuleFor(student => student.StudentEmail)
    .NotEmpty().WithMessage("Student email is required.")
    .EmailAddress().WithMessage("Please enter a valid email address.");
```

| Input                | Result     |
| -------------------- | ---------- |
| `"nand@example.com"` | ✅ Valid   |
| `"wrong-email"`      | ❌ Invalid |

---

## 5. `CreditCard()`

Checks that a string is a **structurally valid** credit card number (correct format/checksum) — it does **not** check whether the card is real or active.

```csharp
RuleFor(student => student.CardNumber)
    .CreditCard()
    .WithMessage("Please enter a valid card number.");
```

---

## 6. `IsInEnum()`

Ensures a value matches one of the defined members of a C# enum.

```csharp
public enum UserRole
{
    Admin = 1,
    Faculty = 2,
    Student = 3
}

RuleFor(student => student.Role)
    .IsInEnum()
    .WithMessage("Role must be Admin, Faculty, or Student.");
```

| Input           | Result     |
| --------------- | ---------- |
| `2` (Faculty)   | ✅ Valid   |
| `9` (undefined) | ❌ Invalid |

---

## 7. `RuleForEach()` — Validating Lists and Arrays

`RuleFor()` validates a single property. `RuleForEach()` validates **every item inside a collection**.

```csharp
RuleForEach(student => student.Courses)
    .NotEmpty().WithMessage("Course name cannot be empty.")
    .MaximumLength(30).WithMessage("Course name is too long.");
```

**Example input:**

```json
{
  "courses": ["Math", "", "Advanced Physics And Chemistry Combined Studies"]
}
```

**Result:** two separate errors — one for the empty string at index 1, one for the too-long name at index 2. Each error reports its own index, so the client knows exactly which item failed:

```json
[
  {
    "propertyName": "Courses[1]",
    "errorMessage": "Course name cannot be empty."
  },
  { "propertyName": "Courses[2]", "errorMessage": "Course name is too long." }
]
```

---

## 8. `DependentRules()` — Conditional Chains

`DependentRules()` lets you say: _"only run these rules if the rule(s) above succeeded."_ It's a cleaner alternative to `Cascade(CascadeMode.Stop)` when you want to group a **block** of dependent checks together.

```csharp
RuleFor(student => student.StudentEmail)
    .NotEmpty().WithMessage("Student email is required.")
    .DependentRules(() =>
    {
        RuleFor(student => student.StudentEmail)
            .EmailAddress()
            .WithMessage("Please enter a valid email address.");
    });
```

**How it behaves:**

- If `StudentEmail` is empty → only the "required" error shows. The `EmailAddress()` check inside `DependentRules()` is **skipped**.
- If `StudentEmail` has a value → the "required" check passes, so FluentValidation proceeds to check `EmailAddress()` too.

**Difference from `Cascade(CascadeMode.Stop)`:**
| Feature | `Cascade(Stop)` | `DependentRules()` |
|---|---|---|
| Scope | Stops all rules on **that property** | Groups rules that depend on the ones above |
| Best for | A simple top-to-bottom stop | Nesting a related block of follow-up checks |

---

## 9. `MustAsync()` — Asynchronous Custom Validation

```csharp
RuleFor(student => student.StudentEmail)
    .MustAsync(async (dto, email, cancellation) =>
 {
     // Allow the user's own current email to pass on Update
     bool exists = await _context.Student
         .AnyAsync(u => u.Email == email, cancellation);
     return !exists;
 })
    .WithMessage("This email is already registered.");
```


---

## 10. Full Example — Extended Validator

```csharp
public class StudentValidator : AbstractValidator<StudentDto>
{
    public StudentValidator()
    {


        RuleFor(student => student.StudentName)
            .Matches(@"^[A-Z]")
            .WithMessage("Student name must start with a capital letter.");

        RuleFor(student => student.StudentEmail)
            .NotEmpty().WithMessage("Student email is required.")
            .DependentRules(() =>
            {
                RuleFor(student => student.StudentEmail)
                    .EmailAddress().WithMessage("Please enter a valid email address.")
                    .MustAsync(async (email, cancellation) =>
                    {
                        bool exists = await _studentRepository.EmailExistsAsync(email);
                        return !exists;
                    })
                    .WithMessage("This email is already registered.");
            });

        RuleFor(student => student.Age)
            .InclusiveBetween(18, 25)
            .WithMessage("Age must be between 18 and 25.");

        RuleFor(student => student.Role)
            .IsInEnum()
            .WithMessage("Role must be Admin, Faculty, or Student.");

        RuleFor(student => student.CardNumber)
            .CreditCard()
            .WithMessage("Please enter a valid card number.");

        RuleForEach(student => student.Courses)
            .NotEmpty().WithMessage("Course name cannot be empty.")
            .MaximumLength(30).WithMessage("Course name is too long.");
    }
}
```

---
