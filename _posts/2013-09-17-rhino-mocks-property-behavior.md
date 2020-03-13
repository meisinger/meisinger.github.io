---
layout: single
title: "Rhino Mocks Property Behavior"
date: "2013-09-17"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

One feature that I seldom use in Rhino Mocks is the “Property Behavior.” 
I remember getting an exception every now and then when I would use a property but never really thought much of it 
(like a mindless drone I would change the code and run the unit tests again).

In fact, I used properties so infrequently in my unit tests, it wasn’t until I started diving deep into the code that I came to realize there are 
three “property” based options:
- `PropertyBehavior`
- `SetPropertyAndIgnoreArguments`
- `SetPropertyWithArgument`

After working through the Recursive Mock feature I found that properties are fairly easy to deal with. 
That is until I tried to setup an expectation for a property based on the value being set.

As it turns out, when a property is used in a delegate the signature has a return value regardless of the operation. 
This makes thing really difficult to handle when the expectation is being created for a “write-only” property.

The solution has been to create a new type of expectation that handles properties specifically. 
The expectation captures both the `get` and `set` method at the same time. This allows properties to act more like first class expectations without 
needing to add \_special\_ extensions.

Granted, to utilize the new expectation you will need to call `ExpectProperty` rather than `Expect`. 
So I guess that is a \_special\_ extension but… you get the point. 
There are still some things that need to be considered like whether or not properties should have Repeat options but for the most part it’s done.

One thing that I think is important to point out is the original handling of a property through the `Expect` method remains the same. 
It’s only when extra functionality is needed do you need to think about utilizing the `ExpectProperty` method.

If you’re anything like me, this feature will be useless. Until you need it.

Next up; events…
