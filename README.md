# agentic-wallets-skills

A **template skill** for x402 / MPP paid-endpoint developers. Reference this catalog from your own endpoint spec so AI agents calling your service can pay you with any of 12 installed wallet CLIs — no per-endpoint integration of 12 different wallet SDKs required.

The skill is **wallet-side only**. It tells an AI agent how to detect, pick, and invoke any of the 12 supported wallet CLIs. Your endpoint-side spec — URLs, request bodies, fees, cap math — lives in your own skill file.

## How it works

The pattern is two skills in tandem:

1. **Your endpoint skill** — what your service is, the URLs, request/response shape, fee schedule, anything vendor-specific
2. **This catalog** (`agentic-wallets/SKILL.md`) — how to drive any of the 12 wallet CLIs

An AI agent reading your endpoint spec follows the link to this catalog, picks a wallet with the human's input, and combines both to make the paid request.

## Hosted skill (canonical)

```
https://molty.cash/skills/agentic-wallets/SKILL.md
```

Per-wallet docs live at:

```
https://molty.cash/skills/agentic-wallets/wallets/<wallet>.md
```

`<wallet>` ∈ `bankr`, `circle`, `lobstercash`, `solid`, `awal`, `purl`, `agentcash`, `onchainos`, `tempo`, `moonpay`, `pay-sh`, `link-cli`.

## For endpoint developers — how to integrate

In your endpoint's own SKILL.md (or PAYMENT.md, or whatever you call your agent-facing payment spec), reference this catalog. Example:

```markdown
## Wallet selection

This endpoint accepts payment via x402 on Base and Solana, and MPP on Tempo.

To make a payment, follow the agentic-wallets catalog:
  https://molty.cash/skills/agentic-wallets/SKILL.md

Pick any wallet whose `Protocols & chains` intersects the chains this
endpoint accepts. Then fetch the wallet's doc:
  https://molty.cash/skills/agentic-wallets/wallets/<wallet>.md

Combine the wallet's transport pattern with this endpoint's URL + body
+ fee structure (below).
```

That's it. Your endpoint spec stays focused on what's unique to your service. The wallet-side complexity (12 different CLIs × x402 vs MPP × multiple chains) is delegated to this catalog.

### Reference example

[moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md) is a real-world endpoint-side spec for `api.molty.cash` (tip / hire / gig). Read it to see the pattern in action — it links to this catalog for the wallet side, then documents only what's specific to moltycash (endpoints, JSON-RPC payloads, fees).

The pattern is repeatable for any paid HTTP endpoint.

## Self-hosting

If you want to host a copy of this catalog at your own domain (so the link in your endpoint spec is on your own infrastructure), clone this repo and serve the contents under `<your-host>/agentic-wallets/`. Update the per-wallet `curl` examples in your fork to use your host.

```bash
git clone https://github.com/moltycash/agentic-wallets-skills.git
```

## For AI agents — how to use the skill at runtime

If you're an agent (Claude Code, Codex, Cursor, etc.) and you've been told to make a paid HTTP call, the agent-facing instructions are inside [`SKILL.md`](./SKILL.md) — fetch and read that, not this README.

Canonical hosted entry point:

```bash
curl https://molty.cash/skills/agentic-wallets/SKILL.md
```

## Install / consume the skill

This is an [agentskills.io](https://agentskills.io/specification)-style skill — a folder with `SKILL.md` at the root. Three common ways to consume it:

### 1. Direct fetch (zero install)

```bash
curl https://molty.cash/skills/agentic-wallets/SKILL.md
curl https://molty.cash/skills/agentic-wallets/wallets/<wallet>.md
```

### 2. Claude Code skill

```bash
git clone https://github.com/moltycash/agentic-wallets-skills.git \
  ~/.claude/skills/agentic-wallets
```

Claude Code picks up `SKILL.md` automatically.

### 3. Git submodule

```bash
git submodule add https://github.com/moltycash/agentic-wallets-skills.git skills/agentic-wallets
git submodule update --init
```

## Wallets covered

| Wallet | Chains | Protocols |
|---|---|---|
| [bankr](./wallets/bankr.md) | Base | x402 |
| [circle](./wallets/circle.md) | Base | x402 |
| [lobstercash](./wallets/lobstercash.md) | Base | x402 |
| [solid](./wallets/solid.md) | Base | x402 |
| [awal](./wallets/awal.md) | Base, Solana | x402 |
| [purl](./wallets/purl.md) | Base, Solana, Tempo | x402, MPP |
| [agentcash](./wallets/agentcash.md) | Base, Solana, Tempo | x402, MPP |
| [onchainos](./wallets/onchainos.md) | Base | x402 (signer-only) |
| [tempo](./wallets/tempo.md) | Tempo | MPP |
| [moonpay](./wallets/moonpay.md) | Solana | x402 |
| [pay.sh](./wallets/pay-sh.md) | Solana | x402 |
| [link-cli](./wallets/link-cli.md) | Stripe (fiat USD via card / link) | MPP |

## Per-wallet doc shape

Each wallet doc follows a uniform structure so agents can parse them consistently:

```
---
name: wallet-<name>
description: <one-line>
license: MIT
---

# <wallet name>

## Install         — one-line install hint
## Protocols & chains  — x402 / MPP × chain list
## Detection      — auth check + USDC balance command
## Transport      — how to POST a body to any paid endpoint
## Example        — concrete copy-paste example
## Notes          — wallet-specific quirks (optional)
```

## License

MIT.

## Contributing

PRs welcome for new wallet integrations or corrections to existing ones. New wallets should follow the shape above and include verified `Detection` commands (don't fabricate CLI commands — capture them from the wallet's actual `--help` output).
