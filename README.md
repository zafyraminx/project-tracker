# Client Project Tracker

A small full-stack app for tracking client projects at a digital agency.
Express REST API on the backend, React on the frontend, SQLite for storage.

```
project-tracker/
├── backend/          Node.js + Express API (SQLite)
├── frontend/         React + Vite SPA
└── test_data.json    Sample projects used to seed the database
```

## Technology choices

- **Node.js + Express** for the API. No framework magic, easy to read top to bottom.
- **SQLite via `node:sqlite`** (built into Node 22). No database server to install,
  no native compilation, and the whole DB lives in one file at `backend/data/tracker.db`.
- **React + Vite** for the frontend. Vite's dev server proxies `/api/*` to the
  backend so there's no CORS setup needed in development.
- Plain CSS, no component library. The styling follows a dark theme based on the
  [Nexbit](https://nexbit-temlis.webflow.io) template.

## Setup

You need Node.js **22.5 or newer** (for the built-in `node:sqlite` module). Nothing else.

```bash
# terminal 1 — backend
cd backend
npm install
npm start            # http://localhost:3001

# terminal 2 — frontend
cd frontend
npm install
npm run dev          # http://localhost:5173
```

Open http://localhost:5173. Both servers need to be running.

On first start the backend creates `backend/data/tracker.db` and seeds it with
the 12 sample projects from `test_data.json`. To reset to a clean slate, stop
the server, delete `backend/data/tracker.db`, and start it again.

Other useful commands:

```bash
cd backend && npm run dev       # backend with auto-restart on file changes
cd frontend && npm run build    # production bundle in frontend/dist
cd frontend && npm run preview  # serve the built bundle at http://localhost:4173
```

## How to use it

The main screen lists every project as a card with the client, name, description,
dates, a priority tag, and a status badge. The toolbar handles search (matches
client, project name, or description), status/priority filters, and sorting.
"+ New Project" opens the create form; each card has Edit and Delete.

## API

Base URL: `http://localhost:3001`

| Method | Path            | Description                     |
|--------|-----------------|---------------------------------|
| GET    | `/projects`     | List projects                   |
| GET    | `/projects/:id` | Get one project                 |
| POST   | `/projects`     | Create a project                |
| PUT    | `/projects/:id` | Update a project (partial body OK) |
| DELETE | `/projects/:id` | Delete a project                |

`GET /projects` takes optional query params: `search`, `status`, `priority`,
`sortBy` (`id`, `client_name`, `project_name`, `status`, `priority`,
`start_date`, `due_date`, `created_at`) and `order` (`asc`/`desc`).

Fields: `client_name` (required), `project_name` (required), `description`,
`status` (`Planning` / `In Progress` / `On Hold` / `Completed`), `priority`
(`Low` / `Medium` / `High`), `start_date` and `due_date` (`YYYY-MM-DD`).

Validation errors come back as 400 with details per field:

```json
{ "error": "Validation failed.", "details": { "client_name": "Client Name is required." } }
```

Quick smoke test:

```bash
curl http://localhost:3001/projects

curl -X POST http://localhost:3001/projects \
  -H "Content-Type: application/json" \
  -d '{"client_name":"Acme Co","project_name":"New Site","priority":"High"}'
```

## Assumptions

- Single user, no authentication. This is an internal tool for one agency,
  so there are no accounts, roles, or login.
- SQLite is enough. The dataset is small (dozens to hundreds of projects,
  one writer at a time), so a file-based DB beats running a database server.
- Status and priority are fixed enums. New values would need a code change,
  which seemed acceptable for how rarely those lists change.
- Dates are plain `YYYY-MM-DD` strings with no timezone handling. Due date
  can't be earlier than start date; both are optional.
- `PUT /projects/:id` accepts a partial body and merges it onto the existing
  row (the merged record is still fully validated). This matches how the edit
  form sends only what changed.
- The seed only runs when the projects table is empty, so it never overwrites
  data you've entered.
- Ports 3001 (API) and 5173 (dev server) are assumed free.