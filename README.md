# jsdd-mcp

An MCP server for [JoDi-SDD](https://github.com/JoDi-SDD/JoDi-SDD), the spec-driven development kit. It delivers the kit to any MCP client: the kit's skills served over the Skills extension, and its rules enforced in tools, so the model changes spec state only through checks the server runs.

> **Not usable yet.** This repository is being set up. What it will do is in [docs/intent.md](docs/intent.md) and [PLAN.md](PLAN.md); nothing is implemented or released.

## How it relates to the kit

The Claude Code plugin and this server are two channels for the same kit. The server:

- bundles a pinned kit release (its `skills/` and `bin/jsdd`) and checks at startup that the kit version is one it supports;
- uses the kit only through `bin/jsdd --json` and the shipped `skills/`, never its internals;
- never reimplements a rule: every check runs in `bin/jsdd`.

If the server needs something the kit doesn't expose, that is a kit change first, released, then a pin bump here.

v1 is a local stdio server. Hooks stay in the Claude Code plugin; MCP has no equivalent.

## License

[MIT](LICENSE)
