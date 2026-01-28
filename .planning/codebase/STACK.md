# Technology Stack

**Analysis Date:** 2026-01-28

## Languages

**Primary:**
- TOML - Hugo configuration (`hugo.toml`)
- Markdown - Content authoring for blog posts
- HTML - Theme templates
- CSS - Styling with Tailwind CSS

**Secondary:**
- JavaScript - Build tooling (Node.js scripts)
- Shell - Docker entrypoint and CI scripts

## Runtime

**Environment:**
- Node.js 24.13.0 - Build tooling and asset processing
- Hugo 0.154.5+extended - Static site generator

**Package Manager:**
- npm 11.6.2 - Node.js package management
- Lockfile: `package-lock.json` present

## Frameworks

**Core:**
- Hugo 0.154.5 - Static site generation, templating, markdown processing
- Tailwind CSS 3.4.0 - Utility-first CSS framework

**Styling & Processing:**
- PostCSS 8.4.32 - CSS transformation pipeline
- Autoprefixer 10.4.16 - CSS vendor prefix automation
- @tailwindcss/typography 0.5.10 - Typography plugin for Tailwind

**Search:**
- Pagefind 1.0.4 - Static site search indexing

## Key Dependencies

**Critical:**
- `pagefind` 1.0.4 - Generates searchable index from built static site during Docker build
- `tailwindcss` 3.4.0 - CSS utility framework with dark mode support
- `postcss` 8.4.32 - CSS processing for Tailwind compilation

**Infrastructure:**
- `postcss-cli` 11.0.0 - CLI for PostCSS processing
- `autoprefixer` 10.4.16 - Vendor prefix addition for CSS compatibility

## Configuration

**Environment:**
- Hugo configuration: `hugo.toml` - Site metadata, pagination, syntax highlighting (Dracula theme), RSS feeds, taxonomies, menus
- Tailwind configuration: `tailwind.config.js` - Content sources, dark mode via CSS class, extended fonts (Inter, JetBrains Mono), typography customization
- PostCSS configuration: `postcss.config.js` - Simple pipeline with Tailwind and Autoprefixer
- Nix development shell: `flake.nix` - Provides Hugo and Node.js in dev environment

**Build:**
- `Dockerfile` - Multi-stage build: Hugo build stage, then nginx runtime with compiled site
- `docker-compose.yml` - Service orchestration for containerized blog
- `.github/workflows/deploy.yml` - CI/CD pipeline

## Platform Requirements

**Development:**
- Nix flakes support (flake.nix provides hermetic Hugo and Node.js environment)
- Git for version control

**Production:**
- Docker and Docker Compose for containerization
- Traefik reverse proxy for routing and SSL/TLS (Let's Encrypt)
- Nginx Alpine as web server in production container
- External Docker registry: `ghcr.io` (GitHub Container Registry) for image storage

## Build Process

**Development Commands:**
```bash
npm run dev       # Hugo server with drafts and future posts
npm run build     # Hugo minified build
npm run build:search  # Pagefind search index generation
```

**Production Build Flow:**
1. Multi-stage Docker build using `hugomods/hugo:exts` builder image
2. npm install of Tailwind and PostCSS dependencies
3. Hugo site generation with minification
4. Pagefind search index generation
5. Nginx Alpine runtime serving compiled HTML from `/public`

---

*Stack analysis: 2026-01-28*
