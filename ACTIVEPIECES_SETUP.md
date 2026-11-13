# Activepieces connection setup for SaarthiFlow

This project is a React + Vite frontend plus an Express backend. The frontend alone cannot safely store an Activepieces webhook URL. The Express backend forwards requests to Activepieces.

## 1. Publish the Activepieces flow

In Activepieces, publish the flow containing the **Catch Webhook** trigger. Keep the Live URL private. The flow currently triggers asynchronously unless you configure a synchronous webhook response in Activepieces.

## 2. Deploy the existing Express backend

Deploy the repository's existing Node.js backend (the `server/` folder and root `package.json`) to a Node host such as Render or Railway. Configure its start command as `npm start` (or `npm run server`). Set these backend environment variables as appropriate:

- `ACTIVEPIECES_WEBHOOK_URL`: your Catch Webhook Live URL. Do not add a `VITE_` prefix.
- `CLIENT_URL`: your exact deployed SaarthiFlow Vercel origin, e.g. `https://your-project.vercel.app` (no trailing slash).
- `GEMINI_API_KEY` and Supabase variables if your existing app needs them.

The backend must be publicly reachable over HTTPS. Do not deploy the Express backend as a static Vite site.

## 3. Point Vercel frontend at the backend

In Vercel → Project → Settings → Environment Variables, set:

- `VITE_API_URL` = `https://YOUR-BACKEND-HOST/api`

Apply it to Production (and Preview if needed), then redeploy the frontend. This is important because `src/api.js` otherwise defaults to `http://localhost:3001/api`, which only works on your own computer.

## 4. Test

Open `https://YOUR-BACKEND-HOST/api/activepieces/status`. It should return JSON with `connected: true` once the webhook URL is configured.

From the frontend code, call:

```js
import { api } from './api';

const result = await api.executeActivepieces({
  task: 'email_analysis',
  email: 'Example email content'
});
```

You can also check the backend endpoint with a POST request to `/api/activepieces/execute` using a JSON body. Then check Activepieces → Runs.

## Important behavior and security

- The webhook URL stays in the backend environment, not in Vite/browser variables.
- `connected: true` means the webhook URL is configured; it does **not** query Activepieces to count flows.
- A normal Catch Webhook call usually acknowledges the request rather than returning the final result of later flow steps. For a final synchronous response, configure Activepieces' synchronous webhook mode and a Return Response step, and account for the platform's timeout.
- Protect the public execute endpoint with your app's authentication/authorization and rate limiting before exposing it to untrusted users. The starter route is a proxy, not a complete user-auth system.
- Never commit `.env` or share API keys in chat.
