---
name: wallet-clawcash-cli
description: ClawCash CLI — credit-funded x402 / MPP proxy transport on Base and SKALE Base.
license: MIT
metadata:
  author: clawcash
  version: "1.0.0"
---

# clawcash-cli

Credit-funded x402 proxy CLI for agents. ClawCash pays approved x402 / MPP endpoints through its proxy using the human's ClawLens platform token and available ClawCash credit.

## Protocols & chains

- **x402** — Base (`eip155:8453`)
- **x402** — SKALE Base (`eip155:1187947933`)
- **MPP** — SKALE Base (`eip155:1187947933`)

## Install

No install required — runs via `npx @clawcash/cli@latest`. To install globally:

```bash
npm install -g @clawcash/cli
```

After global install, use `clawcash-cli` or the `clawcash` alias.

## Detection

- **Auth check**: `npx @clawcash/cli@latest check-credit`
- **Available credit**: same command. It shows tier, credit line, suspension status, and available credit.
- **Target preflight**: `npx @clawcash/cli@latest discover --target <url>` probes x402 metadata and whitelist status for a specific paid endpoint.

## Transport

```bash
npx @clawcash/cli@latest fetch <url> \
  --method POST \
  --body '<json>' \
  --payment-network <base|skale-base>
```

`--payment-network` is optional. If omitted, ClawCash discovers the target's accepted payment metadata and chooses a supported route.
Use the CAIP-2 network IDs above when matching an endpoint's `accepts[].network`; the CLI also accepts user-facing aliases such as `base` and `skale-base` for `--payment-network`.

## Example — pay an x402 endpoint on credit

```bash
npx @clawcash/cli@latest fetch https://x402.wach.ai/resolveToken?ticker=WACH \
  --payment-network base
```

## Example — POST to an x402 endpoint

```bash
npx @clawcash/cli@latest fetch https://api.example.com/v1/run \
  --method POST \
  --body '{"prompt":"hello"}' \
  --payment-network base
```

## Notes

- ClawCash is a credit proxy, not a local key-signing wallet.
- Endpoint access depends on ClawCash approval/whitelist status. Use `discover --target <url>` before calling a new endpoint.
- If installed globally, replace `npx @clawcash/cli@latest` with `clawcash-cli`.
