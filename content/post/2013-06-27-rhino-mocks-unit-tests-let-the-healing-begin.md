---
categories:
- Development
date: "2013-06-27T00:00:00Z"
read_time: true
tags:
- Rhino Mocks
title: Rhino Mocks Unit Tests (let the healing begin)
---

Over the past few days I have been focusing on “removing” the Record/Replay functionality from Rhino Mocks. 
I am a little surprised of how easy it was. The first step was to change all of the public methods in the **MockRepository** class to internal. 
The second step (and last step) was to add some methods to generate repositories. 
Granted, this doesn’t actually remove the Record/Replay functionality but it does “hide” it. _Clearly taking baby steps._

Attempting to build everything and run the unit tests… not so much. _Definitely got the “red” part down._

So I did the only thing that seemed reasonable, exclude every single class from the unit test project. 
I would have loved to have stopped right there and called it a day but (as with all good unit testers); no “green,” no “go.”

Now I am in the middle of changing every test, one-by-one. It is amazing how many of the tests utilize the Record/Replay style. 
It actually feels like every single test.

So far, the stats are as follows:
- Original Passing Tests: 806, Original Skipped Tests: 2
- Current Passing Tests: 461, Current Skipped Tests: 6

As I modify these tests (attempting to maintain the original intent), I have been noticing some of the constructs simply don’t make sense any more. 
Things like the `BackToRecord` and `Replay` methods as well as the **Expect**, **LastCall**, **SetupResult** and **With** utility classes. 
And don’t get me started on **OrderedMethodRecorder** and **UnorderedMethodRecorder**.

I am confident that these constructs will find their place in history soon enough. 
But first… the unit tests must be cured. Then the real slicing can begin.
