# LAB 9 — EF Core Part 1

---

## 1. ORM (Object Relational Mapping)

### What is ORM?

**ORM = Object Relational Mapping**

It is a tool that lets you talk to a **database** using **C# code**, instead of writing raw SQL queries.

ORM connects your C# classes to database tables automatically.

### Why do we need it?

Without ORM, you write SQL manually:

```sql
SELECT * FROM Users WHERE Id = 1;
```

With ORM (EF Core), you write C#:

```csharp
var user = context.Users.FirstOrDefault(u => u.Id == 1);
```

Same result — but ORM lets you stay in C# the whole time.

---

## 2. EF Core Basics — Simple Explanation

### What is EF Core?

**EF Core (Entity Framework Core)** is Microsoft's ORM for C#. It lets you work with a database using C# classes and code, instead of writing SQL for every action.

### Core Ideas

EF Core has a few core ideas. Once you understand these, everything else in this course builds on top of them.

| Term           | Meaning                                               |
| -------------- | ----------------------------------------------------- |
| **Entity**     | A C# class that represents a table                    |
| **DbContext**  | The main class that connects your app to the database |
| **DbSet\<T\>** | A collection that represents one table                |
| **Migration**  | A version of your database schema                     |
| **LINQ**       | The C# way to query data (instead of SQL)             |

---

## 3. Approaches — Code-First vs Database-First

There are two ways to build the connection between your C# code and the database.

| Approach           | What you do first                                                        |
| ------------------ | ------------------------------------------------------------------------ |
| **Code-First**     | Write C# models → EF Core generates the database                         |
| **Database-First** | Design database first → EF Core generates C# models from existing tables |

### Example

**Code-First** (First write class → generate database):

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

**Database-First** (First Design database → generate classes):

```bash
dotnet ef dbcontext scaffold "YourConnectionString" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

---

## 4. Navigation Properties

### What are Navigation Properties?

A **Navigation Property** is a property in a model class that links one entity to another related entity — representing the same relationship that a **Primary Key (PK) → Foreign Key (FK)** creates in the database.

```csharp
public class Role
{
    public int Id { get; set; }
    public string Name { get; set; }

    // Navigation: A role can have many users
    public ICollection<User> Users { get; set; } = new List<User>();
}

public class User
{
    public int Id { get; set; }
    public string FullName { get; set; }

    public int RoleId { get; set; }

    // Navigation: A user belongs to one role
    public Role? Role { get; set; }
}
```

---

## 5. EF Core Packages

To use EF Core in a project, you need to install a few NuGet packages. Each one has a specific job.

```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

| Package               | Why                            |
| --------------------- | ------------------------------ |
| `EntityFrameworkCore` | Core ORM engine                |
| `.Design`             | Needed for migrations          |
| `.SqlServer`          | Talks to SQL Server            |
| `.Tools`              | Enables migration CLI commands |

---

## 6. AppDbContext

### What is AppDbContext?

`AppDbContext` is the main class that connects your C# code to the database. It tells EF Core which tables to work with.

This file is usually created **manually** — EF Core does not generate it for you.

### Purpose of This File

- Acts as a bridge between your models and the database
- Holds one `DbSet<T>` that represents a table
- Lets you configure small rules (like unique columns) using `OnModelCreating()`

### How to Create It

First Create `Data` Folder Then
Create a File Inside of `Data/AppDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using StudentProjectAPI.Models;

namespace StudentProjectAPI.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }

    public DbSet<User> Users => Set<User>();
    public DbSet<Role> Roles => Set<Role>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Unique constraint
        modelBuilder.Entity<User>()
            .HasIndex(u => u.Email)
            .IsUnique();

        modelBuilder.Entity<Role>()
            .HasIndex(r => r.Name)
            .IsUnique();
    }
}
```

### What's Inside It

| Part                                                   | Purpose                                         |
| ------------------------------------------------------ | ----------------------------------------------- |
| Constructor `(DbContextOptions<AppDbContext> options)` | Receives database connection settings           |
| `DbSet<T>`                                             | Represent Like a table                          |
| `OnModelCreating()`                                    | Extra rules (unique index, relationships, etc.) |

### Breaking Down `public DbSet<User> Users => Set<User>();`

```csharp
public DbSet<User> Users => Set<User>();
```

| Part                          | Meaning                        |
| ----------------------------- | ------------------------------ |
| `User` (inside `DbSet<User>`) | The Model Class (Entity)       |
| `Users`                       | The Table Name in the database |

**In short:** `DbSet<ModelClass> TableName => Set<ModelClass>();`

### OnModelCreating — Other Common Methods

Besides `IsUnique()`, there are a few other methods you'll commonly use inside `OnModelCreating()` to configure rules.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Unique constraint
    modelBuilder.Entity<User>()
        .HasIndex(u => u.Email)
        .IsUnique();
}
```

| Method                                | Purpose                       |
| ------------------------------------- | ----------------------------- |
| `IsUnique()`                          | No duplicate values allowed   |
| `HasMaxLength(n)`                     | Limit column length           |
| `IsRequired()`                        | Column cannot be null         |
| `HasOne().WithMany().HasForeignKey()` | Define relationship (PK → FK) |
| `HasDefaultValueSql()`                | Set a default DB value        |

### Inshort

**AppDbContext = the class that represents your whole database inside C#.**

---

## 7. Add Service in Program.cs

### Step 1: Add Connection String

In `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=StudentProjectDb;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true"
  }
}
```

### Step 2: Register AppDbContext in Program.cs

```csharp
using Microsoft.EntityFrameworkCore;
using StudentProjectAPI.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Register AppDbContext as a service
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

### Line-by-Line Breakdown

| Line                                                   | Meaning                                             |
| ------------------------------------------------------ | --------------------------------------------------- |
| `builder.Services.AddDbContext<AppDbContext>(...)`     | Registers `AppDbContext` so it can be used app-wide |
| `options.UseSqlServer(...)`                            | Tells EF Core to use SQL Server as the database     |
| `builder.Configuration.GetConnectionString("Default")` | Reads the connection string from `appsettings.json` |

---

## 8. Migration Commands

### What is a Migration?

A **Migration** is a version of your database schema. It records changes to your models so EF Core can apply them to the actual database.

### Step 0: Install EF Core CLI Tool (one-time)

```bash
dotnet tool install --global dotnet-ef
```

### Step 1: Create a Migration

```bash
dotnet ef migrations add InitialCreate
```

This generates a `Migrations/` folder with:

- `XXXXXXXXXXXXXX_InitialCreate.cs` — contains `Up()` and `Down()` methods
- `AppDbContextModelSnapshot.cs` — a snapshot of your current model(data model, tables, and relationships)

### Up() and Down() Example

```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Creates tables, columns, PKs, FKs, indexes
        migrationBuilder.CreateTable(name: "Roles", ...);
        migrationBuilder.CreateTable(name: "Users", ...);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Reverses everything (drops tables in reverse order)
        migrationBuilder.DropTable(name: "Users");
        migrationBuilder.DropTable(name: "Roles");
    }
}
```

| Method   | What it does                                        |
| -------- | --------------------------------------------------- |
| `Up()`   | Forward changes — "add this table, add that column" |
| `Down()` | Rollback — "undo what Up() did"                     |

### Step 2: Apply the Migration to the Database

```bash
dotnet ef database update
```

This runs `Up()` against your database, creating real SQL tables.

### Key Commands Reference

| Command                         | What it does                               |
| ------------------------------- | ------------------------------------------ |
| `dotnet ef migrations add Name` | Create a new migration                     |
| `dotnet ef database update`     | Apply pending migrations                   |
| `dotnet ef migrations list`     | Show all migrations                        |
| `dotnet ef migrations remove`   | Remove the last migration (if not applied) |
