# Issue tracker: Jira

Issues and PRDs for this repo live as Jira issues. Use the Jira REST API with a Personal Access Token (PAT).

## Environment Variables

```bash
export JIRA_PAT="<your-jira-personal-access-token>"
export JIRA_BASE_URL=""
export JIRA_PROJECT_KEY=""
```

## Conventions

- **Create an issue**:
  ```bash
  curl -s -X POST "$JIRA_BASE_URL/rest/api/3/issue" \
    -H "Authorization: Bearer $JIRA_PAT" \
    -H "Content-Type: application/json" \
    -d '{
      "fields": {
        "project": {"key": "'"$JIRA_PROJECT_KEY"'"},
        "summary": "Issue title",
        "description": {"type": "doc", "version": 1, "content": [{"type": "paragraph", "content": [{"type": "text", "text": "Description here"}]}]},
        "issuetype": {"name": "Task"},
        "labels": ["needs-triage"]
      }
    }'
  ```

- **Read an issue**:
  ```bash
  curl -s "$JIRA_BASE_URL/rest/api/3/issue/MCO-123" \
    -H "Authorization: Bearer $JIRA_PAT" | jq .
  ```

- **List issues (JQL)**:
  ```bash
  curl -s "$JIRA_BASE_URL/rest/api/3/search" \
    -H "Authorization: Bearer $JIRA_PAT" \
    -G --data-urlencode "jql=project=$JIRA_PROJECT_KEY AND labels=needs-triage" | jq .
  ```

- **Add a comment**:
  ```bash
  curl -s -X POST "$JIRA_BASE_URL/rest/api/3/issue/MCO-123/comment" \
    -H "Authorization: Bearer $JIRA_PAT" \
    -H "Content-Type: application/json" \
    -d '{"body": {"type": "doc", "version": 1, "content": [{"type": "paragraph", "content": [{"type": "text", "text": "Comment text"}]}]}}'
  ```

- **Update labels**:
  ```bash
  curl -s -X PUT "$JIRA_BASE_URL/rest/api/3/issue/MCO-123" \
    -H "Authorization: Bearer $JIRA_PAT" \
    -H "Content-Type: application/json" \
    -d '{"fields": {"labels": ["ready-for-agent"]}}'
  ```

- **Transition issue** (e.g., close):
  ```bash
  # First get available transitions
  curl -s "$JIRA_BASE_URL/rest/api/3/issue/MCO-123/transitions" \
    -H "Authorization: Bearer $JIRA_PAT" | jq .
  
  # Then apply transition
  curl -s -X POST "$JIRA_BASE_URL/rest/api/3/issue/MCO-123/transitions" \
    -H "Authorization: Bearer $JIRA_PAT" \
    -H "Content-Type: application/json" \
    -d '{"transition": {"id": "31"}}'  # Use ID from above
  ```

## When a skill says "publish to the issue tracker"

Create a Jira issue using the REST API with `JIRA_PAT`. Ensure the environment variables are set.

## When a skill says "fetch the relevant ticket"

Use the REST API to get issue details:
```bash
curl -s "$JIRA_BASE_URL/rest/api/3/issue/<issue-key>?expand=comments" \
  -H "Authorization: Bearer $JIRA_PAT"
```
