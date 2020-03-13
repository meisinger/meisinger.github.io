---
layout: single
title: "Rhino Mocks Final Stretch–Field Problems"
date: "2014-03-04"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

The final modifications have been made to the Rhino Mocks code base. It has been interesting working through Properties and Events. 
What was more interesting, however, were the delegates. Previously these were Callback, Do and WhenCalled and allowed you to pass in a delegate, 
an Action or a Func to modify the execution flow, change the return value or simply call a delegate. 
Understanding when to use Callback, Do or WhenCalled was confusing to me so I decided to wrap them up a little differently.

Previously, WhenCalled allowed you to execute an Action that took, as a single parameter, a data structure encapsulating the method invocation. 
Since an Action was being invoked the return value of the expectation could not be changed.

In the current version, WhenCalled has been modified slightly to simply execute an Action. 
If the arguments of the calling method are desired then the Action will need to have the same signature of the calling method. 
One thing that I am continuing to work through is the ability to match the arguments with the parameters of the Action (or delegate) without having 
to specify all of them. It works but not with “out” or “ref” arguments.

The Callback functionality wasn’t something that I used in the past. The exact reason why this functionality was added is also unknown. 
From the call signature, unit tests and the way it was executed I can only assume that it was used to validate the arguments of the calling method. 
This feature has not been brought forward. Not that there isn’t a need but with the available argument constraints I don’t understand the need. 
If the Callback feature is to ever return it will be re-worded to something like ValidateArgs.

“Do” has been replaced with DoInstead and is meant to allow you to replace the return value from the given Func (or delegate). 
I don’t particularly like the “Do” notion and may change it to “ReturnInstead.” 
This may seem like something silly to be considering but the names of the extension methods should reflect their intent as much as possible.

In order to provide a similar feature as the original “WhenCalled”, a new method “Intercept” has been added which expects an Action that takes, as a single parameter, 
a data structure encapsulating the method invocation. Again, not too certain that I like the name of the method. 
I also question whether having access to the method invocation is necessary or not. Only time will tell.

I have updated the existing unit tests to use these new constructs over the old and they all pass. 
So while the code and execution remains the same (or similar), the names of the methods and properly expressing their intent may need some more work.

With all of the “major” unit tests now passing, it is on to the Field Problem unit tests. 
There are only 104 files to go over so hopefully it will go by quickly. I am hoping that all of them can be converted and pass with limited modification. 
I have an uneasy feeling, however, that some may need to be removed. 
It will be important to identify and document what get’s removed in order to speak to them, provide justification and alternatives.

As a side note, someone had asked if this code is on GitHub. The current version that I am working on can be found here:

[Innovation Branch](https://github.com/meisinger/rhino-mocks/tree/innovation)

This will get merged into the “master” branch once the Field Problem unit tests have been added (and passing).

Well… back to unit tests…

