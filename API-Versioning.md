# What is Versioning

API versioning means updating your software without breaking the older versions that your users are already using.

<img width="1311" height="745" alt="Versioning Daigram" src="https://github.com/user-attachments/assets/d2e5f75b-9d5e-4592-b800-baea837e457e" />

# 1. Type of Versioning
| # | Strategy | Client Sends Version Via | Example |
|---|---|---|---|
| 1 | URL Versioning | The URL path itself | `GET /api/v1/students` |
| 2 | Query String Versioning | A query string parameter | `GET /api/products?api-version=1.0` |
| 3 | Header Versioning | A custom request header | `GET /api/orders` + `X-Api-Version: 1.0` |
| 4 | **Consumer-Based Versioning** | **Consumer ID; server determines the API version from the consumer mapping** | `GET /api/products` + `Consumer-Id: Client-A` → **V1** |

---

# 2. Install the Required NuGet Packages

Open a terminal in your project folder and run:

```bash
dotnet add package Asp.Versioning.Http
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

---

# 3. Configure API Versioning in `Program.cs`

Open `Program.cs` and add the namespace:

```csharp
using Asp.Versioning;
using Asp.Versioning.ApiExplorer;
using Scalar.AspNetCore;
```

```csharp
builder.Services
    .AddApiVersioning(options =>
    {
        // Use v1.0 if no API version is provided
        options.AssumeDefaultVersionWhenUnspecified = true;

        // Set the default API version to 1.0
        options.DefaultApiVersion = new ApiVersion(1, 0);

        // Add supported/deprecated API versions to response headers
        options.ReportApiVersions = true;

        // Allow API version to be specified in multiple ways
        options.ApiVersionReader = ApiVersionReader.Combine(

            // Read version from URL: /api/v1/products
            new UrlSegmentApiVersionReader(),

            // Read version from query: ?api-version=1.0
            new QueryStringApiVersionReader("api-version"),

            // Read version from header: X-Api-Version: 1.0
            new HeaderApiVersionReader("X-Api-Version"),

            // Read version from Accept header: application/json;v=1.0
            new MediaTypeApiVersionReader("v")
        );
    })

    // Connect API versioning with MVC controllers
    .AddMvc()

    // Enable API version information for Swagger/OpenAPI
    .AddApiExplorer(options =>
    {
        // Display API groups as v1, v2, v3, etc.
        options.GroupNameFormat = "'v'VVV";

        // Replace {version} in URL with actual version number
        options.SubstituteApiVersionInUrl = true;
    });
```

---

# 4. Strategy 1 — URL Versioning

The version is part of the path: `/api/v1/students`, `/api/v2/students`.

**`Controllers/UrlVersioning/V1/StudentsController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers.UrlVersioning.V1;

[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/students")]
public class StudentsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new[]
        {
            new { studentId = 1, studentName = "Nand", studentEmail = "nand@gmail.com", departmentId = 1 }
        });
    }
}
```

**`Controllers/UrlVersioning/V2/StudentsController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers.UrlVersioning.V2;

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/students")]
public class StudentsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        // departmentId removed in V2
        return Ok(new[]
        {
            new { studentId = 1, studentName = "Nand", studentEmail = "nand@gmail.com" }
        });
    }
}
```

**Test:**

```http
GET /api/v1/students
GET /api/v2/students
```

---

# 5. Strategy 2 — Query String Versioning

The version is passed as `?api-version=1.0`. The route itself has **no** version token.

**`Controllers/QueryVersioning/V1/ProductsController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers.QueryVersioning.V1;

[ApiController]
[ApiVersion("1.0")]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() =>
        Ok(new[] { new { productId = 1, name = "Keyboard", price = 799 } });
}
```

**`Controllers/QueryVersioning/V2/ProductsController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers.QueryVersioning.V2;

[ApiController]
[ApiVersion("2.0")]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() =>
        Ok(new[] { new { productId = 1, name = "Keyboard", price = 799, currency = "INR" } });
}
```

**Test:**

```http
GET /api/products?api-version=1.0
GET /api/products?api-version=2.0
```

---

# 6. Strategy 3 — Header Versioning

The version travels in a custom request header, `X-Api-Version`. The URL never changes.

**`Controllers/HeaderVersioning/V1/OrdersController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers.HeaderVersioning.V1;

[ApiController]
[ApiVersion("1.0")]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() =>
        Ok(new[] { new { orderId = 101, status = "Pending" } });
}
```

**`Controllers/HeaderVersioning/V2/OrdersController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers.HeaderVersioning.V2;

[ApiController]
[ApiVersion("2.0")]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() =>
        Ok(new[] { new { orderId = 101, status = "Pending", trackingNumber = "TRK-5590" } });
}
```

**Test (cURL, since Scalar can also set custom headers per-request):**

```bash
curl -H "X-Api-Version: 1.0" https://localhost:PORT/api/orders
curl -H "X-Api-Version: 2.0" https://localhost:PORT/api/orders
```

---

![QueryVersioning&HeaderVersioning Daigram](<./Versioning(Header&Query).png>)
