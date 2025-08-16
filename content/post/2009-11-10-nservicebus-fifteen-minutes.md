---
categories:
- Development
date: "2009-11-10T00:00:00Z"
read_time: true
tags:
- NServiceBus
title: NServiceBus - Fifteen Minutes...
---

After downloading the assemblies I was ready to create my first sample application using NServiceBus. 
From what I can tell, a lot of time and effort has gone into building the **NServiceBus.Host** executable so if you plan on using it… you’re in luck, 
there are plenty of examples out there on how to use it. If you don’t want to use it, however, get ready for some digging 
(take a guess on which direction I went in). Hopefully this post will serve as a good example of how to handle things when you don’t want to use **NServiceBus.Host**. 
Time will tell…

The most basic example is sending a message. Note that this is not publishing a message but simply sending a message. 
To send a message you will need something that sends the message and something that will receive the message. 
Go ahead and create two console projects: 1) “Sender” and 2) “Receiver”.

You will need to be able to define a message that both the sender and receiver will be able to understand so go ahead and create library project 
(name it “Messages” for now to keep things simple).

Again, doing the most simple thing possible, add a reference to “NServiceBus.dll” to the “Messages” project. 
Create a new class, add the “Serializable”  attribute, and have the class implement the “IMessage” interface. 
This interface is nothing more than a indication to NServiceBus that it can be used as a message. 
And since we are going to be passing this class around as a message we need to be able to serialize it across the wire so… don’t forget the “Serializable” attribute.

```csharp
[Serializable]
public class ExampleMessage : IMessage {
  public int Id { get; set; }
  public string Message { get; set; }
}
```

A “message” can be thought of as a Data Transfer Object (DTO) in that the message should only contain data and really shouldn’t contain any methods or logic. 
There can be some interaction with the data like formatting or structuring but avoid putting any logic. Remember that this is going to be serialized.

From here add the “Messages” project as a reference to the “Sender” and “Receiver” console applications. 
While we are at it, go ahead and add a reference to “NServiceBus.dll” and “NServiceBus.Core.dll”.

Up to now we have three projects: Messages (contains our message class), Sender (the application that will send the message) and, 
Receiver (the application that will receive the message). So far so good.

### The Sender

So now we want the “Sender” application to send a message out into the world. 
Prior to being able to send a message we first have to instantiate a service bus (for NServiceBus this is an IBus). 
NServiceBus has a “fluent interface” class named “Configure” that uses an Inversion of Control (IoC) container to build up, configure, 
and create an instance of a service bus. By default this is done through Spring.Net which is all nice and good but I prefer to use 
StructureMap so… that is what I choose to use here. The only difference in this example is the “StructureMapBuilder” call which tells 
NServiceBus to use StructureMap as the IoC container. If you leave this line out… you will use Spring.Net.

```csharp
var bus = Configure.With()
  .StructureMapBuilder()
  .XmlSerializer()
  .MsmqTransport()
  .IsTransactional(false)
  .PurgeOnStartup(false)
  .UnicastBus()
  .ImpersonateSender(false)
  .CreateBus()
  .Start();
```

Now we have a service bus that we can send messages with. 
In order to send a message all we need to do is new up an instance of our class from the “Messages” project and call the **Send** method from our service bus instance. 
For this simple example we do this in a while loop where each time we hit “Enter” we send a message:

```csharp
while (Console.ReadLine() != null) {
  var message = new ExampleMessage(1, "My First Example");
  bus.Send(message);
}
```

Finally it is time to configure our “Sender” application. Here we need to add an Application Configuration File (App.config). 
Since we are sending a message we need to configure our MSMQ Transport and our Unicast Bus.

_NOTE: Unlike other configuration sections you cannot “name” the section something different. 
The following section “names” must be as they appear or they do not work. Not 100% why this is but… if you don’t then it will not work_

```xml
<configSections>
  <section name="MsmqTransportConfig" type="NServiceBus.Config.MsmqTransportConfig, NServiceBus.Core" />
  <section name="UnicastBusConfig" type="NServiceBus.Config.UnicastBusConfig, NServiceBus.Core" />
</configSections>
<MsmqTransportConfig InputQueue="MySenderQueue" ErrorQueue="error" NumberOfWorkerThreads="1" MaxRetries="5" />
<UnicastBusConfig DistributorControlAddress="" DistributorDataAddress="" ForwardReceivedMessagesTo="">
  <MessageEndpointMappings>
    <add Messages="MessageTypes" Endpoint="MyReceiverQueue" /> 
  </MessageEndpointMappings> 
</UnicastBusConfig>
```

Right now the important pieces to cover in the configuration are: InputQueue, Endpoint, and Messages.

`InputQueue` defines where this service bus can receive messages. Since the “Sender” is never going to receive a message in this example we can safely put anything we want here. For consistency I put “MySenderQueue”.

`Endpoint` defines where this service bus will send messages. This is a really important setting. We will be using this value later when we configure the “Receiver”.

`Messages` defines what type of messages this service bus will send to the `Endpoint`. Here we can break this up to be type specific but for this example we will simply instruct the service bus to send all of the messages from our “Messages” project to the `Endpoint`.

**The Receiver**

Now we want the “Receiver” to receive messages from the world. Prior to being able to receive messages we first have to instantiate another service bus similar to the way we did in our “Sender” project. The only real big difference in configuring this service bus is telling NServiceBus to load in our message handlers.

```csharp
var bus = Configure.With()
  .StructureMapBuilder()
  .XmlSerializer()
  .MsmqTransport()
  .IsTransactional(false)
  .PurgeOnStartup(false)
  .UnicastBus()
  .ImpersonateSender(false)
  .LoadMessageHandlers()
  .CreateBus()
  .Start();
```

Since we don’t want our “Receiver” application closing as soon as we bring it up we need to add a little bit of code to make sure it sticks around while we test:

```csharp
Console.ReadLine();
```

If you notice the only difference between the “Sender” and the “Receiver” is the **LoadMessageHandlers** call. 
So in order for NServiceBus to load our message handlers… we need to have a message handler. 
To do this simply add a new class and implement the `IHandleMessages<>` interface:

```csharp
public class ExampleMessageHandler : IHandleMessages<examplemessage> {
  public void Handle(ExampleMessage message) {
    Console.WriteLine("I got an example message... now what?");
  }
}
```

Finally it is time to configure our “Receiver” application. Here we need to add an Application Configuration File (App.config). 
Since we are only receiving messages we only need to configure our MSMQ Transport (no need to configure the Unicast Bus).

```xml
<configSections>
  <section name="MsmqTransportConfig" type="NServiceBus.Config.MsmqTransportConfig, NServiceBus.Core" />
</configSections> 
<MsmqTransportConfig InputQueue="MyReceiverQueue" ErrorQueue="error" NumberOfWorkerThreads="1" MaxRetries="5" /> 
```

In order to receive messages properly we need to make sure that the `InputQueue` value is the same as the `Endpoint` value from the “Sender” configuration. 
For this example it is “MyReceiverQueue”.

### Done

Now you are ready to go and start sending messages from the “Sender” to the “Receiver”.

To do so simply right click the “Sender” project and click “Start new instance” from the “Debug” submenu. 
After this console app is up and running right click the “Receiver” project and click “Start new instance” from the “Debug” submenu. 
With both console applications up and running hit “Enter” from the “Sender” window and watch the message appear in the “Receiver” window.

### Summary

All and all I would say that this took about fifteen to twenty minutes to get setup and running. 
That is pretty impressive especially when you start digging into what is actually happening in the background.

In future posts we can go over Publish and Subscribe, Request and Reply, and talk about the Distributor (which is some really neat stuff).

