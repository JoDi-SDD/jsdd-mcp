# CLAUDE.md

Guidance for Claude Code in this repository.

## What this is

`jsdd-mcp` is the MCP channel for the JoDi-SDD kit: a local stdio MCP server that serves the kit's `skills/` and exposes its rules as tools. The Claude Code plugin is the other channel and lives in `JoDi-SDD/JoDi-SDD`, next to the kit's core.

The project is set up for spec-driven development: start from [docs/intent.md](docs/intent.md), [specs/constitution.md](specs/constitution.md), [PLAN.md](PLAN.md), and `/jsdd:status`. The original design brief is kept in [docs/legacy/brief.md](docs/legacy/brief.md) for reference; where it disagrees with the kit files, the kit files win. The language and runtime are not chosen yet (ND-01 in [docs/decisions.md](docs/decisions.md)); don't add code or tooling until that's decided.

## Boundary rules

1. **Rules live in `bin/jsdd`.** The server calls the bundled, pinned copy and never reimplements a check.
2. **Kit access only through the public surface:** `bin/jsdd --json` and the shipped `skills/`. Never read or copy the kit's internals.
3. **Missing in the kit means a kit change first.** Open it in `JoDi-SDD/JoDi-SDD`, wait for the release, then bump the pin here.
4. **Client-neutral (P4).** No code, tool output, or served content names the Claude Code plugin or a specific client, and the bundled `skills/` are never edited. Project docs may explain how this channel relates to the plugin.
5. **Pinned kit version.** Startup refuses a kit version outside the supported range, with a clear message.

## Working rules

- Work follows the kit's flow: specs in `specs/NNN-slug/` through the `/jsdd:*` skills, requirements in `docs/requirements.md`, decisions in `docs/decisions.md`, open questions and spikes in `BACKLOG.md`.
- Commit messages cite the spec and task (`[001] T3 ...`), or `[meta]` for work outside a spec.
- Keep `CHANGELOG.md` up to date under `## Unreleased`.
