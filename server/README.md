# Chat Server API Documentation

## Table of Contents
- [Overview](#overview)
- [Authentication](#authentication)
- [REST API Endpoints](#rest-api-endpoints)
  - [Authentication](#auth-endpoints)
  - [Rooms](#room-endpoints)
  - [Messages](#message-endpoints)
- [WebSocket Events](#websocket-events)
- [Data Models](#data-models)
- [Error Handling](#error-handling)
- [Rate Limiting and Security](#rate-limiting-and-security)
- [Environment Variables](#environment-variables)
- [Health Check](#health-check)

---

## Overview

The Chat Server provides real-time messaging with:
- Express.js for REST API endpoints
- Socket.IO for WebSocket communication
- MongoDB for persistence
- JWT for authentication

Base URL: `http://localhost:5000` (configurable with `PORT`)

CORS accepts the frontend origin from `FRONTEND_ORIGIN`.

---

## Authentication

All protected endpoints and WebSocket connections require a JWT token.

### Obtaining a Token

Use `POST /api/auth/login`.

### Using the Token

REST API header:

```text
Authorization: Bearer YOUR_JWT_TOKEN
```

WebSocket handshake:

```javascript
const socket = io('http://localhost:5000', {
  auth: {
    token: 'YOUR_JWT_TOKEN'
  }
});
```

Token expiry: 7 days.

---

## REST API Endpoints

### Auth Endpoints

#### Login

Endpoint: `POST /api/auth/login`  
Authentication: Not required

Request body:

```json
{
  "username": "john_doe",
  "password": "securePassword123"
}
```

Response (200):

```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "username": "john_doe"
  }
}
```

Error responses:
- `400`: Missing username or password
- `401`: Invalid credentials

---

### Room Endpoints

#### Create Room

Endpoint: `POST /api/rooms`  
Authentication: Required

Request body:

```json
{
  "name": "General Discussion"
}
```

Response (201):

```json
{
  "success": true,
  "room": {
    "id": "507f1f77bcf86cd799439011",
    "name": "General Discussion",
    "code": "ABC123",
    "users": ["507f1f77bcf86cd799439012"]
  }
}
```

Error responses:
- `400`: Missing or invalid room name
- `401`: Missing or invalid token
- `500`: Failed to generate unique room code

#### Join Room

Endpoint: `POST /api/rooms/join`  
Authentication: Required

Request body:

```json
{
  "roomCode": "ABC123"
}
```

Response (200):

```json
{
  "success": true,
  "room": {
    "id": "507f1f77bcf86cd799439011",
    "name": "General Discussion",
    "code": "ABC123",
    "users": ["507f1f77bcf86cd799439012", "507f1f77bcf86cd799439013"]
  }
}
```

Error responses:
- `400`: Missing room code
- `401`: Missing or invalid token
- `404`: Room with specified code does not exist

Notes:
- Room codes are case-insensitive
- If user is already a member, existing room data is returned
- User is automatically added to room users

#### List Rooms

Endpoint: `GET /api/rooms`  
Authentication: Required

Response (200):

```json
{
  "success": true,
  "rooms": [
    {
      "id": "507f1f77bcf86cd799439011",
      "name": "General Discussion",
      "code": "ABC123",
      "memberCount": 5,
      "createdAt": "2024-01-15T10:30:00.000Z"
    }
  ]
}
```

Error responses:
- `401`: Missing or invalid token

Notes:
- Rooms are sorted by most recently updated first
- Empty array is returned if user is not in any room

#### Get Room Members

Endpoint: `GET /api/rooms/:roomId/users`  
Authentication: Required

Path parameters:
- `roomId` (string): MongoDB ObjectId

Response (200):

```json
{
  "success": true,
  "room": {
    "id": "507f1f77bcf86cd799439011",
    "name": "General Discussion",
    "code": "ABC123"
  },
  "members": [
    {
      "id": "507f1f77bcf86cd799439012",
      "username": "john_doe",
      "joinedAt": "2024-01-15T10:30:00.000Z"
    }
  ]
}
```

Error responses:
- `400`: Invalid roomId format
- `401`: Missing or invalid token
- `403`: User is not a room member
- `404`: Room does not exist

---

### Message Endpoints

#### Get Messages

Endpoint: `GET /api/messages/:roomId`  
Authentication: Required

Path parameters:
- `roomId` (string): MongoDB ObjectId

Query parameters:
- `page` (number, optional): default `1`
- `limit` (number, optional): default `50`, max `100`

Response (200):

```json
{
  "success": true,
  "messages": [
    {
      "id": "507f1f77bcf86cd799439014",
      "roomId": "507f1f77bcf86cd799439011",
      "senderId": "507f1f77bcf86cd799439012",
      "senderUsername": "john_doe",
      "text": "Hello everyone!",
      "createdAt": "2024-01-15T12:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "hasMore": false
  }
}
```

Error responses:
- `400`: Invalid roomId format
- `401`: Missing or invalid token
- `403`: User is not a room member
- `404`: Room does not exist

Notes:
- Messages are sorted by most recent first (`createdAt` descending)
- `hasMore` indicates additional pages

---

## WebSocket Events

### Connection

URL: `ws://localhost:5000` or `http://localhost:5000`

Authentication via handshake token:

```javascript
const io = require('socket.io-client');

const socket = io('http://localhost:5000', {
  auth: {
    token: 'YOUR_JWT_TOKEN'
  }
});

socket.on('connect', () => {
  console.log('Connected to chat server');
});

socket.on('connect_error', (error) => {
  console.error('Connection failed:', error.message);
});
```

Connection errors:
- `Unauthorized`: Missing or invalid token

### Events from Client to Server

#### join_room

Payload:

```javascript
{
  roomId: "507f1f77bcf86cd799439011"
}
```

Success: broadcasts `user_joined` to the room.  
Error: emits `socket_error` to sender.

Error codes:
- `INVALID_ROOM_ID`
- `ROOM_NOT_FOUND`
- `UNAUTHORIZED`
- `INTERNAL_ERROR`

#### send_message

Payload:

```javascript
{
  roomId: "507f1f77bcf86cd799439011",
  text: "Hello everyone!"
}
```

Success: broadcasts `receive_message` to room (including sender).  
Error: emits `socket_error` to sender.

Validation:
- Text is trimmed
- Text length must be 1-5000
- Sender must be room member

Error codes:
- `INVALID_ROOM_ID`
- `INVALID_MESSAGE`
- `MESSAGE_TOO_LONG`
- `ROOM_NOT_FOUND`
- `UNAUTHORIZED`
- `INTERNAL_ERROR`

### Events from Server to Client

#### user_joined

Payload:

```javascript
{
  roomId: "507f1f77bcf86cd799439011",
  userId: "507f1f77bcf86cd799439012",
  username: "john_doe",
  joinedAt: "2024-01-15T12:00:00.000Z"
}
```

#### receive_message

Payload:

```javascript
{
  id: "507f1f77bcf86cd799439014",
  roomId: "507f1f77bcf86cd799439011",
  senderId: "507f1f77bcf86cd799439012",
  senderUsername: "john_doe",
  text: "Hello everyone!",
  timestamp: "2024-01-15T12:00:00.000Z"
}
```

#### socket_error

Payload:

```javascript
{
  code: "INVALID_ROOM_ID",
  message: "Invalid roomId"
}
```

#### disconnect

Client-side event when connection is lost.

---

## Data Models

### User

Collection: `users`

```javascript
{
  _id: ObjectId,
  username: String,
  password: String,
  createdAt: Date,
  updatedAt: Date
}
```

Index: `username` unique.

### Room

Collection: `rooms`

```javascript
{
  _id: ObjectId,
  name: String,
  code: String,
  users: [ObjectId],
  createdAt: Date,
  updatedAt: Date
}
```

Index: `code` unique.

### Message

Collection: `messages`

```javascript
{
  _id: ObjectId,
  roomId: ObjectId,
  senderId: ObjectId,
  text: String,
  createdAt: Date
}
```

Indexes:
- `roomId`
- `{ roomId: 1, createdAt: -1 }`

---

## Error Handling

Error response format:

```json
{
  "success": false,
  "error": "Error message describing what went wrong"
}
```

HTTP status codes:
- `200`: OK
- `201`: Created
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Not Found
- `500`: Internal Server Error

---

## Rate Limiting and Security

Recommendations before production:

1. Add rate limiting for REST and Socket.IO events.
2. Configure request body size limits.
3. Add Helmet security headers.
4. Sanitize user input.
5. Add refresh token flow.
6. Add structured logging.
7. Move secrets to a proper secret manager.

---

## Environment Variables

Required configuration:

```bash
PORT=5000
MONGODB_URI=mongodb://localhost:27017
MONGODB_DB_NAME=chat_db
JWT_SECRET=your-super-secret-key-min-32-chars
FRONTEND_ORIGIN=http://localhost:3000
```

---

## Health Check

Endpoint: `GET /health`  
Authentication: Not required

Response (200):

```json
{
  "success": true,
  "status": "ok"
}
```

---

## Version

API Version: 1.0.0  
Last Updated: 2026  
Server Package: fchat-server
