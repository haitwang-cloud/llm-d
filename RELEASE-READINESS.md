# llm-d Release Readiness Process

## Overview

Every llm-d release ships on the **1st of the month**. Code freeze is the **23rd of the prior month** — your repo must have an approved release readiness issue by then to be included.

The release readiness board tracks which of the 26 shipping repos (12 production + 14 incubation) are ready for each release.

- **Board**: https://github.com/orgs/llm-d/projects/25
- **Issue template**: Filed on [llm-d/llm-d](https://github.com/llm-d/llm-d/issues/new?template=release-readiness.yml)

## Timeline (monthly)

| Date | Event |
|---|---|
| 25th (month before) | Lifecycle workflow opens the release cycle — creates tracking issue, labels, board options |
| 25th → 23rd | Repo maintainers file readiness issues, validation runs automatically |
| **23rd** | **Code freeze** — all readiness issues must be validated by this date |
| 1st (next month) | Release ships |
| Post-release | Lifecycle workflow closes the cycle — marks all issues as released |

## For Repo Maintainers

### 1. File a Release Readiness Issue

Go to **[llm-d/llm-d → Issues → New Issue → Release Readiness Request](https://github.com/llm-d/llm-d/issues/new?template=release-readiness.yml)**.

Fill in:
- **Which repo** — select your repo from the dropdown
- **Version** — the exact semver tag you plan to release (e.g., `v0.8.0`)
- **Prerequisites** — maintainer approval status, tests run, hardware affected
- **Documentation** — whether guides are updated and tested
- **Risk assessment** — breaking changes, cross-repo dependencies
- **Attestation** — hardware verification, rollback plan, release notes

### 2. What Happens Automatically

Once you submit:

1. **Labels applied**: `readiness/filed`, `release/vX.Y`, `tier/production` or `tier/incubation`
2. **Board sync**: Your issue appears on the [Release Readiness board](https://github.com/orgs/llm-d/projects/25) with all fields populated
3. **Validation runs**: 24 automated checks execute against your repo and post results as a comment

### 3. Automated Checks (24 total)

| Category | What's Checked |
|---|---|
| **CI Health** (5) | Default branch CI passing, no failing required checks, branch protection enabled, signed commits/DCO, no critical Dependabot alerts |
| **Governance** (5) | OWNERS, LICENSE, CONTRIBUTING.md, SECURITY.md, CODE_OF_CONDUCT.md exist |
| **Artifacts** (4) | Tag follows semver, container image exists (if Dockerfile present), CHANGELOG/release notes, no blocking draft releases |
| **Testing** (4) | Recent CI runs healthy, nightly E2E passing, no critical open issues, test coverage present |
| **Docs** (3) | README.md non-trivial, docs/ or api/ directory present, link health |
| **Hygiene** (3) | No merge conflicts on default branch, dependency pinning verified, cross-repo dependency compatibility |

### 4. Re-run Validation

If you've fixed issues flagged by the checks, comment `/revalidate` on your readiness issue. The validation workflow will re-run all 24 checks.

### 5. Label Progression

Your issue moves through these statuses (reflected on the board):

```
Filed → Validating → Passing or Failing → Validated → Released
                                    ↘ Blocked (manual, by release team)
```

| Label | Meaning |
|---|---|
| `readiness/filed` | Submitted, awaiting validation |
| `readiness/validating` | Automated checks running |
| `readiness/passing` | All automated checks pass |
| `readiness/failing` | One or more checks fail — fix and `/revalidate` |
| `readiness/blocked` | Blocked by release team (manual) |
| `readiness/validated` | Approved for release inclusion |
| `readiness/released` | Included in the final release |

## For the Release Team

### Board Views

| View | What It Shows |
|---|---|
| **Repo Status** | Table of all repos for the current release, sorted by validation score |
| **Release Board** | Kanban board grouped by status for the current release |
| **Blocked** | Only items with Status = Blocked |
| **Release History** | All releases (no filter) — historical audit trail |
| **By Tier** | Board view for the current release, grouped by repo tier |

All views except Release History and Blocked are filtered to the current release. To switch releases, update the filter (e.g., change `release:"v0.7 (May 2026)"` to `release:"v0.8 (Jun 2026)"`).

### Generate a Status Report

Run the lifecycle workflow manually with `status-report` action:

```
Actions → Release Readiness — Lifecycle → Run workflow → action: status-report
```

This posts a summary (validated/passing/failing/blocked/not-filed counts) as a comment on the tracking issue.

### Close a Release Cycle

After the release ships, run the lifecycle workflow with `close-cycle`:

```
Actions → Release Readiness — Lifecycle → Run workflow → action: close-cycle
```

This applies `readiness/released` to all issues, closes them, and posts a final summary.

### Open a Release Cycle Manually

The lifecycle workflow runs automatically on the 25th, but you can trigger it manually:

```
Actions → Release Readiness — Lifecycle → Run workflow → action: open-cycle
```

Optionally provide a `release_version` override (e.g., `v0.9`). If omitted, it auto-computes the next release.

## Secrets Required

| Secret | Scope | Purpose |
|---|---|---|
| `PROJECT_TOKEN` | PAT with `project`, `read:org` | Modify org-level project board fields |
| `ORG_GITHUB_TOKEN` | PAT with `repo` across llm-d orgs | Cross-repo validation checks |

Both are org-level secrets on the `llm-d` org.

## FAQ

**Q: My repo isn't in the dropdown. How do I add it?**
A: Edit `.github/ISSUE_TEMPLATE/release-readiness.yml` in `llm-d/llm-d` and add your repo to the dropdown options. Also add it to the `PRODUCTION_REPOS` or `INCUBATION_REPOS` list in `release-readiness-lifecycle.yml`.

**Q: I missed the code freeze. Can I still get included?**
A: Talk to the release team. They can manually add `readiness/validated` if the risk is acceptable.

**Q: The automated checks flagged something that doesn't apply to my repo.**
A: Some checks (e.g., container image, nightly E2E) are skipped if not applicable. If a check is incorrectly failing, comment on your readiness issue and the release team will review.

**Q: How do I see which repos haven't filed yet?**
A: Check the tracking meta-issue (`[Release Tracking] vX.Y`) — it has a checklist of all 26 repos. Or run the `status-report` action for a current count.

**Q: Do incubation repos have the same requirements as production?**
A: Yes, the same 24 checks run. The `tier/incubation` label is informational — the release team may apply different criteria for blocking.
