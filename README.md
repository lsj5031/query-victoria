# VictoriaLogs Query Skill

An [Agent Skill](https://agentskills.io) for querying VictoriaLogs using the HTTP API with LogsQL syntax.

## Installation

### Amp / Claude Code

```bash
# Clone to your skills directory
git clone https://github.com/lsj5031/query-victoria.git ~/.config/agents/skills/query-victoria
```

Or add to your Claude Code settings to auto-install from the repository.

## Usage

Once installed, the agent will automatically load this skill when you ask about VictoriaLogs queries.

Example prompts:
- "Query VictoriaLogs for errors in the last hour"
- "Show me the top 5 error types from VictoriaLogs"
- "Get hourly error counts for the past 3 hours"

## Environment Variables

Set these in your shell before querying:

```bash
export VICTORIALOGS_URL="http://your-victorialogs-host:9428"
export VICTORIALOGS_USERNAME="your-username"
export VICTORIALOGS_PASSWORD="your-password"
```

## License

MIT
