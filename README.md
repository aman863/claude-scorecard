# Claude Code Scorecard

A Claude Code plugin that analyzes your session transcripts across **8 effectiveness dimensions** and generates a beautiful interactive HTML scorecard.

## What it measures

| Dimension | What it tracks |
|---|---|
| 🎯 Prompt Clarity | How specific and structured your prompts are |
| ⚡ Iteration Speed | How quickly you catch and correct mistakes |
| 🏗️ Architecture | How well you guide system design decisions |
| 🔍 Debug Strategy | How effectively you isolate and communicate bugs |
| 🧠 Human Judgment | How critically you evaluate Claude's suggestions |
| 📐 Scope Control | How focused and atomic your tasks are |
| 🧩 Context Management | How proactively you manage Claude's context window |
| 🛠️ Tool Leverage | How fully you use Claude Code's feature set |

## Installation

### Step 1 — Add the marketplace

```shell
/plugin marketplace add amanjain/claude-scorecard
```

### Step 2 — Install the plugin

```shell
/plugin install claude-scorecard@claude-scorecard
```

### Step 3 — Run the scorecard

```shell
/claude-scorecard:analyze
```

Claude will scan your `~/.claude/projects/` session files, score your usage across all 8 dimensions, and write an interactive HTML scorecard to `~/claude-scorecard.html`.

## Output

- **Radar chart** — visual snapshot of your strengths and gaps
- **Score bars** — per-dimension scores with context anchors
- **Highest-leverage recommendation** — the single change with the most impact
- **Scoring guide** — expandable accordion with indicators and per-dimension recommendations
- **Copy scorecard** — ASCII art version you can paste anywhere

## Local development / testing

```bash
# Clone the repo
git clone https://github.com/amanjain/claude-scorecard
cd claude-scorecard

# Test the plugin locally
claude --plugin-dir ./plugins/claude-scorecard
```

Then inside Claude Code:
```shell
/claude-scorecard:analyze
```

## License

MIT
