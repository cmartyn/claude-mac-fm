---
name: use-fm
description: Use when a task is light and low-stakes enough to offload to the on-device model and save cloud tokens — quick common-knowledge facts, simple text transforms (summarize, rephrase, classify, tag, extract), a fast sanity-check, or a first draft of small, self-contained code you then verify yourself (test, review, or refine). Wraps Apple's on-device Foundation Models CLI (`fm`) on Apple Silicon. Keep debugging, multi-step reasoning, and math in your own hands, and verify anything load-bearing before relying on it.
allowed-tools: [Bash, Read]
---

# use-fm — offload light work to the on-device `fm` model

## Overview
`fm` is Apple's **Foundation Models CLI** (Apple Silicon). The default `system` model runs **on-device** — free, private, offline, and **zero cloud tokens**. Apple's stack spans tiers: a fast on-device model and **Private Cloud Compute (PCC)**, a more capable, privacy-preserving cloud tier the system can step up to when a task warrants it. You can't always tell which tier answered, so the rule of thumb is simple: it's great for offloading light work, and you verify anything that matters.

## Precondition (check, then proceed)
Only offload if the model is present and ready:
```bash
command -v fm >/dev/null && fm available 2>&1 | grep -q 'System model available'
```
If that fails, just do the task yourself. Checking once per session is enough.

## When to USE it — reach for it automatically, no need to ask
- **Quick common-knowledge facts & definitions** — capitals, well-known terms, basic explanations.
- **Simple text transforms** — summarize, rephrase, change tone, classify, tag, extract entities/emails. Its sweet spot.
- **Cheap second-opinion sanity-check** — cross-check a short answer or classification you already produced, a low-cost "belt and suspenders."
- **First drafts of small, self-contained code** — a well-specified function, a config snippet, boilerplate. A starting point you verify before it lands (see [Using it for code](#using-it-for-code--draft-then-verify)).

## Keep these in your own hands
Not because the model can't help, but because you have the context and these are easy to get subtly wrong:
- **Multi-step reasoning & math** — verify any number or chain of logic rather than taking it on faith; often it's quicker to reason it through yourself.
- **Debugging** — you hold the codebase context, so lead it yourself (the model can still draft a snippet you fold in).
- **Obscure / long-tail facts** — like any model it can be confidently wrong on niche details, so confirm before relying.
- **Long inputs** — mind the on-device context window; chunk or handle large documents yourself.

## Trust rule
Trust `fm` for trivial/cosmetic output (a tag, a rephrase, a summary you'll read anyway). **Verify anything that affects code, files, or decisions** — partly because you can't always tell which tier answered. When an `fm` result *disagrees* with your own, treat it as a prompt to double-check, not a verdict.

## Using it for code — draft, then verify
**Good draft candidates:** a single small function, boilerplate/scaffolding, or a config snippet — roughly *one self-contained function (~40 lines), no exotic dependencies*. Anything bigger, or that spans the existing codebase, is yours to lead.

**Write these yourself** even when they look small: correctness-sensitive code — email/URL/date regexes, parsing, concurrency, security. And **if verifying the draft would cost more than just writing it, skip `fm`.**

A draft is **never** committed as-is. Before it lands, close the loop — and for anything runnable, *running it wins*:
- **Test it (preferred for runnable code)** — execute it on normal inputs and an edge case; reading alone isn't enough for code you can run.
- **Analyze it** — read critically for correctness, edge cases, security, and fit with surrounding code.
- **Improve it** — treat the draft as a starting point and refine it with your own context and expertise.

You own the final result.

## Cheatsheet
```bash
fm respond 'PROMPT'                          # one-shot, on-device system model (default)
echo 'TEXT' | fm respond                     # pipe stdin as the prompt
fm respond -i 'Answer in one word' 'PROMPT'  # -i = instructions; control format/terseness
fm respond -i 'Output only code' 'Write a Python slugify(s)'  # code DRAFT — then test/review it
fm respond --model pcc 'PROMPT'              # request the more-capable Private Cloud Compute tier (if available)
fm respond --greedy 'PROMPT'                 # deterministic (good for repeatable checks)
fm token-count -q 'TEXT'                     # bare integer token count
```
- **Use `-i` to control output.** Without instructions the model is chatty; `-i 'Answer in one word'` → `Paris`. Add `-i 'Output only the answer, no preamble'` (or `'Output only code'`) to suppress stray meta-text at the source.
- **Multi-line / quote-heavy input:** pipe it via stdin (`echo`, `printf`, or a heredoc) or `--text`, rather than cramming it into the quoted prompt argument.
- **Batching:** to classify or transform several short, independent items, one call with a numbered list is fine and cheaper than one call each.

## Gotchas
- **Never run `fm chat`** — it's an interactive REPL and will hang. Always use `fm respond`.
- Always pass a prompt argument or pipe stdin; bare `fm respond` waits on stdin (add `</dev/null` if you ever call it without input).
- Default model is on-device `system`; `--model pcc` requests the cloud tier when it's available in your context.
- `fm available` prints a PCC status line plus `System model available` — match that string rather than relying on exit code alone.

## More
Full subcommand/flag reference — structured `--schema` JSON, `--image` vision, multi-turn transcripts, `fm serve` local API, and notes on the model tiers — in [references/fm-reference.md](references/fm-reference.md).
