# Frappe/ERPNext Docker setup

Docker-based setup for Frappe/ERPNext: a **base image** (Frappe + core apps) and an optional **custom image** (base + custom apps). Includes a **pwd.yml** stack compatible with [official frappe_docker](https://github.com/frappe/frappe_docker) for local/dev use.

## Project structure

| Path | Purpose |
|------|--------|
| `Dockerfile.base` | Base image: Frappe + optional core apps (ERPNext, Education, HRMS, Helpdesk, etc.). |
| `Dockerfile.custom` | Custom image: extends base, installs apps from `resources/custom/apps.json` and builds only those apps’ assets. |
| `pwd.yml` | Compose stack (backend, frontend, configurator, create-site, queue, scheduler, websocket, db, redis). |
| `resources/base/apps.json` | Core apps for the base image (url + branch per app). |
| `resources/custom/apps.json` | Custom apps for the custom image (url, branch, optional `name` for asset build). |
| `resources/core/nginx/` | Nginx template and entrypoint for the frontend service. |

## Build requirements

- **Internet** during build: image builds clone Frappe and apps from GitHub (and any Git hosts in your app lists). If you see `Failed to connect to github.com port 443`, fix outbound HTTPS for Docker.

## Quick start

Run all commands from the repo root.

### 1. Build base image

Installs Frappe and the apps listed in `resources/base/apps.json`.

**Windows (PowerShell):**
```powershell
$baseB64 = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes((Get-Content -Raw resources\base\apps.json)))
docker build -f Dockerfile.base --build-arg BASE_APPS_JSON_BASE64="$baseB64" -t frappe-base:latest .
```

**Linux / macOS:**
```bash
docker build -f Dockerfile.base --build-arg BASE_APPS_JSON_BASE64="$(base64 -w0 resources/base/apps.json)" -t frappe-base:latest .
```

**Frappe only (no base apps):**  
`docker build -f Dockerfile.base -t frappe-base:latest .`

### 2. Build custom image (optional)

Adds apps from `resources/custom/apps.json`. Each entry can include optional `"name"` (bench app name); when present, only those apps’ assets are built (no full `bench build`).

**Windows (PowerShell):**
```powershell
$customB64 = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes((Get-Content -Raw resources\custom\apps.json)))
docker build -f Dockerfile.custom --build-arg BASE_IMAGE=frappe-base:latest --build-arg APPS_JSON_BASE64="$customB64" -t frappe-custom:latest .
```

**Linux / macOS:**
```bash
docker build -f Dockerfile.custom --build-arg BASE_IMAGE=frappe-base:latest --build-arg APPS_JSON_BASE64="$(base64 -w0 resources/custom/apps.json)" -t frappe-custom:latest .
```

**No custom apps:**  
`docker build -f Dockerfile.custom --build-arg BASE_IMAGE=frappe-base:latest -t frappe-custom:latest .`

### 3. Run the stack (pwd.yml)

Uses **frappe-base:latest** by default. To use the custom image:

```bash
# Default: base image
docker compose -f pwd.yml up -d

# Custom image
CUSTOM_IMAGE=frappe-custom CUSTOM_TAG=latest docker compose -f pwd.yml up -d
```

Wait **~10–15 minutes** for create-site to finish (it creates site `frontend` and installs all base + custom apps). Then open **http://localhost:8080** (Administrator / admin).

## Operations

### Create-site already ran: “Site frontend already exists”

To recreate the site and run create-site again (fresh install of all apps):

```bash
docker compose -f pwd.yml run --rm backend bench --site frontend drop-site --force
docker compose -f pwd.yml up -d --force-recreate create-site
docker compose -f pwd.yml logs -f create-site
```

### Install remaining or custom apps on existing site

```bash
# List installed apps
docker compose -f pwd.yml exec backend bench --site frontend list-apps

# Install one or more apps (use your app names from custom/base apps.json)
docker compose -f pwd.yml exec backend bench --site frontend install-app menumate
# Or multiple: install-app education, hrms, helpdesk, dfp_external_storage, frappe_pdf, menumate
```

If the backend container is not running, use a one-off run:

```bash
docker compose -f pwd.yml run --rm backend bench --site frontend install-app menumate
```

### Updating custom app code (e.g. menumate) on the server

1. **Push** your changes to the Git repo/branch referenced in `resources/custom/apps.json`.
2. **Rebuild** the custom image (Step 2 above).
3. **Redeploy** so containers use the new image:
   ```bash
   docker compose -f pwd.yml up -d --force-recreate
   ```
4. **Migrate and clear cache** (and rebuild assets if you changed frontend code):
   ```bash
   docker compose -f pwd.yml exec backend bench --site frontend migrate
   docker compose -f pwd.yml exec backend bench build --app menumate
   docker compose -f pwd.yml exec backend bench --site frontend clear-cache
   ```

## App list format

- **Base / custom:** `[{"url": "https://github.com/org/repo.git", "branch": "main"}]`
- **Custom only:** optional `"name": "app_name"` per entry. When set, only that app’s assets are built (`bench build --app app_name`), so other apps (e.g. helpdesk) are not rebuilt.

## Nginx

The frontend service uses `resources/core/nginx/nginx-template.conf` and `nginx-entrypoint.sh`; the entrypoint fills env vars (e.g. `BACKEND`, `SOCKETIO`) and starts Nginx.

## See also

- **ARCHITECTURE.md** – Image layout, build strategy, and deployment checklist.
