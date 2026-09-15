# Email as a sweep source — opt-in, read-only, thread-keyed

**Status: ACCEPTED (2026-09-15, Nick — "I want everything built and published").**

## Context

For a founder selling, the pipeline arrives by email: inbound signups, demo requests, replies to outreach, intros. Calendar and transcripts (the sweep's sources until now) see the meetings but not the threads around them. The packet's private predecessor planned Gmail as a second-stage source and never built it; this ADR builds it once, in the public skill.

Email is a wider privacy surface than a calendar or a meeting recorder — a mailbox holds everything — so it is not a default source. It is opt-in per pipeline, read-only, and narrowly filtered.

## Decision

1. **Opt-in only.** `email` becomes a third value for the auto-log sources line (`- auto-log[gobeyond]: transcripts, calendar, email`) and for an explicit ask ("sweep my email"). No line, no `email` value → the skill never reads mail. The mailbox is read by capability: any connected mail tool that can search threads by date and read a thread (Gmail's connector: `search_threads`, `get_thread`).
2. **A mailbox belongs to a business.** Multi-pipeline sellers map mailboxes: `- email-pipeline: nick@gobeyond.global=gobeyond`. A thread's pipeline is the mailbox's unless an existing relationship file says otherwise; with one pipeline registered no line is needed.
3. **Narrow eligibility, then the auto-log gate.** A thread qualifies as a conversation only if: the counterparty is external (not the user's own domains, not `noreply`/`no-reply`/notification senders, not a mailing list or newsletter), it resolves to exactly one person (address → existing relationship, else name from the header), and the thread carries real correspondence — at least one message *from* the user OR an inbound message addressed to the user personally with substantive content (an intro, a question, a reply). Automated product notifications ("new signup: …") are **signals, not conversations**: they are listed in the digest under "inbound signals" and never logged as touchpoints unless the user says so. Everything eligible then passes rule 6's ordinary gate (one person, determinable pipeline) or parks.
4. **One touchpoint per thread**, channel `email`, key `touch:<tool>-thread:<thread-id>` (the contract's existing messaging-thread form; for Gmail, `touch:gmail-thread:<id>`), evidence `gmail thread <id>[, auto-log]`, summary distilled from the thread (2–5 sentences; never the raw mail). Later replies on an already-captured thread are not re-logged — the key matches and the sweep skips it; a materially new development is captured by the user with "log my email with <name>", which appends a dated entry on the same relationship with a date-form key.
5. **Never a send.** Reading mail never marks, replies, forwards, labels, or drafts anything in the mailbox. Follow-ups a thread implies stay in the summary (backfill hygiene) and surface in the digest.
6. **Disclosure.** The README's privacy section states that mail is read only when the user has opted a pipeline into `email` or asks for an email sweep, and what is read (thread metadata and bodies in the window, to distill summaries).

## Alternatives considered

- **Email on by default.** Rejected — the privacy surface is too wide to assume.
- **Log every inbound thread.** Rejected — most mail is noise; the signals-vs-conversations line keeps the memory a record of relationships, not an inbox mirror.
- **Notification emails as touchpoints.** Rejected for auto mode (they would fabricate "conversations" with people who never wrote); kept as digest signals the user can promote in a sentence.

## Consequences

- Sweep watermarks gain an `email` source line; the receipt lists it. The scheduled task prompt names the mail tool.
- `references/extending.md` gains the mail-tool capability slots (search by date, read thread, stable thread id) so other mail connectors qualify the same way.
- Promotion needs testing.md rows: an eligible thread auto-logged, a notification correctly surfaced as a signal and not logged, a noise sender filtered, a re-sweep skipping the captured thread.
