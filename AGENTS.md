# AGENTS.md - Ralph Agent Loop for Opencode

Ralph is an autonomous agent loop that runs AI via Opencode repeatedly until all PRD items are complete. Each iteration is a fresh instance with clean context. Memory persists via git history, `ralph/progress.txt`, and `prd.json`.

**Primary Language:** Shell (Bash) with JavaScript/Node.js dependencies  
**Dependencies:** Opencode, jq, git

## Build/Test/Lint Commands

This is a shell script-based orchestration system. It's designed to be **deployed into other repositories** that have their own quality tooling.

### Running Ralph

```bash
./ralph/ralph.sh                                    # Default: kimi-k2.5-free, 10 iterations
./ralph/ralph.sh --model opencode/kimi-k2.5         # Custom model
./ralph/ralph.sh --model github-copilot/claude-opus-4.5 20  # Custom model + iterations
```

### Running Tests in Target Projects

When in a target project, use that project's test commands. Ralph expects `typecheck`, `lint`, and `test` commands.

**Running a single test:**
```bash
npm test -- path/to/test.spec.ts    # Jest
npx vitest run path/to/test.spec.ts # Vitest
node --test path/to/test.js         # Node test runner
```

## Code Style Guidelines

### Git Commits (Conventional Commits)

```
<type>[optional scope]: <description>
```

**Types:** `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`, `style`, `build`, `ci`

**Ralph commits:** `feat: [Story ID] - [Story Title]`

**Rules:**
- Imperative mood ("add" not "added")
- No period at end
- Under 50-72 characters
- Focus on "why" in body, not "what"
- NEVER include AI tool attribution

### Shell Script Style

- Use `#!/usr/bin/env bash` shebang
- Set `set -e` for error handling
- Quote all variables: `"$VAR"` not `$VAR`
- Use `$(command)` not backticks

## Project Structure

```
ralph-opencode/
├── AGENTS.md                 # This file
├── opencode.json             # Opencode configuration
├── ralph/
│   ├── ralph.sh              # Main loop script
│   ├── prompt.md             # Per-iteration instructions
│   ├── progress.txt          # Append-only progress log
│   ├── prd.json              # Current PRD in JSON format
│   └── archive/              # Previous run archives
└── .opencode/
    ├── package.json          # Node.js dependencies
    └── skills/               # Skill definitions (git, prd, ralph, playwriter)
```

## Ralph Workflow

1. **Generate PRD:** Use `prd` skill → `tasks/prd-[feature].md`
2. **Convert to JSON:** Use `ralph` skill → `ralph/prd.json`
3. **Run Ralph:** Execute `./ralph/ralph.sh`

### Per-Iteration Behavior

1. Read `prd.json` for user stories
2. Read `ralph/progress.txt` (check Codebase Patterns first)
3. Ensure correct branch from PRD `branchName`
4. Pick highest priority story where `passes: false`
5. Implement that single story
6. Run quality checks (typecheck, lint, test)
7. Commit: `feat: [Story ID] - [Story Title]`
8. Update `prd.json` to set `passes: true`
9. Append progress to `ralph/progress.txt`

### Progress Report Format

APPEND to `ralph/progress.txt`:

```markdown
## [Date/Time] - [Story ID]
- What was implemented
- Files changed
- **Learnings for future iterations:**
  - Patterns discovered
  - Gotchas encountered
---
```

### Codebase Patterns

Add reusable patterns to `## Codebase Patterns` at TOP of `ralph/progress.txt`:

```markdown
## Codebase Patterns
- Use `sql<number>` template for aggregations
- Always use `IF NOT EXISTS` for migrations
```

## Quality Requirements

- ALL commits must pass quality checks
- Do NOT commit broken code
- Keep changes focused and minimal
- Work on ONE story per iteration

## Browser Testing (Frontend Stories)

For UI changes:
1. Load the `playwriter` skill
2. Navigate to the relevant page
3. Verify UI changes work as expected

A frontend story is NOT complete until browser verification passes.

## PRD JSON Format

```json
{
  "project": "[Project Name]",
  "branchName": "ralph/[feature-name-kebab-case]",
  "userStories": [{
    "id": "US-001",
    "title": "[Story title]",
    "description": "As a [user], I want [feature] so that [benefit]",
    "acceptanceCriteria": ["Criterion 1", "Typecheck passes"],
    "priority": 1,
    "passes": false,
    "notes": ""
  }]
}
```

## Stop Condition

After completing a story, check if ALL stories have `passes: true`. If complete:

```
<promise>COMPLETE</promise>
```

## Key Files

| File | Purpose |
|------|---------|
| `ralph/ralph.sh` | Main loop script |
| `ralph/prompt.md` | Per-iteration instructions |
| `ralph/progress.txt` | Append-only progress log |
| `ralph/prd.json` | Current PRD in JSON |
| `.opencode/skills/*/SKILL.md` | Skill definitions |

## Error Handling

- Exit code 0: Successful completion
- Exit code 1: Max iterations reached
- Check `ralph/progress.txt` for status on failure
- Previous runs archived in `ralph/archive/`
