---
title: Using threading channels
date: 2023-02-10 06:00:00 +/-0000
categories: [Threading]
tags: [threading,netcore,channels,servicebus]     # TAG names should always be lowercase
---

## Idea behind... ##

In one of my tasks I had a case where I had to use the same Azure ServiceBus message from one subscription in multiple Web API endpoints. One of these endpoints serves data on publish/subscribe basis and has to execute publish operation as soon as something new becomes available.

## Solution ##

In order to utilize a single subscription in ServiceBus, a single reader was created for reading these messages that come on irregular basis. In order to serve incoming messages to the publish/subscribe endpoint and potentially any other listeners, some sort of distribution mechanism had to be put in place. For this task I chose ``Microsoft.Threading.Channels`` - a thread safe producer/consumer library. In short, ``System.Threading.Channels`` allows creation of ``Channel<T>`` property, which has ``ChannelWriter<TWrite> Writer`` and ``ChannelReader<TRead> Reader``. In other words, ``Writer`` is used to write messages to ``Channel`` and ``Reader`` is used to access them. ``Channel`` is an object that stores written messages and allows reading them in "queue" like fashion.

Implementation of ``System.Threading.Channels`` logic is quite straightforward. First, we have to define ``Channel`` itself, it can be Bound and Unbound. Bound channel has a specific capacity, which you have to set when initializing the channel. Unbound channels however do not have this capacity limitation. In my case I chose Bound channel, as my APIs require only the latest messages.

```csharp
public class ExampleDataModelChannel
{
    private readonly Channel<ExampleDto> _channel;

    public ExampleDataModelChannel()
    {
        _channel = Channel.CreateBounded<ExampleDto>(
            new BoundedChannelOptions(10)
            {
                AllowSynchronousContinuations = false,
                SingleReader = false,
                SingleWriter = false,
                FullMode = BoundedChannelFullMode.DropOldest
            });
    }

    public ChannelReader<ExampleDto> Reader => _channel.Reader;

    public bool Publish(ExampleDto data)
    {
        return _channel.Writer.TryWrite(data);
    }
}
```

Given code allows me to write to ``Channel<T>`` using ``Publish`` whenever I instantiate ExampleDataModelChannel class.

```charp
_exampleDataModelChannel.Publish(model);
```
and read utilizing ``Reader``, as in example below.


In this scenario, whenever a ServiceBus reader gets a new message it gets distributed to ``Channel<T>``. To read the message, ``ChannelReader<TRead> Reader`` has to be utilized. So in order to notify subscribers from pub/sub API, a message would have to be served to them as soon as it becomes available in ``Channel<T>``. Since logic that handles distributing messages to subscribed clients in my case runs as a separate BackgroundService, I'm able to read given ``Channel<T>`` and execute publish logic only when there is something to serve subscribed clients.

```csharp

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await foreach (var model in _dataChannel.Reader.ReadAllAsync(stoppingToken))
        {
            try
            {
                //do magic here
            }
            catch (Exception e)
            {
                _logger.LogError(e, "Something broke");
            }
        }
    }
```

## Summary ##

In my particular case, I was able to utilize ``Microsoft.Threading.Channels`` for distributing ServiceBus messages in producer/consumer fashion across applications, as soon as ServiceBus reader receives them. Therefore not only allowing various reading scenarios for a single ServiceBus subscription message receiver, but also separating responsibility and logic outside the ServiceBus reader.

More info on threading channels here: <https://learn.microsoft.com/en-us/dotnet/core/extensions/channels> 

Thank you for reading.
