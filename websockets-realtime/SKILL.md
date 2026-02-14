---
name: websockets-realtime
description: "Build real-time features with WebSockets, Server-Sent Events (SSE), Socket.io, and WebRTC. Implement chat, live updates, notifications, and collaborative features."
---

# WebSockets & Real-Time Communication Skill

Implement real-time, bidirectional communication for interactive applications.

## When to Use

Use this skill when the user wants to:
- Build real-time chat applications
- Implement live notifications and updates
- Create collaborative editing features
- Build real-time dashboards and analytics
- Implement presence indicators (online/offline)
- Create multiplayer games
- Build live streaming features
- Implement WebRTC video/audio calls
- Create real-time data synchronization
- Build live sports scores or stock tickers

## Technologies

### WebSockets (Native)
- **Protocol**: ws:// or wss:// (secure)
- **Use Case**: Low-level, bidirectional communication
- **Advantages**: Native browser support, low overhead
- **Disadvantages**: No automatic reconnection, no fallback

### Socket.io
- **Protocol**: WebSocket with fallbacks
- **Use Case**: Real-time apps with cross-browser support
- **Advantages**: Auto-reconnection, rooms, broadcasting, fallbacks
- **Disadvantages**: Slightly higher overhead

### Server-Sent Events (SSE)
- **Protocol**: HTTP-based, one-way (server to client)
- **Use Case**: Live updates, notifications
- **Advantages**: Simple, HTTP-compatible, auto-reconnection
- **Disadvantages**: One-way only, browser connection limits

### WebRTC
- **Protocol**: Peer-to-peer
- **Use Case**: Video/audio calls, file sharing
- **Advantages**: Peer-to-peer, low latency
- **Disadvantages**: Complex setup, NAT traversal

## WebSocket Implementation

### Server (Python - FastAPI)
```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List
import json

app = FastAPI()

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: str):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            message = json.loads(data)

            # Broadcast to all clients
            await manager.broadcast(
                json.dumps({
                    'client_id': client_id,
                    'message': message.get('text'),
                    'timestamp': time.time()
                })
            )

    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(
            json.dumps({'event': 'user_left', 'client_id': client_id})
        )
```

### Client (JavaScript)
```javascript
class WebSocketClient {
    constructor(url) {
        this.url = url;
        this.ws = null;
        this.reconnectInterval = 1000;
        this.maxReconnectInterval = 30000;
        this.reconnectDecay = 1.5;
        this.timeoutInterval = 2000;
    }

    connect() {
        this.ws = new WebSocket(this.url);

        this.ws.onopen = () => {
            console.log('WebSocket connected');
            this.reconnectInterval = 1000; // Reset reconnect interval
            this.onOpen?.();
        };

        this.ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            this.onMessage?.(data);
        };

        this.ws.onerror = (error) => {
            console.error('WebSocket error:', error);
            this.onError?.(error);
        };

        this.ws.onclose = () => {
            console.log('WebSocket closed');
            this.onClose?.();
            this.reconnect();
        };
    }

    reconnect() {
        setTimeout(() => {
            console.log('Attempting to reconnect...');
            this.reconnectInterval = Math.min(
                this.maxReconnectInterval,
                this.reconnectInterval * this.reconnectDecay
            );
            this.connect();
        }, this.reconnectInterval);
    }

    send(data) {
        if (this.ws?.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(data));
        } else {
            console.error('WebSocket is not connected');
        }
    }

    close() {
        this.ws?.close();
    }

    // Event handlers (to be set by user)
    onOpen = null;
    onMessage = null;
    onError = null;
    onClose = null;
}

// Usage
const client = new WebSocketClient('ws://localhost:8000/ws/user123');
client.onMessage = (data) => {
    console.log('Received:', data);
    // Update UI
};
client.connect();
```

## Socket.io Implementation

### Server (Node.js)
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
    cors: { origin: '*' }
});

// Middleware
io.use((socket, next) => {
    const token = socket.handshake.auth.token;
    if (isValidToken(token)) {
        socket.userId = getUserIdFromToken(token);
        next();
    } else {
        next(new Error('Authentication error'));
    }
});

// Connection handling
io.on('connection', (socket) => {
    console.log('User connected:', socket.userId);

    // Join user-specific room
    socket.join(`user:${socket.userId}`);

    // Handle chat messages
    socket.on('chat:message', async (data) => {
        const message = {
            id: generateId(),
            userId: socket.userId,
            text: data.text,
            timestamp: Date.now()
        };

        // Save to database
        await saveMessage(message);

        // Broadcast to room
        io.to(data.roomId).emit('chat:message', message);
    });

    // Handle typing indicator
    socket.on('chat:typing', (data) => {
        socket.to(data.roomId).emit('chat:typing', {
            userId: socket.userId,
            isTyping: data.isTyping
        });
    });

    // Handle presence
    socket.on('presence:update', (status) => {
        io.emit('presence:update', {
            userId: socket.userId,
            status: status
        });
    });

    // Disconnection
    socket.on('disconnect', () => {
        console.log('User disconnected:', socket.userId);
        io.emit('presence:update', {
            userId: socket.userId,
            status: 'offline'
        });
    });
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

### Client (React)
```javascript
import { useEffect, useState } from 'react';
import io from 'socket.io-client';

function ChatComponent() {
    const [socket, setSocket] = useState(null);
    const [messages, setMessages] = useState([]);
    const [inputValue, setInputValue] = useState('');

    useEffect(() => {
        const newSocket = io('http://localhost:3000', {
            auth: { token: getAuthToken() }
        });

        newSocket.on('connect', () => {
            console.log('Connected to server');
        });

        newSocket.on('chat:message', (message) => {
            setMessages(prev => [...prev, message]);
        });

        newSocket.on('chat:typing', (data) => {
            // Show typing indicator
        });

        setSocket(newSocket);

        return () => newSocket.close();
    }, []);

    const sendMessage = () => {
        if (socket && inputValue.trim()) {
            socket.emit('chat:message', {
                roomId: 'room123',
                text: inputValue
            });
            setInputValue('');
        }
    };

    const handleTyping = (isTyping) => {
        socket?.emit('chat:typing', {
            roomId: 'room123',
            isTyping
        });
    };

    return (
        <div>
            <div className="messages">
                {messages.map(msg => (
                    <div key={msg.id}>{msg.text}</div>
                ))}
            </div>
            <input
                value={inputValue}
                onChange={(e) => setInputValue(e.target.value)}
                onFocus={() => handleTyping(true)}
                onBlur={() => handleTyping(false)}
            />
            <button onClick={sendMessage}>Send</button>
        </div>
    );
}
```

## Server-Sent Events (SSE)

### Server (Python - Flask)
```python
from flask import Flask, Response
import time
import json

app = Flask(__name__)

def event_stream():
    """Generate server-sent events."""
    while True:
        # Fetch latest data
        data = get_latest_updates()

        yield f"data: {json.dumps(data)}\n\n"
        time.sleep(1)

@app.route('/stream')
def stream():
    return Response(
        event_stream(),
        mimetype='text/event-stream',
        headers={
            'Cache-Control': 'no-cache',
            'X-Accel-Buffering': 'no'
        }
    )

# Named events
def notification_stream():
    while True:
        notification = get_notification()

        # Named event
        yield f"event: notification\n"
        yield f"data: {json.dumps(notification)}\n\n"

        time.sleep(5)
```

### Client (JavaScript)
```javascript
const eventSource = new EventSource('/stream');

// Handle default messages
eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Received:', data);
    updateUI(data);
};

// Handle named events
eventSource.addEventListener('notification', (event) => {
    const notification = JSON.parse(event.data);
    showNotification(notification);
});

// Handle errors
eventSource.onerror = (error) => {
    console.error('SSE error:', error);
    if (eventSource.readyState === EventSource.CLOSED) {
        console.log('Connection closed');
    }
};

// Close connection
// eventSource.close();
```

## Real-Time Patterns

### Presence System
```javascript
// Server
const presence = new Map(); // userId -> { status, lastSeen }

socket.on('presence:heartbeat', () => {
    presence.set(socket.userId, {
        status: 'online',
        lastSeen: Date.now()
    });
});

// Check for inactive users every 30 seconds
setInterval(() => {
    const now = Date.now();
    for (const [userId, data] of presence.entries()) {
        if (now - data.lastSeen > 60000) { // 1 minute timeout
            presence.set(userId, { status: 'offline', lastSeen: data.lastSeen });
            io.emit('presence:update', { userId, status: 'offline' });
        }
    }
}, 30000);
```

### Typing Indicators
```javascript
// Client
let typingTimeout;

input.addEventListener('input', () => {
    socket.emit('typing:start', { roomId });

    clearTimeout(typingTimeout);
    typingTimeout = setTimeout(() => {
        socket.emit('typing:stop', { roomId });
    }, 1000);
});
```

### Live Notifications
```javascript
// Server
function sendNotification(userId, notification) {
    io.to(`user:${userId}`).emit('notification', {
        id: notification.id,
        type: notification.type,
        message: notification.message,
        timestamp: Date.now()
    });
}
```

### Collaborative Editing (Operational Transform)
```javascript
// Server
let document = '';
let version = 0;

socket.on('document:operation', (operation) => {
    if (operation.version === version) {
        // Apply operation
        document = applyOperation(document, operation);
        version++;

        // Broadcast to others
        socket.broadcast.emit('document:operation', {
            ...operation,
            version
        });
    } else {
        // Send full document for sync
        socket.emit('document:sync', { document, version });
    }
});
```

## Best Practices

### Connection Management
- **Heartbeat/ping-pong**: Keep connections alive
- **Automatic reconnection**: Handle disconnects gracefully
- **Exponential backoff**: Prevent server overload
- **Connection pooling**: Manage resources efficiently

### Security
- **Authentication**: Verify users before connecting
- **Authorization**: Check permissions for rooms/channels
- **Rate limiting**: Prevent abuse
- **Input validation**: Sanitize all messages
- **CORS configuration**: Restrict origins
- **SSL/TLS**: Use wss:// in production

### Performance
- **Message batching**: Combine multiple updates
- **Throttling**: Limit update frequency
- **Compression**: Use permessage-deflate
- **Binary protocols**: Use MessagePack or Protocol Buffers
- **Horizontal scaling**: Use Redis adapter for Socket.io

### Scalability
```javascript
// Socket.io with Redis adapter
const redisAdapter = require('socket.io-redis');

io.adapter(redisAdapter({
    host: 'localhost',
    port: 6379
}));

// Now works across multiple server instances
```

## Monitoring & Debugging

### Metrics to Track
- Active connections count
- Messages per second
- Connection duration
- Reconnection rate
- Error rate
- Latency

### Logging
```javascript
// Log all events
io.on('connection', (socket) => {
    console.log(`[${new Date().toISOString()}] User connected: ${socket.userId}`);

    socket.onAny((eventName, ...args) => {
        console.log(`[${new Date().toISOString()}] Event: ${eventName}`, args);
    });
});
```

## Testing

### Unit Testing
```javascript
const { createServer } = require('http');
const { Server } = require('socket.io');
const Client = require('socket.io-client');

describe('Socket.io tests', () => {
    let io, serverSocket, clientSocket;

    beforeAll((done) => {
        const httpServer = createServer();
        io = new Server(httpServer);
        httpServer.listen(() => {
            const port = httpServer.address().port;
            clientSocket = new Client(`http://localhost:${port}`);
            io.on('connection', (socket) => {
                serverSocket = socket;
            });
            clientSocket.on('connect', done);
        });
    });

    afterAll(() => {
        io.close();
        clientSocket.close();
    });

    test('should send and receive messages', (done) => {
        clientSocket.on('hello', (arg) => {
            expect(arg).toBe('world');
            done();
        });
        serverSocket.emit('hello', 'world');
    });
});
```

## Common Use Cases

1. **Chat Applications**: Real-time messaging, group chats
2. **Live Notifications**: Push notifications, alerts
3. **Collaborative Tools**: Shared documents, whiteboards
4. **Dashboards**: Real-time metrics, analytics
5. **Gaming**: Multiplayer games, leaderboards
6. **Live Feeds**: Social media feeds, activity streams
7. **Stock Tickers**: Real-time price updates
8. **Sports Scores**: Live game updates
9. **Auction Sites**: Real-time bidding
10. **Customer Support**: Live chat support

## Deliverables

- WebSocket server implementation
- Client connection management
- Room/channel system (if needed)
- Authentication and authorization
- Reconnection logic
- Error handling
- Message validation
- Monitoring and logging
- Documentation
- Test suite

## Quality Checklist

- Authentication is implemented
- Auto-reconnection works properly
- Error handling is comprehensive
- Messages are validated and sanitized
- Rate limiting is in place
- SSL/TLS is configured for production
- Heartbeat/ping-pong implemented
- Horizontal scaling strategy defined
- Monitoring and logging enabled
- Load testing performed
- Documentation is complete
