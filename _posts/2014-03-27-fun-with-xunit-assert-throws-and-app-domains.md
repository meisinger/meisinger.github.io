---
layout: single
title: "Fun with xUnit Assert Throws and App Domains"
date: "2014-03-27"
read_time: true
categories: [Development]
tags: [xUnit]
---

I recently decided to update the shared libraries and tools that are used to build and unit test Rhino Mocks.

I updated [psake](https://github.com/psake/psake) from version 0.22 to version 4.3.1.0. 
Now I don’t know about you but that is a significant jump in version number. After the update, no problems. 
I actually find it easier to deal with and have come to the realization that I should revisit the existing tasks.

I then went ahead and update [xUnit](https://github.com/xunit/xunit) from version 1.1.0 to version 1.9.2. 
After the update I had a significant number of build errors. All of the build errors were concerning the `Assert.Throws` extension method. 
In the old version this method had an overload that took a “message” as a parameter. 
In the latest version this overload was removed.

I was very concerned with this breaking change. 
There were so many unit tests that used this method to verify the thrown exception message matched the expectation. 
I cracked open the previous version in dotPeek and took a look at the method. 
And what I saw was rather surprising.

```csharp
public static T Throws<T>(string userMessage, ThrowsDelegate testCode)
  where T: Exception
{
  return (T) Throws(typeof(T), testCode);
}
```

Clearly the `userMessage` parameter is not being used.

Now realizing that the “message” parameter was never being used. 
I changed all of the unit tests and removed the messages being passed. 
After I fixed the unit tests everything was working perfectly. 
This has now become a fortunate change of events. 
This now provides me with the ability to change the exception messages to provide better context and more information.

All was right in the world. Well…

It was an hour after the update that I realized that I only updated the [xUnit](https://github.com/xunit/xunit) libraries. 
I hadn’t updated the executable or console application that actually ran the unit tests. 
So back to NuGet I went to get the latest (stable) version of the [xUnit](https://github.com/xunit/xunit) console application.

This is where things took a turn for the worse. There was one (and only one) unit test that was now failing. 
I am going to give you the namespace and test class name and let you figure out what the problem was: **Rhino.Mocks.Test.Remoting.ContextSwitchTests**

Talk about fun.

I will save you the gory (and embarrassing) details on how I corrected the problem. 
The bottom line is that I had to change the way the unit test was creating the App Domain to match the same 
way the [xUnit](https://github.com/xunit/xunit) was loading the unit test into it’s App Domain.

There’s a lesson in here somewhere but right now… I can’t think of it.

Coming up next is documentation and getting the first 4.0.0 alpha build out on NuGet.

