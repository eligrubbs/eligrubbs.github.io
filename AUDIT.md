# Codebase Audit — eligrubbs.github.io

_Audited 2026-10-03, at commit `7a6fb67` on `main`._

## Summary

The site is a two-page static GitHub Pages site (a landing page and an "Under Construction" projects page) served at `www.eligrubbs.com`. It was set up to use Tailwind CSS, but **Tailwind is not actually used**: the pages link the raw, uncompiled source stylesheet, and the compiled `dist/output.css` is stale and linked from nowhere. All visible styling comes from about 100 lines of hand-written CSS.

The existing content is small (a name, a one-line bio, three icon links), so the planned overhaul is effectively a rewrite. Very little needs to be preserved.

## Inventory

| File | Size | Purpose | Status |
|---|---|---|---|
| `index.html` | 70 lines | Landing page: name, one-line bio, email/GitHub/LinkedIn icons | Live |
| `pages/project.html` | 44 lines | Projects page: "🚧 Under Contruction 🚧" | Live, placeholder |
| `src/input.css` | 102 lines | Tailwind directives + all custom CSS | **Served directly to browsers** |
| `dist/output.css` | 807 lines | Compiled Tailwind output | **Unused and stale** |
| `tailwind.config.js` | 8 lines | Tailwind config | Effectively unused |
| `package.json` / `package-lock.json` | — | Single devDependency: `tailwindcss ^3.3.3` | Effectively unused |
| `favicon.ico` | 1.1 KB | 16×16 icon | Live |
| `CNAME` | — | `www.eligrubbs.com` | Live |
| `README.md` | 5 lines | Minimal description | — |

There is no `.gitignore`, no `404.html`, no GitHub Actions workflow, and no JavaScript.

## Findings

### 1. Build and tooling

- **Tailwind is not wired up.** Both pages load `src/input.css` (`index.html:6`, `pages/project.html:6`). Browsers ignore the `@tailwind base/components/utilities` lines in it, so no Tailwind styles are applied. Neither HTML file uses any Tailwind utility classes.
- **`dist/output.css` is dead code from an earlier design.** It contains utilities such as `.bg-neutral-950`, `.max-w-7xl`, and `.mt-56` that do not appear in the current HTML, and the custom rules from `src/input.css` (`.nav-ul`, `.social-icon`, etc.) are missing from it. Nothing links to it.
- **The Tailwind `content` globs miss the projects page.** `tailwind.config.js` scans `./index.html` and `./src/**/*`, not `./pages/**`, so a rebuild would drop any classes used only there.
- **There is no build step.** GitHub Pages serves the repository as-is, and there is no workflow that compiles CSS. Using Tailwind properly would require either committing a compiled CSS file after every change or adding an Actions workflow.
- **Tailwind 3.3.3 is outdated** (v4 is current). This only matters if Tailwind is kept.
- **There is no `.gitignore`.** `node_modules/` isn't committed today, but nothing prevents it from being added by accident.

**Recommendation:** For a minimal, low-color site, drop Tailwind entirely and use one small hand-written stylesheet. Delete `package.json`, `package-lock.json`, `tailwind.config.js`, `src/input.css`, and `dist/`. That leaves no build step, no dependencies, and nothing to keep in sync.

### 2. HTML structure and duplication

- **The `<head>` and nav are copy-pasted** between both pages. Each new page will add another copy. With only 2–3 pages, hand-duplication is acceptable. If the projects list grows, consider Jekyll, which GitHub Pages runs natively with no extra tooling, for a shared layout and a data file of projects.
- **Three of the four nav links are dead.** "about", "posts", and "other" all point to `/` (`index.html:23-33`).
- **The class `.container` collides with Tailwind's built-in `.container` utility**, which could cause conflicts if Tailwind were ever compiled in.
- **There are no-op CSS rules.** `nav` sets `flex-direction` and `justify-content` while `display: inline-block` is in effect (`src/input.css:37-42`). `.nav-ul` has only a `border-right`, which leaves a single stray divider after the last item.
- **The projects page has no `<h1>`**, only an `<h3>`. "Contruction" is misspelled (`pages/project.html:39`).
- **The projects page lives at `/pages/project.html`**, which is a clunky URL. `/projects/` (that is, `projects/index.html`) or `/projects.html` would be cleaner.

### 3. Metadata and SEO

- **Both pages are titled "Personal Website".** The browser tab and search results never show your name.
- **There is no `<meta name="description">`**, no Open Graph or Twitter card tags (so link previews on LinkedIn and Slack will be bare), and no canonical URL.
- **The favicon is a single 16×16 `.ico`.** There is no high-resolution PNG/SVG or `apple-touch-icon`, so it will look blurry on modern displays. The `shortcut icon` and `icon` links are redundant.
- **There is no `404.html`.** GitHub's default 404 page will be shown.

### 4. Accessibility

- **Icon-only links rely on `title` for their names.** The email, GitHub, and LinkedIn links contain only an SVG, and `title` is announced inconsistently by screen readers. Add `aria-label` to the links and `aria-hidden="true"` to the SVGs.
- **The LinkedIn icon is styled differently from the others** (`index.html:58`). It uses the default black fill with `stroke="white"` instead of `currentColor`, so it won't follow the text color (for example, in dark mode) and looks heavier than the other two outline icons.
- **Links have no visible focus style beyond the browser default**, and `text-decoration: none` on all links makes inline links indistinguishable from body text.
- **The `<header class="banner">` sits inside `.container` with the main content**, rather than both being direct children of `<body>`. This is minor, but it makes the landmark structure less clear.

### 5. Performance and privacy

- **Roboto loads from Google Fonts** through the legacy `css?family=` API without `display=swap` (`index.html:7`). That is an extra third-party request on every page view, it sends visitor IPs to Google, and text may flash during load. A system font stack would be faster, need no third-party requests, and suit a minimal design.
- **`font-feature-settings` asks for `tnum`, `zero`, `ss01`, and more** (`src/input.css:18`). These features are mostly irrelevant here, and support varies by font.
- Overall page weight is tiny. Apart from the font request, performance is not a concern.

### 6. Links and content

- **`mailto:` links use `target="_blank"`** (`index.html:45`), which opens a blank tab in some browsers. Remove it from the email link.
- **The bio is one line** ("CS Graduate from The University of Michigan.") and needs updating for the overhaul.
- **The `<!-- I copied design from enjeeneer.io -->` comment** is in both pages. If the new design is original, drop it. If you borrow from that site again, a credit is good practice, but check its license first.
- **Leftover comments** like `<!-- Favicon Schriften Font-->` are vestigial.

### 7. Repository hygiene

- **Domain config:** `CNAME` is `www.eligrubbs.com`, while the README links `eligrubbs.com`. Confirm that the apex domain redirects to `www` (DNS A/ALIAS records plus "Enforce HTTPS" in the Pages settings).
- **The history has 13 noisy CNAME create/delete commits** from 2023-10-18. They are harmless and not worth rewriting.
- **The README doesn't say how to run the site locally.** Something like `python3 -m http.server` would be enough once there's no build step.

## What's worth keeping

- `CNAME` (must keep, or the custom domain breaks).
- `favicon.ico` (or replace it with a better icon set).
- The GitHub and email SVG icon paths, if you want icons. The LinkedIn one should be swapped for a matching outline icon.
- The social URLs: `mailto:contact@eligrubbs.com`, `https://github.com/eligrubbs`, `https://linkedin.com/in/grubbse`.

## Suggested direction for the overhaul

1. Remove Tailwind and Node tooling. Use one `style.css` of roughly 50–80 lines, built on a system font stack, a narrow max-width content column, and 1–2 colors (text plus one accent), with optional `prefers-color-scheme` dark mode.
2. Make the landing page (`index.html`) hold a name, a short intro paragraph, and a small row of text links or icons (LinkedIn, GitHub, email).
3. Add a projects section, either on the landing page itself (simplest, given the "minimal" goal) or at `/projects/`. Give each project a title, a 1–2 sentence description, and links (repo/demo).
4. Add proper `<title>`, meta description, Open Graph tags, `404.html`, and a `.gitignore`.
5. Optional: if the project list will grow or you want blog posts later, use Jekyll layouts and a `_data/projects.yml` file. GitHub Pages builds these natively.
