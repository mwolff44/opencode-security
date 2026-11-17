# Issue tracker: GitHub

Issues and specifications live in `mwolff44/opencode-security`. Use the `gh` CLI.
PRs as a request surface: no.

## Conventions

Create, inspect, comment on, label, and close issues using `gh issue`.
Use issue titles as linked names in human-facing reports.
Implementation issues close when merged; decision tickets close when their
resolution is recorded.

## Wayfinding operations

Map: one issue labelled `wayfinder:map`.
Tickets: native GitHub sub-issues labelled `wayfinder:research`,
`wayfinder:prototype`, `wayfinder:grilling`, or `wayfinder:task`.
Create issues first, then wire blocking edges.

Use these operations:

```bash
# Create the map or a decision ticket.
gh issue create --repo mwolff44/opencode-security --title "..." --body "..." --label wayfinder:map

# Read an issue, including its resolution comments.
gh issue view <number> --repo mwolff44/opencode-security --comments

# Link a child to its map.
gh api --method POST repos/mwolff44/opencode-security/issues/<map>/sub_issues \
  -F sub_issue_id=<child-database-id>

# Declare a blocker of a child ticket.
gh api --method POST repos/mwolff44/opencode-security/issues/<child>/dependencies/blocked_by \
  -F issue_id=<blocker-database-id>
```

IDs are numeric database IDs, not issue numbers or node IDs.
If native relationships are unavailable, use a task list on the map plus
`Part of` and `Blocked by` links on tickets, and explicitly record the fallback.

Frontier: open, unassigned children with no open blockers.
List the native children with
`gh api repos/mwolff44/opencode-security/issues/<map>/sub_issues --paginate`.
For each candidate, inspect its assignees and
`issue_dependencies_summary.blocked_by`; the latter counts open blockers.
If a fallback is in use, resolve every linked blocker's state instead.
First in map order wins.

Claim by assigning the ticket to the driving developer before research:
`gh issue edit <number> --repo mwolff44/opencode-security --add-assignee @me`.

Resolve with a sourced resolution comment, close the ticket, and append its
linked title and a one-line gist to the map's Decisions so far.
Unresolved decisions remain on their tickets; the map is an index, not a
duplicate store.
