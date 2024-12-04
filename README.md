# OSMessengerBuddy
Creating a simple messenger app for devices within the same network can be achieved using Node.js. You can use **UDP** or **WebSocket** for communication, depending on the type of messaging you want. Here's how you can build it step-by-step:

---

### **1. Using WebSocket (Recommended for Messaging Apps)**

#### **Install Dependencies**
First, install the WebSocket library for Node.js:
```bash
npm install ws
```

#### **Server Code**
The server acts as a central hub to relay messages between devices on the same network.

```javascript
const WebSocket = require('ws');

// Start a WebSocket server
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
    console.log('A new client connected');

    // Broadcast incoming messages to all connected clients
    ws.on('message', (message) => {
        console.log(`Received: ${message}`);
        wss.clients.forEach((client) => {
            if (client.readyState === WebSocket.OPEN) {
                client.send(message); // Send the message to all connected clients
            }
        });
    });

    ws.on('close', () => {
        console.log('A client disconnected');
    });
});

console.log('WebSocket server is running on ws://localhost:8080');
```

#### **Client Code**
Each device in the network runs this client to connect to the server.

```javascript
const WebSocket = require('ws');

// Connect to the WebSocket server
const ws = new WebSocket('ws://<SERVER_IP>:8080');

// Prompt for user input
const readline = require('readline').createInterface({
    input: process.stdin,
    output: process.stdout,
});

ws.on('open', () => {
    console.log('Connected to the server');
    
    // Listen for user input and send it to the server
    readline.on('line', (input) => {
        ws.send(input);
    });
});

ws.on('message', (message) => {
    console.log(`Message received: ${message}`);
});

ws.on('close', () => {
    console.log('Disconnected from the server');
});
```

- Replace `<SERVER_IP>` with the IP address of the server (e.g., `192.168.x.x`).
- Run the server on one device and the client on multiple devices.

---

### **2. Using UDP (Lightweight, No Central Server)**

UDP is a lightweight alternative for sending messages directly between devices without a central server.

#### **Install UDP Module**
Node.js has a built-in `dgram` module for working with UDP.

#### **Device Code**
Each device runs this code to send and receive messages.

```javascript
const dgram = require('dgram');
const readline = require('readline');

const PORT = 12345; // The port to send and receive messages
const BROADCAST_ADDR = '192.168.x.255'; // Replace with your network's broadcast address

const socket = dgram.createSocket('udp4');

// Bind the socket to a port for receiving messages
socket.bind(PORT, () => {
    socket.setBroadcast(true); // Enable broadcasting
    console.log(`Listening for messages on port ${PORT}`);
});

// Handle incoming messages
socket.on('message', (msg, rinfo) => {
    console.log(`Message from ${rinfo.address}: ${msg}`);
});

// Prompt for user input to send messages
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
});

rl.on('line', (input) => {
    const message = Buffer.from(input);
    socket.send(message, 0, message.length, PORT, BROADCAST_ADDR, (err) => {
        if (err) console.error(err);
    });
});
```

- Replace `BROADCAST_ADDR` with your local network's broadcast address (e.g., `192.168.x.255`).
- This code broadcasts messages to all devices on the network.

---

### **3. Testing the App**

1. Ensure all devices are on the same Wi-Fi network.
2. Run the server script for WebSocket or the device script for UDP.
3. Use different terminals or devices to simulate messaging.

---

### **Comparison of WebSocket vs UDP**
| Feature                 | WebSocket                          | UDP                              |
|-------------------------|------------------------------------|----------------------------------|
| **Use Case**            | Reliable, ordered messaging       | Fast, unordered messaging        |
| **Central Server**      | Required                          | Not required                    |
| **Ease of Use**         | Simple setup                      | Slightly complex setup           |
| **Delivery Guarantee**  | Yes                               | No                               |

If you plan to add more features like user authentication, message persistence, or file sharing, starting with **WebSocket** is better.

Would you like to extend this example or move on to advanced features?
