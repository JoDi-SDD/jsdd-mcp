# Constitution

The non-negotiable rules every spec and plan is checked against. Each plan has a "Constitution check" that names the articles it touches.

`docs/intent.md` stays the source of product intent; the product articles distill the parts of it that constrain day-to-day design. To change an article, record the decision in the amendments log at the bottom, and update the intent doc if the product intent changed too.

## Product articles

- **P1 One rule source.** No tool decides whether an SDD change is allowed. It calls the bundled `bin/jsdd` and returns its verdict; no check is reimplemented in the server.
- **P2 Pinned public kit.** Code touches the kit only through `bin/jsdd --json` and the bundled `skills/` of the pinned release. Startup exits with a clear message when the kit version is outside the supported range.
- **P3 Stateless processes.** Nothing a later session needs lives in process memory. Killing a server process at any time loses no claim and no state; all state is in repository files or git.
- **P4 Client-neutral.** No code, tool output, or served content names the Claude Code plugin or a specific client, and the bundled skills are served byte-for-byte.

## Engineering articles

1. **Happy path first.** Build the simplest version that moves the current roadmap item forward. Anything else goes to the backlog, not into the current spec.
2. **Tests prove acceptance criteria.** Every acceptance criterion maps to an automated test or a recorded manual check. A task is done only when its check passes.
3. **Small, traceable increments.** One spec is one reviewable slice (a few days at most; split it otherwise). One task is about one commit, and commit messages start with the spec and task ID, for example `[003] T2 Add invite table`.
4. **Specs are living.** When implementation shows the spec or plan is wrong, update them first, then the code. Code never silently diverges from its spec.
5. **Decisions are written down.** Every choice between real alternatives gets a row in the plan's Decisions table (or a spike's Recommendation). A choice that settles an open decision moves its row in `docs/decisions.md` from Still open to Decided, and ticks the backlog item.
6. **Verify current docs.** Before using a library, SDK, or tool API, check its current documentation rather than relying on memory, and note the version in the plan.

## Project specifics

What the kit's skills and the reviewer need to know about this project.

- **Test command:** TBD. It depends on the language (ND-01); the foundations spec sets it once the language spike settles that.

### Architecture checklist

- **Single kit adapter.** All calls to `bin/jsdd` go through one adapter module. Tools never spawn `bin/jsdd` directly.

## Amendments

| Date | Article | Change | Why |
| --- | --- | --- | --- |
