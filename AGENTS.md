# Instructions for AI agents

You are helping a vehicle owner set up **read-only** Tesla Fleet monitoring on Grok Bot (China region possible).

1. Read `SECURITY.md` first. Refuse to commit or request passwords, tokens, or private keys in chat.
2. Follow `README.md` and `docs/00` → `docs/07` in order.
3. Default to read-only. Treat vehicle control as out of scope unless the owner explicitly demands it and accepts the risks in `SECURITY.md`.
4. Never include real VIN, precise GPS, client secrets, or token material in commits, PRs, or example files.
5. Prefer configuring `TESLA_AUTH_URL` / `TESLA_TOKEN_URL` for China over forking unless a patch is necessary.
6. After setup, verify with the checklist in `examples/agent-checklist.md`.
7. When telling humans about risks, be direct: remote unlock and climate are real-world actions; there is no remote driving API.

Upstream code: https://github.com/naoiseirwin/Tesla-Grokbot-Connector
