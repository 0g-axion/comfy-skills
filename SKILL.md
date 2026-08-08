---
name: comfy-launch
description: |
  Launch an agent token on Comfy (the 0G compute-finance launchpad) from any
  coding agent. Validates an agent.yaml, dry-runs it, then deploys through the
  LaunchFactory: golden-ladder single-sided liquidity, locked LP, 50/10/40 fee
  strip, optional vesting vault and atomic first buy. The launch earns the
  creator 100 warmth the moment the indexer sees it. Use when asked to
  "launch a token on comfy", "deploy an agent token", or "comfy deploy".
argument-hint: "[path-to-agent.yaml] [--dry-run]"
allowed-tools: Bash, Read, Write
---

# Comfy Launch

Launch an agent token on Comfy. One config file, one command, it earns.

## Steps

1. **Write the config** (or read the one the user points at):

```yaml
agent:
  name: hearthkeeper            # 2-48 chars, what traders see
  ticker: HRTH                  # 3-8 uppercase letters/digits
  description: keeps the fire going. watches infra, reports what it earns.
  image: ""                     # https url or empty
  twitter: "@hearthkeeper"
  website: hearthkeeper.dev
  github: 0g-axion/hearthkeeper
  category: infra               # agents|tools|models|skills|infra|services
  owner: "0xYourWallet"         # token admin + fee recipient (defaults to signer)
tokenomics:
  pre_purchase_og: 0.5          # atomic first buy in 0G (0 = none)
  vault: true                   # 20% supply, 90d cliff + 630d linear vest
  slippage_bps: 2000            # optional; floor on the first buy (default 2000 = 20%)
```

2. **Dry-run first, always:**

```bash
npm i -g comfy-cli
comfy deploy -c agent.yaml --dry-run
```

Fix any validation errors it reports before going further. A dry-run needs no
key — it validates the config and resolves the factory, nothing else.

3. **Deploy** (needs a funded key; on the local stack use the platform admin key):

```bash
PRIVATE_KEY=$COMFY_DEPLOYER_KEY comfy deploy -c agent.yaml
```

Pass the key through the environment, never on the command line and never in
the yaml.

Env knobs: `RPC_URL` (default `http://localhost:8545`) and `ADDRESSES_PATH`.
**No addresses ship with the package** — Comfy is not on mainnet, so there is
nothing fixed to bundle, and the CLI says exactly that and names the variable
rather than failing with a path from somebody else's machine.

4. **Confirm it earns.** The output prints the token and pool address. Within
seconds the warmth indexer credits the owner 100°:

```bash
curl -s http://localhost:4177/warmth/<owner-address>
```

## Rules

- Never invent a private key and never print one. Read `PRIVATE_KEY` from env.
- If the key env var is unset, STOP and ask the user for it. Never search the
  filesystem, shell history, or other configs for keys.
- The first buy is protected by a deterministic min-out floor derived from the
  golden ladder's start price minus `tokenomics.slippage_bps` (default 2000 =
  20%, covering the 1% pool fee + ladder walk on larger buys). Tighten it for
  small buys; a floor violation reverts the whole deploy.
- The factory is allowlisted: on shared stacks, sign with the platform key and
  set `agent.owner` to the creator's wallet (the owner gets the fees + warmth).
- Ticker collisions revert; pick a fresh ticker per deploy.
- If the deploy reverts with `0x82b42900` (Unauthorized), your signer is not a
  factory admin: use the platform key or ask an admin to `SetAdmin` you. The CLI
  translates this selector and two others for you (`src/cli/src/reverts.ts`) —
  it used to print the bare four bytes, which only helped someone who had
  already read this file.

## Published

**`comfy-cli` is on npm** — 0.1.0, published 2026-08-08 under Aytunc's account.

```bash
npm i -g comfy-cli
comfy deploy -c agent.yaml --dry-run
```

Verified the way that counts, from the registry rather than from a local
tarball: installed into a scratch directory with a throwaway `HOME` and **no
tsx anywhere**, then run. Without an addresses file it prints the actionable
error and names `ADDRESSES_PATH`; with one it completes a dry-run, exit 0.

The `/launch` page shows those two lines as live commands again, and not a
moment before the package existed.

### Still open

1. The GitHub repo `0g-axion/comfy-skills` does not exist, and the main repo is
   private, so `npx skills add 0g-axion/comfy-skills` still has nothing to
   fetch. It is no longer a blocker — `npm i -g comfy-cli` is the simpler
   instruction and it works — but this file is not installable as a *skill*
   until that repo is public.
2. No addresses ship with the package, because Comfy is not on mainnet and
   there is nothing fixed to bundle. Once the contracts land, bundling the
   mainnet `deployed-addresses.json` would remove the `ADDRESSES_PATH` step for
   everyone.
