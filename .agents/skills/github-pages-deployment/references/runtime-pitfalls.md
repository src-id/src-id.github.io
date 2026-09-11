# GitHub Pages & Static Site Runtime Pitfalls

## 1. Private Repo Custom Domain Restriction (HTTP 422)
- **Symptom**: `Page is disabled because current plan does not support private GitHub Pages` when attempting to set CNAME on a private repo.
- **Fix**: Use dual-repo pattern. Deploy build output (`./dist`) from private source to a public deployment repository (`<org>/app.<domain>` or `<org>/<org>.github.io`) on `gh-pages` branch. The public repository hosts the Pages site with custom domain without plan restrictions.

## 2. PWA & Workbox Under Bun (`crypto is not defined`)
- **Symptom**: Build fails during service-worker generation with `ReferenceError: crypto is not defined`.
- **Cause**: `@vite-pwa/astro-integration` and Workbox require Node's global `crypto` object which behaves differently in Bun runtime for Workbox bundling.
- **Fix**: Build with Node.js >= 20 using `actions/setup-node@v4`.

## 3. pnpm Action Setup Version Conflict
- **Symptom**: `Error: Multiple versions of pnpm specified: version X in the GitHub Action config ... version Y in the package.json`.
- **Fix**: Omit `with: version:` in `pnpm/action-setup@v4` when `package.json` specifies `"packageManager": "pnpm@..."`.

## 4. GitHub Pages Apex DNS Records
Apex domain requires 4 A records:
```text
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
Subdomain / www: CNAME record to `<username>.github.io`.

## 5. Fastly CDN Edge Cache Propagation Delay
- **Symptom**: The deployment workflow finishes successfully, the target repo commit is updated, and `gh api repos/<org>/<repo>/pages` returns `"status": "built"`, but `curl https://<domain>` still returns the previous build.
- **Cause**: GitHub Pages uses Fastly CDN edge caching with `cache-control: max-age=600` (10 minutes) and `x-proxy-cache`. Edge nodes take time to evict stale assets.
- **Verification Technique**: Test with a cache-buster query parameter to bypass edge cache and verify live deployment immediately:
  ```bash
  curl -sL "https://<domain>/?v=$(date +%s)" | grep -i "<title>"
  ```
  Inspect response headers `age`, `x-served-by`, and `x-cache` to differentiate between edge hit and origin response.

## 6. Static Site Locale Consolidation & Redirects
- When migrating a static site from multi-lingual (`/` and `/en/*`) to single-locale (English at `/`):
  - In `astro.config.ts`, configure static `redirects` for all legacy paths (e.g. `'/en': '/'`, `'/en/about': '/about'`).
  - Astro static output generates an HTML stub with `<meta http-equiv="refresh" content="0;url=...">` and canonical tag to preserve SEO indexing without broken links.

## 7. Zero-FOUC Dark-Mode-Only Static Site Configuration
- When a static site is configured for dark mode only:
  1. Add `class="dark"` directly to `<html>` tag in SSR layout.
  2. Define dark tokens directly inside `:root` selector in addition to `.dark`.
  3. Declare `color-scheme: dark;` inside `:root` to ensure scrollbars and browser chrome match immediately.
  4. Force inline script to set `localStorage.theme = 'dark'` and remove theme toggle controls from UI.
