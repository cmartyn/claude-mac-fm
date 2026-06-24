---
name: local-model
description: Use when a task is light and low-stakes enough to offload to the on-device model and save cloud tokens — quick common-knowledge facts, simple text transforms (summarize, rephrase, classify, tag, extract), a fast sanity-check, or a first draft of small, self-contained code you then verify yourself (test, review, or refine). Wraps Apple's on-device Foundation Models CLI (`fm`) on Apple Silicon. Not for debugging, multi-step reasoning, math, obscure facts, or long documents; always verify its output before relying on it.
allowed-tools: [Bash, Read]
---

# local-model — offload light work to the on-device `fm` model

## Overview
`fm` is Apple's **Foundation Models CLI** (Apple Silicon). Its default `system` model runs **on-device**: free, private, offline, and it **costs zero cloud tokens**. Use it to hand off light, low-stakes work so cloud tokens are spent only where they matter. It is a *small* model — quick and capable for simple language tasks and short code drafts, but not a substitute for your own reasoning or judgment.

## Precondition (check, then proceed)
Only offload if the model is actually present and ready:
```bash
command -v fm >/dev/null && fm available 2>&1 | grep -q 'System model available'
```
If that fails, just do the task yourself — don't mention `fm`. Checking once per session is enough; no need to re-run it before every call.

## When to USE it — reach for it automatically, no need to ask
- **Quick common-knowledge facts & definitions** — capitals, well-known terms, basic explanations.
- **Simple text transforms** — summarize, rephrase, change tone, classify, tag, extract entities/emails. This is its sweet spot (measured reliable).
- **Cheap second-opinion sanity-check** — cross-check a short answer or classification you already produced, as a low-cost "belt and suspenders."
- **First drafts of small, self-contained code** — a well-specified function, a config snippet, or boilerplate. The draft is a starting point, **never** the final word — you verify it before it lands (see [Using it for code](#using-it-for-code--draft-then-verify)).

## When NOT to use it — do it yourself
- **Debugging & reasoning-heavy work** — diagnosing a failure, multi-step logic, or arithmetic. Measured: it got "bat & ball" and a 4-digit multiplication confidently **wrong**. Do these yourself.
- **Code that needs codebase context** — large, cross-file, or architectural changes, or a refactor that hinges on understanding surrounding code. Keep these Claude-led; `fm` is for small, local drafts.
- **Obscure / long-tail facts** — it **fabricates** confidently (it invented a detailed plot for a film that doesn't exist).
- **Long documents** — small context window.
- **Shipping anything unverified** — an `fm` result is never a final answer on its own.

## Trust rule — low-stakes trust
Trust `fm` for trivial/cosmetic output (a tag, a rephrase, a summary you'll read anyway). **Validate — or just do it yourself — anything that affects code, files, or decisions.** When an `fm` answer *disagrees* with yours, that's a signal to slow down and check, not an answer to adopt.

The USE list is about what the model is *capable* of; this rule is about *consequences*. Classification, extraction, or a code draft is in its wheelhouse — but if the result feeds a real decision or lands in the codebase, you verify it first.

## Using it for code — draft, then verify
**Good draft candidates:** a single small function, boilerplate/scaffolding, or a config snippet — roughly *one self-contained function (~40 lines), no exotic dependencies*. Anything bigger, or that spans the existing codebase, is your job, not `fm`'s.

**Poor candidates even when they look small:** correctness minefields — email/URL/date regexes, parsing, concurrency, or anything security-sensitive. Write those yourself. And **if verifying the draft would cost more than just writing it, skip `fm`.**

A draft is **never** committed as-is. Before it lands, close the loop — and for anything runnable, *running it wins*:
- **Test it (preferred for runnable code)** — actually execute it on normal inputs and an edge case. Reading alone isn't enough for code you can run.
- **Analyze it** — read critically for correctness, edge cases, security, and fit with surrounding code. Use when you genuinely can't execute it.
- **Improve it** — treat the draft as a starting point and refine it with your own context and expertise.

You own the final result.

## Cheatsheet
```bash
fm respond 'PROMPT'                          # one-shot, on-device system model (default)
echo 'TEXT' | fm respond                     # pipe stdin as the prompt
fm respond -i 'Answer in one word' 'PROMPT'  # -i = instructions; control format/terseness
fm respond -i 'Output only code' 'Write a Python slugify(s)'  # code DRAFT — then test/review it
fm respond --greedy 'PROMPT'                 # deterministic (good for repeatable checks)
fm token-count -q 'TEXT'                     # bare integer token count
```
- **Use `-i` to control output.** Without instructions the model is chatty (e.g. *"Yes, the capital of France is Paris"*); `-i 'Answer in one word'` → `Paris`. Add `-i 'Output only the answer, no preamble'` (or `'Output only code'`) to suppress stray meta-text at the source — cleaner than stripping it afterward.
- **Multi-line / quote-heavy input:** pipe it via stdin (`echo`, `printf`, or a heredoc) or `--text`, rather than cramming it into the quoted prompt argument — avoids shell-escaping pain.
- **Batching:** to classify or transform several short, independent items, one call with a numbered list is fine and cheaper than one call each; use per-item calls only when you need cleanly separated outputs.

## Gotchas
- **Never run `fm chat`** — it's an interactive REPL and will hang. Always use `fm respond`.
- Always pass a prompt argument or pipe stdin; bare `fm respond` waits on stdin (add `</dev/null` if you ever call it without input).
- Default model is on-device `system`. `pcc` (Private Cloud Compute) is often unavailable — don't assume it.
- `fm available` prints a PCC error line even when the system model is fine — match the string `System model available`, don't rely on exit code alone.

## More
Full subcommand/flag reference — structured `--schema` JSON, `--image` vision, multi-turn transcripts, `fm serve` local API, and a measured capability table — in [references/fm-reference.md](references/fm-reference.md).
