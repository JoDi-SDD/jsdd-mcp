# Requirements

> The product-level list of what the product must do. Specs in `specs/` make these rows testable and link back by ID; progress comes from `jsdd coverage`, never from a status column here.

**Format.** `jsdd coverage` reads every table row whose first cell is an ID like `FR-AUTH-01` or `NFR-SEC-01`. Functional rows have four cells (ID, requirement, source, priority); the priority is `M` Must, `S` Should, `C` Could, or `W` Won't. NFR rows may leave out source and priority. IDs never change once a spec cites them: reword a row in place, or add a new one (`FR-AUTH-01b`) for a split.

## 1. Functional requirements

### 1.1 Kit pin and startup

| ID | Requirement | Source | Pri |
|---|---|---|---|
| FR-KIT-01 | The server package bundles the pinned kit release's `skills/`, `bin/jsdd`, `templates/`, and `starters/`. | docs/legacy/brief.md D3; DEC-11 | M |
| FR-KIT-02 | At startup the server reads the kit version from `bin/jsdd --json` and refuses to start, with a clear message, when it is outside the supported range. | docs/legacy/brief.md D3, NFR-5 | M |

### 1.2 Skills

| ID | Requirement | Source | Pri |
|---|---|---|---|
| FR-SKL-01 | The server declares the `io.modelcontextprotocol/skills` extension and implements `skills/list` and `skills/get` for the bundled skills. | docs/legacy/brief.md FR-1 | M |
| FR-SKL-02 | Each skill manifest lists every file with a SHA-256 digest and byte size computed from the bytes served. | docs/legacy/brief.md FR-2 | M |
| FR-SKL-03 | Every file of a bundled skill can be read through `resources/read`. | docs/legacy/brief.md §1 | M |

### 1.3 Status and coverage

| ID | Requirement | Source | Pri |
|---|---|---|---|
| FR-STAT-01 | `get_status` returns the project's spec board as `bin/jsdd` reports it. | docs/legacy/brief.md §6, §9 | M |
| FR-STAT-02 | `get_coverage` returns requirement coverage as `bin/jsdd coverage` computes it. | docs/legacy/brief.md §6, A6 | M |

### 1.4 Claims and ticking

| ID | Requirement | Source | Pri |
|---|---|---|---|
| FR-WORK-01 | `claim_task` is atomic: two concurrent claims on the same task produce exactly one winner. | docs/legacy/brief.md FR-4 | M |
| FR-WORK-02 | Claims made through the server and claims made with `bin/jsdd` directly are visible to each other. | docs/legacy/brief.md FR-5 | M |
| FR-WORK-03 | `tick_task` succeeds only when the task's check passes, and otherwise returns the failing check's output. | docs/legacy/brief.md FR-3 | M |
| FR-WORK-04 | Each claim is a lease with an owner and a heartbeat; the owner's tool calls refresh it, and it becomes stale after its TTL. | docs/legacy/brief.md §7 | S |
| FR-WORK-05 | `release_claim` releases the caller's own claim; taking over another session's stale claim asks the user through Elicitation first. | docs/legacy/brief.md §7 | S |
| FR-WORK-06 | When stdin closes, the server releases its own claims on a best-effort basis. | docs/legacy/brief.md §7 | C |

### 1.5 Resources

| ID | Requirement | Source | Pri |
|---|---|---|---|
| FR-RES-01 | The server declares the `resources` capability and exposes the project's intent, requirements, decisions, backlog, and each spec's spec, plan, tasks, and verify report under `jsdd://` URIs. | docs/legacy/brief.md FR-8, §6 | S |

### 1.6 Specs, plans, and decisions

| ID | Requirement | Source | Pri |
|---|---|---|---|
| FR-SPEC-01 | `create_spec` creates a feature spec and assigns its `FR-`/`NFR-` IDs. | docs/legacy/brief.md §6 | S |
| FR-SPEC-02 | `submit_plan` and `amend_spec` write nothing until the user approves through Elicitation. | docs/legacy/brief.md FR-6 | S |
| FR-SPEC-03 | `record_decision` adds a decision to `docs/decisions.md`. | docs/legacy/brief.md §6 | S |
| FR-SPEC-04 | `scaffold_project` sets up a project for the kit, creating its starter files. | docs/legacy/brief.md §6 | S |

### 1.7 Long runs

| ID | Requirement | Source | Pri |
|---|---|---|---|
| FR-LONG-01 | `run_verification` runs as a Task and writes the spec's `verify-report.md`. | docs/legacy/brief.md FR-7 | S |
| FR-LONG-02 | `run_spec_to_pr` takes a spec to a pull request as a Task, pausing for the user through Elicitation. | docs/legacy/brief.md §6 | C |

## 2. Non-functional requirements

### 2.1 State

| ID | Requirement |
|---|---|
| NFR-STATE-01 | A server process killed at any point loses no claim and no state another session needs. |

### 2.2 Security

| ID | Requirement |
|---|---|
| NFR-SEC-01 | `resources/read` and `skills/*` reject any URI that resolves outside the allowed paths. |

### 2.3 Portability

| ID | Requirement |
|---|---|
| NFR-PORT-01 | The server runs on macOS, Linux, and Windows with Git Bash. |

## 3. Open questions

Open decisions are in [decisions.md](decisions.md) "Still open"; research questions are spikes in [BACKLOG.md](../BACKLOG.md).
