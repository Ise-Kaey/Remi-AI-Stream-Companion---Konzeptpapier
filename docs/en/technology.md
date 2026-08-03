# Technology: one channel, several meanings

[Deutsch](../de/technik.md) | [English (Here)] | [Overview](../index.md)

---

This page explains how the agent triggers actions - and why that is solved
differently from how one would usually build it today. The short version first;
everything else is its justification:

> **The model proposes. The server decides and acts.**

## One channel, several meanings

From the outside the server looks like an ordinary chat endpoint in the common
format: a list of messages in, a reply out. In fact rather more than chat runs
through the content field of those messages. It is a multiplexed control
channel.

The same text can be an ordinary viewer message, a system event such as a raid,
or a deterministic command that never reaches the language model at all. The
same on the way back - alongside the visible reply text, a response may carry
moderation commands or a signal that simply means "do not answer".

One field, several layers of meaning. That is a protocol, not a text field.

## Why through the content field?

Because the client cannot read anything else. The shipped client deserialises
the response and accesses only that one field. An additional field in the JSON
would never arrive.

Rather than rebuilding the client, everything was tunnelled through the one
channel it speaks. That sounds like a compromise, but it is precisely the
decision that makes the server client-agnostic: any client that speaks the
common chat format thereby speaks the full protocol - without a line of special
handling.

## Directions

The token families are strictly separated by direction. There is one family for
the path from client to server, one for the path from model through server to
client, and one for model output that never leaves the server.

The separation is not merely tidiness. It makes **reflection impossible**: a
token belonging to one direction is not a valid token on the other path, it is
text. There would be routes back aplenty - a replayed conversation history, a
client that echoes, a log that finds its way into the prompt again. With
separate poles an entire class of confusions disappears without any check having
to exist for it.

## The model proposes, the server decides

No token executes itself. What the language model produces is a **proposal in
text form** - nothing more. The server reads it, checks it against rules the
model was never shown, and then acts on it or does not.

The same separation holds a layer down. An optional small language model
maintains user records in the background - it too writes nothing. It returns
structured proposed operations; the actual writing happens deterministically in
server code.

The distinction sounds academic but is the whole point. A model that calls tools
directly is an acting party. A model that produces text which a program
evaluates is a proposing party. Only in the second case can one say what may
happen at worst - namely at most what the server permits anyway.

## The feedback loop

So far this has sounded like a one-way street: the model proposes an action, the
server carries it out, done. The more interesting case is the one where the
proposal does not act outward but inward - on the data from which the next reply
is built.

The agent can append a **state note** at the end of a reply. It is invisible to
chat; the server cuts it out before anything is sent. Two kinds of entry appear
in it:

| What is noted | Effect |
| --- | --- |
| Its own state - mood, energy, tension, current activity | The agent does not stay stuck on its initial values but develops across a stream |
| Its assessment of the person currently being spoken to, plus a free-text note | Returning viewers are not treated as strangers every time |

The circle closes on the next pass: what the server wrote, it makes available
again when assembling the next prompt. Out of "model writes a note, server
stores it, server reads it back next time" arises what looks from the outside
like memory and like a relationship with individual people. Without this loop
the persona would stay static - the same mood, the same distance to everyone,
regardless of what came before.

What matters is that the rule holds unchanged throughout. The note is a
**proposal in text form**, not write access. Which fields are writable at all,
which values are admissible, whose record is affected - the server decides.
Some entries are locked after being set once, so the model cannot overwrite them
later.

This is the sharper example of the separation than any moderation action. With a
timeout it is obvious that a program ought to check. Here the model acts on the
very data from which its own next prompt is built - and even there it does not
write itself.

What follows from this for the stored data is on the [data page](data.md).

## Why not tool calls?

The obvious question: function and tool calling exist, so why parse strings?
Four reasons.

### Tool calling assumes *one* principal

Tool-calling semantics amount to: "the user asked for it, so run it." That holds
for an assistant with one user.

Here **N mutually unfamiliar people write into one context window**, and public
chat is hostile input by definition. In that setting tool calling conflates "the
model requested it" with "an authorised principal requested it". With a single
user those are the same thing; with many unfamiliar speakers in a shared context
they are two different things.

### No schema disclosure

A tool definition hands the model the attack surface as a map: names,
parameters, permitted values. An injection thereby has a well-formed target - it
does not have to guess, it reads it off.

With tokens the model knows *that* a capability exists and what it is for - but
not how it becomes an action. Whether the type is admissible in the current
context at all, how the target is resolved, which limits apply: that semantics
lives entirely in the server and is never exposed to the model.

### Fail-closed

An unknown tool call is an error in most frameworks. It is reported, often
retried, and generates signal - signal an attacker can work along.

An unknown token, by contrast, matches no pattern and remains text. A known
token with an unknown type runs into nothing. In both cases the outcome is the
same: nothing happens, and there is no feedback from which anything could be
learned. With hostile input that is the only correct default.

### Model portability and speed

Tool calling is not a base capability of a language model but trained behaviour
plus a provider-specific format. Emitting tokens, on the other hand, is
something any instruction-following model can do - including a small one running
locally. If a weaker model emits nonsense, fail-closed applies and an ordinary
chat reply remains. The failure mode is "less capable", not "broken".

That is at the same time why running without a cloud provider is viable at all:
the control layer imposes no requirement that only large models meet.

It also saves a round trip. Canonical tool calling is request → tool call →
execution → result back → final generation, so two inferences. Here the action
and the reply text arise in **one**. Since the server serialises requests
globally, that counts double: halving the calls per command also means
throughput for everyone currently waiting.

> In fairness, this last advantage holds only as long as the action need not
> flow *back* into the reply. A moderation command is fire-and-forget. A future
> extension that returns a result for the agent to then talk about inevitably
> needs a second inference - regardless of which mechanism triggers it.

## Further reading

- [Security](security.md) - the threat model and how the promises are enforced
- [Data](data.md) - what is stored and what leaves the machine
