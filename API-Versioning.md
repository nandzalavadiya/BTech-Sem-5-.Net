
# What is Versioning

API versioning means updating your software without breaking the older versions that your users are already using.
<img width="1311" height="745" alt="Versioning Daigram" src="https://github.com/user-attachments/assets/7488f51f-855b-43d4-854c-8140487f80b8" />
![Versioning Daigram](<./Versioning Daigram.png>)

# 1. Type of Versioning

| #   | Strategy                      | Client Sends Version Via                                                     | Example                                                |
| --- | ----------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------ |
| 1   | URL Versioning                | The URL path itself                                                          | `GET /api/v1/students`                                 |
| 2   | Query String Versioning       | A query string parameter                                                     | `GET /api/products?api-version=1.0`                    |
| 3   | Header Versioning             | A custom request header                                                      | `GET /api/orders` + `X-Api-Version: 1.0`               |
| 4   | **Consumer-Based Versioning** | **Consumer ID; server determines the API version from the consumer mapping** | `GET /api/products` + `Consumer-Id: Client-A` → **V1** |

---

# 2. Install the Required NuGet Packages

Open a terminal in your project folder and run:

```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

---

# 3. Strategy 1 — URL Versioning

The version is part of the path: `/api/v1/students`, `/api/v2/students`.

**` Program.cs` **

```csharp
using Asp.Versioning;
using Asp.Versioning.ApiExplorer;

builder.Services.AddApiVersioning(options =>
{
     // Set the default API version to 1.0
    options.DefaultApiVersion = new ApiVersion(1, 0);

    // Use v1.0 if no API version is provided
    options.AssumeDefaultVersionWhenUnspecified = true;

    // Add supported/deprecated API versions to response headers
    options.ReportApiVersions = true;

    // Read version from URL: /api/v1/products
    options.ApiVersionReader = new UrlSegmentApiVersionReader();

})
.AddApiExplorer(options =>
{
    // Display API groups as v1, v2, v3, etc.
    options.GroupNameFormat = "'v'VVV";

    // Replace {version} in URL with actual version number
    options.SubstituteApiVersionInUrl = true;

});
```

**`Controllers/UrlVersioning/V1/StudentsController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace ApiVersioningDemo.Controllers.v1;

[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]/[action]")] // Route: api/v1/users
public partial class StudentsController  : ControllerBase
{
    [HttpGet]
    public IActionResult GetUser(int id)
    {
        var userV1 = new
        {
            id = id,
            full_name = "Jane Doe",
            contact_email = "jane@example.com"
        };
        return Ok(userV1);
    }
}
```

**`Controllers/UrlVersioning/V2/StudentsController.cs`**

```csharp
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace ApiVersioningDemo.Controllers.v2;

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]/[action]")] // Route: api/v2/users
public partial class StudentsController     : ControllerBase
{
    [HttpGet]
    public IActionResult GetUser1(string id)
    {
        var userV2 = new
        {
            id = $"usr_{id}",
            name = new { first_name = "Jane", last_name = "Doe" },

        };
        return Ok(userV2);
    }
}
```

**Test:**

**_ https://localhost:7117/api/v2/Students/GetUser _**

![Versioning Daigram](./URLV1.png)

**_ https://localhost:7117/api/v2/Students/GetUser _**

![Versioning Daigram](./URLV2.png)

---

# 4. Strategy 2 — Query String Versioning

The version is passed as `?api-version=1.0`. The route itself has **no** version token.
**` Program.cs` **

```csharp
using Asp.Versioning;
using Asp.Versioning.ApiExplorer;

builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;

    // Read version from URL: /api/v1/products
    options.ApiVersionReader = new UrlSegmentApiVersionReader();
    // Read version from Query: /api/products?api-version=1.0
    options.ApiVersionReader = new QueryStringApiVersionReader("api-version"); //New Line
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});

**`Controllers/QueryVersioning/ProductsController.cs`**
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace ApiVersioningDemo.Controllers;

[ApiController]
[Route("api/[controller]")] // Static URL path: api/users
public class ProductsController : ControllerBase
{

    [HttpGet]
    [ApiVersion("1.0")]
    public IActionResult GetV1()
    {
        return Ok(new
        {
            version = "1.0",
            data = new {
                 productId = 1, name = "Keyboard", price = 799 }
        });
    }

    [HttpGet]
    [ApiVersion("2.0")]
    public IActionResult GetV2()
    {
        return Ok(new
        {
            version = "2.0",
            data = new { productId = 1, name = "Keyboard", price = 799, currency = "INR" }
        });
    }
}
```

`**Test**
**_ Default _**
**_ https://localhost:7117/api/Products_**

![Query V1](./QueryDefault.png)

** https://localhost:7117/api/Products/?api-version=1.0 **

![Query V1](./QueryV1.png)

**\_ https://localhost:7117/api/Products?api-version=2.0**

![Query V2](./QueryV2.png)``

---

# 5. Strategy 3 — Header Versioning

Pass Version For Header Paramater `GET /api/orders` + `X-Api-Version: 1.0

**` Program.cs` **

```csharp
using Asp.Versioning;
using Asp.Versioning.ApiExplorer;

builder.Services.AddApiVersioning(options =>
{

    ...
    // Read version from Header: X-Version=1.0
    options.ApiVersionReader = new HeaderApiVersionReader("X-Version"); //New Line
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = false; //// Header versioning does not alter URL paths
});
```

**`Controllers/HeaderVersioning/UsersController.cs`**
namespace ApiVersioningDemo.Controllers;

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
	[HttpGet]
	[MapToApiVersion("1.0")]
	public IActionResult GetV1()
	{
		return Ok(new { id = 101, full_name = "Jane Doe", contact_email = "jane@example.com" });
	}

	[HttpGet]
	[MapToApiVersion("2.0")]
	public IActionResult GetV2()
	{
		return Ok(new { id = "usr_101", name = new { first_name = "Jane", last_name = "Doe" }, email = "jane@example.com" });
	}
}
```

`**Test**

** `GET /api/orders` + `X-Api-Version: 1.0 **

![Query V1](./HeaderV1.png)
 
** `GET /api/orders`+`X-Api-Version: 2.0 **

![Query V2](./HeaderV2.png)``
