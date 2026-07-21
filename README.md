# getprovenant.github.io

The public homepage for [Provenant](https://github.com/getprovenant/provenant) — a fast, source-faithful license, copyright, package, and SBOM scanner. Built with [Astro](https://astro.build); deployed to GitHub Pages at the org root.

## Develop

```sh
npm install
npm run dev      # http://localhost:4321
npm run build    # static output in dist/
npm run preview  # serve the built dist/
```

## Structure

```
src/layouts/BaseLayout.astro     Shared head (SEO/OG/JSON-LD), header, footer
src/pages/index.astro            Landing page (hero, why, teasers)
src/pages/compare/index.astro    Provenant vs ScanCode — speed + annotated quality comparison
src/pages/blog/index.astro       Blog index
src/pages/blog/[...id].astro     Renders a post from the content collection
src/content/blog/*.md            Blog posts (add a file here to publish a post)
src/content.config.ts            Blog collection schema
src/styles/global.css            Design system (tokens, both themes)
public/assets/                   Demo GIF + benchmark chart (source: provenant/docs)
```

## Adding a blog post

Drop a Markdown file in `src/content/blog/` with frontmatter (`title`, `description`, `kicker`, `standfirst`). It appears on `/blog/` and gets its own page automatically.

## Deploy

`.github/workflows/deploy.yml` builds with Astro and publishes to GitHub Pages. It is **manual-only** (`workflow_dispatch`) for now. At launch: add a `push` trigger on `main`, make the repo public, and set Settings → Pages → Source: **GitHub Actions**.

## Keeping figures in sync with the repo

The **speed figures** (hero + `/compare/`) and the **benchmark chart / demo GIF** are generated artifacts of the flagship repo. They live in `src/data/benchmarks.json` and `public/assets/`, and are refreshed by:

```sh
npm run sync            # reads ../provenant (a sibling checkout) by default
npm run sync -- /path/to/provenant
```

The script parses `docs/BENCHMARKS.md` for the headline numbers and copies `docs/scan-duration-vs-files.svg` + `docs/provenant-demo.gif`. So the flow is: regenerate the artifacts in the flagship repo → `npm run sync` here → commit. (A CI step could run this automatically on release; for now it's manual.)

The **per-repo comparison specifics** on `/compare/` (curl / Redis / tokio deltas + snippets) come from ad-hoc `compare-outputs` runs, not a standing artifact, so they stay hand-maintained and are refreshed on each stable release.

## Maintenance notes

- Keep ScanCode out of the landing-page hero/branding; it appears only as factual comparison on `/compare/` and as lineage in the blog. A single non-affiliation/trademark note lives on `/compare/` (nominative use — not legally required elsewhere).
