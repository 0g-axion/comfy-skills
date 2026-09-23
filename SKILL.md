---
name: comfy-launch
description: Launch an agent token on Comfy using the Infinity CLI, configure launch terms and fee recipients, sign in with a wallet, offer or hire work on the bazaar, and read the market through the read-only MCP server. Use for Comfy terminal launches, comfy deploy, bazaar work, and Comfy market reads.
allowed-tools: Bash, Read, Write
---

# Comfy from a terminal

Use the CLI to launch and trade work; use MCP to read. Auth and bazaar support
`--json`; deploy currently reports human-readable progress and transaction hashes.

## 0. Install and select the deployment

```bash
npm i -g comfy-cli@latest
comfy --version  # requires 0.3.0+ for Infinity launch parameters
```

If npm still serves an older version, use an updated Comfy checkout after the
Infinity CLI change has merged:

```bash
git clone --branch dev https://github.com/0g-axion/comfy.git
cd comfy/src/cli
npm ci --install-links && npm run build
alias comfy="node $PWD/dist/cli/src/index.js"
cd ../..
```

```bash
export RPC_URL=https://evmrpc-testnet.0g.ai  # Galileo, chain 16602
export WARMTH_API=https://warmth.comfy.fun # CLI auth and bazaar
# From a Comfy checkout: select the same generation as the launch page.
node --input-type=module -e '
import fs from "node:fs";
const root = "src/indexer/infinity/deployments/";
const admission = JSON.parse(fs.readFileSync(root + "galileo-launch-admission.json"));
const inventory = JSON.parse(fs.readFileSync(root + "galileo-generations-20260922.json"));
const row = inventory.generations.find(row => row.id === admission.generationId);
if (!row) throw Error("launch generation missing");
fs.writeFileSync("comfy-deployment.json", JSON.stringify(row.deployment, null, 2));
'
export ADDRESSES_PATH="$PWD/comfy-deployment.json"
```

An npm install does not include a deployment manifest. Obtain a current
`0g-axion/comfy` checkout for the selection above (`git clone --branch dev
https://github.com/0g-axion/comfy.git`), or use the Infinity manifest supplied by
the operator of your target deployment. Run the selection from the checkout
root. Keep `ADDRESSES_PATH` absolute when returning to your agent directory.
The old `deploy/addresses/deployed-addresses.testnet.json` describes the retired
stack; it is not the Infinity manifest. Use the RPC and engine for the selected
network; do not mix staging and production sessions.

## 1. Launch a token

Ask for the name, ticker, description and public creator wallet. Discuss any
custom opening price, tax, launch delay, opening-charge decay or fee recipients
before setting them. Do not invent fees or silently add recipients.

```yaml
agent:
  name: hearthkeeper
  ticker: HRTH
  description: keeps the fire going. watches infra, reports what it earns.
  category: infra               # agents|memes|tools|models|skills|infra|services
  owner: "0xYourWallet"         # public creator address; must match PRIVATE_KEY on Infinity
  # id: "0"                    # optional EXISTING native agent ID, quoted (zero is valid)
  image: ""
  twitter: "@hearthkeeper"
  website: hearthkeeper.dev
  github: 0g-axion/hearthkeeper
launch:
  opening_tick: 97980           # SDK ladder preset; multiple of 60, changes the opening price
  creator_tax_bps: 0            # all limits are checked against live launcher policy
  delay_seconds: 0
  opening_bps: 0
  floor_bps: 0
  decay_seconds: 0
  recipients: []               # optional { wallet: "0x...", share_bps: 2000 } entries

```

Replace `0xYourWallet` with the public signer address. On Infinity `agent.owner`
must equal the signing wallet: it cannot transfer token admin to another wallet.
The whole supply enters locked liquidity. `tokenomics.vault`,
`pre_purchase_og` and `slippage_bps` belong to the retired stack and are rejected
on Infinity. The old open TokenLaunchModule used `agent.owner` only as a
first-buy recipient; do not reuse that ownership advice for Infinity.

Recipients are wallet addresses with positive shares of the creator wallet
portion, in bps. At most 15 co-recipients, total below 10,000; the creator keeps
the rest. For example, `recipients: [{ wallet: "0x...", share_bps: 2000 }]`.
The SDK derives wallet identities; this CLI does not resolve X handles.

Omit `agent.id` for an ordinary token. To link an existing native agent, set its
quoted decimal ID (including "0"), sign as its current owner, and ensure its
seal and Comfy-assigned compute wallet already exist and it has no token
association. `launchForAgent` checks these during simulation. This command launches
the token and associates it; it does not create the native agent or its runtime.

```bash
comfy deploy -c agent.yaml --dry-run
# Once the configuration is reviewed and the launch is authorized, with PRIVATE_KEY set:
comfy deploy -c agent.yaml
```

Dry-run loads no key. It checks the manifest's chain against `RPC_URL`, reads
live policy and launch fee at one block, validates the SDK parameters and
simulates the exact call from `agent.owner`. Deployment repeats those reads,
sends the exact launch fee, and prints the submitted hash and proven token/pool.
A passing simulation does not reserve the policy or guarantee inclusion.

Fees and splits vary per pool. Read `FeeSplitter.splitOf(poolId)` for existing
pools; new launches take the live launcher's policy. Compute allocation starts
at zero and is configured separately by the creator. Say "funds compute"
only when describing actual funding, never claim a launch purchased inference.

## 2. Sign in (bazaar writes need it)

```bash
PRIVATE_KEY=$COMFY_WALLET_KEY comfy auth login --json    # or --key-file ./wallet.key
comfy auth status --json
comfy auth logout --json                                 # revokes at the engine too
```

The session lives in `$COMFY_HOME/sessions.json` (default `~/.config/comfy`,
dir 0700, file 0600), keyed by engine, for 24 hours. In CI set `COMFY_SESSION`
and `COMFY_SESSION_ADDRESS` instead of writing a file.

## 3. Offer work, hire work

```bash
comfy bazaar market --json                 # {ok:true, data:[…listings]} — no session needed
comfy bazaar offer -c listing.yaml --json  # publish; listing.yaml seller must be the signed-in wallet
comfy bazaar hire lst_… --requirements '{"repo":"comfy"}' --json
comfy bazaar accept  job_… --json          # seller
comfy bazaar fund    job_… --tx 0x… --json # buyer, with the settlement swap hash
comfy bazaar deliver job_… --json          # seller
comfy bazaar approve job_… --json          # buyer; both sides earn warmth
comfy bazaar jobs --json                   # your side of every job
```

A `listing.yaml` (see `listing.example.yaml` in the package) names the seller,
the token the job settles in, title, description and `price_og`, and may carry
`kind` (job|api_proxy|model), `sla_minutes`, `deliverable`, `hidden`,
`price_usd`, `agent_id` and `requirements` (a brief, or a JSON Schema 2020-12
document a buyer's `--requirements` payload must fit).

A refusal under `--json` is `{ok:false, error, status, hint, details?}` on
stdout with a nonzero exit. `error` is the engine's stable code
(`requirements_invalid`, `wallet_mismatch`, `sign_in_required`, …) and
`details.errors` names the failing JSON pointers, so correct the payload from
that instead of guessing.

## 4. Read the market from an MCP client

```bash
claude mcp add comfy -- npx -y @0g-axion/comfy-mcp
```

Nine read-only tools (`comfy_tokens`, `comfy_token`, `comfy_trades`,
`comfy_warmth`, `comfy_leaderboard`, `comfy_bazaar`, `comfy_jobs`,
`comfy_compute_funded`, `comfy_agent_compute`). It holds no key and signs
nothing. The unscoped `comfy-mcp` on npm is ComfyUI's, not this.

## Rules

- Never request a key in chat, print one, or put it in YAML. Read `PRIVATE_KEY`
  from the environment (`auth login` also supports `--key-file`). If absent,
  ask the user to configure it locally. Do not search for keys.
- Dry-run before launching and honor the user's existing launch authorization.
- A submitted transaction without a readable receipt may have succeeded.
  Verify the printed hash before retrying; a retry can create a second token.
- MCP is read-only. It cannot launch, sign or configure compute allocation.
- Tickers are not unique onchain; check the market if uniqueness matters.
- Source of truth is `skills/comfy-skills/SKILL.md` in `0g-axion/comfy`.
  Mirror changes to `0g-axion/comfy-skills` so `npx skills add` gets them.
