---
categories:
- Development
date: "2008-07-17T00:00:00Z"
read_time: true
tags:
- ASP.Net
- MVC
title: Handling Validation Errors in MVC (a different approach)
---

I just got done reading [Scott Guthrie](http://weblogs.asp.net/scottgu/default.aspx)'s and [Phil Haack's](http://haacked.com/) posts on the 
[MVC Preview 4 release](http://haacked.com/archive/2008/07/16/aspnetmvc-codeplex-preview4.aspx) and there was one little piece in the "What's Next" 
section that really peeked my interest. As it turns out the MVC guys are figuring out how to tackle the problem with handling validation errors and 
reporting them back to the user. How great is that? I don't know about you but validation errors are one of the worst things to have to deal with 
when it comes to a web based application.

As we all know you should never rely on client side validation and you should always validate input (especially user input) prior to calling any business methods 
or database calls. My problem has always been with trying to get the validation messages back to the client. I am always determined to have my validation routines 
in my business layer. But then comes time to report validation errors back to the client and I end up sticking them in the code behind. 
Sure I might (emphasis on might) create a utility or helper class but then the routines become more and more specific and less and less common and I am right back 
to sticking them in the code behind.

Where else can you put them since you not only have access to the control but you also have all the tools you need to render the output?

While I have been playing with MVC I have found myself once again struggling with sending validation messages back to the user. 
This time, however, I decided to try a few different techniques and one has proven to work well (for me at least).

I first started off creating a class to store my validation messages. Since my primary focus is capturing form related validation errors 
I decided to add a **PropertyInfo** property that will give me some more context later on if I need it.

```csharp
public class ValidatorDetail
{
  public string Message { get; set; }
  public PropertyInfo Property { get; set; }
  public IList<ValidatorDetail> Details { get; set; }
}
```

So now that I have a class that contains a validation message with some extra info layered in I turn my attention to how to store the errors. 
At first I decided to make a single collection that would be used as the return type of my validation routines but quickly found it to be too restrictive. 
I decided to leverage the **HttpContext.Items** collection and the **CallContext** class instead. The **HttpContext.Items** collection for web based usage 
and the **CallContext** for unit testing. To isolate myself from these details I created a little wrapper class to manage the switching between web usage and unit testing.

```csharp
public static class ThreadContext
{
  public static T Get<T>(string name) where T : class
  {
    HttpContext context = HttpContext.Current;
    if (context == null)
      return CallContext.LogicalGetData(name) as T;
    else
      return context.Items[name] as T;
  }

  public static void Set<T>(string name, T value)
  {
    HttpContext context = HttpContext.Current;
    if (context == null)
      CallContext.LogicalSetData(name, value);
    else
      context.Items.Add(name, value);
  }

  public static void FreeNamedDataSlot(string name)
  {
    HttpContext context = HttpContext.Current;
    if (context == null)
      CallContext.FreeNamedDataSlot(name);
    else
    context.Items.Remove(name);
  }
}
```

This class uses generic methods so I can strongly type the objects being stored. With that out of the way I created another class specific to my validation 
storage which does nothing more than make calls to the **ThreadContext** class.

```csharp
public static class ValidationStorage
{
  private static readonly string storageName = "ValidationContext";

  public static Validator Current
  {
    get { return ThreadContext.Get<Validator>(storageName); }
  }

  public static bool Exists()
  {
    return (Current != null)
  }

  public static void SetStorage(Validator value)
  {
    ThreadContext.Set<Validator>(storageName, value);
  }

  public static void FreeNamedDataSlot()
  {
    ThreadContext.FreeNamedDataSlot(storageName);
  }
}
```

Finally I create a class that handles the addition and retrieval of the validation messages. Let's first take a look at the constructor for this class.

```csharp
public class Validator : IDisposable
{
  // members

  public Validator()
  {
    Details = new List<ValidatorDetail>();

    if (Exists)
      Parent = ValidatorStorage.Current;

    ValidatorStorage.SetStorage(this);
  }

  // methods

  // idispose
}
```

As you can see I first create a new collection to store the validation messages. Next I check to see if a **ValidationStorage** class already exists and if 
so I set the existing instance to the _Parent_ property (more on this later). Finally I make the instance the _Current_ object in the **ValidationStorage** class. 
I could have easily used a Stack of Validator but rather than popping and pushing I decided to take this approach instead.

Now let's turn our attention to the Dispose method.

```csharp
public class Validator : IDisposable
{
  // members

  // constructor

  // methods

  public void Dispose()
  {
    if (Parent != null)
    {
      ((List<ValidatorDetail>)Parent.Details).AddRange(Details);
      ValidationStorage.SetStorage(Parent);
    }
    else
      ValidationStorage.FreeNamedDataSlot();
  }
}
```

As you can see here when I dispose the **Validator** object I check to see if the instance has a _Parent_ and if it does I take it's _Details_ and add 
them to _Parent.__Details_.

Now on to the properties. Nothing much going on here but in order to understand what the methods are doing you will need to know the properties that are available.

```csharp
public class Validator : IDisposable
{
  public IList<ValidatorDetail> Details { get; private set; }

  public bool FailedValidation { get; private set; }

  public Validator Parent { get; private set; }

  public static Validator Current
  {
    get { return ValidatorStorage.Current; }
  }

  public static bool Exists
  {
    get { return ValidatorStorage.Exists(); }
  }

  public static bool Failed
  {
    get 
    { 
      return (Current != null)
        ? Current.FailedValidation
        : false;
    }
  }

  // constructor

  // methods

  // idispose
}
```

The _Current_ and _Exists_ properties make calls to the **ValidationStorage** class. The _Failed_ property will return "false" if a **Validator** does not exist 
in the **ValidationStorage**. Later on you get a better understand of why these properties are static.

On to the instance methods.

```csharp
public class Validator : IDisposable
{
  // members

  // constructor

  public void AddDetail(ValidatorDetail detail)
  {
    AddDetail(detail, false);
  }

  public void AddDetail(ValidatorDetail detail, bool inError)
  {
    Details.Add(detail);
    FailedValidation = (inError ? true : FailedValidation);
  }

  public bool HasMessage(string propertyName)
  {
    return Details.AsQueryable()
      .Any(p => p.Property.Name == propertyName);
  }

  public string GetMessage(string propertyName)
  {
    return Details.AsQueryable()
      .Where(p => p.Property.Name == propertyName)
      .Select(p => p.Message)
      .SingleOrDefault();
  }

  public IList<string> GetMessages(string propertyName)
  {
    return Details.AsQueryable()
      .Where(p => p.Property.Name == propertyName)
      .Select(p => p.Message)
      .ToList();
  }

  // idispose
}
```

The _HasMessages_ method allows me to check to see if I have any **ValidationDetail** for a given property by name. 
And if there are messages then I can call _GetMessage_ or _GetMessages_.

Finally the static methods. These methods are what will be accessed from within the validation routines.

```csharp
public class Validator : IDisposable
{
  // members

  // constructor

  public static void Warn(ValidatorDetail detail)
  {
    if (!Exists)
      new Validator();

    Current.AddDetail(detail);
  }

  public static void Error(ValidatorDetail detail)
  {
    if (!Exists)
      new Validator();

    Current.AddDetail(detail, true);
  }

  public static void Exception(Exception ex, string message)
  {
    if (!Exists)
      new Validator();

    StringBuilder buffer = new StringBuilder();
    buffer.AppendLine(message);
    buffer.AppendLine(ex.Message);
    buffer.AppendLine(ex.StackTrace);

    ValidatorDetail detail = new ValidatorDetail
    {
      Message = buffer.ToString()
    };

    Current.AddDetail(detail, true);
  }

  // idispose
}
```

Thee methods add the **ValidationDetail** object to the _Details_ collection. Prior to adding it, however, I check to see if there is an existing **Validator** 
available to use and if not I new up a new instance. The _Error_ and _Exception_ methods will also pass in a boolean to the _AddDetail_ method to indicate that 
validation has failed.

So now let's take a look at how to use this. For simplicity assume that there is a controller that performs some basic validation routines against a simple model. 
Let's first take a look at the model.

```csharp
public class UserDataModel
{
  public string UserName { get; set; }
  public string FirstName { get; set; }
  public string LastName { get; set; }
  public string EmailAddress { get; set; }
  public int? Age { get; set; }
  public string JobTitle { get; set; }
}
```

The controller then needs to have a few validation routines so let's look at those now.

```csharp
private void ValidateUserData(UserDataModel userData)
{
  if (string.IsNullOrEmpty(userData.FirstName))
  {
    Validator.Error(new ValidatorDetail
    {
      Message = "First Name cannot be empty.",
      Property = typeof(UserDataModel).GetProperty("FirstName")
    });
  }

  if (string.IsNullOrEmpty(userData.LastName))
  {
    Validator.Error(new ValidatorDetail
    {
      Message = "Last Name cannot be empty.",
      Property = typeof(UserDataModel).GetProperty("LastName")
    });
  }

  if (string.IsNullOrEmpty(userData.EmailAddress))
  {
    Validator.Error(new ValidatorDetail
    {
      Message = "Email Address cannot be empty.",
      Property = typeof(UserDataModel).GetProperty("EmailAddress")
    });
  }
}
```

As you can see here when a validation error occurs I simply call the static method _Error_ and that's it. 
But how does it convey the messages back to the client? This can bee seen in the following code.

```csharp
public ActionResult Save()
{
  UserDataModel userData = new UserDataModel();
  BindingHelperExtensions.UpdateFrom(userData, Request.Form);

  ValidateUserData(userData);
  if (Validator.Failed)
    return View("Edit", userData);

  return View("View", userData.UserName);
}
```

After the controller calls the validation routines it makes a call to the static _Failed_ property of the **Validator** class which will 
indicate whether or not validation has failed (obviously). If it has failed validation then I simply render the _Edit_ view again. Within 
the _Edit_ view a check is made to see if a **Validator** exists using the _Exists_ property and if so the the **ValidationDetail** objects 
can be blindly displayed or a check against per property can be made for a more contextual error messages.

```jsp
<div class="label">First Name:</div>
<div<%= Html.TextBox("FirstName", ViewData.Model.FirstName); %></div>
<div class="message">
  <%
    if (Validator.Failed && Validator.Current.HasMessage("FirstName"))
    {
      Response.Write(Validator.Current.GetMessage("FirstName"));
    }
  %>
</div>
```

One key thing that I think really makes this work well is the ability that this gives to call the **Validator** from almost anywhere in my code. 
Validation routines can now reside within the business layer like they should. And since the **ThreadContext** is in place web services can use 
and implement the same checks that are done in the controller.

While I think this does the job I also feel that there can be some things improved upon or added to really make it shine. For instance:
- Adding a severity property which would indicate if the client has entered something really harmful and allow for an exception page to be rendered
- Changing the _Details_ collection to a **Dictionary** to easily store multiple validation errors per property or data element
- Add an Html extension method to deal with the checking and displaying of validation messages

Any ideas or comments are welcomed.

