# Security

[Deutsch](../de/sicherheit.md) | [English (Here)] | [Overview](../index.md)

---

This page assumes the [technology page](technology.md) - in particular the
separation between proposal and execution.

## The threat model

Most safeguards around AI assistants assume a user who is well disposed towards
the assistant. The user has a goal, the assistant helps, and the risk lies in
the model doing something nonsensical.

Here the situation is different. In a public chat, **many mutually unfamiliar
people write simultaneously into one shared context window**. They do not know
each other, they pursue different intentions, and some of them are trying out
what happens. Public chat is therefore not "occasionally problematic input" but
hostile input by definition.

From that follows the question this paper sets out to answer: what can a single
person who is allowed to write arbitrary text trigger at worst?

## The invariants

Three rules carry the safety of the approach. They are deliberately kept short
enough that a violation is noticeable on reading.

**1. The model proposes, the server decides.** No token executes itself. Covered
at length on the [technology page](technology.md).

**2. The actor comes from the transport, never from the model output.** Who
triggers an action follows from the request context - that is, from who
demonstrably sent the message. The *target* of a moderation action may come from
the parameters of the proposal; the *actor* never may. Without that separation
anyone allowed to write text could act in someone else's name.

**3. System events come only from a reserved sender.** Events such as raids or
ad breaks are accepted only when marked as originating from that sender. This
cannot be forged from chat, because the client prepends the real name to every
viewer message - the marking then no longer sits where it would count.
Additionally the name is reserved and can never exist as a real chatter, so the
namespace is collision-free.

In addition: control blocks are removed before a reply is written to the
conversation history. Control text therefore cannot re-inject itself by way of
the log.

## Structure instead of filtering

The approach stands or falls on one promise: **user text must not be read as
control.** Otherwise someone types a command into chat and the server takes it
seriously.

What is notable is how this is solved - namely **not** by filtering.

Control tokens are not stripped from incoming text. Nor do they need to be. The
client prepends the real name to every viewer message; the message therefore
never begins with a token but always with the name. The server's check is a
check on the start of the string - it simply does not fire. Not because
something was filtered out, but because the token never sits where it would
count.

The difference from a filter list is essential:

| | Filter list | Structure |
| --- | --- | --- |
| Covers | every token someone thought of | tokens that do not exist yet as well |
| On a new token | must be extended | still valid unchanged |
| Failure case | one forgotten entry opens a hole | no single entry that could be missing |
| Visibility of the failure | silent | - |

A list has to grow with every new token, and whoever forgets one does not
notice. The structural variant automatically covers tokens that had not been
invented when it was written.

## What the server sanitises anyway

Two things are not covered structurally and are therefore handled actively. Both
happen **server-side** - so they also apply to self-built clients that left them
out or got them wrong.

**Placeholder syntax.** The system prompt works with placeholders, and user text
reaches that same prompt by way of the conversation history. Characters that
could be used to reconstruct a placeholder are therefore replaced on arrival by
visually equivalent but inert variants. The replacement is idempotent: if a
client has already done it, nothing further happens, so old and new client
versions can be mixed.

**The conversation history format.** The history is a second in-band channel -
one line per turn, with sender and timestamp. That was precisely a gap: it was
enough to begin a new line inside one's own message to invent further turns.
Including replies by the bot persona. The language model could thereby be
conditioned, through in-context learning, on a history that never happened. The
attack needed no special characters and no knowledge of internal formats - it
needed a line break.

This is handled where the history format is created, not at the entrance. The
protection therefore also covers the voice path and any future caller - the same
principle as "the server decides, not the model".

## On the way back

Conversely, what the language model produces must not reach chat unchecked.
Internal control blocks are removed before sending, including orphaned markers
in case the model forgets a closing tag. All output paths run through the same
function so that no control token becomes visible by a detour.

## Limits

A security paper without limits is advertising. Three things belong here:

**The access layer is a shared secret.** The server checks a configurable key on
requests. If none is set and the service is run on an open network, it is
reachable by anyone who finds it. That is an operational decision this paper
cannot take away.

**The safeguards protect the server, not the model from itself.** Everything
described above ensures that a viewer cannot trigger an action they are not
entitled to. It does not ensure that the language model answers sensibly or
cannot be talked into an inappropriate remark. That is a different class of
problem, and it is not solved.

**The client is part of the trusted zone.** The load-bearing promise - that the
real name is prepended to every viewer message - is kept by the client. A client
that sends raw user text without that marking defeats the structural layer. The
server cannot tell the difference, because producing that very distinguishability
is what the prefix does.

## Further reading

- [Technology](technology.md) - how the protocol works
- [Data](data.md) - what is stored and what leaves the machine
