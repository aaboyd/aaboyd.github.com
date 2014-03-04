---
layout: post
title: NancyFx with Entity Framework
description: NancyFx with Entity Framework.  A simple CRUD NancyModule using EntityFramework and includes integration testing.
tags:
- C#
- entity framework
- nancy
- SQL
---
<div class="alert success">Example Project on <a href="http://github.com/aaboyd/Dinner">Github</a></div>

# Introduction
NancyFx has recently become one of the most popular web development tools for C# developers.  It is largely inspired by ruby's sinatra, but this didn't mean much to me as I don't write any ruby stuff, but it is much more of a flask than a django if you come from the python world.  Getting back into C# and learning various web platforms and tools, I have found [NancyFx](http://nancyfx.org/) to be intuitive and it stays the hell out of my way.  If I want to do something my own way, I just do it.  It's one of the nicest web development libraries / frameworks I have used, I highly recommend it.  Shortly after any decent amount of web development a database is needed, I opted to go with a Microsoft supported framework in [Entity Framework](http://msdn.microsoft.com/en-us/data/ef.aspx).  That leads me to this article, after a few hours trying to figure out how to build and test a simple application, I wrapped it up in a blog article.

# Get Nancy and EntityFramework Up and Running
Setting up NancyFx is simple and easy.  There are various resources around the web on this specific topic, but I will quickly run through it.

## Create Empty ASP.NET application
Using the Visual Studio 2013 New Project Wizard, create a new empty ASP.NET application.

<img class="six columns" src="/assets/img/nancyfx-ef-testing/create-proj-1.png" />
<img class="six columns" src="/assets/img/nancyfx-ef-testing/create-proj-2.png" />

## Install Dependencies
Using the Nuget Package Manager Console we can add all of our dependencies.
{% highlight powershell %}
Install-Package Nancy.Hosting.Aspnet
Install-Package Nancy.Serialization.JsonNet
Install-Package EntityFramework
{% endhighlight %}

## Create Your First NancyModule
Create file called NancyModule inside your ```Dinner``` project.

{% highlight csharp %}
using Nancy;

namespace AlexBoyd.Dinner
{
  public class DinnerModule : NancyModule
  {
    public DinnerModule() : base("/dinner")
    {
      Get["/"] = x =>
      {
        return "Hello World";
      };
    }
  }
}
{% endhighlight %}

*Note: At this point, if you wanted to run the project you should be able to hit the URL at http://localhost:{ASP.NET_PORT}/dinner and get the Hello World response.*

## Create Database
*This step requires an installation of [SQL Server Express](http://www.microsoft.com/en-us/sqlserver/editions/2012-editions/express.aspx) or [SQL Server](http://www.microsoft.com/sqlserver).*


In your Empty ASP.NET project create a folder App_Data, this is where the database file will live.  Creating a database inside the folder isn't too difficult, but it took a few minutes to figure out the easiest way to do it.  Right click the App_Data folder then navigate to Add -> SQL Server Database.  You will be prompted to pick a name, remember this name.

## Create Simple Model
A POCO can be used for an EntityFramework model which will be mapped to a table in a SQL database.  There are more than one approach to this including fluent vs annotations.  This example is simple and won't get into the debate of which is better for your situation.

Create a POCO called Dinner.  I borrowed some of this from the fairly popular example project [Nerd Dinner](http://www.nerddinner.com/).  Here is my POCO.

{% highlight csharp %}
using System;

namespace AlexBoyd.Dinner
{
  public class Dinner
  {
    public int Id { get; set; }
    public string Title { get; set; }
    public DateTime Date { get; set; }
    public string Address { get; set; }
    public string HostedBy { get; set; }
  }
}
{% endhighlight  %}

## Create DBContext
A DBContext is the primary class that is responsible for interacting with your SQL Database.  To read more on this and how it relates to your POCO's and your SQL Database, check out [Microsoft's article](http://msdn.microsoft.com/en-us/data/jj729737.aspx).

Our DbContext will contain only one IDbSet for our Dinner object.  Other than that there is some EF Fluent API configuration for our POCO.  Notice the use of two constructors, the application will use the empty constructor and the tests will use the constructor taking the string argument.

{% highlight csharp %}
using System.ComponentModel.DataAnnotations.Schema;
using System.Data.Entity;

namespace AlexBoyd.Dinner
{
  public class MyContext : DbContext
  {
    public MyContext() {}
    public MyContext(string connectionString) : base(connectionString) { }

    public IDbSet<Dinner> Dinners { get; set; }

    protected override void OnModelCreating(DbModelBuilder builder) 
    {
      builder.Entity<Dinner>().Property(d => d.Id).HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity);
      builder.Entity<Dinner>().HasKey(d => d.Id);
    }
  }
}
{% endhighlight %}

### Add a Connection String
To construct the correct connectionString and name you will need to remember some of the stuff that you have previously done.  The name should be the name of your DbContext class.
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  ...
  <connectionStrings>
    <add name="MyContext"
         connectionString="Data Source=(LocalDb)\v11.0;AttachDbFilename=|DataDirectory|\MyDb.mdf;Initial Catalog=MyDB;Integrated Security=True" providerName="System.Data.SqlClient" />
  </connectionStrings>
  ...
</configuration>
{% endhighlight %}

## Get a handle of the context in NancyModule
Nancy comes with some built-in AutoRegister functionality.  If you create an interface and add it to the constructor of your module, it will automatically be injected.  For this to work you must only have one implementation of that interface, but that is going to be the case for our simple application.

Let's extract an interface from our DbContext with some methods we will need.

{% highlight csharp %}
public interface IMyContext
{
  IDbSet<Dinner> Dinners { get; set; }

  DbEntityEntry Entry(object entity);
  int SaveChanges();
}
{% endhighlight %}

And of course, lets implement this interface in our context class.

{% highlight csharp %}
public class MyContext : DbContext, IMyContext
{
  ...
}
{% endhighlight %}

Lastly, add the IMyContext as a dependency in our module.

{% highlight csharp %}
public DinnerModule(IMyContext ctx) : base("/dinner")
{
  ...
}
{% endhighlight %}


# Start Testing
Now that we have configured the application to use Nancy and allow us to interact with our EntityFramework entities, we can move on to setting up some simple tests.

## Add CRUD Skeleton
Setting up some simple CRUD operations in Nancy couldn't be simpler.  Trying to stay (somewhat) true to REST principles, I will layout some skeleton code for a REST interface to our Dinner POCO.

{% highlight csharp %}
public class DinnerModule : NancyModule
{
  public DinnerModule(IMyContext ctx) : base("/dinner")
  {
    Get["/"] = x =>
    {
      return HttpStatusCode.NotImplemented;
    };

    Post["/"] = _ =>
    {
      return HttpStatusCode.NotImplemented;
    };

    Put["/{id:int}"] = parameters =>
    {
      return HttpStatusCode.NotImplemented;
    };

    Delete["/{id:int}"] = x =>
    {
      return HttpStatusCode.NotImplemented;
    };
  }
}
{% endhighlight %}

## Create Test Project
Using the "New Project Wizard" create a new UnitTest project, call it Dinner.Test.  We will stick with MSTest for this particular example, but the code should be very simple to adapt to other frameworks.

Even though this is a UnitTest project these are actually a bit more like integration tests.  We won't be mocking our DbContext, but rather using a file database (SQLCE) for each test.  I have not tried this with other DB's like SQLite, but in theory everything should work fine.

## Install Dependencies ( Nancy.Testing, EntityFramework, 
In the package manager console, execute the following commands.

{% highlight powershell %}
Install-Package Nancy.Testing
Install-Package EntityFramework.SqlServerCompact
{% endhighlight %}

**We also will need to create a reference to our Dinner project.**

## (Re)Create Database before each test
Each test will have it's own isolated version of the database.  To do this, we simply need to create a method and have it run before every test.  Here is what I came up with.  It's pretty self explanatory.

{% highlight csharp %}
namespace AlexBoyd.Dinner.Test
{
  [TestClass]
  public class TestDinner
  {
    private string _ConnectionString;

    [TestInitialize]
    public void SetupTest()
    {
      //get the full location of the assembly with tests in it
      string assemblyPath = System.Reflection.Assembly.GetExecutingAssembly().Location;
      string directory = Path.GetDirectoryName(assemblyPath);
      string filePath = Path.Combine(directory, "test.sdf");

      // delete it if it already exists
      if (File.Exists(filePath)) File.Delete(filePath);

      _ConnectionString = "Datasource = " + filePath;

      Database.SetInitializer<MyContext>(new CreateDatabaseIfNotExists<MyContext>());
      using (var context = new MyContext(_ConnectionString))
      {
        context.Database.Create();
      }
    }
  }
}
{% endhighlight %}

## Fill in Test Code
Now that we have a _ConnectionString and  create database, we can write our tests and watch them fail miserably.  These tests aren't exhaustive or even that great, but it is a good start, and shows use of the DB before and after testing the NancyModule's.

{% highlight csharp %}
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Nancy;
using Nancy.Testing;
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.IO;
using System.Threading.Tasks;

namespace AlexBoyd.Dinner.Test
{
  [TestClass]
  public class TestDinner
  {
    private string _ConnectionString;

    [TestInitialize]
    public void SetupTest()
    {
      //get the full location of the assembly with tests in it
      string assemblyPath = System.Reflection.Assembly.GetExecutingAssembly().Location;
      string directory = Path.GetDirectoryName(assemblyPath);
      string filePath = Path.Combine(directory, "test.sdf");

      // delete it if it already exists
      if (File.Exists(filePath)) File.Delete(filePath);

      _ConnectionString = "Datasource = " + filePath;

      Database.SetInitializer<MyContext>(new CreateDatabaseIfNotExists<MyContext>());
      using (var context = new MyContext(_ConnectionString))
      {
        context.Database.Create();
      }
    }

    [TestMethod]
    public async Task Test_delete_dinner()
    {
      AlexBoyd.Dinner.Dinner dinner = null;
      using (var ctx = new MyContext(_ConnectionString))
      {
        dinner = ctx.Dinners.Add(new AlexBoyd.Dinner.Dinner()
        {
          Title = "Dinner Title",
          Date = DateTime.UtcNow,
          Address = "Awesome Street, Detroit, MI",
          HostedBy = "The Artist Formerly Known as Prince"
        });

        ctx.SaveChanges();
      }
      var bootstrapper = new ConfigurableBootstrapper(with =>
      {
        with.Module<DinnerModule>();
        with.Dependencies<IMyContext>(new MyContext(_ConnectionString));
      });

      var browser = new Browser(bootstrapper);

      var response = browser.Delete("/dinner/" + Convert.ToString(dinner.Id));
      Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);

      using (var ctx = new MyContext(_ConnectionString))
      {
        int count = await ctx.Set<AlexBoyd.Dinner.Dinner>().CountAsync();
        Assert.AreEqual(0, count);
      }
    }

    [TestMethod]
    public void Test_read_dinner()
    {
      AlexBoyd.Dinner.Dinner dinner = null;
      using (var ctx = new MyContext(_ConnectionString))
      {
        dinner = ctx.Dinners.Add(new AlexBoyd.Dinner.Dinner()
        {
          Title = "Dinner Title",
          Date = DateTime.UtcNow,
          Address = "Awesome Street, Detroit, MI",
          HostedBy = "The Artist Formerly Known as Prince"
        });

        ctx.SaveChanges();
      }
      var bootstrapper = new ConfigurableBootstrapper(with =>
      {
        with.Module<DinnerModule>();
        with.Dependencies<IMyContext>(new MyContext(_ConnectionString));
      });

      var browser = new Browser(bootstrapper);

      var response = browser.Get("/dinner/");

      Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);

      String responseString = response.Body.AsString();

      IList<AlexBoyd.Dinner.Dinner> dinners = response.Body.DeserializeJson<IList<AlexBoyd.Dinner.Dinner>>();
      Assert.AreEqual(1, dinners.Count);
    }

    [TestMethod]
    public async Task Test_update_dinner()
    {
      AlexBoyd.Dinner.Dinner dinner = null;
      using (var ctx = new MyContext(_ConnectionString))
      {
        dinner = ctx.Dinners.Add(new AlexBoyd.Dinner.Dinner()
        {
          Title = "Dinner Title",
          Date = DateTime.UtcNow,
          Address = "Awesome Street, Detroit, MI",
          HostedBy = "The Artist Formerly Known as Prince"
        });

        ctx.SaveChanges();
      }
      var bootstrapper = new ConfigurableBootstrapper(with =>
      {
        with.Module<DinnerModule>();
        with.Dependencies<IMyContext>(new MyContext(_ConnectionString));
      });

      var browser = new Browser(bootstrapper);
      var newTitle = "I am different";
      var response = browser.Put("/dinner/" + Convert.ToString(dinner.Id), with =>
      {
        with.JsonBody<AlexBoyd.Dinner.Dinner>(new AlexBoyd.Dinner.Dinner()
        {
          Title = newTitle,
          Date = dinner.Date,
          Address = dinner.Address,
          HostedBy = dinner.HostedBy
        });
      });

      Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);

      using (var ctx = new MyContext(_ConnectionString))
      {
        var updatedDinner = await ctx.Set<AlexBoyd.Dinner.Dinner>().FirstAsync(d => d.Id == dinner.Id);
        Assert.AreEqual(newTitle, updatedDinner.Title);
      }
    }

    [TestMethod]
    public async Task Test_create_dinner()
    {
      var bootstrapper = new ConfigurableBootstrapper(with =>
      {
        with.Module<DinnerModule>();
        with.Dependencies<IMyContext>(new MyContext(_ConnectionString));
      });

      var browser = new Browser(bootstrapper);
      var response = browser.Post("/dinner/", with =>
      {
        with.JsonBody<AlexBoyd.Dinner.Dinner>(new AlexBoyd.Dinner.Dinner()
        {
          Title = "My First Dinner",
          Date = DateTime.UtcNow,
          Address = "1 Campus Martius, Detroit, MI",
          HostedBy = "Alex Boyd"
        });
      });

      Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);

      using (var ctx = new MyContext(_ConnectionString))
      {
        var count = await ctx.Dinners.CountAsync();
        Assert.AreEqual(1, count);
      }
    }
  }
}
{% endhighlight %}

## Get the test's working
Now that we have written some tests that test our CRUD operations, let's implement the DinnerModule to complete our NancyFx + EntityFramework + CRUD example.

{% highlight csharp %}
using System.Linq;

using Nancy;
using Nancy.ModelBinding;
using System.Data.Entity;

namespace AlexBoyd.Dinner
{
  public class DinnerModule : NancyModule
  {
    public DinnerModule(IMyContext ctx) : base("/dinner")
    {
      Get["/"] = x =>
      {
        return Response.AsJson<object>(ctx.Dinners.ToArray());
      };

      Post["/"] = _ =>
      {
        Dinner dinner = this.Bind<Dinner>();

        ctx.Dinners.Add(dinner);
        ctx.SaveChanges();
        
        return HttpStatusCode.OK;
      };

      Put["/{id:int}"] = parameters =>
      {
        Dinner dinner = this.Bind<Dinner>();

        dinner.Id = parameters.id;
        ctx.Dinners.Attach(dinner);
        ctx.Entry(dinner).State = EntityState.Modified;
        ctx.SaveChanges();

        return HttpStatusCode.OK;
      };

      Delete["/{id:int}"] = x =>
      {
        var dinner = new Dinner() { Id = x.id };
        ctx.Entry(dinner).State = EntityState.Deleted;

        ctx.SaveChanges();

        return HttpStatusCode.OK;
      };
    }
  }
}
{% endhighlight %}