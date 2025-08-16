---
categories:
- Development
date: "2010-08-21T00:00:00Z"
read_time: true
tags:
- Architecture
- NServiceBus
title: Head in the Clouds
---

The past couple of months have been really fun as we have moved our whole product (lock stock and barrel) out into the Cloud. 
Along the way I have learned a couple of things about the Cloud and myself that I think (clearly) is enlightening.

_Its weird, actually. It seems like I have been going through a renaissance of sorts lately with everything that has been going on. 
Perhaps it has more to do with the epiphanies and "ah ha" moments as of late but in either case, plenty of topics to cover so let's not waste any time._

It was about four or five months ago when my boss came up to me and said "We're dumping our managed hosting provider and moving everything out to Amazon." 
After I picked my jaw up off the ground I got a little pain in my gut from the uncertainty of our current product being able to move to the "cloud." 
Then it felt like I got kicked in the stomach when he followed up with "I want customers to access it by the end of June."

At this time I had been with the company for about nine months and had taken over a part of our product that was developed with a very academic SOA 
approach to the point were nearly everything had to be a web-service (using WCF of course). One solution had 130 projects and zero unit tests.

Needless to say, the current product was not going to work in the Cloud so a re-architecture was needed. Later posts will cover more of the specifics of how 
this was done/achieved but one thing that sticks out in my mind is our solution for messaging based services.

Initially the thinking was that we would go with [NServiceBus](http://www.nservicebus.com/) out in the Cloud and have these sweet autonomous services with 
publish/subscribe capabilities and dispatching goodness. This way we could dump the WCF web services, flatten the web applications and have these event-like services.

I was really excited about this opportunity since I had come off of a deep dive with [NServiceBus](http://www.nservicebus.com/) and was already considering a 
re-architecture of sort when the request to move to the Cloud was made.

As you might have gathered from earlier, the plan was to use Amazon EC2. With my experience in the Cloud being all of nothing, off I went to research and get 
a handle on this whole Cloud thing. It took all of about ten minutes after bringing up a couple of instances that I found that 
[NServiceBus](http://www.nservicebus.com/) was not going to be a viable solution (_i am still determined to get it working out there_).

**\[Update: It should be noted that the reason behind not using NServiceBus had more to do with the lack of knowledge with the Amazon EC2 than NServiceBus itself. 
To be fair to NServiceBus...\]**

With things I learned just from reading up on [NServiceBus](http://www.nservicebus.com/) and reading [Udi Dahan's](http://www.udidahan.com/) articles however 
the decision to move towards an autonomous message based solution for our services was still crystal clear.

So Amazon SQS was used in lieu of MSMQ and "messaging service" windows services were written in lieu of [NServiceBus](http://www.nservicebus.com/). 
Granted, the whole publish/subscribe thing is a manual effort and there is no dispatching goodness but that's not the point.

So what's the point?

Just because you can't use a framework or tool set doesn't mean that you can't use the principles and ideas behind them. 
[NServiceBus](http://www.nservicebus.com/) was a "no go" out in Amazon but implementing a messaging based service is still an amazingly strong approach.

Another point that I think applies to the overall experience is that moving to the Cloud doesn't mean that you have to change the way you think about 
solutions but rather if you have to make massive changes, modifications, and re-architect your solutions you more than likely didn't have a very 
strong one to begin with.

