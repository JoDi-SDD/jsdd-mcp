# Plan

Phased plan toward the first version described in [docs/intent.md](docs/intent.md). Open questions and spikes live in [BACKLOG.md](BACKLOG.md); settled choices live in [docs/decisions.md](docs/decisions.md).

Each checklist item is delivered through one or more specs in `specs/` (see [specs/README.md](specs/README.md)); `specs/STATUS.md` shows progress. A spec's `refs` quote the item it delivers and the requirement IDs from [docs/requirements.md](docs/requirements.md) it covers.

Items end with the requirements they cover after a `·`: a group such as `FR-AUTH` means every requirement in it, and a full ID means only that one. `jsdd coverage --phase 1 --priority M --missing` lists the Phase 1 Musts that still need a spec.

| Phase | Goal | Status |
| --- | --- | --- |
| 0. Spikes and foundations | Settle the unknowns that can change the design, and stand up an empty but deployed skeleton | In progress |
| 1. MVP | The first version | Not started |
| 2+. Later | Work through `BACKLOG.md` "Next", in priority order | Not started |

## Phase 0: Spikes and foundations

### Spikes

Each is a spike spec run with `/jsdd:spike`; their goal questions are in `BACKLOG.md` "Spikes".

- [x] **Kit surface** · ND-06 · [001](specs/001-kit-surface/spec.md)
- [x] **Language and SDK** · ND-01, ND-03 · [002](specs/002-language-and-sdk/spec.md)
- [ ] **Client support** · ND-02

### Foundations

Each item becomes a feature spec (`/jsdd:specify`), split if it takes more than a few days.

- [ ] Kit release 1, machine output (DEC-09), made in `JoDi-SDD/JoDi-SDD` and pinned here:
  - A version stamped into bin/jsdd, plus `jsdd version --json`.
  - JSON for coverage, claim, release, claims and done.
  - Additive status/next fields, including `action`.
  - Exclusive spec and task claims.
  - `jsdd template <name>`.
  - The Release workflow attaches a bundle tarball (`skills/`, `bin/jsdd`, `templates/`, `starters/`) so dependents can pin its sha256 (DEC-13).
  - Check that the README's requirements match what bin/jsdd needs (bash 3.2 + awk per its header, or also jq and python3).
  - Checks to include: a race test between a spec claim and a task claim, local and remote; and a bundle-only test (`skills/`, `bin/jsdd`, `templates/`, `starters/` copied alone report the version and run `new`).
  - Start from an up-to-date main in the local kit clone.
- [ ] Kit release 2, client-neutral skills (DEC-09):
  - Agent prompts move into the core as `skills/<name>/references/`.
  - `argument-hint` moves under `metadata`.
  - Client-specific text is reworded.
  - Checks to include: `skills-ref validate` passes on every skill; `argument-hint` under `metadata` still gives hints in the plugin.
  - Needed before Phase 1's skills item, not before Phase 0 exits.
- [ ] Repository layout, build, test command, and CI (TypeScript, DEC-12; npm package with the pinned kit, DEC-13). · NFR-PORT-01
  - Windows with Git Bash: a Windows CI job here, required before 1.0 (specs/002-language-and-sdk Q10).
  - Checks to include: `npm pack` with the bundled kit, installed, still runs `bash <bundle>/bin/jsdd version --json`; a minimal `loadSkills` server on the bundled `skills/` passes MCP Inspector's skill verifier.

### Exit criteria

- All three spikes are closed.
- Kit release 1 (DEC-09) is published and pinned here.
- An empty server builds, passes its test command in CI, and starts against the pinned kit.

## Phase 1: MVP

Milestone 1: prove the enforcement idea. Tested with MCP Inspector and from at least one client.

- [ ] Server skeleton, bundled kit, and startup version check. · FR-KIT
- [ ] Skills served from the bundled kit (needs kit release 2). · FR-SKL, NFR-SEC-01
- [ ] Status and coverage tools. · FR-STAT
- [ ] Atomic claims and enforced ticking. · FR-WORK-01, FR-WORK-02, FR-WORK-03, NFR-STATE-01
