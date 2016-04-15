---
layout: post
title: Thoughts on C# 7 Local Functions
tags: chsarp, local functions
---

Frankly when I first saw that C# 7 will come with new local functions I thought that that's just a nice and 
compact way of defining local helpers.
In fact it's much more interesting and useful feature. Today I'm going to talk about it in more details.

Let's start with a brief overview of the current situation.

## Current options 

### Private methods

The first option that existed in C# 1 is having a private method.

``` csharp
class Bar
{
    public void Foo(int[] numbers)
    {
        foreach(var i in numbers)
        {
            Console.WriteLine(AsPrintable(i));
        }
    }
    
    private string AsPrintable(int i) =>  $"I have {i} here";
}
```

That's a clean and simple solution. It has few issues though. 

`AsPrintable` might have no sense outside of `Foo` method, but it's accessible for every other method inside the class.
It will be taken into account by IntelliSense.

![Private method discoverable via IntelliSense](/images/thoughts-on-local-functions/private-method-discoverable.png)


### Func and Action

We can try to hide our helper inside the scope of `Foo` method by converting it to `Func<int, string>`:

``` csharp
 class Bar
 {
     public void Foo(int[] numbers)
     {
         Func<int, string> asPrintable = i => $"I have {i} here"; 
         foreach (var i in numbers)
         {
             Console.WriteLine(asPrintable(i));
         }
     }
 }
```

Any disadvantages? Yep, a lot. 

The call is unnecessary expensive: it will produce [a couple of allocations](http://blogs.msdn.com/b/pfxteam/archive/2012/02/03/10263921.aspx).

There is no elegant way to define recursive lambda: 

![Lambda can not be recursive](/images/thoughts-on-local-functions/func-factorial-not-compilable.png)

We have to use an ugly trick:  

![Lambda can not be recursive](/images/thoughts-on-local-functions/func-factorial-ugly.png)

And the last, but not least is the fact that lambdas are very limited. You cannot use  
 `out`, `ref`,`params` and optional parameters. They cannot be generic. 
 
There is a bright side, though, a lambda can capture variables from enclosing method aka clousure.
 
## Local functions

Local functions can be defined in the body of any method, constructor or property's getter and setter.

Since it will be transformed by compiler to regular private static method: 

![Local function decompiled](/images/thoughts-on-local-functions/local-function-decompiled.png)

there will be no overhead of calling it and it can be have all the properties which regular method declaration can have: 
it can be asynchronous, it can be generic, it can be dynamic. 

Ok, there is a difference. Local functions can not be static. And local functions can capture variables from enclosing block:

```csharp
class Bar
{
    public void Foo(int[] numbers)
    {
        var length = numbers.Length;
        string Length() => $"length is {length}";
        Length();
    }
}
```

## Useful bits

As [Bill Wagner](http://thebillwagner.com/Blog/Item/2016-03-02-C7FeatureProposalLocalFunctions) wrote local function are perfect solution for iterators and asynchronous methods when it comes to parameters validation.

The following code will throw an exception right away and not in lazy maner.

```csharp
public IEnumerable<T> AsEnumerable<T>(params T[] items)
{
    if (items == null) throw new ArgumentException(nameof(items));

    IEnumerable<T2> Enumerate<T2>(T2[] array)
    {
        foreach(var item in array)
        {
            yield return item;
        }
    }

    return Enumerate(items);
}
```


## Other observations: 

Local functions support [Caller Info Attributes](https://msdn.microsoft.com/en-us/library/hh534540.aspx)

``` csharp
 public static void SlimShady()
 {
     void Hi([CallerMemberName] string name = null)
     {
         Console.WriteLine($"Hi! My name is {name}");
     }

     Hi();
 }

```

Fun fact. You can declare local function with same name as an existing private method and local function will hide it. 
Same as for lambdas though.

![Local function has priority over private method](/images/thoughts-on-local-functions/local-function-priority-over-private-method.png)

## Links

* GitHub
* [Bill Wagner. C# 7 Feature Proposal: Local Functions](http://thebillwagner.com/Blog/Item/2016-03-02-C7FeatureProposalLocalFunctions)