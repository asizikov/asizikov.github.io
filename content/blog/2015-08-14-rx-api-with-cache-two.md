---
date: "2015-08-14T00:00:00Z"
tags: 
    - programming
title: Reactive Extensions. API client with Cache-Aside & Refresh-Ahead strategy. Part 2.
slug: rx-api-with-cache-two
---

Hi,

In the [previous post](/2015/07/26/rx-api-with-cache-one/) I told you how to implement the Reactive API client with caching. Well, to prove that our code works we might want to have some unit tests. 

To do so let's create a new project (Class Library), add a reference to the main project, and install a few NuGet packages.

[Rx-Main](https://www.nuget.org/packages/Rx-Main/), [Rx-Testing](https://www.nuget.org/packages/Rx-Testing/), [xUnit.Net](https://www.nuget.org/packages/xunit/) and [Moq](https://www.nuget.org/packages/moq/).

```
Install-Package Rx-Main
Install-Package Rx-Testing
Install-Package xunit
Install-Package Moq
```
Then we’ll create a class  RxGitHubClientTest, inherited from [ReactiveTest](https://msdn.microsoft.com/en-us/library/microsoft.reactive.testing.reactivetest(v=vs.103).aspx).
```ReactiveTest``` is a base class which is shipped with ```Rx-Testing```. It implements some infrastructure for our tests. 
I’m not going to post all the tests here, you can find them on GitHub (the link is at the end of the post).

## Setup

As an example I’m going to test this scenario: If the cache is empty GitHubClient should download them, persist in a cache, and call ```OnNext``` and ```OnCompleted```.

To do that we’ll need to mock ```IHttpClient``` and ```IRatingCache```. Another class that we’ll use is [TestScheduler](https://msdn.microsoft.com/en-us/library/microsoft.reactive.testing.testscheduler(v=vs.103).aspx) from ```Rx-Test``` package.

TestScheduler implements [IScheduler](https://msdn.microsoft.com/en-us/library/system.reactive.concurrency.ischeduler(v=vs.103).aspx) interface and can be substituted instead of a platform-dependent instance of scheduler. It allows us literally to control the time and execute asynchronous code step by step. If you’d like to know more details I recommend to read a good post [“Testing Rx Queries using Virtual Time Scheduling”](http://blogs.msdn.com/b/rxteam/archive/2012/06/14/testing-rx-queries-using-virtual-time-scheduling.aspx).

Now our test setup looks like this:

{{< highlight csharp >}}
public RxGitHubClientTest()
{
    Model = new RatingModel
    {
        Rating = 10, 
        LastModified = new DateTime(2015, 07, 10, 1, 1, 1, 0), 
        Id = UserName
    };
    Cache = new Mock<IRatingCache>();
    Scheduler = new TestScheduler();
    HttpClient = new Mock<IHttpClient>();
}
{{< / highlight >}}
So now we’re ready to start with the test.

## Arrange
First of all we need to set up the mocks' behavior: the cache is empty, http client can download and parse the request.
{{< highlight csharp >}}
Cache.Setup(c => c.HasCached(It.IsAny<string>()))
     .Returns(false);
Cache.Setup(cache => cache.GetCachedItem(UserName))
     .Returns((RatingModel) null);
HttpClient.Setup(http => http.Get(UserName))
          .ReturnsAsync(new RatingResponse
		  {
		      Data = Model,
		      IsSuccessful = true
		  });
var client = CreateClient();
{{< / highlight >}}
In this case we expect to get the sequence of ```OnNext``` and ```OnCompleted``` events.

{{< highlight csharp >}}
var expected = Scheduler
	.CreateHotObservable(OnNext(2, Model), OnCompleted<RatingModel>(2));
{{< / highlight >}}
I guess this part needs some clarification.  ```OnNext(2, Model)``` is a method which is declared in  ```ReactiveTest```.
It has the following signature:
{{< highlight csharp >}}
static Recorded<Notification<T>> OnNext<T>(long ticks, T value)
{{< / highlight >}}
Basically, it just records the fact that ```OnNext``` method was called with the Model parameter. The Magic Number "2" is nothing more than the time in ticks. It’s not real time, it’s the TestScheduler’s virtual time. It's not the most elegant solution though. The TestScheduler is created at tick number zero, on the tick number one we subscribe to events and on the tick two we’ll get the first event. 
## Act
{{< highlight csharp >}}
var results = Scheduler
.Start(() => client.GetRatingForUser(UserName), 0, 1, 10);
{{< / highlight >}}
Here we start our ```TestScheduler```, which will be initialized at tick zero, then it will subscribe to ```client.GetRatingForUser(UserName)``` on the first tick. The last parameter is the virtual time on which the subscriber should be disposed. We can ignore this one for now.

And finally the last step.
## Assert
{{< highlight csharp >}}
ReactiveAssert.AreElementsEqual(expected.Messages, results.Messages);
Cache.Verify(cache => cache.Put(Model), Times.Once);
{{< / highlight >}}
That's how we can be sure that the expected sequence is equal to the actual one. And we verify that new model was cached.

This test could have been split into two parts, to follow the best practices (to have one assert per test method), but I wanted to show that all the regular approaches are valid in Rx-world.

Ok, now we have a green bar.

![tests passed](/images/rx-api-with-cache-two/tests-passed.png)

## Source code

You can find the source code on [GitHub](https://github.com/asizikov/rx-github-client-example)

## Conclusion

I mentioned Windows Phone development in my article, however the code is built for .NET 4.5. I did that on purpose, to let everybody check out and build the solution even if one has no WP SDK installed locally. It’s easy to convert the project to PCL or WP8/8.1 assembly. All of them are supported by Rx.

## Links and sources
* [Testing Rx](http://www.introtorx.com/content/v1.0.10621.0/16_TestingRx.html)
* [Testing Rx Queries using Virtual Time Scheduling](http://blogs.msdn.com/b/rxteam/archive/2012/06/14/testing-rx-queries-using-virtual-time-scheduling.aspx)
* [Testing and Debugging Observable Sequences](https://msdn.microsoft.com/en-us/library/hh242967(v=vs.103).aspx)

