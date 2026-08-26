# Manifest

Every smart tool carries a manifest: a single file, in the tool's own repo, that says what
the tool is, what it is for, and what it needs in order to run. It is how a caller decides
whether this is the right tool before invoking anything, and it is what a registry will
consume once one exists.

The manifest lives in the tool's own repo. Prerequisites and platform support change when
the code changes, so they are updated in the same commit. A registry reads from the repo
rather than holding its own copy.

## Selection, not operation

The manifest answers "is this the tool for my job, and can I run it here?"

It does not answer "how do I call it." That is `--help` and the library's own signatures.

The practical test: a field belongs in the manifest if a caller needs it *before* deciding
to install or invoke the tool. Everything else belongs in the tool.

## Where it lives

`smart-tool.md` at the repo root. YAML frontmatter, then a Markdown body.

The frontmatter is machine-readable and is the part other things depend on. The body is
free-form guidance for whoever is about to use the tool: when to reach for it, what it is
bad at, worked invocations. Advice, not law. It changes whenever taste changes.

## The frontmatter

Fields not listed here are not part of the manifest.

```yaml
smart_tool_format: 1
name: digital-twin-universe
version: 0.4.0
description: >
  Stands up isolated, realistic environments from a declarative profile so software
  can be tested as though actually deployed. Use when "tests pass locally" is not
  enough evidence.
use_cases:
  - Test a web app in a container that mirrors its real deployment
  - Simulate an end-user environment without touching production
  - Verify a CLI tool installs and runs cleanly from scratch
platforms:
  - linux
  - macos
requires:
  - name: incus
    purpose: Runs the isolated environments.
    probe: incus version
    install: docs/installing-incus.md
  - name: docker
    purpose: Mock service sidecars and Gitea mirrors.
    optional: true
    probe: docker version
    install: docs/installing-docker.md
```

**`smart_tool_format`** is the manifest schema version, not the tool's version. It exists so
a reader can tell whether it understands the file at all.

**`name`** is lowercase alphanumeric and hyphens. It is the tool's identity across the
registry, the package index, and the CLI.

**`version`** is the tool's version, and there is exactly one of them. A tool that publishes
different versions in its package metadata, its manifest, and its docs has three answers to
a question that has one, and every consumer picks the wrong one eventually.

**`description`** says what the tool does and when to reach for it, in that order. It is
read by users scanning a registry and by agents deciding whether to route work here, so it
should carry the words someone would actually use when they need this tool.

**`use_cases`** are the concrete jobs the tool is for. These are selection aids, not a
capability list, and they should read like things a person wants, not like functions the
tool exposes.

**`platforms`** is the set of operating systems the tool actually works on. Not the ones it
theoretically compiles for. A tool that has never been run on Windows does not list Windows.

**`requires`** declares what must exist in the environment before the tool can run. Each
entry names the dependency, says what it is for, gives a command that proves whether it is
present, and points at how to install it. `optional: true` means the tool runs without it in
a reduced form, and the entry's `purpose` should make clear what is lost.

Language runtimes and package dependencies are not listed here. Those belong to the packaging
system, which already resolves them.

## The body

Everything below the frontmatter is guidance, written for whoever is about to use the tool,
user or agent. Typical contents: when this tool is the right choice and when it is not,
sharp edges, worked invocations, and pointers to deeper documentation.

It is not versioned, and nothing may depend on a particular sentence being present.
