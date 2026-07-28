# Branching strategy

Trunk-based development. `main` is the only long-lived branch and is always deployable — there are no persistent `develop`, `staging`, or per-environment branches. Environments are promoted by deploying the same build artifact through Dev → QA → Staging → Production (see the Release engineering documentation), not by checking out a different branch.

## Branch naming

| Pattern | Use |
|---|---|
| `feature/<ticket-id>-short-desc` | New functionality. Cut from `main`, short-lived (days, not weeks). |
| `fix/<ticket-id>-short-desc` | Bug fixes. Same rules as `feature/*`. |
| `hotfix/<ticket-id>-short-desc` | Emergency production fix, cut from the current production tag when `main` has diverged from what's deployed. Merges back to `main` immediately. Rare in a trunk-based flow — most fixes just go through the normal path since `main` usually equals production. |

Delete branches after merge. A branch living longer than ~1 week is a signal to split the work or ship it behind a feature flag instead of letting it diverge further.

## Feature flags, not long-lived branches

Incomplete or risky work merges to `main` continuously behind a feature flag that defaults off in production. This decouples "merged" from "released" — the alternative (a long-lived feature branch) just stores up merge conflicts for later.

## Pull request requirements

Every PR into `main` requires, before merge is possible:

1. **CI passing** — build, unit tests (per module: `itsm.Tests`, `sam.Tests`, `dev.Tests`, `docs.Tests`), lint, SAST/dependency scan.
2. **At least one approving review.** Paths under `apps/api/**` additionally require a CODEOWNERS review — see [`.github/CODEOWNERS`](.github/CODEOWNERS).
3. **All review conversations resolved.**
4. **Branch up to date with `main`** before merge (no merging a stale branch).
5. **No direct pushes to `main`** — everything goes through a PR, including admin/maintainer changes.

These are enforced by the repository ruleset at [`.github/rulesets/main-branch-ruleset.json`](.github/rulesets/main-branch-ruleset.json) (see that file's header comment for how to apply it — GitHub rulesets aren't something that can be pushed as a file; the JSON is imported once via Settings → Rules).

## Database migrations

Schema changes use the expand/contract pattern: a backward-compatible migration (new columns/tables) ships before the code that depends on it; old columns are dropped only after the new code path has run clean in production. This means a mid-rollout rollback of the app never leaves the database in a shape the previous code can't read.

## Commit messages

Conventional, imperative mood ("Add category tree to tickets", not "Added" or "Adds"). Reference the ticket ID when one exists.
