# Architecture

**Analysis Date:** 2026-01-28

## Pattern Overview

**Overall:** Static Site Generator (SSG) with Hugo + Build Pipeline

**Key Characteristics:**
- Content-as-code: Markdown files converted to HTML at build time
- Separation of content, templates, and styling
- Client-side enhancement layers (JavaScript for interactivity without server)
- Multi-stage Docker build with nginx static serving

## Layers

**Content Layer:**
- Purpose: Define the source of truth for blog posts and pages
- Location: `content/`
- Contains: Markdown files with YAML frontmatter (posts, pages, taxonomies)
- Depends on: None (input)
- Used by: Hugo template engine

**Template/Layout Layer:**
- Purpose: Define HTML structure and page rendering
- Location: `layouts/`
- Contains: Hugo templates (.html files) organized by page type
- Depends on: Content layer (accesses frontmatter, body), configuration
- Used by: Hugo build process

**Styling Layer:**
- Purpose: Define visual appearance and responsive behavior
- Location: `assets/css/main.css`, `tailwind.config.js`
- Contains: Tailwind CSS directives, custom CSS, typography
- Depends on: Layout layer (scans .html files for class usage)
- Used by: PostCSS/Tailwind build pipeline

**Build & Deployment Layer:**
- Purpose: Compile site, optimize assets, containerize, deploy
- Location: `Dockerfile`, `.github/workflows/deploy.yml`, `docker-compose.yml`
- Contains: Multi-stage Docker build, GitHub Actions CI/CD, container orchestration
- Depends on: All previous layers
- Used by: Production environment

**Runtime/Static Serving Layer:**
- Purpose: Serve static HTML/CSS/JS with caching and security headers
- Location: `nginx.conf`, `public/` (generated)
- Contains: Nginx configuration, static assets, search index
- Depends on: Built artifacts from build layer
- Used by: End users, crawlers

## Data Flow

**Build Time Flow:**

1. **Content Input:** Markdown files in `content/` parsed by Hugo
2. **Frontmatter Processing:** YAML metadata extracted (title, date, tags, description)
3. **Template Rendering:** Hugo applies templates from `layouts/` to content
   - Base template (`baseof.html`) wraps page content
   - Partials (`head.html`, `header.html`, `footer.html`, `scripts.html`) included
   - Page-specific templates (`index.html`, `single.html`, `list.html`) render main content
4. **Asset Pipeline:**
   - CSS (`main.css`) processed through PostCSS
   - Tailwind scans `layouts/` and `content/` for class names
   - Autoprefixer adds vendor prefixes
   - Minified in production
5. **Search Index Generation:** Pagefind crawls built HTML to create search index
6. **Artifact Output:** All files written to `public/` directory

**Runtime Flow:**

1. **Request Arrives:** Client requests URL at `blog.danfrevel.de`
2. **Nginx Resolution:**
   - Static assets (CSS, JS, fonts) served with long cache headers (1 year)
   - HTML pages served with short cache (1 hour)
   - Pagefind index cached for 1 day
3. **Client Execution:**
   - HTML loaded in browser
   - Dark mode script runs immediately (prevents flash)
   - Fonts preload from CDN
   - View transition API setup for smooth navigation
   - Pagefind UI initializes for search
4. **User Interactions:**
   - Dark mode toggle updates localStorage and DOM
   - Mobile menu toggle shows/hides navigation
   - Search modal opens with Cmd/Ctrl+K or button click
   - Post navigation (prev/next) driven by Hugo template logic

**State Management:**

- **Theme preference:** Stored in localStorage (`localStorage.theme`)
- **Mobile menu:** Managed via DOM class toggling (`.hidden` class)
- **Search modal:** Toggled via hidden class, scroll lock on body
- **No backend state:** All state is ephemeral, scoped to browser session

## Key Abstractions

**Template System:**

- Purpose: Reusable HTML components and page layouts
- Examples: `layouts/_default/baseof.html`, `layouts/partials/header.html`
- Pattern: Hugo template inheritance with base template and partials
- The `baseof.html` defines overall HTML structure; content-specific templates define main content block
- Partials (`head.html`, `header.html`, `footer.html`, `scripts.html`) are included in multiple templates

**Markdown-based Content:**

- Purpose: Write blog posts and pages in simple, portable format
- Examples: `content/posts/_index.md`, `content/posts/*.md`, `content/about.md`
- Pattern: YAML frontmatter metadata + Markdown body
- Frontmatter keys: `title`, `date`, `draft`, `tags`, `description`
- Content is versioned in git, changes trigger rebuilds

**Styling with Tailwind CSS:**

- Purpose: Utility-first, composable styling without writing custom CSS
- Examples: Classes like `text-4xl`, `font-bold`, `dark:bg-gray-900`
- Pattern: Tailwind directives in `main.css` + HTML class names
- Dark mode via `dark:` prefix; responsive via `sm:`, `md:`, etc.
- Custom typography via `@tailwindcss/typography` plugin for prose styling

**Static Search (Pagefind):**

- Purpose: Client-side full-text search without database/server
- Pattern: Build-time HTML crawling creates JSON index, JS widget on client
- Search data stored in `public/pagefind/` directory
- UI initialized in `scripts.html` with result display

## Entry Points

**Hugo Build:**
- Location: `hugo.toml` + `package.json` scripts
- Triggers: `npm run dev` (watch mode), `npm run build` (production), `npm run build:search`
- Responsibilities:
  - Reads config from `hugo.toml`
  - Discovers content in `content/`
  - Applies templates from `layouts/`
  - Outputs HTML to `public/`

**CSS Pipeline:**
- Location: `postcss.config.js`, `tailwind.config.js`, `assets/css/main.css`
- Triggers: PostCSS runs during Hugo build
- Responsibilities:
  - Tailwind scans HTML templates for used classes
  - Generates only used CSS
  - Autoprefixer adds browser prefixes
  - Minifies in production

**Docker Build:**
- Location: `Dockerfile`
- Triggers: GitHub Actions on push to main
- Responsibilities:
  - Installs npm dependencies
  - Runs Hugo build
  - Generates search index
  - Copies assets to nginx container
  - Outputs final image to container registry

**Deployment:**
- Location: `.github/workflows/deploy.yml`
- Triggers: Push to main branch or manual workflow dispatch
- Responsibilities:
  - Builds Docker image and pushes to GHCR
  - Connects to remote server via SSH
  - Pulls latest image and runs docker-compose
  - Manages container lifecycle

**Client-side Scripts:**
- Location: `layouts/partials/scripts.html`
- Triggers: Automatically on page load
- Responsibilities:
  - Dark mode detection and toggle
  - Mobile menu toggle
  - Search modal open/close
  - Keyboard shortcuts (Cmd/Ctrl+K for search, Esc to close)
  - Pagefind UI initialization

## Error Handling

**Strategy:** Silent degradation with graceful fallbacks

**Patterns:**

- **Missing content:** Pages render "No posts yet" or "No content found" templates
- **CSS loading failure:** Site remains functional in default (light) styling
- **JavaScript unavailability:** Core navigation still works; search and dark mode unavailable
- **Pagefind loading:** Search button functional, but results won't appear until JS loads
- **404 errors:** Custom 404.html template served by nginx via error_page directive

## Cross-Cutting Concerns

**Logging:** None - Static site has no server-side logging. Client errors not tracked.

**Validation:**
- Frontmatter validation implicit (Hugo fails build if invalid YAML)
- No runtime validation (all checks occur at build time)

**Authentication:** None - Public blog, no authentication required

**Dark Mode:**
- Detected via `prefers-color-scheme` media query on first load
- User toggle persisted to localStorage
- System preference changes monitored via `matchMedia` listener
- Applied via `dark:` prefix on classes and embedded CSS variables

**SEO:**
- Canonical URL in `<head>` prevents duplicate content
- Open Graph meta tags in `head.html` for social sharing
- Structured description in frontmatter
- RSS feed generation for feed readers
- Sitemap generated by Hugo automatically

**Performance:**
- Static output means no database queries, fast responses
- CSS minimization removes unused classes (Tailwind)
- Asset fingerprinting in production prevents cache issues
- Long cache headers (1 year) for versioned assets
- Short cache (1 hour) for HTML pages (allows updates)
- Gzip compression in nginx for text/code
- View Transitions API for smooth navigation without full page reloads

**Security:**
- nginx security headers: X-Frame-Options, X-Content-Type-Options, X-XSS-Protection
- Content Security Policy implicit (no inline scripts except initialization)
- HTTPS enforced by Traefik reverse proxy with Let's Encrypt certs
- HTML unsafe rendering disabled by default; only enabled for specific markdown processing
- No user input accepted (read-only site)

---

*Architecture analysis: 2026-01-28*
