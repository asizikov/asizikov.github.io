## Copilot instructions for this repo

- This repo is a Hugo site. Do not commit generated output such as `public/` or `docs/`.

- Local development:
  - For Codespaces, tunnels, or other port-forwarded environments, prefer `hugo serve -b=/ --appendPort=false` to avoid broken or mixed URLs.
  - `hugo serve` is fine for normal local development.
  - Docker option: `docker compose up hugo`.

- Content workflow:
  - Blog posts live in `content/blog/` and usually follow `YYYY-MM-DD-title.md`.
  - New posts start as drafts; set `draft: false` before publishing.
  - Put images in `static/images/` and reference them with absolute paths such as `/images/example.png`.

- Customization rules:
  - Put CSS overrides in `static/css/custom.css`.
  - Put template overrides in `layouts/`.
  - Avoid editing `themes/international-typo/` unless intentionally updating the theme itself.

- CI and deployment:
  - Pull requests run validation checks.
  - Pushes to `main` deploy the site via GitHub Actions.
  - Keep `static/CNAME` in place for the custom domain.
