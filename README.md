# Smart Tools

A format for packaging domain expertise into tools that any agent can use.

## What are Smart Tools?

A smart tool is a self-contained tool that ships with its own AI capability built in. It
behaves like an ordinary library or CLI, and its straight code paths run with no model
provider configured. Alongside those, it exposes higher-level commands that invoke AI under
the hood, so a caller states what it wants rather than loading a domain's worth of context
and doing the work itself.

At its core, a smart tool is a library, a manifest describing it, and a thin CLI over the
top. Anything else is an optional adapter over the same library.

```
my-smart-tool/
├── SMART_TOOL.md    # Required: the manifest, what it is and what it needs
├── pyproject.toml   # Required: the package definition
├── src/             # Required: the library, plus a thin CLI over it
└── ...              # Optional: MCP server, web UI, other adapters
```

## Why Smart Tools?

Hard-won domain expertise tends to stay where it was built, consumable only from inside the
harness it was built for and only with the right context loaded. Smart tools package that
expertise so it travels:

- **Consumable anywhere**: Copilot, Claude Code, a Python service, a shell script, or any
  future agent.
- **Useful without a model**: deterministic paths run with no provider configured, so a
  caller that never touches the smart commands never needs credentials.
- **The knowledge stays in the tool**: the caller states an intent and gets a structured
  result it can hand straight to code.

## Getting started

- **[Specification](spec/README.md)**: start here
- **[Structure](spec/structure.md)**: the layers a smart tool is built from
- **[Manifest](spec/manifest.md)**: how a smart tool describes itself
- **[Invocation](spec/invocation.md)**: calling a smart tool
- **[Roadmap](ROADMAP.md)**: what is deliberately left open

## License

[MIT](LICENSE)
