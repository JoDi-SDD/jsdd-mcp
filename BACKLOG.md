# Backlog

Open questions, spikes, and work after the MVP. The roadmap is in [PLAN.md](PLAN.md); decisions already made live in [docs/decisions.md](docs/decisions.md).

## MVP: open decisions

One line per open decision: its `ND-nn`, a one-line summary, and a suggested default. The full row (options, leaning, source) lives in `docs/decisions.md` "Still open". When a spike or spec settles it, the row moves to "Decided" there as `DEC-nn`, and the item here is ticked with a link to the spike or spec.

- [x] **ND-01 Language and runtime.** Suggested: Python, unless the TypeScript SDK supports Skills and Tasks clearly better. Settled as DEC-12 (TypeScript) by [002 Language and SDK](specs/002-language-and-sdk/spec.md).
- [ ] **ND-02 Which clients v1 is tested against.** Suggested: the clients that support Skills plus one tools-only client. Settled by the Client support spike.
- [x] **ND-03 How the server ships and bundles the kit.** Suggested: PyPI with `uvx jsdd-mcp`, the build fetching the pinned kit tag. Settled as DEC-13 (npm, `npx jsdd-mcp`, pinned kit tarball sha256) by [002 Language and SDK](specs/002-language-and-sdk/spec.md).
- [ ] **ND-04 How the supported kit range is declared.** Suggested: the `sdd-runtime` convention (one pinned version, checked against `bin/jsdd --json`). Settle in the foundations spec; it can build on `jsdd version --json` from kit release 1 (DEC-09) and the DEC-13 pin (kit version plus tarball sha256).
- [ ] **ND-05 Where server releases are announced.** Suggested: its own Announce step posting "JoDi-SDD MCP server vX.Y.Z" to org Announcements. Settle before the first release.
- [x] **ND-06 Claim store.** Suggested: whatever `bin/jsdd` uses today. Settled as DEC-08 by [001 Kit surface](specs/001-kit-surface/spec.md).

## Spikes

Run each with `/jsdd:spike <topic>`. The skill creates the spike spec from the entry here, one goal question per bullet.

### Kit surface

Settles ND-06, and lists the kit changes DEC-06 needs. Depends on nothing. Reads the kit at `JoDi-SDD/JoDi-SDD` (brief A1–A4, A6).

- How does `bin/jsdd` store and take claims today (location, atomicity, staleness, worktrees vs. remote), and what must change for FR-WORK-01 and FR-WORK-02?
- What do `bin/jsdd --json`, `coverage`, `claim`, and `done` expose today, and what output does the kit need to add for FR-STAT and FR-WORK-03?
- Do the kit's `skills/` follow the Agent Skills format, and which text names Claude Code-only features and needs rewording?

### Language and SDK

Settles ND-01 and ND-03. Depends on nothing.

- Which official MCP SDK, Python or TypeScript, supports the Skills and Tasks extensions and Elicitation better today?
- How would each package and install the server, and fetch and bundle a pinned kit release at build time?

### Client support

Settles ND-02. Depends on nothing.

- Which MCP clients support Skills, Elicitation, and Tasks today, and which should v1 be tested against?

## Next: after the MVP

- Leases with heartbeat and TTL, and `release_claim` with Elicitation takeover. · FR-WORK-04, FR-WORK-05
  - Start with a form elicitation round trip (`input_required`) against MCP Inspector (specs/002-language-and-sdk).
- Resources under `jsdd://`. · FR-RES-01
- Spec, plan, decision, and scaffold tools, with Elicitation for approvals. · FR-SPEC
- `run_verification` as a Task. · FR-LONG-01
- Release own claims on stdin close; `run_spec_to_pr`. · FR-WORK-06, FR-LONG-02
- `.mcpb` bundle for one-click install (DEC-13 follow-up, specs/002-language-and-sdk Q9).
