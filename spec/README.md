# Smart Tools

A smart tool is a self-contained tool that ships with its own AI capability powered by [amplifier-agent](https://github.com/microsoft/amplifier-agent). 
It behaves like an ordinary library or CLI, and its straight code paths run with no model provider
configured. Alongside those, it exposes higher-level commands that invoke Amplifier Agent
under the hood, so a caller states what it wants rather than loading a domain's worth of
context and doing the work itself.

## Overview

Hard-won domain expertise tends to stay where it was built. How to stand up an isolated
environment that actually mirrors production. How to get a useful answer out of M365. How
to turn a pile of material into a beautifully designed presentation. 
Each of these is real knowledge, and each is usually only consumable from 
inside the harness it was built for, with the right skills and context loaded.

A smart tool packages one domain's expertise so it can be consumed anywhere: Claude Code,
Copilot, a Python service, a shell script, or any agent.
The tool holds the domain knowledge, the workflows, and the choice of which of its own
capabilities to use. The caller states an intent and gets a result, usually a structured
artifact it can hand straight to code.

Two consequences shape everything else in this specification.

The first is that a smart tool is standalone. It is a library before it is anything else,
which is what lets the same capability appear in a CLI, in a bespoke UI, embedded in
someone's app, or driven by an agent that is not Amplifier. Making an existing capability
useful in all of those places is the point.

The second is that the AI is what makes it smart. A smart tool has to do something
genuinely powered by a model. If every path through it is deterministic code, it is a tool,
and that is fine, but it is not a smart tool. The straight code paths still run with no provider configured. 
A caller who never touches the smart commands never needs model credentials.

## The specification

Three chapters. Each closes with its own open questions, which are explicitly not frozen.

- **[Structure](structure.md)**: the layers a smart tool is built from. Library core, CLI
  wrapper, optional additional surfaces, and the rules governing the shipped AI capability.
- **[Manifest](manifest.md)**: how a smart tool describes itself: what it is, what it is
  for, what it runs on, and what it can do. This is the artifact a registry will later
  consume, and it is owned in the tool's own repo.
- **[Invocation](invocation.md)**: calling a smart tool. Straight and smart paths, passing
  context in, getting artifacts out, and failure semantics.

A worked example lives at [`docs/examples/digital-twin-universe.md`](../docs/examples/digital-twin-universe.md).

## Reading by audience

- **Building a smart tool?** Structure, then manifest, then invocation.
- **Calling one from an agent or an app?** Invocation, then manifest.
- **Deciding whether something should be a smart tool at all?** The overview above, then
  [what is out of scope](#deliberately-out-of-scope).

## Rules that apply everywhere

These hold for every chapter and every smart tool, and are not restated per surface.

**Fail loud.** Every error names the failure and the remedy. No silent partial results, no
quietly degraded modes, no synthetic stand-ins. A smart command invoked with no provider
configured fails saying so, and says what to configure.

**Provenance.** Anything derived, generated, or recalled carries where it came from.

**Idempotency.** Any operation with a side effect is safely retryable by design.

These are deliberately identical to the Cortex contract house rules, so a single tool can
satisfy both without reconciliation.

## Deliberately out of scope

Each of these is deliberately excluded. Naming them here prevents the specification from
drifting into them.

**This is not a protocol or a standard.** It describes a shape for building smart tools,
not a wire format for interoperating with third-party runtimes. Anyone may follow it, but
cross-vendor interoperation is not a goal it is trying to serve.

**Registry and discovery.** There will be a registry, and discovery is its own separate
project. Its format is undecided. This specification only requires that each tool own a
self-description in its own repo, so a registry can consume it whenever one exists. The
manifest chapter covers that and nothing more.

**Amplifier Agent knowing about smart tools.** The inverse direction, where a host
automatically discovers and offers installed smart tools, is explicitly secondary and is
not blocking. Tools ship first.

**Exposure as model-visible tools.** Whether a smart tool should be surfaced to a model as
a callable tool is unsettled and deliberately unspecified.

**Repo and package naming.** Unresolved and not a specification concern.
