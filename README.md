# comfy-skills

An agent skill for using [Comfy](https://www.comfy.fun) — the compute-finance
launchpad on 0G — from a coding agent: launch an agent token, sign in with a
wallet, offer or hire work on the bazaar, and read the market over MCP.

```bash
npx skills add 0g-axion/comfy-skills
```

The skill drives [`comfy-cli`](https://www.npmjs.com/package/comfy-cli)
(`0.3.0` or newer; run `comfy --version`) and points MCP clients at
[`@0g-axion/comfy-mcp`](https://www.npmjs.com/package/@0g-axion/comfy-mcp).
An Infinity launch creates a token and locks its supply in the price ladder.
Launch terms and recipient shares are configurable; fees and splits come from
the selected deployment and the pool's onchain configuration. The optional
native agent ID links an existing agent to its new token market.

Prefer the CLI directly? `npm i -g comfy-cli@latest`.

The skill itself is [`SKILL.md`](SKILL.md). Source of truth is
`skills/comfy-skills/SKILL.md` in the main Comfy repo; this repository is a
mirror so `skills add` has something to fetch, and changes land here only
after they land there.

MIT.
