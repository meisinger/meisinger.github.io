---
layout: single
title: "Simplifying the Rhino Mocks Code Base"
date: "2013-07-25"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

It has been an interesting last couple of days.

I finished converting all of the existing unit tests from using Record/Replay syntax to the Arrange/Act/Assert (AAA) syntax. 
Oddly enough this lead to a few extra unit tests that were not previously being executed. 
These newly discovered unit tests could have been removed due to relevance or performance but for whatever reason, they have been added back. 
I did find that there were a few (eleven in fact) that were no longer relevant and needed to be “skipped.” 
They are all passing now so… mission accomplished.

With this done I started turning toward the main code base. And… here we go.

### Intercepting the Interceptors

Things started out simple enough; moved similar concerns into a different namespace, removed “generated” files minor code corrections. 
I then moved onto simple refactoring like reducing nesting and leveraging `TryGetValue` on a **Dictionary**. 
Everything was going well. I then started to tackle the interceptors and invocation actions.

If you didn’t already know, Rhino Mocks (as with many other frameworks) utilizes [Castle Windsor](http://www.castleproject.org/) to create proxies 
and intercept method calls (via **DynamicProxy**). This gives us the ability to create proxies for interfaces, delegates and classes that can then have calls intercepted. 
Some of the methods, however, are from the base **System.Object** class which everything in the .Net framework inherits. 
As it turns out, you can actually configure Castle Windsor to ignore these methods. 
If any method is ignored, however, they are never called which leads to some interesting issues with this like equality and comparison.

Rhino Mocks current solution for this is to intercept these calls and check to see if the method belongs to **System.Object**. 
If the method does belong to **System.Object** then the method is allowed to _proceed_. This type of “checking” continues for things like properties and events.

I was a little taken back when I came across this. Not because it was being done but how it was being accomplished. 
As I started to refactor this code I came to the realization that [Castle Windsor](http://www.castleproject.org/) supports something very similar through something 
called the **IInterceptorSelector** interface. When this interface is implemented it allows you to choose which interceptor should handle the method. 
By adding an implementation of this interface and a few more interceptors, all of the existing code and constructs can be removed.

### One Step Forward, Two Steps Back

It would have been great to stop right there. As you might have guessed; that didn’t happen.

After this minor success I turned my focus on the **MethodOptions** that are returned when you create an expectation. 
I decided that since the system knew when a `Void` method was being called there should be no reason to return **MethodOptions** with a `Return` method available. 
Along the same lines, `Return` method (when it is available) should be allowed to take a **Func** to return the value.

Diving into this lead me to understand Record/Replay is much more than just a syntax but is the only way Rhino Mocks currently works. 
When using the AAA syntax, behind the scenes, Rhino Mocks transition the “state” from Replay to Record and back to Replay. 
Kind of sounds like a lot of steps doesn’t it? Just a tad.

Rather than perpetuating the current approach, a simpler strategy is underway. 
This strategy does away with the various “states” and puts the onus on the code creating the expectation. 
The AAA syntax conforms to this nicely since there is really only one way create a expectation. 
Based on my estimates, this strategy should reduce the code base by around 30 classes. 
Clearly there will be additional classes added but will be geared towards usability rather than internal plumbing.

I am planning on having a working code set with a majority of the changes completed within the next few weeks. 
Some of these modifications can be done in the existing code base in which case I’ll cut a minor release. 
Should be exciting!

