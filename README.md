# Autopilot — Claude Code Skill

A structured **Plan → Execute** workflow skill for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) that enforces disciplined development: plan first, validate, then execute autonomously.

## What it does

Autopilot forces Claude Code into a two-phase workflow for any non-trivial task:

1. **Plan** — Claude analyzes your codebase (read-only), produces a detailed step-by-step plan, and waits for your explicit approval.
2. **Execute** — Once approved, Claude implements the entire plan autonomously — no unnecessary comments, no intermediate confirmations, no hand-holding.

### Features

| Feature | Description |
|---|---|
| **Detailed planning** | Full analysis before any file is touched |
| **Mandatory validation** | Nothing executes without your explicit "go" |
| **Silent execution** | Zero chatter, zero intermediate confirmations |
| **Git safety net** | Automatic checkpoint commit before execution |
| **Step checkpoints** | One commit per major step for granular rollback |
| **Auto rollback** | Full restoration on critical failure |
| **Parallel subagents** | Independent steps run simultaneously |
| **Lessons learned** | Errors are logged in `.claude/lessons.md` and consulted in future plans |

## Installation

### Global (all projects)

```bash
mkdir -p ~/.claude/skills/autopilot
curl -fsSL https://raw.githubusercontent.com/aiskool/autopilot/main/SKILL.md \
  -o ~/.claude/skills/autopilot/SKILL.md
```

### Per project

```bash
cd /path/to/your/project
mkdir -p .claude/skills/autopilot
curl -fsSL https://raw.githubusercontent.com/aiskool/autopilot/main/SKILL.md \
  -o .claude/skills/autopilot/SKILL.md
```

### Manual

1. Download `SKILL.md` from this repository
2. Place it at `~/.claude/skills/autopilot/SKILL.md` (global) or `.claude/skills/autopilot/SKILL.md` (project)

## Usage

### Slash command

```
/autopilot Add JWT authentication with refresh tokens
```

### Natural language (auto-trigger)

```
Plan and implement a Redis caching layer for user sessions
```

### Recommended flow

1. `Shift+Tab` → Plan Mode
2. `/autopilot [your task]`
3. Review the plan → type `go`
4. `Shift+Tab` → Auto-Accept Mode
5. Claude executes the full plan
6. Final report + lessons learned

## Git commands after execution

```bash
# View all autopilot checkpoints
git log --oneline --grep="autopilot"

# Roll back to a specific step
git reset --hard <checkpoint-hash>

# Undo the entire operation
git reset --hard <restore-point-hash>
```

## Optional: reinforce in CLAUDE.md

Add this to your project or global `CLAUDE.md`:

```markdown
### Development workflow

- Use the autopilot skill for any non-trivial task (3+ steps)
- Always plan before executing
- After plan validation: execute fully without interruption or unnecessary comments
- Never ask for intermediate approval during execution of a validated plan
- Consult .claude/lessons.md before planning
```

## Requirements

- Claude Code v2.1+
- Works with Opus and Sonnet models
- Git recommended (for safety net and checkpoints) but not required

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Security

See [SECURITY.md](SECURITY.md).

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE).
