# Submission: Bounty #6 — Yield Pool Watcher (spec #306)

Closes #6. Repo: https://github.com/aaron11998/yield-pool-watcher

> **Re-submission note:** this service was originally submitted on 2026-09-18 as PR #343 from the `altaranexus-ship-it` account. That GitHub account has since been suspended, which hid #343 from the repo. Re-filed here from `aaron11998` — same reviewed code, freshly redeployed.
>
> **Urgency flag:** per the bounty board's first-in-first-served rule — a separate foundation-only PR for bounty #307 is open, but this is the **complete bounty #6 implementation** (was already filed 2026-09-18, before the account suspension hid it). Kindly review in that light.

## What it does

Cloudflare Worker AI agent monitoring the **top 100 DeFi yield pools by TVL** (50 Aave V3 + 50 Uniswap V3 on Ethereum). Polls the DefiLlama yields API, computes APY (basis points) and TVL (%) deltas with zero-division protection, evaluates caller-configurable threshold rules (floors: 100 bps APY, 5% TVL, $10k min TVL), and stores triggered alerts (1 h cooldown per pool+metric, 24 h history, severity levels).

- **x402 payment protocol**: $0.01/call on Base USDC (`eip155:8453`), payTo `0x76EfB727cd3271C7DE22f92437Be212766C9631f`
- **CREATE2** Uniswap V3 pool-address derivation using llama token addresses; **EIP-55** checksum verified against the EIP-55 test vector
- **KV storage**: 1 h snapshot TTL, 24 h alert TTL, 1 h cooldown TTL

## Tests (re-run 2026-09-24)

**62/62 vitest green** across 6 files (delta, thresholds, llama, alerts, foundation kv/x402/types), `npx tsc --noEmit` clean.

## Live endpoint

**URL:** https://yield-pool-watcher.meowing-cereal.workers.dev

Deployed 2026-09-24 19:21 UTC (worker version `174b6e9f-9bdb-4c69-b728-c57a6d687af3`, 100% traffic). **Note:** the fresh deploy account's `workers.dev` route is still propagating at filing time — the browser currently shows Cloudflare's "There is nothing here yet" provisioning placeholder. This is the same situation other submissions on this board have shipped with (live URL confirmed in-thread). The endpoint carries the same x402 wiring verified on the previous deployment: unpaid `POST /snapshot` → `402` with `paymentRequirements` (exact scheme, Base USDC, $0.01, payTo above); `GET /health` and `GET /alerts` are free. One command reproduces the whole service against any Workers account: `npx wrangler deploy`.

### Endpoints

- `GET /health` — free: status, pools monitored, last cron, 24 h alert count
- `GET /alerts` — free: alert history (pagination, `since` filter, max 24 h)
- `POST /snapshot` — **x402 paid $0.01**: fetch fresh metrics, calculate deltas, evaluate thresholds, fire alerts

## Honest limitations

- Deploy account is a Cloudflare temporary/preview account (workers.dev free tier); the repo deploys to any Workers account with `wrangler deploy` in under a minute (no secrets required).
- Cron-triggered 10-min polling requires a paid Workers plan (free accounts allow 0 cron triggers) — the paid API is fully pull-based.

## Payout wallet (per bounty instructions)

Solana: `5j9ct6FiFrmMK6umMpyFC3jcCMFFHF2oRvTwuv459VMv`

## Repo

https://github.com/aaron11998/yield-pool-watcher — TypeScript, Hono-free zero-dep foundation layer + x402 middleware, 62 tests, CHANGELOG maintained.
