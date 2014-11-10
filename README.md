#XSockets.NET 4 - Documentation

This section covers server and client API´s for XSockets.NET

##Introduction

Start here if you are new to XSockets.NET

###What is XSockets.NET?
XSockets.NET is a real-time messaging system that allows communication between any device that has TCP/IP. The server can be hosted anywhere (.NET/Mono) and the clients cover every major browser + C#, VB.NET, Android, iOS, NETMF. And it is very easy to connect anything else that has a TCP/IP stack since XSockets allows custom protocols.

***In short terms: RealTime, InternetOfThings and WebRTC in a single framework!***

###Who uses it?

    TODO: Add images here

----------

##Getting started with real-time
###1. Start a server

    using XSockets.Core.Common.Socket;
    using XSockets.Plugin.Framework;
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
    conn.Controller("generic").On<dynamic>("mymessage", data => Console.WriteLine(data.Text));

### 3. Send message

JavaScript

    conn.controller('generic').invoke('mymessage',{Text:'Hello JS RealTime'});
    
C#

    conn.Controller("generic").Invoke("mymessage",new {Text = "Hello C# RealTime"});

----------

### What's next?
#### Client API
Learn to...

 - Use Pub/Sub, RPC or both!

#### Server API
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
        container.Start();
        Console.WriteLine("Started, hit enter to quit");
        Console.ReadLine();
    }

###Install into OWIN (self hosted)
**Note: This requires .NET 4.5+**
Open up the Package Manager Console and install the server

`PM> Install-Package Microsoft.Owin.SelfHost` **(include `-pre` to get v 4.0)**

`PM> Install-Package XSockets.Owin.Host` **(include `-pre` to get v 4.0)**

####How to register XSockets Middleware
UseXSockets is an extension method for the OwinExtensions class.

    using XSockets.Owin.Host;
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

###Install into OWIN (IIS)
**Note: This requires .NET 4.5+**
Open up the Package Manager Console and install the server

`PM> Install-Package XSockets.Owin.Host` **(include `-pre` to get v 4.0)**

####How to register XSockets Middleware
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

Since XSockets.NET is built to run on both .NET and Mono the server can be hosted pretty much anywhere. The server side only has .NET (or Mono equivalent) as requirements!

###Client system requirements

XSockets can be used from any client that has TCP/IP, and since XSockets.NET supports cross-protocol communication it is very easy to add new clients and protocols.

The XSockets.NET team has built a few client libraries to make life easier for our developers. All client libraries is using full-duplex/bi-directional communication.

    Language    Requirements
    C#          .NET 3.5, 4.0+
    iOS         .NET 4.0+ (MonoTouch)
    Android     .NET 4.0+ (MonoDroid)
    NETMF       4.2, 4.3
    JavaScript  Browser with websockets (there is a fallback for IE 9 & IE 8)

##Architecture - How it works

The architecture for XSockets.NET is simple yet powerful. Each client connects to a protocol, the protocol will then allow communication over n controllers. So you can multiplex over several controller on one connection. This architecture enables communication cross-protocol as well as cross-controller.

Different clients have different capabilities, browsers for example have the RFC6455 (websockets) protocol implemented, but other things devices might talk a protocol that you have created. Since XSockets allows “cross-protocol communication” all connected clients can communicate with each other very easily.

----------

###Basics
![Architecture](http://xsockets.net/$2/file/xsocketscommunication-1.png)

The red clients are clients libraries written by XSockets.NET and the blue clients are examples of what we have easily connected with custom protocols.

You may also notice that XSockets enables not only cross-protocol communication, but also cross-controller communication so that you can call a method on another controller. Or even send data to client on another controller with a single line of code.

----------

##Controllers
If you are familiar with the MVC pattern you will find it very easy to work with XSockets controllers.

###How to create and use Controller classes
To create a `Controller`, create a class that derives from `XSockets.Core.XSocket.XSocketController`. The following example shows a simple `Controller` class for a chat application.

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
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

    //using XSockets.Core.XSocket;
    //using XSockets.Plugin.Framework.Attributes;
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

    //using System.Collections.Generic;
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
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

    conn.Controller("chat").On<dynamic>("chatmessage", data => Console.WriteLine(data.Text));

*Above we get a dynamic, but we can of course use any type to get the message deserialized into the correct type and get intellisense. Which is prefered*

####How to hide methods and properties
You might not wanna expose all publish methods and properties to the client API's. When you want to hide a publish method/property just decorate the method/property with the `[NoEvent]` attribute. The attribute is located under `XSockets.Core.Common.Socket.Event.Attributes`

###How to handle errors in the Controller class
Wrap you logic in a try catch block and call the HandleError method that will invoke the OnError event.

When needed you can also send the error to the ErrorInterceptors if you have implemented any.

    //using XSockets.Core.Utility.MessageQueue.Interceptors;
    //using XSockets.Core.XSocket;
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
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    //using XSockets.Plugin.Framework.Attributes;
    
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

##Parameters, Model binding & return values
XSockets have had strongly typed model binding for generations, this is one of the things that makes it easy to use. Just declare methods on the controllers and specify the complex parameter on the methods. XSockets will transform the JSON sent in to the correct parameter type.

You can return any serializable object from the action methods. The value returned will end up at the client that initiated the call.
As of 4.0 you can also return Task and Task&lt;T&gt; from the action methods and the client will get the data back when the Task is completed.

###How to use complex objects as parameters
Above we looked at how you can send complex types to the clients, of course the client can send complex types to the server as well.

**Server**

A simple model for our chat sample

    public class ChatModel
    {
        public string UserName {get;set;}
        public string Text {get;set;}
    }

Server code that accepts a complex object and sends it to all clients. Since we have the username (showed in `How to use state on the server`) we never pass that in, we only set it before sending data out.

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
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

###How to use IMessage as parameter
When you do not process the data server-side there is no need serializing incoming messages. In these situations you can use `IMessage` as parameter since any message sent into XSockets.NET will be transformed into `IMessage` internally.

This is how the `Generic` controller used in the `Getting started with real-time communication` sample is built. The code for `Generic` is very simple and looks exactly like this.

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    public class Generic : XSocketController
    {
        public override OnMessage(IMessage message)
        {
            this.InvokeToAll(message);
        }
    }
    
In the `Generic` `Controller` we override the `OnMessage` method which means that any call to the `Controller` that does not match a `method` will end up in the `OnMessage` method.

You can of course use the `IMessage` parameter in regular methods on the `Controller` and not just the `OnMessage` method.

###Return data synchronously to caller
When you call the server from a client and want to wait for the result you just replace void with the type you want to return and the client API's will wait for the result.

**Server**

    public string Echo()
    {
        return "Echo";
    }
    
*Note: You can also return Task&lt;T&gt;*

You can of course send data before doing the return, for example.

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
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

##Binary data
XSockets has supported binary messages for a long time, but in 4.0 we have made it even easier than before.

###Send binary data to the server
Lets say that we have a file `c:\temp\xfile.txt` with the text `This file was sent with XSockets.NET` and we want to send that file to the server.

**Server**

    //using XSockets.Core.XSocket;
    //using XSockets.Core.Common.Socket.Event.Interface;
    public void MyFile(IMessage message)
    {
        var filecontent = Encoding.UTF8.GetString(message.Blob.ToArray());
    }

**Clients**

Client - JavaScript

    // Create a simple Array buffer and fill it with "something"
    var arrayBuffer = new ArrayBuffer(10);
    // Send the binary data to XSockets
    conn.controller("chat").invokeBinary("myfile",blob);

Client - C#

    var blob = File.ReadAllBytes(@"c:\temp\xfile.txt");
    conn.Controller("chat").Invoke("myfile", blob);

###How to pass meta-data together with binary data
If we want to attach metadata about the binary data that is easy to do. Just pass along the object representing the metadata and XSockets will let you extract that data on the server.

**Server**
    
    //simple class for holding metadata about a file
    public class FileInfo
    {
        public string Name {get;set;}
    }

    //using XSockets.Core.XSocket;
    //using XSockets.Core.Common.Socket.Event.Interface;
    public void MyFile(IMessage message)
    {
        var filecontent = Encoding.UTF8.GetString(message.Blob.ToArray());
        var metadata = message.Extract<FileInfo>();
    }

Just use `Extract<T>` to get back to metadata attached to the binary data.

**Clients**

Client - JavaScript

    // Create a simple Array buffer and fill it with "something"
    var arrayBuffer = new ArrayBuffer(10);
    // Send the binary data and metadata to XSockets
    conn.controller("chat").invokeBinary("myfile",blob,{Name:"xfile.txt"});

Client - C#

    var blob = File.ReadAllBytes(@"c:\temp\xfile.txt");
    conn.Controller("chat").Invoke("myfile", blob, new {Name="xfile.txt"});

###Send binary data to the clients
If you want to send for example and image from the server to the clients it can be done like this. The name of the controller will be "Chat" and the name (or topic) is "myimage".

**Server**

    var blob = File.ReadAllBytes(@"c:\temp\someimage.jpg");
    this.InvokeToAll(blob,"myimage");
    
**Clients**

Client - JavaScript

    conn.controller("chat").on("myimage") = function (b) {
        var uint8Array = new Uint8Array(b.binary);
        var arrayBuffer = uint8Array.buffer;
        var blob = new Blob([arrayBuffer], { type: "image/jpg" });
        var blobUrl = window.URL.createObjectURL(blob);
        $("img").attr("src", blobUrl);
    };

Client - C#

    conn.Controller("chat").On<IMessage>("myimage", message =>
    {
        var ms = new MemoryStream(message.Blob.ToArray());
        var img = Image.FromStream(ms);
        //Do something with the image...
    });
    
----------

##RPC
The RemoteProcedureCall pattern let you do exactly what is says, call procedures remotely. RPC together with XSockets fine grained control let you send messages to specific clients in a very smooth way.

You will see more about this powerful feature combined with RPC below
###How to call methods on the client
If you just want to send a message to the caller of the method, use `Invoke`

**Server**
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    //Send a message to the caller
    this.Invoke("Hello to caller from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').chatmessage = function(data){console.log(data)};

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

###How to call methods on all clients
If you want to send a message to all clients connected to the controller, use `InvokeToAll`

**Server**
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    this.InvokeToAll("Hello to all from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').chatmessage = function(data){console.log(data)};

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

###How to target a subset of the connected clients
If you want to send a message to some of the clients connected to the controller, use `InvokeTo`

This is where you will see the power of state! You will actually get intellisense so that you can write lambda expressions to target the clients you want to send the message to.

If we have the properties Age and Gender on the `Controller` (for example) we can target exactly the ones we want to.
Below for example we send to all clients having the same gender and location as the caller

**Server**
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    this.InvokeTo(p => p.Gender == this.Gender && p.City == this.City,"Hello to some from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').chatmessage = function(data){console.log(data)};

Client - C#

    conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));

###How to send messages to clients connected on another Controller class
It is pretty much the same as sending to the clients on the same `Controller` since XSockets extension-methods are generic.

So, if you are on `Controller` A and want to send to all clients on `Controller` B you just use...

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    //To all
    this.InvokeToAll<B>("Hello to all from server", "target");
    
    //And the powerful `InvokeTo<T>`  would have a signature like:
    this.InvokeTo<T>(Func<T,bool> expression, object obj, string target);

###How to call client methods outside the Controller class
Just create a controller instance (or ask the plugin framework after the specific controller) and then use the extensions to send data

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;

    //Create the instance your self
    var chat = new Chat();
    //Then just use one of the extensions to target, one, some, others or all...
    chat.InvokeToAll<Chat>("Hello from manual instance","say");
    
Notice that you have to pass in <T> in the extension regardless of what controller you want to use. So you can even do `new Chat().InvokeToAll<Stock>("Hello to stock clients from chat instance","hi");`
    
*Note: there is no point in using Invoke/Publish/Send since the actual controller does not have a socket since there is no client connected to it (we created the instance manually).*

----------

##PUB/SUB
The publish/subscribe pattern is useful when your applications need the users to subscribe to the topics that you are about to publish. Of course you can have very good control with XSockets since you can select subsets of subscribers very easy to get more fine grained control.

You will see more about this powerful feature combined with PUB/SUB below.
###How to publish data to subscribers
If you just want to send a message to the publisher/caller of the method, use `Publish`
The big difference between `Publish` and `Invoke` is that the message only will arrive at the client if the client has a `Subscription` registered on the server.

**Server**
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    //Publish a message to the caller
    this.Publish("Hello to caller from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').subscribe('chatmessage', function(data){
        console.log(data);
    });

Client - C#

    conn.Controller("chat").Subscribe<string>("chatmessage", data => Console.WriteLine(data) );

###How to publish to all subscribers
If you want to publish a message to all clients subscribing to a topic, use `PublishToAll`

**Server**
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    this.PublishToAll("Hello to all subscribers from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').subscribe('chatmessage', function(data){
        console.log(data)
    });

Client - C#

    conn.Controller("chat").Subscribe<string>("chatmessage", data => Console.WriteLine(data));

###How to publish to a subset of the subscribing clients
If you want to send a message to some of the subscribing clients, use `PublishTo`

This is where you will see the power of state! You will actually get intellisense so that you can write lambda expressions to target the subscribers you want to send the message to.

If we have the properties Age and Gender on the `Controller` (for example) we can target exactly the ones we want to.
Below for example we send to all clients having the same gender and location as the caller

**Server**
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    this.PublishTo(p => p.Gender == this.Gender && p.City == this.City,"Hello to some from server", "chatmessage");

**Client**

Client - JavaScript

    conn.controller('chat').subscribe('chatmessage', function(data){
        console.log(data)
    });

Client - C#

    conn.Controller("chat").Subscribe<string>("chatmessage", data => Console.WriteLine(data));

###How to publish messages to subscribers connected on another Controller class
It is pretty much the same as publishing to the clients on the same `Controller` since XSockets extension-methods are generic.

So, if you are on `Controller` A and want to publish to all `chatmessage` subscribers on `Controller` B you just use...

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    //To all
    this.PublishToAll<B>("Hello to all 'chatmessage' subscribers on 'controller' B from server", "chatmessage");
    
    //And the powerful `PublishTo<T>`  would have a signature like:
    this.PublishTo<T>(Func<T,bool> expression, object obj, string target);

###How to call subscribers outside the Controller class
Just create a controller instance (or ask the plugin framework after the specific controller) and then use the extensions to send data

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    //Create the instance your self
    var chat = new Chat();
    //Then just use one of the extensions to target, one, some, others or all...
    chat.PublishToAll<Chat>("Hello from manual instance","say");
    
Notice that you have to pass in <T> in the extension regardless of what controller you want to use. So you can even do `new Chat().PublishToAll<Stock>("Hello to stock clients from chat instance","hi");`

*Note: there is no point in using Invoke/Publish/Send since the actual controller does not have a socket since there is no client connected to it (we created the instance manually).*
    
----------

##State is the key to success
State is the most important thing in full-duplex real-time applications. Without state you will not know where to send messages, and you will spend a lot of more time to implement business logic in your applications

### How to use state on the server
All public getters and setters are accessible from the client API, so you can actually `get` and `set` the `Controller` properties from the client API's.

Below you can see that the chat example is extended with a property for username. Since we can use `state` to know who the user is at all times there is no need passing the unnecessary data with every message.

Server `Controller` with state

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
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


----------


##Connection lifetime events
Each controller have events that you can use to know when `Open`, `Close` and `ReOpen` occurs.

###The OnOpen event
Will fire when the controller is used over the connection for the first time.
    
    //using XSockets.Core.XSocket;
    //using XSockets.Core.Common.Socket.Event.Arguments;
    
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

###The OnClose event
Will fire when the socket is closed or if the client choose to close this specific controller on the connection.

    //using XSockets.Core.XSocket;
    //using XSockets.Core.Common.Socket.Event.Arguments;
    
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

###How to do heartbeats (ping/pong control-frames)
Control-frames (ping/pong) is primarily for checking connections, measure latency and validate that the connection is valid.

You can implement custom logic for sending/receiving these frames on the server, but there is a simple helper that you can use to get the functionality.

    //using XSockets.Core.XSocket;
    //using XSockets.Core.Common.Socket.Event.Arguments;
    
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

## Modules/Plugins
This section covers how to create custom modules/plugins for different parts of XSockets. This only covers the `Interfaces` within XSockets, but you can ofcourse add your custom interfaces as plugins by telling the plugin framework about the interfaces you want to use. Read more about custom plugin in `The Plugin Framework` section

###How to implement your custom Pipeline
The server will only have one `XSockets.Core.Common.Socket.IXSocketPipeline`, but you can override the default one by just deriving it.

Each message passing into the server or out of the server will pass the pipeline

    //using XSockets.Core.Common.Socket;
    //using XSockets.Core.XSocket;
    //using XSockets.Core.Common.Socket.Event.Interface;

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
    
### Write a custom AuthenticationPipeline
When the socket is connected and the handshake is completed the `AuthenticationPipeline` will be called. By default the pipeline will look for a FormsAuthenticationTicket, but you can override this pipline by just implementing a interface `XSockets.Core.Common.Socket.IXSocketAuthenticationPipeline`

There can only be one pipeline so even if you implement several pipelines only one wil be used.

    //using XSockets.Core.Common.Protocol;
    //using XSockets.Core.Common.Socket;
    //using XSockets.Plugin.Framework.Attributes;

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

### Interceptors Concept
There can only be one `Pipeline`, and one `AuthenticationPipeline`, but interceptors can be 0 to N. Every interceptor will be called at a specific time. ConnectionInterceptors for example will be called when someone connects, disconnects or when a handshake is completed (ok or not).

Interceptors are common when debugging or logging, but XSockets does not choose a logger for you. Implement the interface and do whatever you want inside of the interceptor(s).

#### How to write ConnectionInterceptors
Sample of a connection interceptor that logs handshake and connect/disconnect with `debug` level

    //using XSockets.Core.Common.Interceptor;
    //using XSockets.Core.Common.Protocol;
    //using XSockets.Core.Common.Utility.Logging;
    //using XSockets.Plugin.Framework;

    public class MyConnectionInterceptor : IConnectionInterceptor
    {

        public void Connected(IXSocketProtocol protocol)
        {
            Composable.GetExport<IXLogger>().Verbose("Connected {@protocol}" , protocol.ConnectionContext);            
        }

        public void Disconnected(IXSocketProtocol protocol)
        {
            Composable.GetExport<IXLogger>().Verbose("Disconnected {@protocol}", protocol.ConnectionContext);
        }

        public void HandshakeCompleted(IXSocketProtocol protocol)
        {
            Composable.GetExport<IXLogger>().Verbose("Handshake ok {@protocol}", protocol.ConnectionContext);
        }

        public void HandshakeInvalid(string rawHandshake)
        {
            Composable.GetExport<IXLogger>().Verbose("Handshake failed {raw}", rawHandshake);
        }
    }
    
#####How to write MessageInterceptors
Sample of a message interceptor that logs in/out messages with `debug` level

    //using XSockets.Core.Common.Interceptor;
    //using XSockets.Core.Common.Protocol;
    //using XSockets.Core.Common.Socket.Event.Interface;
    //using XSockets.Core.Common.Utility.Logging;
    //using XSockets.Plugin.Framework;
    
    public class MyMessageInterceptor : IMessageInterceptor
    {
        public void OnIncomingMessage(IXSocketProtocol protocol, IMessage message)
        {
            Composable.GetExport<IXLogger>().Debug("{incoming message {@message}}", message);
        }

        public void OnOutgoingMessage(IXSocketProtocol protocol, IMessage message)
        {
            Composable.GetExport<IXLogger>().Debug("{outgoing message {@message}}", message);
        }
    }
    
#####How to write ErrorInterceptors
Sample of a error interceptor that logs exceptions with `error` level

    //using System;
    //using XSockets.Core.Common.Interceptor;
    //using XSockets.Core.Common.Utility.Logging;
    //using XSockets.Plugin.Framework;
    
    public class MyErrorInterceptor : IErrorInterceptor
    {        
        public void OnError(Exception exception)
        {
            Composable.GetExport<IXLogger>().Error(exception, "Exception");
        }
    }
    
###How to write a custom JSON Serializer
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
    
###How to write custom Protocols

    TODO
    
##Securing the Controller
XSockets.NET does not provide any features for authenticating users. Instead, you integrate the XSockets:NET features into the existing authentication structure for an application. You authenticate users as you would normally in your application, and work with the results of the authentication in your XSockets.NET code. For example, you might authenticate your users with ASP.NET forms authentication, and then in your `Controller`, enforce which users or roles are authorized to call a method.

XSockets provides the Authorize attribute to specify which users have access to a `controller` or `method`. You apply the Authorize attribute to either a `controller` or particular `methods` in a `controller`. Without the Authorize attribute, all public methods on the controller are available to a client that is connected to the controller. 

If you want to allow unrestrcited access on some methods you can add the `AllowAnonymous` attribute the these methods.

More information about attributes and methods below

### Authorize Attribute

This attribute can be set on Controller or Method level. If set at controller level all action methods that do not have the `AllowAnonymous` attribute will require authentication.

The authorize attribute can take Roles and Users but if using that you will have to implement your own authentication by overriding `OnAuthorization(AuthorizeAttribute authorizeAttribute)`

### AllowAnonymous Attribute

This attribute can be set on action methods and will then allow anonymous access.

### Get FormsAuthentication Ticket

When you have custom authentication you can get the `FormsAuthenticationTicket` from this method.

    var ticket = GetFormsAuthenticationTicket();

**Note: If you do not pass in a cookiename .ASPXAUTH will be used.**

**Important: If you have separate project you will have to use the same origin to be able to get cookies and also use machine-key in the config to be able to get the AuthCookie.**
***See*** http://msdn.microsoft.com/en-us/library/system.web.configuration.machinekeysection.compatibilitymode%28v=vs.110%29.aspx ***if you are using different framework versions in the projects.***

### Write a custom AuthenticationPipeline
When the socket is connected and the handshake is completed the `AuthenticationPipeline` will be called. By default the pipeline will look for a FormsAuthenticationTicket, but you can override this pipline by just implementing a interface `XSockets.Core.Common.Socket.IXSocketAuthenticationPipeline`

There can only be one pipeline so even if you implement several pipelines only one wil be used.

    //using System.Security.Principal;
    //using XSockets.Core.Common.Protocol;
    //using XSockets.Core.Common.Socket;
    //using XSockets.Plugin.Framework.Attributes;

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

        //using System.Linq;
        //using XSockets.Core.Common.Socket.Attributes;
        //using XSockets.Core.XSocket;
        
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

    //using XSockets.Core.Common.Socket.Attributes;

    [Authorize()] //would be valid since we have a authorized `fake` user

    [Authorize(Users = "David")] //would be valid

    [Authorize(Roles = "batman,robin")] //would be invalid
    
    [Authorize(Roles = "batman,hulk")] //would be valid
    
### How to know when authorization fails
Every time a method on a controller needs authentication the `OnAuthorization` method is called (which was overridden in previous sample).

When the `OnAuthorization` returns false the `OnAuthorizationFailed` event is fired.

    //using XSockets.Core.Common.Socket.Event.Arguments;
    //using XSockets.Core.XSocket;
    
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

###How to get information about the client from the ConnectionContext
You can access various information about the connection from the `IConnectionContext`. The context is accessible from both Controller and Protocol.

    this.ConnectionContext
    
The context contains information such as:

- Cookies
- Headers
- Querystring
- IPrincipal
- PersistentId

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

## C-Sharp
The C# Client API has support for:

 - .NET 3.5, 4.0+ (this section)
 - iOS (MonoTouch) (this section)
 - Android (MonoDroid) (this section)
 - .NET MicroFramework 4.2, 4.3 (see specific section for NETMF)
 - Dart (see specific section)
 - Arduino (see specific section)

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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    this.InvokeToAll("I am Yogi the gummi bear","chatmessage");

####Methods without parameters
If the method you're handling does not have parameters just use the non generic `On` method.

**Server**

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;

    this.InvokeToAll(new ChatModel{Name="Yogi", Text="I am a gummi bear"},"chatmessage");

####Methods with parameters, specifying dynamic objects for the parameters
As an alternative to specifying specific types to the On method, you can specify parameters as dynamic objects.

    conn.Controller("chat").On<dynamic>("chatmessage", data => Console.WriteLine(data.Text));
    
*Note: Dynamic keyword will (currently) not exist in clients for Android/iOS.*
*Note: Dynamic keyword does not exist in .NET 3.5 and earlier.*

**Server**

The server code for the examples with the `ChatModel` can look like

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;

    this.InvokeToAll(new ChatModel{Name="Yogi", Text="I am a gummi bear"},"chatmessage");
    
####Methods with parameter of type IMessage
If you for some reason want the actual IMessage sent from the server to the client you can specify IMessage as the type. You will then get the complete object that the client received!

    conn.Controller("chat").On<IMessage>("chatmessage", o => Console.WriteLine("{0}, {1} {2}, {3},",o.Controller, o.Topic, o.Data, o.MessageType);
    
If you know the type contained in the Data part of the IMessage you can use Extract<T> to get the value.

    //example extracting the JSON into a specific type
    var chatModel = o.Extract<ChatModel>();

**Server**

The server code for the examples with the `ChatModel` can look like

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;

    this.InvokeToAll(new ChatModel{Name="Yogi", Text="I am a gummi bear"},"chatmessage");
    
####How to remove a handler
When you want to dispose of a handler you have to remove it from the `Controller` where it was used.

    var listener = conn.Controller("chat").On<string>("chatmessage", data => Console.WriteLine(data));   
                
    conn.Controller("chat").DisposeListener(listener);

####How to send to server methods from the client
To call a method on the server, use the Invoke method on the `Controller`.

If the server method has no return value, use the non-generic overload of the Invoke method.

**Server - method without return value**

    //using XSockets.Core.XSocket;

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
    foreach (Stock stock in stocks.Result)
    {
        Console.WriteLine("Symbol: {0} price: {1}\n", stock.Symbol, stock.Price);
    }
    
If there is no respons for 30000 ms there will be a `TimeoutException` so you should wrap the synchronous call like:

    try
    {
        var stocks = conn.Controller("stockticker").Invoke<IEnumerable<Stock>>("GetStocks");
        foreach (Stock stock in stocks.Result)
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
    
Of course you can set the default 30000 ms to be longer or shorter if needed. Just pass in your timeout as a parameter in the call like
    
    //Timeout will now be 5 seconds
    var stocks = conn.Controller("stockticker").Invoke<IEnumerable<Stock>>("GetStocks",5000);
    
###PUB/SUB
####How to define subscription methods on the client that the server can publish to

    conn.Controller("stockticker").Subscribe<Stock>("newStock", stock => Console.WriteLine("Symbol: {0} price: {1}\n", stock.Symbol, stock.Price));
        
####How to subscribe one time
If you only want to get a message once and then unsubscribe automatically you can use `one`

    conn.Controller("stockticker").One<Stock>("newStock", stock => Console.WriteLine("Symbol: {0} price: {1}\n", stock.Symbol, stock.Price));
    
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
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

###Binary data
XSockets has supported binary messages for a long time, but in 4.0 we have made it even easier than before.

####How to handle binary data
Lets say that we have a file `c:\temp\xfile.txt` with the text `This file was sent with XSockets.NET` and we want to send that file to the server.

**Client**

Client - C#

    var blob = File.ReadAllBytes(@"c:\temp\xfile.txt");
    conn.Controller("chat").Invoke("myfile", blob);

**Server**

    //using XSockets.Core.Common.Socket.Event.Interface;
    //using XSockets.Core.XSocket;
    
    public void MyFile(IMessage message)
    {
        var filecontent = Encoding.UTF8.GetString(message.Blob.ToArray());
    }


####How to pass meta-data together with binary data
If we want to attach metadata about the binary data that is easy to do. Just pass along the object representing the metadata and XSockets will let you extract that data on the server.

**Client**

Client - C#

    var blob = File.ReadAllBytes(@"c:\temp\xfile.txt");
    conn.Controller("chat").Invoke("myfile", blob, new {Name="xfile.txt"});
    
**Server**
    
    //using XSockets.Core.Common.Socket.Event.Interface;
    //using XSockets.Core.XSocket;
    
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

###How to handle errors

To handle errors that the client raises, you can add a handler for the Error event on the connection object.

    conn.OnError += (sender, errorArgs) => Console.WriteLine(errorArgs.Exception.Message);
    
You may also handle errors on individual controller using the OnError event on a specific controller.

    conn.Controller("chat").OnError += (sender, errorArgs) => Console.WriteLine(errorArgs.Exception.Message);

To handle errors from method invocations, wrap the code in a try-catch block. 

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;

    try
    {
        var stocks = conn.Controller("stockticker").Invoke<IEnumerable<Stock>>("GetStocks");
        foreach (Stock stock in stocks.Result)
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
##JavaScript
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;

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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
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
    
###Binary data
XSockets has supported binary messages for a long time, but in 4.0 we have made it even easier than before.

####How to handle binary data
Lets say that we want to send binary data from the client to the server.

**Clients**

Client - JavaScript

    // Create a simple Array buffer and fill it with "something"
    var arrayBuffer = new ArrayBuffer(10);
    // Send the binary data and metadata to XSockets
    conn.controller("chat").invokeBinary("myfile",blob);

**Server**

    //using XSockets.Core.Common.Socket.Event.Interface;
    //using XSockets.Core.XSocket;

    public void MyFile(IMessage message)
    {
        var filecontent = Encoding.UTF8.GetString(message.Blob.ToArray());
    }

####How to pass meta-data together with binary data
If we want to attach metadata about the binary data that is easy to do. Just pass along the object representing the metadata and XSockets will let you extract that data on the server.

**Client**

Client - JavaScript

    // Create a simple Array buffer and fill it with "something"
    var arrayBuffer = new ArrayBuffer(10);
    // Send the binary data and metadata to XSockets
    conn.controller("chat").invokeBinary("myfile",blob,{Name:"xfile.txt"});

**Server**

    //simple class for holding metadata about a file
    public class FileInfo
    {
        public string Name {get;set;}
    }

    //using XSockets.Core.Common.Socket.Event.Interface;
    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    public void MyFile(IMessage message)
    {
        var filecontent = Encoding.UTF8.GetString(message.Blob.ToArray());
        var metadata = message.Extract<FileInfo>();
    }

Just use `Extract<T>` to get back to metadata attached to the binary data.
    
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
## .NET MicroFramework
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
    //The server will publish the message back to all clients subscribing
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

    //using XSockets.Core.XSocket;
    //using XSockets.Core.XSocket.Helpers;
    
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
## Arduino
We have written a very very simple client for the Arduino.

###Client Setup
Download the Arduino client from GitHub and then add the library as shown here http://arduino.cc/en/Guide/Libraries

###How to establish a connection
Use the `connect` method to open a connection. The method returns false if the connection failed and true if it was ok.

	client.connect("192.168.0.108",4502)
    
- The first parameter is the IP of the server
- The second parameter is the port of the server

###How to send messages
The open method just opens the socket. When you communicate to a specific `controller` for the first time the controller will be created on the server. 

You always pass in `controller`, `topic` and `data` when sending from Arduino.

    client.send("chat","message","Hello from Arduino");
    
The code sample above would invoke the method "Message" on the controller "Chat" with the object/data "Hello from Arduino".

###How to receive messages
Set a message delegate to be able to receive data. This delegate will receive all data sent to the client from the server.

	//Set onMessage to handle data
	client.setOnMessageDelegate(onMessage);

	//The onMessage implementation
	void onMessage(XSocketClient client, String data) {
	  //Just write to serial port
	  Serial.println(data);
	}	

###Subscribe
If you use pub/sub in XSockets you will have to tell the server that you subscribe for specific topics.

	client.addListener("controllername","topic");

###Unsubscribe
When you wan to unsubscribe for a topic just use

	client.removeListener("controllername","topic");

###A Sample...
The code below connects to a XSockets server on 192.168.1.2 and port 4502.
If the connection was a success the client send a message (that will also fire the controller generic to be created).

Data received will be handled by the `onMessage` method.

	#include "Arduino.h"
	#include <Ethernet.h>
	#include <SPI.h>
	#include <XSocketClient.h>

	byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
	XSocketClient client;

	void setup() {
	  Serial.begin(9600);
	  Ethernet.begin(mac);
	  Serial.println("Start to connect");
	  if(client.connect("192.168.0.108",4502)){
	      Serial.println("Connected!");
	  }
	  else{
	      Serial.println("Error connecting");
	  }
  
	  client.setOnMessageDelegate(onMessage);
	  delay(1000);
  
	  client.send("generic","hello","Arduino connected");
	}

	void loop() {
	  client.receiveData();
  
	  delay(500);
	}

	void onMessage(XSocketClient client, String data) {
	  Serial.println(data);
	}

TIP: Test this in an easy way by using Putty like this

 1. Start XSockets.
 2. Connect the Arduino (by running sample above), but remember to change IP if needed.
 3. Open putty on the same machine as XSockets
 4. Enter 127.0.0.1 and port 4502
 5. Choose RAW as connection type
 6. When putty opens type "PuttyProtocol" and hit enter
 7. You are now connected....
 8. type `generic|message|Hello from putty`
 9. If all goes well you will see the message at the Arduino side.

----------
##Logging

As of 4.0 XSockets will use http://serilog.net as the default logger. Since it is a plugin you can of course replace SeriLog with something else if you want to.

By default the logger will log everything in `Information`.

###Customize Serilog

It is very easy to set your custom SeriLogger/Configuration/Sink. Just inherit the `XLogger`  class from assembly `XSockets.Logger.dll` and set the configuraiton of choice. For more information about SerilLog see http://serilog.net

All methods on XLogger is virtual so that you can override them, but if you want to redo the whole thing or even replace Serilog with another logger see the next section

    //using Serilog;
    //using XSockets.Logger;
    
    //
    // Sample below will write to console with level Verbose
    //
    public class MyLogger : XLogger
    {
        public MyLogger()
        {
            Log.Logger = new LoggerConfiguration().MinimumLevel.Verbose()
                .WriteTo.ColoredConsole()
                .CreateLogger();
        }
    }

###Custom Logger (replacing Serilog)
If you do not want Serilog or you just dont want to use the `Xlogger` you can implement your own since the default XLogger is an overridable module.

Just implement the `IXlogger` interface and implement all methods and your new logger is good to go.


    //using System;
    //using XSockets.Core.Common.Utility.Logging;
    
    public class MyXLogger : IXLogger
    {
        public void SetEventLevel(LogEventLevel level)
        {
            throw new NotImplementedException();
        }

        public void Verbose(string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Verbose(Exception ex, string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Debug(string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Debug(Exception ex, string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Information(string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Information(Exception ex, string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Warning(string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Warning(Exception ex, string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Error(string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Error(Exception ex, string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Fatal(string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public void Fatal(Exception ex, string template, params object[] parmeters)
        {
            throw new NotImplementedException();
        }

        public bool LevelEnabled(LogEventLevel level)
        {
            throw new NotImplementedException();
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

    //using XSockets.Core.Common.Configuration;
    //using XSockets.Core.Common.Socket;
    //using XSockets.Core.Configuration;
    //using XSockets.Plugin.Framework;
    
    //List of IConfigurationSettings
    var myCustomConfigs = new List<IConfigurationSetting>();
    //Add one configuration
    myCustomConfigs.Add(new ConfigurationSetting("ws://192.74.38.15:4502"));  
    using (var server = Composable.GetExport<IXSocketServerContainer>())
    {
        server.Start(configurationSettings:myCustomConfigs);
        Console.WriteLine("Started, hit enter to quit");
        Console.ReadLine();
        server.Stop();
    }

Note: you can of course pass in several configuration.

### Autoconfiguration with custom port

Just use the `AutoConfiguration` class to create a custom configuration that lets you choose a port to use. For example port 8181 as shown below.
    
    using (var container = Composable.GetExport<IXSocketServerContainer>())
	{
		//Get a custom config on port 8181
		var autoConfigPort8181 = new AutoConfiguration(defaultPort:8181).GetConfigurationSettings(true);
	    
		//Start the server with the autoconfig for port 8181
		container.Start(configurationSettings:autoConfigPort8181);
	    Console.ReadLine();
	}

### Configuration as a plugin

Just inherit the `XSockets.Core.Configuration.ConfigurationSetting` class and implement your configuration. XSockets will find and and use these custom configurations.

    //using XSockets.Core.Configuration;
    
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

    //using XSockets.Core.Configuration;
    
    public class ChuckNorrisConfig : ConfigurationSetting
    {
        public ChuckNorrisConfig() : base(new Uri("ws://chucknorris.com:4502")) { }
    }
    
**Note: This setup will create an endpoint based on the DNS provided. Note that port 4502 have to be open.

##### Public & Private Endpoint
Let's say that you want to connect (client <-> firewall <-> server) to ws://chucknorris.com:4502, but the public endpoint is represented by a firewall. Your firewall will then forward the connection to our servers private IP address (for example 192.168.1.7).

    //using XSockets.Core.Configuration;

    public class ChuckNorrisConfig : ConfigurationSetting
    {
        public ChuckNorrisConfig() : base(new Uri("ws://chucknorris.com:4502"), new Uri("ws://192.168.1.7:4510")) { }
    }

**Note: This setup requires that you forward traffic in your firewall to `192.168.1.7:4510`

### SSL/TLS
To get WSS you have to set the endpoint to be ´wss´ instead of ws, and you will also specify you certificate. This can either be done by setting `CertificateLocation` and `CertificateSubjectDistinguishedName` (as in the sample) or load the certificate from disk.

#### Certificate from store

    //using System.Security.Cryptography.X509Certificates;
    //using XSockets.Core.Configuration;

    public class ChuckNorrisConfig : ConfigurationSetting
    {
        public ChuckNorrisConfig() : base(new Uri("wss://chucknorris.com:4502"))
        {
            this.CertificateLocation = StoreLocation.LocalMachine;
            this.CertificateSubjectDistinguishedName = "cn=chucknorris.com";
        }
    }
    
#### X509Certificate2

    //using System.Security.Cryptography.X509Certificates;
    //using XSockets.Core.Configuration;
    
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

    //using XSockets.Owin.Host;

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
 2. Open up the Command Prompt and type ***cinst XSockets.Windows.Service***
    
###Console Application
Create a new ConsoleApplication and install the package XSockets. This will output some sample code for getting the server started.

**Install:**
`PM> Install-package XSockets`

    //using XSockets.Core.Common.Socket;
    //using XSockets.Plugin.Framework;

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

    TBD


## WebRTC
This repo contains the full source code of the [XSockets.NET][1]  WebRTC experiments.  

###Simple WebRTC application
We have created a simple demo package available on Nuget. This package gives you a very simple "video conference". The demo allows 1-n clients to connect and share MediaStreems, chat and send files.

    
    PM> Install Package XSockets.Sample.WebRTC
    

You can alsoo find a blog post on the following link:
http://xsockets.net/blog/tutorial-building-a-multivideo-chat-with-webrtc

###Testing the WebRTC-sample
When installation is completed just follow these steps

1: Add this class under the App_Start folder

	using System.Web;
	using XSockets.Core.Common.Socket;
	[assembly: PreApplicationStartMethod(typeof(VideoChatDemo.App_Start.xbooter), "Start")]
	namespace VideoChatDemo.App_Start
	{
		public static class xbooter
		{
			private static IXSocketServerContainer container;
			public static void Start()
			{
				container = XSockets.Plugin.Framework.Composable.GetExport<IXSocketServerContainer>();
				container.Start();
			}
		}
	}

2: Hit F5 to start debug

3: Navigate to http://localhost:xxxx/WebRTC/Client/index.html
   where xxxx should be the port for your application.

To build your own conference solution is really easy. Consult the [XSockets.NET developer forum][1] for help and guidance.

*To learn more about the WebRTC API, read the API-Guide below*

----------

  [1]: http://xsockets.net

###Supported platforms

####Server system requirements
Our WebRTC experiment requires a XSockets.NET generation 4.0  server setup.  The connection broker code provided is written and designed to run on XSockets.NET version 4.x. 

You can get XSockets.NET for free as a developer using Nuget. Read more about XSockets.NET 4.0 at http://xsockets.net/docs/4/getting-started-with-real-time.

The code  connection broker can be found in the XSockest.NET WebRTC repo

####Client support
The WebRTC experiment of ours ( Team XSockets.NET Sweden AB) is tested on the following browsers per the 10th of October 2014

     - Google Chrome 37 
     - Google Chrome Canary 40
     - Firefox 31
     - Firefox Aurora
     - Opera 24
    
*All functionality (RTCPeerConnection, MediaStreams and RTCDataChannels) works cross-browsers (according to the list above)* 

###JavaScript API - Documentation

Here follows a brief description of the JavaScript API. 

####Create a PeerConnection
In order to create a PeerConnection (`XSockets.WebRTC`) you need a PeerBroker to broker connections.

    var conn = new XSockets.WebSocket("ws://127.0.0.1:4502", ["connectionbroker"]);
    var broker = conn.controller("connectionbroker");
	    broker.onopen = function () {
	    rtc = new XSockets.WebRTC(broker);
    };
 

####Configuration (Customize)

By passing a custom *configuration* into the ctor of the `XSockets.WebRTC(broker,configuration)` you can easily modify the iceServers, `sdpConstraints` and `streamConstraints` and parameters.  You can also provide a set of expressions (sdpExpressions) that will be abale to intercept the SDP messages.
#####default configuration

    {
    "iceServers": [{
        "url": "stun:stun.l.google.com:19302"
    }],
    "sdpConstraints": {
        "optional": [],
        "mandatory": {
            "OfferToReceiveAudio": true,
            "OfferToReceiveVideo": true
        }
    },
    "streamConstraints": {
        "mandatory": {},
        "optional": []
    },
    "sdpExpressions": []
    }

##### Example modified iceServers & streamConstraints

    var rtc = new XSockets.WebRTC(broker, {
    iceServers: [{
        url: 'stun:404.idonotexist.net'
    }],
    streamConstraints: {
        optional: [{
            'bandwidth': 500
        }]
    }});       
    // Will give you the following result;    
    {
    "iceServers": [{
        "url": "stun:404.idonotexist.net"
    }],
    "sdpConstraints": {
        "optional": [],
        "mandatory": {
            "OfferToReceiveAudio": true,
            "OfferToReceiveVideo": true
        }
    },
    "streamConstraints": {
        "optional": [{
            "bandwidth": 500
        }]
    },
    "sdpExpressions": []
    }
    
#####sdpExpressions 

This expression parses and modifies the sdp and limits the video bandwidth 256 kilobits per second.

    
    expression:[
     function (sdp) { 
         return sdp.replace(/a=mid:video\r\n/g,
             'a=mid:video\r\nb=AS:256\r\n');
    }]

Expressions are passed 

####Context Events
#####oncontextcreated
This fires when you have a connection to the Broker controller

    
    rtc.oncontextcreated = function(ctx) {
      // do op
    }
    

#####oncontextchange
This fires when something happens on the context. Someone joins or leaves! You will get a list of peers on the current context.

    
    rtc.oncontextchange = function(arr) {
     // do op
    });
    
    
    
####Context Methods
##### changeContext
Changes your context and connects to other Peers on the broker. Pass in the Id of the context to join/connect

    rtc.changeContext(ctxId);
    
#####leaveContext
Leave the current context. "Hang up" on all other peers
    
    rtc.leaveContext();

#####connectToContext
Connect to the CurrentContext.  i.e if you want to connect to a predefined context.

    rtc.connectToContext();
    

####Peer Events
#####onconnectionstarted

Fires when the client starts to negotiation.

    rtc.onpeerconnectionstarted = function(peer){
    
    });
    
    

#####onconnectioncreated
Fires when the client has established a peer connection

    
    rtc.onpeerconnectioncreated = functction(peer){
        // do op
    });

#####onconnectionlost
Fires when a peer connection is lost (destroyed)

    
    rtc.onpeerconnectionlost = function(peer){
    
    });
    
#### Other PeerConnection events 
Other PeerConnection (peer) events of interest may be the following.  Note that we are using the `.bind(event,fn)` method for those. 


    rtc.bind("signalingstatechange", function (evt)
    {
    });
    
    rtc.bind("iceconnectionstatechange", function (evt)
    {
    });
    
    rtc.bind("negotiationneeded", function (evt)
    {
    });

####Peer Methods
#####removePeerConnection
Lets you remove a connection from the current context.

    rtc.removePeerConnection(peerId,callback);
    
#####getRemotePeers
Get a list of peerId's on the current context
    
    rtc.getRemotePeers();
    
    // returns an Array of PeerID's  i.e ["d383b53bb29947b5b1f62903bbc64d82"]
    
    
####MediaStream Methods
##### getUserMedia(constrints,success,failure)
Attach a local media stream ( camera / audio ) to the PeerConnection by calling `.getUserMedia(constrints,success,failure)`

    rtc.getUserMedia(rtc.userMediaConstraints.hd(true), function(result){    
        console.log("MediaStream using HD constrints and audio is added to the PeerConnection"
    ,result);
    });

##### addMediaStream(mediaStream,callback)
If you want to a (external) media stream to the PeerConnection (local) call the 
`addMediaStream(mediaStream,callback)`

      window.getUserMedia(rtc.userMediaConstraints.qvga(false), function (stream) {
          // Add the MediaStream capured
          rtc.addLocalStream(stream, function () {
              console.log("Added yet another media stream...");
     });

##### removeStream(streamId)

To remove a local media stream from the PeerConnection and all connected remote peerconnection call the .removeStream(streamID) method

     rtc.removeStream(streamId, function(id) {
        console.log("local stream removed", id);
     });

##### refreshStreams(peerId,callback)

When a media stream is added by using the .getUserMedia or .addMediaStream event you need to call refreshStreams method to initialize a renegotiation.

    rtc.refreshStreams(peerId, function (id) {
        console.log("Streams refreshed and renegotiation is done..");
    });

** to get a list of all remote peerconnections call the .`getRemotePeers()` method.

#####getLocalStreams()

To get a list of the peerconnection (clients ) media-streams call the `.getLocalStreams()` method

    var myLocalStreams = rtc.getLocalStreams();


#####getRemoteStreams()
To get a list of  mediaStreams attach to remote peers call the `getRemoteStreams()` method.

    var myRemoteStreams = rtc.getRemoteStreams();


####MediaStream Events
##### onlocalstream(event)

When a media stream is attached to the PeerConnection using `getUserMedia` och `addMediaStream` the API fires the `onlocalstream(stream)` event.

    
    rtc.onlocalstream = function(event){
         // do op
    });
    
    

##### onremotestream(event)

When a remote PeerConnection is connected the API fires the `onremotestream(event)` .

    rtc.onremotestream = function(event){
        // do op 
    });
    

##### onRemoteStreamLost

When a remote peer removes a stream (`.removeStream(mediaStreamId)`) the JavaScript API fires the `onRemoteStreamLost(streamId`) event on other peers connected to the same context
     
     rtc.onremotestreamlost =  function(function) {
        // do op
     });
     
####MediaSources

To get a list of media sources ( cameras, microphones ) attached to the users device you the API provides you with a few small helper functions located in the  `XSockets.WebRTC.MediaSource()`

#####getSources(fn)

`getSources(fn)` gives you an array of devices (label, kind and id) for each deviced attached.  Note that under a non secure (HTTPS) context you will get a list of anonymous names .

    mediaSources = new XSockets.WebRTC.MediaSource();
    
    mediaSources.getSources(function (sources) {
    
    // Filter video devices
    sources.filter(function (vs) {
        return vs.kind === "video";
    }).forEach(function (source, index) {
        // do op
    });
    // filter audio devices
    sources.filter(function (vs) {
        return vs.kind === "audio";
    }).forEach(function (source, index) {
        // do op
    });
    });

#####applySources(videoSourceId, audioSourceId)

To apply preferred media sources ( video, audio device) you need to call the applySources method on the userMediaConstrains of yours .

    
    // apply the selected sources to the getUserMedia constraints
    // get some QVGA constraints.
  
    var gumConstraints = peer.userMediaConstraints.vga(true);
    // Apply the selected / prefered sources
    
    gumConstraints.applySources(videoSourceId, audioSourceId);
    
    rtc.getUserMedia(gumConstraints, function (s) {
     // do op
    }, function () {});
    



###DataChannels

DataChannels can be attached to a PeerConnection by using the following `XSockets.WebRTC.DataChannel` object.  upon each dataChannel you can publish and subscribe to any topic.  Subscriptions can be added, deleted and modified without  changing the underlying connection (no need for renegotiation).   

####Create a new Datachannel (RTCDataChannel)

     var dc = new XSockets.WebRTC.DataChannel("chat");
     
     // Note you will need to add your DataChannel by calling addDataChannel(dc)
     
     rtc.addDataChannel(dc);
     
     // any event binding (open,close) or subscriptions needs to be tone prior to adding the channel
     
     
     
     
     
     

####DataChannel events

When you created your DataChannel object you can attach event listeners for the following events (not to be mixed with subscriptions)

#####onopen(peerId,event)
Fires when a DataChannel is open (ready)
     
    dc.onopen = function(peerId,event){
        // peerId is the identity of the PeerConnection 
        // event is the RTCDataChannel (native) event
    };
    
   
    
    
#####onclose(peerId,event)
Fires when a DataChannel is closed (by remote peer )
     
    dc.onclose = function(peerId){
        // peerId is the identity of the PeerConnection 
        // event is the RTCDataChannel (native) event
    };

####DataChannel methods

As described shortly above you can take advantake of a simple publish/subscribe pattern unpon you DataChannel.


#####subscribe(topic,cb)
Create a subscription to a topic topic and pass  callback function that  will be invoked when the datachannel receives a message on the actual topic 

    // Where dc is your XSockets.WebRTC.DataChannel object instance
    
    dc.subscribe("foo", function(message)
    {
        console.log("received a message on foo", message);
    });

#####subscribe(topic,cb) : BinaryMessages

Create a subscription to a topic and pass a callback function that will be invoked when a dataChannel  receives a message on the actual topic.  Working with `binaryMessages` is very similar to dealing with any arbitrary data. The example below uses jQuery to create a DOM element (`link`) . See `publishBinary`to get the context of this example.

    dc.subscribe("fileShare", function (file) {
    var blob = new Blob([file.binary], {
        type: file.data.type
    });
    var download = $("<a>").text(file.data.name).attr({
        download: file.data.filename,
        href: URL.createObjectURL(blob),
        target: "_blank"
    });
    // do op's with the download element
    });   

#####unsubscribe(topic, cb)

To remove (unsubscribe) a 'topic' pass the *topic* and an optional callback function that will be called when completed.

    dc.unsubscribe("foo", function() {
        console.log("i'm no longer subscribing to foo");
    });

#####publish(topic,data, cb) 
To send (publish) invoke the publish method using the specific topic, *data* is the payload. the optinal callback (cb) function woll be invoked after the payload is sent.

    dc.publish("foo", { myMessage: 'Petter Northug did not get a medal?' }, function(data){
        // the payload passed will be available here as well...
    });
    
    // if you attach an event listener for onpublish the topic & data will be passed forward 
    
    dc.onpublish = function(topic, data) {
            // do op
    };
    
#####publishTo(id,topic,data,*cb*)
To send (publish) a message to a specific PeerConnection , invoke the `publishTo` method using the PeerID of the target peer,  topic, *data* is the payload. the optinal callback (cb) function will be invoked after the payload is sent.

    dc.publishTo(rtc.getPeerConnections[0],"foo", {who:'Alexander Bard' what:'has a beard'});

*rtc.`getPeerConnections`[0] gets will give you the PeerID of the first peer connection.*

#####publishbinary(topic,bytes,data)
To pass an binary message invoke the `publishBinary` method 

     dc.publishBinary("fileshare", bytes, {
                        name: file.name,
                        type: file.type,
                        size: file.size
                    });

Where bytes is an `arrayBuffer`.   the dataChannel will pass a `XSockets.BinaryMessage`, thats a wrapper object that combines  arbitrary data (objects)  and arrayBuffers. See deal with binary messages described under the .`subscription` section

#####publishbinaryTo(id,topic,bytes, *data*)
To send (publish) a binary message to a specific PeerConnection , invoke the `publishbinaryTo` method using the PeerID of the target peer,  topic, bytes, and *data*. 


###XSockets.AudioAnalyser (experimental)

Not yet documented fully documented.  The main purpose is to be able to detect of the current user is silent / speaking during a certain interval (ms)

    // Simple example where you on the onlocalstream event attaches a analyser anf grabs the results
    rtc.onlocalstream = function(stream) {
    
        // Attach the local stream captured
      
      attachMediaStream(document.querySelector("#localStream"), stream);
      
       // Create a an AudioAnalyzer, this detect if the current user is speaking (each second)    
       
       var analyze = new XSockets.AudioAnalyser(stream, 1000);
        
        analyze.onAnalysis = function (result) {
         console.log(result);
            if (result.IsSpeaking) {
                        $("#localStream").toggleClass("speaks").removeClass("silent");
                        }else {
                                $("#localStream").toggleClass("silent").removeClass("speaks");
                        }
            // Lets notify/share others,
            ws.publish("StreamInfo", { peerId: rtc.CurrentContext.PeerId,streamInfo: result });
            };
        }