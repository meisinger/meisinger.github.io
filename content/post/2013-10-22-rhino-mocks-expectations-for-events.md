---
categories:
- Development
date: "2013-10-22T00:00:00Z"
read_time: true
tags:
- Rhino Mocks
title: Rhino Mocks Expectations for Events
---

If one of the features I don’t use very often is the [Property Behavior]({% post_url 2013-09-17-rhino-mocks-property-behavior %}) then it’s safe to say that 
I have never (ever) used “events” within Rhino Mocks. 
I have seen the `Raise` and `GetEventRaiser` extension methods in the passed but never used them.

After working through the Property Behavior feature I decided to tackle handling events. 
I found setting up expectations for events to be really easy. 
Why? Because there is nothing to them. No really. There is nothing to them at all. 
The only thing that you can do with events is validate (or expect) subscribing or unsubscribing from an event.

Think about that for a second.

When subscribing to an event you (typically) will use the “+=” assignment operator. 
When unsubscribing from an event you (typically) will use the “-=” assignment operator. 
Under the hood you are calling the “add” and “remove” methods respectively but there really isn’t anything you can do beyond that. 
Both methods are “void” which means there is nothing to return or “mock" to return. 
Sure, you could throw an exception but I find that to be an extreme edge case that should never be tested. 
The only other thing you could do is perform a type of callback after subscribing or unsubscribing.

So really, when it comes down to it, the only thing you can do is setup an expectation to validate that an event handler was added or removed. 
As a part of that expectation you may choose to make the event handler very (very) specific.

The one thing that I don’t like about the current implementation is that you can use **null** as the event handler when setting up the expectation. 
Unless you follow that up with a call to the `IgnoreArguments` method, the expectation will be looking specifically for **null** to be passed. 
I can’t tell if this is actually useful to anyone or not. 
Perhaps instead **null** can be used to represent anything and thus avoiding the need to call `IgnoreArguments`.

Just as with the other refinements, there is a new extension method specifically for events named **ExpectEvent**:

```csharp
[Test]
public void New_Event_Extension_Method() {
  var mock = Repository.Mock&lt;IConnection&gt;();
  mock.ExpectEvent(x => x.OnShutdown += null);
  ...
  mock.Verify();
}
```

The options that are returned are greatly reduced. So much so that `GetEventRaiser` has been removed as well. 
Right now I am thinking that the `Raise` extension method works better. 
And speaking of the `GetEventRaiser` and the `Raise` methods (which have not been until now), these methods are simply helper methods for calling the event. 
Nothing more and nothing less.

Having never used events within Rhino Mocks caused a little bit of delay due to the confusion. 
The exact things that I am trying to correct are some of the reason for the confusion. 
By making the extension methods explicit and the" “fluid” interfaces more specific will definitely help.

Up next: Wrapping up.

