# Safe Room

A **multi-room P2P group chat** built with Socket.IO and Tailwind CSS. Users create or join named rooms, and the host can transfer ownership or close the room gracefully.

**[▶ Live Demo](https://woodyhoko.github.io/saferoom)**

---

## Features

- 🏠 **Named rooms** — create or join any room by name
- 👤 **Host controls** — host can accept/auto-accept join requests, promote a new host, or close the room for everyone
- ✅ **Message delivery status** — pending / sent / failed / queued indicators per message
- 🔔 **Participant list** — live sidebar showing who's in the room
- 🎨 **Dark UI** — gray-900 Tailwind theme with cyan accents

---

## Architecture

```
Client (Browser)
    │
    └── Socket.IO  ──►  Server (Node.js)
                            │
                     Room Registry (in-memory)
                            │
                     ◄── Broadcast to room members
```

The frontend is a single HTML file. The server coordinates room state and message routing via Socket.IO events.

---

## Stack

| Layer | Technology |
|---|---|
| Real-time transport | [Socket.IO 4.7](https://socket.io/) |
| Styling | [Tailwind CSS](https://tailwindcss.com/) (CDN) |
| Typography | Inter (Google Fonts) |
| Server | Node.js (Socket.IO server) |

---

## Run Locally

```bash
# Install dependencies
npm install

# Start the server
node server.js

# Open http://localhost:3000
```
