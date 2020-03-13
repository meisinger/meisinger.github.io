---
layout: single
title: "Dependency Injection - Why should we care?"
date: "2008-04-03"
read_time: true
categories: [Education]
tags: [SOLID]
---

In my previous post we covered (briefly) Inversion of Control (IoC) and Dependency Injection. 
In this post we will discuss why we should use Dependency Injection and how it can improve our every day coding life.

Again... from my previous post we found that we can inject objects and values a particular class is dependant upon through the 
constructor or through property setters. 
But this really begs the question of why you would want to do that? Well there are a couple of benefits from doing this:
1. Identifying dependencies for a particular class becomes easier
2. Inheritance fully describes dependencies
3. Dependencies become easier to test, mock or stub

So lets take each of these reasons one by one and give some brief examples. Consider the following class as an example:

```csharp
public class Employee
{
  public Employee() {}

  public static Employee GetEmployeeById(int id)
  {
    Employee employee = new Employee();
    //perform database query here
    //populate employee
    return employee;
  }
  //...
}

public class EmployeeRate
{
  protected Employee employee;

  public EmployeeRate(int employeeId)
  {
    this.employee = Employee.GetEmployeeById(employeeId);
  }

  public decimal CalculateFlatRate()
  {
    return employee.Rate;
  }

  public decimal CalculateSpecialRate()
  {
    if (employee.IsIdle)
      return employee.Rate;

    decimal factor = 
      ((employee.Rate * employee.Experience) / 0.75);

    return factor * 15.75;
  }
}
```

As we can see from this simple example the "EmployeeRate" class is dependent upon the "Employee" class. 
Not only that but we can also see that in order to get the "Employee" class we need to make a call to the database to retrieve the information.

Let's suppose that someone else is trying to create an instance of this class. 
Another developer would never know that this class uses the "Employee" class and it might very well be that they are already working an "Employee" class. 
So if instead we refactor this class to follow the Dependency Injection pattern, anybody would be able to quickly tell that the "Employee" class is indeed a dependent class.

```csharp
public class EmployeeRate
{
  protected Employee employee;

  public EmployeeRate(Employee employee)
  {
    this.employee = employee;
  }

  public decimal CalculateFlatRate()
  {
    return employee.Rate;
  }

  public virtual decimal CalculateSpecialRate()
  {
    if (employee.IsIdle)
      return employee.Rate;

    decimal factor = 
      ((employee.Rate * employee.Experience) / 0.75);

    return factor * 15.75;
  }
}
```

 _(If you are like me you more than likely wouldn't stop there. 
 After looking further into the code you might have seen that there is no real need to have a constructor for this class at all. 
 Instead, simply make the two methods static and pass the "Employee" class in as a parameter. 
 For the purpose of this post however... let's just play along)_

**Let's now take a look at how the inheritance model would benefit from using the Dependency Injection pattern**

Let's once again take the original example and create a new class called "ManagerRate." 
This class needs to do some things that a normal employee wouldn't have to do like factor in the number of employees they manage. 
Regardless of the reason, we need to calculate a managers rate differently then an employee. 
We could add a "switch" statement or "if ... else" condition to apply this logic but... being good little developers we created this class.

```csharp
public class ManagerRate : EmployeeRate
{
  public ManagerRate(int employeeId)
    : base(employeeId)
  {
  }

  public override decimal CalculateSpecialRate()
  {
    if (employee.IsIdle)
      return employee.Rate;

    decimal factor = 
      ((employee.Rate * employee.Experience) / 0.5);

    return factor * 16.05;
  }
}
```

Here we can see that it is not clear to the "ManagerRate" class what dependencies the base class requires. 
Not only that but if we put our "extensibility" hat on for a second we can see that if an employee and a manager are ever 
broken up to be separate classes then we would have a problem and would more than likely result in a change in logic in both classes. 
Let's now see what our "ManagerRate" class would like from our refactored version.

```csharp
public class ManagerRate : EmployeeRate
{
  public ManagerRate(Employee employee)
    : base(employee)
  {
  }

  public override decimal CalculateSpecialRate()
  {
    if (employee.IsIdle)
      return employee.Rate;

    decimal factor = 
      ((employee.Rate * employee.Experience) / 0.5);

    return factor * 16.05;
  }
}
```

Now the "ManagerRate" class can clearly see what the dependencies are for the base class. 
And if we put our "extensibility" hat back on we could see that we might still have an issue if an employee and a manager are 
separated into two classes but the code change would only need to occur in one class and more than likely it would only require they type to change. 
If we really wanted to do it up correctly we would create an interface for the "Employee" class.

**Finally, let's look at how testing can benefit from our use of the Dependency Injection pattern**

If we go back to the original example and look at what happened in the constructor we would see that the constructor called a static method on the 
"Employee" class named "GetById." Suppose now that we want to test the "EmployeeRate" class to ensure that it's "CalculateSpecialRate" method returned 
the correct value. How would we go about doing that?

First we would need to make sure that we have a database with valid (or good) data in it for a given "Employee." 
Normally we would insert a new row in the corresponding table and record the resulting primary key or "id." 
Then when we ran our unit test we would need to make sure that we pass in the correct "id" to our "EmployeeRate" constructor.

If, however, our data gets deleted, modified, or someone else is running our unit tests in another environment then our tests would fail. 
It wouldn't fail because our code is wrong but because the supporting data is missing. So what are we really testing in this scenario... the code or the data?

If we follow the Dependency Injection pattern as we did in our refactored code then we would be able to create reliable and "database proof" unit tests. 
By passing in a mock or stub of an "Employee" class, the data being used will always be the same and our unit tests will always be testing the code and not the data.

Another benefit that we did not have time to cover is how Dependency Injection is used in Aspect Oriented Programming (AOP). 
There are plenty of other posts out there that cover AOP that I recommend you read. 
In short... being able to intercept calls and inject other objects, values, or classes based on the cross-cutting concerns is not only the 
point behind AOP but dramatically reduces code and increases code reuse.

In this post we covered three out of many benefits to using Dependency Injection. 
Hopefully the examples drive this point home and starts to make you rethink the way you design and write code in your everyday life.

