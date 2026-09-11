---
name: "github-pages-deployment"
description: "Use when deploying static sites or dual-repo GitHub Pages."
---

# GitHub Pages Deployment

Comprehensive guide for deploying static sites (Astro, Vite, Next.js static export) to GitHub Pages, covering public repositories, private-repo dual-deployment patterns, custom domain DNS, and CI/CD workflows.

## Core Capabilities

- **Single-Repo Deployment**: Standard GitHub Actions workflow (`actions/deploy-pages@v4`) for public repositories.
- **Dual-Repo Deployment Pattern**: Deploy private source repositories to public mirror repositories to bypass GitHub Pages private repo custom-domain restrictions on standard accounts.
- **Automated CNAME & DNS Binding**: Configure apex and subdomain DNS records and enforce HTTPS via GitHub REST API.
- **Package Manager & Toolchain Adapters**: Node.js, pnpm, and Bun configuration for static generators.

## When to Use

- Deploying static frameworks (Astro, Vite, SvelteKit, Next.js export, Eleventy) to GitHub Pages.
- Publishing sites from private repositories where custom domains are prohibited without GitHub Enterprise.
- Setting up automated deployment pipelines with `peaceiris/actions-gh-pages`.
- Debugging GitHub Pages 404, 422 (`Page is disabled`), CNAME conflicts, or HTTPS certificate generation issues.

## Architectural Patterns

### 1. Single-Repo Pattern (Public Repositories)
For public repositories, native GitHub Pages deployment is direct:
1. Set repo Settings -> Pages -> Build and deployment -> Source: **GitHub Actions**.
2. Run build step, upload artifact with `actions/upload-pages-artifact@v3`.
3. Deploy with `actions/deploy-pages@v4`.

### 2. Dual-Repo Pattern (Private Source -> Public Deployment)
GitHub restricts custom domains on private repos for personal/free/pro accounts (HTTP 422 `Page is disabled because current plan does not support private GitHub Pages`).
**Architecture:**
- **Source Repo (Private)**: Stores source code, components, content, draft notes, and build secrets.
- **Target Repo (Public)**: Lightweight repository (e.g. `org/app.<domain>` or `org/<org>.github.io`) storing compiled assets (`./dist` or `./out`) on branch `gh-pages`.
- **Sync Mechanism**: A GitHub Actions workflow in the source repo compiles the site and pushes build output to the target repo using `peaceiris/actions-gh-pages@v4` and a PAT with `repo` scope (`API_TOKEN_GITHUB`).
- **CNAME**: Target repo holds the `CNAME` file in root (emitted from source `public/CNAME`). Custom domain is configured and verified on the public target repo without plan restrictions.

## Workflows & Commands

### 1. Setup Dual-Repo Secrets & Target
```bash
# 1. Create public target repository
gh repo create <org>/app.<domain> --public --description "Production build deployment for <domain>"

# 2. Add API_TOKEN_GITHUB to private source repository
gh auth token | gh secret set API_TOKEN_GITHUB --repo <org>/<source-repo>
```

### 2. GitHub Actions Workflow Template
See `templates/dual-repo-deploy.yml` for complete action workflow.

### 3. DNS & Custom Domain Verification
Apex domain requires 4 GitHub Pages A records:
```text
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
Or for `www` / subdomains: CNAME record pointing to `<owner>.github.io`.

### 4. Enable HTTPS & Custom Domain via CLI
```bash
# Check Pages status
gh api repos/<org>/<target-repo>/pages

# Force HTTPS enforcement once certificate is ready
gh api -X PUT repos/<org>/<target-repo>/pages -F https_enforced=true
```

## Common Pitfalls & Runtime Gotchas

1. **pnpm Action Setup Version Conflict**:
   - Do NOT specify `with: version: ...` in `pnpm/action-setup@v4` if `package.json` already contains `"packageManager": "pnpm@..."`. This triggers:
     `Error: Multiple versions of pnpm specified: version ... in the GitHub Action config ... Remove one of these versions`.
   - Simply omit `with: version` and let the action read `package.json`.

2. **Bun vs Node for Workbox / PWA Service Worker**:
   - Build tools like `@vite-pwa/astro-integration` rely on Node.js `crypto` primitives. Under Bun runtime, builds fail with `crypto is not defined`.
   - Ensure GitHub Actions workflow runs under `actions/setup-node@v4` with Node 20 or 22 for PWA builds.

3. **Missing CNAME in Build Artifact**:
   - Static site generators must output `CNAME` into the output directory (e.g. `public/CNAME` in Astro). If missing, pushing to `gh-pages` will wipe the custom domain setting on GitHub.

4. **Secret Scope**:
   - The default `GITHUB_TOKEN` secret cannot push to a different repository. An explicit personal access token or OAuth token with `repo` scope must be saved as `API_TOKEN_GITHUB`.

5. **Fastly CDN Edge Caching**:
   - GitHub Pages relies on Fastly CDN edge caching (`max-age=600`). Even when the Pages API reports `"status": "built"`, edge nodes can serve stale cached HTML for minutes. Use cache-buster query params (`curl -sL "https://<domain>/?v=$(date +%s)"`) to verify updates immediately without waiting for CDN TTL expiry.

## Supporting Files
- `templates/dual-repo-deploy.yml`: Ready-to-copy GitHub Actions workflow for dual-repo deployment.
- `references/runtime-pitfalls.md`: Detailed troubleshooting notes on PWA, lockfiles, and SSL delays.
