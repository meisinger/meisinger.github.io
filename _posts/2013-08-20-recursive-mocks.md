---
layout: single
title: "Recursive Mocks"
date: "2013-08-20"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

I have been going through all of the unit tests for Rhino Mocks attempting to get the first “alpha” release ready. 
One of the sets of unit tests that were failing (miserably) were the “Recursive Mocks.” I struggled with this for a while. 
I could implement the feature as intended or simply ignore it and change the unit tests.

Not all of us may know what a Recursive Mock looks like and what the feature enables. Let’s take a quick look at an example. 
First we will look at how the scenario is commonly implemented followed by the Recursive Mock feature.

The scenario is fairly straightforward and common; two interfaces are being mocked where one of the interfaces contains a method which returns the other interface:

```csharp
public interface IChannel {
  bool IsOpen();
}

public interface IConnection {
  IChannel CreateChannel(string path);
}

```

The most common way to mock the `IsOpen` method is as follows:

```csharp
[Fact]
public void Scenario_Common_Approach() { 
  var connection = Repository.Mock<IConnection>();
  var channel = Repository.Mock<IChannel>();

  channel.Stub(x => x.IsOpen())
    .Return(true);

  connection.Stub(x => x.CreateChannel(Arg<string>.Is.Anything))
    .Return(channel);

  // some interesting test here 
}
```

With the Recursive Mock feature, however, this scenario could be mocked in the following way:

```csharp
[Fact]
public void Scenario_Recursive_Approach() {
  var connection = Repository.Mock<IConnection>();
  connection.Stub(x => x.CreateChannel(Arg<string>.Is.Anything).IsOpen())
    .Return(true);

  // some interesting test here
}
```

While this is a contrived example, what we can see from the Recursive Mock approach is that we don’t need to create a mock for `IChannel`. 
Rhino Mocks will actually create a mocked object on-the-fly.

Why would anyone ever use a Recursive Mock?

Imagine what would happen if you had three separate “paths” that you wanted to test. For the first path the `IsOpen` should return `true`. 
For the second path `IsOpen` should return `false`. For the third path `IsOpen` should throw an exception.

That would require three separate `IChannel` mocked objects (and stubs):

```csharp
[Fact]
public void Three_Path_Monty() {
  var connection = Repository.Mock<IConnection>();
  var channelOk = Repository.Mock<IChannel>();
  var channelFail = Repository.Mock<IChannel>();
  var channelThrow = Repository.Mock<IChannel>();

  connection.Stub(x => x.CreateChannel("path-ok"))
    .Return(channelOk);

  connection.Stub(x => x.CreateChannel("path-fail"))
    .Return(channelFail);

  connection.Stub(x => x.CreateChannel("path-throw"))
    .Return(channelThrow);

  channelOk.Stub(x => x.IsOpen())
    .Return(true);

  channelFail.Stub(x => x.IsOpen())
    .Return(false);

  channelThrow.Stub(x => x.IsOpen())
    .Throws(new Exception("error"));

  // something interesting
}
```

The Recursive Mock way… much easier to read:

```csharp
[Fact]
public void Three_Path_Easy() {
  var connection = Repository.Mock<IConnection>();

  connection.Stub(x => x.CreateChannel("path-ok").IsOpen())
    .Return(true);

  connection.Stub(x => x.CreateChannel("path-fail").IsOpen())
    .Return(false);

  connection.Stub(x => x.CreateChannel("path-throw").IsOpen())
    .Throws(new Exception("error"));

  // something interesting
}
```

The current version of Rhino Mocks supports this well with the Record and Last Call constructs. 
With these constructs having been removed in the upcoming version, this feature may be more difficult than what it’s worth. 
At least that’s what I thought… at first.

My biggest concern was with the fact that in the upcoming version an instance of the expectation is created prior to allowing the interceptor to 
handle the actual method. With the return types from one method to the next in the chain being different than the return type of the calling 
delegate I was fearful that my dreams of no state and returning “smart” expectations were lost.

As it turns out this was mitigated through the use of the stack. When a expectation is created it is pushed onto a stack belonging to the mocked object. 
When the interceptor handles the method call it is able to check the mocked object’s stack for the latest expectation. 
Implementing the Recursive Mock feature came down to creating the “next” expectation on the fly.

There is a little bit of code clean-up required to make the implementation a little easier to read but beyond that… Recursive Mocking has been implemented.

Next up: Property Behavior.

As it stands right now, this is way more trouble than it’s worth.

