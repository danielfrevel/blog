# Codebase Concerns

**Analysis Date:** 2026-01-28

## Deployment Security

**SSH Key Exposure Risk:**
- Issue: SSH private key passed via GitHub Secrets to CI/CD pipeline without rotation mechanism
- Files: `.github/workflows/deploy.yml` (line 64)
- Impact: Compromised key requires manual server access to rotate; no automated key management
- Fix approach: Implement key rotation policy, consider using GitHub Deploy Keys or OIDC authentication instead of static SSH keys for better security posture

**Missing Firewall Configuration on SSH Port:**
- Issue: Server SSH configured on custom port 2222 but no mention of firewall rules
- Files: `.github/workflows/deploy.yml` (line 62), deployment instructions in `USAGE.md`
- Impact: SSH brute force attacks possible if firewall not properly configured
- Recommendations: Document firewall requirements, restrict SSH to GitHub Actions IP range if possible

## Deployment Process Gaps

**No Pre-deployment Health Checks:**
- Issue: Docker compose up runs without validation that old container properly shut down
- Files: `.github/workflows/deploy.yml` (lines 67-68)
- Impact: Race conditions possible between old container stopping and new one starting
- Fix approach: Add explicit healthcheck wait before declaring deployment successful; use docker-compose to wait for health status

**Orphaned Container Cleanup Without Verification:**
- Issue: `docker image prune -f` force-removes images without checking if they're still referenced
- Files: `.github/workflows/deploy.yml` (line 69)
- Impact: If new container fails to start, dangling images are already deleted, making rollback harder
- Fix approach: Move image prune to post-deployment verification step, or use more conservative cleanup (--all-unused)

**Missing Deployment Rollback Strategy:**
- Issue: No rollback mechanism if deployment fails or new version introduces bugs
- Files: `.github/workflows/deploy.yml`
- Impact: Bad deployments require manual SSH intervention to recover
- Fix approach: Tag Docker images with commit SHA, keep previous image available for quick rollback

## Build & Docker Concerns

**Large Package Installation Without Cache Control:**
- Issue: `npm install --prefer-offline --no-audit` in Docker build uses loose caching strategy
- Files: `Dockerfile` (line 8)
- Impact: Build times inconsistent; vulnerable packages may be cached despite --prefer-offline
- Fix approach: Use `npm ci` instead of `npm install` with locked dependencies, add explicit Docker layer caching

**Missing Docker Image Base Hardening:**
- Issue: Using `nginx:alpine` as-is without non-root user or additional hardening
- Files: `Dockerfile` (line 20)
- Impact: Container runs as root by default; any RCE vulnerability gives full container access
- Fix approach: Add USER directive, remove unnecessary packages from nginx image, consider distroless alternative

**Lack of Build Reproducibility:**
- Issue: Hugo version not pinned; uses whatever is in `hugomods/hugo:exts`
- Files: `Dockerfile` (line 2)
- Impact: Different developers/CI runs may build with different Hugo versions, causing inconsistent output
- Fix approach: Pin Hugo version explicitly, either in base image tag or via package manager

## Configuration & Secrets

**Placeholder Configuration Not Cleared:**
- Issue: `hugo.toml` contains placeholder values (baseURL, author name, description)
- Files: `hugo.toml` (lines 1, 15, 14)
- Impact: Example content visible in production if config wasn't updated; confusing for new team members
- Fix approach: Use environment variable substitution or separate config for production, document required changes

**Hard-coded Domain in Traefik Config:**
- Issue: Domain `blog.danfrevel.de` hard-coded in docker-compose.yml labels
- Files: `docker-compose.yml` (line 9)
- Impact: Can't easily deploy to different domains without editing compose file
- Fix approach: Extract domain to environment variable, documented in deployment instructions

**No Environment-specific Configurations:**
- Issue: Single `hugo.toml` used for all environments (dev/staging/prod)
- Files: `hugo.toml`
- Impact: Easy to accidentally build/test with production baseURL or analytics
- Fix approach: Split config by environment (hugo.dev.toml, hugo.prod.toml) or use config directory approach

## Performance & Caching

**Font Loading Not Optimized:**
- Issue: Google Fonts (fonts.bunny.net) loaded synchronously in head without preload hints
- Files: `layouts/partials/head.html` (line 59)
- Impact: Blocks rendering until fonts load; no fallback specified
- Fix approach: Add font-display: swap, preload critical fonts, consider system fonts fallback

**No Service Worker or Static Asset Versioning:**
- Issue: CSS/JS fingerprinting used but no service worker for offline capability
- Files: `layouts/partials/head.html` (lines 62-67)
- Impact: Users must have network connectivity; no progressive enhancement for offline reading
- Fix approach: Add basic service worker for offline-first caching strategy

**Search Index Not Cached Efficiently:**
- Issue: Pagefind search index served with 1-day cache while static assets get 1-year
- Files: `nginx.conf` (line 58)
- Impact: Search results lag by up to 24 hours on updates; inconsistent with other caching strategy
- Fix approach: Either short cache on all assets or implement cache busting via versioned paths

## Frontend Concerns

**Dark Mode Flash Risk:**
- Issue: Dark mode script in head may still cause FOUC (Flash of Unstyled Content) due to timing
- Files: `layouts/partials/head.html` (lines 48-55)
- Impact: Brief flash of wrong theme visible before JS runs
- Fix approach: Move script inline before any HTML rendering, use `document.write` as fallback

**Mobile Menu Not Keyboard Accessible:**
- Issue: Mobile menu toggle uses only click handler, no keyboard support (Tab/Enter)
- Files: `layouts/partials/scripts.html` (lines 18-26)
- Impact: Keyboard users cannot access mobile menu
- Fix approach: Add keydown handlers for Enter/Space, set proper ARIA attributes

**Search Modal Accessibility Issues:**
- Issue: Search modal uses `hidden` class, no aria-hidden or focus trap implementation
- Files: `layouts/partials/scripts.html` (lines 28-67)
- Impact: Screen readers may read hidden content; focus can escape modal
- Fix approach: Add aria-hidden, implement focus trap, ensure escape key works consistently

**Inline JavaScript Without CSP**:
- Issue: Multiple inline scripts in head/footer without Content Security Policy
- Files: `layouts/partials/head.html`, `layouts/partials/scripts.html`
- Impact: Vulnerable to XSS attacks; no policy preventing inline script execution
- Fix approach: Add CSP headers in nginx.conf, move inline scripts to external files or use nonce attributes

## Content & Maintenance

**Only 1 Content File in Repository:**
- Issue: Only 2 content files present (about.md and _index.md stub)
- Files: `content/about.md`, `content/posts/_index.md`
- Impact: Unable to validate blog functionality end-to-end; unclear if posting workflow actually works
- Fix approach: Add sample posts to test content pipeline

**No Automated Link Checking:**
- Issue: No tool to validate internal/external links during build
- Impact: Dead links possible in published content; discovered only by manual testing
- Fix approach: Add htmlproofer or similar step to build pipeline

## Testing & Validation

**No Automated Tests:**
- Issue: No test suite for layouts, CSS, or build process
- Impact: Breaking changes to templates discovered post-deployment
- Fix approach: Add Hugo template testing, CSS regression tests, or integration tests

**No Build Validation:**
- Issue: Docker build succeeds but no checks that generated site is valid HTML/CSS
- Files: `Dockerfile` (line 14)
- Impact: Invalid HTML/CSS deployed silently
- Fix approach: Add HTML validation step (e.g., html-validate) to build, check for broken links

## Dependency Management

**Caret Versioning on Critical Dependencies:**
- Issue: All dependencies use `^` (caret) versioning allowing minor/patch updates
- Files: `package.json` (lines 11-16)
- Impact: Automatic updates could introduce breaking changes (e.g., Tailwind CSS major UI changes)
- Fix approach: Use exact versions or tilde for critical frontend deps, manage updates explicitly

**Node Version Not Pinned:**
- Issue: No .nvmrc or node version specification anywhere
- Impact: CI vs local developer environments may use different Node versions
- Fix approach: Add .nvmrc file, specify node version in flake.nix, document in USAGE.md

**Hugo Version Not Pinned:**
- Issue: Using `hugomods/hugo:exts` without version tag
- Files: `Dockerfile` (line 2)
- Impact: New Hugo versions could break build; developers using different local versions
- Fix approach: Pin to specific Hugo version tag (e.g., `hugomods/hugo:exts-0.123.4`)

## Documentation Gaps

**Deployment Path Mismatch in Instructions:**
- Issue: USAGE.md says copy docker-compose.yml to `/opt/blog` but deploy script writes to `/opt/docker/blog`
- Files: `USAGE.md` (line 67), `.github/workflows/deploy.yml` (line 66)
- Impact: Deployment fails if following documentation; undocumented directory structure
- Fix approach: Align paths or document both locations

**Missing Server Prerequisites:**
- Issue: USAGE.md doesn't specify required Docker version, traefik setup, or system resources
- Files: `USAGE.md` (deployment section)
- Impact: First-time deployments fail due to missing traefik network or Docker version incompatibility
- Fix approach: Add server setup checklist (Docker version, traefik network creation, disk space requirements)

**No Monitoring/Alerting Documentation:**
- Issue: No guidance on what to monitor or how to set up alerts for deployment failures
- Impact: Silent failures possible; slow detection of issues
- Fix approach: Document health check endpoints, log aggregation setup, deployment failure notifications

## Fragile Areas

**Search Functionality Dependency:**
- Files: `layouts/partials/scripts.html`, `package.json`
- Why fragile: Pagefind search completely fails if JS disabled or pagefind-ui.js fails to load (no fallback)
- Safe modification: Add loading state, fallback to Google search, graceful degradation
- Test coverage: No tests for search functionality

**CSS Pipeline Fragility:**
- Files: `layouts/partials/head.html`, `tailwind.config.js`, `postcss.config.js`
- Why fragile: Hugo pipes CSS post-processing; if resources directory doesn't exist or Hugo version changes, CSS won't generate
- Safe modification: Always test CSS generation locally before deploying, watch resource files in dev
- Test coverage: No CSS validation tests

---

*Concerns audit: 2026-01-28*
