# Takaichi Cabinet Approval Poll (Cloudflare Worker + KV)

A single Cloudflare Worker that serves a two-choice poll page (支持する /
支持しない), stores votes in Workers KV, limits each visitor to one vote per
calendar week, and renders a public weekly chart of the results — no
separate backend or database needed.

## How it works

- **Frontend + API in one Worker.** `src/index.js` serves the HTML page at
  `GET /` and exposes `GET /api/status`, `POST /api/vote`, `GET /api/results`.
- **One-vote-per-week enforcement.** Each vote is written to
  `vote:<weekId>:<voterHash>` in KV, where `weekId` is an ISO week computed in
  JST (e.g. `2026-W38`) and `voterHash` is a salted SHA-256 hash of the
  visitor's IP address + User-Agent. Because the week is baked into the key,
  voting again simply becomes possible once a new week starts — nothing needs
  to expire or be reset.
- **Weekly aggregation without a counter.** The vote's choice is stored both
  as the KV value and as `metadata` on the key. `GET /api/results` lists all
  `vote:*` keys and tallies `metadata.choice` per week — this reads metadata
  only (no per-key `get()` calls), so it stays cheap even with many votes, and
  there's no read-modify-write counter that could race under concurrent
  voting.
- **Public results.** Anyone who opens the page sees the current week's
  totals and a bar chart of every week so far (Chart.js, loaded from cdnjs).

### Known limitations (by design, for a casual public poll)

- The IP + User-Agent fingerprint deters casual repeat voting; it is not
  Sybil-proof (a new IP, e.g. from a different network or VPN, bypasses it).
- Workers KV is eventually consistent, so under very heavy simultaneous
  traffic a handful of duplicate votes are theoretically possible.
- This is a self-selected, informal poll of whoever clicks the link — not a
  scientifically representative survey. Say so wherever you share it (the
  page already includes a disclaimer footer).

## Prerequisites

- A Cloudflare account (free tier is enough).
- Node.js installed locally.
- The Wrangler CLI: `npm install -g wrangler` (or just use `npx wrangler ...`
  below without installing it globally).

## Deploy

1. **Log in to Cloudflare:**
   ```
   wrangler login
   ```

2. **Create the KV namespace:**
   ```
   wrangler kv namespace create POLL_KV
   ```
   This prints an `id`. Copy it into `wrangler.toml`, replacing
   `REPLACE_WITH_YOUR_KV_NAMESPACE_ID`.

   (Optional, for local dev with `wrangler dev`):
   ```
   wrangler kv namespace create POLL_KV --preview
   ```
   and paste that id into `preview_id` in `wrangler.toml`.

3. **Set the voter-hash salt as a secret** (any random string; it just makes
   the fingerprint harder to precompute — it doesn't need to be memorable):
   ```
   wrangler secret put VOTER_SALT
   ```

4. **Deploy:**
   ```
   wrangler deploy
   ```
   Wrangler prints the live URL, something like:
   ```
   https://takaichi-cabinet-poll.<your-subdomain>.workers.dev
   ```
   That URL is both the voting page and the public results page — nothing
   else to host.

5. *(Optional)* Attach a custom domain from the Cloudflare dashboard
   (Workers & Pages → your worker → Settings → Triggers → Custom Domains) if
   you'd rather share a branded URL.

## Posting to X

Once you have the deployed URL, post it from your own X account (this
session has no X/Twitter connection, so posting has to be done manually).
Draft text:

> 【アンケート】高市内閣を支持しますか？
> 投票はこちら → <あなたのURL>
> ※投票はおひとり様1週間に1回までです
> #高市内閣 #支持率調査

Feel free to re-post the same link weekly — the chart keeps accumulating
history automatically, one bar per week.

## Local development

```
wrangler dev
```
This runs the Worker locally against the preview KV namespace, at
`http://localhost:8787`.
