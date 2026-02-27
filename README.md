This is the Home page for The Attic AI



# .github

Org-level shared workflows and configuration for AtticAI repositories.

## Shared Workflows

### ClickUp PR Sync

Automatically syncs GitHub PR lifecycle events to ClickUp tickets. When a PR is opened, merged, or closed, the corresponding ClickUp ticket gets a status update and a comment linking back to the PR.

**How it works:** The workflow parses the PR's branch name for a ticket prefix (`CU-` or `RB-`), extracts the ClickUp task ID, and calls the ClickUp API to update status and post comments.

#### Prerequisites

1. `CLICKUP_API_TOKEN` must be configured as an org-level secret with access to target repos
2. Branches must follow the naming convention: `CU-{task_id}/{description}` or `RB-{task_id}/{description}`

#### Adoption

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

That's it. PRs on branches with `CU-` or `RB-` prefixes will sync automatically.

#### Configuration

All inputs are optional — defaults work for most repos.

| Input | Default | Description |
|-------|---------|-------------|
| `status_in_review` | `"in review"` | ClickUp status when a PR is opened |
| `status_complete` | `"complete"` | ClickUp status when a PR is merged |
| `status_qa` | `""` | If set, merged PRs go here instead of `status_complete` |
| `severity_field_id` | `""` | ClickUp custom field ID for severity label updates |
| `branch_prefixes` | `"CU-,RB-"` | Comma-separated branch name prefixes to detect |

#### Customizing Status Mapping

If your ClickUp list uses different status names, override them in the caller:

```yaml
jobs:
  sync:
    if: github.event.pull_request.draft == false
    uses: AtticAIInc/.github/.github/workflows/clickup-sync.yml@main
    with:
      status_complete: "completed"  # ReaderBee uses "completed" instead of "complete"
    secrets: inherit
```

#### Event Behavior

| PR Event | ClickUp Action |
|----------|---------------|
| Opened / Ready for review | Ticket → "in review", comment with PR link |
| Merged | Ticket → "complete" (or "qa"), comment with merge SHA |
| Closed without merge | Comment only, no status change |
| Labeled `severity:*` | Updates custom field (if `severity_field_id` is set) |

#### Error Handling

All ClickUp API calls use `continue-on-error`. If the ClickUp API is down or returns an error, the PR is never blocked. Warnings appear in the Actions log for debugging.
