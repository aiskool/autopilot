# Contributing

Contributions are welcome. This project is intentionally small — one skill file — so the bar for changes is quality over quantity.

## How to contribute

1. **Fork** this repository
2. **Create a branch** from `main`: `git checkout -b feature/your-change`
3. **Make your changes** to `SKILL.md`
4. **Test** the skill in Claude Code to verify it works as expected
5. **Open a Pull Request** with a clear description of what changed and why

## Guidelines

### What makes a good contribution

- Bug fixes (e.g., the rollback command has incorrect syntax)
- Improvements to plan format clarity
- Better error handling instructions
- Documentation improvements
- Translations of README (as separate files: `README.fr.md`, `README.es.md`, etc.)

### What will likely be declined

- Changes that add complexity without clear benefit
- Features that require external dependencies or tools beyond Claude Code and git
- Changes that break backward compatibility with Claude Code v2.1+
- Additions that significantly increase SKILL.md size beyond 500 lines (per Claude Code skill best practices)

## Testing your changes

Before submitting:

1. Install the modified SKILL.md in `~/.claude/skills/autopilot/`
2. Open Claude Code
3. Run `/autopilot` with a non-trivial task
4. Verify:
   - Plan is generated correctly
   - Validation gate works (Claude waits for "go")
   - Execution runs without unnecessary commentary
   - Git checkpoints are created (if in a git repo)
   - Final report is produced
   - Lessons are written to `.claude/lessons.md` (if errors occurred)

## Code of Conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## Questions

Open an issue with the `question` label.
