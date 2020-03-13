---
layout: single
title: "Planning the Next Rhino Mock"
date: "2013-05-29"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

It has been a week since I have been given the “keys” to the Rhino Mocks framework and it has been an eye opening experience, to say the least. 
It was great to receive encouragement and interest from so many of you. 
It has empowered me to do even more while at the same time added a little more pressure to not screw up (let’s see how that works). 
Not wanting to waste a minute, I cracked my knuckles and dove into the code base. 
I have been in the code base before but this time there was something different to the way I looked at it… from the perspective of change.

Admittedly there is some hesitation to critique the code base (as some of you may be aware of \[grin\]). 
It is clear, however, that the code base has been built upon over time; adding new functionality and features on top of the original implementation. 
A framework that can survive over time (and versions of the CLR) in this way is impressive. 
With that being said, it is also an unfortunate reality.

So… on to the plan…

First and foremost, the code base needs to be simplified resulting in a reduced number of extensions and “alternatives.” 
It is time to pick one: Record/Replay or Arrange/Act/Assert. My preference is with Arrange/Act/Assert (or AAA) as it produces unit tests that are 
easier to read and comprehend.

This will take some work as the AAA approach is actually built on top of the Record/Replay functionality (which is not surprising given its history). 
There will need to be an effort in which the Record/Replay is internalized and eventually abstracted away as state machine with key touch points 
exposed in which the AAA layer interacts with. _Baby steps… baby steps…_

Secondly, the initial .Net Framework versions to be targeted will be 3.5, 4.0 and 4.5. 
This is important to note as the existing code contains older constructs to support 2.0 (and 1.1 possibly) as well as preprocessor directives to omit functionality. 
These older constructs can (and should) be swapped out with newer ones that better target these “initial” .Net Framework versions.

Rather than depending on preprocessor directives, [psake](https://github.com/psake/psake) can be utilized to target different versions. 
If you haven’t heard or used [psake](https://github.com/psake/psake), I strongly suggest you take a look. 
It is currently being used to generate the AssemblyInfo.cs files on the fly, run ILMerge and execute unit tests. 
It can also be leveraged to target different .Net Framework versions to the point where certain projects can be included or excluded from a build. 
It may even be able to be leveraged to support a Mono build (given a separate code set potentially).

This (initial) plan is somewhat dramatic but important to create a better foundation to build upon. 
Not wanting to get too far ahead of myself, it is time to internalize Record/Replay…

If you have any feedback or input, let me know here or in the [mailing list for Rhino Mocks](https://groups.google.com/group/rhinomocks).

