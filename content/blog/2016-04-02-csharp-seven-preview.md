---
date: "2016-04-02T00:00:00Z"
tags: 
     - programming
title: C# 7 features preview
slug: csharp-seven-preview
---

Last week my twitter feed exploded with lots of entries about Microsoft //Build 2016 conference.
As it's one of the most important events for .NET dev community MSFT prepared quite a few awesome announcements for us:

* [Visual Studio "15" preview](https://www.visualstudio.com/news/vs15-preview-vs)
* C# 7 preview
* [Xamarin going open source and free](https://blogs.msdn.microsoft.com/visualstudio/2016/03/31/mobile-app-development-made-easy-with-visual-studio-and-xamarin/)
* [Ubuntu workspace running natively on Windows 10](https://blogs.windows.com/buildingapps/2016/03/30/run-bash-on-ubuntu-on-windows/)
* And lot more

Since I got a bit sick this weekend I had plenty of time to play with new VS15 and C# 7.

## Getting started

Let's grab a [VS "15" first](https://www.visualstudio.com/en-us/downloads/visual-studio-next-downloads-vs.aspx). 
There is a new installer by the way.


## Enabling experimental features

By default VS15 is using C# 6, so we need to add conditional compilation symbols to our project: 
`__DEMO__`.

To do that you should go to Properties > Build > Conditional compilation symbols
![conditional compilation symbols dialog](/images/2016/04-csharp-7-preview/conditional-demo.png)

Once we're done with that VS15 will pick up changes automatically.

## Features

As of today C# 7 goes with several features:

* Binary literals
* Digit separators
* Local functions
* Ref returns and locals
* Pattern matching

## Binary literals and Digit separators

These are very minor features, you know, nothing to write home about: 
In addition to [existing](https://msdn.microsoft.com/en-us/library/aa664672(v=vs.71).aspx) `int` literals such as hex we can use binary ones.

{{< highlight csharp >}}
class LiteralsDemo
{
  public void BinaryLiterals()
  {
    var numbers = new[] { 0b0, 0b1, 0b10 };
    foreach (var number in numbers)
    {
      Console.WriteLine(number);
    }
  }
}
{{< / highlight >}}

Simple and works as expected.

![binary literals output](/images/2016/04-csharp-7-preview/binary-literals.png)

Same about digit separators. Similar feature exists [in Java](https://www.oreilly.com/learning/java7-features) since version 7. 

{{< highlight csharp >}}
public void DigitSeparators()
{
    var amount = 1_000;
    var thatIsALot = 1_000_000;
    var iAmHex = 0x00_1A0;
    var binary = 0b1_000;
}
{{< / highlight >}}

## Local functions

Local functions can be defined in a scope of a method.

This is something you would do when you need a small helper like this one:
{{< highlight csharp >}}

public void RegularMethod2()
{
  Func<int, bool> even = (number) => number % 2 == 0;
  foreach (var number in Enumerable.Range(0, 10).Where(even))
  {
      Console.WriteLine(number);
  }
}
{{< / highlight >}}

This could be rewritten in the following way now:

{{< highlight csharp >}}
class LocalFunctoins
{
    public void RegularMethod()
    {
       bool Even(int number) => number % 2 == 0;

       foreach(var number in Enumerable.Range(0,10).Where(Even))
       {
          Console.WriteLine(number);
       }
     }
}
{{< / highlight >}}

As you can see local functions support expression bodies, they also can be `async`.

Local functions can capture variables as lambdas do.
{{< highlight csharp >}}
public void Foo(int z)
{
    void Init()
    {
        Boo = z;
    }
    Init();
}
{{< / highlight >}}

Also might be handy for iterators:

{{< highlight csharp >}}
 int[] GetFoos()
 {
     IEnumerable<int> result() // iterator local function
     {
         yield return 1;
         yield return 2;
     }
     return result().ToArray();
 }
{{< / highlight >}}

## Ref returns and locals

Sort of a low-level feature in my opinion. You can return reference from method.
Erric Lippert [thought that](https://blogs.msdn.microsoft.com/ericlippert/2011/06/23/ref-returns-and-ref-locals/) 

>we believe that the feature does not have broad enough appeal or compelling usage cases to make it into a real supported mainstream language feature. 

Not anymore, he-he.

{{< highlight csharp >}}
 static void Main()
 {
     var arr = new[] { 1, 2, 3, 4 };
     ref int Get(int[] array, int index)=> ref array[index]; 
     ref int item = ref Get(arr, 1);
     WriteLine(item);
     item = 10;
     WriteLine(arr[1]);
     ReadLine();
 }
{{< / highlight >}}

Will print:

```
2
10
```

## Pattern matching

>Patterns are used in the `is` operator and in a `switch`-statement to express the shape of data against which incoming data is to be compared. Patterns may be recursive so that subparts of the data may be matched against subpatterns.

This is huge. C# community has been waiting for it for a long time.
Unfortunately the syntax is not final now.

There are several types of patterns supported for now.

### Type pattern

The type pattern is useful for performing runtime type tests of reference types.

{{< highlight csharp >}}
public void Foo(object item)
{
    if (item is string s)
    {
        WriteLine(s.Length);
    }
}
{{< / highlight >}}

### Constant Pattern

A constant pattern tests the runtime value of an expression against a constant value.

{{< highlight csharp >}}
public void Foo(object item)
{
    switch (item)
    {
        case 10:
            WriteLine("It's ten");
            break;
        default:
            WriteLine("It's something else");
            break;
    }
}
{{< / highlight >}}

### Var Pattern

A match to a var pattern always succeeds. At runtime the value of expression bounds to a newly introduced local variable.

{{< highlight csharp >}}
 public void Foo(object item)
 {
     if(item is var x)
     {
         WriteLine(item == x); // prints true
     }
 }
{{< / highlight >}}

### Wildcard Pattern

Every expression matches the wildcard pattern.

{{< highlight csharp >}}
 public void Foo(object item)
 {
     if(item is *)
     {
         WriteLine("Hi there"); //will be executed
     }
 }
{{< / highlight >}}

### Recursive Pattern

{{< highlight csharp >}}
public int Sum(LinkedListNode<int> root)
{
    switch (root)
    {
        case null: return 0;
        case LinkedListNode<int> { Value is var head, Next is var tail }:
            return head + Sum(tail);
        case *: return 0;
    }
}
{{< / highlight >}}   
### Others

`switch` based patterns could contain so-called guard close:

{{< highlight csharp >}}
public void Foo(object item)
{
    switch(item)
    {
        case int i when i > 10:
            WriteLine("That's a good amount");
            break;
        case int i:
            WriteLine("That's fine");
            break;
        default:
            WriteLine("whatever");
            break;
    }      
}
{{< / highlight >}}

Patterns could be joined:

{{< highlight csharp >}}
public void Foo(object item)
{
    if(item is string i && i.Length is int l)
    {
        WriteLine(l > 10);
    } 
}
{{< / highlight >}}
 
    
### Conclusion

Pattern matching is really neat. I spent some time with it and I like it.

{{< gist asizikov cbee9628d38db448622cd98ff1fee1a6 >}}

![console installer output](/images/2016/04-csharp-7-preview/console-installer.png)

## Links
 
* [GitHub Discussion: Local Functions](https://github.com/dotnet/roslyn/issues/2930)
* [GitHub Proposal: Ref Returns and Locals](https://github.com/dotnet/roslyn/issues/118)
* [GitHub Proposal: Binary literals](https://github.com/dotnet/roslyn/issues/215)
* [GitHub Roslyn Docs, Pattern Matching for C#](https://github.com/dotnet/roslyn/blob/024ad0a207c9f38493ff9163a377b9aacae371c7/docs/features/patterns.md)