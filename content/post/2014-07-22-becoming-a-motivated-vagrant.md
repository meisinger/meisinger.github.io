---
categories:
- Development
date: "2014-07-22T00:00:00Z"
read_time: true
tags:
- VM
- Vagrant
title: Becoming a Motivated Vagrant
---

I have been a long time fan of VirtualBox. Setting up and configuring a VM with a Host Only Network to run headless is something that I feel like I can do in my sleep. 
The one thing that was a constant pain, however, was keeping the VM up to date (with as small of a footprint possible) and keeping everything organized.

For those who aren’t familiar with [VirtualBox](https://www.virtualbox.org/), it is general purpose virtualization manager that can run on fairly limited hardware. 
If you are like me and can’t afford a VMWare installation and just want to standup a virtual machine quickly, VirtualBox is a fast a simple way to get started. 
Just like any other operating system, however, it is up to you to keep things in order and up to date. 
Taking snapshots, managing network interfaces, managing disk space, images, etc. 
This is an invaluable learning experience but after a while it gets a little long in the tooth.

Recently I had a problem installing Apache (something about the “apache2-threaded-dev” and “apache2.2-common”). 
I spent hours trying to rebuild packages, reinstall headers, uninstall headers, rebuild… you get the point. 
I wasted about three hours resulting in starting over from scratch.

I decided to take a look into some simple management and configuration tools for my VM’s to potentially make things a little easier. 
I first looked into Puppet and Chef and began to sink quickly (no swimming here). Both of these are fantastic but a little more than what I was looking to do. 
I didn’t really need _that_ much management after the VM was up and running.

I then came across [Vagrant](http://www.vagrantup.com/).

After reading the first few lines in the documentation, I was hooked.

Within minutes (literally) I had a VM up and running with Redis and RabbitMQ, networked and integrated. 
Thinking this was too good to be true, I decided to destroy that VM and build another one with Apache, Passenger, Sinatra and MongoDB. 
This took a little bit longer but no more than forty-five minutes.

Here is a link to my shell script that I used: [Vagrant Bootstrap](https://gist.github.com/meisinger/619e90e999d84ae55179)

The best part, Vagrant works with AWS, VirtualBox and VMWare. Not only that but it also supports Puppet and Chef. 
So rather than being thrown into the deep end I get to play in the shallow end and learn at my own pace. 
Vagrant can be as simple or as complex as you want and grow with you. Do you just need one VM? No problem. 
Do you want multiple VM’s all connected to each other? No problem.

Now I may be getting to the party a little late when it comes to Vagrant so you will forgive me if this is already old hat.

