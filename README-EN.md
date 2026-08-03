# Remi-AI Stream Companion — Concept Paper

[Deutsch](README.md) | [English (Here)] | [Documentation](docs/index.md)

---

**Remi** is a social AI agent for live streaming. It listens on call, reads
along in chat, remembers the people in front of it — and decides for itself
whether, when and how to react.

> ℹ️ **This repository is documentation, not a program.** The implementation is
> closed alpha and not public. What is written here is how the system is built,
> how it handles viewer data, and why the control layer was designed the way it
> was.

## What this is about

Two topics that are rarely treated together for chat agents:

**Security.** In a public chat, many mutually unfamiliar people write
simultaneously into one shared context window. The usual safeguards around AI
assistants assume a well-disposed user. Here hostile input is the normal case,
not the exception.

**Privacy.** An agent that remembers people stores data about people. Which data
that is, how long it stays and what of it leaves the device should be something
one can read up on before putting such a thing into a stream.

| Page | Contents |
| --- | --- |
| [Technology](docs/en/technology.md) | The control protocol, and why the model only proposes |
| [Security](docs/en/security.md) | Threat model, invariants, enforcement, limits |
| [Data](docs/en/data.md) | What is stored, what is transmitted, how to object |

## What is it?

Remi sits between the chat frontend and the AI services and holds everything
that has state: conversation history, user profiles, moderation history, stream
status and persona. The frontend only delivers events and performs actions.

```text
   Client                      Remi                       AI services
   ──────                      ────                       ───────────
   Chat in           ──▶       Persona · History   ──▶     Reply text
   Stream events     ──▶       Memory · Rules      ──▶     Speech output
   Timeout / Ban     ◀──       Voice path
                               Microphone ──▶ ──▶ Speakers
```

The server itself has **no platform integration** — no channels, no credentials,
no interface to Twitch, YouTube or Discord. Moderation actions travel through a
queue to the client, which carries them out and acknowledges them. The server
could not ban anyone even if it wanted to.

That is why the integration is interchangeable — and at the same time why
certain data never arises in the first place.

### What it does

Sketched briefly, so the later claims can be placed — what the agent can do
determines what there is to say about security and privacy in the first place.

| | |
| --- | --- |
| **Voice** | Listens on a key press or a wake word, recognises what was said locally, and answers aloud. After a reply the microphone stays open briefly so follow-up questions need no second key press |
| **Chat** | Reads along and does not answer everything — by probability, but always when addressed directly. Greets viewers on first appearance, summarises on request |
| **Memory** | One shared conversation history for voice and chat, plus profiles of individual viewers. Both feed into every reply |
| **Moderation** | Can **propose** a warning, timeout, ban or pardon. Execution happens exclusively in the client — the server holds no platform rights |
| **Persona** | Who the agent is lives in interchangeable text files and can be switched while running |

Moderation is the part with teeth, and therefore the reason the
[security page](docs/en/security.md) runs as long as it does.

### Streaming is the reference case, not the limit

Live streaming is the use for which Remi was built and is tested daily — and the
most demanding one: many unfamiliar speakers at once, real time, and every
mistake is immediately audible. The architecture itself knows nothing of this.
Three things are interchangeable:

| Part | Determines |
| ---- | ---------- |
| **Persona** | Who the agent is and how it speaks |
| **Databases** | What it remembers |
| **Client** | Where events come from and who performs actions |

Replace all three and you get a different agent on the same machinery:

- **Smart home** — a client on the house automation, persona and database
  tailored to devices and habits. For a voice assistant meant to work without
  the cloud, local speech recognition is the interesting part
- **Meetings** — the same voice path as a minute-taker, the databases as a
  growing knowledge base rather than a viewer memory

There is no second use case today — this is a design property, not a feature.
But it explains why the command layer is kept generic and the client interface
is documented.

## How it stays local

The agent consists of four processing stages that differ in whether data leaves
the device:

| Stage | Where | Does data leave the machine? |
| --- | --- | --- |
| Speech recognition | locally, on the CPU | no — audio stays put |
| Memory maintenance | locally, can be switched off | no |
| Main language model | freely configurable | only with a cloud provider |
| Speech output | cloud provider | yes — text goes out |

If the configuration points at a self-hosted language model, nothing beyond the
text for speech output leaves the machine. Details, and the important
qualification — *what exactly goes into a prompt* — are on the
[data page](docs/en/data.md).

## What this repository is not

- **Not source code.** The implementation is closed alpha.
- **Not installation instructions.** There is nothing here to install.
- **Not interface documentation.** Endpoints, parameters and examples belong to
  the implementation and are deliberately not here.
- **Not an offer and not support.** This is a description, not a commitment.

If you want to take up the concepts: the ideas on the three pages are not
secret, and they are written down deliberately so that they can be rebuilt
without this code.

## Licence

The texts in this repository are licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The software
described is not covered by this.
