---
title: "Rhino Mocks Core"
date: 2025-08-16T11:49:03-04:00
read_time: true
categories:
- Development
tags:
- Rhino Mocks
---

I have been itching to get back to working on Rhino Mocks and I finally have a chance to do so.

The next version of Rhino Mocks will be a complete rewrite moving to dotnet core.

One thing that I have always had difficulty with is the use of the Castle DynamicProxy library. It has always been a lot of hand waving and magic that I never really understood. I also always felt that it was using a sledgehammer to push in a thumbtack.

The approach I am taking is to create a new core library named "Rhino Proxy" that will handle the creation of proxies and the interception of method calls. This will be a much more focused and purposeful library that will simplify the integration and call sites of Rhino Mocks.

With Rhino Proxy taking over the responsibility of creating proxies, the Rhino Mocks library will need to be changed (naturally). Rhino Proxy will be much more lightweight but might require Rhino Mocks to pick up more of the slack in terms of intercepting method calls and managing expectations.

The little that I do remember about the Castle DynamicProxy library is that it had more features for things like Aspect Oriented Programming (AOP) and other advanced features. I don't think I will be going that route with Rhino Proxy. Instead, I want to keep it simple and focused on the core functionality of creating proxies and intercepting method calls.

The code that I have so far does the basic things that I need right now. It can create a proxy for an interface and intercept method calls. I am still working on the details of how to manage expectations and return values, but I am confident that I can get it working.

Never the less, I am excited to get back to working on Rhino Mocks and I hope that this new direction will make it easier to use and understand.
