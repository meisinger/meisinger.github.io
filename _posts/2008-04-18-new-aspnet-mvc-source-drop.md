---
layout: single
title: "New ASP.Net MVC Source Drop"
date: "2008-04-18"
read_time: true
categories: [Development]
tags: [ASP.Net, MVC]
---

[Scott Guthrie](http://weblogs.asp.net/scottgu/default.aspx) recently posted about the 
[new ASP.Net MVC Source Preview](http://weblogs.asp.net/scottgu/archive/2008/04/16/asp-net-mvc-source-refresh-preview.aspx) and I have to tell you that 
I am really excited about these new bits. Not only have they added the source code for the unit tests with this release but that have added two new 
features that I think are really going to make life easier.

If you have played around with ASP.Net MVC, or have read articles covering it, you will know that one of the key features to ASP.Net MVC is the support 
for unit testing in the Presentation tier. Prior to ASP.Net MVC you could perform some unit testing in the Presentation tier but often times you were 
required to use a record/playback mechanism that did nothing more than record the HTTP request, store the data in some XML or other file format, and 
allowed you to "replay" the scenario. More times than not, however, the amount of time and effort to test this tier was too much to make it a common everyday 
task that developers would follow.

In the first few releases of the MVC framework the ability to create and execute unit tests has been made a lot easier and a ton more manageable. 
The biggest complaints out there were that you had to using a "mocking" framework or stub out the entire HttpContext to test the Controllers. 
And while Preview 2 of the MVC framework has even made this easier with the IViewEngine and the HttpContextBase class.

Well... now comes the latest bits. Now in the latest drop you don't need to mock anything or create a fake ViewEngine. 
This is all thanks the to the ActionResult class which is returned for every method call in a Controller class. Not only does this new class make it even easier 
to unit test your Presentation tier but it also opens up a bunch of other opportunities since the ActionResult class contains the ViewData object as well.

But wait... it get's better (boy... do I sound like a late night commercial or what)

This release also contains some changes that have been made to the URL Route Mapping feature. 
In these bits you can now include characters like "-" and "." in your route. The most impressive example that was given in Scott's article was the file extension 
or format example. In this example Scott shows how you can produce a route which would provide an extension to determine the what format to use when rendering the data.

For example, the route "products/browse/{category}.{format}" would pass the "category" and the "format" to the Controller. 
Now imagine that we use a URI like "products/browse/cars.xml". This would not only tell the Controller to go and get the data 
for the "cars" category but it would also let us know that the user is expecting this in an XML format. Now consider a URI like 
"products/browse/cars.json" or "products/browse/cars.yaml". The Controller would be able to determine the right View to execute based on format. 
Or perhaps we would be able to advantage of the ActionResult and create a Factory which based on the format returns a sub-class of 
ActionResult which we can filter on.

I can only imagine what they are coming up with for the next drop... if it is anything like this... it's going to be good.

