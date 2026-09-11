# app-notes

**Notes, on [`mokuroku`](https://github.com/kotoba-lang/mokuroku).**

Design: [ADR-2608035000](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2608035000-app-standard-application-suite-on-a-shared-catalog-kernel.edn).

Capability: `fs/app-data`. Nothing in this repo performs the effect — the host
supplies the provider function, and that is where the grant is spent.

## Three decisions

**A title is derived, never stored.** It is the first non-blank line of the
body. A stored title drifts — the user edits the first line and the list keeps
showing what they typed last week — and deriving makes that impossible.

**Pinning is a view decision, not a sort decision.** Baking pinned-first into
the comparator means a user who sorts by title still gets pinned notes first,
which is not what they asked for.

**A first run is not a failure.** Unlike a process table, an empty note store
is normal — it is what a new install looks like. Saying "nothing matches" to
someone who has never written a note describes a filter they never set.

`fs/app-data` is the app's private store, not the user's filesystem. A notes
app has no business reading Documents; it reads the notes it created. Same
capability aiueos's `examples/apps/notes.edn` requests, for the same reason.

## Test

```sh
kbb -M:local:test    # sibling checkouts
kbb -M:test          # pinned git deps
kbb -M:lint
```

design-quality: 100.00 on every window state including awaiting-grant (2026-08-03).
