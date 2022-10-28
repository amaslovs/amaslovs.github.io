---
title: Cross-App Application Insights
date: 2022-09-20 20:00:00 +/-0000
categories: [Architecture, Application Insights]
tags: [appinsights,telemetry,architecture]     # TAG names should always be lowercase
---

## Idea behind... ##

Once there was an requirement to implement custom payment handling system that gets triggered from Web UI. Whenever all data was collected from the website, something had to be called to process calling payment API endpoints, such as authorization, product loading, etc. Since at that time Azure was and currently is the go-to cloud provider for my project, choice of Azure infrastructure for such a task was made. But then question arose how could we log such data flow in detailed manner, so we could tie actions made in Web UI together with actions later done in for example Azure durable functions and view them in sequential order without heavy load. One neat way that came in mind is Azure Application Insights.
 
Idea behind this is that a Web app would track every meaningful user action to Application Insights as the user was performing it, for example - selecting product categories. In addition to every meaningful action performed by the app itself, such as data query or storing would and could be tracked. Whenever actions of different app, for example Azure Functions, are required to finish process, that given app also can track every action to same Application insights using same unique identifier as Web app.

![Telemetry workflow](/assets/img/posts/telemetry/telemetry.png "Telemetry workflow")

 Esentially every desired action could be tracked. Such tracking can be performed using custom event metrics.
 More info on custom event metrics: <https://learn.microsoft.com/en-us/azure/azure-monitor/app/api-custom-events-metrics>

## Code samples ##

For this we need the `Microsoft.ApplicationInsights` package.
 
In order for us to track same event sequence across multiple apps, we can use **Operation Id** which can be set up in Application Insights data model ( <https://learn.microsoft.com/en-us/azure/azure-monitor/app/data-model-context> ).
Operation ID can be set up by accessing `TelemetryClient` context.
 
For example such method can be created for ease of use across multiple apps:


```csharp
public class AppInsightsTelemetryService : IAppInsightsTelemetryService
{
    public void TrackEvent(string eventName, string operationId, Dictionary<string, string> properties = null)
    {
        var tc = new TelemetryClient();
        if(!string.IsNullOrEmpty(operationId))
            tc.Context.Operation.Id = operationId;

        tc.TrackEvent(string.IsNullOrEmpty(eventName) ? "CustomEvent" : eventName, properties);
    }
}
```

Then to call this method, we can specify `eventName` that would show up in Azure portal transaction search, as well as any properties we wish to pass along. **Operation Id** in this case has to be unique across the entire solution but same in each call you wish to track together with previous.

```csharp
_telemetry.TrackEvent("Add Product",
cart.Data.Id.ToString(),
new Dictionary<string, string>
    {
        {
            "userName", cart.Data.UserName
        },
        {
            "lineItems", JsonConvert.SerializeObject(cart.Data.LineItems)
        },
        {
            "executionTime", sw.Elapsed.ToString()
        }
    });
```

## Azure portal ##

In order to view Custom Events in Azure Portal, we would need to navigate to given Azure Application Insights instance and select **Transaction search**. There we can toggle desired Event types and select "Custom Events". Additional filtering can be applied on top of that, were we can search for our custom **operationId**. Results will display all telemetry data with that Id in order they were logged, no matter from which App.

![Azure portal transacton search](/assets/img/posts/telemetry/telemetry-portal.PNG "Azure portal transacton search")

Happy logging :)

