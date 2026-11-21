# Data

[Deutsch](../de/daten.md) | [English (Here)] | [Overview](../index.md)

---

An agent that remembers people stores data about people. This page states which
data that is, how long it stays, and what of it leaves the machine.

## What is stored

Eight categories, all held locally as files on the operator's machine:

| Category | Contents |
| --- | --- |
| **Conversation history** | What was said and answered, with sender and time. Voice and chat share one history |
| **Diary** | The condensed version of older conversation segments: a short narrative from the agent's point of view, plus key points - topics, mood, what remained open and which of it has been settled since. **Heavily condensed:** a segment of up to around 50,000 word fragments becomes a few hundred, and details such as preferences are lost on the way. Whoever came up in the conversation may be named in it by their display name |
| **Viewer profiles** | Display name, preferred form of address, recorded gender, optionally a birthday, the language they wish to be addressed in, the time of their last message, plus categorised traits ("likes cats"), a free-text field, and the agent's standing assessment of the person (such as *trusted*, *stranger*, *sceptical*). Plus three **timestamps**: when someone first appeared, when they were last greeted, and when they were last wished a happy birthday |
| **Moderation records** | Infractions with time, reason and the measure taken, plus an escalation level and the time it last changed |
| **Community knowledge** | Facts about the community as a whole, not attributed to any person |
| **Invented past** | The persona's backstory: anecdotes the agent invented and marked as such itself, so as not to contradict itself later. They concern the character, not the viewers - the only person who usually appears in them is the streamer, who is set in the system prompt as her counterpart (for instance: how the two met) |
| **Voice profile** (optional) | Number series describing the **streamer's** voice - computed from six recordings they speak themselves during setup. **The recordings themselves are not stored**, only these numbers. They are biometric data (GDPR Art. 9). The profile never concerns viewers; their voices are checked but not kept, see [below](#the-voice-check-stays-on-the-device). The counter-check adds number series of foreign voices from an open corpus |
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
| Conversation history | up to a configurable volume. Once it is reached, the older part is condensed into a diary entry and taken out of the history; the verbatim text then stays in an archive for a period (default 14 days) and is deleted |
| Diary | indefinite. Entries older than a week (configurable) are combined into a retrospective; the replaced entries stay in an archive for a period and are then deleted, the retrospective remains |
| Viewer profiles | indefinite, until the operator deletes or the person concerned objects |
| Moderation records | the escalation level decays automatically over time; the records themselves remain |
| Community knowledge | indefinite |
| Invented past | indefinite |
| Voice profile | until the streamer deletes it in the menu. There are **no backup copies**: deleting removes it completely |
| Operational logs | a fixed period; older files are deleted at startup |

The **verbatim text** of a conversation is thus kept only for a period - what
remains is its condensed version in the diary. Up to 0.6.5.15a this said
older entries simply dropped out; since then they are summarised, and the
summary goes along with every request to the language model. Profiles, diary
and invented past persist - that is their purpose, and it is the point at
which an operator has to make a conscious decision.

**On scale.** Verbatim, the agent keeps a bounded excerpt, plus the most
recently condensed segments for the archive period. For comparison: the
streaming platform itself typically retains the complete chat log for weeks.
**The diary, however, reaches further back** than either - not verbatim, but
as a summary across the whole time of operation.

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
| **Speech recognition** | freely configurable, **local by default** | **Depends.** By default audio is turned into text on the CPU and never leaves the device. If the operator selects a cloud recognizer, the recorded audio goes there |
| **Memory maintenance** | freely configurable, **local by default**, off by default | **It depends.** The operator has to enable it explicitly, and by default it runs on the device. If the operator configures an endpoint, whatever the respective task needs goes there — for fact-keeping the utterance and an excerpt of the history, for condensation **an entire session segment** (see below) |
| **Main language model** | freely configurable | **Depends.** If the configuration points at a cloud provider, data goes there. If it points at a self-hosted model, it does not |
| **Speech output** | freely configurable | **Depends.** With a cloud adapter the spoken sentence goes there - and nothing else. With a local adapter it is synthesised on the CPU, and nothing leaves the device |

For the default case - a cloud provider for both the language model and speech
output, recognition local - that means: audio stays local, text goes out.

**Speech recognition only gained that choice in this version, and it points in
the uncomfortable direction.** Until now this passage said recorded audio never
leaves the device - there simply was no other option. Now there is one:
recognition runs through an exchangeable layer, like speech output, and one of
the selectable recognizers is a cloud service. Anyone who selects it transmits
**the audio itself**, not just derived text - and with it the speaker's voice and
whatever else is said in the room.

Three things bound that, and all three are construction rather than promise:

- **The default stays local.** Change nothing and nothing changes.
- **Only what the voice activity detector recognised as speech is transmitted**,
  and only from the onset of an utterance. A microphone nobody is speaking into
  sends nothing.
- **The wake word is always recognised locally.** The loop that listens
  continuously gets an on-device recognizer by construction, even when a cloud
  service is configured for everything else. Otherwise every utterance in the
  room would go out.

**But speech output need not go to the cloud.** It runs through an exchangeable
layer, and local voices are available that synthesise on the CPU - no account, no
network connection. Anyone who selects one transmits **nothing at all** for
speech output. That is a setting in the menu, not a different build.

As long as a cloud adapter is configured, two qualifications apply, and they cut
both ways. It receives **considerably less** than the language model: no
conversation history, no profile, no moderation data - only the one sentence to
be spoken. And it hangs off the voice path alone; chat replies are not read aloud
and never reach it.

Even so, "no personal data" claims too much. The voice path draws on the same
history as chat. If the agent talks about what is happening in chat, the spoken
sentence can contain names and details about third parties - and that sentence
is exactly what gets transmitted.

### A special case that deserves its own mention

Since 0.6.5.9a the memory maintenance can take on a task that differs from the
others: it forms the **beginning of the reply** while the speaker is still
speaking. To do so it reads the growing interim result of the recognition —
several times per utterance, not once afterwards.

Running locally, that stays on the device like everything else. But if the
operator points it at an endpoint, the difference from the other stages is
material: **fragments of live speech** leave the machine, not the finished
utterance. Anyone weighing confidentiality should assess this separately.

**The default is off.** The feature has to be enabled explicitly, and it
requires memory maintenance to be enabled as well. Which endpoint applies can be
set per task — one task may stay local while another runs over a service.

The software's status display carries its own colour for this: **blue means no
third party sees the content.** What counts is access, not ownership — a rented
server in a data centre counts as not confidential, because the operator can
reach the data.

### Further tasks, each switchable to its own endpoint

Memory maintenance is no longer a single activity but a bundle. These tasks
send something other than fact-keeping, and anyone weighing confidentiality
should look at them separately:

| Task | What goes out if it runs on an endpoint | Default |
| --- | --- | --- |
| **Condensing the history** | an **entire session segment** instead of a single utterance — in pieces, so it fits the context window. This is the largest amount ever transmitted at this point | on, once memory maintenance runs |
| **Opening of the reply** | in addition to the interim recognition result, **the description of the character itself** — personality, manner of speaking, rules. Around 6400 word fragments per call | off |
| **Invented past** | an **anecdote the agent invented and marked as such itself** - from the persona's backstory. The only person who usually appears in it is the streamer | off |
| **Settled check** | after **every reply**, as long as something is open in the newest diary entry: the open points and the last exchange - the message with its sender and the reply to it | on, once memory maintenance runs |
| **Interrupting** | while the agent speaks and someone talks in between: what the agent has said, the interim recognition result and an excerpt of the history | off |

**The character description is not a file about people**, and it is listed here
anyway: it is the one part the operator wrote themselves and may not think about
when configuring an endpoint. Whatever it mentions about third parties goes out
with it.

**The invented past concerns the character, not the audience.** It is the
persona's backstory - origin, experiences, quirks -, and the only human who
regularly appears in it is the streamer, set in the system prompt as her
counterpart. It does arise in conversation, though, and a viewer is not ruled
out. The agent keeps it so as not to contradict itself next time; that same
file can therefore grow over weeks.

### The pre-decision in chat

Since 0.6.6.40000-alpha a small pre-decision stands in front of the main
language model: should the agent reply to a chat message at all? For this,
**the persona's chat rules, the last 15 history entries and the new message**
go out - to the same language model as the reply, or to an endpoint of its own
if the operator sets one.

It is **always active** and by default runs through the main language model.
No new recipient is added that way: what goes there is an excerpt of what the
reply would receive anyway. An **endpoint of its own**, by contrast, is a
further provider with its own excerpt - whoever sets one should count it among
the processors.

### The voice check stays on the device

Since 0.6.6.50000-alpha the agent can learn the streamer's voice. The streamer
speaks six short recordings during setup; from them the **voice profile**
arises - number series, no recordings. After that, every utterance the
microphone picks up is compared with this profile. In the first stage it is
only measured: the result appears as a percentage in the operational log,
nothing is discarded.

Three things hold, and all three are by design:

- **None of it leaves the device.** The model that describes the voice is
  downloaded once and then computes on the CPU. There is no cloud path for this
  check - not even when a cloud service is set up for recognition.
- **Every voice is checked, only one is kept.** A foreign voice in the room is
  compared with the profile too, otherwise the check could not recognise it.
  Nothing arises for it, though: no profile, no number series, only a line with
  the percentage in the log, without a name and without content.
- **The profile belongs to the streamer and can be switched off.** The function
  is off by default, the profile only exists if they set it up themselves, and
  "Delete voice profile" removes it without a backup copy.

The viewers' way to object does not touch the profile - it contains no viewer
data.

After setup a **counter-check** shows how well the profile separates: the agent
compares it with sentences in its own voice and with 20 foreign voices. Two
local stores arise for this - the **sound store** with the sentences generated
once (if the agent's voice is a clone of a real person, it is that person's
voice) and the **counter-check** with the number series of the foreign voices.
These come from LibriSpeech, an open audiobook corpus under CC BY 4.0; they are
fetched **once** during setup via Hugging Face, the sound is discarded, only the
numbers remain. The dataset card asks not to identify the speakers - the
counter-check only asks whether someone is **not** the streamer. If the voice
output is a cloud service, three fixed sentences also go there **once**, after
asking.

### Fully local operation

As of this version **all four stages can run on the device**:

| Stage | Local? |
| --- | --- |
| Speech recognition | **selectable** - the default is local, and the wake word is always recognised locally |
| Memory maintenance | **selectable** - the default is local, and by default it does not run at all |
| Speech output | **selectable** - local voices are available |
| Main language model | **selectable** - the interface is an open standard, any self-hosted model speaking it will do |

That makes "nothing leaves the device" no longer a theoretical edge of the
scale but a configuration one can actually select. Up to the previous version
it was out of reach: the spoken sentence always had to go out.

For speech recognition, though, "selectable" means something different than in
the other three rows. There the default is a cloud service and local is the
alternative; here it is the other way round. The row is in the table because
the paper has to name the paths leading **out** of local operation too, not only
those leading in.

Two qualifications, so the claim holds:

- **Models have to be fetched once.** Recognition, speech output and a local
  language model need an internet connection during setup. **No user data**
  goes out in the process - things are only downloaded. Afterwards no
  connection is required to operate.
- **The client is a separate question.** The server has no platform connection
  (see [Data minimisation](#data-minimisation-by-design)); the link to Twitch,
  YouTube or Discord sits with the frontend. "Local" refers to processing
  inside the agent, not to the chat platform behind it.

### And the opposite pole: fully external operation

The same freedom of choice applies in the other direction. Anyone who points all
four stages at services runs the agent fully externally — and that is no
theoretical edge case but, for some, the obvious setup: a machine without a
graphics card, everything over endpoints.

Then **practically everything** leaves the device, and more than the sum of the
individual rows suggests:

- **the recorded audio** to the recogniser — not derived text, but the voice
  itself and everything else spoken in the room
- **the entire conversation history** with every turn, to the language model
- **the spoken sentence** to speech output
- **segments, character description and invented past** to memory maintenance,
  depending on which of its tasks are switched on

This can go to **four different providers**. Each sees a different slice, none
sees the whole — but each sees enough to recognise people again.

**One thing stays local by design even in this setting: the wake word.** The
loop that listens continuously always gets a recogniser on the device.
Otherwise every utterance in the room would go out, including those meant for
no one. "Fully external" therefore means: everything the agent processes — not
everything the microphone hears.

**Between the two poles lies the normal case.** The four stages are set
individually, and memory maintenance once more per task. The usual setups mix:
recognition local, language model and voice over services, memory maintenance
off. The paper names the poles because they mark the limits of what is possible
— not because anyone has to configure them.

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
endpoint. Together with a local voice and local recognition - the default -
nothing at all then leaves the machine: recognition, model and speech output all
run on the device. The
control layer is deliberately designed to work with small local models too - see
[model portability](technology.md#model-portability-and-speed).

### Four gradations

Between "everything at the provider" and "nothing leaves the device" lie two
middle tiers; the second is likely the most common in practice. Since speech
output can run locally, the bottom one is to be taken **literally**:

| Setup | Where viewer data goes |
| --- | --- |
| Language model at a consumer-tier provider | To the provider, on their terms - retention and possible use for training follow the plan |
| Language model over a business account with a data processing agreement, without logging and without training permission | To the provider, but as a processor acting on instructions |
| Language model self-hosted, speech output in the cloud | Nowhere - except the spoken sentence |
| Language model self-hosted, speech output local | **Nowhere.** Nothing leaves the device |

The table assumes recognition in its default, that is, local. Selecting a cloud
recognizer instead shifts **every** row: audio then goes out, regardless of
where the language model and speech output run. The bottom row no longer
applies.

The middle tier is where the processor question can be settled properly rather
than sidestepped: a contract, purpose limitation, no secondary use. The bottom
tier removes it altogether. The choice rests with the operator, and it is a
matter of configuration - not a different build.

## Objecting

Data subjects can object to their personal data being stored, via a chat
command. The command runs deterministically in the server and does not pass
through the language model at all.

**It does come from the frontend, though, not from the server.** The server
recognises it as a command type of its own, strictly separate from anything the
language model produces - but it has to be sent by the connection to the chat
platform. The one shipped with the software does so. Anyone building their own
must pass the command through, otherwise the objection route runs into nothing
while this paper promises it.

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
| Time of the last message | **retained** |

The dividing line follows a principle: what is cleared is what the agent has
*accumulated* about a person. What is retained is what operation needs in order
to keep working correctly at all - display name and form of address for correct
address, gender for pronouns, time first seen for attribution. Without a name no
attribution, without attribution no lock.

**The time of the last greeting stays for a reason that runs against first
impressions: it prevents intrusiveness.** It is the marker that someone has
*already* been greeted today - without it the agent would welcome them anew with
every message. It stores no content, only that something has already happened.
Deleting it would not bring less attention, but more.

**The time of the last message stays for the same reason.** It tells the agent
whether someone is in a conversation right now or has been away for long. Without
it the person would be treated as if they had just arrived. It stores no content,
only a point in time, and is overwritten with every chat message; no record of
past visits arises from it.

**The birthday marker, by contrast, is cleared along with the rest**, and that
is the counter-test of the same principle: it records when congratulations were
last given and presupposes a known birthday. Once that is cleared and locked,
the marker has nothing left to refer to - it would be a trace without the thing
it points at. What loses its basis does not stay.

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

**The same applies to moderation records**, and it is stated here because
nobody expects it under "objection": recorded violations with time, reason and
the measure taken remain. The escalation level drops on a timer of its own - but
that is the passage of time, not objection.

The reason is the same as for the classification, only plainer: a moderation
record that could be cleared on request would not be data protection but a reset
button for one's own conduct towards others. An objection is no licence to
disregard the chat rules.

**It is not a final state either, and that on two counts.** The escalation level
**fades on its own** - roughly a month per level without a new incident, and
then it is one level lower. Anyone who stays out of trouble long enough ends up
back at zero without anybody having to do anything.

And it can be **actively withdrawn**: a pardon lowers the level by one at once.
It may come from the agent itself - it is allowed to pardon as well as warn - or
from the operator. The record stays on file either way; what is withdrawn is its
**effect**, not the entry.

**That the agent may pardon on its own is a design decision, not a
convenience.** A moderation system that can only escalate is a one-way street:
every incident pushes the level up, and only time pushes it back.

The everyday case behind it is a small one. Someone spams emotes and gets a
warning - level one, without further consequence. They apologise, and the agent
withdraws it in the same conversation; the level drops to zero. Without that
route the warning would stand for a month, although the matter was settled in
the next sentence.

Which states the intention: **everyone gets a second chance**, and it should be
able to happen where the violation happened - in conversation, not as an
administrative act.

Moderation thus has the same mobility as the classification: it does not lock
in. What remains is the record - what changes is what follows from it.

### History, diary and invented past are left untouched

Objecting acts on the profile, not on the running history. That is not an
oversight but follows from what the history is: a **conversation transcript**,
not a record about a person. Its lines refer to one another. Remove one voice
and replies are left without questions while references run into nothing - and
those very references are what make a reaction fit. A purged history would not
be a more data-minimal history but a broken one.

Up to 0.6.5.15a the retention itself defused this: the history rolled over
within hours in an active chat, and eviction did faster what an erasure
request would first have had to set in motion. **That no longer holds.** Since
then the history does not roll; it is cut once it reaches a configurable
volume, and the older part is condensed into a diary entry. The verbatim text
disappears after the archive period (default 14 days) - the summary remains,
and with it whoever is named in it.

**Objecting does not reach the diary, nor the invented past.** It applies to
what the agent has *accumulated* about a person in chat - preferences, traits,
the free-text field of the profile. The diary is no such collection but a
summary of what happened, and the condensation itself limits what it can hold
about individuals: up to around 50,000 word fragments become a few hundred,
and preferences do not survive that. What remains is at most that someone
took part in the conversation under their display name.

**And that name is no fixed reference.** Anyone can change their display name
at any time; unlike the streaming platform, the agent does not recognise a
person under a changed name. An old diary entry can then no longer be linked
to any current profile.

The invented past is the persona's backstory and usually does not concern
viewers at all. Both are **not time-limited**; anyone who still wants an entry
there removed turns to the operator, see below.

The third reason weighs more than the first two, and it applies to the
history unchanged: an objection that took the
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
- **No sensitive details in chat.** The chat rules, which have to be accepted
  before writing for the first time, explicitly forbid sharing sensitive data
  such as an address. What the agent never gets to read, it can neither store
  nor condense.

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
  the necessary agreements in place? There can be **four** -- recognition,
  language model, voice and memory maintenance are set separately and need not
  belong to the same company. If the [pre-decision in
  chat](#the-pre-decision-in-chat) gets an endpoint of its own, it is a fifth. Running with a local language model shrinks this
  question considerably; fully local operation removes it.
- **Access and erasure.** How is a request answered, and who gets at the files?
- **Retention.** Profiles persist indefinitely. Is that intended?

The technical preconditions - local processing where possible, objection by
command, no platform integration - are in place. The decisions about them are
not, and they cannot be delegated to software.

## Recognisability of the AI

Replies produced by a language model are visibly marked in chat. That is the
default; the operator can switch it off, because the underlying obligation is
regional and does not exist everywhere. Where it does exist, it remains his
obligation — the software does not take it off his hands, it merely fulfils it
for as long as he leaves the marking on.

Three design decisions behind that are worth mentioning:

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
