# PR Description Examples

Use these examples to choose a presentation, not as mandatory templates. The measurements,
results, and scenarios below are illustrative; replace them with observed facts for the PR.
Follow required repository sections even when an example uses plain prose.

## Small Configuration Fix

Use a short explanation when motivation and impact fit in a few sentences. Skip a diagram
when it would repeat the configuration diff.

```markdown
Group the parser and rule-testing packages in the same dependency update.

Separate production and development groups currently update them independently, producing
incompatible AST types in rule tests. Grouping the family across dependency types keeps
those updates together. Existing merge and rebase settings stay unchanged.

Local configuration validation passes. Existing update branches still need to be refreshed.
```

## Dependency Refresh With Incomplete Validation

Report security improvements without implying that every advisory is fixed. Distinguish
completed local checks from pending CI and blocked integration coverage. Choose the title
using the repository's release policy; use `fix` when this change requires a patch release.

```markdown
Refresh compatible dependency versions and deduplicate the lockfile to reduce security
findings without taking unrelated major upgrades.

Local audit findings decrease from 18 to 11, including high-severity findings from 6 to 3.
These counts cover the local dependency tree, not the repository's open GitHub alerts.
The remaining high findings come through an exact-pinned build-tool family that needs
a coordinated follow-up upgrade. Public peer ranges and package versions stay unchanged.

Local builds and unit tests pass. CI is pending. Consumer integration tests are blocked
locally by missing private-registry credentials, so that coverage remains unverified.
```

## Complex Behavior Change

Use a compact visual when the ordering or ownership matters more than a list of changed
files. Pair it with a concrete explanation and evidence at the relevant behavioral seam.

````markdown
Invalidate the product cache before announcing an update to subscribers.

Subscribers currently receive the event while the previous cache entry is still available.
Their immediate read can therefore return the old product even though persistence succeeds.

```text
save product
  persist product
  invalidate product cache
  publish product-updated
```

The regression test subscribes to the event and immediately reads the product. Run locally,
it returns the old value before this change and the saved value afterward. The cache integration
suite also passes locally. Event payloads remain unchanged; publication now waits for invalidation.
````

## Risky Migration

Describe compatibility, rollout boundaries, and the point beyond which reverting code is
insufficient. A generic "two-way door" label would hide the important constraint here.

```markdown
Introduce the normalized invoice-reference column while keeping the legacy column readable.

The new lookup uses normalized references to prevent duplicate matches caused by casing
and whitespace. This PR adds the nullable column and dual writes; switching reads and
removing the legacy column remain separate changes.

### Evidence

Local migration and dual-write integration tests pass against a copy of the supported schema.
Mixed-version deployment tests pass in CI, confirming the previous app can still read invoices.
Production backfill throughput has not been measured, so it is not a rollout claim of this PR.

### Rollout and rollback

Deploy the additive schema before enabling dual writes. Keep legacy reads until the backfill
and consistency checks pass. Reverting the application leaves the new column in place;
dropping that column later would require a separate data-preservation decision.
```
