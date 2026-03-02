This is the Home page for The Attic AI



# .github

Org-level shared workflows and configuration for AtticAI repositories.

## ClickUp PR Sync

Two-part setup that connects GitHub PRs to ClickUp tickets:

1. **ClickUp's native GitHub integration** — creates rich PR cards on tickets (status, reviewers, etc.)
2. **Reusable workflow** — automates ticket status transitions (backlog → in review → complete)

### Part 1: Native GitHub Integration (one-time setup)

This gives you the rich PR cards on ClickUp tickets.

1. In ClickUp, go to **Settings → Integrations → GitHub**
2. Click **Connect** and authorize the AtticAIInc GitHub org
3. Select which spaces/repos to connect
4. Under integration settings, disable **"Post comments on GitHub"** (optional — prevents noisy PR comments)

Once connected, any branch or PR title containing `CU-{task_id}` automatically links to the ClickUp ticket with a rich card showing PR status, reviewers, and more.

### Part 2: Status Transition Workflow

The native integration links PRs but doesn't move ticket status. This workflow handles that:

- PR opened → ticket moves to **"in review"**
- PR merged → ticket moves to **"complete"** (or "qa")
- Severity label added → updates a custom field

Add this file to your repo at `.github/workflows/clickup-sync.yml`:

```yaml
name: ClickUp Sync

on:
  pull_request:
    types: [opened, closed, ready_for_review, labeled]

jobs:
  sync:
    if: github.event.pull_request.draft == false
    uses: AtticAIInc/.github/.github/workflows/clickup-sync.yml@main
    secrets: inherit
```

### Configuration

All inputs are optional — defaults work for most repos.

| Input | Default | Description |
|-------|---------|-------------|
| `status_in_review` | `"in review"` | ClickUp status when a PR is opened |
| `status_complete` | `"complete"` | ClickUp status when a PR is merged |
| `status_qa` | `""` | If set, merged PRs go here instead of `status_complete` |
| `severity_field_id` | `""` | ClickUp custom field ID for severity label updates |
| `branch_prefixes` | `"CU-,RB-"` | Comma-separated branch name prefixes to detect |

### Customizing Status Mapping

Override status names for repos that use different ClickUp lists:

```yaml
jobs:
  sync:
    if: github.event.pull_request.draft == false
    uses: AtticAIInc/.github/.github/workflows/clickup-sync.yml@main
    with:
      status_complete: "completed"
    secrets: inherit
```

### Prerequisites

- `CLICKUP_API_TOKEN` configured as an org-level secret
- Branches follow `CU-{task_id}/{description}` or `RB-{task_id}/{description}` naming

### Error Handling

All ClickUp API calls use `continue-on-error`. If the API is down or returns an error, the PR is never blocked. Warnings appear in the Actions log.
