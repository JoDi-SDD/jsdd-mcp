# Intent

> What we are building, for whom, and why: the short version. When a requirement, design, or decision conflicts with this file, either this file is wrong or the other thing is; resolve it, don't ignore it.

## The product in one sentence

A local MCP server that delivers the JoDi-SDD kit to any MCP client: the kit's skills served over MCP, and its rules enforced in tools, so the model changes spec state only through checks the server runs.

## Who it is for

Teams doing spec-driven development with an AI coding agent in any MCP client. Tools work in every client; skills reach the clients that support the Skills extension. Which clients v1 is tested against is still open (ND-02).

## Why it should exist

The kit works only in Claude Code: its rules (Claude can't tick its own boxes, commits must cite a spec) are enforced by hooks and instructions on the user's machine. The MCP 2026-07-28 spec and its official extensions let a server ship skills, enforce rules in tools, ask the user for approval (Elicitation), and run long work (Tasks). That makes a second channel possible, one where enforcement sits in the server rather than in client hooks.

## What we are building

- A local stdio MCP server, one process per session.
- The pinned kit release's `skills/` served through the Skills extension (`skills/list`, `skills/get`, `resources/read`).
- The kit's documents (intent, requirements, decisions, backlog, each spec's files) as read-only resources under `jsdd://`.
- Tools that change SDD state only through `bin/jsdd`: status and coverage, claiming and ticking tasks, creating specs, submitting plans, recording decisions, scaffolding a project.
- Elicitation for approval steps, and Tasks for long runs such as verification.

## Principles

- **One rule source.** Every rule is implemented once, in the bundled `bin/jsdd`. The server calls it and never reimplements a check.
- **Pinned public kit.** The server uses the kit only at a pinned release, through `bin/jsdd --json` and the shipped `skills/`. It refuses to start on an unsupported kit version. Anything missing is a kit change first.
- **Stateless processes.** A server process holds nothing needed across sessions. All state is in the repository's files or in git, so a process can be killed at any time.
- **Client-neutral.** The server never refers to the Claude Code plugin or to any one client, and serves the kit's skills unedited.

## What success looks like

A year from now, in this order:

1. **Enforcement proven.** In at least one client other than Claude Code, the model can't change SDD state except through checks the server runs.
2. **Usable in two or more clients.** A team runs the full specify → plan → implement → verify flow through jsdd-mcp in at least two MCP clients other than Claude Code, with ticking and claiming enforced in the server.
3. **Parity with the plugin.** Every `/jsdd:` command has an MCP equivalent (hooks excepted), released and announced.

## Not now

- Replacing the Claude Code plugin. Both channels coexist.
- Porting hooks. MCP has no client-side hooks, so commit and session-start checks stay in the plugin.
- Porting agents as subagents. They become skills with checklists, or tools.
- A remote or hosted server. v1 is local stdio only.

## First version

See [PLAN.md](../PLAN.md) Phases 0 and 1.
