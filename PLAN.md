# Plan

Phased plan toward the first version described in [docs/intent.md](docs/intent.md). Open questions and spikes live in [BACKLOG.md](BACKLOG.md); settled choices live in [docs/decisions.md](docs/decisions.md).

Each checklist item is delivered through one or more specs in `specs/` (see [specs/README.md](specs/README.md)); `specs/STATUS.md` shows progress. A spec's `refs` quote the item it delivers and the requirement IDs from [docs/requirements.md](docs/requirements.md) it covers.

Items end with the requirements they cover after a `·`: a group such as `FR-AUTH` means every requirement in it, and a full ID means only that one. `jsdd coverage --phase 1 --priority M --missing` lists the Phase 1 Musts that still need a spec.

| Phase | Goal | Status |
| --- | --- | --- |
| 0. Spikes and foundations | Settle the unknowns that can change the design, and stand up an empty but deployed skeleton | Not started |
| 1. MVP | The first version | Not started |
| 2+. Later | Work through `BACKLOG.md` "Next", in priority order | Not started |

## Phase 0: Spikes and foundations

### Spikes

Each is a spike spec run with `/jsdd:spike`; their goal questions are in `BACKLOG.md` "Spikes".

- [ ] **Kit surface** · ND-06
- [ ] **Language and SDK** · ND-01, ND-03
- [ ] **Client support** · ND-02

### Foundations

Each item becomes a feature spec (`/jsdd:specify`), split if it takes more than a few days.

- [ ] Kit release with the DEC-06 changes, made in `JoDi-SDD/JoDi-SDD` and pinned here.
- [ ] Repository layout, build, test command, and CI. · NFR-PORT-01

### Exit criteria

- All three spikes are closed.
- The DEC-06 kit release is published and pinned here.
- An empty server builds, passes its test command in CI, and starts against the pinned kit.

## Phase 1: MVP

Milestone 1: prove the enforcement idea. Tested with MCP Inspector and from at least one client.

- [ ] Server skeleton, bundled kit, and startup version check. · FR-KIT
- [ ] Skills served from the bundled kit. · FR-SKL, NFR-SEC-01
- [ ] Status and coverage tools. · FR-STAT
- [ ] Atomic claims and enforced ticking. · FR-WORK-01, FR-WORK-02, FR-WORK-03, NFR-STATE-01
