# XUnit & Mock Testing in ASP.NET Web API

## Step 1 — What is Testing?

Testing means checking that your code works correctly before it goes live.

**Unit Testing** = testing the smallest part of code (a method, service, or controller action) alone, without other parts.

**Why?** Finds bugs early, keeps code quality high, makes refactoring safe.

**AAA Pattern** (how a test is written):

- **Arrange** – set up test data
- **Act** – call the method
- **Assert** – check the result

---

## Step 2 — xUnit vs Mock (Moq)

| xUnit                                         | Moq                                                           |
| --------------------------------------------- | ------------------------------------------------------------- |
| Framework to **write & run** tests            | It creates fake implementations of dependencies.              |
| Gives structure: `[Fact]`, `[Theory]`, Assert | Fakes DB, APIs, services so tests don't depend on them        |
| Answers: "Does this logic work?"              | Answers: "How do I test logic without the real database/API?" |

---

## Step 3 — Required Packages

```
Microsoft.NET.Test.Sdk
xunit
xunit.runner.visualstudio
```

Install via CLI:

```bash
dotnet add package Microsoft.NET.Test.Sdk
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
```

---

## Step 4 — Create Test Project

**Option A — CLI:**

```bash
dotnet new xunit -n MyWebAPI.Tests
dotnet sln add MyWebAPI.Tests
dotnet add MyWebAPI.Tests reference MyWebAPI
```

**Option B**

### How to Create a Test Project

![View > Test Explorer](./Step-1_Create_Test_Project.png)

![View > Test Explorer](./Step-2_Create_Test_Project.png)

![View > Test Explorer](./Step_3_Create_Test_Project.png)

![View > Test Explorer](./Step_4_Create_Test_Project.png)

---

## Step 5 — Writing xUnit Tests: [Fact], [Theory], [InlineData]

### 5.1 `[Fact]` — Simple Test (no parameters)

Controller:

```csharp
[HttpGet]
public string GetStudents()
{
    return $"Age: 18, City: Rajkot";
}
```

**Creating the test file (Right-Click):**

1. Right-click the `MyWebAPI.Tests` project → **Add** → **Class**
2. Name it `StudentControllerTest.cs` → **Add**
3. Write the test method inside this class (shown below)

Test — checking the **return type**:

```csharp
[Fact]
public void GetStudent_ReturnOkResult()
{
    var controller = new StudentController();
    var result = controller.GetStudents();

    Assert.IsType<string>(result);
}
```

---

### 5.2 `[Theory]` + `[InlineData]` — Test with Multiple Inputs

Used when you want to run the **same test logic** with **different input values**.

Service:

```csharp
public class CalculatorService
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

Test:

```csharp
using Testing.Services;

namespace Testing.Tests
{
    public class CalculatorServiceTests
    {
        [Theory]
        [InlineData(10, 20, 30)]
        [InlineData(5, 5, 10)]
        [InlineData(-10, 20, 10)]
        [InlineData(0, 0, 0)]
        public void Add_ReturnsCorrectResult(int a, int b, int expected)
        {
            var service = new CalculatorService();

            var result = service.Add(a, b);

            Assert.Equal(expected, result);
        }
    }
}
```

### How to Check Testing

## Step - 1

![View > Test Explorer](./TestingInExplorer.png)

## Step - 2 All Test Case Passed

![Run ](./Testing.png)

## Step - 3 One Test Case Failed

![Failed ](./Failed.png)

## Step 6 — Other Common Assert Methods Example

```csharp
public class StudentController
{
    // Used with Assert.StrictEqual
    [HttpGet]
    public double GetPrice()
    {
        return 100.00;
    }

    // Used with Assert.True
    [HttpGet]
    public bool IsStudentActive(int id)
    {
        return id > 0;
    }

    // Used with Assert.Throws
    [HttpGet]
    public Student GetStudentById(int id)
    {
        if (id <= 0)
            throw new ArgumentException("Invalid Student Id");

        return new Student { Id = id, Name = "Rahul" };
    }

    // Used with Assert.NotNull
    [HttpGet]
    public Student GetStudentByName(string name)
    {
        if (string.IsNullOrEmpty(name))
            return null;

        return new Student { Id = 1, Name = name };
    }
}
```

**TestController**

```csharp
public class StudentControllerTest
{
    // Assert.StrictEqual — checks exact type AND value match
    [Fact]
    public void GetPrice_ReturnsExactTypeAndValue()
    {
        var controller = new StudentController();
        var result = controller.GetPrice();

        Assert.StrictEqual(100.00, result);
    }

    // Assert.True — checks a condition is true
    [Fact]
    public void IsStudentActive_ReturnsTrue_WhenIdIsValid()
    {
        var controller = new StudentController();
        var result = controller.IsStudentActive(5);

        Assert.True(result);
    }

    // Assert.Throws — checks calling the method throws an exception
    [Fact]
    public void GetStudentById_ThrowsArgumentException_WhenIdInvalid()
    {
        var controller = new StudentController();

        Assert.Throws<ArgumentException>(() => controller.GetStudentById(-1));
    }

    // Assert.NotNull — checks the result is not null
    [Fact]
    public void GetStudentByName_ReturnsNotNull_WhenNameIsValid()
    {
        var controller = new StudentController();
        var result = controller.GetStudentByName("Rahul");

        Assert.NotNull(result);
    }
}
```
