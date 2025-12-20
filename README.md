# Claude Code Skills Collection

[한국어](./README.ko.md)

A collection of custom skills for Claude Code.

## Installation

```bash
git clone https://github.com/sangre72/claude-skills.git ~/.claude/skills
```

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [gitpush](./gitpush) | `/gitpush` | Auto-analyze changes, commit with Conventional Commits format, merge dev branch, and push |
| [gitpull](./gitpull) | `/gitpull` | Pull dev branch and merge into current branch, then pull current branch |
| [coding-guide](./coding-guide) | `/coding-guide` | Generate coding guidelines for your project and add to CLAUDE.md |
| [gitignore](./gitignore) | `/gitignore` | Generate comprehensive .gitignore file for your project |
| [modular-check](./modular-check) | `/modular-check` | Analyze project modularity and provide architecture guide |
| [refactor](./refactor) | `/refactor` | Check modularity and type guideline compliance, with auto-fix support |

## Skill Details

### /gitpush

Automatically analyzes changes and commits with Conventional Commits format.

- Analyzes changed files and determines appropriate commit type (feat, fix, docs, refactor, etc.)
- Auto-merges dev branch if exists
- Pulls current branch before push

### /gitpull

Synchronizes dev branch with current branch.

- Auto-stash uncommitted changes
- Pull and merge dev branch if exists
- Pull current branch
- Restore stash

### /coding-guide

Generates coding guidelines tailored to your project.

- Language-specific rules (TypeScript/JavaScript, Python, etc.)
- Naming conventions and file naming rules
- Import order and error handling patterns
- Security library rules

### /gitignore

Generates .gitignore file for your project.

- Supports multiple languages: Node.js, Python, Go, Rust, etc.
- Monorepo support (Turborepo/pnpm)
- Includes IDE, environment variables, cache files, etc.

### /modular-check

Analyzes project modularity status.

- Type duplication check
- Circular dependency check
- Layer separation check
- Modularity compliance rate calculation

### /refactor

Checks modularity and type guideline compliance.

- Type consolidation (remove duplicate types)
- Utility consolidation
- Dependency direction check
- Auto-fix support (`--fix`)

## License

MIT
