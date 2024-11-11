

### Why Websockets?

- **HTTP limitations:**
    - HTTP is great for simple requests and responses (client-server model).
    - HTTP doesn't handle **server-side events** efficiently.
    - Building **real-time applications** like chat, collaborative editing, or live data updates requires a persistent connection.
    - **Long-polling** and **server-side events** (SSE) are workaround solutions, but not ideal.
- **Websockets advantages:**
    - Provide a persistent, **full-duplex** communication channel between client and server over a single connection.
    - **Full-duplex** means both client and server can send and receive data simultaneously.
    - Eliminates the need for constant HTTP requests (polling) to check for updates.
    - Enables real-time communication with minimal latency.

### What are Websockets?

- **Definition:** A communication protocol that provides **full-duplex** communication channels over a single, **long-lived connection**.
- **Difference from HTTP:**
    - HTTP: **Request-response** model, connection closes after each response.
    - Websocket: **Persistent connection**, stays open until explicitly closed.
    - Websocket connections initiate with an HTTP request (upgrade request), then switch to the websocket protocol.

### Real-time Applications

- **Examples:**
    - Live financial data charts (e.g., Binance).
    - Collaborative editing tools (e.g., Google Docs, Notion).
    - Chat applications (e.g., WhatsApp, Telegram, Backpack).
    - Multiplayer games (e.g., Minecraft, Fortnite).
- **Mechanism:**
    - The server pushes down events to the client over the persistent websocket connection.
    - This allows for real-time updates without constant polling from the client.

### Websockets vs. WebRTC

- **WebRTC:** Uses UDP for peer-to-peer communication, primarily for audio/video streaming and real-time data transfer.
- **Websockets:** Uses TCP for client-server communication, ideal for reliable delivery of data where every message is critical (e.g., chat).
- **Choice depends on use case:**
    - **WebRTC:** When some data loss is acceptable (e.g., video streaming, games with high FPS).
    - **Websockets:** When every message is important (e.g., chat, collaborative editing).

### Websockets in Node.js

- **Server-side libraries:**
    - `ws` (used in this tutorial), `websocket`, `socket.io` (an abstraction over websockets), etc.
    - These libraries handle websocket protocol handshake and communication.
- **Client-side:**
    - Websocket functionality is native to the browser (like `fetch`).
    - A `WebSocket` object is used to create a connection to the websocket server.
- **Example code:**
    - This tutorial uses `express` to create an HTTP server to handle initial requests and upgrades.
    - The `ws` library is used to create the websocket server and handle websocket connections.
    - The code demonstrates a simple echo server where the server sends back any message it receives from the client.

###  Code Breakdown

**Server-side (index.ts):**

```typescript
import express from "express";
import http from "http";
import { WebSocketServer } from "ws";

const app = express();
const port = 3000;

const server = http.createServer(app);
const wss = new WebSocketServer({ server }); 

// callback for new connections
wss.on('connection', cb: async (ws, req) => {
    console.log("someone connected!"); 
    // callback for received messages
    ws.on('message', (listener: (message: RawData) => {
        console.log(`received: ${message}`);
        // send back the received message
        ws.send(`Hello, you sent: ${message}`); 
    }));

    //Optional: send message from server periodically (using setInterval)
    setInterval(() => {
        ws.send("data from server")
    }, 1000);
});

// callback for health check
app.get('/health', (req, res) => {
    res.json({"msg": "I am healthy!"});
});

// start the server
server.listen(port); 
```

**Explanation:**

1. **Import modules:** Import `express`, `http`, and `WebSocketServer` from the respective libraries.
2. **Create Express app and server:** Initialize an Express app and an HTTP server using the `createServer` method.
3. **Create Websocket server:** Instantiate a new `WebSocketServer` instance, passing the HTTP server as an argument.
4. **Handle new connections:** Use the `wss.on('connection', ...)` event to define a callback function that executes when a new websocket connection is established. Inside this callback:
    - Log a message to indicate a new connection.
    - Use the `ws.on('message', ...)` event to define a callback function for handling incoming messages. This callback will:
        - Log the received message.
        - Send back an echo message to the client using `ws.send()`.
5. **Optional: Send messages periodically:** You can use `setInterval` to send data from the server periodically, demonstrating server-initiated messages.
6. **Create health check endpoint:** Define a basic HTTP GET endpoint `/health` using Express to check server health.
7. **Start the server:** Start the HTTP server using `server.listen(port)`.

**Client-side (index.html):**

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Simple WS Client</title>
        <script>
            // create a websocket connection to the server
            const webSocket = new WebSocket('ws://localhost:3000'); 

            function sendMessage() {
                // send the message entered in the input box
                webSocket.send(document.getElementById('inputbox').value); 
            }

            // handle incoming messages from the server
            webSocket.onmessage = function(event) { 
                // display the message in a div
                document.getElementById('serverMessages').appendChild(document.createTextNode(event.data));
                document.getElementById('serverMessages').appendChild(document.createElement('br'));
            };
        </script>
    </head>
    <body>
        <!-- Input box and button for sending messages -->
        <input type="text" id="inputbox"/><button onclick="sendMessage()">Send</button><br/><br/>

        <!-- Div for displaying messages from the server -->
        <h2>Events from server</h2>
        <div id="serverMessages"></div>
    </body>
</html>
```

**Explanation:**

1. **Create Websocket connection:** Instantiate a new `WebSocket` object with the server URL (`ws://localhost:3000`) to establish a connection.
2. **Send message function:** Define a function `sendMessage()` that gets the message from the input box (`inputbox`) and sends it to the server using `webSocket.send()`.
3. **Handle incoming messages:** Use the `webSocket.onmessage` event to define a callback function that executes when a message is received from the server. This callback:
    - Gets the received message from `event.data`.
    - Appends the message to the `serverMessages` div element.
    - Adds a line break after each message.

###  Distributed Websocket Servers

- **Scaling Websockets:** A single websocket server can handle a limited number of connections. For large-scale applications, we need to distribute the load across multiple websocket servers.
- **Redis for inter-server communication:**
    - Redis can be used as a message broker for communication between different websocket servers.
    - This allows events to be broadcasted to all connected clients, regardless of which server they are connected to.
- **Implementation:**
    - Each websocket server subscribes to a Redis channel for receiving broadcast messages.
    - When a client sends a message, its connected server publishes the message to the Redis channel.
    - All subscribed servers receive the message and forward it to their respective connected clients.

###  Conclusion

This tutorial provides a basic understanding of websockets and their implementation in Node.js. You learned about:

- Limitations of HTTP for real-time applications.
- Advantages and characteristics of websockets.
- How websockets enable real-time communication in applications like Binance, Google Docs, and chat systems.
- Difference between Websockets and WebRTC.
- Basic server-side and client-side code for a simple websocket echo server.
- The concept of distributed websocket servers and using Redis for inter-server communication.

Building upon these fundamentals, you can explore more advanced websocket concepts, integrate them into your own projects, and create scalable, real-time applications.
