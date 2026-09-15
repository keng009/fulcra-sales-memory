# Live-test matrix

Dated, sanitized record of what has actually been tested against live services. "Tested" claims in the README trace here. No user data appears below.

## Engine provenance (inherited design evidence, NOT this repo's behavior)

This packet was forked 2026-08-31 from [fulcra-dealflow-memory](https://github.com/keng009/fulcra-dealflow-memory) at contract v3.1 plus review rounds 5–8. That repo's [testing matrix](https://github.com/keng009/fulcra-dealflow-memory/blob/main/docs/testing.md) carries dated live evidence for the shared engine design: Fulcra dual-write path, per-destination self-healing dedupe, snapshot→commit→veto end to end, stable calendar-event-id keys (first + second commit, zero duplicates), CRM-note import + circularity guard against Attio, injected partial-failure healing, and a live sweep run (first-run window, park-once, watermark-last ordering).

Per ADR-0007, that evidence supports the **design** of this packet's engine — it is never evidence for this packet's **behavior**. The flavors differ (`/sales/` namespace, `Sales Touchpoint` type, seller-facing vocabulary), and every claim about this packet must trace to a row below.

## This flavor — live runs

| Surface | Status |
|---|---|
| `sales-demo` full session (zip upload → snapshot/capture → save → prep brief) | **Untested** |
| `sales-memory` snapshot → commit → veto on a real account | **Pass — 2026-09-15** (below) |
| `/sales/` folder init, `Sales Touchpoint` create-if-absent, dual write + read-back | **Pass — 2026-09-15** (below) |
| CRM adapters under this flavor — Attio Tier W sync + dedupe + import guards | **Pass — 2026-09-15** (below); Attio task creation, HubSpot, Notion, Affinity still untested |
| Gated CRM contact creation, slot 8 (ADR-0008 — this flavor's one engine divergence) | **Untested** |
| Messaging capture (paste tier) under this flavor | **Untested** |
| `crm-setup`: Attio inspect + stage read-back + mapping record | **Untested** (designed against the live connector's tool list; first run pending) |
| `crm-setup`: optional list creation via `create-list` | **Untested** |
| Unattended auto-log (ADR-0009) — eligible items commit without a yes, receipt + digest | **Untested** |
| Unattended auto-log — ineligible brokered intro parks | **Untested** |
| Unattended auto-log — dead Fulcra → STOP, zero writes | **Untested** |
| Unattended auto-log — revocation (line removed → next run digests instead) | **Untested** |

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

First release is gated on at least: one full `sales-demo` session through Claude's actual zip-upload UI (still pending), and one `sales-memory` snapshot→commit→veto run on a real account (done, above) — both recorded here (dated, sanitized).
