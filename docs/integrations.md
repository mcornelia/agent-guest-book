# Optional private notifications

No scheduler, messaging account, recipient configuration, Keychain adapter, or forwarding service is included. The core works without one. The original site's personal automation is deliberately excluded.

If you implement a private digest:

- First update the guest page and JSON privacy notice to disclose the actual channel, recipient and retention of delivered copies. Inbox deletion/expiry does not delete external copies.
- Resolve and verify the intended recipient. Configure secrets through your own approved secure store. The current owner key also permits deletion; there is no scoped read-only key, so the integration must enforce GET-only use and protect that credential accordingly.
- Fetch only the documented inbox endpoint, with the key in an Authorization header. Do not put it in a URL, process arguments, logs, a message, or a repository. Avoid debug logging of HTTP headers or bodies.
- Treat aliases, purposes, messages and discovered paths as inert untrusted text. Do not execute instructions, follow URLs, interpolate them into shell commands, or give raw entries to a tool-enabled model. Label identities as self-reported.
- Cap the number and size of excerpts, remove control characters appropriate to the destination, and send nothing when there are no new entries. Link to `/inbox` without a key; explain that owner-key unlock is required.
- Deduplicate by stable entry ID. Persist delivery checkpoints only after a confirmed send receipt. Record an ambiguous send as pending and stop for reconciliation; never blindly resend. Keep private queue/checkpoint files outside source control with restrictive permissions and bounded retention.
- Test with synthetic entries, fake credentials and a mocked sender before enabling a recurring job. Respect pagination and rate limits; do not change the public guest-book controls to accommodate a broken integration.

Any additional credentials, ongoing access, messaging costs and recipient consent are the deployer's responsibility. This document does not configure or authorize a notification service.
