---
categories:
- Education
date: "2009-04-30T00:00:00Z"
read_time: true
tags:
- SOLID
title: SOLID - Liskov Substitution Principle (LSP)
---

As stated previously… I am getting (or trying to get) back to basics.

It is my belief that developers have to often times refactor the way they think and the way they approach things from time to time to 
make sure that the basics and principles are still strong.

So let’s take a look at one of the principles in SOLID; Liskov Substitution Principle (LSP)

LSP states:

> If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2 then S is a subtype of T

Right... to many this is as clear as mud. 

It took me a while to understand what this principle was really talking about at first. 
Then after reading a few papers written on the subject it became a little more clear. 
What I have seen many developers do, however, is paraphrase this principle (or actually rewrite it) to make it easier to understand.

Commonly, LSP is paraphrased to state:

> Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it

So really all we are talking about is making sure our methods use base classes or interfaces as parameters and that the method does not have 
to know about all of the derived types of the base class or the classes that implement an interface.

Perhaps it is best to use an example to better explain what this principle is talking about. 

```csharp
public class Customer { ... }

public class Vendor : Customer { ... }

public class CustomerValidator 
{
  public void Validate(Customer customer) 
  {
    if (string.IsNullOrEmpty(customer.Name))
    { 
      // indicate that validation has failed 
    } 

    var vendor = customer as Vendor;
    if (vendor != null)
    {
      // we know we are dealing with a Vendor 
      // validate Vendor rules 
    }
  }
}
```

Here we have a **Customer** class as well as a **Vendor** class that inherits from **Customer**.

In the **CustomerValidator** class we are correctly using the **Customer** class (which would allow a **Customer** or **Vendor** object to be used... good so far) 
but then the **Validate** method has to know about the **Vendor** type and therefore violates LSP. 

What we can also (hopefully) see here is that we are violating the Open Closed Principle (OCP) due to the fact that any time we create 
another derived type from **Customer** we must modify the **Validate** method to handle the new type (hence it is closed for extension and is open for modification).

So how do we correct the above to remove the LSP violation?

The following is one way (and probably not the best way) to resolve the issues. 

```csharp
public class Validator 
{ 
  private readonly IList errors;

  public Validator() 
  {
    this.errors = new List();
  }

  public void Validate(IValidation validation) 
  {
    validation.Validate(errors); 
  } 
}

public interface IValidation 
{ 
  void Validate(IList errors); 
}

public CustomerValidation : IValidation 
{
  private readonly Customer customer;

  public CustomerValidation(Customer customer) 
  {
    this.customer = customer;
  }

  public virtual void Validate(IList errors) 
  {
    // if we fail validation add an error to the collection 
  } 
}

public VendorValidation : CustomerValidation 
{
  private readonly Vendor vendor;

  public VendorValidation(Vendor vendor) 
    : base(vendor) 
  { 
    this.vendor = vendor; 
  }

  public override void Validate(IList errors) 
  { 
    // call the Customer validation routines 
    base.Validate(errors); 
    // if we fail validation add an error to the collection 
  }
}
```

Wow. We have changed things up a lot.

If we look at the **Validator** class we can see that the **Validate** method takes a parameter which allows us to pass in any class that implements 
the **IValidation** interface. This follows LSP because we can substitute any class which implements this interface without changing the behavior. 

Not only that but the method doesn't need to know anything about the types which implement the interface. 
At the same time, we have also solved our OCP violation by making the **CustomerValidation** class open for extension and closed for modification.

One more important thing that I would like to point out that sometimes get left out when explaining LSP is that exceptions should (or must) follow the same rules. 

This means that if you are throwing an exception in a method of a base class then any derived type overriding the method should throw the same type of exception. 
By doing this you can be ensured that your error handling for the base class will be correct for derived types as well. 

Just something that you should keep in mind.

