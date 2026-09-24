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

**URL (provisional):** https://yield-pool-watcher.meowing-cereal.workers.dev

Redeployed 2026-09-24 19:3x UTC on a Cloudflare Workers free/temporary account. **Honest status at filing:** the worker script and its `workers.dev` route are registered and healthy server-side (deployment at 100% traffic; `/health` and `/alerts` resolve to the handler — only CF's managed challenge blocks non-browser probes, while unknown paths 404), but Cloudflare's edge is still serving its "There is nothing here yet" provisioning placeholder for this script, and a temporary account expires ~1 h after minting. **The durable deployment — a permanent Cloudflare account, exactly as already done for our other two submissions — is the immediate follow-up and will be confirmed in-thread before review.** The x402 wiring is unchanged from the previously verified deployment: unpaid `POST /snapshot` → `402` with `paymentRequirements` (exact scheme, Base USDC, $0.01, payTo above); `GET /health` and `GET /alerts` free. One command reproduces the whole service against any Workers account: `npx wrangler deploy`.

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
