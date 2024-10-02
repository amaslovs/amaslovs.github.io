---
title: Defining dependency scope within BackgroundService
date: 2024-10-02 06:00:00 +/-0000
categories: [Web API]
tags: [webapi,netcore,backgroundservice]     # TAG names should always be lowercase
---

## Issue ##

In this blog post I'll describe a situation I encountered in one of my projects, where I had to access an interface that was registered as a ``scoped`` service from ``BackgroundService``, which is ``singleton``. In essence, I'll describe a neat trick on how to access ``scoped`` service from ``singleton`` service.

In the ``Program.cs`` of this example web app we have our initialization of builder and method that adds example service in a pattern I really like, split by features, which also makes it more readable. In this case ``AddExampleFeature()`` can contain various service descriptors for a given feature of a web app.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddExampleFeature()
```

Then we have our given ``IServiceCollection`` method that defines lifecycle of example service

```csharp
public static class IServiceCollectionExtension
{
    public static IServiceCollection AddExampleFeature(this IServiceCollection services)
    {
        services.AddScoped<IRandomRepository, RandomRepository>();
        return services;
    }
}
```

If we would want to consume this given interface from ``BackgroundService`` which by design has a ``singleton`` lifecycle, we would run into an exception as ``scoped`` service cannot be consumed by ``singleton`` service. Application would thrown this exception:

`InvalidOperationException: Cannot consume scoped service 'Random.Path.ToFile.RandomRepository' from singleton 'Random.Path.ToBackgroundService.ExampleBackgroundService'.`

## Solution ##

To circumvent this we can pass `IServiceProvider` in the ``BackgroundService`` constructor and create a new ``IServiceScope`` within ``BackgroundService`` to resolve scoped service.

```csharp
public class ExampleBackgroundService(
    IServiceProvider provider)
    : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var scope = provider.CreateScope();
        {
            var randomRepository = scope.ServiceProvider.GetRequiredService<IRandomRepository>();
            //do magic with your random repository
        }
    }
}
```

By utilizing the ``IServiceProvider`` extension method we can now play around with ``transient`` or ``scoped`` services within ``BackgroundService``.

Thank you for reading

More on ``IServiceProvider`` here: <https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider?view=net-8.0>
More on ``IServiceScope`` here: <https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.createscope?view=net-8.0>
More on DI and Lifecycles here: <https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0>