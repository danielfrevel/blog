# Testing Patterns

**Analysis Date:** 2026-01-28

## Test Framework

**Runner:**
- Not detected - No test framework configured

**Assertion Library:**
- Not applicable

**Run Commands:**
```bash
# No test commands configured in package.json
# Current scripts:
npm run dev              # Run dev server
npm run build            # Build production site
npm run build:search     # Generate search index
```

## Test File Organization

**Location:**
- No test files found in project

**Naming:**
- Not applicable

**Structure:**
- Not applicable

## Testing Strategy

**Current State:**
- No automated testing infrastructure
- Manual testing via local development server and browser
- Build process validates Hugo template syntax

## Build-Time Validation

**Hugo Compilation:**
- `npm run build` command executes `hugo --minify`
- Hugo validates all templates and front matter during build
- Errors in templates block build completion
- Syntax highlighting via Chroma validates code fence formatting

**CSS/PostCSS Pipeline:**
- Tailwind CSS processes all `content: ["./layouts/**/*.html", "./content/**/*.md"]`
- PostCSS processes via Hugo Pipes
- Production: minification and fingerprinting applied

## Manual Testing Approach

**Development Workflow:**
- `npm run dev` runs `hugo server --buildDrafts --buildFuture`
- Live reload on file changes at `http://localhost:1313`
- Manual browser testing of:
  - Dark mode toggle and persistence
  - Mobile menu responsiveness
  - Search modal opening/closing
  - Keyboard shortcuts (Cmd/Ctrl+K, Escape)
  - Page transitions
  - Link navigation

**Production Build Testing:**
- `npm run build` generates optimized production output
- `npm run build:search` generates Pagefind search index
- Output validated in `public/` directory
- Manual verification before push to main branch

## Testing Gaps

**Untested Areas:**
- JavaScript functionality (dark mode toggle, search modal, mobile menu)
  - Files: `layouts/partials/scripts.html`
  - No unit or integration tests for DOM manipulation
  - Risk: Browser compatibility issues or event binding failures undetected

- Template rendering logic
  - Files: `layouts/_default/baseof.html`, `layouts/_default/single.html`, `layouts/_default/list.html`, `layouts/index.html`
  - No template output validation
  - Risk: Broken content structure, missing variables

- CSS/Responsive design
  - Files: `assets/css/main.css`, `tailwind.config.js`
  - Only manual browser testing
  - Risk: Breakpoint behavior inconsistencies across devices

- Third-party integrations
  - Pagefind search indexing and UI
  - Tailwind typography plugin
  - No integration tests

**Priority: High** - Critical user interactions (dark mode, search, navigation) have no automated coverage

## Dependencies for Testing

**dev**: Would need to add for testing
- Testing runner: Jest or Vitest
- DOM testing: Testing Library or similar
- Template testing: Hugo template testing library
- Responsive design testing: Playwright or similar

**Current dev dependencies** (from `package.json`):
```json
{
  "@tailwindcss/typography": "^0.5.10",
  "autoprefixer": "^10.4.16",
  "postcss": "^8.4.32",
  "postcss-cli": "^11.0.0",
  "tailwindcss": "^3.4.0",
  "pagefind": "^1.0.4"
}
```

## Browser Compatibility

**Testing Requirements:**
- Dark mode: relies on `localStorage` and `matchMedia()` API
- Search modal: requires DOM Level 2 Events (`addEventListener`)
- Mobile menu: uses `classList` API
- Requires modern browser support (ES6+)

## CI/CD Testing

**Current Pipeline** (`.github/workflows/deploy.yml`):
- No test steps configured
- Direct Docker build → push → deploy on main branch
- Relies on Hugo build succeeding (syntax validation only)

**Recommended Additions:**
- Unit tests for JavaScript functions
- Template snapshot testing
- Visual regression testing
- Lighthouse performance audit

## Docker Build Validation

**Dockerfile (implicit):**
- Hugo build step validates template syntax
- Pagefind indexing validates content structure
- Container startup verifies all assets are accessible

---

*Testing analysis: 2026-01-28*
