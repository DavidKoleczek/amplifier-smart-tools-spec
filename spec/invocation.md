# Invocation

Calling a smart tool should feel like calling any other tool. Arguments in, a result out,
an error that says what went wrong. The fact that some paths consult a model is an
implementation detail of those paths, not a different calling convention.

This chapter covers what a caller passes in, what comes back, and what happens when things
fail.

## Straight and smart paths

A smart tool exposes both kinds of capability through one surface. There is no separate
mode to enter and no separate binary for the model-backed parts.

The difference that matters to a caller is not how a capability is implemented but what it
costs and how reliable it is. A deterministic path returns the same answer every time and
costs nothing beyond compute. A model-backed path may return a different answer on a second
run and consumes tokens. Callers making budget or retry decisions need to know which they
are invoking.

Callers who never invoke a model-backed path never need model credentials, and the tool
must not require them in order to load.

Beyond that signal, the internals stay the tool's business. A caller should not have to
know whether a given result came from code, from a model, or from both in order to use it
correctly.

## Self-description

A smart tool describes its own surface at two levels of detail, for two different readers.

`-h` is the human summary. Terse and scannable: the capabilities and a line about each. It
is what someone types when they want to remember the name of a flag.

`--help` is the complete listing, written for an agent deciding how to call the tool. Every
capability, its arguments and their types, what it returns, and which capabilities are
model-backed. It is longer than a person wants to read, and that is the point. The reader
is something that can take all of it at once and gains nothing from being protected from
detail.

This inverts the usual convention, where the two are aliases. The reason is that the two
readers want genuinely different documents, and a tool that serves only one of them either
buries a person in detail or starves an agent of it.

Neither replaces the manifest. The manifest is read before deciding to install or invoke
the tool at all; these are read once it is present and the question is how to drive it.

## Passing context in

Smart paths take normal typed arguments like any other function. Alongside those, they
accept an optional context payload: additional material the caller already has that would
help the tool do the job well.

Two rules govern it.

**The payload is data, not a reference.** At the library level, a caller passes the actual
content. This keeps the library free of assumptions about where the caller's material lives
and keeps it usable from processes that have no filesystem in common with the caller. A CLI
wrapper may accept a file path and read it into the payload, because that is a convenience
of the command line, not a change to what the library accepts.

**The caller decides how much, and code assembles it.** How much context to hand over is
the caller's judgment. Something like none, partial, and full is the useful granularity,
matching what agent delegation already offers.

Assembly should be mechanical. When an agent composes the payload by deciding what seems
relevant, the tool's result becomes a function of that agent's summarizing rather than of
the material itself, and two callers with identical inputs get different answers. Whatever
selects and packages the context should be code the caller controls.

## What comes back

A result a caller can act on without parsing prose.

At the library level, that means ordinary return values: objects, dataclasses, dictionaries.
At the CLI, it means structured output on stdout. The test is whether the caller can hand
the result to the next step programmatically. A model-backed capability that returns a
paragraph of explanation has moved the work of understanding back onto the caller, which is
the thing the tool exists to absorb.

Where a capability produces an artifact, a profile, a document, a configuration, the result
identifies the artifact rather than embedding it in a message.

Where a result is derived from a model, it carries that fact, so a caller can apply
different trust to a generated value than to a measured one.

Long-running capabilities are worth calling out because they are common in this category. A
capability that fans out across a domain can run far longer than a normal function call.
How progress is reported is not settled below.

## Failure

Failures are loud and they name the remedy. A caller should never have to infer what went
wrong from an empty result.

Three cases are common enough to state:

**A missing prerequisite** fails immediately, naming what is absent and how to install it.
The manifest already declares these, so the failure and the manifest should agree.

**A smart path with no provider configured** fails saying exactly that, and says what to
configure. It does not fall back to a degraded deterministic answer, because a caller that
asked for the smart path and got a lesser result without being told has been misled about
what it received.

**A partial result** is a failure unless the capability documents partial completion as a
valid outcome, in which case the result says which parts succeeded. Silently returning the
portion that worked is the failure mode this rule exists to prevent.

Retrying a failed call should be safe. Capabilities with side effects are designed so that
a caller which retries after a timeout does not cause the work to happen twice.

## Open questions

**Whether smart paths are visibly marked.** The position above is that callers are told
which capabilities are model-backed, on the grounds that cost and determinism are the
caller's business, and that the `--help` listing is where they are told. The competing
position, taken by `agent-tool.v1`, is that a conforming tool should be
implementation-invisible, so that a tool stays free to change how a capability is
implemented without breaking anyone. Both are defensible and they conflict. If the
distinction is dropped, the `--help` listing loses that field and nothing else here changes.

**Progress and streaming.** Nothing here specifies how a long-running capability reports
progress, whether intermediate output can stream, or how a caller cancels. This is reserved
in `agent-tool.v1` as well and is likely the first gap a real tool hits.

**Cost reporting.** Whether a result should carry what it cost in tokens or time. Callers
budgeting across many calls will want it; nothing needs it yet.

**Where context granularity is expressed.** None, partial, and full are described as useful
granularity, not as a required parameter with those names. Whether this becomes a common
convention across tools or stays each tool's own choice is undecided.
