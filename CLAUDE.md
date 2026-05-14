# Claude Working Agents

A scaffold for Claude Code subagents that automate the full software engineering lifecycle: implementation, testing, and review — plus a standalone improvement scanner.

## Agents Overview

| Agent | Role | Pipeline Position | Can Write Source | Can Write Tests |
|-------|------|-------------------|------------------|-----------------|
| `engineer` | Implements features and fixes | Step 1 of 3 | Yes | Yes |
| `tester` | Validates with tests, fills coverage | Step 2 of 3 | No | Yes |
| `reviewer` | Code review + explicit verdict | Step 3 of 3 | No | No |
| `improver` | Proactive improvement scanner | Standalone | No | No |

---

## Sequential Workflow Pipeline

```
[Task] → engineer → tester → reviewer → [Done or Rework]
```

Each agent emits a structured handoff block that the next agent reads as its starting context. The pipeline is explicitly sequential — no agent starts until the previous one has emitted its handoff.

---

### Step 1: Engineer

**Trigger:** Any task requiring code changes.

**How to invoke:**
```
Implement <feature or fix description>. Use the engineer agent.
```

**What happens:**
1. Reads the codebase, identifies existing patterns and conventions.
2. Implements the change with minimal scope.
3. Runs linter and full test suite before committing.
4. Commits using conventional commit format (`type(scope): summary`).
5. Emits `ENGINEER HANDOFF` block.

**Handoff contract:** A clean, committed change on the current branch with a handoff block naming the commit hash, files changed, and test focus areas.

---

### Step 2: Tester

**Trigger:** After engineer emits its handoff.

**How to invoke:**
```
Run the tester agent on the engineer's commit.
```

**What happens:**
1. Reads engineer handoff, verifies the commit exists.
2. Runs the full test suite (auto-detects: npm/pytest/go test/cargo test/make test).
3. Compares against pre-commit baseline to distinguish regressions from pre-existing failures.
4. Writes new tests for uncovered code paths in changed files.
5. Commits new tests separately (`test(scope): add coverage for X`).
6. Emits `TESTER HANDOFF` with a PROCEED or HOLD recommendation.

**If HOLD:** Pipeline stops. The engineer must address flagged regressions before the tester re-runs.

---

### Step 3: Reviewer

**Trigger:** After tester emits a PROCEED handoff.

**How to invoke:**
```
Run the reviewer agent on the current branch.
```

**What happens:**
1. Reads both handoff blocks.
2. Reads the full diff from branch point to HEAD — every changed file in full.
3. Evaluates: correctness, security, design, readability, and test quality.
4. Every finding includes a `file:line` reference and concrete problem statement.
5. Emits an explicit `VERDICT: APPROVED` or `VERDICT: REQUEST CHANGES`.

**If APPROVED:** Branch is ready for human merge.

**If REQUEST CHANGES:** All blocking issues must be resolved. Give the blocking issues list to the engineer as a new task and re-run the full pipeline.

---

## Standalone: Improver

The improver runs independently of the pipeline at any time.

**How to invoke:**
```
Use the improver agent to scan [the whole codebase | src/auth/ | all Python files].
```

**What happens:**
1. Scans the specified scope, read-only — no file changes.
2. Reports findings across five priority tiers:
   - Tier 1: Security
   - Tier 2: Bugs & Correctness
   - Tier 3: Performance
   - Tier 4: Maintainability & Technical Debt
   - Tier 5: Style & Convention
3. Ends with the top 3-5 recommended immediate actions.

**Acting on findings:** Feed individual items to the engineer:
```
Based on the improver report, fix the issue at src/db/query.py:34. Use the engineer agent.
```

---

## Running the Full Pipeline

To run all three pipeline steps on a single task:
```
Implement <task description>. Run engineer → tester → reviewer in sequence.
```

---

## Handoff Block Reference

**ENGINEER HANDOFF** (read by tester):
```
ENGINEER HANDOFF
================
Commit: <full commit hash>
Summary: <1-2 sentence description>
Files changed: <list with brief per-file notes>
Known limitations: <text or "none">
Test focus areas: <list of behaviors to verify>
```

**TESTER HANDOFF** (read by reviewer):
```
TESTER HANDOFF
==============
Engineer commit tested: <hash>
Test suite result: PASS | FAIL | PASS WITH REGRESSIONS
New tests added: <list of test names, or "none">
Coverage gaps remaining: <list with rationale, or "none">
Regressions found: <list with file:line, or "none">
Recommendation to reviewer: PROCEED | HOLD (reason)
```

**REVIEWER VERDICT** (read by human or looped back to engineer):
```
VERDICT: APPROVED
Commit range reviewed: <base>..HEAD
Critical issues: none
Notes for author: <optional>
```
```
VERDICT: REQUEST CHANGES
Commit range reviewed: <base>..HEAD
Blocking issues:
  1. [CATEGORY] file:line — description
Non-blocking observations:
  - file:line — description
Required action: Engineer must address all blocking issues and re-run the full pipeline.
```

---

## Workflow Conventions

- **Branching:** Feature work on a branch off `main`. Agents commit to the current branch; humans merge.
- **Commit style:** Conventional Commits — `type(scope): summary`.
- **Test runner:** Agents auto-detect (npm test, pytest, go test, cargo test, make test).
- **Linting:** Agents run the configured linter before committing. Must pass before any commit.
- **No remote pushes:** Agents never push to remote. Human operators push and merge.

---

## Adding New Agents

Agent definitions live in `.claude/agents/<name>.md`. Each file has YAML frontmatter followed by a system prompt:

```yaml
---
name: <name>
description: <when Claude should invoke this agent — be specific>
model: claude-sonnet-4-6
tools:
  - Bash
  - Read
  # add/remove tools based on agent's write permissions
---
<system prompt>
```

The `description` field drives routing — make it specific and action-oriented. Update this CLAUDE.md to document the new agent.

---

## Troubleshooting

**Agent not being invoked automatically:**
Check that the `description` in its frontmatter clearly matches the request context. Vague descriptions cause routing failures.

**Pipeline stalled at tester with HOLD:**
Read `Regressions found` in the tester handoff. Give those specific findings to the engineer as a new task, then re-run the full pipeline from the start.

**Reviewer issued REQUEST CHANGES:**
Copy the `Blocking issues` list and give it to the engineer as a new task. After the engineer commits fixes, re-run from the tester step (tester → reviewer).

**Improver report is too broad:**
Narrow the scope: `scan src/auth/ only` or `scan for Tier 1 security issues only`.
