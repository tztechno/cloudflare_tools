# Deploying a Static HTML Site via Wrangler CLI (Cloudflare Pages)

This is the manual procedure for deploying a plain static site (HTML/CSS/JS, no backend) to Cloudflare via the CLI, instead of the dashboard's "Upload your static files" flow. The dashboard flow assigns a random auto-generated name (e.g. `weathered-art-5a4a`) unrelated to your project. Deploying via CLI lets you control the project name directly — by default it's based on your folder name.

## Prerequisites

- Node.js and npm installed
- A folder containing your static site, with an `index.html` at its root (plus any CSS/JS/image files)
- A terminal
- Already logged in to Cloudflare via `npx wrangler login` (see `DEPLOY.md` — the same login covers both Workers and Pages; no need to log in again if you've already done it once)

## Example folder layout

```
my-site/
├── index.html
├── style.css
└── script.js
```

## One-time setup and deploy

### 1. Navigate into your site folder

```bash
cd my-site
```

The folder name (`my-site` here) is what Wrangler will suggest as the project name.

### 2. Deploy

```bash
npx wrangler pages deploy .
```

The `.` means "deploy the current folder."

- If this is the first deploy, Wrangler asks whether to create a new Pages project. It suggests a project name based on your folder name — press Enter to accept it, or type a different one.
- If you're not logged in yet, it will prompt you to run `wrangler login` first (same as the Workers flow).

### 3. Skip the prompt (optional but recommended)

To avoid the interactive prompt entirely and guarantee the name you want, pass it explicitly:

```bash
npx wrangler pages deploy . --project-name=my-site
```

On success, Wrangler prints the live URL:

```
https://my-site.pages.dev
```

## Making changes later

Whenever you edit `index.html` (or any file in the folder), redeploy with the same command:

```bash
npx wrangler pages deploy . --project-name=my-site
```

This updates the existing project — it does not create a new one, as long as the project name matches.

## Notes

- Folder/project naming rule: Cloudflare Pages project names must be lowercase, and can only contain letters, numbers, and hyphens (no underscores, no spaces, no Japanese characters). If your folder name doesn't fit this, choose an explicit `--project-name` that does.
- This is separate from the Workers deployment in `DEPLOY.md`. Use this (Pages) for sites with no server-side logic; use the Workers approach when you need an API, KV storage, or other backend behavior.
- If your static site later needs a small backend (e.g. saving form submissions), that's a sign to move to the "Workers with static assets" pattern (a `wrangler.toml`/`wrangler.jsonc` with an `assets` block plus a Worker script) rather than plain Pages — ask if you want a walkthrough of that when the need comes up.
