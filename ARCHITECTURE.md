# Architecture

Short reference for how the images and stack are structured.

## Image layout

- **Base (Dockerfile.base):** Multi-stage. System deps + NVM/Node + bench in one stage; `bench init` (and optional base apps from `BASE_APPS_JSON_BASE64`) in a builder stage; final stage copies only the bench tree (no build tools). Default CMD: gunicorn.
- **Custom (Dockerfile.custom):** `FROM` base. If `APPS_JSON_BASE64` is set: decode `apps.json`, `bench get-app` per entry, `bench setup requirements`, pinned deps, then build only apps that have a `name` in the JSON (`bench build --app <name>` per app), else full `bench build`. Same volumes (sites, sites/assets, logs) and CMD as base for pwd.yml compatibility.

## Versions (defaults)

- Frappe: v15.34.1
- Python: 3.11.6 (slim Bookworm)
- Node: 18.x (NVM)
- App branches are set in `resources/base/apps.json` and `resources/custom/apps.json` (e.g. ERPNext v15.29.3). Keep Frappe and app branches aligned (e.g. all v15).

## Build strategy and cache

- **Base:** Rebuild when Python/Node/Frappe version, `BASE_APPS_JSON_BASE64`, or Nginx COPY changes. Keep version args stable to reuse cache when only base app list changes.
- **Custom:** Rebuild when `BASE_IMAGE` or `APPS_JSON_BASE64` changes. Always build base first, then custom with `--build-arg BASE_IMAGE=frappe-base:tag`.

## pwd.yml

Same service layout as [official frappe_docker pwd.yml](https://github.com/frappe/frappe_docker/blob/main/pwd.yml): backend, frontend, configurator, create-site, queue-short/queue-long/queue-default, scheduler, websocket, db, redis-cache, redis-queue. Create-site creates site `frontend` and installs all base apps (erpnext, education, hrms, helpdesk, dfp_external_storage, frappe_pdf) plus any custom apps you install on the site. Use `CUSTOM_IMAGE` / `CUSTOM_TAG` to point to your custom image.

## Deployment checklist

| Goal | How |
|------|-----|
| Reproducible builds | Pin Frappe branch and app branches in JSON; use fixed base image tags. |
| One base, many stacks | Build base once; multiple custom images with different `APPS_JSON_BASE64`. |
| No secrets in image | Use build-args only for app lists; inject DB and secrets at runtime (env/volumes). |
| Production | Use same service layout as pwd.yml with your image; mount sites and logs; inject config and Nginx env. |
