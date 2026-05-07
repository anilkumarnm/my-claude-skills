---
name: execute-issue
description: Pick an AFK issue from the implementation plan, create isolated worktree, run agent to implement, review commits, create GitLab MR with standard format. Use after /to-issues has created the impl-plan.
---

# Execute Issue

Execute a single issue from an implementation plan using an isolated git worktree. The agent implements the issue, you review the commits, then an MR is created with standard formatting.

## Prerequisites

- Implementation plan exists at `docs/impl-plan/{feature}/0001-implementation-plan.md`
- Jira PAT configured via `JIRA_PAT` environment variable
- GitLab access configured (for MR creation)

## Arguments

- `feature` (required): folder name in `docs/impl-plan/` (e.g., `post-pipeline-metadata-validation`)
- `issue-number` (optional): specific issue # from the Summary Table (e.g., `1` for first issue)

If no issue-number provided, the skill picks the next unblocked AFK issue automatically.

## Process

### 1. Load Implementation Plan

Read `docs/impl-plan/{feature}/0001-implementation-plan.md` and parse:

1. **Summary Table** — extract all issues with their:
   - Issue number (#)
   - Jira ID (e.g., CIPE-1591)
   - Title
   - Type (AFK/HITL)
   - Blocked by (dependencies)
   - PR-able Independently flag

2. **Issue Details** — for the selected issue, extract:
   - "What to build" section
   - Acceptance criteria checklist

See [IMPL-PLAN-FORMAT.md](./IMPL-PLAN-FORMAT.md) for parsing details.

### 2. Select Issue

**If issue-number provided:**
- Use that specific issue from the table
- Verify it's type AFK (warn if HITL)
- Check blockers are resolved

**If no issue-number:**
- Scan table top-to-bottom
- Find first AFK issue where:
  - All "Blocked by" issues have merged MRs (check git branches or impl-plan annotations)
  - Issue is not already in progress

Present the selected issue and ask for confirmation before proceeding.

### 3. Verify Jira Status

Query Jira to confirm the issue is workable:

```
GET /rest/api/2/issue/CIPE-{id}
Authorization: Bearer {JIRA_PAT}
```

Check:
- Issue exists
- Status allows work (not "Done", not "In Progress" by someone else)
- Labels include `ready-for-agent` (or equivalent from triage-labels.md)

If status doesn't allow work, stop and report why.

### 4. Create Worktree

Create an isolated working directory **always based on `develop` branch**:

```bash
# REQUIRED: Fetch and verify develop branch is up to date
git fetch origin develop

# Verify develop exists
git rev-parse --verify origin/develop

# Create worktree with feature branch FROM develop
git worktree add .worktrees/CIPE-{id} -b feature/CIPE-{id}-{slug} origin/develop
```

**IMPORTANT:** Feature branches MUST always be cut from `origin/develop`:
- Never from `main` (production branch)
- Never from current HEAD (may be stale or wrong branch)
- Never from another feature branch (creates dependency chains)

Branch naming follows [BRANCH-FORMAT.md](./BRANCH-FORMAT.md):
- `feature/CIPE-{id}-{kebab-case-title}`
- Example: `feature/CIPE-1591-add-failure-category-column`

Report the worktree path, branch name, and base commit to the user:
```
Worktree: .worktrees/CIPE-1591
Branch:   feature/CIPE-1591-add-failure-category-column
Based on: origin/develop (commit abc1234)
```

### 5. Build Agent Context

Construct a focused prompt for the agent using:

1. **From impl-plan:**
   - "What to build" section (verbatim)
   - Acceptance criteria (as checklist)

2. **From CLAUDE.md:**
   - Relevant patterns (Go/React based on issue scope)
   - Testing requirements

3. **From blockers (if any):**
   - Brief summary of what dependent issues implemented
   - This helps agent understand the foundation

4. **Determine development approach** based on issue type:

#### Issue Type Detection

Analyze the issue title and "What to build" section to determine the type:

| Pattern | Issue Type | Use TDD? |
|---------|------------|----------|
| "Add column", "migration", "schema" | Schema/Migration | No |
| "Add field to struct", "Add X to Y struct" | Data Structure | No |
| "Create activity", "Implement logic", "validation" | Business Logic | **Yes** |
| "Workflow", "integration", "call X after Y" | Orchestration | **Yes** |
| "Unit tests", "test for", "component test" | Test-Only | **Yes** |
| "UI component", "badge", "display" | UI Component | Optional |
| "Update client", "send X to API" | API Integration | **Yes** |

#### TDD Instructions (when applicable)

For logic/behavior issues, include TDD guidance from `/tdd` skill principles:

```markdown
## Development Approach: TDD (Red-Green-Refactor)

This issue involves behavior/logic. Use test-driven development:

1. **RED**: Write ONE failing test for first acceptance criterion
2. **GREEN**: Write minimal code to make it pass
3. **REPEAT**: Next criterion → next test → next implementation
4. **REFACTOR**: Clean up after all tests pass

Rules:
- One test at a time (vertical slices, not horizontal)
- Test behavior through public interfaces, not implementation details
- Tests should survive internal refactors
- Use Ginkgo/Gomega for Go, React Testing Library for UI
```

#### Non-TDD Instructions (for data structures/migrations)

```markdown
## Development Approach: Direct Implementation

This issue is a data structure/schema change. No TDD needed.

1. Implement the change as described
2. Ensure existing tests still pass
3. Add struct comments per CLAUDE.md conventions
```

**Context template:**

```markdown
# Task: CIPE-{id} — {title}

## What to Build
{what_to_build section from impl-plan}

## Acceptance Criteria
{acceptance_criteria as checkboxes}

## Technical Context
- This is a {Go backend / React frontend / both} change
- Follow patterns in CLAUDE.md

## Development Approach
{TDD instructions OR Direct implementation based on issue type detection}

## Dependencies Already Implemented
{brief summary of blocker issues if any, else "None - this is independent"}

## Instructions
1. {If TDD: Follow red-green-refactor cycle}
   {If non-TDD: Implement the change directly}
2. Run relevant tests (`go test ./...` or `npm run lint`)
3. Commit with message: `feat(CIPE-{id}): {short description}`
4. Do NOT push — leave commits local for review
```

### 6. Execute Agent

Change to the worktree directory and run Claude:

```bash
cd .worktrees/CIPE-{id}

# Run agent with the constructed context
claude --print "{context_from_step_5}"
```

**Important:** The agent works in the isolated worktree. Any changes are contained there until explicitly pushed.

Monitor the agent's progress. If it asks questions, the user can interact normally.

### 7. Review Gate (HUMAN REQUIRED)

After the agent completes, present a review summary:

**Show:**
```bash
# Files changed
git diff --stat origin/develop

# Full diff (or summary if large)
git diff origin/develop

# Commits made
git log --oneline origin/develop..HEAD

# Test results (if agent ran them)
```

**Ask the user:**

> **Review the agent's work:**
> 
> 1. **Approve** — Push branch and create MR
> 2. **Request Changes** — Give feedback, agent will iterate
> 3. **Abort** — Discard worktree, no MR created

If "Request Changes": relay feedback to agent and repeat step 6-7.
If "Abort": clean up with `git worktree remove .worktrees/CIPE-{id}`.

### 8. Push Branch

After approval:

```bash
cd .worktrees/CIPE-{id}
git push -u origin feature/CIPE-{id}-{slug}
```

Report the push result to the user.

### 9. Create GitLab MR

Create merge request using GitLab API (via MCP tool or REST):

**Title:** (see [MR-FORMAT.md](./MR-FORMAT.md))
```
feat(CIPE-{id}): {title from impl-plan}
```

**Description:**
```markdown
## Summary

{What to build section from impl-plan}

## JIRA

[CIPE-{id}](https://jira-dc.paloaltonetworks.com/browse/CIPE-{id})

## Backward compatibility

{Describe any breaking changes, or "No breaking changes"}

## Schema Migrations

- [ ] Yes
- [ ] No because ______
- [x] NA

## Temporal Workflows

- [ ] Yes
- [ ] No because ______
- [x] NA

## Unit tests

- [x] Added
- [ ] Not added because ______

## Regression tests

- [ ] Added
- [x] Not added because ______

## Test Plan

{How to verify — from acceptance criteria}

- [ ] `go test ./...` passes
- [ ] `npm run lint` passes (if UI changes)
- [ ] Manual verification as needed

---
Co-Authored-By: Claude <noreply@anthropic.com>
```

**Labels:** (choose ONE based on change type)

| Change Type | Label |
|-------------|-------|
| New feature | `changelog::added` |
| Enhancement | `changelog::changed` |
| Bug fix | `changelog::fixed` |
| Removal | `changelog::removed` |
| Security fix | `changelog::security` |
| Refactor/tests/docs | `changelog::skip` |

**Target branch:** `develop`

Report the MR URL to the user.

### 10. Update Jira

Add comment to the Jira issue:

```
MR created: {MR_URL}

Implemented by Claude Code agent.
```

If Jira workflow allows, transition issue to "In Review" status.

### 11. Cleanup (Optional)

Ask user if they want to keep or remove the worktree:

- **Keep:** Leave `.worktrees/CIPE-{id}` for potential follow-up work
- **Remove:** `git worktree remove .worktrees/CIPE-{id}`

## Example Usage

### Example 1: Schema Change (No TDD)

```
User: /execute-issue post-pipeline-metadata-validation 1

Agent: Issue #1 (CIPE-1591): Add failure_category column to orchestration_stages
       Type: Schema/Migration — Direct implementation (no TDD)
       
       Shall I proceed? [Yes/No/Pick different]

User: yes

Agent: Creating worktree...
       Branch: feature/CIPE-1591-add-failure-category-column
       Path: .worktrees/CIPE-1591
       
       Executing implementation...
       [agent adds migration file and updates Bun model]
       
       Implementation complete. Review:
       - 2 files changed: migrate/sql/00XX.up.sql, models/orchestration_stages.go
       - 1 commit: "feat(CIPE-1591): add failure_category column"
       - Tests: PASS (existing tests unaffected)
       
       Approve, Request Changes, or Abort?

User: approve

Agent: Pushed branch. Creating MR with label: changelog::added
       MR: https://code.pan.run/.../merge_requests/456
       Jira updated with MR link.
```

### Example 2: Business Logic (With TDD)

```
User: /execute-issue post-pipeline-metadata-validation 8

Agent: Issue #8 (CIPE-1598): Implement validation logic
       Type: Business Logic — Using TDD (red-green-refactor)
       Blocked by: CIPE-1595 (skeleton) ✓ merged
       
       Shall I proceed? [Yes/No/Pick different]

User: yes

Agent: Creating worktree...
       Branch: feature/CIPE-1598-implement-validation-logic
       
       Executing with TDD approach...
       
       [RED]  Writing test: "returns success when all keys present and fresh"
       [GREEN] Implementing metadata query call...
       [RED]  Writing test: "returns failure with missing keys list"
       [GREEN] Adding key existence check...
       [RED]  Writing test: "treats stale keys as missing"
       [GREEN] Adding freshness validation...
       [REFACTOR] Extracting helper function...
       
       Implementation complete. Review:
       - 3 files changed
       - 4 commits (one per TDD cycle + refactor)
       - Tests: 5 new tests, all PASS
       
       Approve, Request Changes, or Abort?

User: approve

Agent: Pushed branch. Creating MR with label: changelog::added
       MR: https://code.pan.run/.../merge_requests/461
```

### Example 3: Auto-select Next Unblocked

```
User: /execute-issue post-pipeline-metadata-validation

Agent: Found implementation plan with 19 issues.
       
       Blocked issues: CIPE-1596 (waiting on 1594), CIPE-1601 (waiting on 1592, 1593, 1600)
       
       Next unblocked AFK issues:
         #1 CIPE-1591 - Add failure_category column (Schema)
         #2 CIPE-1592 - Add TriggeredAt to TriggerOutput (Data Structure)
         #7 CIPE-1595 - Create MetadataValidationActivity skeleton (Logic)
       
       Recommend: #1 (CIPE-1591) - first in sequence, unblocks #6
       
       Proceed with #1? [Yes/No/Pick different]
```

## Error Handling

| Error | Action |
|-------|--------|
| Impl-plan not found | Stop, suggest running `/to-issues` first |
| All issues blocked | Report which issues are blocked and by what |
| Jira unreachable | Stop, check JIRA_PAT and network |
| Tests fail | Present to user, ask if they want to fix or abort |
| Push fails | Report error, likely branch conflict |
| MR creation fails | Report error, provide manual command |
