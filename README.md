# Speckit Custom Commands

Custom commands extending [GitHub Spec-Kit](https://github.com/github/spec-kit) with automated workflows.

## Prerequisites

**You must install GitHub Spec-Kit first:**

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
specify init <your-project> --ai claude
```

See the official [GitHub Spec-Kit repository](https://github.com/github/spec-kit) for full installation instructions.

## Coding Tool Compatibility

I use **Claude Code** because it works best for me, but you can use any coding tool that GitHub Spec-Kit supports:

- Claude Code (`--ai claude`)
- GitHub Copilot (`--ai copilot`)
- Cursor (`--ai cursor-agent`)
- Windsurf (`--ai windsurf`)
- Amp (`--ai amp`)

## Custom Commands Included

| Command | Description |
|---------|-------------|
| `/speckit.test` | Run tests, linting, and quality checks |
| `/speckit.clarify-auto` | Auto-accept clarification recommendations |
| `/speckit.analyze-auto` | Auto-fix consistency issues |
| `/speckit.checklist-auto` | Auto-detect domains & generate checklists |
| `/speckit.automate` | Full automated workflow |

## Installation

1. Initialize spec-kit in your project first
2. Copy the `.claude/commands/speckit.*.md` files to your project's `.claude/commands/` directory

## Usage

Run the full automated workflow:
```
/speckit.automate <feature description>
```

Or run individual commands as needed.

## License

MIT
