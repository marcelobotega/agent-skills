# PR Description Examples

Use these examples to choose a presentation, not as mandatory templates. Replace illustrative
facts and measurements with observed facts. Follow required repository sections even when an
example uses plain prose. Keep execution status in the agent's response or GitHub Checks.

## Small Configuration Fix

Use a short explanation when motivation and impact fit in a few sentences. Skip a diagram
when it would repeat the configuration diff.

```markdown
Group the parser and rule-testing packages across production and development dependencies.
Separate updates currently produce incompatible AST types in rule tests.

Existing update branches need to be refreshed to pick up the grouping.
```

## Dependency Security Fix

Scope security improvements without implying every advisory is fixed. Choose the title using
the repository's release policy; use `fix` when this change requires a patch release.

```markdown
Update compatible dependencies to reduce audit findings from 18 to 11, including high-severity
findings from 6 to 3. These counts cover the local dependency tree, not open GitHub alerts.

The remaining high findings come through an exact-pinned build-tool family that needs a
coordinated upgrade. This change leaves public peer ranges unchanged.
```

## Complex Behavior Change

Use a compact visual when ordering or ownership is clearer as a diagram.

````markdown
Invalidate the product cache before announcing an update to subscribers.
Subscribers currently receive the event while the previous cache entry is still available,
so an immediate read can return the old product.

```text
save product
  persist product
  invalidate product cache
   publish product-updated
```
````

## Risky Migration

Describe rollout boundaries and the point beyond which reverting code is insufficient.

```markdown
Add a nullable normalized invoice-reference column and dual writes as the first step toward
preventing duplicate matches caused by casing and whitespace. Legacy reads stay in place;
switching reads and removing the old column remain separate changes.

### Rollout and rollback

Deploy the schema before enabling dual writes. Keep legacy reads until the backfill and
consistency checks complete. Reverting the application leaves the new column in place;
dropping it requires a separate data-preservation decision.
```
