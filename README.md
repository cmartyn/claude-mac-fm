# claude-mac-fm

A [Claude Code](https://claude.com/claude-code) skill that lets Claude offload **light, low-stakes work to Apple's on-device model** — through the `fm` (Apple Foundation Models) CLI — to **conserve cloud tokens** and provide a fast, private, local second opinion.

> Belt-and-suspenders for your token budget: Claude keeps using its cloud models for everything that matters, and quietly hands the trivial stuff to the model already running on your Mac.

## What it does

Once installed, Claude will *proactively* use the on-device model for things like:

- **Quick common-knowledge facts & definitions**
- **Simple text transforms** — summarize, rephrase, change tone, classify, tag, extract
- **Cheap sanity-checks** of an answer it already has
- **First drafts of small, self-contained code** — which Claude then tests, reviews, or refines before it lands

…and it deliberately **won't** use it for debugging, multi-step reasoning, math, obscure facts, or long documents — because the on-device model is small. For small, self-contained code it can write a *first draft*, which Claude then **verifies** (tests, reviews, or refines) before anything lands. These boundaries were set by **actually measuring** the model, not guessing (see the [capability table](plugins/local-model/skills/local-model/references/fm-reference.md#measured-capability-this-machine-m3-max-macos-27-system-model)).

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

## Using it for code (draft → verify)

`fm` can write a **first draft** of small, self-contained code — but Claude never ships it unverified. A real run of the loop:

**1. Draft** (on-device, zero cloud tokens):

```bash
fm respond -i 'Output only Python code' \
  'Write slugify(s): lowercase, collapse non-alphanumeric runs to one hyphen, trim hyphens'
```

**2. Verify by running it.** In testing, this step instantly caught that the draft came back wrapped in Markdown code fences — the kind of thing you catch by *executing*, not skimming. Cleaned up, the draft is sound:

```python
import re
def slugify(s):
    s = s.lower()
    s = re.sub(r'[^\w]+', '-', s)
    return s.strip('-')
```

```text
'Hello, World!'  -> 'hello-world'
'already-a-slug' -> 'already-a-slug'
'Ünïcödé_x'      -> 'ünïcödé_x'   # \w keeps unicode + underscores; refine for ASCII-only
```

**3. Refine if needed,** then it lands. Good `fm` draft candidates: a single small function, boilerplate, or a config snippet. Claude writes the rest itself — debugging, large or cross-file refactors, and correctness minefields (email/URL/date regexes, parsing, security) don't go to `fm`.

## How it works

The skill is just a `SKILL.md` (plus a reference doc) that teaches Claude *when* offloading to `fm` is worthwhile, *how* to call it correctly, and *how much to trust* the result. There's no daemon and no config — Claude shells out to `fm` on demand.

## License

MIT — see [LICENSE](LICENSE).

## Disclaimer

Not affiliated with or endorsed by Apple or Anthropic. `fm` is Apple's tool; this project just teaches Claude Code to use it well. Model capability varies by hardware and OS version — the skill checks availability before relying on it.
