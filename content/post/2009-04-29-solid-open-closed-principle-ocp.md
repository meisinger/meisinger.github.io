---
categories:
- Education
date: "2009-04-29T00:00:00Z"
read_time: true
tags:
- SOLID
title: SOLID - Open Closed Principle (OCP)
---

As stated previously... I am getting (or trying to get) back to basics.

It is my belief that developers have to often times refactor the way they think and the way they approach things from time to time to 
make sure that the basics and principles are still strong.

So let's take a look at one of the principles in SOLID; Open Closed Principle (OCP)

OCP simply states:

> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification

In my mind following this principle is not limited to inheriting from a base class and marking methods and properties as virtual. 
While I do believe that there is a time and a place for following OCP in this manner I feel that this is sometimes the only solution that 
developers think of when thinking about OCP.

Take the following example of following OCP through inheritance.

```csharp
public class OrderService 
{
  private readonly IRepository repo;

  public OrderService(IRepository repo) 
  { 
    this.repo = repo; 
  }

  public virtual void Save(Order order) 
  {
    repo.Save(order);
  }
}

public class EmailOrderService : OrderService 
{
  private readonly ISmptService emailService;

  public EmailOrderService(IRepository repo, ISmptService emailService) 
    : base(repo) 
  {
    this.emailService = emailService;
  }

  public override void Save(Order order) {
    base.Save(order);
    emailService.SendNotification(order);
  }
}
```

Ok so now we have these classes and we are clearly following OCP but I believe that there is an alternative through use of the 
Strategy pattern which would make this a little cleaner.

```csharp
public class OrderService 
{
  private readonly IRepository repo;

  public OrderService(IRepository repo) 
  {
    this.repo = repo; 
  }

  public void Save(Order order) 
  {
    repo.Save(order);
  }

  public void Save(Order order, IOrderStrategy strategy) 
  { 
    this.Save(order);
    strategy.Process(order);
  } 
}

public class EmailOrderStrategy : IOrderStrategy 
{
  private readonly ISmptService emailService;

  public EmailOrderStrategy(ISmptService emailService) 
  {
    this.emailService = emailService; 
  }

  public void Process (Order order) 
  {
    emailService.SendNotification(order);
  }
}
```

Here we can see that OCP is still being followed since the **OrderService** class is still open for extension via the **IOrderStrategy** and is closed for modification. 

We don't need to mark any methods as virtual and don't need to inherit from the **OrderService** class. 
What is even better is that we could have implemented a Specification pattern along with the Strategy pattern and really get cooking.

```csharp
public class OrderService 
{ 
  ... 

  public void Save(Order order, IEnumerable strategies) 
  {
    this.Save(order);
    foreach (var strategy in strategies) 
    {
      if (strategy.CanProcess(order)) 
        strategy.Process(order); 
    }
  }
}

public class EmailOrderStrategy : IOrderStrategy 
{ 
  ... 

  public bool CanProcess(Order order) 
  {
    return order.SendEmail; 
  }

  ... 
}
```

So I guess all this really means is that OCP shouldn't only be thought of as marking methods and properties as virtual. 
OCP should be thought about and reasoned about a little more deeply and carefully.

And granted the examples given here have holes and are more than likely not the best code you have ever seen but it is the ideas these examples show that should be taken.

