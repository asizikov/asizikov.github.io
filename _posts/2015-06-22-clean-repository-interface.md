---
layout: post
title: Keeping the repository interface clean
---

The repository pattern is beeing blamed quite often. The most popular reason to blame it is an uncontrolled growth of the interface.

In the simple scenario we have an interface like this one:

{% highlight csharp %}
 public interface IClientRepository
 {
 	Client Get(int id);
 	IEnumerable<Client> GetAll();
 }
{% endhighlight %}

However models are never that simple and every client might have orders, addresses, contact details, and other nested properties.

We don't want to load our database with unnecessary joins and [Entity Framework has a nice tool for that](https://msdn.microsoft.com/en-us/data/jj574232.aspx).

When we need to retrieve only client's orders we can include it into the query:

{% highlight csharp %}
context.Clients.Where(client => client.Id == id)
	 .Include(client => client.Orders);
{% endhighlight %}

When we need just addresses:

{% highlight csharp %}
context.Clients.Where(client => client.Id == id)
	 .Include(client => client.Addresses);
{% endhighlight %}

And the combination:

{% highlight csharp %}
context.Clients.Where(client => client.Id == id)
	 .Include(client => client.Orders)
	 .Include(client => client.Addresses);
{% endhighlight %}

However the IClientRepository grows exponentially and quickly becomes an ugly monster:

{% highlight csharp %}
 public interface IClientRepository
 {
 	Client Get(int id);
 	Client GetWithOrders(int id);
 	Client GetWithAddresses(int id);
 	Client GetWithOrdersAndAddresses(int id);
 	IEnumerable<Client> GetAll();
 }
{% endhighlight %}

It's not a nice solution for sure: lot's of code duplication and very unclear interface.

A typical way to avoid it is to use [CQRS](http://martinfowler.com/bliki/CQRS.html) pattern and encapsulate the logic into the Query object.

At my current project we're building microservices, and a CQRS looks like an overkill. But it doesn't stop us from borrowing some ideas. 
Let's remove all these GetXXXX methods and add one more parameter to the Get method.

{% highlight csharp %}
 public interface IClientRepository
 {
 	Client Get(int id, FetchPaths<Client> fetchPaths = null);
 	IEnumerable<Client> GetAll(FetchPaths<Client> fetchPaths = null);
 }
 {% endhighlight %}

The FetchPath is a simple generinc class which holds the collection of expressions that we're going to use to build our chain of Include statements.

 {% highlight csharp %}
public sealed class FetchPaths<T>
{
  public IEnumerable<Expression<Func<T, object>>> Paths 
  									{ 
  								     get;
  								     private set; 
  								    }

  public FetchPaths(Expression<Func<T, object>> path) 
  								: this(new[] { path })
  {
  }

  public FetchPaths([NotNull] IEnumerable<Expression<Func<T, object>>> paths)
  {
    if (paths == null)
    {
      throw new ArgumentNullException("paths");
    }

    Paths = paths;
  }

  public FetchPaths<T> Concat([NotNull] FetchPaths<T> other)
  {
    if (other == null)
    {
      throw new ArgumentNullException("other");
    }

    return new FetchPaths<T>(Paths.Concat(other.Paths));
   }
 }
 {% endhighlight %}

Now we're ready to declare the following static class:
{% highlight csharp %}
public static class ClientFetchPaths
{
    static ClientFetchPaths()
    {
        Orders = new FetchPaths<Client>(client => client.Orders);
        Adresses = new FetchPaths<Client>(client => client.Adresses);
        OrdersAndAddresses = Adresses.Concat(Orders);
    }

    public static FetchPaths<Client> Orders { get; private set; }
    public static FetchPaths<Client> Adresses { get; private set; }
    public static FetchPaths<Client> OrdersAndAddresses { get; private set; }
}
{% endhighlight %}

So that we can get the clients including all the nesessary child objects.

{% highlight csharp %}
   var clients = ClientRepository.Get(id, ClientFetchPaths.OrdersAndAddresses);
{% endhighlight %}

A want to mention that so far we don't have any EntityFramework dependency here. Than means that we can keep this code (interface, models and FetchPathes) in the same assembly and reference it from the unit test project.

Now the repository implementation will be clean as well:

{% highlight csharp %}
context.Clients.Where(client => client.Id == id)
     .Fetch(fetchPaths));
{% endhighlight %}

Where Fetch is an extension method for IQueryable.
{% highlight csharp %}
public static class FetchExtensions
{
  public static IQueryable<T> Fetch<T>(
  				this IQueryable<T> queryable, 
  				[CanBeNull] FetchPaths<T> paths) where T : class
  {
    if (queryable == null)
    {
      throw new ArgumentNullException("queryable");
    }

    if (paths == null)
    {
      return queryable;
    }

    return paths.Paths.Aggregate(queryable, 
    	(current, expression) => current.Include(expression));
  }
}
{% endhighlight %}

