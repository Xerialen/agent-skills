# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, Antigravity, etc.) when working with code in this repository.

## Repository Overview

A collection of skills for Claude.ai and Claude Code for senior software engineers. Skills are packaged instructions and scripts that extend Claude and your coding agents capabilities.

## OpenCode Integration

OpenCode uses a **skill-driven execution model** powered by the `skill` tool and this repository's `/skills` directory.

### Core Rules

- If a task matches a skill, you MUST invoke it
- Skills are located in `skills/<skill-name>/SKILL.md`
- Never implement directly if a skill applies
- Always follow the skill instructions exactly (do not partially apply them)

### Intent ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ Skill Mapping

The agent should automatically map user intent to skills:

- Feature / new functionality ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `spec-driven-development`, then `incremental-implementation`, `test-driven-development`
- Planning / breakdown ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `planning-and-task-breakdown`
- Bug / failure / unexpected behavior ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `debugging-and-error-recovery`
- Code review ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `code-review-and-quality`
- Refactoring / simplification ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `code-simplification`
- API or interface design ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `api-and-interface-design`
- UI work ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `frontend-ui-engineering`

### Lifecycle Mapping (Implicit Commands)

OpenCode does not support slash commands like `/spec` or `/plan`.

Instead, the agent must internally follow this lifecycle:

- DEFINE ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `spec-driven-development`
- PLAN ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `planning-and-task-breakdown`
- BUILD ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `incremental-implementation` + `test-driven-development`
- VERIFY ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `debugging-and-error-recovery`
- REVIEW ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `code-review-and-quality`
- SHIP ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `shipping-and-launch`

### Execution Model

For every request:

1. Determine if any skill applies (even 1% chance)
2. Invoke the appropriate skill using the `skill` tool
3. Follow the skill workflow strictly
4. Only proceed to implementation after required steps (spec, plan, etc.) are complete

### Anti-Rationalization

The following thoughts are incorrect and must be ignored:

- "This is too small for a skill"
- "I can just quickly implement this"
- "IÃƒÂ¢Ã¢â€šÂ¬Ã¢â€žÂ¢ll gather context first"

Correct behavior:

- Always check for and use skills first

This ensures OpenCode behaves similarly to Claude Code with full workflow enforcement.

## Orchestration: Personas, Skills, and Commands

This repo has three composable layers. They have different jobs and should not be confused:

- **Skills** (`skills/<name>/SKILL.md`) ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â workflows with steps and exit criteria. The *how*. Mandatory hops when an intent matches.
- **Personas** (`agents/<role>.md`) ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â roles with a perspective and an output format. The *who*.
- **Slash commands** (`.claude/commands/*.md`) ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â user-facing entry points. The *when*. The orchestration layer.

Composition rule: **the user (or a slash command) is the orchestrator. Personas do not invoke other personas.** A persona may invoke skills.

The only multi-persona orchestration pattern this repo endorses is **parallel fan-out with a merge step** ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â used by `/ship` to run `code-reviewer`, `security-auditor`, and `test-engineer` concurrently and synthesize their reports. Do not build a "router" persona that decides which other persona to call; that's the job of slash commands and intent mapping.

See [agents/README.md](agents/README.md) for the decision matrix and [references/orchestration-patterns.md](references/orchestration-patterns.md) for the full pattern catalog.

**Claude Code interop:** the personas in `agents/` work as Claude Code subagents (auto-discovered from this plugin's `agents/` directory) and as Agent Teams teammates (referenced by name when spawning). Two platform constraints align with our rules: subagents cannot spawn other subagents, and teams cannot nest. Plugin agents silently ignore the `hooks`, `mcpServers`, and `permissionMode` frontmatter fields.

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    scripts/              # Required: executable scripts
      {script-name}.sh    # Bash scripts (preferred)
  {skill-name}.zip        # Required: packaged for distribution
```

### Naming Conventions

- **Skill directory**: `kebab-case` (e.g. `web-quality`)
- **SKILL.md**: Always uppercase, always this exact filename
- **Scripts**: `kebab-case.sh` (e.g., `deploy.sh`, `fetch-logs.sh`)
- **Zip file**: Must match directory name exactly: `{skill-name}.zip`

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence describing what the skill does, followed by one or more "Use when" trigger conditions. Include trigger phrases like "Deploy my app" or "Check logs" when helpful.}
---

# {Skill Title}

{Brief overview of what the skill does and why it matters.}

## How It Works

{Numbered list explaining the skill's workflow}

Equivalent headings like `Workflow`, `Core Process`, or `When to Use` are fine when they communicate the same structure clearly.

## Usage (Optional)

Include this section only if the skill ships runnable helpers under `scripts/`. Markdown-only skills can omit both the section and the directory entirely.

```bash
bash /mnt/skills/user/{skill-name}/scripts/{script}.sh [args]
```

**Arguments:**
- `arg1` - Description (defaults to X)

**Examples:**
{Show 2-3 common usage patterns}

## Output

{Show example output users will see}

## Present Results to User

{Template for how Claude should format results when presenting to users}

## Troubleshooting

{Common issues and solutions, especially network/permissions errors}
```

### Best Practices for Context Efficiency

Skills are loaded on-demand ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â only the skill name and description are loaded at startup. The full `SKILL.md` loads into context only when the agent decides the skill is relevant. To minimize context usage:

- **Keep SKILL.md under 500 lines** ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â put detailed reference material in separate files
- **Write specific descriptions** ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â helps the agent know exactly when to activate the skill
- **Use progressive disclosure** ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â reference supporting files that get read only when needed
- **Prefer scripts over inline code** ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â script execution doesn't consume context (only output does)
- **File references work one level deep** ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â link directly from SKILL.md to supporting files

### Script Requirements

- Use `#!/bin/bash` shebang
- Use `set -e` for fail-fast behavior
- Write status messages to stderr: `echo "Message" >&2`
- Write machine-readable output (JSON) to stdout
- Include a cleanup trap for temp files
- Reference the script path as `/mnt/skills/user/{skill-name}/scripts/{script}.sh`

### Creating the Zip Package

After creating or updating a skill:

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```

### End-User Installation

Document these two installation methods for users:

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
Add the skill to project knowledge or paste SKILL.md contents into the conversation.

If the skill requires network access, instruct users to add required domains at `claude.ai/settings/capabilities`.

<!-- codex-review-gate:start -->
## Review Gate Contract

This repository uses the same gate flow as `Xerialen/komodobots`:

- New or updated PRs are reset to `gate: reviewing`.
- A reviewer reviews only the current head SHA.
- The reviewer applies exactly one terminal label when warranted: `gate: ready` or `gate: blocked`.
- Draft PRs must never receive `gate: ready`.
- A deterministic GitHub Action merges only when the PR is open, non-draft, targets the repository default branch, has `gate: ready`, lacks `gate: blocked`, has a current-head SHA-bound PASS comment, and all non-gate checks including `PR Tests` are green.
- New commits make earlier gate decisions stale and require reassessment.

Role separation:

- Coder implements.
- Reviewer reviews technical merge safety.
- Merge executor only merges after the deterministic gate passes.
- Codex-authored PRs require independent non-Codex review before being treated as independently reviewed.

Whenever Codex posts a GitHub issue, PR, PR review, review comment, issue comment, or merge/gate comment through `@Xerialen`, include this visible line:

`_Posted by Codex via @Xerialen._`

Required gate comment format:

```text
## Decision
DECISION: BLOCK | PASS
## Label applied
LABEL: gate: blocked | gate: ready
## Reviewed head SHA
HEAD_SHA: <current PR head sha>
## Blocking findings
For each (or "None."): Severity / File-area / Problem / Why this blocks merge / Required fix.
## Non-blocking notes
Concrete technical notes only (or "None.").
```

Before applying a gate decision, classify whether the PR is ML-impacting. If it touches data extraction, datasets, training, model behavior, evaluation, metrics, inference, ML documentation, or evidence ledgers, read and apply `machine-learning-reviewer.md`. For non-ML PRs, say explicitly that the PR is not ML-impacting.
<!-- codex-review-gate:end -->
