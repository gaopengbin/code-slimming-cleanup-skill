# Code Slimming and Cleanup Skill

A reusable Claude Code skill for safely making code smaller, simpler, and cleaner without turning cleanup into an uncontrolled refactor.

## What it helps with

- Reduce code size without changing behavior
- Remove proven dead code
- Clean up unused imports, variables, helpers, and branches
- Simplify unnecessary abstractions and indirection
- Consolidate duplicated logic when behavior is truly identical
- Avoid risky drive-by refactors

## Install as a Claude Code plugin

From Claude Code, add this repository as a plugin marketplace or install it according to your Claude Code plugin workflow.

The skill lives at:

```text
skills/code-slimming-cleanup/SKILL.md
```

## Core idea

Good cleanup should make reviewers confident that less code now does the same job.

This skill is intentionally conservative:

- Delete only what is in scope.
- Prove code is dead before removing it.
- Prefer deletion over abstraction.
- Preserve behavior unless the user explicitly asks for behavior changes.
- Report what was removed and how it was verified.

## Repository structure

```text
.
├── README.md
├── README.zh.md
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── code-slimming-cleanup/
        └── SKILL.md
```

## License

MIT
