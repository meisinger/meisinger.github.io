---
layout: single
title: "Rhino Mocks 4.0.0 Alpha Released"
date: "2014-03-31"
read_time: true
categories: [Development]
tags: [Rhino Mocks]
---

It has been a little longer than expected but the latest version of Rhino Mocks has been released and is available from [NuGet](https://nuget.org). 
You can download the package from here:

[RhinoMocks 4.0.0-alpha](https://www.nuget.org/packages/RhinoMocks/4.0.0-alpha)

This is an “alpha” release since there are more than likely going to be a few things that come up. 
As a matter of fact, there are two things that I am already thinking about changing.

One is to be a little more explicit with arranging expectations against properties. 
Right now when you arrange an expectation against a property the intent is inferred. 
Instead I think it would be better to be explicit by adding `ExpectPropertyGet` and `ExpectPropertySet`.

The other is more of an internal modification to help track expectations easier. 
Currently an expectation is stored in a single data structure which includes the method, arguments, constraints and return values. 
When a method is intercepted all of the expectations have to be checked to find a match (clearly there are short circuits but… you get the idea). 
This modification will help account for how many times a method has (or hasn’t) be called and increase performance.

Until then, however, enjoy this alpha version and feel free to provide feedback.
