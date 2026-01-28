# Blog Enhancement

## What This Is

A personal Hugo blog being enhanced with interactive visual elements, a knowledge graph, guest comments, and refined layout/styling. The goal is to transform a basic static blog into a distinctive, elegant experience while keeping the codebase simple and maintainable.

## Core Value

Visual distinction through subtle, mathematically-driven interactivity — the blog should feel alive without being distracting.

## Requirements

### Validated

<!-- Shipped and confirmed working from existing codebase. -->

- ✓ Hugo static site generation — existing
- ✓ Markdown content authoring with YAML frontmatter — existing
- ✓ Tailwind CSS utility styling — existing
- ✓ Dark mode toggle with localStorage persistence — existing
- ✓ Pagefind client-side search — existing
- ✓ Basic view transition API — existing
- ✓ Docker containerization with nginx serving — existing
- ✓ RSS feed generation — existing
- ✓ Mobile-responsive layout — existing

### Active

<!-- Current scope. Building toward these. -->

- [ ] Mouse-reactive geometric background animation (math/physics visualization)
- [ ] Obsidian-style knowledge graph (nodes = posts, edges = shared tags + explicit links)
- [ ] Enhanced view transitions with smoother animations
- [ ] Guest comments without authentication (SQLite + small server backend)
- [ ] Sticky footer (content fills viewport)
- [ ] Refined typography (better font pairing, spacing, hierarchy)
- [ ] Optimized content width for readability
- [ ] Light default theme with dark mode option

### Out of Scope

<!-- Explicit boundaries. -->

- User authentication for commenters — complexity not worth it for a personal blog
- Real-time features (websockets, live updates) — static site philosophy
- CMS/admin interface — Markdown + git is the workflow
- Comment moderation UI — manual database queries sufficient for low volume
- Analytics integration — privacy-focused, no tracking

## Context

**Existing stack:** Hugo 0.154.5, Tailwind 3.4, PostCSS, Pagefind, Docker/nginx

**Current state:** Functional but generic dark blog. View transitions exist but basic. Footer floats when content is short. Typography is default Tailwind.

**Infrastructure:** Containerized with Docker, served via nginx behind Traefik with Let's Encrypt SSL. Deployed via GitHub Actions to self-hosted server.

**Comments approach:** SQLite database with minimal Go/Node server alongside the static site. Form submits to API, page fetches comments on load. Spam prevention via honeypot + rate limiting (no captcha).

**Graph visualization:** Will need to generate JSON data at Hugo build time (posts, tags, links between posts). Frontend renders with D3.js or similar force-directed graph library.

**Background animation candidates:**
- Perlin noise flow fields
- Delaunay triangulation mesh
- Voronoi diagrams
- Lorenz/strange attractor
- Sine wave interference patterns
- Particle physics / gravity wells

## Constraints

- **Stack**: Must work with existing Hugo + Tailwind setup — no framework rewrites
- **Performance**: Background animation must not impact scrolling/interaction (requestAnimationFrame, GPU compositing)
- **Bundle size**: Keep JavaScript minimal — no heavy frameworks for comments or graph
- **Simplicity**: Code should be elegant and maintainable, not clever

## Key Decisions

<!-- Decisions that constrain future work. -->

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Light default theme | User preference for minimal/clean aesthetic | — Pending |
| SQLite for comments | Simple, file-based, fits self-hosted philosophy | — Pending |
| Self-hosted comments backend | Full control, no third-party dependencies | — Pending |
| Graph shows tags + explicit links | Richer connection visualization than tags alone | — Pending |

---
*Last updated: 2026-01-28 after initialization*
