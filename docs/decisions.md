# Decisions

The register of decisions. Settled choices get a `DEC-nn` row; open ones keep their `ND-nn` row under "Still open", with a one-line item in [BACKLOG.md](../BACKLOG.md). A spike's Recommendation, or a plan's cross-cutting Decisions row, settles an ND into a DEC here with the same reasoning, and the BACKLOG item is ticked in the same change. Plans cite the `DEC-nn` they rely on. In autonomous mode, the `sdd-product` agent's decisions go under "Assumed" as `AS-nn` rows until the owner confirms one (it becomes a `DEC-nn` row) or rejects it (it is removed, with a BACKLOG item or a spec to undo it).

## Decided

| ID | Decision | Choice | Why | Date |
|---|---|---|---|---|
| DEC-01 | Channels | The Claude Code plugin and this server are peer channels for one kit core (`skills/`, `bin/jsdd`, `templates/`, `starters/`, file formats), which lives in the kit repo. | One product, delivered to different clients without forking it. (docs/legacy/brief.md D1) | 2026-11-27 |
| DEC-02 | Where the server lives | Its own repo, `JoDi-SDD/jsdd-mcp`. | Server-only releases never reach plugin users; follows the `sdd-runtime` pattern. Reopens if the server needs kit internals beyond `bin/jsdd --json` and `skills/`. (docs/legacy/brief.md D2) | 2026-11-27 |
| DEC-03 | Versioning | Own version and release; bundles a pinned kit release and checks its version at startup. Moving to a new kit release is a pin bump. | Kit and server release independently. (docs/legacy/brief.md D3) | 2026-11-27 |
| DEC-04 | Registration | The plugin never registers the server; installing it stays optional. | Plugin users never need the server's runtime. (docs/legacy/brief.md D4) | 2026-11-27 |
| DEC-05 | Boundary rules | Core never names a channel; channels never name each other; rules have one implementation in `bin/jsdd`; the server uses only `bin/jsdd --json` and the shipped `skills/`. | Keeps one source of rules. Recorded as P1, P2, P4 in the constitution. (docs/legacy/brief.md D5) | 2026-11-27 |
| DEC-06 | Kit changes first | Milestone 1 waits on kit releases: JSON output for coverage, claiming, and ticking; atomic claims in the CLI; client-neutral skill text. | The server may not reimplement or reach around the kit. (docs/legacy/brief.md D6) | 2026-11-27 |
| DEC-07 | Transport | v1 is local stdio only, one process per session. | Simplest, and no hosted service to run. (docs/legacy/brief.md §3) | 2026-11-27 |

## Still open

| ID | Decision | Options | Current leaning | Source |
|---|---|---|---|---|
| ND-01 | Language and runtime | Python; TypeScript | Python: nothing new to install, `uvx` one-line install. Check SDK support for Skills and Tasks first. | docs/legacy/brief.md Q1 |
| ND-02 | Which clients v1 is tested against | Clients that support Skills; any client with tools only | Decide from the client support matrix; if few support Skills, lead with tools. | docs/legacy/brief.md Q5 |
| ND-03 | How the server ships and bundles the kit | PyPI (`uvx`); npm; run from a git tag | Follows ND-01. | docs/legacy/brief.md Q6 |
| ND-04 | How the supported kit range is declared and checked | Same convention as `sdd-runtime`'s pin; a range in package metadata | Share the `sdd-runtime` convention. | docs/legacy/brief.md Q7 |
| ND-05 | Where server releases are announced | Own Announce step to org Announcements; none until 1.0 | Own step, titled "JoDi-SDD MCP server vX.Y.Z". | docs/legacy/brief.md Q8 |
| ND-06 | Claim store | Whatever `bin/jsdd` uses today; files under the git common dir; git refs on the remote | Use what `bin/jsdd` does, so all channels see the same claims. | docs/legacy/brief.md §7, A1 |

## Assumed

| ID | Decision | Choice | Class | Basis | Reversal | Spec | Date |
|---|---|---|---|---|---|---|---|
