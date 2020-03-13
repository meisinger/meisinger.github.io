---
layout: single
title: "If you give a Rhino a cookie…"
date: "2013-07-03"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

There is a book that I use to read to my daughter titled “If You Give a Mouse a Cookie.” 
It’s a funny book which takes the reader through a circular pattern of what happens if you give a mouse a cookie. 
My daughter would often wait till the end of the story to ask what happened next. 
In my mind this was a clever attempt to have me read the book again (and again). 
There were sometimes, however, I would re-read the story; especially after a loving “pleeaaassseeee”

As I have been “healing” the existing unit tests, this book came to mind as I begin to see a “circular pattern.” 
Granted this doesn’t produce anything wrong or exhibit poor code but… 
I can see where these “circular patterns” have lead to confusion and claims of Rhino Mocks being overly complex.

A good example of this can be found with the **Expect** and **SetupResult** utility classes versus the `Expect` extension method.

The `Expect` extension method was added as a part of the Arrange Act Assert (AAA) syntax. 
Since I prefer the AAA syntax, this is the method that I use the most. Let’s take a quick look at an example:

```csharp
[Fact]
public void Test_One() {
  var mock = MockRepository.GenerateMock<ISomeClass>();

  mock.Expect(x => x.SayHello())
    .Return("Hello World");

  var hello = mock.SayHello();

  Assert.Equal("Hello World", hello);
  mock.VerifyAllExpectations();
}
```

Unfortunately there is nothing stopping a developer from using one of the other two approaches. 
The following two examples show how the **Expect** and **SetupResult** utility classes are (basically) interchangeable.

```csharp
[Fact]
public void Test_Two() {
  var mock = MockRepository.GenerateMock<ISomeClass>();

  Expect.Call(mock.SayHello())
    .Return("Hello World");

  var hello = mock.SayHello();

  Assert.Equal("Hello World", hello);
  mock.VerifyAllExpectations();
}

[Fact]
public void Test_Three() {
  var mock = MockRepository.GenerateMock<ISomeClass>();

  SetupResult.For(mock.SayHello())
    .Return("Hello World");

  var hello = mock.SayHello();

  Assert.Equal("Hello World", hello);
  mock.VerifyAllExpectations();
}
```

What isn’t obvious, however, is that these two tests will fail. The two worlds simply don’t play nice together. 
It is true that they can co-exist and you could modify the tests above to make them pass but… is it worth it?

In other words, if you give a developer a cookie…

Rhino Mocks will be going on a diet and won’t be offering any more cookies. 
No matter how lovingly I hear “pleeaaassseeee”

