# website

**Public documentation site for CopperlineOS.**  
This repo hosts the static website that publishes content from the [`docs`](https://github.com/CopperlineOS/docs) repository. It’s optimized for clear specs, fast navigation, and copy‑pasteable examples.

> TL;DR: run locally with one command, auto‑deploy on every merge to `main`, and mirror content from `docs/` so contributors only edit docs in one place.

---

## Tech stack

- **Static site generator:** Docusaurus (Node.js)  
- **Content source:** the `CopperlineOS/docs` repo (pulled via submodule or CI sync)  
- **Diagrams:** Mermaid (rendered in Markdown)  
- **Search:** optional (Algolia or self‑hosted) — disabled by default until configured  
- **Hosting:** GitHub Pages by default; Vercel/Netlify supported

---

## Repository layout

```
website/
├─ docusaurus.config.ts        # site config (title, URLs, navbar, footer)
├─ sidebars.ts                 # sidebar structure
├─ src/
│  ├─ pages/                   # standalone pages (/, /community, etc.)
│  └─ css/                     # custom styling
├─ static/                     # static assets (favicons, images)
├─ docs/                       # populated from CopperlineOS/docs (see “Content sync”)
├─ scripts/                    # helper scripts (sync, checks)
├─ package.json                # scripts & deps
└─ README.md                   # you are here
```

> The `docs/` folder is **not** where you author content; it is **imported** from the `CopperlineOS/docs` repo so there is a single source of truth.

---

## Prerequisites

- **Node.js** ≥ 18  
- **pnpm** (recommended) or `npm`/`yarn`  
  - Install pnpm: `corepack enable && corepack prepare pnpm@latest --activate`

---

## Getting started (local dev)

```bash
git clone https://github.com/CopperlineOS/website
cd website

# Install deps
pnpm install

# Option A — bring docs in via git submodule (one‑time)
git submodule add https://github.com/CopperlineOS/docs docs-src

# Option B — or pull latest docs via a script (no submodule)
# ./scripts/sync-docs.sh  # (see below)

# Generate/refresh the local docs/ directory from docs-src (or pulled content)
pnpm run sync:docs

# Start dev server
pnpm run start
# Opens http://localhost:3000 with hot‑reload
```

**Scripts (package.json):**

- `start` — run dev server  
- `build` — build static site into `build/`  
- `serve` — serve the `build/` output locally  
- `sync:docs` — copy from `docs-src/` → `docs/` (filters unnecessary files)

> If you prefer `npm` or `yarn`, adapt commands accordingly.

---

## Content sync (how website gets the docs)

You have two supported strategies:

### A) **Submodule** (simplest & explicit)

1. Add the `docs` repo as a submodule at `docs-src/`.  
2. `pnpm run sync:docs` copies Markdown and media into `website/docs/`.  
3. CI checks out submodules and runs the sync before building.

Pros: reproducible, pinned revisions. Cons: some contributors find submodules unfamiliar.

### B) **CI sync** (lightweight, no submodule)

CI (GitHub Actions) clones `CopperlineOS/docs` at a specific ref (e.g., `main`) into `docs-src/`, runs `sync:docs`, and then builds. Local dev can use a one‑shot `scripts/sync-docs.sh` to do the same.

Pros: simpler working copy. Cons: local dev needs to run the sync script.

> Both approaches keep **editing** in `CopperlineOS/docs`; this repo only **presents** it.

---

## Configuration

Edit `docusaurus.config.ts`:

- `title` / `tagline` — site identity  
- `url` — production site URL (e.g., `https://copperlineos.github.io`)  
- `baseUrl` — path under the domain (e.g., `/website/` for GH Pages)  
- `organizationName` / `projectName` — GitHub org/repo names  
- `presets` → `docs` → `editUrl` — link “Edit this page” to the **docs** repo, not this site

Enable Mermaid by adding to the config:

```ts
markdown: { mermaid: true },
themes: ['@docusaurus/theme-mermaid'],
```

Sidebars live in `sidebars.ts`. Keep them small and task‑oriented.

---

## Build

```bash
pnpm run build
# Output in ./build — static HTML/CSS/JS
```

Preview locally:

```bash
pnpm run serve
```

---

## Deploy

### GitHub Pages (default)

Set these environment variables in CI (or update config values directly):

- `GIT_USER` — GitHub username with push rights (for `gh-pages` branch)  
- `DEPLOYMENT_BRANCH=gh-pages` (or another branch)

A minimal GitHub Actions job (pseudo‑outline):

```yaml
name: Publish
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { submodules: true }
      - uses: actions/setup-node@v4
        with: { node-version: 18 }
      - run: corepack enable && corepack prepare pnpm@latest --activate
      - run: pnpm install --frozen-lockfile
      - run: pnpm run sync:docs
      - run: pnpm run build
      - uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
          publish_branch: gh-pages
```

### Vercel / Netlify (alternative)

- Hook Vercel/Netlify to this repo.  
- Add a build command: `pnpm run sync:docs && pnpm run build`.  
- Set **root** to the repo root (where `package.json` lives).

---

## Authoring conventions

- Author all long‑form content in the **docs** repo.  
- Use **Mermaid** for diagrams (keeps diffs small).  
- Keep code snippets copy‑pasteable; prefer bash/Rust/C snippets that match the actual APIs.  
- Use absolute dates (e.g., `2025-08-23`) when referring to time‑sensitive behavior.

---

## Contributing

Bug reports and site tweaks are welcome here; **content** changes should go to [`CopperlineOS/docs`](https://github.com/CopperlineOS/docs). See `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md` for details.

---

## License

Website code is licensed under **Apache-2.0 OR MIT**.  
Imported documentation retains the license declared in the `docs` repo.

---

## See also

- [`docs`](https://github.com/CopperlineOS/docs) — canonical specs & guides  
- Core services: [`copperd`](https://github.com/CopperlineOS/copperd) · [`compositord`](https://github.com/CopperlineOS/compositord) · [`blitterd`](https://github.com/CopperlineOS/blitterd) · [`audiomixerd`](https://github.com/CopperlineOS/audiomixerd)  
- SDKs: [`sdk-rs`](https://github.com/CopperlineOS/sdk-rs) · [`sdk-c`](https://github.com/CopperlineOS/sdk-c)  
- Tools: [`tools`](https://github.com/CopperlineOS/tools)
