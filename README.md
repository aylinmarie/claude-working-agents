# claude-working-agents

A portable set of subagents for [Claude Code](https://claude.com/claude-code) and Cursor that I drop into every project. It covers the full engineering lifecycle — implement, test, review — plus standalone tools for scanning a codebase and writing docs.

## What's in here

```
agents/
  engineer.md       # writes code
  tester.md         # runs and writes tests
  reviewer.md       # reviews the diff, gives a verdict
  improver.md       # read-only codebase scanner
  writer.md         # tech specs and docs
.claude/agents      # symlink -> ../agents
.claude/skills/
  pipeline/SKILL.md # runs engineer -> tester -> reviewer in one invocation
.claude-plugin/
  plugin.json       # makes this repo installable as a Claude Code plugin
  marketplace.json  # self-hosted marketplace entry for the plugin above
.cursor/agents      # symlink -> ../agents
```

Both tools read the same five agent files. There's nothing to keep in sync.

## The pipeline

```
engineer → tester → reviewer
```

Each agent hands the next one a structured block (commit hash, files changed, what to focus on) instead of relying on shared conversation context. `engineer` writes source and tests and commits; `tester` runs the suite, fills coverage gaps, and gives a PROCEED/HOLD call; `reviewer` reads the full diff and returns an explicit APPROVED or REQUEST CHANGES verdict with `file:line` findings. No agent pushes to remote — that stays a human action.

`improver` and `writer` sit outside the pipeline. Run `improver` any time to get a tiered report (security, correctness, performance, maintainability, style, accessibility) on any slice of the codebase. Run `writer` any time you need a spec, README, or other doc — it re-fetches an anti-AI-writing style guide on every invocation so the output doesn't read like a language model wrote it.

Full behavior, invocation examples, and the handoff block formats are documented in [AGENTS.md](AGENTS.md).

## Using this in a new project

**Claude Code, install once as a plugin (recommended):** the repo is also a self-hosted plugin marketplace, so there's nothing to copy per-project.

```
/plugin marketplace add aylinmarie/claude-working-agents
/plugin install working-agents@working-agents-marketplace
```

Agents show up namespaced as `/working-agents:engineer`, `/working-agents:tester`, etc., in every project without touching that project's repo. Updates land by re-running the marketplace add + install, not by editing N repos.

**Cursor, or Claude Code without the plugin:** copy the repo into the project root, or add it as a submodule:

```bash
git submodule add https://github.com/aylinmarie/claude-working-agents.git .agents
ln -s .agents/agents .claude/agents
ln -s .agents/agents .cursor/agents
```

Or just copy `agents/`, `AGENTS.md`, and the two symlinks directly — there's no build step and no external dependencies. Claude Code and Cursor both pick up agents from `.claude/agents/*.md` and `.cursor/agents/*.md` automatically.

If the target project already has a `CLAUDE.md`, add `@AGENTS.md` to it so the pipeline conventions get pulled in as project instructions.

## Adding an agent

New agent definitions go in `agents/<name>.md` with YAML frontmatter (`name`, `description`, `model`, `tools`) followed by the system prompt. Because `.claude/agents` and `.cursor/agents` are symlinks to `agents/`, a new file is picked up by both tools with no extra wiring. See "Adding New Agents" in [AGENTS.md](AGENTS.md) for the template.
