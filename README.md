# WebSocket Redis Pub/Sub Server

A real-time messaging server that uses WebSocket for client connections and Redis Pub/Sub for scalable message distribution across multiple rooms.

## Overview

This server acts as a bridge between WebSocket clients and Redis Pub/Sub, enabling real-time communication in chat rooms or channels. It efficiently manages subscriptions by only subscribing to Redis channels when at least one user is interested in that room.

## Features

- **WebSocket Server**: Real-time bidirectional communication with clients
- **Redis Pub/Sub Integration**: Scalable message distribution using Redis
- **Room-based Messaging**: Support for multiple chat rooms/channels
- **Efficient Subscription Management**: Only subscribes to Redis channels when needed
- **Dynamic Room Join/Leave**: Users can subscribe and unsubscribe from rooms on the fly

## Prerequisites

- Node.js (v14 or higher)
- Redis server running locally or remotely
- TypeScript

## Installation

```bash
npm install ws redis
npm install --save-dev @types/ws
```

## Usage

### Starting the Server

```bash
npm start
```

The WebSocket server will start on port `8081`.

### Client Connection

Connect to the WebSocket server:

```javascript
const ws = new WebSocket('ws://localhost:8081');
```

### Message Protocol

The server accepts JSON messages with the following types:

#### 1. Subscribe to a Room

```json
{
  "type": "SUBSCRIBE",
  "room": "room-name"
}
```

#### 2. Unsubscribe from a Room

```json
{
  "type": "UNSUBSCRIBE",
  "room": "room-name"
}
```

#### 3. Send Message to a Room

```json
{
  "type": "sendMessage",
  "roomId": "room-name",
  "message": "Your message content"
}
```

### Receiving Messages

When a message is published to a room you're subscribed to, you'll receive it through your WebSocket connection:

```javascript
ws.onmessage = (event) => {
  console.log('Received:', event.data);
};
```

## Architecture

### How It Works

1. **Client Connection**: When a client connects, they receive a unique ID and an empty room list
2. **Room Subscription**: When a user subscribes to a room:
   - The room is added to their subscription list
   - If they're the first person interested in that room, the server subscribes to the Redis channel
3. **Message Publishing**: When a user sends a message:
   - The message is published to the Redis channel for that room
   - Redis distributes the message to all server instances subscribed to that channel
4. **Message Distribution**: When a message arrives from Redis:
   - The server forwards it to all connected WebSocket clients subscribed to that room
5. **Room Unsubscription**: When a user unsubscribes:
   - The room is removed from their subscription list
   - If they were the last person interested, the server unsubscribes from the Redis channel

### Key Functions

- `oneUserSubscribedTo(roomId)`: Checks if exactly one user is subscribed to a room (triggers Redis subscription)
- `lastPersonLeftRoom(roomId)`: Checks if no users remain in a room (triggers Redis unsubscription)
- `randomId()`: Generates unique IDs for connected clients

## Configuration

### WebSocket Port

Default port is `8081`. Modify in the code:

```typescript
const wss = new WebSocketServer({ port: 8081 });
```

### Redis Connection

Default connection is to `localhost:6379`. To configure Redis:

```typescript
const publishClient = createClient({
  url: 'redis://username:password@host:port'
});
```

## Example Client Implementation

```javascript
const ws = new WebSocket('ws://localhost:8081');

ws.onopen = () => {
  // Subscribe to a room
  ws.send(JSON.stringify({
    type: 'SUBSCRIBE',
    room: 'general'
  }));
  
  // Send a message
  ws.send(JSON.stringify({
    type: 'sendMessage',
    roomId: 'general',
    message: 'Hello, everyone!'
  }));
};

ws.onmessage = (event) => {
  console.log('Received:', event.data);
};

ws.onclose = () => {
  console.log('Disconnected');
};
```

## Use Cases

- Real-time chat applications
- Live notifications systems
- Collaborative tools
- Gaming lobbies
- Live event broadcasting
- Multi-room communication platforms

## Scalability

This architecture supports horizontal scaling:
- Multiple server instances can run simultaneously
- Redis Pub/Sub handles message distribution across all instances
- Each instance only subscribes to Redis channels that its connected clients need

## Notes

- Client disconnections are automatically handled by WebSocket
- The server optimizes Redis subscriptions by only subscribing when needed
- Each user can be subscribed to multiple rooms simultaneously
