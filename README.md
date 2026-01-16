# Speckit Custom Commands

Custom commands for speckit that enable autonomous feature development.

## Commands

| Command | Purpose |
|---------|---------|
| `/speckit.automate <desc>` | Run full workflow: specify → plan → tasks → implement → test |
| `/speckit.test` | Validate code against spec with auto-fix |
| `/speckit.test fix` | Auto-fix issues and re-verify |
| `/speckit.test trace` | Show requirement traceability matrix |

## Installation

Copy `.claude/commands/` to your project:

```bash
cp -r .claude/commands/* your-project/.claude/commands/
```

## Usage

```bash
# Automate entire feature
/speckit.automate Add user authentication

# Or run manually
/speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement → /speckit.test
```
