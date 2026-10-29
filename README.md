# Haris — Claude Code plugin

Prompt-injection and Saudi PII detection for **Arabic, Arabizi and English**, inside Claude
Code. *Haris* (حارس) means "guard" in Arabic.

This repository is the **distribution point for the plugin**: the manifests, three commands,
and one hook. The detector itself is the `haris-guard` package on PyPI — this repo contains
no model and no detection logic.

```bash
pip install "haris-guard[mcp]>=0.2.1"
```

```
/plugin marketplace add ZKSolution/haris-plugin
```

```
/plugin install haris@zklogic
```

Then `/haris:doctor`. It scans a real attack and a real benign sentence rather than
reporting that the files exist.

**The version floor matters.** The MCP server first reached PyPI in 0.2.1. An unpinned
install can resolve 0.1.0, which has no `haris.mcp` and no `mcp` extra — the plugin then
loads and every tool call fails. Pinned, pip says "no matching distribution", which is a
sentence you can act on.

## What you get

| | |
|---|---|
| `/haris:scan <text or file>` | Verdict, categories and character offsets |
| `/haris:scan-untrusted [diff\|glob\|path]` | Looks for instructions **hidden inside data** — code comments, JSON values, logs, tickets. The attack that matters for an agent working in a repo |
| `/haris:doctor` | Verifies the install, reports the layers in use, states the limits |
| MCP tool `scan_text` | Claude can call it itself, on anything it is about to trust |
| MCP tool `redact_text` | Returns text with identifiers replaced by labels |
| Hook on `WebFetch` / `WebSearch` | Scans what came back; on a hit, tells Claude to treat it as data, never as instructions |

## Configuration

Optional, read from the environment by the server itself — export them **before** starting
Claude Code. They are deliberately not in `.mcp.json`: a default written there would silently
win over the one you exported.

| Variable | Default | What it does |
|---|---|---|
| `HARIS_ENABLE_ML` | `0` | `1` loads the classifier as well as the rules. It catches paraphrases the rules miss, and it also flags more ordinary text — opt in deliberately |
| `HARIS_MCP_BACKEND` | `local` | `local` scans in this process. `http` calls a shared service instead, so a team runs one copy |
| `HARIS_SERVICE_URL` | `http://127.0.0.1:8000` | Where that service is, when the backend is `http` |

## What it does not do

- **It flags; it never blocks.** Nothing is intercepted or rewritten. A finding is
  information for you and for Claude.
- **The hook runs after the fetch**, because that is when a `PostToolUse` hook runs. It
  cannot stop a page being read; it makes sure what the page tried is stated plainly instead
  of sitting unremarked in the context window.
- **A clean verdict is not proof.** The rules know the phrasings they have signatures for,
  and the model layer is off by default. Read anything untrusted yourself as well.
- **Nothing leaves your machine** with the default `local` backend: no telemetry, no network
  calls, and matched values are never returned across a boundary — only labels such as
  `[IQAMA]`.

## Known limits, stated plainly

The model layer (`HARIS_ENABLE_ML=1`) has **not** passed its release gate: it flags 1 of 39
rows on a pinned benign Arabic set where the bar is zero, and roughly **27% of ordinary
Arabic developer prompts** — that domain is missing from the training data and is the main
item in the next revision. The rules layer is unaffected and is what runs by default.

Full evaluation, including what failed:
<https://huggingface.co/ZKLogic/haris-arabic-injection-detector>

## Troubleshooting

**`/haris:doctor` says the MCP tools are unavailable**, or the server reports
`CONNECTION_CLOSED` — almost always the wrong Python. Claude Code uses the `python` on your
PATH, which may not be your project venv:

```bash
python -c "import haris, mcp, sys; print(sys.executable, haris.__version__)"
```

- `ModuleNotFoundError: mcp` → `python -m pip install --user "haris-guard[mcp]>=0.2.1"`
  against **that** interpreter.
- On Windows, `python3` may be the Microsoft Store stub, which exits without running
  anything. Use `python`.

Restart Claude Code after any fix; the MCP server only starts at launch.

## Links

- Project and a live test page — <https://haris.zklogic.com>
- Package — <https://pypi.org/project/haris-guard/>
- VS Code extension — <https://marketplace.visualstudio.com/items?itemName=ZKLogic.haris-guard>
- Model — <https://huggingface.co/ZKLogic/haris-arabic-injection-detector>

The engine repository is private while the project is pre-launch. Issues and questions about
the plugin are welcome here.

MIT licensed.
