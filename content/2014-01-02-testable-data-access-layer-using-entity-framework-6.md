---
title: "Testable Data Access Layer using Entity Framework 6"
author: "Nicklas Møller Jepsen"
date: "2014-01-02"
url: "/2014/01/02/testable-data-access-layer-using-entity-framework-6/"
comments: "true"
categories:
  - programming
tags:
  - entity framework
  - ef
  - ef6
  - c-sharp
  - in-memory objects
  - mock
  - mock database
  - repository
  - testable
  - unit test
series:
---
##  Introduction

Unit testing is a great way to get some quick feedback on the code you have just implemented. Normally you can just write a unit test where you create an instance of a type and then invoke the method you want to test.  
When you have a database involved you could do the same, however this creates a strong connection between your database and the unit test. One way of avoiding this dependency is to mock your database context.  
To be able to mock your database context when using Entity Framework 6 Model First approach you need to make some modifications to the generated classes. In this guide I will show you how to make your data access layer testable.  
<!--more-->

  
*This guide assumes you know how to set up a database and how to connect to your database from within Visual Studio.*

**If you just want to see the code, scroll to the bottom for a download link of the complete solution.**

##  Generate the Data Model

Add the model to your solution:

[<img class="aligncenter size-full wp-image-243" alt="Add New Item" src="http://www.systemout.net/wp-content/uploads/2014/01/AddNewItem.png" width="955" height="660" />][1]

Select ADO.NET Entity Data Model, name it Model.edmx, Choose Generate From Database

Connect to your database and:  
[<img class="aligncenter size-full wp-image-244" alt="Connect To Db" src="http://www.systemout.net/wp-content/uploads/2014/01/ConnectToDb.png" width="631" height="566" />][2]

In the next screen select Entity Framework 6.0.  
Then you must configure how the types in the model will be generated:

[<img class="aligncenter size-full wp-image-245" alt="Choose Db Ojects" src="http://www.systemout.net/wp-content/uploads/2014/01/ChooseDbOjects.png" width="631" height="566" />][3]

Set a check mark in the box for Tables and in the Pluralize or singularize generated object names checkbox and click Finish.  
The above will create the objects required to interact with the database and add references to the relevant EF components to the current project in Visual Studio. It will also generate an App.config file where the configuration required by EF will be inserted.  
Most relevant for this guide; **templates for class generation is also added to our project.**

##  Wrapping the Generated Classes in Interfaces

This guide’s focus is to make use of interfaces, which enables us to mock the database. To do so we must manipulate with the generated classes. Now we could just type the changes directly in the generated classes, but as I am sure you are aware of these changes will be lost if/when we make changes to our model and then update from the model.  
Luckily, EF uses some text templates to generate the required classes and these templates can be modified to suite our needs. By modifying these templates, we will keep our changes, even if we update our model from the database.  
As I mentioned previously, the templates are already added to the project. Look for the generated Model.edmx file and expand this in the Solution Explorer.  
Now you should see a Model.Context.tt and a Model.tt file:  
[<img src="http://www.systemout.net/wp-content/uploads/2014/01/SolutionExplorerTt.png" alt="Solution Explorer TextTemplates" width="356" height="397" class="aligncenter size-full wp-image-246" />][4]

Basically what we need to do is extract an interface of the Model.Context.cs and make Model.Context.cs implement this interface.  
In Model.Context.cs: Right-click the class name, select Refactor, select Extract, select Interface and mark the DbSet<T> corresponding to the tables in your database. Next we need to change the interface so that all DbSet<T> are changed to IDbSet<T>.  
Now we will modify the Text Templates.

##  Model.Context.tt

On line 56 add 

<pre class="brush: plain; title: ; notranslate" title="">, I&lt;#=code.Escape(container)#&gt;</pre>

so the full line is:

<pre class="brush: plain; title: ; notranslate" title="">&lt;#=Accessibility.ForType(container)#&gt; partial class &lt;#=code.Escape(container)#&gt; : DbContext, I&lt;#=code.Escape(container)#&gt;</pre>

What did we just add? After the DbContext we assigned an interface of the name I\*NameOfTheContextClass\* which corresponds to the name of the interface generated when we refactored our model context.  
On line 311, change the type of DbSet to IDbSet:

<pre class="brush: plain; title: ; notranslate" title="">"{0} virtual IDbSet&lt;{1}&gt; {2} {{ get; set; }}",</pre>

Here we simply just changes the type to an interface type just as we did in our refactored interface.  
Save the .tt file and Visual Studio should automatically generate the files again and if you open the Model.Context.cs it should look something like this:

<pre class="brush: csharp; title: ; notranslate" title="">public partial class EfRepositoryGuideEntities : DbContext, IEfRepositoryGuideEntities
    {
        public EfRepositoryGuideEntities()
            : base("name=EfRepositoryGuideEntities")
        {
        }
    
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            throw new UnintentionalCodeFirstException();
        }
    
        public virtual IDbSet&lt;Artist&gt; Artists { get; set; }
        public virtual IDbSet&lt;Track&gt; Tracks { get; set; }
    }
</pre>

##  Creating the unit test project

We need to make our mock of the ModelContext before we can write our tests. We also need to implement a fake DbSet which we can use when testing.  
To your unit test project, add a class and implement it like the this:

<pre class="brush: csharp; title: ; notranslate" title="">class TestDataContext : IEfRepositoryGuideEntities
    {
        public IDbSet&lt;Artist&gt; Artists { get; set; }
        public IDbSet&lt;Track&gt; Tracks { get; set; }

        public TestDataContext()
        {
            Artists = new FakeDbSet&lt;Artist&gt;();
        }
    }
</pre>

Now add a FakeDbSet class:

<pre class="brush: csharp; title: ; notranslate" title="">public class FakeDbSet&lt;T&gt; : IDbSet&lt;T&gt; where T : class
    {
        readonly ObservableCollection&lt;T&gt; data;
        readonly IQueryable query;

        public FakeDbSet()
        {
            data = new ObservableCollection&lt;T&gt;();
            query = data.AsQueryable();
        }

        public virtual T Find(params object[] keyValues)
        {
            throw new NotImplementedException("Derive from FakeDbSet&lt;T&gt; and override Find");
        }

        public T Add(T item)
        {
            data.Add(item);
            return item;
        }

        public T Remove(T item)
        {
            data.Remove(item);
            return item;
        }

        public T Attach(T item)
        {
            data.Add(item);
            return item;
        }

        public T Detach(T item)
        {
            data.Remove(item);
            return item;
        }

        public T Create()
        {
            return Activator.CreateInstance&lt;T&gt;();
        }

        public TDerivedEntity Create&lt;TDerivedEntity&gt;() where TDerivedEntity : class, T
        {
            return Activator.CreateInstance&lt;TDerivedEntity&gt;();
        }

        public ObservableCollection&lt;T&gt; Local
        {
            get { return data; }
        }

        Type IQueryable.ElementType
        {
            get { return query.ElementType; }
        }

        System.Linq.Expressions.Expression IQueryable.Expression
        {
            get { return query.Expression; }
        }

        IQueryProvider IQueryable.Provider
        {
            get { return query.Provider; }
        }

        System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
        {
            return data.GetEnumerator();
        }

        IEnumerator&lt;T&gt; IEnumerable&lt;T&gt;.GetEnumerator()
        {
            return data.GetEnumerator();
        }
    }
</pre>

In this test we are only using in-memory objects which we inject on runtime. No connection is made to the database, there are no connection string in the unit test projects app.config and this makes our test totally independent of the database and gives us the control of what data we want our test to work with.

I have zipped the solution and please feel free to download it. It is complete with both the console application using the real database and the unit test project using in-memory objects. Of course the Data layer is also included. Please note that if you intend to try the console application, you need to setup a database on your local machine.

<a href="http://sdrv.ms/1kb7gpj" title="Testable Ef Repository Solution" target="_blank">Download the source here!</a>



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>

 [1]: http://www.systemout.net/wp-content/uploads/2014/01/AddNewItem.png
 [2]: http://www.systemout.net/wp-content/uploads/2014/01/ConnectToDb.png
 [3]: http://www.systemout.net/wp-content/uploads/2014/01/ChooseDbOjects.png
 [4]: http://www.systemout.net/wp-content/uploads/2014/01/SolutionExplorerTt.png