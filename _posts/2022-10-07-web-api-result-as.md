---
title: Defining Web API result
date: 2022-10-07 06:00:00 +/-0000
categories: [Web API]
tags: [webapi,netcore,mvcoptions]     # TAG names should always be lowercase
---

In this blog I'm going to showcase how I handled a case when a customer required to have an endpoint which accepts XML object via POST and returns different XML object back.
 
There are multiple ways to go about this, but the main ones would be to declare what we expect and what we will return in the API controller itself by attributes. If given API only serves XML type responses we could do this in service builder as well.
 
However, the first step in this case would be to tell Web API services that we will produce XML in the first place by adding Output formatter in Web API program.cs.


```csharp
var builder = WebApplication.CreateBuilder(args);

ConfigureServices();

void ConfigureServices()
{
    builder.Services.AddMvc(options =>
    {
        options.OutputFormatters.Add(new XmlSerializerOutputFormatter(
            new XmlWriterSettings { OmitXmlDeclaration = false }));
    }).AddXmlSerializerFormatters();
}
```

After this, we can declare both, or either `[Consumes()]` and `[Produces()]` attributes in the controller. `Consumes` in this case would tell the Controller that we expect XML in the body. `Produces ` however tells that result should be specified type, in this case XML.

```csharp
[HttpPost]
[Produces("application/xml")]
[Consumes("application/xml")]
public IActionResult ExampleMethod()
{
    return Ok("fancy XML");
}
```

To access consumed XML, in ASP.NET Core web API we can use parameter binding, like `[FromBody]`, which looks like this.

```csharp
[HttpPost]
[Produces("application/xml")]
[Consumes("application/xml")]
public IActionResult ExampleMethod([FromBody] XmlDocument request)
{
    var result = _someInterface.DoSomething(request);
    return Ok(result);
}
```

As I mentioned in the beginning of this blogpost, we can also set `Produces` and `Consumes` in the service builder, in the same code block that declares output formatting.

```csharp
builder.Services.AddMvc(options =>
{
    options.Filters.Add(new ProducesAttribute("application/xml"));
    options.Filters.Add(new ConsumesAttribute("application/xml"));
    options.OutputFormatters.Add(new XmlSerializerOutputFormatter(new XmlWriterSettings{OmitXmlDeclaration = false}));
}).AddXmlSerializerFormatters();
```

Then in the controller we don't have to specify `Produces` and `Consumes`attributes.
 
Thanks for reading this quick code snippet showcase. Happy coding!
