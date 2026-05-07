# Branch Naming Format

Standard format for feature branches created by `/execute-issue`.

## Format

```
feature/CIPE-{issue_id}-{kebab-case-slug}
```

## Components

| Component | Description | Example |
|-----------|-------------|---------|
| `feature/` | Branch type prefix | `feature/` |
| `CIPE-{id}` | Jira issue ID | `CIPE-1591` |
| `{slug}` | Kebab-case title (shortened) | `add-failure-category-column` |

## Slug Rules

1. **Convert to lowercase**
2. **Replace spaces with hyphens**
3. **Remove special characters** (keep only `a-z`, `0-9`, `-`)
4. **Truncate to ~50 chars** (keep full words)
5. **Remove trailing hyphens**

### Examples

| Issue Title | Branch Name |
|-------------|-------------|
| Add `failure_category` column to orchestration_stages | `feature/CIPE-1591-add-failure-category-column` |
| Create MetadataValidationActivity skeleton | `feature/CIPE-1595-create-metadata-validation-activity` |
| Add failureCategory to UI TypeScript types | `feature/CIPE-1603-add-failure-category-ui-types` |
| Unit tests for MetadataValidationActivity | `feature/CIPE-1607-unit-tests-metadata-validation` |

## Worktree Location

Worktrees are created at:

```
.worktrees/CIPE-{id}/
```

This keeps all agent work isolated from the main working directory.

### Directory Structure

```
mission-control/
├── .worktrees/                    # Git worktrees (gitignored)
│   ├── CIPE-1591/                 # Worktree for issue 1591
│   │   ├── orchestration/
│   │   ├── helm/
│   │   └── ...
│   └── CIPE-1595/                 # Worktree for issue 1595
│       └── ...
├── orchestration/                 # Main working directory
└── ...
```

## Git Commands

### Create Worktree with Branch

```bash
# Fetch latest develop
git fetch origin develop

# Create worktree with new branch based on origin/develop
git worktree add .worktrees/CIPE-{id} -b feature/CIPE-{id}-{slug} origin/develop
```

### List Worktrees

```bash
git worktree list
```

### Remove Worktree

```bash
# After MR is merged or work is abandoned
git worktree remove .worktrees/CIPE-{id}

# Force remove if there are uncommitted changes
git worktree remove --force .worktrees/CIPE-{id}
```

### Prune Stale Worktrees

```bash
# Clean up worktrees whose directories no longer exist
git worktree prune
```

## Branch Lifecycle

```
1. Create:     git worktree add .worktrees/CIPE-{id} -b feature/CIPE-{id}-{slug}
2. Work:       cd .worktrees/CIPE-{id} && [agent implements]
3. Push:       git push -u origin feature/CIPE-{id}-{slug}
4. MR:         Create merge request targeting develop
5. Review:     Human reviews and approves
6. Merge:      MR merged to develop
7. Cleanup:    git worktree remove .worktrees/CIPE-{id}
               git branch -d feature/CIPE-{id}-{slug}
```

## Parallel Work

Multiple issues can be worked on simultaneously using separate worktrees:

```bash
# Issue 1 in one worktree
git worktree add .worktrees/CIPE-1591 -b feature/CIPE-1591-add-failure-category

# Issue 2 in another worktree (no blockers)
git worktree add .worktrees/CIPE-1592 -b feature/CIPE-1592-add-triggered-at

# Each worktree is independent
cd .worktrees/CIPE-1591  # Work on issue 1
cd .worktrees/CIPE-1592  # Work on issue 2
```

This enables parallel agent execution for non-blocking issues.
