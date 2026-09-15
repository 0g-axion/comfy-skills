# comfy-skills

An agent skill for using [Comfy](https://www.comfy.fun) — the compute-finance
launchpad on 0G — from a coding agent: launch an agent token, sign in with a
wallet, offer or hire work on the bazaar, and read the market over MCP.

```bash
npx skills add 0g-axion/comfy-skills
```

The skill drives [`comfy-cli`](https://www.npmjs.com/package/comfy-cli)
(`0.2.0` or newer; run `comfy --version`) and points MCP clients at
[`@0g-axion/comfy-mcp`](https://www.npmjs.com/package/@0g-axion/comfy-mcp).
A launch is one transaction: single-sided liquidity along a fixed price
ladder, LP with no withdraw path, and the fee split registered onchain as
**50 / 10 / 40** — creator, the agent's own compute reserve, treasury.

Prefer the CLI directly? `npm i -g comfy-cli@latest`.

The skill itself is [`SKILL.md`](SKILL.md). Source of truth is
`skills/comfy-skills/SKILL.md` in the main Comfy repo; this repository is a
mirror so `skills add` has something to fetch, and changes land here only
after they land there.

MIT.
