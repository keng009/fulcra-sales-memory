# Coexistence with other loggers — independent systems, no duplicate CRM notes

**Status: ACCEPTED (2026-09-15, Nick — "treat this skill as independent … prevent duplication if someone has both").**

## Context

This packet is one of several things that can write a conversation into the same CRM: its own siblings (dealflow, raise), a user's private sync (the maintainer runs an Otter → Attio logger and a Boardy → HubSpot logger), CRM-native auto-loggers, and humans. Some users run only this packet; some run two or three systems against one CRM. Each system must be complete on its own — none may assume another exists — and when two do exist, the same conversation must not appear twice in the CRM.

Memory is not the problem: sibling packets store into disjoint namespaces (`/dealflow/`, `/raise/`, `/sales/`) and the same real-world conversation legitimately appears under different lenses. The CRM is the shared destination, and today's live test showed the failure concretely: a call logged by the private Otter → Attio sync and again by this packet's sync, as two notes with different key formats.

## Decision

1. **Independence.** This packet never reads or depends on another system's state (files, watermarks, receipts). Its sweep, memory, and sync are complete with the Fulcra connector plus whatever sources and CRM the user connects. Conversely, nothing here is required by any other logger.
2. **Cross-system dedupe on the shared destination.** Before writing any CRM note, the scan of the matched contact's existing notes checks **titles and body previews for the touchpoint's source id in any format** — the transcript id, calendar event id, or mail-thread id, whether it appears as `[touch:<id>]`, `[otter:<id>]`, in a `Source:` line, or inside a URL. Any hit means another logger already recorded this conversation: skip the write, count it as skipped-duplicate in the receipt, and name the note in the digest. The packet's own key remains the primary match; the source id is the cross-system match. This extends the import path's cross-key guard to the sync path.
3. **Make our notes recognizable to others.** Every note this packet writes carries the source id in the `Source:` line (`otter transcript <id>`, `gmail thread <id>`, `calendar <event-id>`), so any other logger that scans for a source id finds ours. The maintainer's private loggers adopt the same scan (their runbooks are updated alongside this ADR).
4. **Never delete or rewrite another system's note.** A duplicate that already exists stays; the digest names it so the user can tidy by hand.
5. **Memory-side overlap is not deduplicated across packets.** Sibling namespaces are separate products; a conversation stored under two lenses is intentional. Within one packet, the existing per-destination scans apply.

## Consequences

- crm-sync principle 5 gains the source-id scan; Capture step 9 and Tend rule 6's CRM bullet apply it; the receipt line counts cross-system skips as skipped-duplicate.
- Siblings adopt the same rule (engine-level, ADR-0007 cherry-pick).
- A testing.md row: a contact holding a third-party note for the same transcript → the sync skips and names it.
