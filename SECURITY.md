# Security Policy

## Scope

This repository contains a Claude Code skill file (SKILL.md) — a set of markdown instructions that guide Claude Code's behavior. It does not contain executable code, binaries, or dependencies.

## Supported Versions

| Version | Supported |
|---|---|
| main branch (latest) | ✅ |
| Older commits | ❌ |

## Security Considerations

### What this skill does

- Instructs Claude Code to create git commits (checkpoint and rollback)
- Instructs Claude Code to read, create, modify, and delete files in your project
- Instructs Claude Code to execute bash commands (git operations only)
- Instructs Claude Code to write to `.claude/lessons.md`

### What this skill does NOT do

- It does not collect, transmit, or store any data outside your local project
- It does not make network requests
- It does not install packages or dependencies
- It does not access environment variables, secrets, or credentials

### User responsibility

- **Review the SKILL.md before installing.** It is a plain text file — read it.
- **Git safety net commits are local.** They are not pushed automatically. You control what gets pushed.
- The `--dangerously-skip-permissions` flag mentioned in the skill is a Claude Code feature. Using it is your decision and your risk.
- The skill may instruct Claude Code to use `--no-verify` on git commits to bypass pre-commit hooks during checkpoints. This is intentional to avoid interference with the autopilot workflow.

## Reporting a Vulnerability

If you discover a security issue with this skill:

1. **Do NOT open a public issue.**
2. Email: `karkhedim@gmail.com`
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
4. You will receive a response within 72 hours.

## Disclosure Policy

- Vulnerabilities will be patched and disclosed within 30 days of confirmation.
- Credit will be given to reporters unless they request anonymity.
