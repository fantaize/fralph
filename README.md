# Fralph

![Fralph](fralph.png)

Ralph is an autonomous AI agent loop that runs [OpenCode](https://opencode.ai) repeatedly until all PRD items are complete. Each iteration is a fresh OpenCode instance with clean context. Memory persists via git history, `progress.txt`, and `PRD.md`.

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/).

## Prerequisites

- [OpenCode](https://opencode.ai) installed
- A git repository for your project

## Setup

### Option 1: Copy to your project

Copy the ralph files into your project:

```bash
# From your project root
mkdir -p scripts/fralph
cp /path/to/fralph/fralph.sh scripts/fralph/
cp /path/to/fralph/prompt.md scripts/fralph/
chmod +x scripts/fralph/fralph.sh
```

### Option 2: Install skills globally

Copy the skills to your Amp config for use across all projects:

```bash
cp -r skill/prd ~/.config/opencode/skill/
```

## Workflow

### 1. Create a PRD

Use the PRD skill to generate a detailed requirements document:

```
Load the prd skill and create a PRD.md
```

Answer the clarifying questions. The skill saves output to `PRD.md`.

### 3. Run Fralph

```bash
./scripts/fralph/fralph.sh [MAX] [SLEEP] [MODEL]
# Defaults:
# 1: 10
# 2: -2
# 3: opencode/glm-4.7-free
```

Default is 10 iterations.

Fralph will:
1. Create a feature branch (from PRD `branchName`)
2. Pick the highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks (typecheck, tests)
5. Edit until checks pass
6. Update `PRD.md`
7. Append learnings to `progress.txt`
8. Commit and push to repo (Specificially in order of 5-7)
8. Repeat until all stories pass or max iterations reached


## Critical Concepts

### Each Iteration = Fresh Context

Each iteration spawns a **new OpenCode instance** with clean context. The only memory between iterations is:
- `progress.txt` (learnings and context)
- `PRD.md` (which stories are done)

### Small Tasks

Each PRD item should be small enough to complete in one context window. If a task is too big, the LLM runs out of context before finishing and produces poor code.

Right-sized stories:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

Too big (split these):
- "Build the entire dashboard"
- "Add authentication"
- "Refactor the API"

### Feedback Loops

Ralph only works if there are feedback loops:
- Typecheck catches type errors
- Tests verify behavior
- CI must stay green (broken code compounds across iterations)

### Stop Condition

When all stories have `[x]`, Fralph outputs `<promise>COMPLETE</promise>` and the loop exits.

## Debugging

Check current state:

```bash
# See which stories are done
cat PRD.md

# See learnings from previous iterations
cat progress.txt

# Check git history
git log --oneline -10
```

## References

- [Geoffrey Huntley's Ralph article](https://ghuntley.com/ralph/)
- [Amp Version of Ralph](https://github.com/snarktank/ralph)
