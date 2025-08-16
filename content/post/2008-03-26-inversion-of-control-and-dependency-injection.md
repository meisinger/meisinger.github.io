---
categories:
- Education
date: "2008-03-26T00:00:00Z"
read_time: true
tags:
- SOLID
title: Inversion of Control and Dependency Injection
---

MVC has not only brought about a change in how we think about designing and building web applications on the .Net framework but it has also reminded 
most of us about the Inversion of Control (IoC) and Dependency Injection patterns. 
While most of us have used these patterns before (without even knowing about it), I think that it is very important to understand what these 
patterns are, how to use them, and why they are important not only with MVC but in everyday "coding" life.

IoC and DI are defined and described at length in an article written back in 2004 by [Martin Fowler](http://martinfowler.com/) named ["Inversion of Control Containers and the Dependency Injection pattern"](http://martinfowler.com/articles/injection.html) that I highly recommend everyone read. I am not going to go into great details about these patterns but...
- IoC means that an object gets other objects that it relies on through an outside source or framework
- Dependency Injection means that an object gets other objects that it relies on through it's constructor or set properties

It is safe to say that these are the same but just remember that there is a difference. 
I have also found that when people or articles have talked about IoC they have been really talking about Dependency Injection. 
So enough about that... what does it look like?

Let's start with a simple class which relies on a Data Access class (named "MyDataAccess") to connect to some database...

```csharp
public class MyObject
{
  private MyDataAccess dataAccess;

  public MyObject()
  {
    this.dataAccess = new MyDataAccess();
  }
}
```

Imagine now that the "MyDataAccess" implements an interface named "IDataAccess" and we change the code to reflect that...

```csharp
publci class MyObject
{
  private IDataAccess dataAccess;

  public MyObject()
  {
    this.dataAccess = (IDataAccess) new MyDataAccess();
  }
}
```

But now if our example changes one more time... and this time we add yet another class which implements the "IDataAccess" interface, 
we would have to add more logic or code to determine which concrete class should be created...

```csharp
publci class MyObject
{
  private IDataAccess dataAccess;

  public MyObject()
  {
    int type = (int)ConfigurationManager.AppSettings["dbType"];
    if (type == 1)
      this.dataAccess = (IDataAccess) new MyDataAccess();
    else
      this.dataAccess = (IDataAccess) new YourDataAccess();
  }
}
```

Clearly this is going to be a problem each time we want add a new class which implements the "IDataAccess" interface. 
Now expand this out to several classes which depend upon an "IDataAccess" object and you start to see the overhead with maintaining that code.

Enter the Dependency Injection pattern where we pass in an "IDataAccess" object as a parameter to the constructor. 
This then moves the responsibility (or control) of creating the "IDataAccess" object outside of our class or outside of it's scope.

```csharp
publci class MyObject
{
  private IDataAccess dataAccess;

  public MyObject(IDataAccess dataAccess)
  {
    this.dataAccess = dataAccess;
  }
}
```

So now any time someone creates an instance of this class they can clearly see that it needs or depends upon an "IDataAccess" object. 
We have also increased the test-ability of our class by being able to pass in stubs or "mock" objects.

In my next post I will cover more of the benefits and show it works amd how it is used in Testing, MVC and LINQ frameworks.

