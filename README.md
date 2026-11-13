# SaarthiFlow

SaarthiFlow turns goals and webpage context into task plans that users can review and approve. The app includes a React dashboard, an Express API, Gemini-powered planning, and an optional Activepieces webhook connection.

## Features

- Generate task proposals from natural-language instructions and public webpage URLs.
- Edit, approve, or reject proposals before tasks are changed.
- Manage tasks, projects, deadlines, activity, and analytics.
- Send a test event from **Settings** to a configured Activepieces flow.
- Store data in Supabase or use the local JSON store for development.

## Stack

- React and Vite
- Node.js and Express
- Google Gemini API
- Supabase (optional)
- Activepieces Catch Webhook (optional)

## Run locally

Requirements: Node.js 18+ and npm.

1. Install dependencies:

   ```sh
   npm ci
   ```

2. Copy `.env.example` to `.env` in the project root and set the values you need. At minimum, `VITE_API_URL` should point to the local API:

   ```env
   PORT=3001
   CLIENT_URL=http://localhost:5173
   VITE_API_URL=http://localhost:3001/api
   ```

   Add a Gemini key to enable Gemini planning:

   ```env
   GEMINI_API_KEY=your_gemini_api_key
   GEMINI_MODEL=gemini-1.5-flash
   ```

   Without a Gemini key, the API uses its deterministic fallback planner.

3. Start the backend and frontend in separate terminals:

   ```sh
   npm run server
   ```

   ```sh
   npm run dev
   ```

4. Open the Vite URL shown in the frontend terminal. The backend health endpoint is `http://localhost:3001/api/health`.

Create `.env` from the example using PowerShell:

```powershell
Copy-Item .env.example .env
```

## Connect Activepieces

SaarthiFlow calls Activepieces through its Express backend; the webhook URL is not exposed to browser code.

1. In Activepieces, create a flow with a **Catch Webhook** trigger and publish it.
2. Set `ACTIVEPIECES_WEBHOOK_URL` in the backend environment to the flow's live webhook URL.
3. Start or redeploy the backend.
4. Open SaarthiFlow **Settings**. The Activepieces status should show as configured. **Send test event** sends a JSON event to the flow and may run its actions.

The test event has this shape:

```json
{
  "source": "SaarthiFlow",
  "event": "connection_test",
  "timestamp": "ISO-8601 timestamp"
}
```

An Activepieces API key is not needed for this incoming webhook integration. Never put webhook URLs or API keys in `VITE_*` variables, frontend code, or a committed `.env` file.

## Environment variables

| Variable | Used by | Purpose |
| --- | --- | --- |
| `PORT` | Backend | Express server port; defaults to `3001` |
| `CLIENT_URL` | Backend | Allowed deployed frontend origin for CORS |
| `GEMINI_API_KEY` | Backend | Optional Gemini API key |
| `GEMINI_MODEL` | Backend | Gemini model; defaults to `gemini-1.5-flash` |
| `ACTIVEPIECES_WEBHOOK_URL` | Backend | Optional published Activepieces Catch Webhook URL |
| `SUPABASE_URL` | Backend | Optional Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Backend | Optional server-side Supabase service key |
| `SUPABASE_ANON_KEY` | Backend | Optional Supabase anon key |
| `VITE_API_URL` | Frontend build | Backend API base URL, including `/api` |

When Supabase is not configured, the backend uses `server/data/db.json`. Use Supabase or another persistent store when deploying to an environment with an ephemeral filesystem.

## Deploy

Deploy the Vite frontend and Express backend separately:

1. Deploy the backend to a Node.js host with start command `npm start`.
2. Set backend environment variables there, including `CLIENT_URL` and `ACTIVEPIECES_WEBHOOK_URL` if needed.
3. In the Vercel frontend project, set `VITE_API_URL` to `https://YOUR-BACKEND-HOST/api`, then redeploy.

The backend must be publicly reachable over HTTPS for the hosted frontend to use it. The Activepieces execute endpoint is not protected by application authentication or rate limiting in this starter project; add appropriate access controls before using it with sensitive or high-impact flows.

## Useful commands

```sh
npm run dev       # Start the Vite frontend
npm run server    # Start the Express backend
npm run build     # Build the frontend for production
npm run preview   # Preview the production frontend build
```
