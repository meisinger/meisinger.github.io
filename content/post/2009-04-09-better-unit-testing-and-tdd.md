---
categories:
- Development
date: "2009-04-09T00:00:00Z"
read_time: true
tags:
- TDD
- Unit Testing
title: Better Unit Testing and TDD
---

I just got done watching [Rob Conery's](http://blog.wekeroad.com/) latest installment in his [Kona](http://blog.wekeroad.com/kona/kona-2) series. 
I have to say that I like the direction that he is going with it and I hope that he continues producing great screen casts.

After watching the video, however, I noticed some of the comments that were made by others and started to get really frustrated over how developers 
view unit testing and practicing TDD.

_I am not saying that I am the end-all-be-all when it comes to unit testing and TDD. 
Far from it... lord knows that I have worked on projects where I didn't produce a single unit 
test and instead relied on firing up a browser and stepping through the application to "test" the code._

I guess the thing that frustrates me the most is when a developer decides to start practicing TDD or they decide that there is enough 
benefit from writing unit tests only to later throw their hands up in disgust and (understandably) frustration when it is not working 
the way they thought it would or how some other blogger showed them how easy it was to write unit tests and practice TDD.

It is my belief that most books, articles, screen casts, blog posts, etc... about unit testing and TDD take the wrong approach. 
Sure, it would be totally awesome if we could all be put on "green field" projects and new code bases where we are starting from scratch. 
The reality is, however, that most of us are brought on to existing projects and existing code bases where no testing has been done or if 
it has... it has been abandoned.

So what is a developer to do? The answer is really simple... write unit tests and start practicing TDD.

The one part of Rob's screen cast that I liked was when he (quickly) covered the unit tests that he wrote for the existing Shopping Cart application. 
While I wish he would have spent a little more time on it, I think that he really hit on what a developer should be doing when trying to write unit 
tests and performing TDD on an existing code base.

### Start from the Story and Requirements

The point of the matter is that if you start from the story, requirements, or business case then you can start to get a clearer understanding of what 
it is that needs to be tested and where to start (regardless of if this a green field project or an existing code base).

Some times you might get stuck and not know where to start even after gathering the requirements or reading the stories. 
This is a clear indication that you don't know the full story in which case you have a couple of options: 1) assume that you know what is going on or 2) ask. 
I think where a lot of people get into trouble is when they start assuming things.

### Assumptions - How bad Tests start

Let's watch what happens when you assume (besides the age old saying of making an ass out of you and me).

You receive (or you read) a story which states "the Company \[object\] must have the ability to add a User \[object\]" and you go off and write the following code:  

```csharp
[Test]  
public void When_Adding_User_UserCount_Is_1() {  
  var customer = new Customer();  
  var user = new User();  
  
  customer.AddUser(user);  
  Assert.AreEqual(1, customer.UserCount);  
}
```

After you fix the compile errors, etc... you run the test and it fails since you have not implemented the **AddUser** method yet. 
So you then start to implement the method and you assume that you are going to store the User in a collection. 
You then further assume that you are going to want to retrieve the User so you decide to create a Dictionary. 
You make another assumption that the Id property from the User will be used as the key for the Dictionary. 
Remembering that you still have to satisfy the test, you change the **UserCount** property to return the count from the Dictionary. 
Then finally... you make one last assumption, the **GetUser** method. 
You write and implement the **GetUser** method to take an Integer and return the User object from the Dictionary. 
Just to be safe, you write a test to prove that after you add the User, you can get the User from the Company object.

### Assumptions - So what went wrong?

So what went wrong here? Everything works and all of your tests pass so... what is the deal?

The assumptions are the deal. First you have assumed that you need to store the User in a collection. 
Nothing about the story or requirement indicated that this needed to be done. After that assumption comes all of the other decisions that were not warranted. 
Finally the big nasty assumption that the User will need to be retrieved and the API created to accomplish it.

With all of those assumptions made, you have coded and unit tested yourself and every other developer on your team (present and future) into a corner. 
Not only is there no need to have a **GetUser** method at this time but you run the risk of the method becoming "supported" because other developers 
now use it to get User objects.

What is worst of all is that there is code now available that isn't backed by a story or requirement.

**Conclusion**

So back to my original point. Writing unit tests and practicing TDD is not as hard as people make it out to be as long as you follow the stories and 
requirements and don't make assumptions.

Perhaps that is wrong to state. Assumptions are fine when you are assuming how something will work like how the Customer object will be reconstituted 
from a database some where. The details aren't important but it is safe to assume that there will be some data access method some where in the framework 
or code that will handle the retrieval of the Customer object for you. Other assumptions that are integration assumptions are also fine since the details 
have more than likely not been ironed out yet.

Assumptions about the test and the functionality needed for the application, however, are bad

