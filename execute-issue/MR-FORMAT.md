# Merge Request Format

Standard format for GitLab MRs created by the `/execute-issue` skill.

## Title Format

```
{type}(CIPE-{id}): {short description}
```

### Type Prefixes

| Type | When to Use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code restructuring without behavior change |
| `test` | Adding or updating tests |
| `docs` | Documentation changes |
| `chore` | Build, config, or tooling changes |
| `perf` | Performance improvements |

### Examples

```
feat(CIPE-1591): add failure_category column to orchestration_stages
feat(CIPE-1595): create MetadataValidationActivity skeleton
fix(CIPE-1602): set failure_category correctly in workflow
test(CIPE-1607): add unit tests for MetadataValidationActivity
refactor(CIPE-1594): add FailureCategory to OrchDbInput
```

### Rules

- Title should be under 72 characters
- Use lowercase after the colon
- Use imperative mood ("add", not "added" or "adding")
- Reference the Jira ID in parentheses
- Description should be specific but concise

## Description Template

```markdown
## Summary

{One paragraph describing what this MR implements. Copy from "What to build" section of impl-plan.}

## JIRA

[CIPE-{id}](https://jira-dc.paloaltonetworks.com/browse/CIPE-{id})

## Backward compatibility

{Describe any backward compatibility considerations, or "No breaking changes" if none}

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

{Describe how to verify the changes work correctly}

- [ ] `go test ./...` passes
- [ ] `npm run lint` passes (if UI changes)
- [ ] Manual verification: {specific check if applicable}

---
Co-Authored-By: Claude <noreply@anthropic.com>
```

## Labels

Apply ONE changelog label based on the type of change:

| Label | When to Use |
|-------|-------------|
| `changelog::added` | New feature or capability |
| `changelog::changed` | Changes to existing functionality |
| `changelog::fixed` | Bug fixes |
| `changelog::removed` | Removed features or capabilities |
| `changelog::security` | Security fixes |
| `changelog::skip` | No changelog entry needed |

### Label Selection Guide

| Issue Type | Changelog Label |
|------------|-----------------|
| New feature implementation | `changelog::added` |
| Enhance existing feature | `changelog::changed` |
| Bug fix | `changelog::fixed` |
| Remove deprecated code | `changelog::removed` |
| Security vulnerability fix | `changelog::security` |
| Internal refactor, tests only, docs | `changelog::skip` |

## Target Branch

- Default: `develop`
- Never target `main` directly (merge train handles that)

## Assignee

- Assign to the user who ran `/execute-issue`
- Or leave unassigned for team to pick up

## Example Complete MR

**Title:**
```
feat(CIPE-1591): add failure_category column to orchestration_stages
```

**Labels:** `changelog::added`

**Description:**
```markdown
## Summary

Add a new nullable `failure_category` column to the `orchestration_stages` table. 
This column will store `PIPELINE_FAILURE` or `METADATA_PUBLISH_FAILURE` to distinguish 
between external pipeline failures and metadata contract violations.

## JIRA

[CIPE-1591](https://jira-dc.paloaltonetworks.com/browse/CIPE-1591)

## Backward compatibility

No breaking changes. New column is nullable, existing rows unaffected.
Follows "fix forward" migration policy.

## Schema Migrations

- [x] Yes
- [ ] No because ______
- [ ] NA

## Temporal Workflows

- [ ] Yes
- [ ] No because ______
- [x] NA

## Unit tests

- [ ] Added
- [x] Not added because migration-only change, tested via integration

## Regression tests

- [ ] Added
- [x] Not added because additive schema change with no behavior modification

## Test Plan

- [x] Migration applies successfully to local postgres via docker-compose
- [x] Query `orchestration_stages` table shows new `failure_category` column
- [x] Existing rows have NULL for failure_category (expected)
- [x] No errors in mission-monitor startup logs

```

## After MR Creation

1. **Post MR URL to Jira** as a comment
2. **Transition Jira** to "In Review" if workflow allows
3. **Update impl-plan** (optional) to track MR link in Summary Table
