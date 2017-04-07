---
title: "Tutorial: ASP.NET Core Web API 2 - Controller and Repository"
author: "Nicklas Møller Jepsen"
date: "2017-04-01"
url: "/2017/04/01/tutorial-asp-net-core-web-api-controller-repository/"
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
  - repository
---
## Tutorial: ASP.NET Core Web API, part 2
#### Controller & Repository
This is the second post in the tutorial about ASP.NET Core Web API.
The [first post](http://systemout.net/2017/04/01/tutorial-asp-net-core-web-api-entity-framework/) we created the initial solution that this post is based upon. If you want to, you can grab the source [here on Github](https://github.com/nicklasjepsen/ProfileApiSample).

### Adding Endpoints
When finished with this part of the tutorial, we will be able to call the following endpoints:
<table>
	<th>HTTP Verb</th>
	<th>URL</th>
	<th>Controller Method</th>
	<th>Comment</th>
	<tr>
		<td>GET</td>
		<td>api/People</td>
		<td>Get()</td>
		<td>Get all people in the database</td>
	</tr>
	<tr>
		<td>GET</td>
		<td>api/People/1</td>
		<td>Get(int id)</td>
		<td>Get Person with id = 1 from the database</td>
	</tr>
		<tr>
		<td>POST</td>
		<td>api/People</td>
		<td>Post(Person model)</td>
		<td>Create a new Person in the database and returns the URI to that Person</td>
	</tr>
		<tr>
		<td>PUT</td>
		<td>api/People/1</td>
		<td>Put(int id, Person model)</td>
		<td>Updates an existing Person in the database</td>
	</tr>
		<tr>
		<td>DELETE</td>
		<td>api/People/1</td>
		<td>Delete(int id)</td>
		<td>Delete Person with id 1 in the database</td>
	</tr>
</table>

All GET methods can be called from the browser, however, you need a tool like [Fiddler](http://www.telerik.com/fiddler) or [Postman](https://www.getpostman.com/) to execute other HTTP Verbs than GET.

## The Repository
Now that we have set up the solution and created our initial database in the previous post, we will move on and create the repository that our controller will be using to access the data. We could as well just implement the repository logic directly in the controller, however, then we would not be able to reuse it and testing would be harder.

* In the Data folder, create a new class (Ctrl+Shift+C), name it PersonRepository

We are going to add and interface to use as a contract for our repository. Again, this is for testing, primarily.

Add the below code to the PersonRepository.cs:

```csharp
public interface IPersonRepository
{
    Task<Person> AddAsync(Person item);
    IEnumerable<Person> GetAll();
    Task<Person> FindAsync(int personId);
    Task RemoveAsync(int personId);
    Task UpdateAsync(Person person);
}
```

This interface exposes the functionality that we are going to implement in the repository.

Here is the implementation of the PersonRepository class:

```csharp
public class PersonRepository : IPersonRepository
{
    private readonly ApplicationDbContext context;

    public PersonRepository(ApplicationDbContext context)
    {
        this.context = context;
    }

    public async Task<Person> AddAsync(Person item)
    {
        context.Add(item);
        await context.SaveChangesAsync();

        return item;
    }

    public IEnumerable<Person> GetAll()
    {
        return context.People.Include(p => p.Gender);
    }

    public async Task<Person> FindAsync(int id)
    {
        return await context.People.Include(p => p.Gender).FirstOrDefaultAsync(p => p.Id == id);
    }

    public async Task RemoveAsync(int id)
    {
        var entity = await context.People.SingleOrDefaultAsync(p => p.Id == id);
        if (entity == null)
            throw new InvalidOperationException("No person found matching id " + id);

        context.People.Remove(entity);
        await context.SaveChangesAsync();
    }

    public async Task UpdateAsync(Person item)
    {
        if (item == null)
            throw new ArgumentNullException(nameof(item));

        context.People.Update(item);
        await context.SaveChangesAsync();
    }
}
```

You should now have a file with the contents like [this](https://github.com/nicklasjepsen/ProfileApiSample/blob/master/ProfileApi.WebApi/Data/PersonRepository.cs).


Now and explanation from the top down:

* In the constructor we are using the built in dependency injection in .NET Core to obtain an instance of our `ApplicationDbContext`class
* Next comes the interface implemenation of the CRUD (Create, Read, Update, Delete - named a little different) methods.
* The most important part to notice is that we are eagerly loading the `Gender`relation since it would otherwise be null when returned in our controller. This is done by calling `.Include(p => p.Gender)`

That's it for our repository, now on to the Web API Controller

## The Controller

* Open the ValuesController in the Controller folder (should have been added by VS when you created the project, if not, just create a new Controller using the template)
* Rename it to PeopleConstroller

Before implementing the methods, we need to import some namespaces and to create a new constructor that gets the `PersonRepository` injected:

* Add the following using statements:

```csharp
using ProfileApi.WebApi.Data;
using ProfileApi.WebApi.Models;
```

* Insert the code below as a first thing in the class:

```csharp
private readonly IPersonRepository repository;
public PeopleController(IPersonRepository repository)
{
    this.repository = repository;
}
```

Above we are also saving a reference to the instance locally in our `PeopleController`.

Now we are ready to implement the the Get method for getting all people:

* Add/replace the Get method with this:

```csharp
// GET api/people
[HttpGet]
public IActionResult Get()
{
    return Ok(repository.GetAll());
}
```

Peace of cake - we are just calling our repository. Nothing special here.

* Now implement the `GET api/People/{id}` method:

```csharp
// GET api/people/5
[HttpGet("{id}")]
public async Task<IActionResult> Get(int id)
{
    var entity = await repository.FindAsync(id);
    if (entity == null)
        return NotFound();

    return Ok(entity);
}
```

Here we check if the id matches, if it does we simply return HTTP 200 OK and return the `Person` as the JSON represantation. If not, we return HTTP 404 Not Found by calling the action method `NotFound`

* Now we implement the Post method that creates a `Person`:

```csharp
// POST api/people
[HttpPost]
public async Task<IActionResult> Post([FromBody]Person model)
{
    if (model == null)
        return BadRequest("Please provide a Person.");
    if (string.IsNullOrEmpty(model.Email))
        return BadRequest("An email address is required.");

    var entity = await repository.AddAsync(model);

    return CreatedAtRoute(nameof(Get), entity.Id, entity);
}
```

We are doing some basic input validation and again using an action method to return a HTTP 400 Bad Request if the person model is null and if the email property is not set. We could add additional checks if we needed to.
Then we create the entity and store it in our database by calling the repository method `AddAsync`.

Special case here is that we return a HTTP 201 Created with a path to the new resource. This is following REST best pratices.


* Now to the Put method which updates an existing entity

```csharp
// PUT api/people/5
[HttpPut("{id}")]
public async Task<IActionResult> Put(int id, [FromBody]Person model)
{
    if (model == null || model.Id != id)
    {
        return BadRequest();
    }

    var entity = await repository.FindAsync(id);
    if (entity == null)
    {
        return NotFound();
    }

    await repository.UpdateAsync(model);
    return new NoContentResult();
}
```

Here we are checking the id parameter and the model. Then we look for an existing entity in our database and if found we do an update against our repository.

* Finally, our Delete method:

```csharp
// DELETE api/people/5
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id)
{
    try
    {
        await repository.RemoveAsync(id);
    }
    catch (InvalidOperationException e)
    {
        Console.WriteLine(e);
        return BadRequest("No Person found with id " + id);
    }

    return new NoContentResult();
}
```

Nothing that we haven't already discussed in this method, except for some exception handling if the entity is not found.


So with all the above method in place in our `PeopleController` we are now ready to start our application and try it out.

## Making Requests against our Controller
To easy things, I am going to specify the debugging/launch URL and port. 

* Replace the Main method in Program.cs with this:

```csharp
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
        .UseKestrel()
        .UseUrls("http://localhost:6001")
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseIISIntegration()
        .UseStartup<Startup>()
        .Build();

    host.Run();
}
```

We are adding the `UseUrls()` call to tell the host what url to use when debugging.

* Now go to the project settings (Rigth-click on the Web API project in Solution Explorer and click Properties)
	* Go to the Debug tab and in Launch select Project. This will run the project as an EXE instead of hosting it on IIS Express

Run your app (Ctrl+F5 to run without debug, for faster launch time, if you don't intend to debug)

Finally navigate to http://localhost:6001/api/People and you should see the 2 people we seeded in the database.
Also, try out the other endpoints as specified in the top of this post.
You should see something like this:

![Get All People](http://systemout.net/images/ProfileApi/GetAllPeople.png)

That's it - we are done for now. Next post will be about unit testing the controller and repository.


