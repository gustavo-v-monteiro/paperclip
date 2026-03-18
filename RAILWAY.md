# Deploying Paperclip on Railway

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/gustavo-v-monteiro/paperclip)

## Prerequisites

- A [Railway](https://railway.app) account
- The [Railway CLI](https://docs.railway.app/develop/cli) (optional, for local workflows)

## Services needed

You need two Railway services:

1. **App service** — built from the `Dockerfile` in this repo.
2. **PostgreSQL service** — provided by the Railway PostgreSQL plugin.

## Steps

### 1. Create a new Railway project

In the Railway dashboard, click **New Project** → **Deploy from GitHub repo** and select this repository. Railway will detect the `Dockerfile` automatically.

Alternatively, add a PostgreSQL service first and then add the app service from the same project.

### 2. Add a PostgreSQL service

In your Railway project, click **New** → **Database** → **Add PostgreSQL**. Railway will automatically inject a `DATABASE_URL` variable into your app service when you link the two services.

### 3. Configure environment variables

In the app service's **Variables** tab, set the following (see `.env.example` for descriptions):

| Variable | Required | Notes |
|---|---|---|
| `DATABASE_URL` | ✅ | Injected automatically when linked to the Railway PostgreSQL service |
| `PORT` | — | Defaults to `3100`; Railway overrides this automatically |
| `HOST` | — | Set to `0.0.0.0` (already the default in the Docker image) |
| `SERVE_UI` | — | Set to `true` to serve the built UI from the server |
| `NODE_ENV` | — | Set to `production` |
| `PAPERCLIP_HOME` | — | Defaults to `/paperclip` |
| `PAPERCLIP_INSTANCE_ID` | — | Defaults to `default` |
| `PAPERCLIP_CONFIG` | — | Defaults to `/paperclip/instances/default/config.json` |
| `PAPERCLIP_DEPLOYMENT_MODE` | — | `authenticated` or `local_trusted` |
| `PAPERCLIP_DEPLOYMENT_EXPOSURE` | — | `private` or `public` |
| `PAPERCLIP_PUBLIC_URL` | ✅ | Set to your Railway-generated domain (e.g. `https://paperclip-production.up.railway.app`) after deploy |
| `BETTER_AUTH_SECRET` | ✅ | Generate a strong random secret (e.g. `openssl rand -hex 32`) |

### 4. Add a persistent volume

Paperclip stores instance configuration and data under `/paperclip`. To persist this across deploys and restarts, attach a Railway volume:

1. In the app service, go to **Settings** → **Volumes**.
2. Click **New Volume**, set the mount path to `/paperclip`, and save.

Without a volume, any configuration written to `/paperclip` will be lost when the container restarts.

### 5. Deploy

Trigger a deploy from the Railway dashboard or by pushing to your connected branch. Railway will build the Docker image, run the health check at `/health`, and start the service.

### 6. Post-deploy: set `PAPERCLIP_PUBLIC_URL`

Once the service is running, copy the generated Railway domain from the **Settings** tab and set it as the `PAPERCLIP_PUBLIC_URL` environment variable. Redeploy the service for the change to take effect.

## Health check

Railway uses `GET /health` to verify the service is healthy. The endpoint is configured in `railway.toml`.
