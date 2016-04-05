---
layout: post
title: ReSharper Annotations. Can it be null?
tags: resharper, annotations, null, notnull, async
---

Hi,

ReSharper is doing great job when it comes to semantics of your code and control flow graph analysis. The special edge case I want to talk about is nullness analysis.

Unfortunately sometimes it's hard to predict whether the method returns ```null``` or it doesn't.

To solve this problem R# provides an option to [annotate your code](https://blog.jetbrains.com/dotnet/2015/08/12/how-to-use-jetbrains-annotations-to-improve-resharper-inspections/). 

Let's look at this snippet:
{% highlight csharp %}
public Bar Foo()
{
    return Random.NextDouble() < 0.1 
                    ? null
                    : new Bar();
}
{% endhighlight %}

The```Foo()```returns```null```with a probability of 10%. However ReSharper doesn't warn us when we forget the null-check.

![screenshot](/images/resharper-annotations-can-it-be-null/no-canbenull-in-action.png)
_<font color="gray">without annotation</font>_

Developer can mark the code with```[CanBeNull]```attribute in order to give a hint to ReSharper.

![screenshot](/images/resharper-annotations-can-it-be-null/canbenull-in-action.png)
_<font color="gray">with annotation</font>_

Unfortunately this trick doesn't work with asynchronous code. 

What if we have a code like this:
{% highlight csharp %}
[CanBeNull]
public async Task<Bar> FooAsync()
{
    await DoAsync();
    return Random.NextDouble() < 0.1
                    ? null
                    : new Bar();
}
{% endhighlight %}

Despite the fact that we return```null```here the actual return type is```Task<Bar>```. Which is not null by the way. So the annotation is wrong as well as the warning given to us by R#.
![screenshot](/images/resharper-annotations-can-it-be-null/no-canbenull-async.png)
_<font color="gray">attribute applied to Task, but no to Bar</font>_

I was always missing the ability to express that it's not the```Task<T>```which can be```null```, but the```Task<T>.Result```.
There was even an [issue created back in 2013](https://youtrack.jetbrains.com/issue/RSRP-376091) and finally in ReSharper 9.2 we have a support of brand new```[ItemCanBeNull]```and```[ItemNotNull]```attributes.

![ItemCanBeNull in action](/images/resharper-annotations-can-it-be-null/canbenull-async.png)
_<font color="gray">ItemCanBeNull in action</font>_

Both attributes can be applied to```IEnumerable<T>```,```Lazy<T>```and```Task<T>```

![ItemCanBeNull applied to Lazy](/images/resharper-annotations-can-it-be-null/lazy-itemcanbenull.png)
_<font color="gray">ItemCanBeNull applied to Lazy</font>_

Happy annotating!

## Links

* [How to use JetBrains Annotations to improve ReSharper inspections](https://blog.jetbrains.com/dotnet/2015/08/12/how-to-use-jetbrains-annotations-to-improve-resharper-inspections/)
* [Using Annotations to Refine Code Inspection](https://www.jetbrains.com/resharper/help/Code_Analysis__Code_Annotations.html)
* [JetBrains.Annotations NuGet package](https://www.nuget.org/packages/JetBrains.Annotations/)




