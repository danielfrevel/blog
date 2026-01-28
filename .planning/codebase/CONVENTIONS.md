# Coding Conventions

**Analysis Date:** 2026-01-28

## Naming Patterns

**Files:**
- Hugo templates: PascalCase or snake_case for partials (e.g., `baseof.html`, `header.html`, `scripts.html`)
- Content files: kebab-case with `.md` extension (e.g., `my-post-title.md`)
- Config files: camelCase or standard conventions (e.g., `tailwind.config.js`, `postcss.config.js`, `hugo.toml`)
- Asset files: lowercase with hyphens (e.g., `main.css`)

**Template Variables:**
- Hugo template variables use dot notation and camelCase (e.g., `.Site.Title`, `.Params.tags`, `.IsMenuCurrent`)
- Local Hugo variables use camelCase (e.g., `$posts`, `$css`)
- HTML element IDs: kebab-case (e.g., `search-button`, `theme-toggle`, `mobile-menu-button`)

**JavaScript Variables:**
- camelCase for function names and variables (e.g., `openSearch`, `closeSearch`, `mobileMenuButton`)
- Const declarations for static references (e.g., `const themeToggle = document.getElementById('theme-toggle')`)

**CSS Classes:**
- Utility-first approach via Tailwind (e.g., `flex items-center gap-6 text-sm font-medium`)
- Custom components defined via `@layer components` in `/home/dan/code/blog/assets/css/main.css`
- Semantic naming for custom classes (e.g., `.highlight pre`)
- Dark mode using `dark:` prefix for dark-specific classes

**Data Attributes:**
- kebab-case for datetime attributes (e.g., `datetime="2026-01-22"`)

## Code Style

**Formatting:**
- HTML: 2-space indentation
- JavaScript: 2-space indentation, modern ES6+ syntax
- CSS: Tailwind utility classes preferred, custom CSS in `@layer` directives
- YAML (hugo.toml): Standard TOML formatting

**Linting:**
- No ESLint or Prettier configuration detected
- Follow existing patterns: smooth formatting, readable spacing

**Comments:**
- Minimal comments - prefer self-documenting code
- HTML comments for structural sections (e.g., `<!-- Dark Mode Toggle -->`, `<!-- Mobile Menu -->`)
- JavaScript inline comments for complex logic (e.g., `// Close on Escape key`)
- CSS layer organization comments (e.g., `@layer base`, `@layer components`)

## Import Organization

**Hugo Templates:**
- Partials imported via `{{- partial "name.html" . -}}` syntax
- Block definitions via `{{ define "main" }}`
- Template inheritance via `baseof.html` base layout

**JavaScript:**
- Inline scripts in HTML partials
- External library loading via CDN links (e.g., Pagefind, Google Fonts)
- No module bundling or import statements used

**CSS/PostCSS:**
- Tailwind directives: `@tailwind base`, `@tailwind components`, `@tailwind utilities`
- Layer organization: `@layer base`, `@layer components`
- PostCSS processing via Hugo Pipes: `resources.Get` and `css.PostCSS`

## DOM Manipulation

**Event Handling:**
- Direct `addEventListener` for interactivity
- ID-based element selection via `document.getElementById()`
- Toggle class patterns: `classList.toggle('hidden')`, `classList.add()`, `classList.remove()`

**Dynamic Behavior:**
- Dark mode: localStorage theme persistence with `localStorage.theme`
- Search modal: visibility toggle with `classList` manipulation
- Mobile menu: responsive hide/show via `classList.toggle('hidden')`

## Template Structure

**Base Layout:**
- `layouts/_default/baseof.html`: Main structural wrapper
- `layouts/partials/head.html`: Head meta tags, fonts, styles
- `layouts/partials/header.html`: Navigation, search, dark mode toggle
- `layouts/partials/scripts.html`: All inline JavaScript
- `layouts/partials/footer.html`: Footer content
- `layouts/_default/single.html`: Single post template
- `layouts/_default/list.html`: List/pagination template
- `layouts/index.html`: Homepage template

**Content Front Matter:**
- YAML front matter with: `title`, `date`, `draft`, `tags`, `description`
- Tags organized as arrays: `tags: ["tag1", "tag2"]`
- Dates in ISO format: `date: 2026-01-22`

## Function Design

**JavaScript Functions:**
- Single responsibility: separate functions for open/close operations (e.g., `openSearch()`, `closeSearch()`)
- Event delegation: one listener per control
- Async handling: setTimeout for DOM-dependent operations (100ms delay for input focus)

**Hugo Functions:**
- Template conditionals: `{{ if }}`, `{{ with }}`, `{{ range }}`
- Template logic: pagination via `{{ template "_internal/pagination.html" . }}`
- Data filtering: `{{ where .Site.RegularPages "Section" "posts" }}`

## Styling Patterns

**Tailwind Utility Classes:**
- Responsive prefixes: `sm:`, `md:`, `lg:` for breakpoint-specific styling
- Dark mode: `dark:` prefix for dark theme variants
- State modifiers: `hover:`, `focus:`, `group-hover:` for interactions
- Spacing: `gap-`, `p-`, `m-`, `px-`, `py-` for margins/padding
- Colors: semantic color scale usage (e.g., `gray-100`, `gray-900`, `blue-600`)

**Custom CSS:**
- Minimal custom CSS, layered via PostCSS
- Scrollbar styling in `@layer base`
- Code block highlighting in `@layer components`
- Search highlight styling: `mark` element styling

## Error Handling

**Strategy:** Defensive DOM operations with existence checks

**Patterns:**
- Optional chaining for potentially missing elements: `const input = searchModal.querySelector('input'); if (input) input.focus();`
- Safe state transitions: check `classList.contains()` before toggling
- Fallback to system preferences: dark mode respects `prefers-color-scheme` media query

## Accessibility

**ARIA Labels:**
- All interactive buttons include `aria-label` (e.g., `aria-label="Search"`, `aria-label="Toggle dark mode"`)
- Semantic HTML: proper button elements, datetime attributes
- Focus management: search input receives focus after modal opens

**Keyboard Interactions:**
- Escape key closes search modal
- Cmd/Ctrl+K opens search (keyboard shortcut)
- Tab navigation preserved in all interactive elements

---

*Convention analysis: 2026-01-28*
