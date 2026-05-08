# Nagar-Connect

Public-transportation management and real-time bus-tracking portal (frontend + Node/Express backend services). This repository contains a Vite + React + TypeScript frontend and three small Express/MongoDB backend servers used by the Customer/Driver/Municipality portals.

---

## Key features

- Real-time bus tracking and active bus management
- Role-based portals (Customer, Driver, Municipality)
- Bus schedules, stops, and travel-time calculation (uses Google Maps APIs)
- Complaint submission and municipality complaint review
- Emergency (SOS) alerts delivered via Socket.IO (driver → municipality)

---

## Tech stack

- Frontend: Vite, React 18, TypeScript, Tailwind CSS, shadcn-style UI components
- State & data: @tanstack/react-query
- Maps: @react-google-maps/api and Google Maps Web APIs (geocoding & directions)
- Backend: Node.js (CommonJS), Express, Mongoose (MongoDB)
- Real-time: Socket.IO (driver server hosts Socket.IO; frontend uses socket.io-client)

---

## Repository layout (important files)

- `package.json` - Frontend (root) dependencies and scripts
- `vite.config.ts` - Vite configuration and path aliases
- `src/` - Frontend source code (pages, components, hooks)
- `backend/` - Backend servers (Express + Mongoose)
	- `backend/index.js` - Backend entry (starts the municipality server)
	- `backend/server/municipality.js` - Municipality REST API (default port 3000)
	- `backend/server/driver.js` - Driver REST API + Socket.IO (default port 3001)
	- `backend/server/passenger.js` - Passenger REST API (default port 3002)
	- `backend/models/` - Mongoose models (Bus, Driver, Municipality, activebuses/Tracking, Complaint, Emergency)

---

## Local development (recommended)

Prerequisites:

- Node.js (16+ recommended)
- npm or yarn
- MongoDB (local or remote) running and accessible

Run frontend (from repo root):

```bash
npm install
npm run dev
```

This starts the Vite development server (default configured to host on all interfaces and port `8080`).

Run backend (in a separate terminal):

```bash
cd backend
npm install
node index.js
```

That starts the municipality server (default port `3000`).

Start the driver server (Socket.IO + REST) in another terminal:

```bash
cd backend
node server/driver.js
```

Start the passenger server in another terminal:

```bash
cd backend
node server/passenger.js
```

Ports used in the current frontend code:

- Frontend: `http://localhost:8080`
- Municipality API: `http://localhost:3000`
- Driver API + Socket.IO: `http://localhost:3001`
- Passenger API: `http://localhost:3002`

Notes:

- `backend/package.json` contains a `start` script that runs `index.js` (municipality server). Its `dev` script currently points at `server/api.js` (which is not present in this repo).
- MongoDB URI handling differs by server:
	- Driver server: uses `process.env.MONGO_URI` with a local fallback.
	- Municipality + Passenger servers: currently use a hard-coded local URI.
- Google Maps Web API keys are currently hard-coded in backend server files. Replace them with `process.env.GOOGLE_API_KEY` before deploying.

Recommended `.env` variables (create `backend/.env`):

- `MONGO_URI` — MongoDB connection string (example: `mongodb://127.0.0.1:27017/realtime-bus-tracking`)
- `PORT` — Server port (each server has its own default; see below)
- `JWT_SECRET` — Secret used by the driver server for JWT tokens
- `GOOGLE_API_KEY` — Google Maps API key (geocoding & directions)

---

## Backend APIs (what the frontend calls)

The frontend currently calls these services directly (hard-coded `localhost` URLs in `src/`):

### Municipality server (default `:3000`)

Implemented in `backend/server/municipality.js`.

- `GET /api/municipality/buses` — Returns all buses with `isactive` flag
- `POST /api/municipality/buses` — Create a bus with schedules; geocodes stops and calculates times using Google APIs
- `GET /api/municipality/all-schedules` — Returns compiled schedules for all buses (includes `isactive` status)
- `GET /api/municipality/bus-location-count` — Analytics (counts occurrences of stops/locations)
- `GET /api/municipality/complaints` — Returns complaints
- `POST /api/municipality/login` — Municipality login (expects `name` and `password`)

### Driver server (default `:3001`)

Implemented in `backend/server/driver.js`.

REST endpoints:

- `POST /api/driver/register`
- `POST /api/driver/login`
- `GET /api/driver/profile` (requires `Authorization: Bearer <token>`)
- `POST /api/driver/start-journey`
- `POST /api/driver/deactivate-bus/:busNumberPlate`
- `GET /api/driver/bus-routes?busNumberPlate=...`

Socket.IO events (same host/port):

- Client → server: `broadcast` (location updates)
- Client → server: `businfo` (join a bus “room”)
- Client → server: `SOS_info` (emergency alert)
- Client → server: `subscribe_emergency` (municipality subscribes to emergency room)
- Server → clients: `broadcast_response`, `businfo_response`, `emergency_alert`, `emergency_subscribed`

### Passenger server (default `:3002`)

Implemented in `backend/server/passenger.js`.

- `GET /api/passenger/search-buses?day=...&startingPlace=...&destination=...`
- `GET /api/passenger/places`
- `GET /api/passenger/bus-number-plates`
- `GET /api/passenger/bus/:busnumberplate/places`
- `GET /api/passenger/bus/:busnumberplate/schedule/:scheduleId`
- `GET /api/passenger/all-schedules`
- `POST /api/passenger/complaint`

---

## Security & hardening notes

- There are Google API keys present in the backend source. Do not commit API keys; move them to `backend/.env` and reference via `process.env.GOOGLE_API_KEY`.
- Passwords are hashed for driver registration and municipality login compares hashed passwords, but you should audit authentication flows and token management before deploying.
- CORS is pre-configured with several localhost origins for development. Adjust to allow only trusted origins for production.

---

## Development tips

- Frontend dev server port is set in `vite.config.ts` (port `8080`). Backend CORS lists several localhost origins including 8080/3000/3001/3002.
- The project uses an alias `@` that maps to `./src`; import paths in the frontend use `@/...`.
- To seed sample buses or run integration tests, create scripts that POST to `/api/municipality/buses` with a properly structured bus object (see `backend/models/Bus.js` for the schema).

---

## Contributing

- This repository appears to be an application scaffold with frontend + backend. If you'd like help adding tests, CI, or dockerization for local reproducible environments, open an issue or submit a PR.

---

## License

This repository has no license file. Add a `LICENSE` file if you intend to publish this project.



