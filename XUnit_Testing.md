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

## Step 2 - Create an API Project and Add described methods
```csharp
[HttpGet]
public string GetStudents()
{
    return $"Age: 18, City: Rajkot";
}
```
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
---

## Step 3 — Create Test Project

**Option A — CLI:**

```bash
dotnet new xunit -n MyWebAPI.Tests
dotnet sln add MyWebAPI.Tests
dotnet add MyWebAPI.Tests reference MyWebAPI
```

**Option B**

### How to Create a Test Project

<img width="1247" height="915" alt="Step-1_Create_Test_Project" src="https://github.com/user-attachments/assets/1338565c-cffa-490e-88d7-ec84a393b864" />


<img width="1905" height="1011" alt="Step-2_Create_Test_Project" src="https://github.com/user-attachments/assets/2194aeaf-b988-4d69-86b2-d1b91cb91f54" />


<img width="1846" height="1036" alt="Step_3_Create_Test_Project" src="https://github.com/user-attachments/assets/1e621ec1-52cd-4bff-822d-98b564840722" />


<img width="972" height="677" alt="Step_4_Create_Test_Project" src="https://github.com/user-attachments/assets/9e3f361d-d345-416f-8b78-49680ee011d1" />


---
## Step 4 — Test Project Configuration

Install Required Packages

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
**Creating the test file (Right-Click):**

1. Right-click the `MyWebAPI.Tests` project → **Add** → **Class**
2. Name it `StudentControllerTest.cs` → **Add**
---

## Step 5 — Writing xUnit Tests: [Fact], [Theory], [InlineData]

[Fact] — Used when methods doesn't requires any parameters and only need to execute the test case once
[Theory] — Used when method have parameters and we want to test multiple time the same method with different data
[InlineData] OR [ClassData] - Provides testing data that we want to use for particular testing
Controller:

**Write the test method inside this class (shown below)**

In below example, we do check only the **return type** and method doesn't required any parameters, so we can use [Fact]:

```csharp
[Fact]
public void GetStudent_ReturnOkResult()
{
    var controller = new StudentController();
    var result = controller.GetStudents();

    Assert.IsType<string>(result);
}
```

In below example, we want to test **same test logic** with **different input values** so we will use the [Theory] with [InlineData] for providing different values

Test:

```csharp
using Testing.Services;

namespace Testing.Tests
{
    public class CalculatorServiceTests
    {
        [Theory] // Allows to run the test case method multiple time using different input provided by [InlineData]
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

<img width="1577" height="970" alt="TestingInExplorer" src="https://github.com/user-attachments/assets/f9e93adb-b8d6-4dee-abb3-012b61fa36be" />


## Step - 2 All Test Case Passed

<img width="1897" height="872" alt="Testing" src="https://github.com/user-attachments/assets/e8ace2a2-b52b-4d29-aa9e-9366cd66a70b" />


## Step - 3 One Test Case Failed

<img width="1867" height="881" alt="Failed" src="https://github.com/user-attachments/assets/2b15bcf1-bd7c-4c7e-a3de-9dfba1a7724a" />


## Step 5 — Other Common Assert Methods Example

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
