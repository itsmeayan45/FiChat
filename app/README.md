# FChat Frontend (Next.js)

This folder contains the frontend for FChat, a real-time chat app.

- Framework: Next.js (App Router)
- UI: React + Tailwind CSS
- State: Zustand
- Realtime client: Socket.IO client
- Backend dependency: Express + Socket.IO API from `../server`

## Prerequisites

- Node.js 20+
- pnpm 9+
- Running backend server (`../server`)

## Environment Variables

Create `.env.local` in this folder:

```env
NEXT_PUBLIC_API_URL=http://localhost:5000
```

If omitted, the app defaults to `http://localhost:5000`.

## Install

```bash
pnpm install
```

## Run (Development)

```bash
pnpm dev
```

Then open:

- http://localhost:3000

## Build and Start (Production)

```bash
pnpm build
pnpm start
```

## Important Routes

- `/login` - authenticate user
- `/chat` - chat home
- `/chat/[roomId]` - room chat screen

## Related Backend

This frontend expects the backend in `../server` to be running.

Default backend URL: `http://localhost:5000`

## Troubleshooting

### Cannot login or load rooms

- Confirm backend is running on `http://localhost:5000`
- Check `NEXT_PUBLIC_API_URL` in `.env.local`
- Verify backend CORS allows `http://localhost:3000`

### Socket not connecting

- Confirm JWT token exists after login
- Confirm backend Socket.IO server is up
- Check browser console for connection/auth errors
