---
title: Creating Web API background service
date: 2022-10-06 06:00:00 +/-0000
categories: [Web API]
tags: [webapi,netcore,backgroundservice]     # TAG names should always be lowercase
---

In ASP.NET Core we can implement background tasks, or Hosted Services, that aid desired logic of a particular application.
 
Previously I wrote about one potential use case of such background task [here]({{site.baseurl}}/posts/web-api-servicebus/).
 
Since .NET Core 2.0 we can use ``IHostedService`` interface to easily implement hosted services for our applications. In the context of the given example I'm using the ``BackgroundService`` base class that implements the ``IHostedService`` interface. ``BackgroundService`` is located in the ``Microsoft.Extensions.Hosting`` package, which comes pre-installed when creating a new .NET Core Web API project.
 
To create a background service in Web API from scratch, the first step is to create the desired background service class and inherit from the ``BackgroundService`` base class and implement the necessary ``ExecuteAsync`` method.

```csharp
public class ExampleService : BackgroundService
{
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        throw new NotImplementedException();
    }
}
```

Here within ``ExecuteAsync`` we can write all required logic for the new background service.
 
In order for it to register to ``Host`` as a background task, we have to register it through ``AddHostedService`` method in Startup class.

```csharp
var builder = WebApplication.CreateBuilder(args);
void ConfigureServices()
{
    builder.Services.AddHostedService<ExampleService>();
}
```

More detailed info about IHostedService here: <https://learn.microsoft.com/en-us/dotnet/architecture/microservices/multi-container-microservice-net-applications/background-tasks-with-ihostedservice>