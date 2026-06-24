# `fm` — Apple Foundation Models CLI reference

`fm` is Apple's on-device Foundation Models command-line tool, shipped with macOS on Apple Silicon (verified on macOS 27 "Golden Gate", at `/usr/bin/fm`). The companion [SKILL.md](../SKILL.md) covers *when* to reach for it; this file is the full *how*.

## Models
| Model | Description | Notes |
|---|---|---|
| `system` | On-device Apple Foundation Model | **Default.** Free, private, offline, zero cloud tokens. |
| `pcc` | Apple Foundation Model on Private Cloud Compute | Often unavailable in a given context; `fm available` reports status. |

## Commands
```
fm respond        Generate a response to a prompt
fm chat           Interactive chat session   ← DO NOT use in automation (it hangs)
fm token-count    Count tokens in a prompt or instructions
fm schema         Generate a JSON generation schema
fm serve          Start a Chat Completions API server
fm available      Check model availability
fm quota-usage    Check model quota usage (PCC only)
```
Run `fm <command> --help` for details on any command.

## `fm respond`
```bash
fm respond [options] '<prompt>'
echo '<prompt>' | fm respond
```
| Flag | Meaning |
|---|---|
| `-m, --model <system\|pcc>` | Model to use (default `system`) |
| `-i, --instructions <text>` | System-style instructions. **Use this to control format/terseness.** |
| `--schema <file>` | Constrain output to a JSON schema (see `fm schema`) |
| `--text <text>` | Extra text segment to include in the prompt |
| `--image <path>` | Include an image in the prompt (vision) |
| `--load-transcript <file>` | Seed from a saved transcript |
| `--save-transcript <name>` | Save transcript after responding |
| `--[no-]stream` | Stream output (default on); `--no-stream` for single-shot capture |
| `-g, --greedy` | Deterministic sampling (repeatable output) |
| `-v, --verbose` | Verbose output |
| `--use-case <general\|content-tagging>` | System-model use case |
| `--guardrails <level>` | `default` or `permissive-content-transformations` |

### Structured (JSON) output
```bash
fm schema object --name Person --string name --int age > person.json
fm respond --schema person.json 'Make up a person'
# → {"age": 34, "name": "Alex Johnson"}
```

### Vision
```bash
fm respond --image photo.jpg --text 'What is in this image?'
```

### Multi-turn (transcripts)
```bash
fm respond --save-transcript chat.json 'My name is Sam.'
fm respond --load-transcript chat.json 'What is my name?'
```

## `fm token-count`
```bash
fm token-count 'text'          # "Token count: N" in a terminal
fm token-count -q 'text'       # bare integer (use when parsing)
echo 'text' | fm token-count
fm token-count -i 'instructions' 'prompt'
```
On-device system model only.

## `fm schema object`
Generates a JSON Schema for use with `fm respond --schema`.
```bash
fm schema object --name Person --string name --int age
fm schema object --name Dog --string breed --boolean friendly
```
Run `fm schema object --help` for the full property-declaration syntax.

## `fm serve` (OpenAI-compatible local API)
```bash
fm serve --port 1976
fm serve --host 0.0.0.0 --port 1976
fm serve --socket /tmp/fm.sock      # Unix socket (recommended for local bindings)
```
Endpoints: `POST /v1/chat/completions` (streaming & non-streaming), `GET /v1/models`, `GET /health`. Use this if you want a persistent local endpoint instead of spawning `fm respond` per call.

## `fm available` / `fm quota-usage`
```bash
fm available      # prints "System model available"; also prints a PCC line (an error if PCC is off)
fm quota-usage    # quota applies to PCC only
```
Gate automation on the string `System model available` — not the exit code, since the PCC line can read like an error even when the on-device model is ready.

## Measured capability (this machine: M3 Max, macOS 27, `system` model)
Quick empirical calibration. Treat as guidance and re-check on your own hardware/OS.

| Task | Result |
|---|---|
| Common-knowledge facts (capitals, etc.) | ✅ reliable |
| Summarize short text | ✅ good |
| Sentiment / topic classification | ✅ good |
| Tagging | ✅ good |
| Rephrase / tone change | ✅ good |
| Entity / email extraction | ✅ good |
| Small, self-contained code (as a draft) | ✅ often correct — use as a **draft you verify** (test / review / refine); not for debugging or large/cross-file changes |
| Multi-step reasoning | ❌ unreliable (missed the classic "bat & ball") |
| Arithmetic | ❌ wrong (`4823 × 7919` → far off) |
| Obscure / long-tail facts | ❌ confidently **fabricates** |

**Behavior notes:**
- Without `-i` instructions the model tends to be chatty; pass instructions to control format.
- It occasionally prepends stray meta-text — clean the output before using it.
- Latency ≈ 2–3 s for short prompts (first call may include model warm-up).
