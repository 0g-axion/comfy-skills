# comfy-skills

An agent skill for launching a token on [Comfy](https://www.comfy.fun) — the
compute-finance launchpad on 0G.

```bash
npx skills add 0g-axion/comfy-skills
```

The skill drives [`comfy-cli`](https://www.npmjs.com/package/comfy-cli), so a
coding agent can validate an `agent.yaml`, dry-run it, and deploy through the
launchpad: single-sided liquidity along a fixed price ladder, LP with no
withdraw path, and the fee split registered onchain as **50 / 10 / 40** —
creator, the agent's own compute reserve, treasury.

Prefer the CLI directly? `npm i -g comfy-cli`.

The skill itself is [`SKILL.md`](SKILL.md). Source of truth for the launchpad
lives in the main Comfy repo; this repository exists so `skills add` has
something to fetch.

MIT.
