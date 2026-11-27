# CLAUDE.md

Guidance for Claude Code in this repository.

## What this is

`jsdd-mcp` is the MCP channel for the JoDi-SDD kit: a local stdio MCP server that serves the kit's `skills/` and exposes its rules as tools. The Claude Code plugin is the other channel and lives in `JoDi-SDD/JoDi-SDD`, next to the kit's core.

The repository is new. [docs/brief.md](docs/brief.md) holds the design so far, including the decisions (§3a) and open questions (§10). The language and runtime are not chosen yet (Q1); don't add code or tooling until that's decided.

## Boundary rules

1. **Rules live in `bin/jsdd`.** The server calls the bundled, pinned copy and never reimplements a check.
2. **Kit access only through the public surface:** `bin/jsdd --json` and the shipped `skills/`. Never read or copy the kit's internals.
3. **Missing in the kit means a kit change first.** Open it in `JoDi-SDD/JoDi-SDD`, wait for the release, then bump the pin here.
4. **Never refer to the Claude Code plugin** in code, docs, or tool output, and never edit the bundled `skills/`.
5. **Pinned kit version.** Startup refuses a kit version outside the supported range, with a clear message.

## Working rules

- Work follows the kit's own flow once the project is set up with `/jsdd:init`: specs in `specs/NNN-slug/`, requirements in `docs/requirements.md`, decisions in `docs/decisions.md`.
- Commit messages cite the spec and task (`[001] T3 ...`), or `[meta]` for work outside a spec.
- Keep `CHANGELOG.md` up to date under `## Unreleased`.
