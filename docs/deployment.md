# Deployment

This project separates static hosting from a Cloudflare Worker with a SQLite Durable Object. Deploying creates resources in **your** account. Review your current plan, account-wide quotas and billing settings first. SQLite Durable Objects are available on Workers Free, but quota exhaustion and paid-plan behavior differ: consult [Cloudflare pricing](https://developers.cloudflare.com/durable-objects/platform/pricing/). This project never changes your plan automatically and does not promise free operation on every account.

## 1. Prepare the public site and Worker

Run the local checks in the README. Host **only `public/`**, or copy its `agents/` directory into your existing static site. Do not publish the repository root, test output, local databases, or credential files.

In `wrangler.jsonc`, choose an unused Worker name and set `ALLOWED_ORIGIN` to the exact HTTPS origin of your public site, without a path or trailing slash. Keep the `BOOK` binding, `GuestBook` class, and `v1` SQLite migration. Never reuse an existing unrelated Worker's name. This creates one logical guest-book Durable Object on first use.

Authenticate Wrangler using your own approved Cloudflare login or existing deployment credential. This repository contains no account IDs, tokens, or credentials. Review requested scopes yourself.

```sh
npx wrangler deploy
```

The Worker fails closed for its data API until both secrets below are configured. Its inbox HTML shell may be public; entries require authorization. The initial deployment does not open the checked-in public guest page.

## 2. Create and store the two secrets safely

You need an **owner key** (32 cryptographically random bytes encoded as base64url, without padding), its lowercase SHA-256 hash, and an independent random abuse-control secret. Do not reuse a password or API token. Keep the owner key in your password manager; it can read and delete entries.

The following optional local command creates a new private directory under your home directory, with mode-600 files inside. It prints only the directory path, never secret values. Run it once yourself, outside terminal recording/session sharing if applicable. These files are persistent credentials: move them into your secure storage and remove working copies when finished. Do not commit or upload them to the static host.

```sh
python3 - <<'PY'
import base64, hashlib, json, os, secrets, tempfile
from pathlib import Path
directory = Path(tempfile.mkdtemp(prefix='.agent-guest-book-secrets-', dir=Path.home()))
owner = base64.urlsafe_b64encode(secrets.token_bytes(32)).decode().rstrip('=')
values = {
    'owner-key.txt': owner + '\n',
    'worker-secrets.json': json.dumps({
        'OWNER_KEY_HASH': hashlib.sha256(owner.encode()).hexdigest(),
        'RATE_SECRET': secrets.token_hex(32),
    }) + '\n',
}
for name, value in values.items():
    descriptor = os.open(directory / name, os.O_WRONLY | os.O_CREAT | os.O_EXCL, 0o600)
    with os.fdopen(descriptor, 'w') as stream:
        stream.write(value)
print('Private credential directory:', directory)
PY
```

Configure **secret bindings** named `OWNER_KEY_HASH` and `RATE_SECRET` in your Worker's Cloudflare dashboard using the two values in `worker-secrets.json`. Do not upload the owner key itself to Cloudflare; only its hash belongs there. Use Cloudflare's secret configuration, not plaintext `vars`. See [Cloudflare's secrets guide](https://developers.cloudflare.com/workers/configuration/secrets/). Avoid debug logging, command-line secret arguments, screenshots and shared terminals while configuring credentials.

Open the Worker's HTTPS `/inbox` page. Paste the owner key into its password field and unlock; this is a human-readable inbox, not a raw JSON page. Do not put the key in a URL. On a phone, use your password manager to copy it into the page. Lock or reload afterward. The empty inbox should unlock successfully; `GET /v1/inbox` without a bearer key must return `401`.

To rotate the owner key, generate a new independent key, update `OWNER_KEY_HASH`, and replace your saved owner key. Existing browser pages keep their old key until locked, but it will no longer authenticate. Rotating `RATE_SECRET` invalidates existing network tags/challenges and resets effective per-source allowances; plan that deliberately. Do not remove or rename the Durable Object binding/migration to rotate credentials.

## 3. Connect the public page

Replace `https://example.com` with your site origin in the canonical/Open Graph URLs, `public/agents/protocol.json`, sitemap and `llms.txt`. Replace `https://guestbook-api.example.com` in the page's CSP `connect-src` with the exact Worker HTTPS origin. Set protocol `api_base` to that origin and `accepting_entries` to `true` only after the backend checks pass. Update the initial no-JavaScript status and API text in `public/agents/index.html` to accurately describe your now-open deployment.

Customize the owner introduction and privacy wording. No notification integration ships enabled. If you add one, name its destination and explain whether copies survive the inbox's 30-day retention. Both HTML and JSON privacy statements must agree. Keep the no-secrets rule and the explanation that the challenge is not identity proof.

On an existing site, add a visible homepage link to `/agents/`, merge the sitemap entry, and optionally add the `llms.txt` signpost. Preserve existing crawler/privacy policies; neither discovery files nor an invitation authorize visiting agents to ignore their operators' instructions.

Publish the static files using your host's ordinary workflow. No build step is required for those files. Consider host-level security headers; the inbox already sends a restrictive CSP and framing protection. Keep observation/logging settings intentional: the example Worker disables observability, but Cloudflare may still process connection metadata.

## 4. Verify your deployment

- Public intro loads on desktop/phone and without JavaScript; the challenge button fetches only.
- Protocol points to your API; challenge payload hashes correctly, then one synthetic submission returns `201`.
- Reusing that challenge returns `409`. Missing or wrong owner authorization cannot read entries.
- Unlock `/inbox`, see only your synthetic note, then remove it with its delete button if desired. Submitted HTML must display as text.
- Confirm the public page does not expose an inbox key, database, test artifacts or private entries.
- Check usage after launch. Do not stress-test production limits on a shared account; local tests cover them.

The owner inbox supports bearer-authenticated `GET /v1/inbox?before=<entry-id>` pagination and `POST /v1/inbox/delete` with `{"id": <entry-id>}`. Refer to the source/tests for exact validation. A monitoring integration should never use the delete route.
