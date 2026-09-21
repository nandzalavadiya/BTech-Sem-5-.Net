# Moq Testing in ASP.NET Web API

## Step 1 — What is Mocking / Moq?

**Mocking** means creating a **fake version** of a dependency (like a database, repository, or external API) so you can test your code **without** relying on the real thing.

**Moq** is a popular .NET library used to create these fake objects.

**Why mock?**

- Real dependencies (DB, APIs) are slow, unreliable, or unavailable during tests.
- You want to test **only your logic**, not the dependency's behavior.
- You can simulate specific scenarios — success, failure, empty data, exceptions — on demand.

---

## Step 2 Diffrence Between XUnit & Moq

| xUnit                                         | Moq                                                           |
| --------------------------------------------- | ------------------------------------------------------------- |
| Framework to **write & run** tests            | It creates fake implementations of dependencies.              |
| Gives structure: `[Fact]`, `[Theory]`, Assert | Fakes DB, APIs, services so tests don't depend on them        |
| Answers: "Does this logic work?"              | Answers: "How do I test logic without the real database/API?" |

## Step 3 — Install Moq Package

```bash
dotnet add package Moq
```

---

## Step 4 — In a API Controller

```csharp
    public interface IUserRepository
    {
        void Add(User user);
        void SaveChanges();
    }
```

```csharp
 public class UserRepository : IUserRepository
 {
     private readonly AppDbContext _context;

     public UserRepository(AppDbContext context)
     {
         _context = context;
     }
     public void Add(User user)
     {
         _context.Users.Add(user);
     }
     public void SaveChanges()
     {
         _context.SaveChanges();
     }
 }
```

```csharp
// Service depends on the INTERFACE, injected via constructor
 public class UserService
 {
     private readonly IUserRepository _userRepository;
     private readonly IConfiguration _config;

     public UserService(IUserRepository userRepository,IConfiguration config)
     {
         _userRepository = userRepository;
         _config = config;
     }
     public string Add(UserDto dto)
     {
         var entity = new User
         {
             UserTypeID = (int)dto.UserTypeID,
             FullName = dto.FullName,
             UserCode = dto.UserCode,
             Email = dto.Email,
             Password = dto.Password,
             MobileNumber = dto.MobileNumber,
             ProfilePicturePath = dto.ProfilePicturePath,
             IsActive = true,
             IsDeleted = false
         };

         _userRepository.Add(entity);
         _userRepository.SaveChanges();
         return ("Record Inserted");
     }
 }
```

```csharp
namespace Student_Project_Management_System.Controllers
{
    [ApiController]
    [Route("api/[controller]/[action]")]

    public class UserController : ControllerBase
    {
        private readonly AppDbContext _context;
        private readonly UserService _userService;
        public UserController(AppDbContext context,UserService userService)
        {
            _context = context;
            _userService = userService;
        }
        [HttpPost]
        public async Task<IActionResult> Add(UserDto dto)
        {
            var message = _userService.Add(dto);
            return Ok(new { Message = message });
        }
```

---

## Step 5 — In a Test Project Add a Class `StudentTest.cs`

```csharp
```csharp
public class UserServiceTests
{
    [Fact]
    public void Add_ReturnsSuccessMessage_AndCallsRepository()
    {
        // Arrange - Create fake repository
        var mockRepo = new Mock<IUserRepository>();

        // Create service and pass the fake repository
        var service = new UserService(mockRepo.Object, config: null);

        // Create test data
        var dto = new UserDto
        {
            UserTypeID = (UserTypeEnum)1,
            FullName = "Rahul Patel",
            UserCode = "STU001",
            Email = "rahul@test.com",
            Password = "Test@123",
            MobileNumber = "9999999999",
            ProfilePicturePath = null
        };

        // Act - Call the Add method
        var result = service.Add(dto);

        // Assert - Check that correct message is returned
        Assert.Equal("Record Inserted", result);

        // Check that repository Add method was called once
        mockRepo.Verify(
            repo => repo.Add(It.IsAny<User>()),
            Times.Once
        );

        // Check that SaveChanges method was called once
        mockRepo.Verify(
            repo => repo.SaveChanges(),
            Times.Once
        );
    }


    [Fact]
    public void Add_Should_Map_Dto_To_User()
    {
        // Arrange - Create fake repository
        var mockRepo = new Mock<IUserRepository>();

        // Create service using the fake repository
        var service = new UserService(
            mockRepo.Object,
            null,
            null
        );

        // Create test DTO
        var dto = new UserDto
        {
            UserTypeID = (UserTypeEnum)1,
            FullName = "Priya Shah",
            UserCode = "STU002",
            Email = "priya@test.com",
            Password = "Pass@456",
            MobileNumber = "8888888888",
            ProfilePicturePath = "/images/priya.png"
        };

        // Act - Send DTO to service
        service.Add(dto);

        // Assert - Check that DTO values are mapped to User entity
        mockRepo.Verify(
            x => x.Add(It.Is<User>(u =>
                u.FullName == "Priya Shah" &&
                u.UserCode == "STU002" &&
                u.Email == "priya@test.com" &&
                u.IsActive == true &&
                u.IsDeleted == false
            )),
            Times.Once
        );
    }
}
```
---

### Testing


<img width="1863" height="932" alt="Screenshot 2026-09-21 215841" src="https://github.com/user-attachments/assets/f7478d82-87d0-40ed-8a63-4b0f3f0d8254" />


<img width="1855" height="886" alt="Moq_Fail" src="https://github.com/user-attachments/assets/a3d69629-d1b9-4fc6-a3e3-d6ba92795302" />

