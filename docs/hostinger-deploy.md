# Hostinger Deployment (Option A: Prebuilt Images)

This setup builds and pushes 3 images from GitHub Actions to GHCR:

- `ghcr.io/<owner>/fam/api`
- `ghcr.io/<owner>/fam/web`
- `ghcr.io/<owner>/fam/web-admin`

Hostinger then pulls images using `docker-compose.prod.yml`, so it does not need submodule build contexts.

## 1) GitHub repository settings

Set these repository **Variables**:

- `NEXT_PUBLIC_API_URL` (example: `https://api.your-domain.com`)

Set these repository **Secrets**:

- `HOSTINGER_HOST` - VPS IP or host
- `HOSTINGER_USER` - SSH user
- `HOSTINGER_SSH_KEY` - private SSH key for the VPS
- `HOSTINGER_APP_PATH` - path on VPS where compose files live (example: `/opt/fam`)
- `HOSTINGER_GHCR_USER` - GitHub username that can read GHCR packages
- `HOSTINGER_GHCR_TOKEN` - GitHub PAT with `read:packages`

## 2) VPS files

Upload to VPS app path (`HOSTINGER_APP_PATH`):

- `docker-compose.prod.yml`
- `.env`

Example `.env`:

```env
GHCR_OWNER=your-github-username
IMAGE_TAG=latest

POSTGRES_USER=family
POSTGRES_PASSWORD=change-this
POSTGRES_DB=family

DATABASE_URL=postgresql://family:change-this@db:5432/family?schema=public
PORT=3001
NODE_ENV=production
JWT_EXPIRES_IN=1d
JWT_REFRESH_EXPIRES_IN=7d
JWT_SECRET=change-this
JWT_REFRESH_SECRET=change-this-too
```

## 3) Workflow behavior

- On push to `main`, workflow builds all 3 images and pushes to GHCR.
- If Hostinger secrets are set, it SSHes to VPS and runs:
  - `docker compose -f docker-compose.prod.yml pull`
  - `docker compose -f docker-compose.prod.yml up -d`

## 4) First-time VPS bootstrap (manual)

Run once on VPS:

```bash
docker login ghcr.io -u <github-user>
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
```
