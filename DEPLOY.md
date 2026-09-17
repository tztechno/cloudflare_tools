# Deploying cf-todo via Wrangler CLI

This is the manual procedure for deploying the `cf-todo` Cloudflare Worker (TODO list backed by Workers KV) from the command line, as opposed to using the Cloudflare dashboard.

## Prerequisites

- Node.js and npm installed on your machine
- The `cf-todo` project folder (contains `src/index.js`, `wrangler.toml`, `package.json`)
- A terminal (e.g. the integrated terminal in VS Code)

## One-time setup

### 1. Install dependencies

Open a terminal, navigate into the project folder, and run:

```bash
cd cf-todo
npm install
```

This installs Wrangler (Cloudflare's CLI tool) and its dependencies into the project.

### 2. Log in to Cloudflare

```bash
npx wrangler login
```

This opens your browser to Cloudflare's OAuth login page. Click "Allow" to authorize Wrangler. The terminal will print `Successfully logged in.` when done.

### 3. Create a KV namespace

```bash
npx wrangler kv namespace create TODO_KV
```

This creates a new Workers KV namespace dedicated to this app and prints output like:

```
[[kv_namespaces]]
binding = "TODO_KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

Copy the `id` value — you'll need it in the next step.

### 4. Update `wrangler.toml`

Open `wrangler.toml` in an editor and find this block:

```toml
[[kv_namespaces]]
binding = "TODO_KV"
id = "REPLACE_WITH_YOUR_KV_NAMESPACE_ID"
preview_id = "REPLACE_WITH_YOUR_PREVIEW_KV_NAMESPACE_ID"
```

Replace both `id` and `preview_id` with the ID you copied in step 3 (it's fine to use the same ID for both):

```toml
[[kv_namespaces]]
binding = "TODO_KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
preview_id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

Save the file.

### 5. Deploy

```bash
npx wrangler deploy
```

On success, Wrangler prints the live URL, e.g.:

```
https://cf-todo.<your-subdomain>.workers.dev
```

Open that URL to confirm the TODO list loads and that adding/checking/deleting tasks works.

## Making changes later

Once the one-time setup above is done, shipping a code change is just:

1. Edit `src/index.js` (or other project files) as needed.
2. Run `npx wrangler deploy` again from the `cf-todo` folder.

No dashboard interaction is needed — the KV binding and namespace are already configured in `wrangler.toml` and stay in place across deploys.

## Notes

- `npx wrangler kv namespace create TODO_KV` only needs to be run once. Running it again creates a *new*, empty namespace — don't repeat it unless you intend to start with fresh data.
- The Wrangler version used here (3.114.17) reported that a newer version (4.x) is available. Upgrading is optional; run `npm install --save-dev wrangler@4` if you want to update later.
- Keep `wrangler.toml` in version control if you use git — it records the exact KV namespace this Worker is bound to.
