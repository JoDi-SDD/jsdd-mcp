---
id: 002
title: "Language and SDK"
type: spike
status: done
phase: 0
depends_on: []
refs:
  - "BACKLOG.md Spikes: Language and SDK"
  - "PLAN.md Phase 0 Spikes: Language and SDK · ND-01, ND-03"
updated: 2026-11-27
---

# 002 · Language and SDK

<!--
A spike answers questions and settles decisions; it does not ship production code. Run it with /jsdd:spike 002:
the spike-researcher agent prepares a briefing and questions, you answer and add information over as many sessions
as it takes, and everything learned is recorded here. Prototypes are throwaway and live under specs/002-*/prototype/.
Progress counts the checkboxes in Goal questions, Questions for you, and Deliverables.
-->

## Inbox

<!-- Drop notes, links, constraints, or decisions here at any time, in any form. The next /jsdd:spike session moves them into the Log and Findings and clears this section. -->

## Why

Settles **ND-01** (language and runtime) and **ND-03** (how the server ships and bundles the kit). No code or tooling goes into this repository until ND-01 is decided, so this unblocks the Phase 0 foundations item "Repository layout, build, test command, and CI" and, through it, Phase 1. The bundle contents are fixed by DEC-11 (`skills/`, `bin/jsdd`, `templates/`, `starters/` from the pinned kit release); the startup version check builds on `jsdd version --json` (DEC-09).

## Goal questions

<!-- What the spike must answer, from BACKLOG.md. Ticked once Findings answers it completely. -->

- [x] **G1** Which official MCP SDK, Python or TypeScript, supports the Skills and Tasks extensions and Elicitation better today?
- [x] **G2** How would each package and install the server, and fetch and bundle a pinned kit release at build time?

## Briefing

<!-- Prepared by the spike-researcher agent: what the repository and current docs already say, with sources. -->

### G1 · SDK support for Skills, Tasks, Elicitation

- Known: the server must declare `io.modelcontextprotocol/skills` and implement `skills/list`, `skills/get`, and `resources/read`, with a SHA-256 and size manifest computed from the bytes it serves (docs/requirements.md FR-SKL-01..03).
- Known: Tasks are wanted for `run_verification` (FR-LONG-01, S) and `run_spec_to_pr` (FR-LONG-02, C). Elicitation is wanted for takeover (FR-WORK-05, S) and approvals (FR-SPEC-02, S). None of these is a Phase 1 Must (PLAN.md Phase 1).
- Known: transport is local stdio only (DEC-07). Processes keep no state (P3), so the stateless 2026-07-28 core suits it. Serving skills waits on kit release 2 (PLAN.md Phase 1, DEC-09).
- Suggested: Python, "unless the TypeScript SDK supports Skills and Tasks clearly better" (BACKLOG.md ND-01; decisions.md ND-01 Current leaning).
- Unknown: which SDK the owner would rather maintain, and whether depending on unreleased or experimental SDK APIs is acceptable.

### G2 · Packaging, install, and bundling the pinned kit

- Known: the bundle is `skills/`, `bin/jsdd`, `templates/`, and `starters/` from a pinned kit release tag on GitHub (DEC-11, FR-KIT-01). Startup reads the version from `jsdd version --json` (DEC-09, FR-KIT-02). It must run on macOS, Linux, and Windows with Git Bash (NFR-PORT-01).
- Known: `bin/jsdd` is a bash script (shebang `#!/usr/bin/env bash`, "Plain bash 3.2 + awk"). The kit README lists `bash`, `awk`, `jq`, and `python3` as requirements; the legacy brief says "bash/jq/python3". So users need bash whatever language the server is written in. (The shebang comment and the README disagree on jq/python3; worth checking in the kit.)
- Known: kit releases are a git tag plus a GitHub release with notes only. No archive is attached (kit .github/workflows/release.yml runs `gh release create` with notes only), so GitHub publishes no digest to check against.
- Suggested: PyPI with `uvx jsdd-mcp`, the build fetching the pinned kit tag (BACKLOG.md ND-03). The supported range would follow the sdd-runtime convention of one pinned version (BACKLOG.md ND-04, settled later in the foundations spec).
- Unknown: whether the org can publish to PyPI or npm under the name `jsdd-mcp`, how the fetched kit is verified, and whether a `.mcpb` bundle is wanted.

### Researched

All checked 2026-11-27.

- **Spec.** The current revision is 2026-07-28. Its core is stateless: no `initialize` handshake; `server/discover` replaces it. Server-to-client requests become multi round-trip requests (MRTR): a tool returns `input_required`. Tasks moved out of the core into an extension (blog.modelcontextprotocol.io/posts/2026-07-28).
- **Skills extension** `io.modelcontextprotocol/skills` (SEP-2640) is Final: `skills/list`, `skills/get`, `resources/read`, optionally `resources/directory/read`. Skills should stay under 512 files or 16 MiB each (modelcontextprotocol.io/extensions/skills/overview).
- **Tasks extension** `io.modelcontextprotocol/tasks` (SEP-2663) is Final, schema 2026-07-28 Stable: `tasks/get`, `tasks/update`, `tasks/cancel`; a tool result can be `resultType: "task"` (modelcontextprotocol.io/seps/2663-tasks-extension; ext-tasks README).
- **SDK tiers.** Python and TypeScript are both Tier 1. Tiers do not require extensions such as Tasks (modelcontextprotocol.io/docs/sdk, /community/sdk-tiers).
- **Python** `mcp` 2.3.0, released 2026-10-02, Python >=3.10 on PyPI (v2 docs say 3.11+). v2.0.0 (2026-07-28) implements 2026-07-28; 1.x gets security fixes only (pypi.org/project/mcp; python-sdk releases).
  - Skills: PR #3485 (`mcp.server.skills`, handler-based, no filesystem discovery) merged 2026-10-11, **not yet released**. A filesystem provider is pending in #3795 (python-sdk #3485; ext-skills docs/implementations.md).
  - Tasks: not implemented. The roadmap says the 2026-07-28 design is wire-incompatible with the older Tasks. Issue #2806 open; PR #3005 passed interop tests 2026-11-26 but defers background execution and cancellation (python-sdk ROADMAP.md, #2806).
  - Elicitation: form mode through MRTR (`InputRequiredResult`, `ctx.elicit()`) documented. No URL mode found in the v2 docs read (py.sdk.modelcontextprotocol.io/v2).
- **TypeScript** v2.4.0, released 2026-10-23: `@modelcontextprotocol/server` and `@modelcontextprotocol/client` (`node` at 2.1.2). v2 implements 2026-07-28; 1.x (1.32.2) is maintenance only (typescript-sdk releases; ts.sdk.modelcontextprotocol.io/v2).
  - Skills: **released** in 2.4.0 as `ext/skills`. `loadSkills('./skills')` builds a digest manifest from a folder and refuses to serve bytes that no longer match it. `attach` is experimental. Works on 2025-11-25 and 2026-07-28 (ts.sdk…/v2/servers/skills.html).
  - Tasks: on 2026-07-28 the core serves only `tasks/get` and `tasks/cancel` through explicit-schema handlers. `@modelcontextprotocol/ext-tasks` gives requester (client) APIs and only 2025-11-25 receiver (server) support. No server-side guide for 2026-07-28 Tasks (ts.sdk…/v2/migration/support-2026-07-28.html; ext-tasks README).
  - Elicitation: form and URL modes; on 2026-07-28 the server returns `inputRequired(...)` or `inputRequired.elicitUrl(...)`, with a shim for 2025 clients (ts.sdk…/v2/servers/elicitation.html).
- **Hand-rolling.** Both SDKs let custom methods be registered (Python `add_request_handler`; TypeScript `{id, setup(ctx)}`, experimental), so Skills could be hand-rolled in either (SDK v2 docs).
- **Hatchling.** A custom build hook (`hatch_build.py`, `BuildHookInterface.initialize`) can add fetched files through `build_data["force_include"]`. `shared_scripts` keeps permissions from hatchling 1.24.1. Wheels normalise modes to 0644/0755 (hatch.pypa.io build-hook reference, wheel builder, history; discuss.python.org wheel-spec thread).
- **npm.** File modes in a package tarball are unreliable after install; only `bin` targets are made executable (npm/pacote #39). Calling `bash <bundle>/bin/jsdd` avoids the problem in both ecosystems.
- **uvx.** `uvx jsdd-mcp@1.2.3` pins an exact version; `uvx --from git+https://…@tag` runs from a tag (docs.astral.sh/uv/guides/tools, dated 2026-10-14). uv can download a managed Python when none is found (uv python-versions docs).
- **Integrity.** GitHub shows a `sha256:` digest for uploaded release files (since 2025-06-03). Auto-generated source archives carry no checksum guarantee; GitHub recommends release files for stable checksums (github.blog changelog 2025-06-03; github.blog 2023 archive-stability post).
- **MCPB** (`.mcpb`): a zip plus `manifest.json`, server types node, python, uv (experimental, manifest 0.4+), binary. One-click install in one desktop host on macOS and Windows; the repo recommends Node because that host ships it (github.com/modelcontextprotocol/mcpb).
- Client support matrix (modelcontextprotocol.io/extensions/client-matrix) could not be read; it belongs to the Client support spike.

## Questions for you

<!-- Prepared by the agent, highest impact first; each says which goal questions it serves, why it matters, the options, and a suggested answer. Ticked when answered or skipped. -->

- [x] **Q1** Is it acceptable for users to need Node.js (via `npx`) on top of bash, or must the server add nothing beyond what the kit already requires? (G1, G2) *Why:* decides how much TypeScript's lead on Skills is worth. Python with `uvx` needs only uv, which can fetch Python itself; TypeScript needs Node 18+. *Options:* Node is fine / only uv or Python / either, if documented. *Suggested:* either, because both SDKs are Tier 1 and the kit already asks for several tools.
- [x] **Q2** Which language would you rather maintain this server in long term? (G1) *Why:* both SDKs are Tier 1 and both lag on 2026-07-28 Tasks; the differences are temporary, the maintenance cost lasts. *Options:* Python / TypeScript / no preference. *Suggested:* Python, matching the ND-01 leaning and the kit's existing `python3` requirement.
- [x] **Q3** Should Skills support released in an SDK today count for more than support merged but not yet released? (G1) *Why:* TypeScript 2.4.0 ships `loadSkills`; Python's Skills is merged, unreleased, with no filesystem loader. The skills item can't start before kit release 2 anyway. *Options:* released today decides / likely released by Phase 1 is enough / we can hand-roll the three methods. *Suggested:* likely released by Phase 1 is enough, with hand-rolling as a fallback.
- [x] **Q4** May the server depend on experimental SDK APIs, or an SDK installed from git, that can change in a minor release? (G1, G2) *Why:* TypeScript's `skills.attach` and extensions API are experimental; Python Skills or Tasks today means pinning a git commit, which a PyPI package can't depend on. *Options:* yes, with an exact pin / experimental is fine but released versions only / stable APIs only. *Suggested:* experimental is fine with exact pins, released versions only.
- [x] **Q5** Should SDK Tasks support count in the language choice for now? (G1) *Why:* neither SDK has full server-side 2026-07-28 Tasks, and FR-LONG is only Should/Could. *Options:* ignore Tasks for now / weigh it heavily. *Suggested:* ignore for now; revisit when FR-LONG is planned.
- 2026-11-27 · answer to Q6 · Publish to npm as unscoped `jsdd-mcp` (`npx jsdd-mcp`), with trusted publishing from GitHub Actions.
- 2026-11-27 · answer to Q7 · The kit's Release workflow attaches a bundle tarball (folded into kit release 1) and the build pins its sha256; tag plus commit SHA is the stopgap until then.
- 2026-11-27 · answer to Q8 (reworded for npm) · The repository holds only the pin; a build script fetches and verifies the kit, which is gitignored. The published npm package contains it.
- 2026-11-27 · answer to Q10 · macOS and Linux first; Windows with Git Bash required before 1.0, with Windows CI added in the foundations spec.
- 2026-11-27 · answer to Q9 · `.mcpb` bundle goes to the backlog, after the MVP.
- [x] **Q6** Can the org publish to PyPI and/or npm as `jsdd-mcp`, with trusted publishing from GitHub Actions? (G2) *Why:* `uvx jsdd-mcp` / `npx jsdd-mcp` need the name and a publishing setup; otherwise the fallback is running from a git tag. *Options:* PyPI / npm / both / git tag only for now. *Suggested:* register for the chosen language only.
- [x] **Q7** How should the build verify the kit it fetches? (G2) *Why:* kit releases attach no files, so GitHub gives no digest; source archives have no checksum guarantee. Adding a release file is a kit change first (boundary rule 4). *Options:* kit Release workflow attaches a tarball and we pin its `sha256:` / pin tag plus commit SHA and fetch with git / pin a sha256 of the source archive. *Suggested:* kit attaches a tarball, folded into kit release 1, with tag plus commit SHA as a stopgap.
- [x] **Q8** Should the bundled kit also go into the source package, so the server builds offline? (G2) *(Reworded for npm after Q2: vendor the kit into the repo, or fetch it at build and gitignore it; the npm package always contains it.)* *Why:* fetching only at wheel build means source builds need GitHub access. *Options:* in the sdist and wheel / fetched at wheel build only. *Suggested:* in both; the kit is small and the pinned copy travels with the source.
- [x] **Q9** Is a `.mcpb` bundle wanted for v1? (G2) *Why:* one-click install in one desktop host, favours Node, adds a release artifact and CI work. *Options:* v1 / backlog / never. *Suggested:* backlog.
- [x] **Q10** Must Windows with Git Bash work from the first build, or can it follow macOS and Linux? (G2) *Why:* the server must locate `bash` to run `bin/jsdd` (NFR-PORT-01); this shapes the kit adapter and adds a Windows CI job. *Options:* from the first build / before 1.0 / best effort. *Suggested:* before 1.0, with Windows CI added in the foundations spec.

## Log

<!-- Append-only, oldest first: "- YYYY-MM-DD · <source> · <what was answered, added, or found>". -->

## Findings

<!-- Per goal question. Label each point as a decision (by the user), a fact (with source), or an assumption (to confirm). -->

### G1

- **decision:** TypeScript, with the official TypeScript SDK v2 (`@modelcontextprotocol/server`), as the language to maintain long term (Q2).
- **decision:** a Node.js requirement on top of bash is acceptable if documented (Q1).
- **decision:** released SDK versions only, pinned exactly; experimental APIs (such as `skills.attach`, the extensions API) are allowed under that pin (Q3, Q4).
- **decision:** SDK Tasks support does not weigh in the choice now; revisit when FR-LONG is planned (Q5).
- **fact:** TypeScript SDK 2.4.0 (2026-10-23) ships Skills as `ext/skills` with `loadSkills` building a digest manifest from a folder, and Elicitation in form and URL modes (ts.sdk.modelcontextprotocol.io/v2, Researched). Python's Skills is unreleased (python-sdk #3485).
- **fact:** server-side 2026-07-28 Tasks are incomplete in both SDKs (Researched).

### G2

- **decision:** ship on npm as unscoped `jsdd-mcp`, installed with `npx jsdd-mcp` (exact version: `npx jsdd-mcp@x.y.z`), published from GitHub Actions with trusted publishing (Q6).
- **decision:** the kit's Release workflow attaches a bundle tarball (`skills/`, `bin/jsdd`, `templates/`, `starters/`) to each release; this server pins the kit version and the tarball's sha256. Added to kit release 1. Until it ships, pin the tag plus its commit SHA and fetch with git (Q7).
- **decision:** the repository holds only the pin; a build script fetches and verifies the kit into a gitignored folder, and the npm package includes it (Q8).
- **decision:** macOS and Linux first; Windows with Git Bash must work before 1.0, with a Windows CI job added in the foundations spec (Q10).
- **decision:** `.mcpb` bundle is backlog, after the MVP (Q9).
- **fact:** npm does not reliably keep file modes after install (only `bin` targets become executable), so the server runs `bash <bundle>/bin/jsdd` instead of executing it directly (npm/pacote #39, Researched).
- **fact:** users need Node.js (for `npx`) and bash (for `bin/jsdd`), plus whatever the kit itself requires (Briefing G2).

## Recommendation and decisions

<!-- Drafted when every goal question is answered, and agreed with the user. -->

| Decision | Choice | Alternatives considered | Why |
| --- | --- | --- | --- |
| DEC-12 Language and runtime (settles ND-01) | TypeScript on Node.js, with the official TypeScript MCP SDK v2 (`@modelcontextprotocol/server`). Released SDK versions only, pinned exactly; experimental APIs are allowed under that pin. | Python with `mcp` v2 | The owner prefers to maintain TypeScript. SDK 2.4.0 already ships Skills (`loadSkills` with a digest manifest) and form and URL Elicitation; Python's Skills is unreleased. Tasks are incomplete in both SDKs and are revisited when FR-LONG is planned. A documented Node requirement is acceptable (Q1–Q5). |
| DEC-13 Shipping and bundling (settles ND-03) | npm, unscoped `jsdd-mcp`, installed with `npx jsdd-mcp`, published from GitHub Actions with trusted publishing. The repository holds only the pin (kit version and the sha256 of the kit's release tarball); a build script fetches and verifies the kit into a gitignored folder, and the npm package includes it. The server runs `bash <bundle>/bin/jsdd`. Until the kit attaches a tarball, the pin is the tag plus its commit SHA, fetched with git. | PyPI with `uvx`; run from a git tag; vendor the kit in the repo; sha256 of GitHub's source archive | Kit releases attach no files and source archives have no stable checksum; an uploaded release file has a stable sha256. npm does not reliably keep the executable bit, so `bash` is called explicitly (Q6–Q8). |

## Deliverables

- [x] Every goal question answered, and the recommendation agreed with the user
- [x] BACKLOG.md items settled by this spike updated

## Follow-ups

<!-- New specs, BACKLOG items, experiments, or open questions this spike created. -->

- Kit release 1 gains one item: the kit's Release workflow attaches a bundle tarball (`skills/`, `bin/jsdd`, `templates/`, `starters/`) to each release (DEC-13). Added to the PLAN.md release 1 item.
- Windows with Git Bash before 1.0, with a Windows CI job in the foundations spec (Q10). Noted on the PLAN.md repository layout item.
- `.mcpb` bundle after the MVP (Q9). Added to BACKLOG.md "Next".
- ND-04 stays open for the foundations spec; it can build on the DEC-13 pin (version plus sha256). Noted on its BACKLOG.md line.
- Things to try, each placed or dropped at close:
  - Throwaway TypeScript server with `loadSkills` on a copy of the kit's `skills/` plus a tool running `bash bin/jsdd version --json`, checked with MCP Inspector's skill verifier → attached as a check to the PLAN.md repository layout item.
  - `npm pack` with the bundle inside, install, inspect the mode of `bin/jsdd` → attached as a check to the PLAN.md repository layout item.
  - Form elicitation round trip (`input_required`) against MCP Inspector, for FR-WORK-05 and FR-SPEC-02 → BACKLOG.md "Next", with the takeover item.
  - Python prototype and `hatch_build.py` wheel test → dropped: DEC-12 chose TypeScript.
- Check whether `bin/jsdd` needs `jq` and `python3` (kit README) or only bash 3.2 + awk (its header), and align the README → attached as a check to the PLAN.md kit release 1 item.

## Changelog

- 2026-11-27 Created.
- 2026-11-27 Briefing and 10 questions prepared; in progress.
- 2026-11-27 All questions answered. Recommendation agreed with the user and recorded as DEC-12 and DEC-13.
- 2026-11-27 Verified and closed. Follow-ups placed: kit tarball and README check on PLAN.md release 1, Windows CI and prototype checks on the repository layout item, `.mcpb` and an elicitation round trip in BACKLOG.md "Next"; Python experiments dropped.
