---
categories:
- Development
date: "2013-08-13T00:00:00Z"
read_time: true
tags:
- Rhino Mocks
title: Am I Just Getting Lazy?
---

So I have been working on simplifying the Rhino Mock code base for a little while now and have been making good progress. 
While I have no empirical data yet, I fully expect this simplified code base to be faster and easier to understand.

It has been really interesting watching some of the core concepts shift from being “in your face” to a more subtle “nudge” as the code is simplified. 
For instance; generating a **Strict** mock versus a **Dynamic**, **Stub** or **Partial** mock. 
There are some very important concepts behind choosing the right “type” but what are they and how are they presented?

This has actually caused confusion in the past… and rightly so. A **Strict** mock is one where any method call that has not had an expectation set will throw an exception. 
A **Dynamic** mock is one where only those methods that have had expectations set are verified. Those methods that have no expectations simply return default values. 
A **Stub** mock acts just like a **Dynamic** mock but does not perform any verifications. 
A **Partial** mock… well… gives you the ability to call the original method as long as it isn’t `abstract` (and is `virtual` of course).

While this is a greatly simplified overview, it identifies a few problems.

In my mind a **Strict** mock is a verification concern and not a generation (or creation) decision. 
When expectations are being verified I should have the option to perform the verification strictly or not. 
The type of mocked object shouldn’t dictate that choice.

```csharp
[Test]
public void Strict_Verification() {
  var mock = MockRepository.Mock<IPerson>();
  mock.Expect(x => x.GetName())
    .Return("Mike");

  mock.GetName();
  mock.GetAge();

  mock.VerifyExpectations(VerificationType.Strict);
}
```

In my mind a **Stub** mock is an expectation concern rather than a generation decision. 
Either you are setting up an expectation or you are stubbing out the method.

```csharp
[Test]
public void Stub_A_Stub() {
  var mock = MockRepository.Mock<IPerson>();
  mock.Stub(x => x.GetName())
    .Return("stub?");

  mock.Expect(x => x.GetAge())
    .Return(1);

  mock.GetName();

  mock.VerifyExpectations();
}
```

If **Stub** is removed as a “type” of mock object then we also get the added benefit of eliminating the confusion around when to call `Expect` and when to call `Stub`. 
If both **Strict** and **Stub** are removed, does **Dynamic** make any sense?

The **Partial** mock is the "most correct” decision that needs to be made when creating a mocked object. 
A **Partial** mock is a mixture between a concrete class and a derived proxy class that allows me to choose to execute the base method or “mock it away.” 
A **Partial** mocked object, however, should be the only type which would expose the `CallOriginalMethod` method.

```csharp
[Test]
public void Partial_Person() {
  var mock = MockRepository.Partial<Person>();
  mock.Stub(x => x.CalculatePayRate(Arg<int>.Is.Anything))
    .CallOriginalMethod();

  var rate = mock.CalculatePayRate(35);

  Assert.AreEqual(11.25, rate);
}
```

Through simplification of the code Rhino Mocks will now only offer two options to create a mocked object: **Mock** and **Partial**. 
None of the core concepts have been removed but they have been muted a little.

The thing I need to figure out now is whether or not I arrived at this decision because I am too lazy to implement those different “types” or 
because it makes more sense… maybe a little bit of both.

