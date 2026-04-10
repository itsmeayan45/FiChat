# FChat

Real-time chat application with a modern Next.js frontend and an Express + Socket.IO backend.

## Highlights

- Real-time room messaging with Socket.IO
- JWT-based authentication
- Room creation and join-by-code flow
- MongoDB persistence for users, rooms, and messages
- TypeScript frontend with App Router (Next.js)

## Repository Layout

~~~text
fchat/
	app/      # Next.js frontend (UI, state, client sockets)
	server/   # Express API, Socket.IO server, MongoDB models
~~~

## Tech Stack

- Frontend: Next.js, React, Tailwind CSS, Zustand, Axios, Socket.IO client
- Backend: Node.js, Express, Socket.IO, Mongoose, JWT, bcrypt
- Database: MongoDB
- Package manager: pnpm

## Quick Start

### 1. Prerequisites

- Node.js 20+
- pnpm 9+
- MongoDB instance (local or remote)

### 2. Configure backend environment

Create a file named .env inside server/ with:

~~~env
PORT=5000
MONGODB_URI=mongodb://localhost:27017
MONGODB_DB_NAME=chat_db
JWT_SECRET=replace-with-a-long-random-secret
FRONTEND_ORIGIN=http://localhost:3000
~~~

### 3. Configure frontend environment

Create a file named .env.local inside app/ with:

~~~env
NEXT_PUBLIC_API_URL=http://localhost:5000
~~~

### 4. Install dependencies

~~~bash
cd server
pnpm install

cd ../app
pnpm install
~~~

### 5. Run in development

Use two terminals.

Terminal 1:

~~~bash
cd server
pnpm dev
~~~

Terminal 2:

~~~bash
cd app
pnpm dev
~~~

Open http://localhost:3000

## Production Run

Build and start each app separately.

Backend:

~~~bash
cd server
pnpm start
~~~

Frontend:

~~~bash
cd app
pnpm build
pnpm start
~~~

## Available Scripts

### app

- pnpm dev
- pnpm build
- pnpm start
- pnpm lint

### server

- pnpm dev
- pnpm start
- pnpm create:user

## API and Docs

- Backend API docs are in server/README.md
- Frontend setup docs are in app/README.md

## Current Architecture

The project is currently split into two deployable apps:

- app: frontend client
- server: backend API + WebSocket server

This is a clean modular base and can later be migrated into a monolith if desired.

## Roadmap Ideas

- Monolithic deployment option (single process)
- Message reactions and read receipts
- Presence indicators and typing status
- Rate limiting and enhanced security middleware
- CI checks and automated tests

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Open a pull request with a clear summary
