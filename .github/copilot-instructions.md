## Copilot instructions for this repo (Hugo blog)

Big picture
- This is a Hugo static site for a personal blog. Core config lives in `config.toml`. The active theme is `international-typo` (see `themes/international-typo/`).
- Hugo publishes the site into the `docs/` directory (`publishDir = "docs"`). GitHub Pages serves from that folder. Do not hand-edit `docs/`; regenerate it with Hugo.
- Content lives under `content/` (e.g., `content/blog/...`, `content/about.md`). Static assets live in `static/` and are copied as-is to the output (e.g., `static/images/*`, `static/CNAME`).

Local development
- Preferred (Codespaces/port-forwarded):
  - `hugo serve -b=/ --appendPort=false` (baseURL set to `/` avoids broken links; no port suffix)
- Alternative (Docker): `docker compose up hugo` using `docker-compose.yml` (serves on http://localhost:1313).
- Standard local: `hugo serve` also works; the `-b=/` flag is mainly for cloud/dev-tunnel environments.

Build and deploy
- Build the site: `hugo --cleanDestinationDir` (writes to `docs/`).
- Ensure `static/CNAME` exists; Hugo will copy it so `docs/CNAME` is present after build. Don’t manually edit `docs/CNAME`.
- GitHub Pages is already configured (badge in `README.md`). Commit changes to `content/`, re-run `hugo`, commit `docs/`, and push.

Content conventions
- Posts go in `content/blog/` and typically follow `YYYY-MM-DD-title.md`. Permalinks are configured as `/:year/:month/:day/:slug/` in `config.toml`.
- Use the archetype `archetypes/default.md` when creating new content: `hugo new blog/2025-08-12-my-post.md` creates front matter with `draft: true`.
- Before publishing, set `draft: false`. Add tags via front matter, e.g.:
  ```
  ---
  title: "My Post"
  date: 2025-08-12T00:00:00Z
  tags: ["hugo", "notes"]
  draft: false
  ---
  ```

Assets, links, and styling
- Place images under `static/images/` and reference with absolute paths, e.g. `![alt](/images/example.png)`.
- Custom CSS is included via `css/custom.css` (see `Params.customCSS` in `config.toml`). Put overrides in `static/css/custom.css`. Avoid editing files inside `themes/international-typo/`; prefer overrides.

Theme and layout overrides
- To customize templates, add files under `layouts/` (not currently present) to override theme templates selectively. Keep theme changes out of `themes/` unless intentionally updating the theme.

Key files to know
- `config.toml`: baseURL, theme, permalinks, menus, taxonomies (`tags`), output formats, highlighting style, etc.
- `docker-compose.yml`: containerized `hugo server` on port 1313.
- `README.md`: quick serve command used in Codespaces.
- `static/`: CNAME, verification files, images, CSS.

Gotchas
- Don’t commit manual edits to `docs/`; always rebuild with Hugo. If `docs/` is wiped, re-run `hugo` to restore (including `CNAME` via `static/CNAME`).
- When serving behind tunnels/preview URLs, keep `-b=/ --appendPort=false` to avoid mixed/incorrect URLs.
