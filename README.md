# Safe Room — Multiplayer P2P Group Chat

A **Socket.IO-based group chat** with named rooms, host controls, and real-time message delivery tracking. Users create or join rooms by name; the host mediates entry and controls the room lifecycle.

**[▶ Live Demo](https://woodyhoko.github.io/saferoom)**

---

## 1. Features

- **Named rooms** — create or join any room by name
- **Host role** — first user to create a room becomes host; host can accept/reject join requests, promote a new host, or close the room for all participants
- **Message delivery states** — every sent message progresses through `pending → sent → delivered` (or `failed` / `queued` if the recipient disconnects)
- **Participant list** — live sidebar showing all connected users in the room, updated in real time on join/leave
- **Dark UI** — Tailwind CSS `gray-900` with cyan accent, optimized for extended sessions

---

## 2. Architecture

```
Browser Client (HTML + Socket.IO client)
        │
        │   WebSocket (bidirectional, full-duplex)
        │
    Node.js Server (Socket.IO 4.7)
        │
    In-memory Room Registry
    ┌─────────────────────────────────┐
    │  roomId → { host, members[] }  │
    └─────────────────────────────────┘
        │
        └── Broadcast / targeted emit to room members
```

### 2.1 Socket.IO event model

The server uses **event-based messaging** rather than HTTP request-response. Key events:

| Event (client → server) | Payload | Server action |
|---|---|---|
| `create-room` | `{ roomName, username }` | Create room, emit `room-created` back |
| `join-request` | `{ roomName, username }` | Notify host; emit `join-pending` |
| `accept-join` | `{ socketId }` | Add member; broadcast `user-joined` |
| `send-message` | `{ roomId, text }` | Broadcast `receive-message` to room |
| `promote-host` | `{ socketId }` | Transfer host role |
| `close-room` | `{ roomId }` | Emit `room-closed` to all; delete room |

### 2.2 Message delivery tracking

Each message gets a UUID on creation. The server emits an acknowledgment callback when the broadcast completes:

```javascript
socket.to(roomId).emit('receive-message', msg, (ack) => {
    io.to(senderId).emit('message-delivered', { id: msg.id });
});
```

The client updates the message bubble's status icon from a spinner to a check mark on `message-delivered`, or to a warning icon on timeout.

### 2.3 Room persistence

Room state is held **in-memory** on the server — no database. This is intentional: rooms are ephemeral and no message history is retained after the room closes, matching the "safe room" privacy model.

---

## 3. Security considerations

- **No message persistence** — messages are never written to disk or database
- **No authentication** — usernames are self-reported; host controls prevent anonymous flooding
- **Transport security** — deploy behind HTTPS + WSS (e.g. Nginx reverse proxy with TLS)
- **Rate limiting** — recommended for production: `socket.io-rate-limiter` or middleware-level throttling

---

## 4. Stack

| Layer | Technology |
|---|---|
| Real-time transport | Socket.IO 4.7 (WebSocket + HTTP long-poll fallback) |
| Server | Node.js |
| Styling | Tailwind CSS (CDN build) |
| Typography | Inter (Google Fonts) |

---

## 5. Run locally

> **Note:** This repository contains the **frontend only**. A Socket.IO signaling server (`server.js`) is required but not included in this repo. You will need to run your own Socket.IO server and point the frontend's `SIGNALING_SERVER_URL` constant to it.

A minimal server can be started with:

```bash
npm install socket.io express
node server.js   # your own server implementation
# open http://localhost:3000
```

Open two browser tabs to simulate two participants — one creates the room, the other joins.
