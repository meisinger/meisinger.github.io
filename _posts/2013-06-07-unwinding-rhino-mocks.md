---
layout: single
title: "Unwinding Rhino Mocks"
date: "2013-06-07"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

Finally, progress!. Sure, the “progress” is 124 build errors but… it’s progress; right?

One of the things that I have been doing is cleaning up the code base by removing projects, assemblies and code that are no longer relevant. 
I am actually surprised of how much code was never built. It has also been interesting finding artifacts from previous build strategies and repositories. 
Things like NAnt scripts and merge diff files (from SVN). Even found an old version of NUnit hiding out.

_To be honest, this is some of the most fun that I have had in a while. There is nothing better than deleting things!  
(Perhaps I need to get out more)_

Attempting to update to the latest version of Castle Dynamic Proxy (now in Castle.Core assembly), however, has caused a little problem. 
About 124 problems to be exact. Armed with my trusty editor (Vim), these issues have been quickly dispatched.

With the build errors fixed there are now nine tests failing (out of 806). Progress indeed. 
Hopefully these will be corrected tomorrow so I can commit the changes and start digging into some “real” changes.
