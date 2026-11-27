# Brief: JoDi-SDD MCP server (`jsdd-mcp`)

> **How to use this file:** this is the starting design for this repository. Use it as
> input for `/jsdd:spike` (to check the assumptions in §4 against the kit's code in
> `JoDi-SDD/JoDi-SDD`), followed by `/jsdd:specify`. Once specs exist, they take over
> from this file. Don't start implementing from it directly.
>
> **Updated 2026-11-27** with the decisions in §3a. Where an older section disagrees,
> §3a wins.

---

## 1. Context

JoDi-SDD is a Claude Code plugin for spec-driven development. Its parts are `skills/`
(the `/jsdd:` commands), `agents/`, `hooks/`, the `bin/jsdd` bash CLI, and Markdown
files in the user's repo.

It works **only in Claude Code**. Its rules (for example, Claude can't tick its own
boxes, and commits must cite a spec) are enforced by hooks and instructions that
run on the user's machine.

The MCP 2026-07-28 spec plus its official extensions let a server ship:

- **Skills** (`io.modelcontextprotocol/skills`): playbooks served through `skills/list`, `skills/get`, `resources/read`
- **Tools**: actions with server-side enforcement
- **Resources**: read-only context through URIs
- **Elicitation**: the server pauses a call to ask the user (`resultType: "input_required"`, and the client retries with `inputResponses`)
- **Tasks** (`io.modelcontextprotocol/tasks`): long-running work, polled with `tasks/get`

## 2. Goal

Add an MCP server as a **second distribution channel** for JoDi-SDD that:

1. Serves the existing `skills/` content to any MCP client that supports Skills
2. Moves the rules into tools, so the model can change SDD state only through
   checks the server enforces
3. Uses Elicitation for the approval steps and Tasks for long runs

## 3. Non-goals

- Replacing the Claude Code plugin. Both coexist.
- Porting hooks. MCP has no client-side hooks, so commit and session-start checks stay in the plugin.
- Porting agents as subagents. They collapse into skills plus checklists, or into tools.
- A remote or hosted server. v1 is **local stdio only**.

## 3a. Decisions so far

- **D1 Channels.** The Claude Code plugin and the MCP server are channels: peers that
  deliver the same product to different clients. The shared **core** is `skills/`,
  `bin/jsdd` (the rules), `templates/`, `starters/`, and the Markdown file formats. It
  lives in the kit repo, next to the Claude Code channel (`hooks/`, `agents/`,
  `.claude-plugin/`).
- **D2 Own repo** (answers Q2). The server lives in `JoDi-SDD/jsdd-mcp`, not in the kit
  repo, so server-only releases never reach plugin users. It follows the pattern
  `sdd-runtime` already uses: pin a kit release and talk to it only through
  `bin/jsdd --json`.
- **D3 Own version, pinned kit** (answers Q3). The server has its own version and
  release. It bundles the pinned kit release's `skills/` and `bin/jsdd` in its package,
  and checks at startup that the kit version (from `bin/jsdd --json`) is one it supports.
  Moving to a new kit release is a pin bump in the server.
- **D4 No auto-registration** (answers Q4). The plugin never registers the server.
  Installing it stays optional, so plugin users never need its runtime.
- **D5 Boundary rules.**
  1. The core never refers to a channel. Skill text stays client-neutral (see A4).
  2. Channels never refer to each other.
  3. Rules have one implementation, in `bin/jsdd`. The server calls it and never
     reimplements it.
  4. The server uses the kit only through `bin/jsdd --json` and the shipped `skills/`,
     never its internals. Anything missing is a kit change first, released, then a pin
     bump.
- **D6 Kit changes first.** Milestone 1 needs kit releases before server work: JSON output
  for `coverage`, claiming, and ticking (A2, A6); atomic claims in the CLI (A1, FR-4,
  FR-5); client-neutral skill text (A4).

**What would reopen D2:** the spike finding that the server needs kit internals beyond
`bin/jsdd --json` and the shipped `skills/`.

## 4. Assumptions to check first (spike)

Before specifying anything, read the code and confirm or correct each of these:

- [ ] A1: How `bin/jsdd` implements claims today (storage location, atomicity, staleness/TTL, worktree vs. remote mode)
- [ ] A2: How a task's "check" is defined and run, and what "check passed" means in code
- [ ] A3: Whether `skills/*/SKILL.md` already follow the Agent Skills format (frontmatter `name` matching the folder name, plus `description`)
- [ ] A4: Which `skills/` content references Claude Code-only features (slash commands, hooks, subagents) and would need rewording to be client-neutral
- [x] A5: Whether the private `sdd-runtime` repo overlaps with this work (shared core, or a place to put it). **Answered:** no overlap. The runtime has no shared core library; it pins a kit version in its image (`JSDD_VERSION`) and drives it only through `bin/jsdd --json`. It's the model for D2 and D3, and the version check at startup could be a convention both share.
- [ ] A6: How `jsdd coverage` computes its numbers, so `get_coverage` can reuse the logic and not reimplement it

Record the findings in `decisions.md`.

## 5. Architecture

```
Session A ──stdio──> jsdd-mcp (process A) ─┐
                                            ├──> shared state: repo Markdown files + claim store
Session B ──stdio──> jsdd-mcp (process B) ─┘
```

- **The MCP processes are stateless.** One stdio process serves one session and is
  disposable. All state lives in files or git refs that every session can see.
- **Single source of rules.** Enforcement logic lives in `bin/jsdd`, and the MCP tools
  call the bundled, pinned copy (D3, D5).

### Proposed layout (adjust after the spike)

```
jsdd-mcp/                            # repo JoDi-SDD/jsdd-mcp (D2)
├── src/
│   ├── index.ts                     # stdio entry point
│   ├── server.ts                    # capabilities: tools, resources, skills ext, tasks ext
│   ├── session.ts                   # SESSION_ID = hostname-pid-shortuuid
│   ├── skills/                      # loader, manifest (sha256 + size), handlers
│   ├── resources/                   # jsdd:// URI → file mapping
│   ├── tools/
│   │   ├── read/                    # get_status, get_coverage
│   │   ├── specs/                   # create_spec, submit_plan*, amend_spec*
│   │   ├── work/                    # claim_task, tick_task, release_claim*
│   │   ├── decisions/               # record_decision
│   │   ├── project/                 # scaffold_project
│   │   └── long/                    # run_verification†, run_spec_to_pr†*
│   └── rules/                       # adapter to the bundled bin/jsdd
└── test/
# * = uses Elicitation   † = runs as a Task
```

The kit's `skills/` is **reused as-is** (after the A3/A4 fixes in the kit): the server
bundles the pinned release's copy at build time and never edits or forks it.

## 6. Mapping: plugin → MCP

| Plugin today | MCP | Notes |
|---|---|---|
| `/jsdd:init` | skill `init` + tool `scaffold_project` | Interview = guidance; file creation = tool |
| `/jsdd:specify` | skill `specify` + tool `create_spec` | Tool assigns `FR-`/`NFR-` IDs |
| `/jsdd:plan` + reviewer agent | skill `plan` (review checklist in `references/`) + tool `submit_plan` | `submit_plan` elicits approval |
| `/jsdd:implement` | skill `implement` + tools `claim_task`, `tick_task` | `tick_task` refuses unless the check passes |
| `/jsdd:verify` | skill `verify` + tool `run_verification` (Task) | Writes `verify-report.md` |
| `/jsdd:status` | tool `get_status` | Read-only aggregate |
| `jsdd coverage` | tool `get_coverage` | Reuse CLI logic (A6) |
| `/jsdd:spike` | skill `spike` + tool `record_decision` | Appends to `decisions.md` |
| `/jsdd:baseline` | skill `baseline` | Writes through tools |
| `/jsdd:auto` | tool `run_spec_to_pr` (Task + Elicitation) | Autopilot pauses at each step; autonomous mode asks only before the PR |
| hooks | **stay in plugin** | No MCP equivalent |

### Resources

```
jsdd://intent                     docs/intent.md
jsdd://requirements               docs/requirements.md
jsdd://decisions                  docs/decisions.md
jsdd://backlog                    BACKLOG.md
jsdd://specs/{id}/spec            specs/{id}/spec.md
jsdd://specs/{id}/plan            specs/{id}/plan.md
jsdd://specs/{id}/tasks           specs/{id}/tasks.md
jsdd://specs/{id}/verify-report   specs/{id}/verify-report.md
```

## 7. Claims design (stateless stdio)

- **Identity:** each process generates `SESSION_ID` at startup. One process is one session.
- **Storage:**
  - Same machine and worktrees: the claim files go under `git rev-parse --git-common-dir`,
    created with exclusive-create (`O_EXCL` / `flag: "wx"`) so a race has exactly one winner
  - Across machines: git refs on the remote (for example `refs/jsdd/claims/<spec>`), with
    the push acting as compare-and-swap
  - **Prefer whatever `bin/jsdd` already does (A1).** Plugin and MCP sessions must see the same claims.
- **Leases:** each claim holds `owner`, `claimedAt`, `heartbeatAt`, `ttlMinutes`.
  Every tool call from the owner refreshes the heartbeat. A claim is stale when it's
  older than the TTL.
- **Takeover:** `release_claim` on someone else's stale claim → Elicitation:
  "Claim on 012 last seen 2h ago. Take it over?"
- **Shutdown:** release own claims when stdin closes, but treat it as best-effort
  and never rely on it.

## 8. Candidate requirements (refine in `/jsdd:specify`)

**Functional**
- FR-1: The server declares the `resources` capability and the `io.modelcontextprotocol/skills` extension, and implements `skills/list` and `skills/get`
- FR-2: Skill manifests list every file with a SHA-256 digest and byte size computed from the bytes served
- FR-3: `tick_task` succeeds only when the task's check passes, and otherwise returns the failing check's output
- FR-4: `claim_task` is atomic: two concurrent claims on the same task produce exactly one winner
- FR-5: Claims made through MCP and through the plugin/CLI are visible to each other
- FR-6: `submit_plan` and `amend_spec` require user approval through Elicitation before writing
- FR-7: `run_verification` runs as a Task and writes `verify-report.md`
- FR-8: Resources expose the documents listed in §6

**Non-functional**
- NFR-1: The MCP process keeps no state needed across sessions (it can be killed at any time without losing claims)
- NFR-2: `resources/read` and `skills/*` reject any URI that resolves outside the allowed paths (path traversal)
- NFR-3: Enforcement logic has a single implementation, in `bin/jsdd`, which the MCP tools call
- NFR-4: Works on Windows with Git Bash, matching the plugin's existing requirements
- NFR-5: The server refuses to start, with a clear message, when the bundled kit version is outside the range it supports

## 9. First slice (milestone 1)

Keep it small and prove the enforcement idea:

1. stdio server skeleton + capability declaration
0. The kit releases from D6 that this slice needs, then pin that release in the server
2. Skills served from the bundled `skills/` (only `implement` needs to be correct for now)
3. `get_status`, `get_coverage` (shelling out to the bundled `bin/jsdd`)
4. `claim_task` + `tick_task` with real enforcement
5. Test with MCP Inspector (`--skill-verification`) and from a Claude Code session

Elicitation, Tasks, and the remaining tools come in later slices.

## 10. Open questions

- Q1: Language/runtime. A TypeScript server adds Node as a dependency on top of bash/jq/python3. Is that acceptable, or should it be Python (already required)? *Leaning Python: nothing new to install, and `uvx` gives a one-line install. Check which official SDK supports the Skills and Tasks extensions better before deciding.*
- ~~Q2: Where does it live?~~ Answered by D2: its own repo, `JoDi-SDD/jsdd-mcp`.
- ~~Q3: How are the plugin and server versioned together?~~ Answered by D3: separately, with the server pinning a kit release.
- ~~Q4: Should the plugin register the server automatically?~~ Answered by D4: no.
- Q5: Client support for the Skills extension is still early. Which clients is v1 actually targeting? *If few clients support Skills, milestone 1 should lead with the enforcing tools, not skill serving.*
- Q6: How does the server ship: PyPI (`uvx jsdd-mcp`), npm, or run straight from a git tag? And how does its build fetch and bundle the pinned kit release?
- Q7: How is the supported kit range declared and checked (NFR-5)? Could it be the same convention `sdd-runtime` uses for its pin?
- Q8: Where do server releases get announced: its own Announce step, titled "JoDi-SDD MCP server vX.Y.Z", posting to the same org Announcements?

## 11. References

- Skills over MCP: https://modelcontextprotocol.io/extensions/skills/overview
- Tasks extension (SEP-2663): https://modelcontextprotocol.io/seps/2663-tasks-extension
- MCP 2026-07-28 release notes: https://blog.modelcontextprotocol.io/posts/2026-07-28/
- Extension client support matrix: https://modelcontextprotocol.io/extensions/client-matrix
- Agent Skills spec: https://agentskills.io/specification
