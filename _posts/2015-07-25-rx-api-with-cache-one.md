---
layout: post
title: Reactive Extensions. Api client with Cache-Aside & Refresh-Ahead strategy. Part 1.
tags: reactive-extensions howto rx cache
---

Hi,

Today I want to talk about the development of the api client library. Let’s say it is an imaginary GitHub RESTfull Api that returns user’s rating. To make this routine more interesting we’ll add caching and mix it with Reactive Extensions. In fact the article is a summary of my Windows Phone development experience, and  the approach in question was taken in a several applications with different modifications.

<font color="gray">Note: In this article I assume that the reader is familiar with the concept of [FRP](https://en.wikipedia.org/wiki/Functional_reactive_programming) and [Reactive Extensions](https://msdn.microsoft.com/en-us/data/gg577609.aspx).</font>

Let’s say that we have an url like that

nonexisting-api.github.com/v1/raiting/user_name

which returns the following json response:
{% highlight json %}
{
     "id" : "requested user name",
     "rating" : 123,
     "lastModified" : "2015-07-20"
}
{% endhighlight %}
Calling the api, deserializing the response and displaying the result to the user is a trivial task.  
{% highlight csharp %}
public async Task<RatingModel> GetRatingForUser(string userName)
{
    var ratingResponse = await HttpClient.Get(userName);
    if (!ratingaResponse.IsSuccessful)
    {
        throw ratingResponse.Exception;
    }
    return ratingResponse.Data;
}
{% endhighlight %}
<center><font color="gray">the naive implementation</font></center>
##Let’s add caching
Assuming that we’re building the mobile app, and taking into account that our data is not business critical we can consider following requirements:

1. The app has to start quickly;
2. It has to render the cached data first;
3. It should try getting the fresh data;
4. If succeeded it should put fresh data to its cache;
5. Render the new data or report the error;

Let me illustrate it with a sequence diagram. I hope I still remember how to draw them :)
![sequence diagram](/images/rx-api-with-cache-one/diagram.png)
There are different local cache strategies. The one I’m going to follow is a 
[Cache-Aside](https://msdn.microsoft.com/en-us/library/dn589799.aspx) one.

The basic idea is to treat the cache as a passive data storage so that the responsibility to update the cached data is delegated to the cache client. 

The points 2,3 and 4 are somewhat the realization of the [Refresh-Ahead](http://www.google.com/url?q=http%3A%2F%2Fdocs.oracle.com%2Fcd%2FE15357_01%2Fcoh.360%2Fe15723%2Fcache_rtwtwbra.htm&sa=D&sntz=1&usg=AFQjCNEinVNWh8WsT-SxfW1ZIlgv40SlEA) caching pattern. Not the classical version though, but that’s what we need patterns for.

##Implementaton

Let’s create a new Class Library (.NET 4.5) project in visual studio and add the Rx-Main NuGet package to it.

{% highlight bash %}
Install-Package Rx-Main
{% endhighlight %}

We’ll need the interface for the cache.
{% highlight csharp %}
public interface ICache<TKey, TValue> where TValue : IEntity<TKey>
{
    bool HasCached(TKey key);
    TValue GetCachedItem(TKey key);
    void Put(TValue updatedRating);
}

public interface IRatingCache : ICache<string, RatingModel>
{
}
{% endhighlight %}

The http-client abstraction:
{% highlight csharp %}
public interface IHttpClient
{
    Task<RatingResponse> Get(string userName);
}
{% endhighlight %}

And the models
{% highlight csharp %}
public class RatingResponse
{
    public bool IsSuccessful { get; set; }
    public RatingModel Data { get; set; }
    public Exception Exception { get; set; }
}

public class RatingModel 
{
    public string UserName { get; set; }
    public int Rating{ get; set; }
    public DateTime LastModified { get; set; }
}
{% endhighlight %}

Let’s keep the actual http-request, parsing, deserialization and error handling outside the scope of this article

The api client interface will look like that

{% highlight csharp %}
public interface IRatingClient
{
    IObservable<RatingModel> GetRatingForUser(string userName);
}
{% endhighlight %}

The key part here is the [IObservable&lt;T&gt;](http://www.google.com/url?q=http%3A%2F%2Fmsdn.microsoft.com%2Fen-us%2Flibrary%2Fdd990377(v%3Dvs.110).aspx&sa=D&sntz=1&usg=AFQjCNEndHmJ-ZVw1iN6PqzCDcW8PUcNAQ) i.e. the stream of events which you can subscribe to.

The Rx power is in the ability to build the [pipeline](http://martinfowler.com/articles/collection-pipeline/) using the basic blocks, which are quite simple. We can wrap the cache usage into the reusable component/rx-operator:
{% highlight csharp %}
public static IObservable<T> WithCache<T>(
        this IObservable<T> source, 
        Func<T> get, 
        Action<T> put) where T : class
{
    return Observable.Create<T>(observer =>
    {
        var cached = get();
        if (cached != null)
        {
            observer.OnNext(cached);
        }
        source.Subscribe(item =>
        {
            put(item);
            observer.OnNext(item);
        }, observer.OnError, observer.OnCompleted);
        return Disposable.Empty;
    });
}
{% endhighlight %}

Note that there is neither dependency on cache type, nor any other information about the cache. 

Generally it’s not a great idea to re-render the UI with no reason. To avoid an unnecessary invocation of OnNext delegate we’re going to use [DistinctUntilChanged](https://msdn.microsoft.com/en-us/library/system.reactive.linq.observable.distinctuntilchanged(v=vs.103).aspx) operator.

To do that we should define the custom comparer:
{% highlight csharp %}
public class RatingComparer : IEqualityComparer<RatingModel>
{
    public bool Equals(RatingModel x, RatingModel y)
    {
        return x.Rating == y.Rating 
        && x.LastModified == y.LastModified 
        && x.UserName == y.UserName;
    }
    public int GetHashCode(RatingModel obj)
    {
        return obj.GetHashCode();
    }
}
{% endhighlight %}
Now we have all the blocks required to get the desired behavior:

* Cache
* Rx caching operator
* Duplicates filter 

That means we are ready to chain them together and build the RxGitHubClient implementation:
{% highlight csharp %}
public sealed class RxGitHubClient : IRatingClient
{
    private IRatingCache Cache { get; set; }
    private IHttpClient HttpClient { get; set; }
    private IScheduler Scheduler { get; set; }

    public RxGitHubClient(IRatingCache cache, 
    	IHttpClient httpClient, 
    	IScheduler scheduler)
    {
        Cache = cache;
        HttpClient = httpClient;
        Scheduler = scheduler;
    }

    public IObservable<RatingModel> GetRatingForUser(string userName)
    {
        return GetRatingIntrn(userName)
            .WithCache(
            	() => Cache.GetCachedItem(userName), 
				model => Cache.Put(model))
            .DistinctUntilChanged(new RatingModelComparer());
       }
    private IObservable<RatingModel> GetRatingIntrn(string userName)
    {
        return Observable.Create<RatingModel>(observer =>
        Scheduler.Schedule(async () =>
        {
            var ratingResponse = 
				await HttpClient.Get(userName);
            if (!ratingResponse.IsSuccessful)
            {
                observer.OnError(ratingResponse.Exception);
            }
            else
            {
                observer.OnNext(ratingResponse.Data);
                observer.OnCompleted();
            }
        }));
    }
}
{% endhighlight %}

Now the usage of the client becomes as simple as the client from the first example.
{% highlight csharp %}
Client.GetRatingForUser(userName)
	  .Subscribe(RenderRating);
{% endhighlight %}
In the next blog post I’m going to write unit tests for this api client. Stay tuned.
##Links and sources

* [ReactveX home](http://reactivex.io/)
* [Introduction to Rx](http://www.introtorx.com/content/v1.0.10621.0/05_Filtering.html)
* [The Reactive Extensions (Rx)](https://msdn.microsoft.com/en-us/data/gg577609.aspx)
* [Read-Through, Write-Through, Write-Behind, and Refresh-Ahead Caching](http://docs.oracle.com/cd/E15357_01/coh.360/e15723/cache_rtwtwbra.htm#COHDG5177)
* [Cache-Aside Pattern](https://msdn.microsoft.com/en-us/library/dn589799.aspx)
* [www.websequencediagrams.com](https://www.websequencediagrams.com/)



