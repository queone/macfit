# macfit History

> One section per release, newest first: what the release added, changed, removed, or deferred in the product, its design, and its code, and why. Governance, toolchain, and release mechanics stay in `CHANGELOG.md` and git. Settled design is in `arch.md`. Search it to learn why a release changed the product.

## v1.13.0

Changed:
- `st` shows the update result when the proxy check succeeds but the cache write fails.
- `st` shows `unknown (ERROR)` only when the proxy check itself fails.

Decisions:
- The Director chose direct handling with no acceptance criteria document.

Risks:
- A failed cache write now goes unreported, so every `st` asks the proxy until the write succeeds.

## v1.12.0

Changed:
- `st` says `none (vX.Y.Z is the latest, checked TIME)` only when the proxy's latest equals the running version.
- `st` says `none (checked TIME)` when the proxy's latest is older, because the proxy can lag a new tag.
- The update check asks the proxy again after an upgrade, because gkit v0.96.0 keys its cache by the running version.
- `st` keeps a joined update-check error on one line, with `; ` between the errors.

Deferred:
- `st` still shows `unknown` when the proxy request succeeds but the cache write fails. `plan.md` tracks the fix.

Decisions:
- The Director kept the cache-write case out of this release, to keep the change to the display rule.

Risks:
- The first `st` after this upgrade asks the proxy once more, because older caches record no running version.
