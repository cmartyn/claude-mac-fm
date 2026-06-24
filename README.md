# claude-mac-fm

A [Claude Code](https://claude.com/claude-code) skill that teaches Claude to offload **light, low-stakes work to an on-device model** — via Apple's [Foundation Models](https://developer.apple.com/documentation/foundationmodels) (`fm`) — so it spends cloud tokens only where they matter, with a fast, private, local second opinion always on hand.

> Belt-and-suspenders for your token budget: Claude keeps using its cloud models for everything that counts, and quietly hands the trivial stuff to the model already running on your Mac.

## What the skill does

It's a set of instructions that shape *when* and *how* Claude leans on the local model:

- **Offloads light work automatically** — quick common-knowledge facts, simple text transforms (summarize, rephrase, classify, tag, extract), and cheap sanity-checks of an answer it already has.
- **Drafts small code, then verifies it** — for a single small, self-contained function or some boilerplate, Claude can take a local first draft, but it never ships it unverified: it runs it, reviews it, or refines it first. Correctness-sensitive code (email/URL/date parsing, security) it writes itself.
- **Keeps the hard parts in Claude's hands** — debugging, multi-step reasoning, and math stay with Claude's own reasoning; a local draft is a help, not the authority.
- **Verifies anything load-bearing** — local output is trusted for trivial/cosmetic results and double-checked whenever it affects code, files, or decisions. (Apple's stack can transparently step up from the on-device model to the more capable Private Cloud Compute tier, and you can't always tell which answered — so verification is the rule for anything that matters.)

## Requirements

An **Apple Silicon Mac** with Apple's on-device Foundation Models available (the `fm` command). See Apple's [Foundation Models documentation](https://developer.apple.com/documentation/foundationmodels). The skill checks availability before using it, and quietly does the task itself if `fm` isn't present — so it's safe to install anywhere.

## Install

```
/plugin marketplace add cmartyn/claude-mac-fm
/plugin install local-model@claude-mac-fm
```

Then `/reload-plugins` (or restart Claude Code). That's it — Claude will start reaching for the local model on suitable tasks.

## How it works

The skill is just a `SKILL.md` (plus a reference doc) that tells Claude *when* offloading is worthwhile, *how much* to trust the result, and *when* to verify. There's no daemon and no configuration — Claude calls the local model on demand and owns the final result.

## License

MIT — see [LICENSE](LICENSE).

## Disclaimer

Not affiliated with or endorsed by Apple or Anthropic. The on-device model is Apple's; this project just teaches Claude Code to use it well, and to verify anything that matters.
