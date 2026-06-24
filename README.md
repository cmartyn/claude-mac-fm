# claude-mac-fm

A [Claude Code](https://claude.com/claude-code) skill that lets Claude offload **light, low-stakes work to Apple's on-device model** — through the `fm` (Apple Foundation Models) CLI — to **conserve cloud tokens** and provide a fast, private, local second opinion.

> Belt-and-suspenders for your token budget: Claude keeps using its cloud models for everything that matters, and quietly hands the trivial stuff to the model already running on your Mac.

## What it does

Once installed, Claude will *proactively* use the on-device model for things like:

- **Quick common-knowledge facts & definitions**
- **Simple text transforms** — summarize, rephrase, change tone, classify, tag, extract
- **Cheap sanity-checks** of an answer it already has

…and it deliberately **won't** use it for project code, multi-step reasoning, math, obscure facts, long documents, or anything load-bearing — because the on-device model is small. These boundaries were set by **actually measuring** the model, not guessing (see the [capability table](plugins/local-model/skills/local-model/references/fm-reference.md#measured-capability-this-machine-m3-max-macos-27-system-model)).

It treats local output with **low-stakes trust**: fine for trivial/cosmetic results, validated for anything that touches code, files, or decisions.

## Requirements

- An **Apple Silicon Mac** (M-series) with the **`fm` CLI** (Apple Foundation Models). Verified on **macOS 27 "Golden Gate"**, where `fm` ships at `/usr/bin/fm`.
- Check yours:
  ```bash
  command -v fm && fm available
  ```
  You want to see `System model available`.

## Install

### As a Claude Code plugin (recommended)

```
/plugin marketplace add cmartyn/claude-mac-fm
/plugin install local-model@claude-mac-fm
```

Then `/reload-plugins` (or restart Claude Code).

### Manually (no plugin system)

Copy the skill into your personal skills directory:

```bash
git clone https://github.com/cmartyn/claude-mac-fm
cp -r claude-mac-fm/plugins/local-model/skills/local-model ~/.claude/skills/
```

## Usage

You don't have to do anything — Claude reaches for it on its own when a task is light enough. To use `fm` directly:

```bash
fm respond 'What is the capital of Australia?'
fm respond -i 'Answer in one word' 'Capital of Japan?'
echo 'long text…' | fm respond -i 'Summarize in one sentence.'
fm token-count -q 'How many tokens is this?'
```

See the [full reference](plugins/local-model/skills/local-model/references/fm-reference.md) for structured JSON output (`--schema`), vision (`--image`), multi-turn transcripts, and the OpenAI-compatible local server (`fm serve`).

## How it works

The skill is just a `SKILL.md` (plus a reference doc) that teaches Claude *when* offloading to `fm` is worthwhile, *how* to call it correctly, and *how much to trust* the result. There's no daemon and no config — Claude shells out to `fm` on demand.

## License

MIT — see [LICENSE](LICENSE).

## Disclaimer

Not affiliated with or endorsed by Apple or Anthropic. `fm` is Apple's tool; this project just teaches Claude Code to use it well. Model capability varies by hardware and OS version — the skill checks availability before relying on it.
