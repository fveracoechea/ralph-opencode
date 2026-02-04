# Ralph Agent Loop for Opencode

Ralph is an autonomous agent loop that runs AI via Opencode repeatedly until all PRD items are complete. Each iteration is a fresh instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

## Requirements

- [Opencode](https://opencode.ai/)
- [jq](https://jqlang.org/)
- Git repository with an `AGENTS.md` file

## Installation

Copy the skills and ralph folder to your project:

```sh
cp -r .opencode/skills/* /path/to/your/project/.opencode/skills/
cp -r ralph /path/to/your/project/
```

## Usage

### 1. Generate a PRD

```
Load the prd skill and create a PRD for [your feature description]
```

Answer the clarifying questions. Output saves to `tasks/prd-[feature-name].md`.

### 2. Convert to Ralph format

```
Load the ralph skill and convert tasks/prd-[feature-name].md to prd.json
```

### 3. Run Ralph

```sh
./ralph/ralph.sh                                     # Default: kimi-k2.5-free, 10 iterations
./ralph/ralph.sh --model opencode/kimi-k2.5          # Custom model
./ralph/ralph.sh --model github-copilot/claude-opus-4.5 20  # Custom model + iterations
```

## How It Works

Each iteration Ralph will:

1. Create/checkout the feature branch from `prd.json`
2. Pick the highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks (typecheck, lint, test)
5. Commit if checks pass
6. Mark story as `passes: true` in `prd.json`
7. Append learnings to `ralph/progress.txt`
8. Repeat until complete or max iterations reached

## Useful CLI Commands

```sh
# Interactive TUI
opencode                              # Start interactive session
opencode -m opencode/claude-sonnet-4  # Start with specific model
opencode -c                           # Continue last session

# Run a single prompt (non-interactive)
opencode run "your prompt here"
opencode run -m opencode/kimi-k2.5 "your prompt"
opencode run -f file.ts "explain this file"

# Models and stats
opencode models                       # List all available models
opencode models github-copilot        # List models for a provider
opencode stats                        # Show token usage and costs

# Session management
opencode session                      # List sessions
opencode export <sessionID>           # Export session as JSON
opencode import <file>                # Import session

# GitHub integration
opencode pr <number>                  # Checkout PR and start opencode
```

## Project Structure

```
your-project/
├── AGENTS.md              # Required: agent instructions for your codebase
├── ralph/
│   ├── ralph.sh           # Main loop script
│   ├── prompt.md          # Per-iteration instructions
│   ├── progress.txt       # Append-only progress log
│   └── prd.json           # Current PRD
└── .opencode/skills/      # Skill definitions (git, prd, ralph, playwriter)
```
