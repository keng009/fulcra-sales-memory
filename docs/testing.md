# Live-test matrix

Dated, sanitized record of what has actually been tested against live services. "Tested" claims in the README trace here. No user data appears below.

## Engine provenance (inherited design evidence, NOT this repo's behavior)

This packet was forked 2026-08-31 from [fulcra-dealflow-memory](https://github.com/keng009/fulcra-dealflow-memory) at contract v3.1 plus review rounds 5–8. That repo's [testing matrix](https://github.com/keng009/fulcra-dealflow-memory/blob/main/docs/testing.md) carries dated live evidence for the shared engine design: Fulcra dual-write path, per-destination self-healing dedupe, snapshot→commit→veto end to end, stable calendar-event-id keys (first + second commit, zero duplicates), CRM-note import + circularity guard against Attio, injected partial-failure healing, and a live sweep run (first-run window, park-once, watermark-last ordering).

Per ADR-0007, that evidence supports the **design** of this packet's engine — it is never evidence for this packet's **behavior**. The flavors differ (`/sales/` namespace, `Sales Touchpoint` type, seller-facing vocabulary), and every claim about this packet must trace to a row below.

## This flavor — live runs

| Surface | Status |
|---|---|
| `sales-demo` full session (zip upload → snapshot/capture → save → prep brief) | **Pass — 2026-09-15** (below) |
| `sales-memory` snapshot → commit → veto on a real account | **Pass — 2026-09-15** (below) |
| `/sales/` folder init, `Sales Touchpoint` create-if-absent, dual write + read-back | **Pass — 2026-09-15** (below) |
| CRM adapters under this flavor — Attio Tier W sync + dedupe + import guards; email-touchpoint sync | **Pass — 2026-09-15** (below); Attio task creation, Notion, Affinity still untested |
| Gated CRM contact creation, slot 8 (ADR-0008 — this flavor's one engine divergence) | **Untested** |
| Messaging capture (paste tier) under this flavor | **Untested** |
| `crm-setup`: Attio inspect + stage read-back + mapping record | **Untested** (designed against the live connector's tool list; first run pending) |
| `crm-setup`: optional list creation via `create-list` | **Untested** |
| Unattended auto-log (ADR-0009) — eligible items commit without a yes, receipt + digest | **Pass — 2026-09-15** (below; manually triggered, scheduled trigger pending) |
| Unattended auto-log — ineligible items park | **Pass — 2026-09-15** (below; calendar-only item with undeterminable pipeline; the no-summary brokered variant not yet observed) |
| Unattended auto-log — dead Fulcra → STOP, zero writes | **Untested** |
| Email as a source (ADR-0010): eligible threads auto-logged; notifications surfaced as signals, not logged; noise senders filtered | **Pass — 2026-09-15** (below); re-sweep skip pending the next run |
| HubSpot Tier W: note write (key on first body line) + association + read-back via associated-notes scan | **Pass — 2026-09-15** (below); tasks untested |
| Coexistence (ADR-0011): sync skips a conversation another logger already recorded (source id found in a third-party note) | **Untested as a rule** — adopted after today's live runs observed the duplication it prevents (two loggers, two notes, same call); first row on the next sync that meets a third-party note |
| Unattended auto-log — revocation (line removed → next run digests instead) | **Untested** |
| Scheduled trigger built by the skill (Tend rule 7): task created via the desktop app's scheduling tool, run fires and completes a sweep | **Pass — 2026-09-15** (below) |

## 2026-09-15 — Snapshot → Commit → Veto, live end to end (Claude Google Calendar connector + Otter + official Fulcra connector)

Run on the maintainer's real account against a fresh `/sales/` namespace (no folder, no type). The snapshot was presented with its commit ledger on 2026-09-02; the maintainer's explicit yes came 2026-09-15 and the commit ran on that yes (ADR-0005). The account has two registered pipelines, so this run also exercises multi-pipeline support.

| Step | Result |
|---|---|
| 30-day snapshot sweep, weekly chunks, dual-surface calendar (Fulcra-native absent → Claude calendar connector), transcripts listed | Pass — a real month reduced to 9 sales-shaped conversations, 7 ambiguous items, the rest filtered (internal, declined-without-source, personal, travel/holds) |
| Commit ledger before the yes (Will save / Parked, split by pipeline) | Pass — the yes covered exactly the ledger's lines |
| Veto-set-first on an uninitialized namespace | Pass — folder absent → empty set, then initialization |
| Folder init (README, INDEX, handoff with `## Pipelines`, review queue with 7 parked rows) + `create_data_type` (`base_type: "moment"`) | Pass — type stored as `MomentAnnotation/<uuid>` |
| Dual write: 8 relationship files (`Pipeline:` lines) + 9 typed records (`pipeline` field), all keyed `touch:<transcript-id>` | Pass — including one person with two touchpoints; one record carries a source-derived `stage_noted` |
| Backfill hygiene | Pass — zero open follow-ups created; every `evidence` names `otter transcript <id>, calendar <date>` |
| Declined-events rule on real data | Pass — a declined invite with a transcript proving the call happened was KEPT |
| Read-back | Pass — all 9 payloads round-trip intact via `get_records` (pipeline and stage fields included); no read lag observed |
| Veto → tombstone | Pass — one committed touchpoint vetoed: relationship file rewritten as a new version without the entry (earlier version retained and listed as archived), key added to `## Vetoed keys`; the typed record remains stored and is excluded by the read filter |

Still untested from this flow: the release-ZIP upload journey end to end (#1 — human step); CRM slot 8 (#4); the scheduled sweep (#5); slot 6 (#6); messaging capture (paste tier).

## 2026-09-15 — Attio CRM path under this flavor (official Attio connector, real workspace, real contacts)

Run on the maintainer's own workspace against two real contacts from the `gobeyond` pipeline, on the maintainer's explicit choice of real contacts over a throwaway. Attio has no connector delete, so the two sync notes remain.

| Test | Result |
|---|---|
| Contact match, email first then name | Pass — one contact matched cleanly; the other existed as TWO person records with different emails, and the email from the calendar attendee list selected exactly one — the rule exists for this case |
| Dedupe scan before writing (`search-notes-by-metadata` filtered to the record) | Pass — third-party notes present, no title carrying the touchpoint key → write proceeds |
| Sync note write (`create-note`, person as parent): title ends with `[touch:<key>]`, body = Summary / Follow-ups / `Source:` trio | Pass — body read back byte-identical via `get-note-body` |
| Dedupe re-run | Pass — the key is found in the note title on both contacts; a second write is skipped |
| Import path, cross-key guard: third-party notes whose titles carry a transcript id already stored as `touch:<transcript-id>` | Pass — recognized as the same conversation, not imported under a second key |
| Import path, circularity guard: this system's own sync notes (title carries `[touch:`) | Pass — refused as own output on the re-scan |
| Tasks (one per follow-up) | Not exercised — both touchpoints were backfills, which never carry open follow-ups; the task path remains untested under this flavor |
| Delete | Confirmed absent — cleanup is manual in the Attio UI, as the veto disclosure says |

Side observation, not this packet's behavior: a separate auto-logger in the same workspace had written each third-party note twice (seconds apart) — the cross-key guard handled both copies identically.

## 2026-09-15 — First unattended auto-log run (ADR-0009, Tend rule 6), manually triggered

Run on the maintainer's real account minutes after the standing yes was written (`auto-log[<pipeline>]: transcripts, calendar` for both registered pipelines). Window: from the last committed activity to now (first run, stated). No CRM mapping existed under `## Preferences`, so no CRM writes were attempted — correct per the rule. The scheduled trigger itself remains the residual (#5).

| Step | Result |
|---|---|
| Standing yes read from `## Preferences` before anything else; veto set loaded | Pass |
| Sources swept: transcripts (11 in window) + calendar (two weeks) | Pass — internal team meetings, community events, personal items, and task-like entries filtered out |
| Eligibility applied item by item | Pass — 5 items met all three conditions (real transcript summary, one resolved person, pipeline unambiguous from the transcript); 3 did not |
| Brokered intros (two Boardy-booked) | Pass — identity taken from the non-broker attendee email, pipeline classified from the transcript, not the broker's blurb |
| Calendar-only item with an external attendee email but no transcript and no existing relationship | Pass — pipeline not determinable under two registered pipelines → parked, not committed (the looser eligibility path's designed edge) |
| Auto-commit writes: 3 new relationship files, 1 existing file updated newest-first, 5 typed records, INDEX updated | Pass — every `evidence` ends in `, auto-log`; two records carry source-derived `stage_noted`; no tasks and no open follow-ups created |
| Read-back | Pass — all 5 payloads round-trip via `get_records` with `pipeline` and `auto-log` marker intact |
| Receipt + watermarks written last, after full resolution | Pass — one `## Sweep log` line (committed 5 / parked 3 / skipped-duplicate 0 / failed none); both source watermarks advanced to the run's start time |
| Digest posted | Pass — presented to the user after the run |

Still untested for auto mode: the dead-Fulcra STOP (can only be caught, not staged) and revocation (line removed → next run digests instead).

## 2026-09-15 — Scheduled trigger (Tend rule 7), live

A `sales-memory-sweep` task was created through the desktop app's scheduling tool with the self-contained prompt from `references/scheduling.md` (weekdays, late afternoon, the user's timezone), then fired once manually — the same path a scheduled fire takes. The fresh session loaded the skill file from disk, ran Tend rules 5–6 against the live store minutes after a manual sweep had advanced the watermarks, and completed.

| Step | Result |
|---|---|
| Task creation via the scheduling tool, cron in local time, single task (no duplicate) | Pass |
| Fresh-session run loads the skill from the repo path and follows it (no "run from memory") | Pass — run status succeeded |
| Sweep with nothing new since the previous watermark | Pass — see the receipt line the run appended and the watermark advance (verified by reading `handoff.md` back) |
| Notification on completion to the creating session | Pass |

Residual: the first run of a task pauses for connector approvals, which then stick to the task — the skill tells the user this when it creates the schedule.

## 2026-09-15 — `sales-demo` through Claude's real skill-upload UI (the release gate)

Run by the maintainer in Claude's chat app, on the real account — the harder case, since `/sales/` already held memory from earlier the same day.

| Step | Result |
|---|---|
| Zip upload (Customize → Skills → + Create skill → Upload a skill) | Pass on the second try — see the finding below |
| Frontmatter accepted; skill listed as `sales-demo` | Pass |
| Preflight, catalog moment, source detection, weekly-chunk sweep of 30 days | Pass — read-only, said so |
| Existing memory recognized as the baseline: 14 stored conversations and the day's sweep watermark used, nothing re-saved | Pass — the dedupe scan working through the real UI ("nothing new gets duplicated") |
| Commit ledger: one new Will-save line (a brokered intro classified from its transcript), two parked | Pass — the user ruled on the parked items in-session (one assigned to a pipeline pending details, one dropped as personal) and the skill honored both |
| Save: relationship file, typed record, INDEX line; read-back of both representations | Pass |
| Prep brief generated from the stored data, with an explicit "confirm this — may be a mis-transcription" on an uncertain product name | Pass |
| Outro: the five daily phrases, the cross-assistant test prompt, the no-migration note | Pass |

**Finding (fixed in the docs):** a zip built with PowerShell's `Compress-Archive` on Windows stores entry paths with backslashes (`sales-demo\SKILL.md`); Claude's uploader rejects it with "Zip file contains path with invalid characters". Zips built with forward-slash entries upload fine; the release workflow (Linux) always produces forward slashes. The quick reference now says: on Windows, use the release zip.

**Finding (fixed in the skills):** "preflight" leaked into spoken output; added to the plain-words rail.

## 2026-09-15 — Email as a source (ADR-0010), first live run

Run on the maintainer's real mailbox (the Gmail connector) minutes after \`email\` was added to both pipelines' auto-log lines, over a two-week window. Manually triggered; the scheduled task's next run will exercise the re-sweep skip.

| Step | Result |
|---|---|
| Mailbox read by capability (search threads by date, read thread), read-only | Pass — no label, reply, or send |
| Noise filtered: newsletters, event broadcasts, a scheduler's own status mails, calendar invitation mail | Pass |
| Signals-vs-conversations line: a product notification (an evaluator lead) and a broker's intro offers/debriefs surfaced as **signals** in the digest, not logged | Pass |
| Eligible conversations: external counterparty, one resolved person, real correspondence | Pass — 8 threads across 6 people; 5 existing relationships updated newest-first, 1 new relationship created |
| Pipeline classified from content first (a thread in the Go Beyond mailbox whose content was a Fulcra engineering conversation went to \`fulcra\`, not the mailbox default) | Pass — the mailbox default is a fallback, not a rule |
| Keys \`touch:gmail-thread:<id>\`, channel \`email\`, evidence \`gmail thread <id>, auto-log\`, one touchpoint per thread | Pass — read back via \`get_records\` |
| CRM sync of email touchpoints where the pipeline had a CRM mapped | Pass — Attio notes on three matched contacts, dedupe scan first |

## 2026-09-15 — HubSpot Tier W (official connector, real portal, real contact)

| Step | Result |
|---|---|
| Contact search by name (\`search_crm_objects\` CONTACT) | Pass |
| Dedupe scan: list the contact's notes (\`search_crm_objects\` NOTE, \`associatedWith\` the contact) and check body previews for the key | Pass — a third-party note present, no key → write proceeds |
| Note write (\`manage_crm_objects\` createRequest, objectType \`notes\`, key on the first body line, \`hs_timestamp\`, association to CONTACT) | Pass — created and associated in one call |
| Read-back: the key is findable in the associated note's body preview | Pass |
| Tasks | Not exercised |

Release gate met: both required runs recorded above. v0.1.0 tagged 2026-09-15.
