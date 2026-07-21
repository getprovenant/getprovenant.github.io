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

## Maintenance notes

- **Speed figures** on `/compare/` come from `provenant/docs/BENCHMARKS.md`. **Quality figures** (curl, Redis) come from `compare-outputs` runs at the commits noted on the page; regenerate and refresh on each stable Provenant release.
- Keep ScanCode out of the landing-page hero/branding; it appears only as factual comparison on `/compare/` and as lineage in the blog. Keep the non-affiliation disclaimer in the footer.
