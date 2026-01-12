# Speckit Custom Commands

Additional commands for [GitHub Spec-Kit](https://github.com/github/spec-kit) that enable fully autonomous workflows with reduced hallucination.

## Prerequisites

1. Install [GitHub Spec-Kit](https://github.com/github/spec-kit)
2. Install [Claude Code](https://github.com/anthropics/claude-code)

## What This Adds

Copy the `.claude/commands/` files into your spec-kit project to get:

| Command | What it does |
|---------|--------------|
| `/speckit.test` | Auto-run tests, lint, quality checks |
| `/speckit.clarify-auto` | Auto-accept best recommendations |
| `/speckit.analyze-auto` | Auto-fix consistency issues |
| `/speckit.checklist-auto` | Auto-generate requirement checklists |
| `/speckit.automate` | Run entire workflow autonomously |

## Why

These commands reduce manual intervention and hallucination by:
- Using structured defaults instead of guessing
- Auto-validating against existing artifacts
- Following deterministic decision trees
