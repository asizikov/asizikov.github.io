---
date: "2017-12-23T00:00:00Z"
image: /images/2017/12-mocking/mockingbird.jpg
title: You are mocking it wrong.
tags: 
  - programming
slug: you-are-mocking-it-wrong    
---

Well, probably *you* are not, but let me grumble a little bit anyway.

![Mockingbird picture](/images/2017/12-mocking/mockingbird.jpg)
_Mockingbird knows how to mock._

I've been working with various code bases throughout of my career, and there is one pattern which I see rather often. As you may have already guessed it's the unit-tests and mocking I'm going to talk about here. To give it a nice catchy start, I'd claim here, that mocks should be used when you *have to*, but not when you *can*. 

I'll give a few examples of somewhat useless and even harmful tests.  
All the examples are going to be made up, but I hope you'd get the point. 

So, let's start with the typical `Greeter` example. This is a "Hello, world" of Unit Testing. In one way or another, this sample gets repeated in all the articles, posts and books dedicated to unit-tests and mocking frameworks.

```csharp
IGreeter{
   string Greet(string name, string title);
}

public class Greeter: IGreeter{
  public string Greet(string name, string title){
    return "Hello," + title + " " + name;
  }
}

public class Client{
  private IGreeter _greeter;
  public Client(IGreeter greeter){
    _greeter = greeter;
  }
  
  public string FormatPageHeader(string name, string title){
    return "<h>" +  _greeter(name, title) + "</h>";
  }  
}

 public class ClientTests {
    pubic void Test(){
        var mock = new Mock<IGreeter>();
        mock.Setup(greeter => greeter.Greet("John", "Mr.")).Returns("Hello, Mr. John");
        var result = new Client(mock.Object).FormatPageHeader("John", "Mr.");
        Assert.AreEquals(result, "<h>Hello, Mr. John</h>"); //All good here, what does it test, tho?
    } 
 }
```

So far so good. Tests are green. The Greeter interface isn't perfect, though. Two `strings` as a parameter? So easy to mess up, isn't it? Most probably you have a method like that in your project. 

Ok then, imagine that you decided to make this method less error prone. You don't have much time for a proper refactoring because you have a feature to work on. Let's just swap the parameters. It's much more natural for a human to put the title before the name. Depends on the culture, I know. 

("John", "Mr.") is more awkward compared to ("Mr.", "John") and ("Dr.", "Smith"). 

So, we'll end up with the greeter like that: 

```csharp
//Version 2
public class Greeter2 : IGreeter{
  public string Greet(string title, string name){
    return "Hello, " + title + " " + name;
  }
} 

IGreeter{
   string Greet(string title, string name);
}
```

`add.`, `commit`, `push`.  Our beloved build server will pick the changes up, will run the tests and will fail of course. We forgot to update the test for the `IGreeter` implementation. Once they fixed we're good, aren't we?

Not really, remember the `Client` class? Now it's incorrect, we still pass the title and the name in the old order, even though our test claims that everything is fine.

![This is fine](/images/2017/12-mocking/fine.jpg)

Here is the paradox, we introduced mocking so that we can test our class in isolation, but we had to put some code to make that mock return something. It's still the logic, isn't it? Now `IGreeter` has at least two implementations. Most probably in your codebase, you have tens of implementations for the mocked class. Just because it's a very much reused dependency, and you have to mock it all over again. 

We can improve that test, tune it, but the only way we can make it fail is to repeat the `Greeter` logic in our mock set up. But wait a minute, if we have the same implementation, why don't we reuse the existing code? 

The more complex your mocked object is the more complex your mock setup becomes. If that's not the case, most probably you're testing just the interaction with your dumb mock, i.e. you're not testing anything. Just like the `ClientTest` above. The only positive outcome is the extra 20c which your company pays to Amazon for that CPU time you wasted on the build server.

It's not rare to have more than one dependency. I can imagine that our `Client` could use some `IHmlRenderer` which would consume the `IGreeter` result. You would mock that one too, right? 

Sweet as. You now have a test which verifies how two mocks are integrated with each other. How does that prove your code correctness, thought? I don't know. 

I'm going to step back now, and talk about it from another point of view. What does the typical system look like? I assume, it's not the bunch of isolated classes, but it's more like a spiderweb of dependencies. If you look at type dependency diagram you would see a group of clusters, where each cluster is a somewhat tightly coupled set of classes. This is how we manage complexity, we break down one large class into small pieces, but those pieces would never be used in isolation. Each of them would be responsible for a little piece of work. You would write a separate test suite for them, mock out all the dependencies (they all belong to the same cluster). 

How is that different to the attempt to [test a private method](https://stackoverflow.com/questions/34571/how-do-i-test-a-private-function-or-a-class-that-has-private-methods-fields-or)? 

That little convenience wrapper for the standard library class is nothing more than an implementation detail. 

### But integration tests are slow...

That would be an expected note. If we come back to our `ClientTest`. What performs better, the test with mock, or the test with the concrete Greeter implementation? I guess the answer is obvious. We shouldn't forget that that test doesn't prove anything. It is slower, it allocates more, and it's wrong. I'd say it's harmful. 

### So you're going to send out those emails every time you test?

Ok, that is a good question. Remember I said that we should mock when we *have to*, not when we *can*? That's exactly the right situation for the test isolation.

You don't want to make HTTP calls while running your unit tests suite, you don't want to send out emails, neither do I. 

### Hey, I tried not to mock, and I got sick of initing all the dependencies

This is where it gets painful. I saw systems like that, it's a nightmare to maintain. Manually setting up the dependency of the dependency is really not the way to go. It's hard, and everyone would avoid writing a new test. But that's an easy task to solve. 

How do we build a dependency tree in the runtime? I hope you're using the DI framework which wires up interfaces and implementations together, it knows how to create a new dependency, it knows when and how to rebuild a dependency tree. If you're building a web service you should reset the state between requests, so keep as many dependencies request scoped as it's possible. 

The same pattern is applicable for your test suite. Everything is request scoped and stateless already, so treat each test as a 'request' and let the DI container to build the system under test for you. 

Of course, you would have to set up the DI container differently. 

Module (or cluster) level test would build the dependency tree for that module but would mock out the rest of the world. Like no HTTP calls, no DB, no emails, you've got the point.

At this point, you would realize that you don't need that test which verifies that your utility class can split the string, but you would be sure that your `MortgageCalculator` does the job. After all, that is your business rule, and that is the feature you're building. Unless you are the low-level framework developer of course. Most of us are not, though.

Once you've got all the clusters well tested, all the interfaces established you may try to break it down a little bit. Or you may want to leave it as is, it's up to you. 

Imagine you want to extract some logic into the separate class. Now your system has a new dependency, but that logic has already been tested. You don't need to set up a new mock, you don't need to repeat the logic of the extracted class in your test set up, you don't need to update the test to instantiate the tested class. It's just there, and your safety net still works. If you extract and modify the logic, you would break the test.  Do you see the beauty? Extracting the class and introducing a new dependency is as easy as moving the logic into a private method.

### Summary

The approach above would give you the following benefits: 

* Fewer meaningful, useless and harmful tests
* No need to maintain duplicated implementation (mock set ups)
* Feature driven tests, tests that verify that the cluster does the job
* An easy to refactor code base
* You can improve module level access (no need to make the class public just for the testing)

From my experience tests, based on that cluster approach, are much more reliable. When they break, they actually mean that the system is dysfunctional, when they pass you can be sure that you did not break the logic. 

We still have to run integration tests to be sure that the third party integrations are working, that the system's runtime configuration is valid. 

I am more than open to any criticism, feel free to tear that post apart. 
