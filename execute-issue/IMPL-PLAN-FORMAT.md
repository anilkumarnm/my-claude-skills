# Implementation Plan Format

How to parse implementation plans created by `/to-issues`.

## File Location

```
docs/impl-plan/{feature-slug}/0001-implementation-plan.md
```

## Structure

### Header Section

```markdown
# {Feature Name} — Implementation Plan

**Parent PRD:** [link to PRD]
**ADR:** [link to ADR if exists]
**Created:** {date}
```

### Summary Table

The Summary Table is the source of truth for issue tracking:

```markdown
## Summary Table

| # | Jira | Title | Type | Blocked by | User Stories | PR-able Independently? |
|---|------|-------|------|------------|--------------|------------------------|
| 1 | [CIPE-1591](url) | Add failure_category column | AFK | None | 5, 18-21 | Yes |
| 2 | [CIPE-1592](url) | Add TriggeredAt to TriggerOutput | AFK | None | 16, 17 | Yes |
| 5 | [CIPE-1596](url) | Update MM client | AFK | CIPE-1594 | 5, 18 | Yes (with #4) |
```

**Columns:**

| Column | Description | How to Parse |
|--------|-------------|--------------|
| # | Issue sequence number | Integer, used for local reference |
| Jira | Link to Jira issue | Extract ID from `[CIPE-XXXX](url)` pattern |
| Title | Short description | Plain text |
| Type | AFK or HITL | AFK = agent can do it, HITL = human required |
| Blocked by | Dependencies | "None" or comma-separated CIPE-IDs or issue #s |
| User Stories | Which PRD stories | Informational only |
| PR-able Independently? | Can be merged alone | Yes/No, informational |

### Parallel Work Streams (Optional)

```markdown
## Parallel Work Streams

| Stream | Focus | Slices | Can Start Immediately |
|--------|-------|--------|----------------------|
| A | Database + API | 1, 6 | Yes |
| B | Data structs | 2, 3, 4, 5 | Yes |
```

Informational — helps understand groupings but not used for dependency resolution.

### Issue Details

Each issue has a detailed section:

```markdown
### Issue {#}: {Title}

**Type:** AFK
**Blocked by:** None — can start immediately
  OR
**Blocked by:** Issue 4

#### What to build

{Detailed description of what to implement}

#### Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3
```

## Parsing Rules

### Extract Jira ID

```regex
\[CIPE-(\d+)\]\(([^)]+)\)
```

- Group 1: Issue number (e.g., `1591`)
- Group 2: Full URL

### Parse "Blocked by" Column

| Value | Meaning |
|-------|---------|
| `None` | No dependencies, can start immediately |
| `CIPE-1594` | Blocked by single issue |
| `CIPE-1592, CIPE-1593` | Blocked by multiple issues |
| `Issue 4` or `#4` | Blocked by issue number from this plan |

Normalize all references to Jira IDs for consistency.

### Determine if Issue is Workable

An issue is workable when:

1. Type is `AFK` (not `HITL`)
2. All "Blocked by" issues are resolved (MR merged to develop)
3. Issue is not already in progress (check Jira status)

### Find "What to build" Section

Look for the pattern:
```markdown
### Issue {#}: {Title}
...
#### What to build

{content until next #### or ### heading}
```

### Find Acceptance Criteria

Look for the pattern:
```markdown
#### Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
```

Collect all `- [ ]` lines until the next section.

## Example Parsing

Given this table row:
```
| 5 | [CIPE-1596](https://jira-dc.../CIPE-1596) | Update MM client | AFK | CIPE-1594 | 5, 18 | Yes |
```

Extract:
```json
{
  "number": 5,
  "jira_id": "CIPE-1596",
  "jira_url": "https://jira-dc.../CIPE-1596",
  "title": "Update MM client",
  "type": "AFK",
  "blocked_by": ["CIPE-1594"],
  "user_stories": ["5", "18"],
  "pr_able": true
}
```

## Tracking Progress

After an MR is created, the skill can optionally update the Summary Table:

```markdown
| # | Jira | Title | Type | Blocked by | MR |
|---|------|-------|------|------------|-----|
| 1 | [CIPE-1591](url) | Add failure_category | AFK | None | [!456](mr_url) |
```

This allows the skill to track which issues have completed MRs for dependency resolution.
