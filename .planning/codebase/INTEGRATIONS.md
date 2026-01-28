# External Integrations

**Analysis Date:** 2026-01-28

## APIs & External Services

**Fonts:**
- Fonts.bunny.net - CDN for web fonts (Inter, JetBrains Mono)
  - Source: `layouts/partials/head.html`
  - URL: `https://fonts.bunny.net/css?family=inter:400,500,600,700|jetbrains-mono:400,500`

**Container Registry:**
- GitHub Container Registry (ghcr.io) - Docker image hosting
  - Image: `ghcr.io/danielfrevel/dande:latest`
  - Authentication: GitHub OIDC token via Actions
  - Reference: `.github/workflows/deploy.yml`

## Data Storage

**Databases:**
- None - Static site, no database

**File Storage:**
- Local filesystem only - All content stored in git (`content/` directory)
- Generated static files: `public/` directory (not committed)

**Caching:**
- Nginx static cache configured for:
  - Assets (CSS, JS, images): 1 year immutable cache
  - HTML pages: 1 hour with must-revalidate
  - Pagefind search index: 1 day
  - RSS feed: 1 hour
  - Configuration: `nginx.conf`

## Authentication & Identity

**Auth Provider:**
- None - Public blog with no auth requirements
- Deployment auth: GitHub Actions secrets for SSH key and server credentials

## Monitoring & Observability

**Health Checks:**
- Docker container health check: HTTP GET to `http://localhost/` every 30s
  - Implementation: `Dockerfile` HEALTHCHECK directive
  - Tool: wget

**Error Tracking:**
- None configured

**Logs:**
- Docker logs via compose - captured from nginx access/error logs
- No external log aggregation

## CI/CD & Deployment

**Hosting:**
- Custom VPS via SSH deploy (address: `${{ secrets.SERVER_HOST }}`)
  - SSH port: 2222
  - SSH key: `${{ secrets.SERVER_SSH_KEY }}`
  - SSH user: `${{ secrets.SERVER_USER }}`
  - Deploy path: `/opt/docker/blog`

**CI Pipeline:**
- GitHub Actions
- Triggers: Push to main branch, manual workflow_dispatch
- Build steps:
  1. Docker Buildx for multi-platform builds
  2. Log in to ghcr.io with GitHub OIDC token
  3. Build and push Docker image with GitHub Actions cache
  4. SSH into server and run docker compose pull/up on successful build
  - Workflow file: `.github/workflows/deploy.yml`

**Reverse Proxy & SSL/TLS:**
- Traefik (external docker network)
  - Domain: `blog.danfrevel.de`
  - Entry point: websecure
  - TLS resolver: letsencrypt (automatic certificate generation)
  - HSTS enabled: 31536000 seconds (1 year)
  - Network: `traefik` (external, shared with reverse proxy container)
  - Configuration: `docker-compose.yml` labels

## Environment Configuration

**Required env vars:**
- `SERVER_HOST` - Target VPS hostname for deployment
- `SERVER_USER` - SSH user for deployment
- `SERVER_SSH_KEY` - Private SSH key for authentication
- GitHub OIDC automatically provides `GITHUB_TOKEN` for registry login

**Secrets location:**
- GitHub Actions Secrets (.github/workflows/deploy.yml references)

**Build Configuration:**
- Hugo settings: `hugo.toml` for site metadata, RSS, syntax highlighting, menus
- Tailwind JIT scanning: `tailwind.config.js` scans `layouts/` and `content/` for class names
- Dark mode: CSS class strategy (manually toggled by theme switcher)

## Webhooks & Callbacks

**Incoming:**
- None - Static site generation triggered by push events only

**Outgoing:**
- RSS feed generated: `index.xml` (standard Hugo RSS output)
- Static site content delivery via Nginx

## External Dependencies

**Hugo Module System:**
- Not used - pure Hugo configuration

**NPM Registry:**
- npm packages sourced from registry.npmjs.org
- Lock file enforces exact versions

---

*Integration audit: 2026-01-28*
