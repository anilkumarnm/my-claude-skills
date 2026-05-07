# Merge Request Format

Standard format for GitLab merge requests in this repository.

## Title

```
{type}(CIPE-{id}): {short description}
```

**Types:** `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`

**Examples:**
- `feat(CIPE-1591): add failure_category column to orchestration_stages`
- `fix(CIPE-1602): set failure_category correctly in workflow`
- `test(CIPE-1607): add unit tests for MetadataValidationActivity`

## Description

```markdown
## Summary

{What this MR implements — one paragraph}

## JIRA



## Backward compatibility

{Breaking changes or "No breaking changes"}

## Schema Migrations

- [ ] Yes
- [ ] No because ______
- [ ] NA

## Temporal Workflows

- [ ] Yes
- [ ] No because ______
- [ ] NA

## Unit tests

- [ ] Added
- [ ] Not added because ______

## Regression tests

- [ ] Added
- [ ] Not added because ______

## Test Plan

{How to verify the changes}

```

## Labels (Choose ONE)

| Label | When |
|-------|------|
| `changelog::added` | New feature or capability |
| `changelog::changed` | Changes to existing functionality |
| `changelog::fixed` | Bug fixes |
| `changelog::removed` | Removed features or capabilities |
| `changelog::security` | Security fixes |
| `changelog::skip` | No changelog entry needed (refactor, tests, docs) |

## Target Branch

Always target `develop` (never `main` directly).

## Reference

See `.claude/skills/execute-issue/MR-FORMAT.md` for full details and examples.
