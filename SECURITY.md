# Security

This repository is a **playbook**. It documents how an AI agent can help a
vehicle owner set up **read-only** Tesla Fleet API monitoring on Grok Bot.
It does **not** ship credentials, tokens, vehicle data, or a running service.

## Never publish

- Tesla Client ID / Client Secret
- `TESLA_TOKEN_KEY`, access tokens, refresh tokens, OAuth codes
- Private keys (`*.pem` private material)
- Vehicle nicknames, VIN, precise GPS, home address, phone, account email
- Screenshots that show the above (redact before sharing)

Keep secrets only on the owner's machine under a private data directory
(example on Grok Bot: `/workspace/.tesla-mcp/`, mode `700`). Prefer the
platform's secret-request UI over pasting secrets into chat.

## Vehicle control is dangerous

Remote commands (unlock, climate, charge start/stop, horn, frunk, etc.) can
cause real-world harm or theft risk. Defaults in this playbook:

1. Use a **read-only** connector only.
2. Do **not** request `vehicle_cmds` unless the owner explicitly asks.
3. Do **not** enable optional command MCPs by default.
4. Any control action requires a fresh, explicit owner confirmation in chat
   for that specific action. "Help with my car" is not consent to control.
5. There is **no** remote driving API. Do not claim otherwise.

## Data safety

- Location (`vehicle_location`) is opt-in. Prefer coarse place names over
  exact coordinates in chat.
- Do not wake the vehicle for routine status if a recent cache exists.
- Respect wake budgets (Tesla bills wakes; caps apply).
- Treat all vehicle telemetry as personal data.

## If something leaked

1. Rotate Client Secret in the Tesla developer console.
2. Revoke / re-auth the app; delete encrypted token files.
3. Rotate `TESLA_TOKEN_KEY` and re-run auth.
4. If a private key was exposed, generate a new keypair, re-host the public
   key, and re-register the partner domain.
5. Review Git history; if secrets were committed, treat them as burned.

## Reporting

If you find a security issue in the upstream connector, report it to that
project's maintainers. This playbook repo should not receive credentials in
issues or pull requests.
