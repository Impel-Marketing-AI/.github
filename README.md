# .github

This is the **organization-level `.github` repository** for [Impel-Marketing-AI](https://github.com/Impel-Marketing-AI). GitHub treats this repo as a special location for shared configuration, reusable workflows, community health files, and label definitions that apply across all repositories in the org.

> **Visibility:** Private — contents are available to all repos within the org but not publicly visible.

---

## What's in this repo

```
.github/                              ← repo root
├── .github/
│   ├── release-drafter.yml           ← Org-wide Release Drafter config
│   └── workflows/
│       └── qa-gate.yml               ← Reusable QA Gate workflow
├── CODEOWNERS                        ← Org-wide code ownership rules
├── labels/
│   └── labels.json                   ← Shared label definitions
└── README.md                         ← This file
```

### Release Drafter (org-level config)

The `.github/release-drafter.yml` file in this repo serves as the **default Release Drafter configuration for all repositories** in the org. This works because Release Drafter has built-in support for org-level config inheritance from the `.github` repository.

**How it works:**

1. When Release Drafter runs in a repo, it looks for a config file in this order:
   - The repo's own `.github/release-drafter.yml` (or the path set via `config-name`)
   - The org `.github` repo's `.github/release-drafter.yml` (automatic fallback)
2. If a repo has no local config and no `config-name` override, it automatically picks up the org-level config from this repo.

**For consuming repos, no extra setup is needed** — just ensure:
- The repo's build workflow calls `release-drafter/release-drafter@v6` without a `config-name` input
- There is **no** local `.github/release-drafter.yml` in the repo (otherwise it takes precedence)

A repo can still override the org config by adding its own `.github/release-drafter.yml`.

> **Reference:** [Release Drafter — Configuration options](https://github.com/release-drafter/release-drafter#configuration-options)

### Reusable Workflows

| Workflow | File | Purpose |
|----------|------|---------|
| **QA Gate** | `.github/workflows/qa-gate.yml` | Checks PRs for the `qa: tested` label before allowing merge. Reports pass/fail as a `QA Gate` status check. |

### Label Definitions

The `labels/labels.json` file defines standardized labels used across org repositories:

| Label | Color | Purpose |
|-------|-------|---------|
| `qa: needs testing` | `#D4A017` | PR is ready for QA verification |
| `qa: tested` | `#13A688` | QA has verified the changes in this PR |
| `qa: failed` | `#D03C38` | QA found issues with this PR |
| `feature` | `#410099` | Release-drafter: new feature or capability |
| `enhancement` | `#635DFF` | Release-drafter: improvement to existing feature |
| `fix` | `#E5534B` | Release-drafter: bug fix |
| `bug` | `#E5534B` | Release-drafter: something isn't working |
| `chore` | `#8B8680` | Release-drafter: maintenance, refactoring, or tooling |
| `dependencies` | `#1D76DB` | Release-drafter: dependency updates |

---

## How repos consume reusable workflows

Any repository in the org can reference a reusable workflow from this repo using the `uses` keyword with the full org path. Per [GitHub security best practices](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#using-third-party-actions), pin the reference to a **commit SHA** rather than a branch name to ensure immutable, auditable builds.

```yaml
# .github/workflows/qa-gate.yml (in the consuming repo)
name: QA Gate
on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]
    branches: [develop, 'releases/*']

jobs:
  qa-gate:
    # Pin to a specific commit SHA for security — update when the reusable workflow changes
    uses: Impel-Marketing-AI/.github/.github/workflows/qa-gate.yml@afe23d613d2eff78c62c4e6241674aa2f9ce8156
```

> **Tip:** To find the latest SHA, run `git ls-remote https://github.com/Impel-Marketing-AI/.github refs/heads/main` or check the latest commit on the `main` branch.

The caller workflow is typically only a few lines — all logic lives in the reusable workflow in this repo.

### Required permissions

The reusable QA Gate workflow needs to remove labels and post comments on pull requests. Because reusable workflows **inherit the caller's `GITHUB_TOKEN` permissions**, the caller must ensure the token has the required scopes.

| Permission | Level | Why |
|-----------|-------|-----|
| `pull-requests` | `write` | Remove the `qa: tested` label when the PR author self-applies it; post an explanatory comment |
| `contents` | `read` | Default read access for checkout (already granted by GitHub's default token) |

**Option A — Rely on org/repo defaults (recommended)**

If your org or repo default token permissions already include `pull-requests: write` (GitHub's default for non-fork PRs), no `permissions` block is needed in the caller workflow:

```yaml
name: QA Gate
on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]
    branches: [develop, 'releases/*']

jobs:
  qa-gate:
    uses: Impel-Marketing-AI/.github/.github/workflows/qa-gate.yml@afe23d613d2eff78c62c4e6241674aa2f9ce8156
```

**Option B — Explicit permissions (if org defaults are restrictive)**

If your org or repo has set the default `GITHUB_TOKEN` to read-only, you must explicitly grant the required permissions in the caller:

```yaml
name: QA Gate
on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]
    branches: [develop, 'releases/*']

permissions:
  contents: read
  pull-requests: write

jobs:
  qa-gate:
    uses: Impel-Marketing-AI/.github/.github/workflows/qa-gate.yml@afe23d613d2eff78c62c4e6241674aa2f9ce8156
```

> **Warning:** Setting `permissions: contents: read` alone (without `pull-requests: write`) will break the QA Gate workflow — it will be unable to remove labels or post comments.

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
- [GitHub Docs — Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
- [Release Drafter — Configuration](https://github.com/release-drafter/release-drafter#configuration-options)
