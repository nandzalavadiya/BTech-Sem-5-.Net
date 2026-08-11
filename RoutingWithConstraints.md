# Routing With Constraints in ASP.NET Core Web API

## 1. What Is a Route Constraint?

A route constraint checks whether a URL value has the expected format before the action method is selected.

Example:

```http
GET /api/students/10
```

If the route is:

```csharp
[HttpGet("{id:int}")]
```

then `10` is accepted because it is an integer.

But this request:

```http
GET /api/students/abc
```

does not match the route and returns **404 Not Found**.

---

## 2. Why Use Route Constraints?

Route constraints help us:

- Accept only valid values in the URL.
- Avoid calling the wrong action method.
- Make API routes clear.
- Reduce unnecessary validation inside action methods.

---

## 3. Basic Controller Setup

Create a controller:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace EFCoreCrudDemo.Controllers;

[ApiController]
[Route("api/[controller]")]
public class RouteDemoController : ControllerBase
{
}
```

---

## 4. `int` Constraint

Accepts only integer values.

```csharp
[HttpGet("student/{id:int}")]
public IActionResult GetStudent(int id)
{
    return Ok($"Student Id: {id}");
}
```

### Test

Valid:

```http
GET /api/RouteDemo/student/10
```

Invalid:

```http
GET /api/RouteDemo/student/abc
```

The invalid request returns **404 Not Found** because `abc` is not an integer.

---

## 5. `bool` Constraint

Accepts only `true` or `false`.

```csharp
[HttpGet("active/{isActive:bool}")]
public IActionResult GetActiveStudents(bool isActive)
{
    return Ok($"Is Active: {isActive}");
}
```

### Test

```http
GET /api/RouteDemo/active/true
```

```http
GET /api/RouteDemo/active/false
```

---

## 6. `min` Constraint

Accepts a number greater than or equal to the specified value.

```csharp
[HttpGet("minimum/{id:int:min(1)}")]
public IActionResult GetMinimum(int id)
{
    return Ok($"Id: {id}");
}
```

### Test

Valid:

```http
GET /api/RouteDemo/minimum/1
```

Invalid:

```http
GET /api/RouteDemo/minimum/0
```

---

## 7. `max` Constraint

Accepts a number less than or equal to the specified value.

```csharp
[HttpGet("maximum/{id:int:max(100)}")]
public IActionResult GetMaximum(int id)
{
    return Ok($"Id: {id}");
}
```

### Test

Valid:

```http
GET /api/RouteDemo/maximum/100
```

Invalid:

```http
GET /api/RouteDemo/maximum/101
```

---

## 8. `range` Constraint

Accepts a number within a specified range.

```csharp
[HttpGet("age/{age:int:range(18,60)}")]
public IActionResult GetByAge(int age)
{
    return Ok($"Age: {age}");
}
```

### Test

Valid:

```http
GET /api/RouteDemo/age/25
```

Invalid:

```http
GET /api/RouteDemo/age/17
```

---

## 9. `alpha` Constraint

Accepts only alphabetic characters.

```csharp
[HttpGet("name/{name:alpha}")]
public IActionResult GetByName(string name)
{
    return Ok($"Name: {name}");
}
```

### Test

Valid:

```http
GET /api/RouteDemo/name/Nand
```

Invalid:

```http
GET /api/RouteDemo/name/Nand123
```

---

## 10. `minlength` Constraint

Accepts text with at least the specified number of characters.

```csharp
[HttpGet("minimum-name/{name:minlength(3)}")]
public IActionResult GetMinimumName(string name)
{
    return Ok($"Name: {name}");
}
```

### Test

Valid:

```http
GET /api/RouteDemo/minimum-name/Nand
```

Invalid:

```http
GET /api/RouteDemo/minimum-name/AB
```

---

## 11. `maxlength` Constraint

Accepts text with no more than the specified number of characters.

```csharp
[HttpGet("maximum-name/{name:maxlength(10)}")]
public IActionResult GetMaximumName(string name)
{
    return Ok($"Name: {name}");
}
```

---

## 12. Complete Demo Controller

```csharp
using Microsoft.AspNetCore.Mvc;

namespace EFCoreCrudDemo.Controllers;

[ApiController]
[Route("api/[controller]")]
public class RouteDemoController : ControllerBase
{
    [HttpGet("student/{id:int}")]
    public IActionResult GetStudent(int id)
    {
        return Ok($"Student Id: {id}");
    }

    [HttpGet("active/{isActive:bool}")]
    public IActionResult GetActiveStudents(bool isActive)
    {
        return Ok($"Is Active: {isActive}");
    }

    [HttpGet("age/{age:int:range(18,60)}")]
    public IActionResult GetByAge(int age)
    {
        return Ok($"Age: {age}");
    }

    [HttpGet("name/{name:alpha}")]
    public IActionResult GetByName(string name)
    {
        return Ok($"Name: {name}");
    }

    [HttpGet("minimum-name/{name:minlength(3)}")]
    public IActionResult GetMinimumName(string name)
    {
        return Ok($"Name: {name}");
    }
}
```

---

## 13. Important Difference

### Route Constraint

Checks the value in the **URL** and decides whether the route matches.

```csharp
[HttpGet("{id:int}")]
```

Invalid URL value normally returns:

```http
404 Not Found
```

### FluentValidation

Checks request data, usually from the **request body**.

Invalid request data normally returns:

```http
400 Bad Request
```

---

## 14. Summary

| Constraint | Purpose | Example |
|---|---|---|
| `int` | Integer only | `{id:int}` |
| `bool` | `true` or `false` | `{isActive:bool}` |
| `min` | Minimum numeric value | `{id:int:min(1)}` |
| `max` | Maximum numeric value | `{id:int:max(100)}` |
| `range` | Numeric range | `{age:int:range(18,60)}` |
| `alpha` | Letters only | `{name:alpha}` |
| `minlength` | Minimum text length | `{name:minlength(3)}` |
| `maxlength` | Maximum text length | `{name:maxlength(10)}` |

> **Key Point:** Route constraints validate the URL path. They are not a replacement for full request validation.
