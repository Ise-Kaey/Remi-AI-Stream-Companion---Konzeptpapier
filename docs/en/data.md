# Data

[Deutsch](../de/daten.md) | [English (Here)] | [Overview](../index.md)

---

An agent that remembers people stores data about people. This page states which
data that is, how long it stays, and what of it leaves the machine.

## What is stored

Five categories, all held locally as files on the operator's machine:

| Category | Contents |
| --- | --- |
| **Conversation history** | What was said and answered, with sender and time. Voice and chat share one history |
| **Viewer profiles** | Display name, preferred form of address, recorded gender, optionally a birthday, plus categorised traits ("likes cats"), a free-text field, and the agent's standing assessment of the person (such as *trusted*, *stranger*, *sceptical*) |
| **Moderation records** | Infractions with time, reason and the measure taken, plus an escalation level |
| **Community knowledge** | Facts about the community as a whole, not attributed to any person |
| **Operational logs** | The course of operation - what failed, what was triggered. Conversation content does not appear there, see [below](#operational-logs) |

Identifiers are the display names from chat. No real names, e-mail addresses,
payment details or platform identifiers are collected - the server has no access
to any of those in the first place, see [data minimisation](#data-minimisation-by-design).

Alongside these sits a further body of data, mentioned here only for
completeness: the **agent's own state** - its mood, energy level, current
activity, together with figures about the running stream. It contains no
personal data and therefore does not appear in the sections that follow. How it
arises and why it exists is covered under [the feedback
loop](technology.md#the-feedback-loop).

## For how long

| Category | Retention |
| --- | --- |
| Conversation history | bounded; the oldest entries drop out automatically. Two limits apply in parallel - number of entries and volume |
| Viewer profiles | indefinite, until the operator deletes or the person concerned objects |
| Moderation records | the escalation level decays automatically over time; the records themselves remain |
| Community knowledge | indefinite |
| Operational logs | a fixed period; older files are deleted at startup |

The conversation history is thus the only category with automatic eviction.
Profiles persist - that is their purpose, and it is the point at which an
operator has to make a conscious decision.

**On scale.** The history is bounded to a few hundred messages, configurable; in
a lively chat that is minutes to hours. For comparison: the streaming platform
itself typically retains the complete chat log for weeks. The agent therefore
stores a small excerpt of what is already being logged anyway.

That is not a justification - the operator is responsible for their own
processing regardless of what the platform does, and the two are separate
controllers with separate purposes. But it puts the order of magnitude in
perspective.

### Operational logs

A service that runs writes down what it does. These logs are the only category
with neither purpose nor use in the agent's memory - they exist for debugging
and accountability. Which is exactly why they are the place where personal data
is most likely to sit unnoticed.

Three measures apply there:

| | |
| --- | --- |
| **Do not write it** | Lines that would carry chat messages, voice transcripts, replies or profile data are marked as personal at the point where they arise, and are discarded before reaching the file |
| **Pseudonymise** | What remains - chiefly moderation events, which should stay documented for accountability - carries a salted, stable identifier instead of the display name |
| **Time-limit** | Older files are deleted. The default is oriented on the streaming platform's own retention; it can be shortened or switched off entirely |

What is notable about the first measure is that it does **not** work by pattern
matching. There is no attempt to filter out of a finished log line whatever
looks like a name in it - instead, when the line is written, a decision is made
about whether its content is personal. Whoever later adds a new log message
makes that decision at exactly the point where they know the content. The same
reasoning as the refusal of filter lists in
[security](security.md#structure-instead-of-filtering): detection that has to
guess fails silently.

Not covered is the log file that serves development only - it is overwritten on
every start and therefore exists only for the duration of a session.

## What leaves the machine

The agent consists of four processing stages. They differ in whether data leaves
the device:

| Stage | Where it runs | Does data leave the machine? |
| --- | --- | --- |
| **Speech recognition** | locally, on the CPU | **No.** Recorded audio is turned into text locally and never leaves the device |
| **Memory maintenance** | locally, can be switched off entirely | **No.** |
| **Main language model** | freely configurable | **Depends.** If the configuration points at a cloud provider, data goes there. If it points at a self-hosted model, it does not |
| **Speech output** | cloud provider | **Yes**, but only on the voice path. What is transmitted is the spoken text |

For the default case - a cloud provider for the language model - that means:
audio stays local, text goes out.

Two qualifications about speech output, and they cut both ways. It receives
**considerably less** than the language model: no conversation history, no
profile, no moderation data - only the one sentence to be spoken. And it hangs
off the voice path alone; chat replies are not read aloud and never reach the
provider.

Even so, "no personal data" claims too much. The voice path draws on the same
history as chat. If the agent talks about what is happening in chat, the spoken
sentence can contain names and details about third parties - and that sentence
is exactly what gets transmitted.

## What goes into the prompt

This is the most important statement on this page, and it is often overlooked:
**what goes to the main language model is not just the current message.**

Before each request the server assembles a context. Among other things it
contains:

- the recent conversation history, and therefore contributions by other people
- the profile of the person speaking, including stored traits
- the profile of a second person, if they are mentioned in the message
- whether a moderation level is active for this person
- state data about the stream and the persona

So anyone using a cloud provider for the language model is thereby transmitting
the stored viewer data to that provider - not as a database export, but
continuously and in excerpts. This is not a side effect but the mechanism by
which the agent remembers people at all.

Anyone who does not want that can point the language model at a self-hosted
endpoint. Then nothing beyond the text for speech output leaves the machine. The
control layer is deliberately designed to work with small local models too - see
[model portability](technology.md#model-portability-and-speed).

### Three gradations

Between "everything at the provider" and "nothing leaves the device" lies a
middle tier that is likely the most common in practice:

| Setup | Where viewer data goes |
| --- | --- |
| Language model at a consumer-tier provider | To the provider, on their terms - retention and possible use for training follow the plan |
| Language model over a business account with a data processing agreement, without logging and without training permission | To the provider, but as a processor acting on instructions |
| Language model self-hosted | Nowhere. Speech output remains |

The middle tier is where the processor question can be settled properly rather
than sidestepped: a contract, purpose limitation, no secondary use. The bottom
tier removes it altogether. The choice rests with the operator, and it is a
matter of configuration - not a different build.

## Objecting

Data subjects can object to their personal data being stored, via a chat
command. The command runs deterministically in the server and does not pass
through the language model at all.

It **clears and locks** the fields concerned. The locking is the more important
part: mere deletion would mean that memory maintenance collects the same details
again during the next conversation. A locked field cannot be repopulated, not
even by the model.

**What is retained, and why.** The objection is not complete. This is the most
important qualification on this page, so it is broken down in full:

| Field | On objection |
| --- | --- |
| Birthday | cleared and locked |
| Collected traits | cleared and locked |
| Free-text field | cleared and locked |
| Display name | **retained** |
| Preferred form of address | **retained** |
| Recorded gender | **retained** |
| The agent's assessment | **retained** |
| Time first seen | **retained** |

The dividing line follows a principle: what is cleared is what the agent has
*accumulated* about a person. What is retained is what operation needs in order
to keep working correctly at all - display name and form of address for correct
address, gender for pronouns, time first seen for attribution. Without a name no
attribution, without attribution no lock.

One field warrants its own justification: the **assessment**. It differs in
origin from all the others - it is not something the person stated but a
judgement the agent forms about their behaviour. That is precisely why it is
retained.

If objecting reset it, objecting would be a reset button: disrupt the chat,
object, and be treated as a blank slate again - warmly, unsuspectingly, from the
top. The assessment is the memory of how someone behaved **towards others**, and
it protects not the agent but everyone else present. The right to have one's own
data erased does not extend to the consequences of one's own behaviour for third
parties.

Important here, and the reason this is not a stigma: **the assessment is a
snapshot, not a file.** It is continuously updated from behaviour towards the
agent and towards everyone else present, and it moves in both directions.
Behave differently and you are assessed differently. It does not latch, and it
has no final state - the same mobility with which the moderation level declines
of its own accord.

What remains is the honest residue: the assessment cannot be removed on request.
It can be changed - but only by the route through which it arose. That is
intended, and justified here, not overlooked.

### The conversation history is left untouched

Objecting acts on the profile, not on the running history. That is not an
oversight but follows from what the history is: a **conversation transcript**,
not a record about a person. Its lines refer to one another. Remove one voice
and replies are left without questions while references run into nothing - and
those very references are what make a reaction fit. A purged history would not
be a more data-minimal history but a broken one.

What defuses this is the retention itself. The history is bounded to a few
hundred messages and rolls over within hours in an active chat. It needs no
deletion function because it is only ever a short window - eviction does faster
what an erasure request would first have to set in motion.

The third reason weighs more than the first two: an objection that took the
history with it would be a free pass. The history is the window in which a
violation is visible at all, before it is recorded as a moderation entry.
Anyone able to clear it on request could remove the evidence of their own
conduct before anyone reacts to it - disrupt, object, carry on unencumbered.

It is the same principle as with the assessment, only where it shows most
plainly: the right to have one's own data erased does not extend to removing
traces of one's own conduct towards others.

Anyone wanting complete deletion has to approach the operator - the data sits as
files on their machine, and removing it entirely is an administrative act there,
not a function of the agent.

## Data minimisation by design

Some things are not stored not because they are deleted, but because they never
arrive:

- **No platform integration.** The server holds no credentials for Twitch,
  YouTube or Discord, no channels, no platform interface. It sees only what a
  client sends it.
- **No execution rights.** Moderation actions are proposed by the server and
  handed to the client. They are carried out there. The server could not ban
  anyone even if it wanted to.
- **No identities.** What is stored are chat display names. Whether a real
  person stands behind one, and which, the server does not know.

This is not accidental but follows from the architecture: the client delivers
events and performs actions, the server holds the state. Where there is no
connection, no data arises either.

## What the operator has to decide

This software is not a service. It runs on one person's machine, and that person
is the one answerable to the viewers. This paper therefore cannot declare an
installation "compliant" - that depends on how it is set up.

What an operator should settle before running it in production:

- **Purpose and legal basis.** Why is viewer data stored, and on what does that
  rest?
- **Transparency.** Do viewers know that an agent is reading along and
  remembering things? Do they know how to object?
- **Recognisability of the AI.** Is it apparent that the replies come from a
  language model? That is a separate duty under a separate legal act - it does
  not coincide with data-protection transparency and applies even if nothing
  were stored at all. See below.
- **Processors.** Which providers are involved, what do they transmit, and are
  the necessary agreements in place? Running with a local language model shrinks
  this question considerably.
- **Access and erasure.** How is a request answered, and who gets at the files?
- **Retention.** Profiles persist indefinitely. Is that intended?

The technical preconditions - local processing where possible, objection by
command, no platform integration - are in place. The decisions about them are
not, and they cannot be delegated to software.

## Recognisability of the AI

Replies produced by a language model are visibly marked in chat. Three design
decisions behind that are worth mentioning:

**The marking sits in the server, not in the client.** The interface is
documented and served by self-built clients - a client-side solution could
simply be forgotten. Server-side it applies to every connected client,
including the one that does not exist yet.

**It is not in the prompt.** An instruction to "identify yourself as an AI"
would depend on the model following it - and would thereby be a target for
precisely the attacks the rest of the system defends against. Anyone who got the
agent to drop its marking would not have won a joke but disabled a legal
obligation. It is therefore applied in code to the finished reply, out of the
model's reach. The same principle as everywhere else:
[the server decides, not the model](technology.md#the-model-proposes-the-server-decides).

**What gets marked is what the model produced - not everything the bot says.**
Command acknowledgements, error messages and fixed texts do not carry the mark.
A sign that appears on every output says nothing about how that output came
about; it would not unify the signal but devalue it.

The marking covers chat. For spoken output there is no textual counterpart -
there, recognisability rests on the context in which the agent appears and on
the notices the operator places outside chat.

## Further reading

- [Technology](technology.md) - how the protocol works
- [Security](security.md) - the threat model and enforcement
