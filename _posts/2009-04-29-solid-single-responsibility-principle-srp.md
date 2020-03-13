---
layout: single
title: "SOLID - Single Responsibility Principle (SRP)"
date: "2009-04-29"
read_time: true
categories: [Education]
tags: [SOLID]
---

As stated previously... I am getting (or trying to get) back to basics.

It is my belief that developers have to often times refactor the way they think and the way they approach things from time to time to 
make sure that the basics and the principles are still strong.

So let's take a look at one of the principles in SOLID; Single Responsibility Principle (SRP)

SRP simply states:

> _There should be one and only one reason for a class to change_

### Um... yeah! That makes sense... I guess

So what types of change are we really talking about when dealing with SRP?

This is a question that I have always had trouble answering for two reasons: 
* I didn't really understand SRP to begin with
* My focus was always in the wrong area

I have always understood SRP through examples of classes that violated SRP in that it would take someone to state that a given class was violating SRP and why. 
Then I would simply remember that pattern or example and avoid it rather than taking the time to learn what it meant. 
When I started taking the time to learn it and gain a better understanding I started to take things too literal. 

_If I needed to change a method was that violating SRP?_

What I started to do was to think about things as a whole. 
In other words, while "change" is a very important word in the definition of SRP, the word "class" was what I was missing. 
When I finally took the word "class" into context... I had my "ah ha" moment (it was really like a "duh" moment).

### Ah ha...!

Ok, great. So what does that mean?

The word "class" is a little tricky beast that can mean so many things to different people at different times that I feel it losses it's importance. 

Case in point, I find my self interchanging the word "object" with "class" as in "new up a StringBuilder class." 
While I don't think that I will ever stop doing that or stop using the word "class" in my everyday vernacular, 
I think that word "class" in terms of SRP needs to be changed to "concern."

So I started to take the common (or the one most popularized) definition of SRP and replace "class" with "concern":

> _There should be one and only one reason for a **concern** to change_

Wait for it... ah ha! By making this contextual change I started to realize that my classes needed to be responsible for a single concern.

Now I could start looking at classes in terms of their concern. 

Thinking about a class that dealt with a Customer object and it's database interactions became a concern. 
And with that thinking I realized that the class had one responsibility, interacting with the database for the Customer object. 

So now the only reason for the class to change was to change the way the Customer object interacted with the database.

