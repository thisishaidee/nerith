# NERITH controlled devnet testing

**Purpose:** test wallet connection, ownership proof and transaction preparation against an isolated Solana devnet Worker without touching the production Worker or production D1 database.

## Safety boundary

- Local development uses `wrangler.devnet.toml` only with `wrangler dev --local` and local D1 commands.
- Remote Android/browser testing uses the separate `wrangler.devnet.remote.toml` configuration and the dedicated `nerith-devnet-testing` Worker.
- The remote Worker uses its own D1 database (`nerith-devnet-testing`), devnet RPC and devnet USDC mint. It is not production.
- Never deploy the local-only `wrangler.devnet.toml` file. Never replace the remote test Worker or D1 identifiers with production values.
- Use a dedicated low-balance devnet test wallet. Never use a primary treasury wallet.
- Keep local `.dev.vars` files untracked. Never paste secrets, seed phrases or private keys into chat, issues, logs or screenshots.
- The remote test Worker has its own `ADMIN_BOOTSTRAP_TOKEN` secret configured. Its value is not readable through the Cloudflare API. Use the securely stored value; if it is unavailable, rotate only the devnet test secret in Cloudflare. Never reuse the production token.

## Remote Android test endpoint

- Worker: `nerith-devnet-testing`
- URL: https://nerith-devnet-testing.haideecoded-e0f.workers.dev
- D1: `nerith-devnet-testing` (dedicated database)
- Cluster: Solana devnet
- RPC: `https://api.devnet.solana.com`
- USDC mint: `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU`
- Telegram: not configured; Telegram remains deferred until website acceptance tests pass.

The remote deployment workflow runs typecheck and the full automated test suite before applying migrations to the isolated devnet D1 and deploying this Worker. A passing deploy or health check does not prove that a wallet app opens or that a signing flow works on a physical Android device.

## Local database and Worker

From the repository root:

```sh
npm install
npm run build:dashboard
npx wrangler d1 migrations apply DB --local --config wrangler.devnet.toml --persist-to .wrangler/state/nerith-devnet
npx wrangler dev --local --config wrangler.devnet.toml --persist-to .wrangler/state/nerith-devnet
```

The repository currently has no committed npm lockfile, so `npm ci` is not yet valid. Adding and reviewing a lockfile, then switching CI and test instructions to `npm ci`, remains a pre-production supply-chain hardening requirement.

## Test order

1. Health check and database migration status.
2. Create a disposable business and admin session in the isolated devnet Worker.
3. Open the dashboard from Android Chrome and test each wallet-app handoff.
4. Register and prove ownership of a devnet test wallet.
5. Verify wallet role assignment and owner/admin authorization.
6. Prepare a small devnet USDC funding leg.
7. Sign with one wallet, then test two distinct signers.
8. Submit the exact signed payload and verify Worker-side checks.
9. Confirm finalized transaction status and actual fee reconciliation.
10. Confirm reserve replenishment finalizes before final-payment preparation.
11. Exercise rejection, disconnect, stale blockhash, duplicate submission, RPC timeout and replay cases.

## Evidence classification

- Unit tests and local D1 integration are automated evidence.
- A successful remote Worker deployment is environment evidence, not wallet-device evidence.
- A wallet app opening is not proof of connection or signing.
- A wallet connection is not proof of signing.
- A successful broadcast is not proof of settlement.
- Only a finalized on-chain transaction independently reconciled by the Worker counts as a completed funding leg.
- Hermes HTTP 402 challenge capture is unpaid evidence, not a successful paid x402 transaction.
