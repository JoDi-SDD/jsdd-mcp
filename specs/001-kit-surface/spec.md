---
id: 001
title: "Kit surface"
type: spike
status: done
phase: 0
depends_on: []
refs:
  - "BACKLOG.md Spikes: Kit surface"
  - "PLAN.md Phase 0 Spikes: Kit surface · ND-06"
updated: 2026-11-27
---

# 001 · Kit surface

<!--
A spike answers questions and settles decisions; it does not ship production code. Run it with /jsdd:spike 001:
the spike-researcher agent prepares a briefing and questions, you answer and add information over as many sessions
as it takes, and everything learned is recorded here. Prototypes are throwaway and live under specs/001-*/prototype/.
Progress counts the checkboxes in Goal questions, Questions for you, and Deliverables.
-->

## Inbox

<!-- Drop notes, links, constraints, or decisions here at any time, in any form. The next /jsdd:spike session moves them into the Log and Findings and clears this section. -->

## Why

Settles **ND-06** (claim store) and lists the kit changes **DEC-06** needs before Milestone 1: JSON output for coverage, claiming, and ticking; atomic claims in the CLI; client-neutral skill text. Those changes become the Phase 0 foundations item "Kit release with the DEC-06 changes". Reads the kit at `JoDi-SDD/JoDi-SDD` (docs/legacy/brief.md A1–A4, A6), through its public surface and docs.

## Goal questions

<!-- What the spike must answer, from BACKLOG.md. Ticked once Findings answers it completely. -->

- [x] **G1** How does `bin/jsdd` store and take claims today (location, atomicity, staleness, worktrees vs. remote), and what must change for FR-WORK-01 and FR-WORK-02?
- [x] **G2** What do `bin/jsdd --json`, `coverage`, `claim`, and `done` expose today, and what output does the kit need to add for FR-STAT and FR-WORK-03?
- [x] **G3** Do the kit's `skills/` follow the Agent Skills format, and which text names Claude Code-only features and needs rewording?

## Briefing

<!-- Prepared by the spike-researcher agent: what the repository and current docs already say, with sources. -->

`K/` = the installed kit release 0.8.1 at `~/.claude/plugins/cache/jodi-sdd/jsdd/0.8.1`. Its behaviour matches the latest release, v0.8.3, because 0.8.2 and 0.8.3 are `[meta]` only. The local clone `JoDi-SDD/JoDi-SDD` is stale: a 0.2.0-era branch is checked out, so it was not used.

### G1 · Claim store and atomicity

- Local mode: each claim is a directory under `$(git rev-parse --git-common-dir)/sdd-claims/`, so every worktree of a clone shares it. Outside git it is `specs/.claims` (K/bin/jsdd:22-28).
- Layout: `NNN/` (whole spec), `NNN-Tn/` (task, plus a `done` marker once ticked), `id-NNN/` (a reserved spec ID). Each holds an `owner` file with `owner=`, `since=`, `at=`, and `branch=` once done (K/bin/jsdd:399-417).
- Local atomicity comes from `mkdir`, which fails if the directory exists (K/bin/jsdd:554-561).
- Remote mode (`git config jsdd.claims remote`, `claims: remote` in specs/jsdd.yml, or `SDD_CLAIMS=remote`): each claim is the ref `refs/sdd-claims/<name>` on `jsdd.claimsRemote`, default origin (K/bin/jsdd:62-73, 527-533).
- Remote atomicity: a push with `--force-with-lease=<ref>:` and an empty lease, which the remote accepts only if the ref does not exist. Release and done are leased pushes (K/bin/jsdd:543-598).
- Remote reads reuse a fetch younger than 30 s. Writes always fetch first and exit 7 when the remote is unreachable (K/bin/jsdd:472-502).
- Owner ID: `$JSDD_SESSION_ID`, else `$CLAUDE_CODE_SESSION_ID`, else `user@host` (K/bin/jsdd:76).
- Staleness is only a "(stale)" label, shown once a claim is older than `SDD_STALE_HOURS` (default 8). There is no heartbeat, TTL, or automatic takeover; `release --force` takes another session's claim (K/bin/jsdd:77, 602-606, 964-983).
- Ticking is enforced only through `jsdd done`:
  - The task claim must be held by this owner.
  - The check must match `checks.lock`.
  - The check runs with `bash -c` from the repo root, and the box is ticked only on exit 0.
  - A manual check needs `--manual "<note>"`.
  - Source: K/bin/jsdd:1066-1161.
- FR-WORK-01 is already met: concurrent claims on the same task have exactly one winner in both modes. FR-WORK-02 holds by construction when both channels call the same `bin/jsdd` with the same mode and repo.
- **Gap:** a spec claim and a task claim racing each other can both win. Each checks for the other's claim, then takes its own, so the check and the take are separate steps (K/bin/jsdd:707-726).
- **Gap:** the `owner` file is written after `mkdir`, so a reader can briefly see a claim with no owner (K/bin/jsdd:559-560).
- The brief's `refs/jsdd/claims/<spec>` idea (docs/legacy/brief.md §7) differs from the kit's actual `refs/sdd-claims/<name>`.

### G2 · Machine output

- `--json` (or `JSDD_OUTPUT=json`) exists only for next, status, mode, charter, profile, paths, auto queue/dry-run/doctor/run, and run status/result. Any other command given `--json` exits 2 (K/docs/machine-output.md; confirmed: `jsdd coverage --json` exits 2).
- Envelope: `{schema, jsdd, ok, exit, data, warnings, errors}`. Schemas are named `jsdd.<cmd>/<n>`, and a schema only gains fields within one `n`. Exit codes: 0 ok, 1 failed, 2 usage, 3 nothing to do, 4 parked, 5 budget, 6 claim_lost, 7 remote_unreachable.
- The version is the envelope's `jsdd` field. It is read from `$PLUGIN_DIR/.claude-plugin/plugin.json`, a plugin-only file (K/bin/jsdd:786, confirmed). There is no `version` command, and a bare `jsdd --json` exits 2 (confirmed). `paths --json` is the only JSON command that runs without `specs/`.
- `jsdd.status/1` lists specs with their claim, but:
  - `claim` is a display string ("by … since … (stale)").
  - There are no per-task claims.
  - There is no coverage.
  - `next` holds slash commands such as `/jsdd:plan 003`, which would break P4 if passed through (K/bin/jsdd:910-939, 3027-3064).
- `coverage` prints text only. The row data (id, priority, specs, state, text) exists internally (K/bin/jsdd:1283-1349).
- `claim`, `release` and `claims` print text only. `claim` exits 6 when another owner holds the claim (K/bin/jsdd:707-767, 964-983, 1230-1249).
- `done` prints text only:
  - The check's stdout and stderr pass straight through, mixed with jsdd's own lines.
  - A failure prints "check failed (exit N)" and exits 1.
  - There is no timeout on the check.
  - It suggests `/jsdd:verify` (K/bin/jsdd:1122-1159).
- JSON event lines for claim, release, and task.done exist with `JSDD_EVENTS`.
- **New blocker for FR-KIT-01/02:**
  - `bin/jsdd` needs `.claude-plugin/plugin.json` for its version, and `templates/` and `starters/` for `new` and `init`.
  - A bundle of only `skills/` and `bin/jsdd` would report an empty version.

### G3 · Agent Skills conformance and client-specific text

- All 9 skills have a `name` that matches the folder and a `description` under 1024 characters.
- Every skill also has `argument-hint`, which the reference validator, skills-ref, rejects. It allows only name, description, license, compatibility, metadata, and allowed-tools. `init`'s value is not valid YAML, and `status`'s parses as a list.
- The MCP Skills extension (SEP-2640, Final) requires servers to serve valid Agent Skills and to preserve all frontmatter. Hosts read only the files listed in the skill's manifest.
- Client-specific text in K/skills:
  - `$ARGUMENTS`: line 9 of all 9 skills, plus plan:32, verify:32, baseline:35, init:44.
  - `${CLAUDE_PLUGIN_ROOT}/templates|starters`: auto:91, 135; init:29; plan:49; verify:54, 90. These files sit outside the skill folders, so a manifest cannot include them.
  - AskUserQuestion: auto:32, 49; baseline:43; init:16, 17, 43; specify:66; spike:58.
  - Plugin-only agents (sdd-reviewer, sdd-product, sdd-surveyor, spike-researcher) are launched from auto, baseline, implement, init, plan, specify, spike, and verify.
  - Hooks and session start are named at line 11 of most skills, auto:32, 40, 42, and init:20, 34.
  - `/jsdd:*` slash commands appear in 5 descriptions and in about 100 body lines.
  - Plugin and `.claude` configuration is named at auto:74, 80, 92 and init:3, 19-21, 29, 65.
  - Skills run `jsdd …` from a shell, which assumes `bin/` is on PATH.
- `bin/jsdd`'s own output names slash commands and Claude Code (K/bin/jsdd:75-76, 239-240, 736, 910-939, 1159, 2261-2273).
- The kit roadmap already plans to generate other clients' agent files from `agents/*.md` at release (K/ROADMAP.md §10).

### Draft list of DEC-06 kit changes

1. A core version source, plus `jsdd version --json` that runs without `specs/`.
2. `coverage --json` (`jsdd.coverage/1`): rows plus a per-priority summary, honouring `--phase`, `--priority` and `--missing`.
3. `claim`, `release` and `claims --json`, with structured holder, staleness, waits-on, and mode.
4. `done --json`: captured check output with its size, exit code, ticked, all_done, and the next action. `done` must capture the output instead of passing it through.
5. Additive changes to `status/1` and `next/1`: a structured claim object, per-task claims, a coverage summary, and a neutral `action: {skill, spec}`.
6. Claims: make spec claims and task claims exclusive atomically, and write the owner before the claim is visible.
7. Skills: fix the frontmatter, and reword the client-specific text. Templates and agent prompts must be reachable from the core.
8. Docs (`docs/machine-output.md`), scenario tests, and a minor release.

### Sources

- K/bin/jsdd, K/docs/machine-output.md, K/skills/*, K/CHANGELOG.md, K/ROADMAP.md (0.8.1)
- https://github.com/JoDi-SDD/JoDi-SDD/releases (v0.8.3, 2026-11-27)
- https://agentskills.io/specification
- skills-ref `validator.py` (github.com/agentskills/agentskills)
- https://modelcontextprotocol.io/extensions/skills/overview (SEP-2640)

## Questions for you

<!-- Prepared by the agent, highest impact first; each says which goal questions it serves, why it matters, the options, and a suggested answer. Ticked when answered or skipped. -->

- [x] **Q1** (G2, G3) Should DEC-06 ship as **one kit release, or two**? Two would put the machine output first, which blocks the status, coverage, claim and tick tools, and the client-neutral skills second, which block the Skills item. *Suggested:* two releases, with PLAN.md amended. The JSON is small and additive, and the skill rewording is large.
- [x] **Q2** (G1) Should ND-06 settle as **"bin/jsdd's existing store, local or remote as the project configures, with no new store"**? *Options:* as-is / a new store / leave it open. *Suggested:* as-is.
- [x] **Q3** (G1) Should the kit release also close the **race between a spec claim and a task claim**? *Options:* fix now / backlog it. *Suggested:* fix now. It is small and in the same code.
- [x] **Q4** (G1) Which **owner ID** should a server session use? *Options:* a new UUID per process as `JSDD_SESSION_ID` / an ID the client supplies / `user@host`. *Suggested:* a UUID per process for Milestone 1, accepting orphaned claims until FR-WORK-04/05.
- [x] **Q5** (G2) Where should the **kit version** live, so a bundle without `.claude-plugin/` can report it? *Options:* a VERSION file / stamped into bin/jsdd at release / a `jsdd version --json` command reading either. *Suggested:* stamped into bin/jsdd, plus `jsdd version --json`.
- [x] **Q6** (G2) Should the server's tick tool **accept manual checks**? *Options:* refuse in Milestone 1 / allow with a note / allow after an Elicitation approval (later). *Suggested:* refuse for now and report "needs a person".
- [x] **Q7** (G3) How do skills and `bin/jsdd` **reach `templates/` and `starters/`** in a bundle? *Options:* copy them into each skill's `assets/` / a `jsdd template <name>` command / bundle templates/ and starters/ and amend FR-KIT-01. *Suggested:* a `jsdd template` command, plus bundling them with FR-KIT-01 amended.
- [x] **Q8** (G3) Where should the **agent prompts** (reviewer, product, surveyor, researcher) live? *Options:* in the core as `skills/<name>/references/*.md`, with the plugin's agents generated from them / kept in the plugin, with neutral fallback text / as separate skills. *Suggested:* core reference files, with generated agents.
- [x] **Q9** (G3) Should skill steps keep naming **`jsdd <command>`**, with the server mapping those names to its tools? *Options:* keep the CLI vocabulary / reword to abstract operations. *Suggested:* keep it, because the CLI is the core's public surface.
- [x] **Q10** (G2, G3) How should **next steps** be named in the core instead of `/jsdd:<skill>`? *Options:* add a neutral `action: {skill, spec}` to the JSON / reword the text everywhere / leave it. *Suggested:* add `action` now and reword the skill text in the skills release.
- [x] **Q11** (G3) What should happen to **`argument-hint`**? *Options:* drop it / move it under `metadata:` / keep it and accept non-conformance. *Suggested:* move it under `metadata`, quoted.
- [x] **Q12** (G2) Should the tick result carry **all of a failing check's output, or a capped tail**? *Options:* full / a tail plus the total size / a tail plus a log path. *Suggested:* a tail (for example 200 lines or 64 KB) plus the total size.

## Log

<!-- Append-only, oldest first: "- YYYY-MM-DD · <source> · <what was answered, added, or found>". -->

- 2026-11-27 · research · spike-researcher read kit release 0.8.1 (the local clone is stale). FR-WORK-01 is met by today's CLI. Gaps found: the spec/task claim race, the version read from a plugin-only file, and no JSON for coverage, claim, release, claims or done. Spot-checked in a shell: `coverage --json` and bare `--json` exit 2, and the version comes from `.claude-plugin/plugin.json`.
- 2026-11-27 · answers to Q1, Q2, Q3, Q5 · Q1: two kit releases, machine output first and client-neutral skills second (PLAN.md to be amended). Q2: ND-06 settles as bin/jsdd's existing store, local or remote as configured, with no new store. Q3: fix the spec/task claim race in the kit release, and write the owner before the claim is visible. Q5: stamp the version into bin/jsdd at release and add `jsdd version --json`, runnable without specs/.
- 2026-11-27 · answers to Q4, Q6, Q7, Q8 · Q4: the server uses a new UUID per process as `JSDD_SESSION_ID`, accepting orphaned claims (stale after 8 h, `release --force`) until FR-WORK-04/05. Q6: the tick tool refuses manual checks in Milestone 1 and reports "needs a person"; revisit with Elicitation after the MVP. Q7: add `jsdd template <name>`, bundle `templates/` and `starters/` too, and amend FR-KIT-01. Q8: move the agent prompts into the core as `skills/<name>/references/*.md` ("run as a sub-task if your client can, else inline") and generate the plugin's agents from them at release.
- 2026-11-27 · answers to Q9, Q10, Q11, Q12 · Q9: skills keep `jsdd <command>` as their vocabulary, and the server maps those commands to its tools. Q10: add a neutral `action: {skill, spec}` to the status and next JSON in release 1 (additive, stays on /1), and reword the skill and CLI text in release 2. Q11: move `argument-hint` under `metadata:`, quoted, and check that Claude Code still shows the hints. Q12: a failing tick returns a tail of the check's output (for example the last 200 lines or 64 KB) plus the total size.
- 2026-11-27 · user · agreed to the recommendation as drafted (DEC-08 to DEC-11) and its follow-ups.

## Findings

<!-- Per goal question. Label each point as a decision (by the user), a fact (with source), or an assumption (to confirm). -->

### G1

- **fact:** claims are directories under `<git-common-dir>/sdd-claims/` (local, `mkdir`-atomic) or refs `refs/sdd-claims/<name>` pushed with an empty lease (remote). All worktrees of a clone share them (K/bin/jsdd:22-28, 527-598).
- **fact:** FR-WORK-01 (same-task atomicity) and FR-WORK-02 (visibility across channels) already hold when both channels call the same bin/jsdd.
- **decision:** ND-06: use bin/jsdd's existing store, local or remote as the project configures, with no new store (Q2).
- **decision:** the kit release fixes the race between a spec claim and a task claim, and writes the owner before the claim is visible (Q3).
- **decision:** a server session uses a new UUID per process as `JSDD_SESSION_ID`. Orphaned claims go stale after 8 h or need `release --force`, until leases arrive (FR-WORK-04/05) (Q4).

### G2

- **fact:** JSON exists for status, next, paths and a few others, but not for coverage, claim, release, claims or done. The version comes from the plugin-only `.claude-plugin/plugin.json` (K/bin/jsdd:786).
- **decision:** two kit releases. The first carries the machine output: the version, `coverage`, `claim`, `release`, `claims` and `done` JSON, and additive `status`/`next` fields. The second carries the client-neutral skills (Q1).
- **decision:** the version is stamped into bin/jsdd at release, and `jsdd version --json` runs without specs/ (Q5).
- **decision:** the server's tick tool refuses manual checks in Milestone 1 and reports "needs a person" (Q6). This is a server rule, so no kit change is needed.
- **decision:** status and next JSON gain a neutral `action: {skill, spec}` next to `next` in release 1 (Q10).
- **decision:** `done --json` captures the check's output and returns a tail (for example the last 200 lines or 64 KB) plus the total size (Q12).

### G3

- **fact:** all skills carry `argument-hint`, which the Agent Skills validator rejects. The skill text names client-only features ($ARGUMENTS, CLAUDE_PLUGIN_ROOT, AskUserQuestion, plugin agents, hooks, /jsdd:*). See Briefing.
- **decision:** add `jsdd template <name>`, bundle `templates/` and `starters/` with the server, and amend FR-KIT-01 (Q7).
- **decision:** the agent prompts move into the core as `skills/<name>/references/*.md`, and the plugin's agents are generated from them at release (Q8).
- **decision:** skills keep naming `jsdd <command>`, and the server maps those commands to its tools (Q9).
- **decision:** `argument-hint` moves under `metadata:`, quoted (Q11).
- **decision:** release 2 rewords `$ARGUMENTS`, `${CLAUDE_PLUGIN_ROOT}`, AskUserQuestion, hooks and session start, `/jsdd:*`, and the plugin configuration text to client-neutral wording.

## Recommendation and decisions

<!-- Drafted when every goal question is answered, and agreed with the user. -->

| Decision | Choice | Alternatives considered | Why |
| --- | --- | --- | --- |
| DEC-08 Claim store (settles ND-06) | Use bin/jsdd's existing store: `<git-common-dir>/sdd-claims/` locally, `refs/sdd-claims/*` on the remote in remote mode. No new store. | A new store such as `refs/jsdd/claims/` (brief §7); leave it open | It is already atomic per task in both modes, and every channel calling bin/jsdd sees the same claims (Q2). |
| DEC-09 DEC-06 ships as two kit releases | **Release 1, machine output:** the version stamped into bin/jsdd plus `jsdd version --json`, runnable without specs/; JSON for `coverage`, `claim`, `release`, `claims` and `done`, where a failing check returns a tail of its output plus the total size; additive `status`/`next` fields (a structured claim, per-task claims, a coverage summary, `action: {skill, spec}`); spec claims and task claims made atomically exclusive, with the owner written before the claim is visible; `jsdd template <name>`. **Release 2, client-neutral skills:** agent prompts moved into the core as `skills/<name>/references/*.md`, with the plugin's agents generated from them; `argument-hint` moved under `metadata`; `$ARGUMENTS`, `${CLAUDE_PLUGIN_ROOT}`, AskUserQuestion, hooks and session start, `/jsdd:*` and plugin configuration reworded; skills keep the `jsdd <command>` vocabulary. | One release | Release 1 is small and additive, and it unblocks the Milestone 1 tools. The skill rewording is large (Q1, Q3, Q5, Q7–Q12). |
| DEC-10 Server session rules | Each server process sets a new UUID as `JSDD_SESSION_ID`. The tick tool refuses manual checks and reports "needs a person". | A client-supplied ID; `user@host`; allowing manual ticks with a note | Orphaned claims after a restart are acceptable until leases (FR-WORK-04/05). Manual ticks would let the agent tick its own box (Q4, Q6). |
| DEC-11 Bundle contents | The server bundles `skills/`, `bin/jsdd`, `templates/` and `starters/` from the pinned kit release. FR-KIT-01 is amended to match. | Copying templates into each skill's `assets/`; bundling them with no `template` command | bin/jsdd needs templates/ and starters/ for `new` and `init`, and this keeps one copy (Q7). |

## Deliverables

- [x] Every goal question answered, and the recommendation agreed with the user
- [x] BACKLOG.md items settled by this spike updated

## Follow-ups

<!-- New specs, BACKLOG items, experiments, or open questions this spike created. -->

- Amend FR-KIT-01 to add `templates/` and `starters/` (DEC-11). Done in `docs/requirements.md` at close.
- Split the PLAN.md kit release item into release 1 (machine output) and release 2 (client-neutral skills) (DEC-09). Done in PLAN.md at close; the Phase 0 exit criteria now need release 1, and Phase 1's skills item needs release 2.
- ND-04 can build on `jsdd version --json`. Noted on its BACKLOG.md line.
- Start the kit work from an up-to-date main in the local clone, which is on a 0.2.0-era branch. Noted on the PLAN.md release 1 item.
- Things to try, each attached as a check to its PLAN.md item:
  - Race test between a spec claim and a task claim, local and remote → release 1.
  - Bundle-only test (version and `new` from the copied folders) → release 1.
  - `skills-ref validate` on every skill → release 2.
  - `argument-hint` under `metadata` in Claude Code → release 2.

## Changelog

- 2026-11-27 Created.
- 2026-11-27 Briefing and 12 questions prepared; in progress.
- 2026-11-27 All questions answered. Recommendation agreed with the user and recorded as DEC-08 to DEC-11.
- 2026-11-27 Verified and closed. Follow-ups placed: FR-KIT-01 amended, PLAN.md kit release item split into release 1 and release 2, experiments attached as checks.
