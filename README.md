# claude-monitor

Audit logging for Claude Code tool usage via hooks. Every tool call is recorded to a JSONL file for inspection, analysis, or compliance.

## What it does

Hooks into five Claude Code lifecycle events and appends a structured JSON record to `~/.claude/audit.jsonl`:

| Event | Logged fields |
|-------|--------------|
| `PreToolUse` | ts, session, tool, id, cwd, input |
| `PostToolUse` | ts, session, tool, id, cwd, input |
| `PostToolUseFailure` | ts, session, tool, id, cwd, input |
| `SessionStart` | ts, session, cwd |
| `SessionEnd` | ts, session |

### Example log entries

```jsonl
{"ts":"2026-04-02T10:00:00Z","event":"session_start","session":"abc123","cwd":"/home/user/project"}
{"ts":"2026-04-02T10:00:01Z","event":"pre","session":"abc123","tool":"Read","id":"tool_01","cwd":"/home/user/project","input":{"file_path":"/home/user/project/main.py"}}
{"ts":"2026-04-02T10:00:01Z","event":"post","session":"abc123","tool":"Read","id":"tool_01","cwd":"/home/user/project","input":{"file_path":"/home/user/project/main.py"}}
{"ts":"2026-04-02T10:00:05Z","event":"session_end","session":"abc123"}
```

## Requirements

- [jq](https://jqlang.github.io/jq/) must be installed and on `PATH`

## Installation

Copy `settings.json` into your Claude Code project directory (alongside `.claude/`) or merge it into an existing `settings.json`:

```sh
cp settings.json /path/to/your/project/settings.json
```

The audit log is written to `~/.claude/audit.jsonl`. The file is created automatically on first use.

## Querying the log

```sh
# All tool calls in the current session
jq 'select(.session == "YOUR_SESSION_ID")' ~/.claude/audit.jsonl

# All errors
jq 'select(.event == "error")' ~/.claude/audit.jsonl

# Count tool calls by tool name
jq -r 'select(.event == "pre") | .tool' ~/.claude/audit.jsonl | sort | uniq -c | sort -rn

# Activity for a specific working directory
jq 'select(.cwd == "/home/user/project")' ~/.claude/audit.jsonl
```
