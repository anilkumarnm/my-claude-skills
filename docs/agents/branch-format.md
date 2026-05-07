# Branch Naming Format

Standard format for feature branches in this repository.

## Format

```
feature/CIPE-{issue_id}-{kebab-case-slug}
```

## Examples

| Issue | Branch |
|-------|--------|
| CIPE-1591: Add failure_category column | `feature/CIPE-1591-add-failure-category-column` |
| CIPE-1595: Create MetadataValidationActivity | `feature/CIPE-1595-create-metadata-validation-activity` |

## Worktree Location

Agent-executed issues use isolated worktrees at:

```
.worktrees/CIPE-{id}/
```

## Commands

```bash
# Create worktree with branch
git worktree add .worktrees/CIPE-{id} -b feature/CIPE-{id}-{slug} origin/develop

# Remove worktree after merge
git worktree remove .worktrees/CIPE-{id}
```

## Reference

See `.claude/skills/execute-issue/BRANCH-FORMAT.md` for full details.
