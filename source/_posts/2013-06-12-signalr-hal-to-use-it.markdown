---
layout: post
title: "SignalR - HAL To Use It"
date: 2013-06-12 14:11
comments: true
categories: 
---


## SignalR - HAL to use it

You probably shouldn't be using it without a good reason, but if you have anything that you need to update in "real time" in your application you should use it.

SignalR for those who don't know is

> ASP.NET SignalR is a new library for ASP.NET developers that makes it incredibly simple to add real-time web functionality to your applications. What is "real-time web" functionality? It's the ability to have your server-side code push content to the connected clients as it happens, in real-time. - SignalR

SignalR is an official Microsoft project.

### Installation

You can install it with

  Install-Package Microsoft.AspNet.SignalR -pre

Warning it's still in RC but it's stable

### Hello, Dave

The fastest way to use SignalR is to

Add New File -> Class
    
    public class MyHub : Hub
    {
      public string Hello()
      {
        return "Hello, Dave.";
      }
    }
 
Then in your `Global.ascx` add this line

    RouteTable.Routes.MapHubs();

(For people familiar with previously using SignalR, it no longer generates a javascript proxy for you more on this later)

Then in a view add the appropriate scripts

    <script type="text/javascript" src="~/Scripts/jquery-1.9.0.js"></script>
    <script type="text/javascript" src="~/Scripts/jquery.signalR-1.0.0-rc2.min.js"></script>
     
    <script>
      $(function() { 
        var connection = $.hubConnection(); //this is a connection specifically for a hub
        var proxy = connection.createHubProxy('myHub'); //note the capitalization
     
        //We must start the connection before we do anything with it
        connection.start();
     
        //again note capitalization
        proxy.invoke('hello').done(function(result) { 
          alert(result);
        });
      }):
    </script>
 
This should prompt an alert with "Hello, Dave."

### Interact

Now adding the method

    public void OpenThePodBayDoors()
    {
      Clients.Caller.response('I'm sorry Dave I can't do that');
    }
     
    Add the following to your view

    <button id="clicker">Please</button>
    Add to your script section

    $('#clicker').bind('click', function() { 
      proxy.invoke('openThePodBayDoors');
    });
     
    proxy.on('response', function(message) { 
      alert(message); 
    });
     
So in the above examples what we have done is created a proxy to our hub, via the createHubProxy(hubName) call.
Bound to specific server raised events with proxy.on(event, callback) and invoked events from the client with `proxy.invoke('event', data);

### Using A Generated Proxy

If we change the MapHubs call to contain the options to generate the Javascript Proxy for us we can change our code to look like this

 
    RouteTable.Routes.MapHubs(new HubConfiguration() { EnableJavaScriptProxies = true });

    <script>
      $(function() { 
        var connection = $.connection.hub(); 
        var hub = $.connection.myHub;
     
        //Again we must start the connection before we do anything with it
        connection.start();
     
     
        hub.server.hello().done(function(result) { 
          alert(result);
        });
     
        $('#clicker').bind('click', function() { 
          hub.server.openThePodBayDoors();
        });
     
        hub.client.response = function(message) { 
          alert(message); 
        };
     
      }):
    </script>
 
In this scenario we have two properties on our hub proxy which is created under $.connection.hubName (in this case hubName is myHub), client and server, which seperate the client and server methods from each other. We then can invoke server side methods, or set client side methods to a callback.

Notice - other than the change to the MapHubs call there is no change to our Hub.

### Groups

SignalR contains the ability to assign a connection to a Group. The best thing to know about groups is that they are effectively channels that messages are only sent to clients that are in that channel.

The api for groups is as follows

  Groups.Add(string ConnectionId, string GroupName)

  Groups.Remove(string ConnectionId, string GroupName)

A thing to remember is that SignalR doesn't persist anything, so it's up to you if you want to be able to access who is in a group.

A good way to use groups is for something like a chat room.

  public void JoinRoom(string room)
  {
    //Context.ConnectionId is the ConnectionId of the person who called the method
    Groups.Add(Context.ConnectionId, room);
  }
   
  public void LeaveRoom(string room)
  {
    Groups.Remove(Context.ConnectionId, room);
  }
   
  public void SendMessage(string room, string message)
  {
    Clients.Groups(room).newMessage(message); 
  }
   
  There is nothing to be done to specifically handle groups on the client end but the code would look similar to

  function joinRoom(roomName) {
    proxy.invoke('joinRoom', roomName);
  }
   
  function leaveRoom(roomName) {
    proxy.invoke('leaveRoom', roomName);
  }
   
  proxy.on('newMessage', function(message) { 
    alert(message);
  });
 
### Closing the Doors

Hopefully this has provided you with a brief glance of what is capable in SignalR via the javascript client (at least for this post). I plan to cover more advanced use of SignalR and using it with other Clients in the near future.