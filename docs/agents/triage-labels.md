# Triage Labels

Labels used by the `triage` skill to categorize and route issues.

## Label Mapping

| Role | Label String |
|------|--------------|
| Maintainer needs to evaluate | `needs-triage` |
| Waiting on reporter | `needs-info` |
| Fully specified, agent can pick up | `ready-for-agent` |
| Needs human implementation | `ready-for-human` |
| Will not be actioned | `wontfix` |

## Usage

When creating or updating issues, use these exact label strings. The triage skill will apply these labels based on issue analysis.
