---
date: "2015-08-02T00:00:00Z"
tags: 
    - programming
    - resharper
title: AsyncSuffix ReSharper extension
slug: async-suffix-resharper-plugin
---

There is a tendency in a .NET world to build asynchronous CPU-bound or I/O-related API. We also can see that some APIs support both asynchronous and synchronous versions for backward compatibility reasons. 

That puts us in a situation where we should be able to distinguish between async and non-async versions of a method. [Microsoft suggests](https://msdn.microsoft.com/en-us/library/hh873175.aspx) we follow the naming convention where every asynchronous method includes an Async suffix after the operation name. 

There are some questions about this guideline. What do we call an asynchronous method? Is it the one with the ```async``` keyword?

Well, [according to Stephen Toub](http://stackoverflow.com/questions/15951774/naming-convention-for-async-methods
) it’s the Task-returning method in general (with a few exceptions).

>If a public method is Task-returning and is asynchronous in nature (as opposed to a method that is known to always execute synchronously to completion but still returns a Task for some reason), it should have an “Async” suffix. That’s the guideline. The primary goal here with the naming is to make it very obvious to a consumer of the functionality that the method being invoked will likely not complete all of its work synchronously; it of course also helps with the case where functionality is exposed with both synchronous and asynchronous methods such that you need a name difference to distinguish them. How the method achieves its asynchronous implementation is immaterial to the naming: whether async/await is used to garner the compiler’s help, or whether types and methods from System.Threading.Tasks are used directly (e.g. TaskCompletionSource) doesn’t really matter, as that doesn’t affect the method’s signature as far as a consumer of the method is concerned.

Unfortunately, developers tend to forget about this guideline as it’s not so natural. I noticed that this topic pops up quite often during the code reviews. What do lazy developers do in this situation? They automate the routine.

Since we’re using ReSharper, it seems to be an obvious solution to delegate this task to it.

ReSharper supports [custom code inspections](https://www.jetbrains.com/resharper/help/Code_Inspection__Creating_Custom_Inspections_and_QuickFixes.html) and [we can build the new inspection](http://stackoverflow.com/a/22509370/555014). 

This is a good solution, but it has a few disadvantages. First of all, it highlights the whole method with all the expressions in it, and it is very annoying when you work with legacy code and can’t change the method name. 

![Custom inspection in action](/images/async-suffix-plugin/custom-inspection-in-action.png)

Also, it's not easy to skip override methods from analysis. 

So I’ve built an extension for ReSharper and named it AsyncSuffix. You can grab it from the [official NuGet feed](https://resharper-plugins.jetbrains.com/packages/Sizikov.AsyncSuffix/).

This extension analyses the task-returning method declarations, and suggests the Rename refactoring when it detects that the ‘Async’ suffix is missing. It also has an option to exclude test methods from the analysis. By test methods I mean the methods with the ```[Test]```, ```[Fact]``` or ```[TestMethod]``` annotations. So, as you see, it supports only NUnit, MSTest, and xUnit.Net frameworks. This feature is turned on by default, but you can turn it off if you'd like.

![extension in action](/images/async-suffix-plugin/in-action.gif)


The extension code can be found on [GitHub](https://github.com/asizikov/AsyncSuffix
). 

It’s a first version of the plugin, so feel free to [raise an issue](https://github.com/asizikov/AsyncSuffix/issues) if you see any.
