# Some of the new features in 4.0
You have probably been reading about most of them since you signed up for this beta, but a unordered list with the biggest changes are:

 - Support for RPC (as a complement or replacement for pub/sub)
 - Multiplexing over several controllers on one connection
 - Owin Support
 - Clients for iOS (MonoTouch), Android (MonoDroid) and NETMF that all use full-duplex/bi-directional communication
 - Synchron communication so that you can wait for the result from a controller method
 - IMessage replace ITextArgs and IBinaryArgs, much easier and better binary support
 - Binary support to any method in a controller (just like any object)
 - You can pass meta data with binary data to get information about files etc being sent
 - Hearbeat helpers for sending control-frames (pings/pongs)
 - AuthenticationPipline, a new plugin that you can use to set the IPrincipal of the ConnectionContext
 - Easier to create custom protocols for connecting InternetOfThings etc
 - Enterprise: scaling, loadbalancing

#XSockets.NET 4 - Introduction

    TODO

##What is it?
XSockets.NET is a real-time messaging system that allows communication between any device that has TCP/IP. The server can be hosted anywhere (.NET/Mono) and the clients cover every major browser + C#, VB.NET, Android, iOS, NETMF. And it is very easy to connect anything else that has a TCP/IP stack.

***In short terms: RealTime, InternetOfThings and WebRTC in a single framework!***

##Who uses it?

    TODO: Add images here

----------

##Getting started with real-time communication
###1. Start a server

    using (var container = Composable.GetExport<IXSocketServerContainer>())
    {
        container.Start();
        Console.ReadLine();
    }
    
###2. Create a client and listen for "MyMessage"

JavaScript

    var conn = new XSockets.WebSocket('ws://127.0.0.1',['generic']);
    conn.controller('generic').mymessage = function(data){
        alert(data.Text);
    };

C#

    var conn = new XSocketClient("ws://127.0.0.1:4502", "http://localhost","generic");
    conn.Open();
    conn.Controller("generic").On("mymessage", data => Console.WriteLine(data.Text));

### 3. Send message

JavaScript

    conn.controller('generic').invoke('mymessage',{Text:'Hello JS RealTime'});
    
C#

    conn.Controller("generic").Invoke("mymessage",new {Text = "Hello C# RealTime"});

### 4. What's next?
#### JavaScript/C# Client API
Learn to...

 - Use Pub/Sub, RPC or both!

#### Server
Create...

 - Powerful server-side controllers
 - Custom pipeline
 - Interceptors (for messages, connections and errors)
 - Protocol-plugins (for connecting other things, devices, languages)
 - Clients in .NET, NodeJS, C, Perl, Raw sockets or whatever you feel like connecting!

----------

## Installing XSockets.NET
XSockets.NET is distributed through nuget.org and chocolatey.org. From Chocolatey you install our Windows Service and on Nuget we have all our packages for development. You can read more about all the packages after the installation samples.

**NOTE: Use the suffix `-pre` to get XSockets 4.0 assemblies from nuget**

### Install into a Self-Hosted application
In the nuget Package Manager Console run the command below to install the server.

`PM> Install-Package XSockets.Server` **(include `-pre` to get v 4.0)**
####Start the server

Inside of the Main method start the server with the code below.

    using XSockets.Core.Common.Socket;
    using XSockets.Plugin.Framework;
    using (var container = Composable.GetExport<IXSocketServerContainer>())
    {
        server.StartServers();
        Console.WriteLine("Started, hit enter to quit");
        Console.ReadLine();
    }

####Install into OWIN (self hosted)
**Note: This requires .NET 4.5+**
Open up the Package Manager Console and install the server

`PM> Install-Package Microsoft.Owin.SelfHost` **(include `-pre` to get v 4.0)**

`PM> Install-Package XSockets.Owin.Host` **(include `-pre` to get v 4.0)**

#####How to register XSockets Middleware
UseXSockets is an extension method for the OwinExtensions class.

    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            app.UseXSockets(true);
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            var url = "http://localhost:9090";
            WebApp.Start<Startup>(url);
            Console.WriteLine("Listening at " + url);
            Console.ReadLine();
        }
    }

####Install into OWIN (IIS)
**Note: This requires .NET 4.5+**
Open up the Package Manager Console and install the server

`PM> Install-Package XSockets.Owin.Host` **(include `-pre` to get v 4.0)**

#####How to register XSockets Middleware
UseXSockets is an extension method for the OwinExtensions class.

    using Microsoft.Owin;
    using Owin;
    using XSockets.Owin.Host;
    [assembly: OwinStartupAttribute(typeof(MyApplication.Startup))]
    namespace MyApplication
    {
        public partial class Startup
        {
            public void Configuration(IAppBuilder app)
            {           
                app.UseXSockets(true);       
            }
        }
    }
    
So basically just call the UseXSockets extension from where you have your StartUp class for OWIN.

##Supported Platforms
###Server system requirements

Since XSockets.NET is built to run on both .NET and Mono the server can be hosted pretty much anywhere.

###Client system requirements

XSockets can be used from any client that has TCP/IP, and since XSockets.NET supports cross-protocol communication it is very easy to add new clients and protocols.

The XSockets.NET team has built a few client libraries to make life easier for our developers. All client libraries is using full-duplex/bi-directional communication.

    Language    Requirements
    C#          .NET 3.5, 4.0+
    iOS         .NET 4.0+ (MonoTouch)
    Android     .NET 4.0+ (MonoDroid)
    NETMF       4.2, 4.3
    JavaScript  Browser with websockets (there is a fallback for IE 9 & IE 8)

###Communication - How it works

The architecture for XSockets.NET is simple yet powerful. Each client connects to a protocol, the protocol will then allow communication over n controllers. So you can multiplex over several controller on one connection. This architecture enables communication cross-protocol as well as cross-controller.

Different clients have different capabilities, browsers for example have the RFC6455 (websockets) protocol implemented, but other things devices might talk a protocol that you have created. Since XSockets allows “cross-protocol communication” all connected clients can communicate with each other very easily.

####Basic architecture
[image here]

The red clients are clients libraries written by XSockets.NET and the blue clients are examples of what we have easily connected with custom protocols.

You may also notice that XSockets enables not only cross-protocol communication, but also cross-controller communication so that you can call a method on another controller. Or even send data to client on another controller with a single line of code.

#XSockets.NET 4 - Documentation

This section covers server and cliens API´s for XSockets.NET

##Server API Guide
This section provides examples for server side development but contains sample code for both sever side and client side

###How to create and use Controller classes
To create a `Controller`, create a class that derives from `XSockets.Core.XSocket.XSocketController`. The following example shows a simple `Controller` class for a chat application.

    public class Chat : XSocketController
    {
        public void ChatMessage(string message)
        {
            this.InvokeToAll(message,"chatmessage");
        }
    }

In this example, a connected client can call the ChatMessage method, and when it does, the data received is broadcasted (RPC) to all clients connected to the `Controller`.

####Give the controller an Alias

If you want to specify a different name for clients to use, add the `XSocketsMetadata` attribute and set the `Alias` to the name you want for the controller. You might wanna do this is you have long and complex names for controller on the server.

Server

    [XSocketMetadata(PluginAlias = "Chat")]
    public class MyToLongAndComplexClassNameForTheChat : XSocketController
    
And then you can connect using the `Alias` as shown below.
    
Client - JavaScript

    var conn = new XSockets.WebSocket('ws://127.0.0.1:4502',['chat']);

Client - C#

    var conn = new XSocketsClient("ws://127.0.0.1:4502","http://localhost","chat");

####Controller object lifetime

You don't instantiate the `Controller` class or call its methods from your own code on the server. This is all done for you by the XSockets.NET plugin framework. XSockets.NET creates a new instance of your Controller the first time you use it on you connection. The `Controller` will live in memory for as long as the client is connected to it. This provides the possibility to have state on the controllers which is the most important feature when working with real-time frameworks.

Because the instances of the 'Controller' class ARE transient, you can use them to maintain state from one method call to the next. Each time the server receives a method call from the client, it will be the same instance of your ´Controller´ class per connection that processes the message. Since XSockets.NET does not recycle you will not loose data even though it is stored in-memory. Of course you should persist information that is important since information will be lost if/when the server is stopped.

If you want to send messages to clients from your own code that runs outside the `Controller` class but in the same context as the server, you can do it by instantiating a `Controller` class instance. Note: Instances created like this will not have a socket but they can still be used to send messages to clients connected on any `Controller`.

####How to define methods in the Controller class that the clients can call

To expose a method on the `Controller` that you want to be callable from the client, declare a public method, as shown in the following examples.

    public class Chat : XSocketController
    {
        public void ChatMessage(string message)
        {
            this.InvokeToAll(message,"chatmessage");
        }
    }
    public class StockTicker : XSocketController
    {
        public IEnumerable<Stock> GetAllStocks()
        {
            return _stockTicker.GetAllStocks();
        }
    }
    
You can specify a return type and parameters, including complex types and arrays, as you would in any C# method. Any data that you receive in parameters or return to the caller is communicated between the client and the server by using JSON, and XSockets.NET handles the binding of complex objects and arrays of objects automatically.

####How to call client methods from the Controller class
To call client methods from the server, use the extensionmethods for the `IXSocketController` interface. The following example shows server code that calls `chatmessage` on all connected clients, and the client code that defines the method in a JavaScript and C# clients.

There are many extensions for both PUB/SUB and RPC, and you can of course write your own if needed.

Server

    public class Chat : XSocketController
    {
        public void ChatMessage(string message)
        {
            this.InvokeToAll(message,"chatmessage");
        }
    }
    
Client - JavaScript

    conn.controller('chat').chatmessage = function(data){
        console.log(data);
    };

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));
    
You can specify complex types and arrays for the parameters. The following example passes a complex type to the client in a method parameter.

Server code that calls a client method using a complex object

    public void ChatMessage(string message)
    {
        this.InvokeToAll(new {Text=message},"chatmessage");
    }

We use an anonymous "complex" object here, but you can of course create custom models (classes) as well.

Client - JavaScript

    conn.controller('chat').chatmessage = function(data){
        console.log(data.Text);
    };

Client - C#

    conn.Controller("chat").On("chatmessage", data => Console.WriteLine(data.Text));

*Above we get a dynamic since we do not specify the datatype, but we can of course use a complex type to get the message deserialized into the correct type and get intellisense.*

#### How to use state on the server
All public getters and setters are accessible from the client API, so you can actually `get` and `set` the `Controller` properties from the client API's.

Below you can see that the chat example is extended with a property for username. Since we can use `state` to know who the user is at all times there is no need passing the unnecessary data with every message.

Server `Controller` with state

    public class Chat : XSocketController
    {
        public string UserName {get;set;}
        
        public void ChatMessage(string message)
        {
            this.InvokeToAll(new {UserName = this.UserName, Text = message},"chatmessage");
        }
    }
    
Set the UserName from the client API's

Client - JavaScript

    conn.controller('chat').setProperty('username','Espen Knutsen');

Client - C#

    conn.Controller("chat").SetProperty("username","Espen Knutsen");
    
So to repeat... Since we at all times know the username there is no need for passing it to the server when we can attach the user on the message going out.

####How to hide methods and properties
You might not wanna expose all publish methods and properties to the client API's. When you want to hide a publish method/property just decorate the method/property with the `[NoEvent]` attribute. The attribute is located under `XSockets.Core.Common.Socket.Event.Attributes`

####How to use complex objects as parameters
Above we looked at how you can send complex types to the clients, of course the client can send complex types to the server as well.

**Server**

A simple model for our chat sample

    public class ChatModel
    {
        public string UserName {get;set;}
        public string Text {get;set;}
    }

Server code that accepts a complex object and sends it to all clients. Since we have the username (showed in `How to use state on the server`) we never pass that in, we only set it before sending data out.

    public void ChatMessage(ChatModel message)
    {
        message.UserName = this.UserName;
        this.InvokeToAll(message,"chatmessage");
    }

**Clients**

Client - JavaScript

    conn.controller('chat').invoke('chatmessage',{Text:'Hello from JS'});

Client - C#

    conn.Controller("chat").Invoke("chatmessage", new ChatModel{Text="Hello from CS"});

####How to use IMessage as parameter
When you do not process the data server-side there is no need serializing incoming messages. In these situations you can use `IMessage` as parameter since any message sent into XSockets.NET will be transformed into `IMessage` internally.

This is how the `Generic` controller used in the `Getting started with real-time communication` sample is built. The code for `Generic` is very simple and looks exactly like this.

    public class Generic : XSocketController
    {
        public override OnMessage(IMessage message)
        {
            this.InvokeToAll(message);
        }
    }
    
In the `Generic` `Controller` we override the `OnMessage` method which means that any call to the `Controller` that does not match a `method` will end up in the `OnMessage` method.

You can of course use the `IMessage` parameter in regular methods on the `Controller` and not just the `OnMessage` method.

####Return data synchronously to caller
When you call the server from a client and want to wait for the result you just replace void with the type you want to return and the client API's will wait for the result.

**Server**

    public string Echo()
    {
        return "Echo";
    }
    
You can of course send data before doing the return, for example.

    public string Echo()
    {
        this.InvokeToAll("someData", "someMethod");
        //and so on...
        return "Echo";
    }
    
**Clients**

Client - JavaScript

    conn.controller('chat').invoke('echo').then(function(d){console.log(d)});

Client - C#

    var echo = conn.Controller("chat").Invoke<string>("echo");
    Console.WriteLine(echo.Result);

####How to do overloading of methods

    TBD

####How to handle binary data
Lets say that we have a file `c:\temp\xfile.txt` with the text `This file was sent with XSockets.NET` and we want to send that file to the server.

**Server**

    public void MyFile(IMessage message)
    {
        var filecontent = Encoding.UTF8.GetString(message.Blob.ToArray());
    }

**Clients**

Client - JavaScript

    N/A

Client - C#

    var blob = File.ReadAllBytes(@"c:\temp\xfile.txt");
    conn.Controller("chat").Invoke("myfile", blob);

####How to pass meta-data together with binary data
If we want to attach metadata about the binary data that is easy to do. Just pass along the object representing the metadata and XSockets will let you extract that data on the server.

**Server**
    
    //simple class for holding metadata about a file
    public class FileInfo
    {
        public string Name {get;set;}
    }

    public void MyFile(IMessage message)
    {
        var filecontent = Encoding.UTF8.GetString(message.Blob.ToArray());
        var metadata = message.Extract<FileInfo>();
    }

Just use `Extract<T>` to get back to metadata attached to the binary data.

**Clients**

Client - JavaScript

    N/A

Client - C#

    var blob = File.ReadAllBytes(@"c:\temp\xfile.txt");
    conn.Controller("chat").Invoke("myfile", blob, new {Name="xfile.txt"});

----------

###RPC
The RemoteProcedureCall pattern let you do exactly what is says, call procedures remotely. RPC together with XSockets fine grained control let you send messages to specific clients in a very smooth way.

You will see more about this powerful feature combined with RPC below
####How to call methods on the client
If you just want to send a message to the caller of the method, use `Invoke`

**Server**
    
    //Send a message to the caller
    this.Invoke("Hello to caller from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').chatmessage = function(data){console.log(data)};

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

####How to call methods on all clients
If you want to send a message to all clients connected to the controller, use `InvokeToAll`

**Server**
    
    this.InvokeToAll("Hello to all from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').chatmessage = function(data){console.log(data)};

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

####How to target a subset of the connected clients
If you want to send a message to some of the clients connected to the controller, use `InvokeTo`

This is where you will see the power of state! You will actually get intellisense so that you can write lambda expressions to target the clients you want to send the message to.

If we have the properties Age and Gender on the `Controller` (for example) we can target exactly the ones we want to.
Below for example we send to all clients having the same gender and location as the caller

**Server**
    
    this.InvokeTo(p => p.Gender == this.Gender && p.City == this.City,"Hello to some from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').chatmessage = function(data){console.log(data)};

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

####How to send messages to clients connected on another Controller class
It is pretty much the same as sending to the clients on the same `Controller` since XSockets extension-methods are generic.

So, if you are on `Controller` A and want to send to all clients on `Controller` B you just use...

    //To all
    this.InvokeToAll<B>("Hello to all from server", "target");
    
    //And the powerful `InvokeTo<T>`  would have a signature like:
    this.InvokeTo<T>(Func<T,bool> expression, object obj, string target);

####How to call client methods outside the Controller class
Just create a controller instance (or ask the plugin framework after the specific controller) and then use the extensions to send data

    //Create the instance your self
    var chat = new Chat();
    //Then just use one of the extensions to target, one, some, others or all...
    chat.InvokeToAll<Chat>("Hello from manual instance","say");
    
Notice that you have to pass in <T> in the extension regardless of what controller you want to use. So you can even do `new Chat().InvokeToAll<Stock>("Hello to stock clients from chat instance","hi");`
    
*Note: there is no point in using Invoke/Publish/Send since the actual controller does not have a socket since there is no client connected to it (we created the instance manually).*

----------

###PUB/SUB
The publish/subscribe pattern is useful when your applications need the users to subscribe to the topics that you are about to publish. Of course you can have very good control with XSockets since you can select subsets of subscribers very easy to get more fine grained control.

You will see more about this powerful feature combined with PUB/SUB below.
####How to publish data to subscribers
If you just want to send a message to the publisher/caller of the method, use `Publish`
The big difference between `Publish` and `Invoke` is that the message only will arrive at the client if the client has a `Subscription` registered on the server.

**Server**
    
    //Publish a message to the caller
    this.Publish("Hello to caller from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').subscribe('chatmessage', function(data){
        console.log(data);
    });

Client - C#

    conn.Controller("chat").Subscribe<string>("chatmessage", data => Console.WriteLine(data) );

####How to publish to all subscribers
If you want to publish a message to all clients subscribing to a topic, use `PublishToAll`

**Server**
    
    this.PublishToAll("Hello to all subscribers from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').subscribe('chatmessage', function(data){
        console.log(data)
    });

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

####How to publish to a subset of the subscribing clients
If you want to send a message to some of the subscribing clients, use `PublishTo`

This is where you will see the power of state! You will actually get intellisense so that you can write lambda expressions to target the subscribers you want to send the message to.

If we have the properties Age and Gender on the `Controller` (for example) we can target exactly the ones we want to.
Below for example we send to all clients having the same gender and location as the caller

**Server**
    
    this.PublishTo(p => p.Gender == this.Gender && p.City == this.City,"Hello to some from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').subscribe('chatmessage', function(data){
        console.log(data)
    });

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

####How to publish messages to subscribers connected on another Controller class
It is pretty much the same as publishing to the clients on the same `Controller` since XSockets extension-methods are generic.

So, if you are on `Controller` A and want to publish to all `chatmessage` subscribers on `Controller` B you just use...

    //To all
    this.PublishToAll<B>("Hello to all 'chatmessage' subscribers on 'controller' B from server", "chatmessage");
    
    //And the powerful `PublishTo<T>`  would have a signature like:
    this.PublishTo<T>(Func<T,bool> expression, object obj, string target);

####How to call subscribers outside the Controller class
Just create a controller instance (or ask the plugin framework after the specific controller) and then use the extensions to send data

    //Create the instance your self
    var chat = new Chat();
    //Then just use one of the extensions to target, one, some, others or all...
    chat.PublishToAll<Chat>("Hello from manual instance","say");
    
Notice that you have to pass in <T> in the extension regardless of what controller you want to use. So you can even do `new Chat().PublishToAll<Stock>("Hello to stock clients from chat instance","hi");`

*Note: there is no point in using Invoke/Publish/Send since the actual controller does not have a socket since there is no client connected to it (we created the instance manually).*
    
----------

###How to handle connection lifetime events in the Controller class
Each controller have events that you can use to know when `Open`, `Close` and `ReOpen` occurs.

####The OnOpen event
Will fire when the controller is used over the connection for the first time.
    
    public class Chat : XSocketController
    {
        public Chat()
        {
            this.OnOpen += Chat_OnOpen;
        }

        void Chat_OnOpen(object sender, OnClientConnectArgs e)
        {
            //Connection to controller is open
        }
    }

####The OnClose event
Will fire when the socket is closed or if the client choose to close this specific controller on the connection.

    public class Chat : XSocketController
    {
        public Chat()
        {
            this.OnClose += Chat_OnClose;
        }

        void Chat_OnClose(object sender, OnClientDisconnectArgs e)
        {
            //Connection to controller was closed
        }
    }

####How to do heartbeats (ping/pong control-frames)
Control-frames (ping/pong) is primarily for checking connections, measure latency and validate that the connection is valid.

You can implement custom logic for sending/receiving these frames on the server, but there is a simple helper that you can use to get the functionality.

    public class Chat : XSocketController
    {
        public Chat()
        {
            this.OnOpen += Chat_OnOpen;
        }

        void Chat_OnOpen(object sender, OnClientConnectArgs e)
        {
            this.ProtocolInstance.Heartbeat(notifyAction: (reason)=>{ Console.WriteLine(reason); });
        }
    }

### Modules/Plugins
This section covers how to create custom modules/plugins for different parts of XSockets. This only covers the `Interfaces` within XSockets, but you can ofcourse add your custom interfaces as plugins by telling the plugin framework about the interfaces you want to use. Read more about custom plugin in `The Plugin Framework` section

####How to implement your custom Pipeline
The server will only have one `XSockets.Core.Common.Socket.IXSocketPipeline`, but you can override the default one by just deriving it.

Each message passing into the server or out of the server will pass the pipeline

    public class MyPipeline : XSocketPipeline
    {
        public override void OnIncomingMessage(IXSocketController controller, IMessage e)
        {
            Debug.WriteLine(string.Format("IN:  Controller:{}, Topic:{1}, Data:{2}", e.Controller, e.Topic, e.Data));
            base.OnIncomingMessage(controller, e);
        }

        public override IMessage OnOutgoingMessage(IXSocketController controller, IMessage e)
        {
            Debug.WriteLine(string.Format("OUT: Controller:{}, Topic:{1}, Data:{2}", e.Controller, e.Topic, e.Data));
            return base.OnOutgoingMessage(controller, e);
        }
    }
    
#### Write a custom AuthenticationPipeline
When the socket is connected and the handshake is completed the `AuthenticationPipeline` will be called. By default the pipeline will look for a FormsAuthenticationTicket, but you can override this pipline by just implementing a interface `XSockets.Core.Common.Socket.IXSocketAuthenticationPipeline`

There can only be one pipeline so even if you implement several pipelines only one wil be used.

    [Export(typeof(IXSocketAuthenticationPipeline))]
    public class MyAuthenticationPipeline : IXSocketAuthenticationPipeline
    {
        public IPrincipal GetPrincipal(IXSocketProtocol protocol)
        {            
            if (protocol.ConnectionContext.User == null)
            {
                //creating a fake super user ;)
                var roles = new string[]{"superman","hulk"};
                var userIdentity = new GenericIdentity("David");
                protocol.ConnectionContext.User = new GenericPrincipal(userIdentity, roles);
            }            

            return protocol.ConnectionContext.User;
        }
    }

#### Interceptors Concept
There can only be one `Pipeline`, and one `AuthenticationPipeline`, but interceptors can be 0 to N. Every interceptor will be called at a specific time. ConnectionInterceptors for example will be called when someone connects, disconnects or when a handshake is completed (ok or not).

Interceptors are common when debugging or logging, but XSockets does not choose a logger for you. Implement the interface and do whatever you want inside of the interceptor(s).

##### How to write ConnectionInterceptors
Sample of a connection interceptor that logs handshake and connect/disconnect with `debug` level

    public class MyConnectionInterceptor : IConnectionInterceptor
    {

        public void Connected(IXSocketProtocol protocol)
        {
            LogHelper.Log(LogEventLevel.Verbose, "Connected {@protocol}" , protocol.ConnectionContext);            
        }

        public void Disconnected(IXSocketProtocol protocol)
        {
            LogHelper.Log(LogEventLevel.Verbose, "Disconnected {@protocol}", protocol.ConnectionContext);
        }

        public void HandshakeCompleted(IXSocketProtocol protocol)
        {
            LogHelper.Log(LogEventLevel.Verbose, "Handshake ok {@protocol}", protocol.ConnectionContext);
        }

        public void HandshakeInvalid(string rawHandshake)
        {
            LogHelper.Log(LogEventLevel.Verbose, "Handshake failed {raw}", rawHandshake);
        }
    }
    
######How to write MessageInterceptors
Sample of a message interceptor that logs in/out messages with `debug` level

    public class MyMessageInterceptor : IMessageInterceptor
    {
        public void OnIncomingMessage(IXSocketProtocol protocol, IMessage message)
        {
            LogHelper.Log(LogEventLevel.Debug, "{incoming message {@message}}", message);
        }

        public void OnOutgoingMessage(IXSocketProtocol protocol, IMessage message)
        {
            LogHelper.Log(LogEventLevel.Debug, "{outgoing message {@message}}", message);
        }
    }
    
######How to write ErrorInterceptors
Sample of a error interceptor that logs exceptions with `error` level

    public class MyErrorInterceptor : IErrorInterceptor
    {
        public void OnError(IXSocketException exception)
        {
            LogHelper.Log(LogEventLevel.Error, "{Exception {@ex}}", exception);
        }
    }
    
####How to write a custom JSON Serializer
You can replace the default serializer with your own favorite serializer. As everything else in XSockets.NET it is just a module/plugin. Implement the `IXSocketJsonSerializer` interface and export the new plugin to tell XSockets to use your serializer instead of the defualt one.

In the sample below we use the ServiceStack.Text serializer

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using ServiceStack.Text;
    using XSockets.Core.Common.Utility.Serialization;

    namespace MyNameSpace.Serialization
    {
        [Export(typeof(IXSocketJsonSerializer))]
        public class MyJsonSerializer : IXSocketJsonSerializer
        {
            public MyJsonSerializer()
            {
                JsConfig.ExcludeTypeInfo = true;
                JsConfig.IncludeTypeInfo = false;
            }

            public string SerializeToString<T>(T obj)
            {
                return JsonSerializer.SerializeToString(obj);            
            }

            public string SerializeToString(object obj, Type type)
            {
                return JsonSerializer.SerializeToString(obj, type);
            }

            public T DeserializeFromString<T>(string json)
            {
                return JsonSerializer.DeserializeFromString<T>(json);
            }

            public object DeserializeFromString(string json, Type type)
            {           
                return JsonSerializer.DeserializeFromString(json, type);
            }

            public IDictionary<string,string> DeserializeFromString(string json, params string[] keys)
            {
                var obj = JsonSerializer.DeserializeFromString<List<JsonObject>>(@json);
                return keys.ToDictionary(key => key, key => obj[0].Child(key));
            }
        }    
    }
    
####How to write custom Protocols

    TODO
    
###Securing the Controller
XSockets.NET does not provide any features for authenticating users. Instead, you integrate the XSockets:NET features into the existing authentication structure for an application. You authenticate users as you would normally in your application, and work with the results of the authentication in your XSockets.NET code. For example, you might authenticate your users with ASP.NET forms authentication, and then in your `Controller`, enforce which users or roles are authorized to call a method.

XSockets provides the Authorize attribute to specify which users have access to a `controller` or `method`. You apply the Authorize attribute to either a `controller` or particular `methods` in a `controller`. Without the Authorize attribute, all public methods on the controller are available to a client that is connected to the controller. 

If you want to allow unrestrcited access on some methods you can add the `AllowAnonymous` attribute the these methods.

More information about attributes and methods below

#### Authorize Attribute

This attribute can be set on Controller or Method level. If set at controller level all action methods that do not have the `AllowAnonymous` attribute will require authentication.

The authorize attribute can take Roles and Users but if using that you will have to implement your own authentication by overriding `OnAuthorization(AuthorizeAttribute authorizeAttribute)`

#### AllowAnonymous Attribute

This attribute can be set on action methods and will then allow anonymous access.

#### Get FormsAuthentication Ticket

When you have custom authentication you can get the `FormsAuthenticationTicket` from this method.

    var ticket = GetFormsAuthenticationTicket();

**Note: If you do not pass in a cookiename .ASPXAUTH will be used.**

**Important: If you have separate project you will have to use the same origin to be able to get cookies and also use machine-key in the config to be able to get the AuthCookie.**
***See*** http://msdn.microsoft.com/en-us/library/system.web.configuration.machinekeysection.compatibilitymode%28v=vs.110%29.aspx ***if you are using different framework versions in the projects.***

#### Write a custom AuthenticationPipeline
When the socket is connected and the handshake is completed the `AuthenticationPipeline` will be called. By default the pipeline will look for a FormsAuthenticationTicket, but you can override this pipline by just implementing a interface `XSockets.Core.Common.Socket.IXSocketAuthenticationPipeline`

There can only be one pipeline so even if you implement several pipelines only one wil be used.

    [Export(typeof(IXSocketAuthenticationPipeline))]
    public class MyAuthenticationPipeline : IXSocketAuthenticationPipeline
    {
        public IPrincipal GetPrincipal(IXSocketProtocol protocol)
        {            
            if (protocol.ConnectionContext.User == null)
            {
                //creating a fake super user ;)
                var roles = new string[]{"superman","hulk"};
                var userIdentity = new GenericIdentity("David");
                protocol.ConnectionContext.User = new GenericPrincipal(userIdentity, roles);
            }            
            return protocol.ConnectionContext.User;
        }
    }

#### How to override the OnAuthorization method
The `OnAuthorization` method is called for every method on a controller that has authentication. The attribute for the method (or controller) is passed in and we check that 

 1. That the user is authenticated
 2. If the user name is required we verify a match
 3. If user name was not required or did not match we check roles

        public virtual bool OnAuthorization(IAuthorizeAttribute authorizeAttribute)
        {
            try
            {
                var user = this.ProtocolInstance.ConnectionContext.User;
                if (user.Identity.IsAuthenticated)
                {
                    if (!string.IsNullOrEmpty(authorizeAttribute.Roles) || !string.IsNullOrEmpty(authorizeAttribute.Users))
                    {                    
                        if (authorizeAttribute.Users.Split(',').Contains(user.Identity.Name)){
                            return true;
                        }
                        return authorizeAttribute.Roles.Split(',').Any(user.IsInRole);      
                    }
                    return true;
                }
                return false;
            }
            catch
            {
                return false;
            }
        }

**Example**
Based on the `Write a custom AuthenticationPipeline` where we added the username `Hero` and the roles `hulk`, `superman` we have the following scenario

    [Authorize()] //would be valid since we have a authorized `fake` user

    [Authorize(Users = "David")] //would be valid

    [Authorize(Roles = "batman,robin")] //would be invalid
    
    [Authorize(Roles = "batman,hulk")] //would be valid
    
#### How to know when authorization fails
Every time a method on a controller needs authentication the `OnAuthorization` method is called (which was overridden in previous sample).

When the `OnAuthorization` returns false the `OnAuthorizationFailed` event is fired.

    //Constructor
    public Chat()
    {
        this.OnAuthorizationFailed += Chat_OnAuthorizationFailed;
    }

    void Chat_OnAuthorizationFailed(object sender, OnAuthorizationFailedArgs e)
    {
        Console.WriteLine("Auth Failed: {0},{1}", e.Controller, e.MethodName); 
    }

The `OnAuthorizationFailedArgs` contains information about the controller and the method being called. Information about the user that called the method can be accessed from `this.ProtocolInstance.ConnectionContext.User`

###How to handle errors in the Controller class
Wrap you logic in a try catch block and call the `HandleError` method that will invoke the `OnError` event. 

When needed you can also send the error to the `ErrorInterceptors` if you have implemented any.

    try
    {
        throw new Exception("boom!");
    }
    catch(Exception ex)
    {
        this.HandleError(ex);
        ErrorInterceptorsQueue.Push(ex);
    }

###How to create internal (long-running) Controllers
A common scenario is that you want to do something every x seconds on the server. It can be polling a legacy database etc. And then push information to clients based on criterias just as you do in any XSockets server side method.

In XSockets you can write long-running controllers. A long-running controller will be a `singleton` that only executes inside of the server. Clients can´t connect to a long-running controller.

The simple sample below will send a chatmessage to all clients connected to the `Chat` controller from the long-running controller. The line that will make the controller a long-running controller is `[XSocketMetadata("MyLongrunningController", PluginRange.Internal)]` That tells XSockets to use the controller as a singleton and that it should be internal only.

    /// <summary>
    /// This is a longrunning controller. This cant be connected to.
    /// It is a singleton that will run inside xsockets as long as the server is alive.
    /// Most common is to start a timer in it to perform some task
    /// </summary>
    [XSocketMetadata("MyLongrunningController", PluginRange.Internal)]   
    public class MyLongrunningController : XSocketController
    {
        private Timer timer;
        
        public MyLongrunningController()
        {
            timer = new Timer(10000);
            timer.Elapsed += timer_Elapsed;
            timer.Start();
        }

        void timer_Elapsed(object sender, ElapsedEventArgs e)
        {
            //Sending a message to all clients on the Chat controller
            this.InvokeToAll<Chat>("Hello from long-running controller", "say");
        }
    }

###How to get information about the client from the ConnectionContext

####Getting Cookies
You can get the `CookieCollection` by using

    var cookies = this.ProtocolInstance.ConnectionContext.CookieCollection;

Note: you will only be able to access cookies if your server is on the same domain as the cookie domain.

####Getting Headers
You can get the headers `NameValueCollection` by using

    var headers = this.ProtocolInstance.ConnectionContext.Headers;

####Getting Query Strings
You can access the query string `NameValueCollection` by using

    var querystring = this.ProtocolInstance.ConnectionContext.QueryString;

####Getting the current User (IPrincipal)
You can get/set the IPrincipal by using

    var principal = this.ProtocolInstance.ConnectionContext.User;


----------

## C# Client API Guide
The C# Client API has support for:

 - .NET 3.5, 4.0+
 - iOS (MonoTouch)
 - Android (MonoDroid)
 - .NET MicroFramework 4.2, 4.3 (see specific API for NETMF)

###Client Setup
To get the client just get the latest package from http://nuget.org/packages/xsockets.client

 **NOTE: Use the suffix `-pre` to get v 4.0**

The C# clients ALWAYS talk full-duplex/bi-directional communication, and just like the XSockets server this behavior has nothing to do with what OS or WebServer you are running.

###How to establish a connection
Just like in the JavaScript client you can multiplex over several `Controller` on one connection. To get a connections just create an instance of the XSocketClient.

    var conn = new XSocketClient("ws://localhost:4502", "http://localhost", "chat");
    conn.Open();
    
- The first parameter is the endpoint of the server
- The second parameter is the origin of the client
- The third parameter is params string[] for setting your controllers

Do note that the call to `Open` is synchron and will wait until the connection is open and the handshake is completed.

###How to configure the connection
####How to use multiple Controllers
To multiplex over several controllers on one connection you just pass in the controllers to use.

Below we connect to `Controllers` `one` and `two`

    var conn = new XSocketClient("ws://localhost:4502", "http://localhost", "one","two");
    
Do note that you do not have to specify all controllers in the creation of the client. As soon as you start using a controller in the client API you will get an instance of it.

So if there is no controller `three` defined in the connection you can still do:

    conn.Controller("three").Invoke("somemethod");

####How to specify query string parameters
You can add querystrings to the `NameValueCollection` named `QueryString` on the `IXSocketClient`
    
    var conn = new XSocketClient("ws://localhost:4502","http://xsockets.net", "chat");
    conn.QueryString.Add("color","blue");
    conn.Open();
    
####How to specify HTTP headers

    var conn = new XSocketClient("ws://localhost:4502","http://xsockets.net", "chat");
    conn.Headers.Add("headername", "headervalue");
    conn.Open();
    
####How to specify client credentials

##### Connection header
    
    var conn = new XSocketClient("ws://localhost:4502","http://xsockets.net", "chat");
    conn.Headers.Add("myauthtoken", /* the token data */);
    conn.Open();
    
Then verify the token in a custom `AuthenticationPipeline` or in the `OnOpen` even of the specific controller

##### Cookie with Forms Authentication

    var authTicket = new FormsAuthenticationTicket(1, "Uffe",DateTime.Now,DateTime.Now.AddMinutes(15),false, "batman|hulk");            
    string encTicket = FormsAuthentication.Encrypt(authTicket);
    var authcookie = new Cookie(FormsAuthentication.FormsCookieName, encTicket);
    conn.Cookies.Add(authcookie);

##### Certificate
    
    var conn = new XSocketClient("ws://localhost:4502", "http://xsockets.net", "chat");
    conn.AddClientCertificate(new X509Certificate2("mycert.pfx"));
    conn.Open();
    
##### Windows Authentication

    TODO
    
###RPC
####How to define methods on the client that the server can call
You define the methods per `Controller` since one connection can communicate over several `Controllers`

**Client**

The example below creates a listener for `ChatMessage` and it expects the data to be a string.

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

**Server**

The server code for the examples with the `ChatModel` can look like

    this.InvokeToAll("I am Yogi the gummi bear","chatmessage");

####Methods without parameters
If the method you're handling does not have parameters just use the non generic `On` method.

**Server**

    public class Chat : XSocketController
    {
        public void CallAllClients()
        {
            this.InvokeToAll("test");
        }
    }
    
Note: You can of course use `InvokeTo<T>` or `Invoke` as well

**Client**
    
    conn.Controller("chat").On("test", () => Console.WriteLine("Test Was Called"));

####Methods with parameters, specifying parameter types

If we have a complex object being sent from the server, like the `ChatModel` we can of course just do like this

    conn.Controller("chat").On<ChatModel>("chatmessage", data => Console.WriteLine(data.UserName + " - " + data.Text));
    
**Server**

The server code for the examples with the `ChatModel` can look like

    this.InvokeToAll(new ChatModel{Name="Yogi", Text="I am a gummi bear"},"chatmessage");

####Methods with parameters, specifying dynamic objects for the parameters
As an alternative to specifying parameters as generic types of the On method, you can specify parameters as dynamic objects. Below we get the `data` parameter as an dynamic since we did not specify the generic parameter for the `On` method.

    conn.Controller("chat").On("chatmessage", data => Console.WriteLine(data.Text));
    
*Note: Dynamic keyword will not exist in clients for Android/iOS or .NET 3.5 and earlier.*

**Server**

The server code for the examples with the `ChatModel` can look like

    this.InvokeToAll(new ChatModel{Name="Yogi", Text="I am a gummi bear"},"chatmessage");
    
####Methods with parameter of type IMessage
If you for some reason want the actual IMessage sent from the server to the client you can specify IMessage as the type. You will then get the complete object that the client received!

    conn.Controller("chat").On<IMessage>("chatmessage", o => Console.WriteLine("{0}, {1} {2}, {3},",o.Controller, o.Topic, o.Data, o.MessageType);
    
If you know the type contained in the Data part of the IMessage you can use Extract<T> to get the value.

    //example extracting the JSON into a specific type
    var chatModel = o.Extract<ChatModel>();
    
*Note: Dynamic keyword will not exist in clients for Android/iOS or .NET 3.5 and earlier.*

**Server**

The server code for the examples with the `ChatModel` can look like

    this.InvokeToAll(new ChatModel{Name="Yogi", Text="I am a gummi bear"},"chatmessage");
    
####How to remove a handler
When you want to dispose of a handler you have to remove it from the `Controller` where it was used.

    var listener = conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));   
                
    conn.Controller("chat").DisposeListener(listener);

####How to send to server methods from the client
To call a method on the server, use the Invoke method on the `Controller`.

If the server method has no return value, use the non-generic overload of the Invoke method.

**Server - method without return value**

    public class Chat : XSocketController
    {
        protected string UserName {get;set;}
        public void SetUserName(string userName)
        {
            this.UserName = userName;
        }
    }
    
**Client - calling a method that has no return value**

    conn.Controller("chat").Invoke("setusername", new {userName = "Steve"});

####How to call a server side method and wait for the result
If the server method has a return value, specify the return type as the generic type of the Invoke method.

**Server - for a method that has a return value**

    public IEnumerable<Stock> GetStocks()
    {
        return _stockTicker.GetAllStocks();
    }
    
The Stock class used for return value

    public class Stock
    {
        public string Symbol { get; set; }
        public decimal Price { get; set; }
    }


**Client - calling a method that has a return value in a synchronous method**

    var stocks = conn.Controller("stockticker").Invoke<IEnumerable<Stock>>("GetStocks");
    foreach (Stock stock in stocks)
    {
        Console.WriteLine("Symbol: {0} price: {1}\n", stock.Symbol, stock.Price);
    }
    
If there is no respons for 2000 ms there will be a `TimeoutException` so you should wrap the synchronous call like:

    try
    {
        var stocks = conn.Controller("stockticker").Invoke<IEnumerable<Stock>>("GetStocks");
        foreach (Stock stock in stocks)
        {
            Console.WriteLine("Symbol: {0} price: {1}\n", stock.Symbol, stock.Price);
        }
    }
    catch (AggregateException ae)
    {
        ae.Handle((x) =>
        {
            if (x is TimeoutException)
            {
                //The communication did not respond within given time frame
                //Handle it...
                return true;
            }
            //Another exception handle it... or return false to stop app
            return false;
        });
    }
    
Of course you can set the default 2000 ms to be longer or shorter if needed. Just pass in your timeout as a parameter in the call like
    
    //Timeout will now be 5 seconds
    var stocks = conn.Controller("stockticker").Invoke<IEnumerable<Stock>>("GetStocks",5000);
    
###PUB/SUB
####How to define subscription methods on the client that the server can publish to

    conn.Controller("stockticker").Subscribe<Stock>("newStock", data => Console.WriteLine("Symbol: {0} price: {1}\n", stock.Symbol, stock.Price));
        
####How to subscribe one time
If you only want to get a message once and then unsubscribe automatically you can use `one`

    conn.Controller("stockticker").One<Stock>("newStock", data => Console.WriteLine("Symbol: {0} price: {1}\n", stock.Symbol, stock.Price));
    
This will make sure that the client unsubscribe to the topic after getting the first message.

####How to subscribe n times
If you want to get a message 1 to n times and then unsubscribe automatically you can use `many`

    conn.Controller("stockticker").Many<Stock>("change", 3, data => Console.WriteLine(data));

This will unsubscribe the `change` topic after getting 3 messages.    

#### How to know when the subscription is ready
When you compare PUB/SUB with RPC an obvious disadvantage with PUB/SUB is that you do not know if the server has bound the subscription when you do a publish. Therefor you can pass in an additional function to get a callback from the server when the subscription is completed.

This works similar for all subscribe methods.

    Subscribe<T>(string topic, Action<T> callback, Action<IMessage> confirmCallback);
    One<T>(string topic, Action<T> callback, Action<IMessage> confirmCallback);
    Many<T>(string topic, uint limit, Action<T> callback, Action<IMessage> confirmCallback);
    
where `callback` is called when a publish occurs and `confirmCallback` is the callback that confirms the subscription. 

####How to remove a subscription
When you no longer want to subscribe to a `topic` you just use the `unsubscribe` method to tell the server to remove the subscription.

    conn.Controller("chat").Unsubscribe("chatmessage");

####How to publish to server methods from the client

**Client**

    conn.Controller("chat").Publish("chatmessage",new {Text= "Hello people!"});

**Server**

    //The server migth publish the message back to all clients subscribing
    public void ChatMessage(ChatModel chatModel)
    {
        this.PublishToAll(chatModel, "chatmessage");
    }

###How to handle connection lifetime events
The events on connection level provide information about the socket being opened/closed. The controllers has their own lifetime events.

#### OnConnected

    conn.OnConnected += (sender, eventArgs) => Console.WriteLine("Connected");
    conn.Open();
                
#### OnDisconnected
    
    conn.OnDisconnected += (sender, eventArgs) => Console.WriteLine("Disconnected");
    conn.Open();
    
###How to handle controller lifetime events
The controller has nothing to do with the actual socket. These events tell you about the controllers you are using over your connection.

#### OnOpen

    conn.Controller("chat").OnOpen += (sender, connectArgs) => {
        Console.WriteLine("Opened");
    };

#### OnClose
    
    conn.Controller("chat").OnClose += (sender, disconnectArgs) =>{
        Console.WriteLine("Closed");
    };

### Open a controller
As soon as you start to communicate over a new controller the `OnOpen` event will fire. You actually do not need to specify the controller in the connection. As long as the controller exists on the server the `OnOpen` event will fire.

### Close a controller
To close as controller (not the actual connection/socket) you just call the `Close` method on the controller instance. This will fire the `OnClose` event on the controller.

    conn.Controller("chat").Close();

#### OnOpen

    conn.Controller("chat").OnOpen += (sender, connectArgs) => {
        Console.WriteLine("Open {0}", connectArgs.ClientInfo.Controller);
    };
    
#### OnClose

    conn.Controller("chat").OnClose += (sender, disconnectArgs) => {
        Console.WriteLine("Closed {0}", disconnectArgs.ClientInfo.Controller);
    };

###How to handle errors

To handle errors that the client raises, you can add a handler for the Error event on the connection object.

    conn.OnError += (sender, errorArgs) => Console.WriteLine(errorArgs.Exception.Message);
    
You may also handle errors on individual controller using the OnError event on a specific controller.

    conn.Controller("chat").OnError += (sender, errorArgs) => Console.WriteLine(errorArgs.Exception.Message);

To handle errors from method invocations, wrap the code in a try-catch block. 

    try
    {
        var stocks = conn.Controller("stockticker").Invoke<IEnumerable<Stock>>("GetStocks");
        foreach (Stock stock in stocks)
        {
            Console.WriteLine("Symbol: {0} price: {1}", stock.Symbol, stock.Price);
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error invoking GetStocks: {0}", ex.Message);
    }

### Client Pool
The client pool lets you call methods on the server. The client pool is made for sending only and is often used in legacy applications to boost them to real-time with 2 lines of code.

For example, if you have a half-duplex service such as a WCF you can boost it to real-time by just doing this.

    var conn = ClientPool.GetInstance("ws://localhost:4502","http://localhost");
    conn.Send("Hello from client pool","chatmessage","chat");
    
Above we send the text `Hello from client pool` to the method `ChatMessage` on the controller `Chat`. As always you can send any object that can be serialized.

### Using In-Memory Storage
This feature lets you store any serializable object on the server by using simple methods on the client API's. This is useful when you need to store data between connections/pages for the client. You are responsible for cleaning up the memory your self since data written to the storage will remain there until you remove it or the server stops.

When you use the storage you always work on a controller instance since each controller can have its own storage

***Note: You can set any serializable object as value in the storage***

#### Setting data on the server
Set the generic type that you want to store (in this case string)

    conn.Controller("chat").StorageSet("color","blue");

#### Get notifications when the data changes on the server
Since the storage is per client you will only get notifications for changes in the current client.

    conn.Controller("chat").StorageOnSet<string>("color", s => Console.WriteLine("Color {0} was set in storage", s));
    
#### Getting data from the server

    var color = conn.Controller("chat").StorageGet<string>("color");
    Console.WriteLine("Got color {0} from storage", color);

#### Removing data on the server

    conn.Controller("chat").StorageRemove("color");

#### Get notifications when the data is removed on the server

    conn.Controller("chat").StorageOnRemove<string>("color", s => Console.WriteLine("Color {0} was removed from storage", s));

#### Clear the storage
    
    conn.Controller("chat").StorageClear();

----------
##JavaScript Client API  Guide
To get the client just get the latest package from http://nuget.org/packages/xsockets.jsapi

The JavaScript API supports RPC, PUB/SUB and WebRTC.

###Client Setup
The JavaScript client has no dependencies, just add the `XSockets.latest.js` to your page.
You can get the latest package from http://nuget.org/packages/xsockets.jsapi

**NOTE: Use the suffix `-pre` to get v 4.0**

   &lt;script src=&quot;Scripts/XSockets.latest.min.js&quot;&gt;&lt;/script&gt;
  
###How to establish a connection
To get a connection just pass in the endpoint to the server.

    var conn = new XSockets.WebSocket('ws://localhost:4502',['chat']);

Note that the second parameter is an array of controllers to connect to. We only use one controller in this sample.

###How to configure the connection
You have a few options when creating the connection, such as sub protocol, query strings and controllers.
    
####How to specify query string parameters
You might wanna pass in query string parameters in the connection. This is done by setting the third parameter to key-value pairs of JSON-object.

    var conn = new XSockets.WebSocket('ws://localhost:4502',['chat'],{username:'steve',age:35});

####How to specify controllers
Since you can multiplex over several controllers on one connection you can specify the controllers to use when you create the connection. If you would like to connect to three controllers `A`, `B` and `C` you would do:

    var conn = new XSockets.WebSocket('ws://localhost:4502',['a','b','c']);

The names of the controllers is not case sensitive so `A` is the same as `a`.

###RPC

####How to set properties on the server from the client
Since we have state on all controllers in the connection and can take advantage of that we can store information server side and not send trivial things like a user name each time we want to communicate.

**Server**

A simple model for a chat

    public class ChatModel
    {
        public string UserName {get;set;}
        public string Text {get;set;}
    }

A controller where we use state to only send in text since the username is known.

    public class Chat : XSocketController
    {
        public string UserName {get;set;}
        public void ChatMessage(ChatModel message)
        {
            message.UserName = this.UserName;
            this.InvokeToAll(message,"chatmessage");
        }
    }

**Client**

    conn.controller('chat').setProperty('username','David');
  
####How to call server methods from the client

**Server**

A simple model for a chat

    public class ChatModel
    {
        public string UserName {get;set;}
        public string Text {get;set;}
    }

A controller where we use state to only send in text since the user name is know. See `How to set properties on the server from the client`

    public class Chat : XSocketController
    {
        public string UserName {get;set;}
        public void ChatMessage(ChatModel message)
        {
            message.UserName = this.UserName;
            this.InvokeToAll(message,"chatmessage");
        }
    }

**Client**

Since we already have set the property of `UserName` on the server we only need to send the `Text` property

    conn.controller('chat').invoke('chatmessage',{Text:'Calling chatmessage on server and passing a part of the complex object'});
 
####How to define methods on the client that the server can call
To define a method that the server can call from a `Controller`just add a method to the controller in JavaScript. The parameters can be complex objects.

Method name has to be lowercase in the JavaScript API. 
For example:

    this.Invoke('Hello from server','ChatMessage'); 
    
on the server will execute `chatmessage` on the client.

**Client**

    var conn = new XSockets.WebSocket('ws://localhost:4502',['chat']);
    
    conn.controller('chat').chatmessage = function(data){
        console.log(data.UserName + " - " + data.Text);
    };

**Server**

A simple model for a chat

    public class ChatModel
    {
        public string UserName {get;set;}
        public string Text {get;set;}
    }
  
A controller where we use state to only send in text since the user name is know. See `How to set properties on the server from the client`

    public class Chat : XSocketController
    {
        public string UserName {get;set;}
        public void ChatMessage(ChatModel message)
        {
            message.UserName = this.UserName;
            this.InvokeToAll(message,"chatmessage");
        }
    }

####How to call a server side method and wait for the result

**Client**

    conn.controller('stockticker').invoke('getstocks').then(function(data){
        console.log(data);
    });

**Server**

    public IEnumerable<Stock> GetStocks()
    {
        return _stockService.GetAll();
    }

###PUB/SUB
####How to define subscription methods on the client that the server can publish to

    conn.controller('chat').subscribe('chatmessage', function(data) {
        console.log(data);
    });
        
#####How to subscribe one time
If you only want to get a message once and then unsubscribe automatically you can use `one`

    conn.controller('chat').one('chatmessage', function(data){
        console.log(data);
    });
    
This will make sure that the client unsubscribe to the topic after getting the first message.

#####How to subscribe n times
If you want to get a message 1 to n times and then unsubscribe automatically you can use `many`

    conn.controller('chat').many('chatmessage', 3 function(data){
        console.log(data);
    });

This will unsubscribe the `chatmessage` topic after getting 3 messages.    

#### How to know when the subscription is ready
When you compare PUB/SUB with RPC an obvious disadvantage with PUB/SUB is that you do not know if the server has bound the subscription when you do a publish. Therefor you can pass in an additional function to get a callback from the server when the subscription is completed.

This works similar for all subscribe methods.

    subscribe(topic, fn, cb);
    one(topic, fn, cb);
    many(topic, n, fn, cb);

where `fn` is called when a publish occurs and `cb` is the callback that confirms the subscription.

####How to publish to server methods from the client

**Client**

    conn.controller("chat").publish("chatmessage",{Text= "Hello people!"});

**Server**

    //The server migth publish the message back to all clients subscribing
    public void ChatMessage(ChatModel chatModel)
    {
        this.PublishToAll(chatModel, "chatmessage");
    }

###How to handle connection lifetime events
The events on connection level provide information about the socket being opened/closed. The controllers has their own lifetime events.

#### OnConnected

    conn.onconnected = function(){
        console.log('socket connected');
    };
                
#### OnDisconnected
    
    conn.ondisconnected = function(){
        console.log('socket disconnected');
    };
    
###How to handle controller lifetime events
The controller has nothing to do with the actual socket. These events tell you about the controllers you are using over your connection.

#### OnOpen Event

    conn.controller("chat").onopen = function(ci){
        console.log('opened',ci);
    };

#### OnClose Event
    
    conn.controller("chat").onclose = function(ci){
        console.log('closed',ci);
    };

### Open a controller
As soon as you start to communicate over a new controller the `OnOpen` event will fire. You actually do not need to specify the controller in the connection. As long as the controller exists on the server the `OnOpen` event will fire.

### Close a controller
To close as controller (not the actual connection/socket) you just call the `Close` method on the controller instance. This will fire the `OnClose` event on the controller.

    conn.controller("chat").close();
    
###How to handle errors

    conn.controller("chat").onerror = function(err){
        console.log(err);
    };
    
### Using In-Memory Storage
This feature lets you store any serializable object on the server by using simple methods on the client API's. This is useful when you need to store data between connections/pages for the client. You are responsible for cleaning up the memory your self since data written to the storage will remain there until you remove it or the server stops.

***Note: You can set any serializable object as value in the storage***

#### Setting data on the server
Set the generic type that you want to store (in this case string)

    conn.controller('chat').storageSet('color','red');
    
#### Getting data from the server

    conn.controller('chat').storageGet('color', function(data){console.log(data)});

#### Removing data on the server

    conn.controller('chat').storageRemove('color');

#### Clear the storage
    
    conn.controller('chat').storageClear();
    
----------
## .NET MicroFramework Client API Guide
The .NET MicroFramework has support for .NET MicroFramework 4.2 and 4.3

###Client Setup
To get the client just get the latest package from http://nuget.org/packages/xsockets.client

The C# clients ALWAYS talk full-duplex/bi-directional communication, and just like the XSockets server this behavior has nothing to do with what OS or WebServer you are running.

###How to establish a connection
Just like in the JavaScript client you can multiplex over several `Controller` on one connection. To get a connections just create an instance of the XSocketClient.

    var conn = new XSocketClient("192.168.0.106", 4502);
    conn.OnOpen += (sender, args) =>
            {
                //Connection open
            };
    conn.Open();
    
- The first parameter is the IP of the server
- The second parameter is the port of the server

Do note that the call to `Open` is asynchron and you will have to wait for the OnOpen event to fire before you communicate over the connection.

###How to send messages
Sending data is easy, just use the `Publish` method and pass in the `topic`, the `object` and the name of the `controller`

    conn.Publish("chatmessage","Hello from NetDuino", "Chat");
    
The code sample above would invoke the method "Message" on the controller "Chat" with the object/data "Hello from NetDuino".

###How to receive messages
When ever data is received on the NETMF client the OnMessage event will be invoked with an `IMessage` as parameter.

The client will receive data if there is a `subscription` for the specific topic, or if the server is using RPC to target clients based on `state`. 

    public static void Main()
    {
        var conn = new XSocketClient("192.168.0.106", 4502);
        conn.OnOpen += (sender, args) =>
        {
                //Connection open
        };
        conn.OnMessage += ConnOnMessage;
        conn.Open();
        Thread.Sleep(Timeout.Infinite);
    }

    private static void ConnOnMessage(object sender, IMessage message)
    {            
        //TODO: Add logic to handle the IMessage arrived
    }

###How to parse the message into a strongly typed object
When ever data is received on the NETMF client the OnMessage event will be invoked with an `IMessage` as parameter. 
The property "D" on the message will contain the JSON representation of the data being sent. 
"C" will contain the controller name and "T" will be the topic...
So if we want to have some logic and parse the JSON into something useful you can do like this

    private static void ConnOnMessage(object sender, IMessage message)
    {            
        switch (message.C)
        {
            case "chat":
                switch (message.T)
                {
                    case "chatmessage":
                        //A chatmessage was received form Chat controller, convert JSON into the ChatMessage object
                        var chatmessage = (ChatMessage)conn.Parse(message.D, typeof (ChatMessage));
                        //Do something clever :)
                        Debug.Print(chatmessage.Text);
                        break;
                }
                break;
            default:
                //Unknown controller
                break;
        }
    }

###How to handle connection lifetime events
The events on connection level provide information about the socket being opened/closed.

#### OnOpen

    conn.OnOpen += (sender, args) =>
    {
        //Connection open
    };
                
#### OnClose
    
    conn.OnClose += (sender, args) =>
    {
        //Connection closed
    };

###How to handle errors

    conn.OnError += (sender, args) =>
        {                
            //Error
        };
    
###PUB/SUB
####How to subscribe to a topic
Just pass in the `topic` and the `controller` to notify the server about the subscription.
Do note that the client will receive data even if there is no subscription if the server-side logic uses RPC and he client is matching the requirements to where you send data.

    conn.Subscribe("chatmessage","chat");
        
####How to remove a subscription
When you no longer want to subscribe to a `topic` you just use the `unsubscribe` method to tell the server to remove the subscription.

    conn.Unsubscribe("chatmessage", "chat");

####How to publish to server methods from the client

**Client**

    conn.Publish("chatmessage",new ChatModel{Text= "Hello people!","chat");

**Server**

    //The server migth publish the message back to all clients subscribing
    public void ChatMessage(ChatModel chatModel)
    {
        this.PublishToAll(chatModel, "chatmessage");
    }
    
###How to set properties on the server from the client
Since we have state on all controllers in the connection and can take advantage of that we can store information server side and not send trivial things like a user-name each time we want to communicate.

**Server**

A simple model for a chat

    public class ChatModel
    {
        public string UserName {get;set;}
        public string Text {get;set;}
    }

A controller where we use state to only send in text since the user-name is known.

    public class Chat : XSocketController
    {
        public string UserName {get;set;}
        public void ChatMessage(ChatModel message)
        {
            message.UserName = this.UserName;
            this.InvokeToAll(message,"chatmessage");
        }
    }

**Client**

    conn.SetProperty("username","David");

Note: If the property on the controller is an `Enum` use the `SetEnum` method instead

----------
##Logging

As of 4.0 XSockets will use http://serilog.net as the default logger. Since it is a plugin you can of course replace SeriLog with something else if you want to.

By default the logger will log everything in `Debug` and only `Fatal` level in `Release`.

It is very easy to set your custom SeriLogger/Configuration/Sink. Just implement the `IDefaultLogger` interface and set the configuraiton of choice. For more information about SerilLog see http://serilog.net

    //
    // Sample below will write to file and console
    //
    [Export(typeof(IDefaultLogger))]
    public class MyLogger : IDefaultLogger
    {       
        public ILogger Logger
        {
            get
            {
                return new LoggerConfiguration()
                    .WriteTo.ColoredConsole()
                    .WriteTo.File("log.txt")
                    .CreateLogger();
            }
        }
    }

----------

##Configuration
When self-hosted the XSockets server will start with two endpoints loopback & machine IP (both on port 4502). For example my current machine would start the server at `ws://127.0.0.1:4502` and `ws:192.168.1.6:4502`

### Custom configuration
The namespace for configuration is `XSockets.Core.Configuration`

There are two ways of creating custom configuration. 

#### Pass in configurations/settings to the StartServers method
Create one or more configuration classes that XSockets will implement at startup
Passing configuration as a parametertop

Just create the configurations needed and pass them to StartServers

    //List of IConfigurationSettings
    var myCustomConfigs = new List<IConfigurationSetting>();
    //Add one configuration
    myCustomConfigs.Add(new ConfigurationSetting("ws://192.74.38.15:4502"));  
    using (var server = Composable.GetExport<IXSocketServerContainer>())
    {
        server.StartServers(configurationSettings:myCustomConfigs);
        Console.WriteLine("Started, hit enter to quit");
        Console.ReadLine();
        server.StopServers();
    }

Note: you can of course pass in several configuration.

#### Configuration as a plugin

Just inherit the `XSockets.Core.Configuration.ConfigurationSetting` class and implement your configuration. XSockets will find and and use these custom configurations.

    public class MyTestConfig : ConfigurationSetting
    {
        public MyTestConfig() : base("ws://195.74.38.15:4502")
        {
        }
    }
    
Now there is no need to pass in anything to the StartServers method, it will find the configuration above. When you use this technique the server will not create the default configuration. If you want to have for example 127.0.0.1:4502 as a configuration you have to add that as a plugin as well.

    container.Start();
    
### What can I configure

The `Start` method has a number of options, and then the `ConfigurationSetting` class itself has a number of options.

#### Method signature of Start

    void Start(bool useLoopback = true, bool withInterceptors = true, IList<IConfigurationSetting> configurationSettings = null);
    
*Note: Interceptors are by default enabled!

#### DNS Configuration
One of the most common questions about configuration is how to enable DNS configuration.

##### Public Endpoint
Let's say that you want to connect (client <-> server) to ws://chucknorris.com:4502 the configuration for that could look like

    public class ChuckNorrisConfig : ConfigurationSetting
    {
        public ChuckNorrisConfig() : base(new Uri("ws://chucknorris.com:4502")) { }
    }
    
**Note: This setup will create an endpoint based on the DNS provided. Note that port 4502 have to be open.

##### Public & Private Endpoint
Let's say that you want to connect (client <-> firewall <-> server) to ws://chucknorris.com:4502, but the public endpoint is represented by a firewall. Your firewall will then forward the connection to our servers private IP address (for example 192.168.1.7).

    public class ChuckNorrisConfig : ConfigurationSetting
    {
        public ChuckNorrisConfig() : base(new Uri("ws://chucknorris.com:4502"), new Uri("ws://192.168.1.7:4510")) { }
    }

**Note: This setup requires that you forward traffic in your firewall to `192.168.1.7:4510`

##### SSL/TLS
To get WSS you have to set the endpoint to be ´wss´ instead of ws, and you will also specify you certificate. This can either be done by setting `CertificateLocation` and `CertificateSubjectDistinguishedName` (as in the sample) or load the certificate from disk.

######Sample 1 - Certificate from store

    public class ChuckNorrisConfig : ConfigurationSetting
    {
        public ChuckNorrisConfig() : base(new Uri("wss://chucknorris.com:4502"))
        {
            this.CertificateLocation = StoreLocation.LocalMachine;
            this.CertificateSubjectDistinguishedName = "cn=chucknorris.com";
        }
    }
    
######Sample 2 - X509Certificate2

    public class ChuckNorrisConfig : ConfigurationSetting
    {
        public ChuckNorrisConfig() : base(new Uri("wss://chucknorris.com:4502"))
        {
            this.Certificate = new X509Certificate2("file.name", "password");
        }
    }
    
----------
##Hosting
###OWIN
**Note: Requires .NET 4.5+**
To host XSockets in OWIN is easy and also let you access the HttpContext and the authenticated user (if any).

**Install**
`PM> Install-Package XSockets.Owin.Host`

Register XSockets in Owin IAppBuilder

    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            app.UseXSockets(true);
        }
    }
    
###Windows Service
Use the XSockets.Windows.Service package from chocolatey to install XSockets as a windows service. You can then add you custom configurations, controller, interceptors etc into the location of the service and restart it to make it find you custom code.

Note that by default you will get the `Generic` controller as well as the `WebRTC` controller named "Broker" installed with the service.


**Install:**

 1. If not done install chocolatey (http://chocolatey.org/)
 2. Open up the Command Prompt and type cinst XSockets.Windows.Service
    
###Console Application
Create a new ConsoleApplication and install the package XSockets. This will output some sample code for getting the server started.

**Install:**
`PM> Install-package XSockets`

    using (var container = XSockets.Plugin.Framework.Composable.GetExport<IXSocketServerContainer>())
    {
        container.Start();
        foreach(var server in container.Servers)
        {
            Console.WriteLine(server.ConfigurationSetting.Endpoint);
        }
        Console.WriteLine("Server started, hit enter to quit");
        Console.ReadLine();
    }
    
###Azure
You will have to have the Azure SDK installed.

 1. Create a Windows Azure Worker Role
 2. Open the Package Manager Console
 3. Install the XSockets.Server using
    
        PM -> Install-Package XSockets.Server

 4. Open the Property Page for the Worker Role. 
    You will find it in the /Roles/ foler of your WorkerRole Project.
 5. Add a TCP endpoint using the Endpoints Tab.
    - Name the Endpoint i.e 'MyEndoint'
    - Set the type to input
    - Set the protocol, to TCP
    - Define the Public & Private port that you want to use.

 6. Add a configuration setting using the Settings tab, we need to define the EndPoint Uri here.
    - Set a name on the setting i.e 'MyUri'
    - Set the type to string
    - Set the value to i.e `ws://127.0.0.1:4510` (this depends on your enviorment).

 7. Add the following code the the OnStart() method of your WorkerRole (WorkerRole.cs)

        var container = XSockets.Plugin.Framework.Composable.GetExport<IXSocketServerContainer>();
        // Create a Custom Configuration based on what we have defined using property pages.
        var myCustomConfig = new List<IConfigurationSetting>();
        var config = new ConfigurationSetting(new Uri(RoleEnvironment.GetConfigurationSettingValue("MyUri")))
        {
            Endpoint = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints.FirstOrDefault(n => n.Key.Equals("MyEndpoint"))
                .Value.IPEndpoint
        };
        myCustomConfig.Add(config);
        container.Start(configurationSetting: myCustomConfig);

 8. Compile and run

Now, Try connect to the Generic Controller using the following piece of JavaScript Code

    var ws = new XSockets.WebSocket('ws://127.0.0.1:4510',['generic']);
    
###Amazon

    TODO

----------
## Fallback
To use the fallback in XSockets you have to have OWIN, .NET 4.5 and WebAPI.

    TODO
    
----------

##Performance & Scaling
XSockets has a built-in scaling module for enterprise solutions. This module scales with sockets instead of a slow backplane like SQL-Server. You can scale in any direction you want, so if you only want to send data in one direction (from one server to another) that´s fine. However the most common scenario is to scale both ways so that all clients gets data regardless of what server they are connected to.
 
### Scaleout
In the `Enterprise` version you can scaleout XSockets over `n` servers. By default XSockets offers scaleout over sockets, this means that there will be no bottle neck like in other solutions that scaleout over MSSQL etc.

You can add several servers to the scaleout, this is done by requesting the `IXSocketsScaleOut` plugin from the `plugin framework` and then just add a server using the method `AddScaleOut`.

Note that this will scale in one direction from this server to `ws://127.0.0.1:4503`. So to get scaling both ways you just add this servers location to the server on `4503`

    Composable.GetExport<IXSocketsScaleOut>().AddScaleOut("ws://127.0.0.1:4503");
    
#### Implementing Custom ScaleOut
If you rather scaleout over SQL, Redis etc you can write a custom scaleout by just deriving the ´BaseScaleout´ class

    public class MyScaleOut : BaseScaleOut
    {
        /// <summary>
        /// Will be called by the BaseScaleout Ctor
        /// </summary>
        public override void Init()
        {
            //Do stuff that may be needed at startup for your custom scaleout
        }  
        /// <summary>
        /// Will be called when a message arrives at the server
        /// </summary>
        /// <param name="message"></param>
        public override void Publish(IMessage message)
        {
            //Save message to your selected scaleout, for example SQL-server
        }
        /// <summary>
        /// Will be called by the BaseScaleout Ctor after the call to Init.        
        /// </summary>
        public override void Subscribe()
        {
            //Poll every 10 sec            
            var t = new Timer(10000);
            t.Elapsed += (sender, args) =>
            {
                //1: Implement a mechannism that will get data from a storage of some kind
                IList<IMessage> messages = null;//get messages from service
                foreach (var message in messages)
                {
                    //2: Publish to the interceptors
                    if (this.Container.WithInterceptors)
                        MessageInterceptorsQueue.Push(new MessageInfo { Message = message, MessageDirection = MessageDirection.In });
                    //3: send into the server by using the pipeline and the correct plugin
                    Pipeline.OnIncomingMessage(this.Factory.Plugins.Single(p => p.Alias == message.Controller), message);
                }
            };
            t.Start();
        }      
    }
    
###Loadbalacing

    TODO

### Performance Counters

    TODO

----------

##FAQ

    TODO
    
----------

##Security	
XSockets.NET does not provide any features for authenticating users. Instead, you integrate the XSockets:NET features into the existing authentication structure for an application. You authenticate users as you would normally in your application, and work with the results of the authentication in your XSockets.NET code. For example, you might authenticate your users with ASP.NET forms authentication, and then in your `Controller`, enforce which users or roles are authorized to call a method.

XSockets provides the Authorize attribute to specify which users have access to a `controller` or `method`. You apply the Authorize attribute to either a `controller` or particular `methods` in a `controller`. Without the Authorize attribute, all public methods on the controller are available to a client that is connected to the controller. 

If you want to allow unrestrcited access on some methods you can add the `AllowAnonymous` attribute the these methods.

More information about attributes and methods below

### Authorize Attribute

This attribute can be set on Controller or Method level. If set at controller level all action methods that do not have the AllowAnonymous attribute will require authentication.

The authorize attribute can take Roles and Users but if using that you will have to implement your own authentication by overriding OnAuthorization(AuthorizeAttribute authorizeAttribute)

### AllowAnonymous Attribute

This attribute can be set on action methods and will then allow anonymous access.

### Get FormsAuthentication Ticket

When you have custom authentication you can get the FormsAuthenticationTicket from this method.

    var ticket = GetFormsAuthenticationTicket();

**Note: If you do not pass in a cookiename .ASPXAUTH will be used.**

**Important: If you have separate project you will have to use the same origin to be able to get cookies and also use machine-key in the config to be able to get the AuthCookie.**
***See*** http://msdn.microsoft.com/en-us/library/system.web.configuration.machinekeysection.compatibilitymode%28v=vs.110%29.aspx ***if you are using different framework versions in the projects.***

### Write a custom AuthenticationPipeline
When the socket is connected and the handshake is completed the `AuthenticationPipeline` will be called. By default the pipeline will look for a FormsAuthenticationTicket, but you can override this pipline by just implementing a interface `XSockets.Core.Common.Socket.IXSocketAuthenticationPipeline`

There can only be one pipeline so even if you implement several pipelines only one wil be used.

    [Export(typeof(IXSocketAuthenticationPipeline))]
    public class MyAuthenticationPipeline : IXSocketAuthenticationPipeline
    {
        public IPrincipal GetPrincipal(IXSocketProtocol protocol)
        {            
            if (protocol.ConnectionContext.User == null)
            {
                //creating a fake super user ;)
                var roles = new string[]{"superman","hulk"};
                var userIdentity = new GenericIdentity("David");
                protocol.ConnectionContext.User = new GenericPrincipal(userIdentity, roles);
            }            
            return protocol.ConnectionContext.User;
        }
    }

### How to override the OnAuthorization method
The `OnAuthorization` method is called for every method on a controller that has authentication. The attribute for the method (or controller) is passed in and we check that 

 1. That the user is authenticated
 2. If the user name is required we verify a match
 3. If user name was not required or did not match we check roles

        public virtual bool OnAuthorization(IAuthorizeAttribute authorizeAttribute)
        {
            try
            {
                var user = this.ProtocolInstance.ConnectionContext.User;
                if (user.Identity.IsAuthenticated)
                {
                    if (!string.IsNullOrEmpty(authorizeAttribute.Roles) || !string.IsNullOrEmpty(authorizeAttribute.Users))
                    {                    
                        if (authorizeAttribute.Users.Split(',').Contains(user.Identity.Name)){
                            return true;
                        }
                        return authorizeAttribute.Roles.Split(',').Any(user.IsInRole);      
                    }
                    return true;
                }
                return false;
            }
            catch
            {
                return false;
            }
        }

**Example**
Based on the `Write a custom AuthenticationPipeline` where we added the username `Hero` and the roles `hulk`, `superman` we have the following scenario

    [Authorize()] //would be valid since we have a authorized `fake` user
    [Authorize(Users = "David")] //would be valid
    [Authorize(Roles = "batman,robin")] //would be invalid
    [Authorize(Roles = "batman,hulk")] //would be valid
    
### How to know when authorization fails
Every time a method on a controller needs authorization the `OnAuthorization` method is called (which was overridden in previous sample).

When the `OnAuthorization` returns false the `OnAuthorizationFailed` event is fired.

    //Constructor
    public Chat()
    {
        this.OnAuthorizationFailed += Chat_OnAuthorizationFailed;
    }
    void Chat_OnAuthorizationFailed(object sender, OnAuthorizationFailedArgs e)
    {
        Console.WriteLine("Auth Failed: {0},{1}", e.Controller, e.MethodName); 
    }

The `OnAuthorizationFailedArgs` contains information about the controller and the method being called. Information about the user that called the method can be accessed from `this.ProtocolInstance.ConnectionContext.User`

----------

## The Plugin Framework
The plugin framework is inspired by MEF (Managed Extensibility Framework). MEF is awesome, but we wrote our own plugin framework to be able to run everywhere and also to avoid any dependencies. We did not copy stuff from MEF that we did not need and we added some extra features the we thought would be nice to have.

### Quick Start
A very basic example based on a MEF sample that you can find at http://www.amazedsaint.com/2010/06/mef-or-managed-extensibility-framework.html

First of all... Open up the `Package Manager Console` (Tools->Library Package Manager->Package Manager Console) below called PMC. Install by typing `Install-Package XSockets.Plugin.Framework` into the PMC and hit enter.

The plugin framework has `no dependencies` and can be used in any project without using the rest of XSockets.NET

#### Introduction
Just like in MEF it is all about `Exports` and `Imports`. In `XSockets.Plugin.Framework` you can choose to export on interface level which means that you only have to implement the interface to get `Exports` and `Imports` to work. You actually dont event need the `Exports/Imports` since the framework allows you to register exported types at runtime, but the default scenario is to use the attributes.

#### Define Interfaces
So lets build a zoo (just like the example in the link above).

First of all we need a `IAnimal` interface for our animals. We also decorate the class with the `Export` attribute

    [Export(typeof(IAnimal))] 
    public interface IAnimal
    {
        void Eat();
    }

We also need A `IZoo` interface, it has an `IEnumerable of IAnimal`. Just like the IAnimal interface the `IZoo` has the `Export` attribute. Also note that we use the `ÌmportMany` attribute to get all `Exports` of `IAnimal`

    [Export(typeof(IZoo))]
    public interface IZoo
    {
        [ImportMany(typeof(IAnimal))]
        IEnumerable<IAnimal> Animals { get; set; }
    }

#### Classes Implementing Interfaces
First we create `Lion` and `Rabbit` classes and implement the `IAnimal` interface so that the classes become modules/plugins

Note that we do not have to specify anything on the classes since the interfaces has the `Exports` and `Imports`

    public class Lion : IAnimal
    {
        public Lion()
        {
            Console.WriteLine("Grr.. Lion got created");
        } 
        public void Eat()
        {
            Console.WriteLine("Grr.. Lion eating meat");
        }
    }
    public class Rabbit : IAnimal
    {
        public Rabbit()
        {
            Console.WriteLine("Meeep.. Rabbit got created");
        } 
        public void Eat()
        {
            Console.WriteLine("Meeep.. Rabbit eating carrot");
        }
    }

Now we just need a `Zoo` to keep the `Animals` in.

    public class Zoo : IZoo
    {
        public IEnumerable<IAnimal> Animals { get; set; }
    } 


####Using the Plugin Framework

Note that the the interfaces decides how the classes should behave regarding Exports/Imports

    class Program
    {
        static void Main(string[] args)
        {
            //Get a single instance of Zoo... We only expect one zoo to exist
            var zoo = Composable.GetExport<IZoo>();
            //List the animals found in the zoo
            foreach (var animal in zoo.Animals)
                animal.Eat();
            Console.ReadLine();
        }
    }

The output from the program would be...

    Grr.. Lion got created
    Meeep.. Rabbit got created
    Grr.. Lion eating meat
    Meeep.. Rabbit eating carrot

###Only load specific assemblies

If you for some reason do not want to load all assemblies/executables when the plugin framework starts you can tell the framework what to load.

The sample below will  load only assemblies starting with XSockets.* and also a specific assembly named "SomeAssembly.dll"

    Composable.ClearPluginFilters();
    Composable.AddPluginFilter("XSockets.*.dll");
    Composable.AddPluginFilter("SomeAssembly.dll");

### Handling Exceptions
You can use the `Composable.AddErrorAction` to get information about any exceptions taht occurs inside of the plugin framework. If you add several `actions` they will all be called when/if an exception is thrown.

    Composable.AddErrorAction(ex => Console.WriteLine("First ErrorAction: {0}", ex.Message));
    Composable.AddErrorAction(ex => Console.WriteLine("Second ErrorAction: {0}", ex.Message));

### Understanding the concept & add custom plugins
The concept of the plugin framework is all about exporting and importing interfaces. We believe that all you need at compile time is the knowledge of the interface. The framework should load all exported and imported types at startup (runtime).

By having that kind of architecture we get very decoupled systems. We can replace current functionality (or add new functionality) by just implementing an interface. We can even do so at runtime if we want to.

#### Config
By default the plugin framework will search for plugins at the location where the executable (or eqvivalent) is running. The plugin will be loaded at first usage of the plugin framework, but you can add assemblies (or paths to search) at runtime.

The framework will by defualt load assemblies (*.dll) and executables (*.exe) but you can also add your own filters to the configuration.

#### Exports & Imports
By setting the Export attribute on a interface or class you tell the framework that you want to export this. And not surprising you import stuff by using ImportOne or ImportMany.

#### Attributes
To explain the concept easily we will create a few simple classes and decorate them.

##### ImportOne

    TODO:

##### ImportMany

    TODO:

##### ImportingConstructor

    TODO:

##### Export
By adding the Export attribute to an interface (or a class) the plugin framework will detect and use the instance as a module/plugin. You always have to provide the type being exported when using the Export attribute, but there other setting as well

###### InstancePolicy

Sets the lifecycle for the export. Using InstancePolicy.Shared will create a singleton of the export and only one instance will live in the app-domain. The default lifecycle is InstancePolicy.NewInstance and you will not need to set this option.

###### Rewritable

By default the Rewritable option is set to Rewritable.No, but if you set it to Rewritable.Yes your module/plugin can be overridden if someone implements the interface on another class. That framework will then throw your version away and use the new module/plugin.

##### MetaDataExport

    TODO:

#### Adding functionality at runtime (re-compose)
Sometimes you might want to add both new interfaces and plugins at runtime, below is a simple example of howto do so.

Lets say that we have an interface IAnimal located in a separate assembly (project)

    /// <summary>
    /// This interface is not exported and 
    /// will not be available in the XSockets.Plugin.Framework (by default)
    /// 
    /// All interfaces that you are planning on using have to be known a compile time
    /// </summary>
    public interface IAnimal
    {
        void Says();
    }
    
In another assembly (not referenced) we have a class Lion that implements the IAnimal interface

    /// <summary>
    /// This class does not have to be know at compile time
    /// It might be added later and added to the XSockets.Plugin.Framework
    /// It can be used since it has an interface!
    /// </summary>
    public class Lion : IAnimal
    {
        public void Says() { Console.WriteLine("Grrr"); }
    }

In our XSockets solution we only reference the project/assembly with the interface IAnimal. This interface is not exported by default, so we tell the plugin framework (at runtime) to also use IAnimal as an export.

That is done by

    Composable.RegisterExport<IAnimal>();

But the Lion class is unknown so we add the assembly with Lion

    Composable.LoadAssembly(@"C:\temp\someassembly.dll");

Then we tell the plugin framework to "recompose" = satisfy all imports/exports

    Composable.ReCompose();

We can now get all instances of IAnimal using GetExport or GetExports...

##### Two different approaches listed below!

**Approach 1**

Here the classes implementing IAnimal is known in the current solution so we do not have to load any assemblies.

    //Lets make the plugin framework aware of IAnimal
    Composable.RegisterExport<IAnimal>();
    Composable.ReCompose();
    foreach (var animal in Composable.GetExports<IAnimal>())
    {
        animal.Says();
    }
    
**Approach 2**

Here there are implementations of IAnimal in a assembly not yet loaded.

    Composable.RegisterExport<IAnimal>();
    Composable.LoadAssembly(@"C:\temp\PluginDemo.dll");
    //OR... load many assemblies from a folder and possibly subfolders
    //Composable.AddLocation(@"c:\temp\", SearchOption.AllDirectories);
    Composable.ReCompose();
    foreach (var animal in Composable.GetExports<IAnimal>())
    {
        animal.Says();
    }

----------
## Packages
Describes all available packages on http://nuget.org and http://chocolatey.org
###Chocolatey Packages
####XSockets.Windows.Service

You can install XSockets as a windows service from chocolatey, see http://chocolatey.org/packages/XSockets.Windows.Service

###Nuget Packages
####XSockets

This package will behave in different ways depending on where you install it. If you do not know what to do, use this package.

This package will install ItemTemplates for XSockets.NET 4 into Visual Studio.

 - XSockets.Server
 - XSockets.Core
 - XSockets.Plugin.Framework
 - XSockets.Fallback (if installed into a web-application with .NET 4.5+)

####XSockets.Core

This package contains the functionality to write custom real-time controllers. Install this package into a class-library if you want to develop custom server-side functionality

Dependencies:

- XSockets.Plugin.Framework

####XSockets.OWIN.Host

This package contains the functionality to host XSockets inside of OWIN.

Dependencies: 

- XSockets.Server

####XSockets.Plugin.Framework

A package for building modular applications in a blink.

Dependencies:

- none

####XSockets.Fallback

A fallback from websocket to longpolling via WebAPI if installed into .NET 4.5+

Dependencies:

- none

####XSockets.Client

Provides a socket client for connecting to a XSockets.NET server.
Supported platforms are:

 - Mono
 - .NET 4.0+
 - iOS (MonoTouch)
 - Android (MonoDroid)
 - NETMF 4.2, 4.3

Dependencies: 

- XSockets.Core

####XSockets.JsApi

Provides a JavaScript API for communication with text/binary messages to a XSockets server with a publish/subscribe pattern

Dependencies:

- none

####XSockets.Sample.Stockticker

A sample project copied from the SignalR stockticker but rewritten for XSockets so that you can compare it easier with SignalR.

Dependencies:

- XSockets

####XSockets.Sample.WebRTC

A sample project that will provide a sample of a multi-video chat within the browser. Source code found at: https://github.com/XSockets/WebRTC

Dependencies:

- XSockets

----------