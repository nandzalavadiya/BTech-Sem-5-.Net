# Model Binding (Query, Route, Body, Form, Header)

---

## 1. What is Model Binding?

###

**Model Binding** is the process where ASP.NET Core automatically takes data from an incoming HTTP request and converts it into C# parameters/objects for your controller action — so you don't have to manually read the request yourself.

### Where can data come from?

| Source    | Example                         |
| --------- | ------------------------------- |
| URL Query | `/api/users?age=20`             |
| URL Route | `/api/users/5`                  |
| Body      | JSON sent in a POST/PUT request |
| Form      | HTML form / file upload         |
| Header    | `Authorization: Bearer xyz`     |

---

## 2.Binding Attributes — Overview

| Attribute      | Reads From                        | Common Use                            |
| -------------- | --------------------------------- | ------------------------------------- |
| `[FromQuery]`  | Query string (`?key=value`)       | Filters, search, paging               |
| `[FromRoute]`  | Route/URL segment (`{id}`)        | Identifying a specific resource       |
| `[FromBody]`   | Request body (usually JSON)       | Sending a full object (Create/Update) |
| `[FromForm]`   | Form data (`multipart/form-data`) | File uploads, HTML forms              |
| `[FromHeader]` | HTTP request headers              | Tokens, API keys, metadata            |


---

## 3. `[FromQuery]` — Query String

### What it does

Reads values that come **after `?`** in the URL.

```
GET /api/students?age=20&city=Rajkot
```

### Example

```csharp
[HttpGet]
public IActionResult GetStudents([FromQuery] int age, [FromQuery] string city)
{
    return Ok($"Age: {age}, City: {city}");
}
```
**Test**

<img width="1738" height="812" alt="FromQuery" src="https://github.com/user-attachments/assets/57ca8217-23ef-4331-a18b-39dd734effde" />


---

## 4. `[FromRoute]` — Route Parameter

### What it does

Reads values that are part of the **URL path itself**, defined in the route template with `{ }`.

```
GET /api/students/1
```

### Example

```csharp
[HttpGet("{id}")]
public IActionResult GetStudent([FromRoute] int id)
{
    return Ok($"Student Id: {id}");
}
```
**Test**

<img width="1731" height="807" alt="FromRoute" src="https://github.com/user-attachments/assets/d67995c7-c2f5-442e-9e1a-ba0313a5376b" />

---

## 5. `[FromBody]` — Request Body

### What it does

Reads data from the **body of the request**, usually sent as **JSON**. Used when the client sends a full object — most common in `POST` and `PUT`.

### Example

```csharp
public class CreateStudentDto
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

```csharp
[HttpPost]
public IActionResult CreateStudent([FromBody] CreateStudentDto dto)
{
    return Ok($"Created: {dto.Name}, Age {dto.Age}");
}
```

Request body (JSON):

```json
{
  "name": "Ravi",
  "age": 21
}
```

**Test**
<img width="1729" height="777" alt="FromBody" src="https://github.com/user-attachments/assets/d6229921-c27e-41fe-b5c9-797b23d1eb23" />

---

## 6. `[FromForm]` — Form Data

### What it does

Reads data sent as **form data** (`multipart/form-data` or `application/x-www-form-urlencoded`), instead of JSON. This is the standard way to handle **file uploads**.

### Example — Simple Form

```csharp
[HttpPost("register")]
public IActionResult Register([FromForm] string username, [FromForm] string password)
{
    return Ok($"Registered: {username}");
}
```

```

### `[FromBody]` vs `[FromForm]`

|                 | `[FromBody]`           | `[FromForm]`                |
| --------------- | ---------------------- | --------------------------- |
| Content-Type    | `application/json`     | `multipart/form-data`       |
| Can send files? | ❌ No                  | ✅ Yes                      |
| Typical use     | Create/Update via JSON | File uploads, HTML `<form>` |

---
```

## 7. `[FromHeader]` — HTTP Headers

### What it does

Reads a value directly from the **HTTP request headers** — commonly used for tokens, API keys, or metadata like language/version.

### Example

```csharp
[HttpGet("secure-data")]
public IActionResult GetSecureData([FromHeader(Name = "Authorization")] string authToken)
{
    return Ok($"Token received: {authToken}");
}
```

Request header:

```
Authorization: Bearer abc123xyz
```
**Test**

<img width="1737" height="761" alt="FromHeader" src="https://github.com/user-attachments/assets/8f6a4bd6-f977-4129-8422-46119016cf02" />

---

## 8. Full Example — All Together in One Controller

```csharp
using Microsoft.AspNetCore.Mvc;

namespace StudentProjectAPI.Controllers;

[ApiController]
[Route("api/[controller]")]
public class StudentsController : ControllerBase
{
    // FromQuery: /api/students?age=20
    [HttpGet]
    public IActionResult GetAll([FromQuery] int? age)
    {
        return Ok($"Filtering by age: {age}");
    }

    // FromRoute: /api/students/5
    [HttpGet("{id}")]
    public IActionResult GetById([FromRoute] int id)
    {
        return Ok($"Student Id: {id}");
    }

    // FromBody: JSON in request body
    [HttpPost]
    public IActionResult Create([FromBody] CreateStudentDto dto)
    {
        return Ok($"Created: {dto.Name}");
    }

    // FromForm: file upload
    [HttpPost("{id}/photo")]
    public IActionResult UploadPhoto([FromRoute] int id, [FromForm] IFormFile photo)
    {
        return Ok($"Photo for student {id}: {photo.FileName}");
    }

    // FromHeader: reading a custom header
    [HttpGet("check")]
    public IActionResult Check([FromHeader(Name = "X-Client-Version")] string version)
    {
        return Ok($"Client version: {version}");
    }
}

```
