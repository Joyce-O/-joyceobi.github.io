---
title: "All things Sockets & Networks - Using NodeJs framework - Part 1"
date: 2021-11-02T00:00:00+00:00
author: Joyce Obi
layout: post
permalink: /all-things-sockets-&-networks-Using-NodeJs framework/
categories: Backend Stuff
tags: [socket, network, nodejs]
table-of-content: [WebSocket]
---

![Image by WilliamsCreativity from Pixabay]({{ site.baseurl }}/assets/images/post/server-g9.jpg "ServerImage")

My team recently had a challenge that required a web app (we'll call it device A) and another electronic device (we'll call device B) interacting, where several requests - responses are to  be exchanged and at various intervals.

Before the advent of the modern internet, this type of program would require a wire cable to be plugged at some point in the integration process, but alas, there wasn't one in sight and instead we had to rely completely on software components to do the job.

What are these components that made communication possible? You may ask. Without further ado, let's start from the modest 👇🏾

<div id="WebSocket"></div>

### WebSocket
>
> **Technical explaination:**
> *WebSocket is a computer communications protocol, providing full-duplex communication channels over a single TCP connection.* - says [Wikipedia](https://en.wikipedia.org/wiki/WebSocket)

...Okay! What role exactly did webSocket play here?

You see, we needed a way to receive the requests and responses sent by device B, that has determined not to respond immediately it receives a message from device A. 

Http as you may know, shines when it involves one request and one instant response and that is not what is happening here. WebSocket on the other hand, connection stays open and it is not just one request response call, making it the ideal candidate in this case.

Adding webSocket to your web app is as simple as doing the following;

{% highlight c %}

// Create WebSocket connection.
const socket = new WebSocket('ws://localhost:8080');

// Connection opened
socket.addEventListener('open', function (event) {
    socket.send('Hello Server!');
});

// Listen for messages
socket.addEventListener('message', function (event) {
    console.log('Message from server ', event.data);
});

{% endhighlight %}
 
And viola! You can now expect the server to push messages without you needing to send another request.

Now that we have connected device A to the server and established connection, rememeber that device B is still busy with other things and hasn't bothered to fetch our request from the server - the **server** by the way is acting as the middleman in this whole interaction.

Before we go on to establish connection between device B and the server, let's look briefly at what the server looks like

{% highlight c %}

<!-- Note: This was implemented in NodeJs -->

const http = require('http');
//import npm package - websocket
const WebSocketServer = require('websocket').server;

const httpServer = http.createServer((req, res) => {
    console.log((new Date()) + ' Received request for ' + request.url);
    response.writeHead(404);
    response.end();
});

const wsServer = new WebSocketServer({
    httpServer: server,
    // You should not use autoAcceptConnections for production
    // applications, as it defeats all standard cross-origin protection
    // facilities built into the protocol and the browser.  You should
    // *always* verify the connection's origin and decide whether or not
    // to accept it.
    autoAcceptConnections: false
});

const originIsAllowed = (origin) => {
  // put logic here to detect whether the specified origin is allowed.
  return true;
}

//handle event here
wsServer.on('request', function(request) {
    if (!originIsAllowed(request.origin)) {
      // Make sure we only accept requests from an allowed origin
      request.reject();
      console.log((new Date()) + ' Connection from origin ' + request.origin + ' rejected.');
      return;
    }

   let connection = request.accept('echo-protocol', request.origin);

    console.log((new Date()) + ' Connection accepted.');

    connection.on('message', function(message) {
        if (message.type === 'utf8') {
            console.log('Received Message: ' + message.utf8Data);
            connection.sendUTF(message.utf8Data);
        }
        else if (message.type === 'binary') {
            console.log('Received Binary Message of ' + message.binaryData.length + ' bytes');
            connection.sendBytes(message.binaryData);
        }
    });
    connection.on('close', function(reasonCode, description) {
        console.log((new Date()) + ' Peer ' + connection.remoteAddress + ' disconnected.');
    });
});

httpServer.listen(8080, () => {
    console.log('Http server listening on port', 8080);
});


{% endhighlight %}

As always, there're other libraries such as **[ws](https://github.com/websockets/ws)** that can be used to acheive the same thing!

Here, we're basically instantiating **WebSocketServer** class of the library we imported in line 2 and telling it to attach to this http server instance.

Server is up and running and we can now go over to the other client, device B.

So far, we've looked at **webSocket** as one of the components in modern internet communication. In addition to that, we will be touching other components and the role they played in this very important communication.

Stay tuned for part 2.

### Further Reading
I found this **[article](https://www.tutorialspoint.com/communication_technologies/communication_technologies_history_of_networking.htm)** informative, especially if you want to learn the history