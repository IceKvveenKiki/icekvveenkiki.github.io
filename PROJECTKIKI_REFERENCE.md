# ProjectKiki.tv — Project Reference

This document captures the design decisions, structure, and conventions for the ProjectKiki.tv website. Use it as context when working with Claude Code or returning to this project after time away.

---

## Project Overview

A personal hub website for Kiki (Twitch: `icekvveenkiki`), connecting her DayZ community to server information, Twitch chat commands, and her Etsy shop. The site is hosted on GitHub Pages at `icekvveenkiki.github.io` with the custom domain `icekvveenkiki.com`. The domain `projectkiki.tv` redirects here.

**Files in the repo:**
- `index.html` — the entire site (single file, no build tools)
- `images/` — folder for locally hosted images (e.g. Etsy product photo)

---

## Hosting & Workflow

- **Host:** GitHub Pages (free tier, public repo)
- **Editor:** VS Code
- **Deployment:** Push `index.html` to the `main` branch — GitHub Pages auto-deploys within ~30 seconds
- **No build process** — pure HTML, CSS, and vanilla JavaScript. No frameworks, no npm, no compilation step.

---

## Color Scheme

All colors are defined as CSS custom properties (variables) in the `:root` block at the top of the `<style>` section.

| Variable | Value | Usage |
|---|---|---|
| `--pink` | `#FF0081` | Primary accent — logo, headings, active states, icons |
| `--pink-dim` | `rgba(255,0,129,0.08)` | Subtle pink backgrounds (badges, hover fills) |
| `--pink-border` | `rgba(255,0,129,0.25)` | Pink-tinted borders |
| `--bg-base` | `#080c12` | Page background |
| `--bg-surface` | `#0d1520` | Bento cell / card backgrounds |
| `--bg-deep` | `#060a10` | Footer, deepest background layer |
| `--bg-mid` | `#0b1018` | Table headers, filter bars, nav |
| `--border` | `#1a2535` | All borders and dividers |
| `--text-primary` | `#c8d4e0` | Main readable text |
| `--text-secondary` | `#5a7090` | Secondary/body text |
| `--text-muted` | `#2a3d52` | Labels, placeholders, de-emphasized text |
| `--text-white` | `#ffffff` | High-contrast headings only |

**Rule:** Pink is an accent only — never a background, never overused. The base palette is dark navy.

---

## Typography

Fonts are loaded from Google Fonts.

| Variable | Font | Usage |
|---|---|---|
| `--font-sans` | `DM Sans` | All body text, UI elements |
| `--font-mono` | `DM Mono` | Commands, IP addresses, restart times, code |

Icons come from the **Tabler Icons** webfont (loaded via CDN). Icon classes follow the pattern `ti ti-[icon-name]`, e.g. `ti ti-brand-twitch`.

---

## Layout — Bento Grid

The page uses a **bento box layout** — a grid of cards with rounded corners and subtle borders, each containing a distinct piece of content.

```
┌─────────────────────────┬───────────────┐
│  About / Welcome        │  Etsy product │
│  (1.65fr)               │  (1fr)        │
├─────────────────────────┴───────────────┤
│  DayZ Servers (collapsible)             │
├─────────────────────────────────────────┤
│  Kiki's Chat Commands (collapsible)     │
└─────────────────────────────────────────┘
```

The top row uses CSS Grid: `grid-template-columns: 1.65fr 1fr`.
Below that, each section is full width.
Max content width: `1100px`, centered with `margin: 0 auto`.

**Bento cell base styles** (`.bento` class):
- Background: `var(--bg-surface)`
- Border: `0.5px solid var(--border)`
- Border radius: `var(--radius-lg)` = `14px`
- Padding: `22px`

---

## Page Sections

### Nav
- Sticky at top (`position: sticky; top: 0`)
- Logo: "ProjectKiki.tv" in `--pink`
- Nav links scroll to `#servers`, `#commands`, `#about`

### Social Bar
- Sits below nav
- Has a "Social Links:" label above it (`.social-bar-header`)
- Pills for: Twitch (purple), Instagram (pink), Etsy (orange), X/Twitter (blue-grey)
- Each pill is an `<a>` tag with class `social-pill [platform]`

### About / Welcome Bento
- ID: `#about`
- Large heading using `.about-name` class (35px, white with pink `<span>` for "Kiki")
- Monospace handle line below
- Bio paragraph in `.about-bio`
- Decorative pink circle accent (`.about-accent`) — purely visual, bottom-right corner

### Etsy Product Bento
- Shows product image hosted in `/images/` folder on GitHub
- Falls back to placeholder text if image fails to load
- Orange-tinted "View on Etsy" button linking directly to the listing

### DayZ Servers (collapsible)
- ID: `#servers`
- Collapsible using `.collapsible` + `.server-body` pattern
- Header uses `.about-name` style (pink heading) with `.cell-header-sub` subtitle
- Filter chips: All / Legacy / Deathmatch / Playtest / In-development
- Search input filters by name, map, host, type, region
- Sortable columns: Server Name, Map, Type, Region (click to sort asc/desc)
- IP addresses hidden behind a copy-to-clipboard button
- Links column: Discord icon, Battlemetrics icon, Useful Links icon
- **Data source:** Google Sheets published CSV (see Data section below)

### Chat Commands (collapsible)
- ID: `#commands`
- Same collapsible pattern as servers
- Filter chips: All / Chat responses / Stream controls / Media / On-screen
- Permissions color-coded: VIPs (pink/purple), Mods (green), Everyone (default)
- **Data source:** Hardcoded in the JavaScript `COMMANDS` array

---

## Collapsible Pattern

Both major tables use the same expand/collapse pattern:

```html
<div class="collapsible" id="section-id">
  <div class="collapse-hdr" role="button" aria-expanded="false" tabindex="0">
    <!-- header content -->
    <i class="ti ti-chevron-down chevron"></i>
  </div>
  <div class="server-body"> <!-- or cmd-body -->
    <!-- table content -->
  </div>
</div>
```

- `.collapsible.open` class is toggled by JavaScript
- Chevron rotates 180° when open (CSS `transform: rotate`)
- Both sections auto-open if the URL hash matches (`#servers` or `#commands`)

---

## Data Sources

### Server Table — Google Sheets CSV
The server list is fetched live from a published Google Sheets CSV on every page load. No API key needed.

**CSV URL** (defined as `SHEETS_CSV_URL` in the script):
```
https://docs.google.com/spreadsheets/d/e/2PACX-1vQk4A1JGYO61CqaoEF7qWNMlvvuYu1XvNDAw1zESu_xNxGktTmhxe3BRWLGvSRubZ9fRRX0gVPQCUKQ/pub?gid=448090529&single=true&output=csv
```

**To update server data:** Edit the Google Sheet. The site reflects changes on next page load automatically.

**Google Sheet column headers** (must match exactly):

| Column | Maps to |
|---|---|
| Server Name | `name` |
| Host / Creator | `host` |
| Map | `map` |
| Server Type | `type` — must be `Legacy`, `Deathmatch`, `Playtest`, or `In-Development` |
| IP Address | `ip` — use `TBD` if not live yet |
| Restart Times (PDT) | `restarts` |
| Server Location | `region` |
| Discord | `discord` |
| Battlemetrics Page | `battlemetrics` |
| Useful links | `usefulLinks` |

### Chat Commands — Hardcoded
Defined in the `COMMANDS` array in the `<script>` block. Edit directly in `index.html` to add, remove, or update commands.

Each command object:
```javascript
{
  command: "!example",
  response: "The response text",
  type: "Twitch chat response", // or "Stream control", "Media", "On-screen indicator"
  permissions: "Everyone",      // or "VIPs", "Mods"
  note: ""                      // optional footnote shown in small text
}
```

---

## Mobile Behavior

Breakpoint: `720px`

- Top bento row stacks to single column
- Nav and social bar reduce padding
- Both tables switch from row layout to **card layout** — each row becomes a stacked block with small inline labels
- Restart times column is visible on mobile (shown as a labeled card row)
- Sort headers are hidden on mobile (no `<thead>` displayed)

---

## Conventions & Rules

- **Always use CSS variables** for colors — never hardcode hex values in new rules
- **Pink is accent only** — never use it as a background color
- **0.5px borders** throughout — thinner than default, gives a refined look
- **`escapeHTML()`** must wrap any user data rendered into the table to prevent XSS
- Social pill colors: each platform has its own border/text color defined in `.social-pill.[platform]`
- Badge colors for server types are defined as `.badge-legacy`, `.badge-dm`, `.badge-pt`, `.badge-dev`

---

## Etsy Product Image

Hosted at: `images/[filename]` in the GitHub repo.
Referenced in the HTML as: `src="images/[filename]"`

The `<img>` tag has an `onerror` handler that shows a placeholder if the image fails to load:
```html
onerror="this.style.display='none'; this.nextElementSibling.style.display='flex';"
```

---

## Links

| Destination | URL |
|---|---|
| Live site | https://icekvveenkiki.github.io |
| Custom domain | https://icekvveenkiki.com |
| Redirect domain | projectkiki.tv |
| Twitch | https://twitch.tv/icekvveenkiki |
| Etsy shop | https://icekvveenkiki.etsy.com |
| Etsy listing | https://icekvveenkiki.etsy.com/listing/4400142320/dayz-chernarus-flag-country-code-sticker |
| Instagram | https://instagram.com/icekvveenkiki |
| X / Twitter | https://x.com/icekvveenkiki |
| Discord | https://discord.icekvveenkiki.com/join |
