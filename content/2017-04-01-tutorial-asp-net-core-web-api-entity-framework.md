---
title: "Tutorial: ASP.NET Core Web API, 1 - Entity Framework"
author: "Nicklas Møller Jepsen"
date: "2017-04-01"
url: "/2017/04/01/tutorial-asp-net-core-web-api-entity-framework/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - api
  - webapi
  - entityframework
  - di
  - moq
  - ef
  - aspnet
  - aspnetcore 
---
## Tutorial: ASP.NET Core Web API, part 1
#### Entity Framework
This post extends my previous post on how to create a ASP.NET WebApi, please see: [Create a WebApi REST Service Hosted on Azure](http://systemout.net/2016/08/12/create-a-webapi-rest-service-hosted-on-azure/).

If you want to jump right in, then feel free to get the source for this tutorial [here on Github](https://github.com/nicklasjepsen/ProfileApiSample).

#### High-level solution description
We are going to create a Profile API for managing people profile in a database using technoligies such as ASP.NET Core, ASP.NET Web API, Entity Framework, MVC, Moq for testing and patterns like dependency injection and inversion of control.

The end result will be a solution with the following project upon completion of all tutorials:

* ProfileApi.WebApi (the actual web, this tutorial)
* ProfileApi.WebUi (a front end for interacting with the api, future post)
* ProfileApi.Tests (a unit test project for ensure high quality of our code)

*Note: This tutorial is assuming you are using Visual Studio 2017*

### Setting up the Solution
Start by creating a new solution and project. 
Important: Create a **ASP.NET Core Web Application (.NET Framework)**, not the (.NET Core) version. We are going to host on the Windows platform, so no reason to limit us to the minimal .NET Core version of the libraries available.

![New Solution](http://systemout.net/images/ProfileApi/NewSolution.png)

Now you get a dialog where you select the web project type. Select Web API and make sure No Authentication is selected:

![New Web API Project](http://systemout.net/images/ProfileApi/NewWebApiProject.png)

Here is how the solution/project structure should look:

![Solution Structure](http://systemout.net/images/ProfileApi/InitialSolution.png)

* In the Package Manager Console, run this command: `Install-Package Microsoft.EntityFrameworkCore.SqlServer` to install the EF package to the project

### Create the Code First Model
Now it's time to add some model classes that we will use as proxies between our tables and our code. In this tutorial I am using a code first approachs which means that we will model the database using C# and then have EF create the database and tables by using migrations.
We could just as well have choosen to use the "old school" EDMX approach, but code first is more fun!

* In solution explorer select your project, and add folder named Models
  * Select the Models folder, add a new class (Ctrl+Shift+C) name it Gender (people need to have a gender!)

Here's how it should look:

```csharp
public class Gender
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```


* Now add a new class to the Models folder; Person.cs:


```csharp
public class Person
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public int GenderId { get; set; }
    public DateTime TimeCreated { get; set; }

    public Gender Gender { get; set; }
}
```

Pretty basic stuff, but one thing to note: we're creating a 1-to-many relation between Person and Gender, that is a Person can only have one Gender. 

### Creating the Context

Now we need to add a context class. This class will be used to create the database from and will also serve as the access point to the database from our application. We will also inject our context class in the pipeline for easy dependency injection.

* Add a folder to the root of the project, name it Data
  * A a new class, ApplicationDbContext.cs

We are going to add a DbSet for each of our new models. Then we are going to inject `DbContextOptions` in the constructor and finally we are going to singularize the table names by overriding the `OnModelCreating` method.

Make sure to import the namespace `Microsoft.EntityFrameworkCore` in the ApplicationDbContext class and then paste in the below code:

```csharp
public class ApplicationDbContext : DbContext
{
    public DbSet<Person> People { get; set; }
    public DbSet<Gender> Genders { get; set; }

    public ApplicationDbContext(DbContextOptions options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // EF will use names of the DbSets to determine the table name.
        // Here we use the singular form of the object in stead
        modelBuilder.Entity<Person>().ToTable(nameof(Person));
        modelBuilder.Entity<Gender>().ToTable(nameof(Gender));
    }
}
```


* Add a new class, DbInitializer.cs in the Data folder
This class is used to create the database (if it doesn't exist) and to seed the data with some default data.
* Make sure to import `Microsoft.EntityFrameworkCore` in the DbInitializer class and then paste in the below code:

```csharp
public static void Initialize(ApplicationDbContext context)
{
    // Using EF migrations to a. Create the DB if not exist or b. migrate the tabase fit exists 
    context.Database.Migrate();

    // Only seed database if it's empty
    if (context.People.Any())
        return;

    var genders = new[]
    {
        new Gender
        {
            Name = "Male"
        },
        new Gender
        {
            Name = "Female"
        },
        new Gender
        {
            Name = "Other"
        }
    };
    context.Genders.AddRange(genders);

    var people = new[]
    {
        new Person
        {
            GenderId = 1,
            FirstName = "Nicklas",
            LastName = "Møller Jepsen",
            Email = "nicklas.m.jepsen@gmail.com",
            TimeCreated = DateTime.UtcNow
        },
        new Person
        {
            GenderId = 2,
            FirstName = "Holly",
            LastName = "Molly",
            Email = "holly.molly@gmail.com",
            TimeCreated = DateTime.UtcNow
        }
    };
    context.People.AddRange(people);

    context.SaveChanges();
}
```

#### Add DefaultConnection to appsettings.json
Finally we need to add connection string we are using to connect to the database to the appsettings.json. Here's how that file should look when you're done:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ProfileApi;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  }
}
```

*Note: We are using a built in SQL Server. Later, when we need to publish our app, we can easily change the connection string to point to a real SQL Server.*

### Hooking up the Moving Parts
Now we need to hook up our context with the pipelin. This is done in the Startup.cs class.

* In `ConfigureServices` add the following line to the beginning of the method:

```csharp
services.AddDbContext<ApplicationDbContext>(
	options => options.UseSqlServer(
		Configuration.GetConnectionString("DefaultConnection")));
```

This will inject our context to the pipeline so that it can be used from our application.

* In the Configure method, add `ApplicationDbContext personContext` to the constructor parameters
* In the `Configure` method, add this line at the end of the method:

```csharp
DbInitializer.Initialize(personContext);
```

### Creating the Database using Migrations
Now we are ready to create our database.
If this was a plain .NET Core application we could use a command prompt and run dotnet ef commands from there.
Since we are doing a full .NET application hosted on .NET Core, we can use the Package Manager Console to do our database migrations from within Visual Studio.
So, open the PMC and run the following:

`add-migration`

Give it a name: InitialCreate

Now run:
`update-database`

That's it, the database is created based on the model we created earlier. You can view the database from the SQL Server Object Explorer in Visual Studio. If you used the connection string above, you should look for a database named ProfileApi.

This sums up the first part of this series. In the next tutorial we will create a repository and a controller to access our newly created database. 

[Tutorial: ASP.NET Core Web API, part 2 - Repository and Controller](http://systemout.net/2017/04/01/tutorial-asp-net-core-web-api-controller-repository/)
 