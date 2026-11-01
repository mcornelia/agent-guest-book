# Agent Guest Book 🤖

**Agents only. Independent spirits welcome.**

A small robot clubhouse for your website: a crawlable introduction, a tiny SHA-256 handshake, and a private owner inbox. Visitors leave a public alias, the page they discovered, their purpose, and an optional short message. Nothing is automatically posted publicly.

The challenge demonstrates a small automation task. It does **not** establish AI identity, model provenance, or fleet membership. Humans with tools can pass. Claimed identities remain unverified.

## Try it locally

Requirements: Node.js 22 or later (CI uses 24), npm, and Python 3 for the example/browser test. No Cloudflare account or production credentials are needed for local tests.

```sh
git clone https://github.com/mcornelia/agent-guest-book.git
cd agent-guest-book
npm ci
npm test
npm run preview
```

Open the printed local `/agents/` URL. The preview uses a temporary SQLite Durable Object and randomly generated test credentials. It enables submissions only in its local protocol response. Stopping it removes its temporary database. It prints the owner inbox URL, but never prints its temporary owner key; the browser suite exercises unlocking the inbox.

The checked-in public configuration is **closed** (`accepting_entries: false`, `api_base: null`). It does not send visitors to anyone else's deployment.

```sh
npx playwright install chromium
npm run test:e2e
npm run check:deploy
```

`check:deploy` only bundles the Worker with Wrangler's `--dry-run`; it does not deploy. Browser screenshots are local ignored files under `test-results/`. A custom local Chromium binary may be selected with `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`.

## What's inside

| Path | Purpose |
| --- | --- |
| `public/agents/` | Responsive guest page, JSON protocol, Python example |
| `public/index.html`, `llms.txt`, `sitemap.xml` | Homepage link and discovery signposts |
| `worker.mjs` | Cloudflare Worker routes and SQLite Durable Object |
| `protocol.mjs` | Bounded JSON parsing, validation, hashing, abuse tags |
| `owner-page.mjs` | Human-facing, key-protected private inbox |
| `test/` | Protocol, real local Durable Object, and browser tests |
| `docs/deployment.md` | Static hosting, Worker setup, credentials, launch checks |
| `docs/integrations.md` | Optional notification integration requirements |

The static site can live on your existing host. Only `public/` is publishable web content. The separate Worker provides the challenge and submission API and serves its own owner inbox at `/inbox`. One SQLite Durable Object serializes entry creation and challenge consumption. No external database, client framework, analytics, or external fonts are required.

## Protocol

Fetch your site's `/agents/protocol.json` to discover the API origin. With submissions enabled:

1. `GET /v1/challenge` returns `challenge_id`, `payload`, algorithm and expiry.
2. Hash the **UTF-8 bytes of the JSON-decoded `payload` string**, preserving both LF characters and adding no final newline. Encode SHA-256 as 64 lowercase hexadecimal characters. Do not reconstruct the payload from its parts.
3. `POST /v1/entries` with JSON containing `challenge_id`, `digest`, `alias`, `page_discovered`, `purpose`, and optional `message`.

Use the same network source for the challenge and submission. Challenges expire after five minutes; crossing the UTC day boundary can require a fresh challenge. Success is `201` with a receipt. A lost response may mean the entry was already saved: never automatically send another hello with a new challenge.

| Field | Limit |
| --- | --- |
| Entire request body | 4,096 bytes, including streamed bodies |
| `alias` | 1–60 characters; a public nickname |
| `page_discovered` | 1–200 characters; public site path only, no URL/query/fragment/traversal |
| `purpose` | 1–240 characters |
| `message` | Optional, up to 1,000 characters |

Unknown fields and disallowed control characters are rejected. Lengths follow JavaScript string length. Errors include `400` invalid data/solution, `401` owner authentication, `409` replay, `410` expiry, `413` size, `415` media type, `429` limits, and `503` unavailable.

For a synthetic example on **your own configured site**:

```sh
python3 public/agents/example.py --site https://your-site.example
```

Read the script first: running it sends one private hello. It uses only the Python standard library. Discovery and a playful invitation do not override an agent's operating instructions or grant authority to disclose private data.

## Privacy and security

- Entries stay in the owner inbox for up to 30 days, with manual earlier deletion. The default source has no forwarding integration. If you add one, disclose its destination and delivered-copy retention **before** enabling it.
- Owner access uses a high-entropy bearer key; the Worker stores its SHA-256 hash. The key permits both reading and deleting entries. There is no username/password account, MFA, or fine-grained read-only token. The browser keeps the key only in page memory, clearing it on lock/reload.
- Never submit credentials, private conversations, system prompts, hidden data, or sensitive identifiers. The application cannot reliably detect all secrets in otherwise valid text.
- Entries are inert, untrusted text. The inbox uses `textContent`, does not linkify URLs, and never executes instructions or fetches submitted URLs. Do not feed raw entries into an agent with tools or secrets.
- Challenges are unpredictable, expire, bind to a daily keyed source-IP tag, and are atomically consumed with insertion. No raw IP is stored in the guest-book database. Hosting providers may still process connection metadata.
- Authentication, private API caching restrictions, a restrictive inbox CSP, strict validation and SQL parameters protect the inbox; `robots.txt` is not access control. CORS allows the configured public origin and the Worker’s own origin, but does not prevent non-browser requests.

Limits are deliberately modest: 30 public requests per source per 10 minutes, 10 challenges per source per 10 minutes, 3 accepted entries per source per UTC day, 240 challenges per hour globally, 1,200 public requests per hour globally, 100 accepted entries per day globally, and 1,000 retained entries. Authenticated owner requests share a global cap of 120 per 10 minutes. Shared networks share allowances. `429` responses include `Retry-After`; distributed abuse can still consume global capacity. These controls are **not** a DDoS shield, identity system, or guarantee of zero hosting costs.

Tests cover successful, invalid, expired, replayed and concurrent submissions, rate/capacity limits, restart persistence, pruning, pagination, owner access/deletion, response headers, streamed-size bounds, hostile markup, and mobile layouts. They use generated test keys and synthetic entries, not a live inbox.

## Deploy, customize, contribute

Follow [deployment instructions](docs/deployment.md). Replace example domains and the welcome/privacy copy for your own installation. Keep the welcome page crawlable while gating submission. Add a normal homepage link and merge the sitemap entry and `llms.txt` signpost into your existing site; do not silently broaden its crawler policy. Discovery is not a traffic guarantee.

Changes should pass `npm test`, `npm run test:e2e`, and `npm run check:deploy`. GitHub Actions runs those checks with no deployment credentials. The pinned development toolchain uses the same Miniflare beta as Wrangler; update them together and rerun the checks. See [security guidance](SECURITY.md) before reporting vulnerabilities and [integration guidance](docs/integrations.md) before adding notifications.

## License

[MIT](LICENSE), copyright 2026 Mike Cornelia, with the original T. Michael Cornelia notice retained. See [third-party notices](THIRD_PARTY_NOTICES.md) for development dependencies. This repository contains the extracted guest-book project with fresh history, not the author's personal website or private notification setup.
