# .github

This is the **organization-level `.github` repository** for [Impel-Marketing-AI](https://github.com/Impel-Marketing-AI). GitHub treats this repo as a special location for shared configuration, reusable workflows, community health files, and label definitions that apply across all repositories in the org.

> **Visibility:** Private — contents are available to all repos within the org but not publicly visible.

---

## What's in this repo

```
.github/
├── CODEOWNERS                      # Org-wide code ownership rules
├── labels/
│   └── labels.json                 # Shared label definitions (QA, release-drafter, etc.)
├── workflows/
│   └── qa-gate.yml                 # Reusable QA Gate workflow
└── README.md                       # This file
```

### Reusable Workflows

| Workflow | File | Purpose |
|----------|------|---------|
| **QA Gate** | `workflows/qa-gate.yml` | Checks PRs for the `qa: tested` label before allowing merge. Reports pass/fail as a `QA Gate` status check. |

### Label Definitions

The `labels/labels.json` file defines standardized labels used across org repositories:

| Label | Color | Purpose |
|-------|-------|---------|
| `qa: needs testing` | — | PR requires QA verification |
| `qa: tested` | — | QA has verified the PR — unblocks merge |
| `qa: failed` | — | QA found issues — merge remains blocked |
| `feature` | — | Release-drafter: new feature |
| `enhancement` | — | Release-drafter: improvement |
| `fix` / `bugfix` / `bug` | — | Release-drafter: bug fix |
| `chore` | — | Release-drafter: maintenance |

---

## How repos consume reusable workflows

Any repository in the org can reference a reusable workflow from this repo using the `uses` keyword with the full org path:

```yaml
# .github/workflows/qa-gate.yml (in the consuming repo)
name: QA Gate
on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]
    branches: [develop, 'releases/*']

jobs:
  qa-gate:
    uses: Impel-Marketing-AI/.github/.github/workflows/qa-gate.yml@main
```

The caller workflow is typically only a few lines — all logic lives in the reusable workflow in this repo.

---

## Branch protection & rulesets

The **QA Gate** workflow is enforced via GitHub org-level rulesets:

- **Ruleset:** `QA Gate – Develop & Releases`
- **Scope:** `mfe-location-management` (POC), branches `develop` and `releases/*`
- **Required check:** `QA Gate`
- **Bypass:** Org admins only

---

## References

- [GitHub Docs — Creating a `.github` repository](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
- [GitHub Docs — Reusing workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)
- [GitHub Docs — Repository rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
