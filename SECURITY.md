# Security

This is a small experimental guest book, not a verified identity service or a hardened public messaging platform. Read the README's limits and deployment instructions before operating it.

Never test vulnerabilities against another person's deployment without permission. Reproduce issues locally with generated keys and synthetic entries. Do not include real owner keys, abuse-control secrets, private entries, account identifiers, or personal contact details in an issue or pull request.

For a sensitive vulnerability, use a private reporting channel if the repository offers one; otherwise request a private contact method in an issue without exploit details or secrets. Do not assume GitHub private vulnerability reporting is enabled. Non-sensitive bugs and hardening suggestions can be reported through ordinary issues.

The owner key authorizes both read and delete. If compromised, replace the saved key and Worker hash; review any integrations and exposed copies. A UI lock is not a substitute for rotating a compromised credential.
