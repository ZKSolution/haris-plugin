# Haris — Claude Code plugin

Prompt-injection and Saudi PII detection for Arabic, Arabizi and English, inside Claude Code.

It adds three things: the Haris **MCP tools**, three **commands**, and one **hook** that
checks content fetched from the web before you act on it.

## Install

Haris must be importable by the `python` on your PATH — the same `python` Claude Code
will use, which is often not your project venv:

```bash
pip install "haris-guard[mcp]>=0.2.1"
```

The version floor is deliberate. The MCP server first reached PyPI in 0.2.1, and an unpinned
install can resolve 0.1.0, which has no `haris.mcp` and no `mcp` extra — the plugin
then loads and every tool call fails. Pinned, pip says "no matching distribution"
instead, which is a sentence you can act on.

Check it with `python -c "import haris, mcp; print(haris.__version__)"`. If that prints
a different interpreter's version than you expect, see **Troubleshooting** below.

Then, in Claude Code:

```
/plugin marketplace add ZKSolution/haris-plugin
```

```
/plugin install haris@zklogic
```

Check it with `/haris:doctor`. It tests a real attack and a real benign sentence rather
than reporting that the files exist.

## What you get

| | |
|---|---|
| `/haris:scan <text or file>` | Scan something and report verdict, categories and offsets |
| `/haris:scan-untrusted [diff\|glob\|path]` | Look for instructions **hidden inside data**: code comments, JSON values, logs, tickets. This is the attack that matters for an agent working in a repo |
| `/haris:doctor` | Verify the install, the layers in use, and state the limits |
| MCP tool `scan_text` | Claude can call it itself, on anything it is about to trust |
| MCP tool `redact_text` | Returns text with identifiers replaced by labels |
| Hook on `WebFetch` / `WebSearch` | Scans what came back; on a hit, tells Claude to treat it as data, never as instructions |
| Hook on **your prompt**, before it is sent | Checks what you typed. Personal data **stops the prompt** so you can redact it; an injection you pasted for analysis is annotated instead |

## Configuration

All optional, and read from the environment by the server itself — set them in your shell
**before** starting Claude Code. They are deliberately not in `.mcp.json`: a default written
there would silently win over the one you exported.

| Variable | Default | What it does |
|---|---|---|
| `HARIS_ENABLE_ML` | `0` | `1` loads the classifier as well as the rules. It catches paraphrases the rules miss, and it also flags more ordinary text — opt in deliberately |
| `HARIS_MCP_BACKEND` | `local` | `local` scans in this process. `http` calls a shared service instead, so a team runs one copy |
| `HARIS_SERVICE_URL` | `http://127.0.0.1:8000` | Where that service is, when the backend is `http` |
| `HARIS_PROMPT_GUARD` | `warn` | The pre-submit check. `warn`: personal data blocks, pasted injection is annotated. `block`: both block. `pii`: only personal data, injection ignored. `off`: disabled |

## What it does not do

- **It flags; it never blocks — with one deliberate exception.** The pre-submit check can
  stop *your own* prompt when it contains personal data, because a prompt cannot be unsent
  and you are the person being protected. Nothing else is intercepted or rewritten, and
  `HARIS_PROMPT_GUARD=off` removes even that.
- **A stopped prompt is not a secret kept.** The block happens before the text leaves your
  machine, which is the point — but the text can still reach your local session transcript
  and prompt history. Claude Code's own documentation says so, and so do we.
- **The hook runs after the fetch**, because that is when a `PostToolUse` hook runs. It
  cannot stop a page being read; it makes sure what the page tried is stated plainly
  instead of sitting unremarked in the context window.
- **A clean verdict is not proof.** The rules know the phrasings they have signatures for,
  and the model layer is off by default. Read anything untrusted yourself as well.
- **Nothing leaves your machine** with the default `local` backend: no telemetry, no
  network calls, and the matched values are never returned across a boundary — only
  labels such as `[IQAMA]`.

Project: <https://haris.zklogic.com> · try it: <https://haris.zklogic.com/try.html>
