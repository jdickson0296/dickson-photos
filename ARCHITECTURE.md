# Architecture — Dickson Photos

Portfolio photography website built with Hugo and deployed on Cloudflare Pages.

**Live site:** https://dicksonphotos.com  
**Theme repo:** https://github.com/jdickson0296/newsroom-dicksonphotos

---

## Repository Layout

```
dickson-photos/                        ← git root
├── ARCHITECTURE.md
├── README.md
├── .gitmodules                        ← theme submodule declarations
└── dickson-photos/                    ← Hugo site root (run all hugo commands from here)
    ├── config.toml                    ← Hugo configuration
    ├── archetypes/
    │   └── default.md                 ← new-content template
    ├── content/                       ← all gallery pages and sections
    ├── data/                          ← YAML data files (menu, carousel, designer)
    ├── layouts/
    │   └── shortcodes/
    │       └── gallery.html           ← custom Lightbox2 gallery shortcode (only layout override)
    ├── static/
    │   └── images/                    ← site-level static images
    ├── themes/
    │   └── newsroom-dicksonphotos/    ← active theme (git submodule)
    ├── resources/_gen/                ← Hugo build cache (do not commit)
    └── public/                        ← build output (do not commit)
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Static site generator | Hugo (extended, ≥ 0.63.0) |
| Theme | Newsroom — custom fork (`newsroom-dicksonphotos`) |
| Styling | SASS/SCSS (compiled by Hugo extended) |
| JS | Vanilla JS (theme) + jQuery + Lightbox2 via CDN (galleries) |
| Hosting | Cloudflare Pages |
| Deployment | Automatic on merge to `main` |
| Version control | Git with theme as submodule |

---

## Hugo Configuration (`dickson-photos/config.toml`)

```toml
baseURL = "https://dicksonphotos.com/"
languageCode = "en-us"
title = "Dickson Photography"
theme = "newsroom-dicksonphotos"
paginate = 5
```

Dark mode is enabled by default (system preference detection + manual toggle via the theme).

---

## Theme

The active theme is `newsroom-dicksonphotos`, a personal fork of the open-source
[Newsroom](https://github.com/onweru/newsroom) theme (MIT license). It lives at
`dickson-photos/themes/newsroom-dicksonphotos/` and is tracked as a git submodule.

**Key theme capabilities:**
- Apple Newsroom-inspired minimalist design
- Responsive grid layout (CSS Grid + Flexbox, no jQuery in theme itself)
- Dark mode (system preference detection + toggle)
- Native lazy loading for images and iframes
- Homepage image carousel (configured via `data/carousel.yml`)
- Navigation driven by `data/menu.yml`

**Theme asset pipeline:**
- SASS source lives in `themes/newsroom-dicksonphotos/assets/sass/`
- Hugo extended compiles SASS → CSS at build time (cached in `resources/_gen/`)
- JS entry point: `themes/newsroom-dicksonphotos/assets/js/index.js`

**The only local layout override** is `layouts/shortcodes/gallery.html`, which adds
Lightbox2-powered photo galleries on top of the base theme.

---

## Content Organization

All content lives under `dickson-photos/content/` using Hugo
[page bundles](https://gohugo.io/content-management/page-bundles/).

```
content/
├── _index.md                         ← homepage
├── about.md
├── contact/
│   └── _index.md
├── graduations/
│   ├── _index.md                     ← section landing page
│   └── <shoot-name>/                 ← one bundle per shoot
│       ├── index.md
│       └── img/                      ← JPGs for this shoot
├── portraits/
│   ├── _index.md
│   └── <shoot-name>/
│       ├── index.md
│       └── img/
├── sports/
│   └── ...
└── travel/
    └── ...
```

**Naming convention for shoot folders:** `<location>-<month>-<year>` (e.g. `baker-beach-jul-2022`)

**Each shoot's `index.md`:**

```yaml
---
title: "Shoot Title"
date: 2024-12-31T00:00:00-08:00
draft: false
---
{{< gallery >}}
```

The `{{< gallery >}}` shortcode reads all images from the bundle's `img/` directory,
resizes them to 1000×1000 px at quality 100 via Hugo image processing, and renders
them as a Lightbox2 gallery.

---

## Data Files (`dickson-photos/data/`)

| File | Purpose |
|---|---|
| `menu.yml` | Top navigation links (Home, Graduation, Portrait, Travel, Sports, Contact) |
| `carousel.yml` | Homepage carousel — lists image paths (`static/images/home-gallery/h1.jpg` … `h7.jpg`) |
| `designer.yml` | Photographer name and Instagram URL shown in theme footer |

---

## Deployment

Cloudflare Pages builds and deploys the site automatically when a pull request is
merged into `main`. No manual deploy step is required.

**Cloudflare Pages build settings (configured in the Cloudflare dashboard under Workers & Pages → dickson-photos → Settings):**

| Setting | Value |
|---|---|
| Git repository | `jdickson0296/dickson-photos` |
| Build command | `hugo` |
| Build output directory | `public` |
| Root directory | `dickson-photos` |
| Production branch | `main` |
| Automatic deployments | Enabled |
| Build system version | Version 3 |
| Build cache | Disabled |

Cloudflare automatically deploys on every merge to `main` and generates a preview
URL for every open pull request so you can review changes before merging.

---

## Hugo Commands

All commands must be run from the **`dickson-photos/` subdirectory** (the Hugo site root).

```bash
cd dickson-photos
```

### Local development server

```bash
hugo serve
```

- Starts a live-reload server at http://localhost:1313
- Watches for file changes and rebuilds automatically
- Add `-D` to also render pages marked `draft: true`

```bash
hugo serve -D
```

### Production build

```bash
hugo
```

- Outputs static files to `public/`
- Minifies HTML by default in the theme
- Does **not** include draft pages

### New content

```bash
# New gallery shoot (use the appropriate section: graduations, portraits, travel, sports)
hugo new portraits/location-mon-year/index.md
```

Then add your JPG images to `content/portraits/location-mon-year/img/`.

### Check Hugo version

```bash
hugo version
```

Hugo **extended** is required (for SASS compilation). The version string will include
`extended` if correctly installed.

### Update theme submodule

```bash
git submodule update --remote themes/newsroom-dicksonphotos
```

---

## UI Development — Lessons Learned

Hard-won notes from editing the theme and layout. Read this before touching CSS, JS, or layout files.

### Never edit the theme submodule directly

All customisation goes in the project's `layouts/` and `static/` directories. Hugo merges
these with the theme at build time — project files always win. Files to know:

| Project file | Overrides |
|---|---|
| `layouts/index.html` | Homepage layout |
| `layouts/_default/baseof.html` | Root HTML shell (html, head, body) |
| `layouts/partials/head.html` | `<head>` content |
| `layouts/partials/styles.html` | CSS loading |
| `layouts/partials/footer.html` | Footer + JS loading |
| `layouts/shortcodes/gallery.html` | Per-shoot photo gallery |
| `static/css/custom.css` | All custom CSS overrides |

### The compiled CSS URL must be relative

`styles.html` uses `resources.Fingerprint` to generate a cache-busted CSS filename.
Always use **`.RelPermalink`**, not `.Permalink`, for both CSS and JS resources:

```html
<!-- correct — works on localhost and production -->
<link rel="stylesheet" href="{{ $styles.RelPermalink }}" ...>
<script src="{{ $scripts.RelPermalink }}"></script>

<!-- wrong — hardcodes baseURL, breaks local dev -->
<link rel="stylesheet" href="{{ $styles.Permalink }}" ...>
<script src="{{ $scripts.Permalink }}"></script>
```

If styles or JS appear broken locally (white background, hamburger not working, arrows not
working), check the rendered HTML — if asset URLs start with `https://dicksonphotos.com`
instead of `/`, this is the cause. Restart `hugo serve` after fixing and hard refresh.

### Dark mode is not automatic — it must be forced

The theme's dark mode depends on two things: `prefers-color-scheme: dark` (system setting)
OR a `colorMode` value in `localStorage`. Neither is reliable as a default.

The current setup forces dark mode through three layers so it works regardless of system
preference or stored browser state:

1. `baseof.html` sets `data-mode="dim"` on `<html>` by default
2. A script at the top of `<body>` clears any stored `"lit"` preference before the theme JS runs: `localStorage.removeItem('colorMode')`
3. `custom.css` hardcodes the dark mode CSS variables on `html` with `!important`

Note: the dark mode toggle UI (`mode.html`) is commented out in the theme, so there is
intentionally no way for visitors to switch to light mode.

### Overriding the carousel arrows

The theme's carousel arrow click handlers use a brittle `offsetLeft` calculation to
scroll between slides. This breaks when the carousel is full-width (100vw). The arrows
are also hidden by default (`display: none`) in the theme's inline styles.

The fix lives in `layouts/partials/footer.html` — after the theme JS loads, a script
clones the arrow elements (removing the original event listeners) and replaces them
with `scrollBy`-based handlers:

```js
clone.addEventListener('click', function() {
  ul.scrollBy({ left: direction * ul.offsetWidth, behavior: 'smooth' });
});
```

### Full-width carousel breakout

The carousel is inside `<main>` (outside `.wrap`) and uses the CSS viewport breakout
technique to span 100vw regardless of the parent's max-width:

```css
.carousel {
  width: 100vw;
  position: relative;
  left: 50%;
  transform: translateX(-50%);
}
```

### Hugo deprecated APIs (as of v0.128)

The theme submodule has not been updated to use the new Hugo APIs. These are fixed in
the project-level overrides:

| Old (broken) | New (correct) | File |
|---|---|---|
| `resources.ToCSS` | `css.Sass` | `layouts/partials/styles.html` |
| `paginate = 5` | `[pagination] pagerSize = 5` | `config.toml` |
| `$paginator.PageSize` | `$paginator.PagerSize` | `layouts/index.html` |

### Portrait vs landscape in the carousel

The carousel uses `object-fit: contain` so portrait shots display in full without
cropping (black bars fill the sides). Do not change this to `object-fit: cover` —
it crops portrait subjects at the face.

---

## Adding a New Photo Shoot (end-to-end)

1. Create a feature branch: `git checkout -b <shoot-name>`
2. `cd dickson-photos`
3. `hugo new <section>/<shoot-folder>/index.md` — fill in title and set `draft: false`
4. Copy JPGs into `content/<section>/<shoot-folder>/img/`
5. `hugo serve` — verify gallery renders correctly at http://localhost:1313
6. Commit and open a PR against `main`
7. Cloudflare Pages generates a preview URL for review
8. Merge PR → Cloudflare automatically deploys to https://dicksonphotos.com
