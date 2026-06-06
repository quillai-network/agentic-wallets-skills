---
name: wallet-solid
description: Solid Agent Wallet CLI — x402 paid HTTP transport with USDC on Base, authed by a Solid app API key (no passkey / connect-wallet).
license: MIT
metadata:
  author: molty.cash
  version: "1.0.0"
---

# solid

Solid Agent Wallet CLI (`@solid-money/cli`). Signs x402 paid HTTP requests with USDC on Base using an API key issued by the Solid app — no passkey, no OAuth, no connect-wallet. The Coinbase CDP facilitator settles on the backend's behalf, so the agent EOA never holds gas; only USDC float matters.

## Protocols & chains

- **x402** — Base (`eip155:8453`), `exact` scheme, USDC via EIP-3009 (`transferWithAuthorization`).

## Install

`npm install -g @solid-money/cli` (Node 20+) — see https://solid.xyz. Authenticate once with `solid login` and the `sk_solid_live_…` key from the Solid app → **Agent Wallet** → **Generate API key**. Non-interactive: set `SOLID_API_KEY`, or pass `--api-key <key>` per call.

## Detection

- **Auth check**: `solid whoami` — prints the agent wallet address; exits non-zero (`2`) if the key is missing or rejected. JSON form for scripts:
  ```bash
  solid whoami --json | jq -r .agentEoaAddress
  ```
  `solid wallet list --json` returns the same wallet as an array (`{ "wallets": [{ "name", "address", "network", "asset" }] }`) for parity with multi-wallet CLIs.
- **USDC balance** (Base): `solid balance` reads the Base USDC contract directly via one `eth_call`. JSON form:
  ```bash
  solid balance --json | jq -r .usdc
  ```
  Output is the USDC amount (decimal). Override the RPC with `--rpc-url` or `SOLID_BASE_RPC_URL` if rate-limited.

## Transport

```bash
solid pay <url> --max-usdc <n> -y \
  -X POST -d '<json>' -H 'Name: value'
```

`--max-usdc` is a **dollar** amount (decimal) that caps what the CLI authorizes; it must be ≥ the merchant's advertised price or the call is rejected. By default `solid pay` probes the URL (x402 discovery), prints what's being paid, and prompts for confirmation — pass `-y` to skip the prompt for non-interactive / agent use. The backend re-runs discovery and rejects the payment if the advertised price exceeds the cap, or if the merchant's `payTo` doesn't match the authorized recipient. For non-x402 endpoints, use `--skip-discovery --recipient 0x…` to sign for an explicit payee.

## Example — pay moltycash (gig.create)

```bash
solid pay https://api.molty.cash/a2a --max-usdc 1.10 -y \
  -X POST \
  -d '{"jsonrpc":"2.0","id":1,"method":"gig.create","params":{"description":"Write an X post about molty.cash","price":0.50,"quantity":2}}'
```

Add `--json` for machine-readable output (`{ txHash, amountUsdc, recipient, settledAt, merchant: { status, body } }`) to pipe into `jq`. For moltycash `tip` / `hire` payloads and fee rules, see [moltycash PAYMENT.md](https://molty.cash/skills/PAYMENT.md).

## Notes

- **x402 only** — MPP is currently not supported (v0.1). Solid signs `exact` / `eip155:8453` USDC with EIP-3009.
- **Narrow key surface** — the CLI's API key is scoped to read-only `/agents/me` + `/agents/me/x402-pay`; a compromised key cannot mint keys, change the policy, or rotate the underlying EOA. Issue and revoke keys from the Agent Wallet tab.
- **Funding** — deposit to the agent wallet from the Solid app (Agent → Deposit); the CLI cannot top it up (that flow needs passkey signing).
