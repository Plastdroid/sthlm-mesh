# CLAUDE.md

This file provides guidance for AI assistants working with the STHLM-MESH codebase.

## Project Overview

STHLM-MESH is a Swedish-language community website for the Stockholm Meshtastic LoRa mesh
network. It is built with **Hugo** (static site generator) using the **Docsy** theme and
deployed to **GitHub Pages** at [sthlm-mesh.se](https://sthlm-mesh.se).

- **Primary language of content:** Swedish
- **License:** Apache 2.0 (code), CC BY 4.0 (content)
- **Repository:** https://github.com/Roslund/sthlm-mesh

## Build & Development Commands

### Local Development (Docker - preferred)

```bash
docker compose up --build
# Site available at http://localhost:1313 with live reload
```

### npm Commands

| Command | Description |
|---------|-------------|
| `npm run serve` | Start local dev server with drafts/future content |
| `npm run build` | Development build (includes drafts, future, expired) |
| `npm run build:production` | Production build with minification |
| `npm run clean` | Remove `public/` and `resources/` directories |
| `npm run test` | Run link checking (currently pending implementation) |

### Hugo Version

Hugo is **frozen at version 0.145.0** (extended) due to Docsy theme compatibility issues.
Do not upgrade Hugo without testing theme compatibility. This version is pinned in:
- `package.json` (`hugo-extended: "0.145.0"`)
- `docker-compose.yaml` (`hugomods/hugo:exts-0.145.0`)
- `.github/workflows/hugo.yml` (`hugo-version: '0.145.0'`)

## Project Structure

```
sthlm-mesh/
├── content/sv/              # All site content (Swedish)
│   ├── _index.md            # Homepage
│   ├── blog/                # Blog posts and meetups
│   │   ├── meetups/         # Upcoming meetups
│   │   └── past-meetups/    # Archived meetups
│   ├── docs/                # Main documentation
│   │   ├── hardware/        # Hardware guides (antennas, devices, etc.)
│   │   ├── meshtastic/      # Meshtastic configuration docs
│   │   └── firmware/        # Firmware information
│   ├── about/               # About page
│   ├── status/              # Live mesh network dashboard
│   └── messages/            # Message feed page
├── layouts/                 # Custom Hugo templates
│   ├── shortcodes/          # Custom shortcodes (lazy-img, image-compare)
│   ├── blog/                # Blog-specific templates
│   ├── partials/            # Reusable template partials
│   └── _default/            # Default template overrides
├── assets/scss/             # Custom SCSS styles
├── static/
│   ├── js/                  # Client-side JavaScript
│   │   ├── status/          # Dashboard chart scripts (17 files)
│   │   ├── messages.js      # Live message feed
│   │   ├── esp-flasher.js   # Browser-based ESP32 firmware flasher
│   │   ├── firmware-ui.js   # Firmware flashing UI
│   │   └── rsvp-tracker.js  # Meetup RSVP system
│   ├── firmware/            # Binary firmware files for devices
│   └── images/              # Static images
├── hugo.yaml                # Main Hugo configuration
├── go.mod                   # Hugo module dependencies (Docsy theme)
├── package.json             # npm dependencies and scripts
├── docker-compose.yaml      # Local dev container
└── .github/workflows/       # CI/CD (GitHub Actions)
```

## Architecture & Key Patterns

### Content System

- All content lives under `content/sv/` and is written in Markdown
- Content uses Hugo frontmatter (YAML) for metadata:
  ```yaml
  ---
  title: Page Title
  linkTitle: Short Title
  weight: 30              # Controls menu ordering
  draft: false
  hide_readingtime: true
  ---
  ```
- Blog permalinks follow `/blog/:year/:slug/` pattern
- Documentation is the primary content area at `content/sv/docs/`

### Theme: Docsy

- Imported as a Hugo module via `go.mod` (github.com/google/docsy v0.11.0)
- Provides Bootstrap-based responsive layouts, sidebar navigation, and block shortcodes
- Custom template overrides go in `layouts/` (they take precedence over the theme)

### Custom Shortcodes

- **`lazy-img`**: Lazy-loads images with a spinner placeholder
  ```
  {{< lazy-img src="image.png" max-width="600px" aspect-ratio="16/9" >}}
  ```
- **`image-compare`**: Interactive before/after image slider
  ```
  {{< image-compare left="before.png" right="after.png" caption="..." >}}
  ```

### JavaScript / Dashboard

- Status dashboard at `/status/` fetches live data from `https://map.sthlm-mesh.se/api/v1/`
- Chart scripts in `static/js/status/` use Chart.js for data visualization
- `static/js/status/shared.js` contains shared utilities (node fetching, common helpers)
- The ESP firmware flasher uses esptool-js for browser-based device flashing

### Styling

- Bootstrap (via Docsy theme) for layout and components
- Custom SCSS in `assets/scss/` for project-specific styles
- Prettier configured with `proseWrap: always` and `singleQuote: true`

## CI/CD & Deployment

- **GitHub Actions** (`.github/workflows/hugo.yml`) builds and deploys on push to `main`
- Pull requests trigger builds but do not deploy
- Deployment uses `peaceiris/actions-gh-pages@v3` to publish `./public` to GitHub Pages
- Custom domain `sthlm-mesh.se` is configured via CNAME
- **Dependabot** runs daily for npm dependencies (hugo-extended is excluded from auto-updates)

## Conventions & Guidelines

### Content Contributions

- Write content in Swedish (or "Swenglish" for technical terms)
- Place documentation under `content/sv/docs/`
- Place blog posts under `content/sv/blog/`
- Use `weight` in frontmatter to control page ordering in navigation
- Archive past meetups by moving them from `blog/meetups/` to `blog/past-meetups/`

### Code Style

- No formal linter; spell checking via cSpell (`.cspell.yml`)
- JavaScript files are vanilla JS (no framework, no bundler)
- Dashboard scripts follow a pattern: fetch data from API, process it, render Chart.js charts
- Commit messages should be in English

### What NOT to Do

- Do not upgrade Hugo without testing Docsy theme compatibility (currently frozen at 0.145.0)
- Do not commit `public/`, `resources/`, `node_modules/`, or `package-lock.json` (all in `.gitignore`)
- Do not add content in languages other than Swedish without discussion
- The firmware binaries in `static/firmware/` are large; avoid unnecessary changes to them

### External API

All dashboard data comes from `https://map.sthlm-mesh.se/api/v1/`. Key endpoints:
- `/nodes/` - All mesh network nodes
- `/text-messages` - Recent messages
- `/stats/channel-utilization-stats` - Channel utilization metrics
