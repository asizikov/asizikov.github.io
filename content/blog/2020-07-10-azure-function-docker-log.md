---
date: "2020-07-10T00:00:00Z"
image: /images/2020/07-func-logs/initial-state.png
title: Fix console logs for Azure Functions running in a Docker container
tags: 
 - azure
 - docker
 - serverless
slug: azure-function-docker-log  
---

Local development of C# Azure Functions on macOS is still a bit painful. 
Even the simple-ish logging might cause issues. Let's assume that we have Azure Functions Core Tools installed and we have a basic function app with one `TimerTrigger` function created.

![Azure Functions app in Rider](/images/2020/07-func-logs/initial-state.png)

With the default Run/Debug configuration

![Run configuration](/images/2020/07-func-logs/default-config.png)

We can compile and run our function

![Azure Functions Log](/images/2020/07-func-logs/log-console.png)

Our function will start and log to console as expected.

I wish all the functions were that simple, right? Unfortunately, sometimes we need more, sometimes we even have to use some shared libraries. So let's create one: 

{{< gist asizikov 1f09de4906a61d768c2ef85b980b1a0c >}}

Nothing fancy there, one class, one interface accepting the `ILogger<T>` as a dependency.

Let's update function code accordingly: will make is non-static, and use construction injection to get the `IAsyncGreeter` injected:

{{< gist asizikov 3528207d6fba89f70257278c68fe6ade >}}

In order to register this external dependency and use a proper .NET Core-style DI we need to install `Microsoft.Azure.Functions.Extensions` NuGet package and create a `Startup` class like this one: 

{{< gist asizikov 4ca721ce2e96ba1663f2c63fcbc50e0a >}}

You would expect it to work, right?

I wish...

![Just the main app is logging](/images/2020/07-func-logs/func-logs.png)

As you can see it still just logs from the function.

Why would one logger print it's output to console while another one just flushes our logs to void?

Because they are two different loggers with different settings. One logger is instantiated and configured in our `Startup.cs` class and another one is built and configured by function host and injected into the method instead. Oh, well.

I want to run this function in docker, so let's create a `Dockerfile`.

{{< gist asizikov 7bbd14242e0a7ea198979746aa24d129 >}}

Note that I'm setting `AzureFunctionsJobHost__Logging__Console__IsEnabled` environment variable here.

![Just the main app is logging](/images/2020/07-func-logs/docker-log.png)

The missing link here is the `hosts.json` logging configuration.

```json
    "logLevel": {
        "default": "Information"
    }
```
{{< gist asizikov 8ec81eac670f989ab10fc4fbcaa977d9 >}}

And voilà, we've got logs in our container: 

![Logs in a docker container](/images/2020/07-func-logs/docker-fixed.png)

I hope that would save you some time. Happy logging!

