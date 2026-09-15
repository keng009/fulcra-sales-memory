# CRM adapters — capability tiers and per-CRM guidance

How the `sales-memory` skill works with a connected CRM. Read this file when CRM sync is on and a touchpoint is about to be written to the CRM, or when deciding what a connected CRM can do. The dedupe key and provenance formats are defined in `conventions.md` (same folder) — that file is canonical; nothing here overrides it.

CRM involvement is optional and detected, never required. If no CRM tools are connected, none of this applies and the skill should not mention CRMs at all. A user who *wants* a CRM but has none connected — or has one with no pipeline structure yet — is handed to the `crm-setup` skill in this packet, which walks through connecting one and giving it stages that mirror this skill's vocabulary.

## Capability tiers — a CRM qualifies by what its tools can do, not by name

Map whatever CRM tools are connected onto eight capability slots (5–8 are optional):

| Slot | Capability | Needed for |
|---|---|---|
| 1 | Search contacts (by email, then name) | Everything below |
| 2 | List a contact's notes (titles or first lines) | Dedupe scan; tracked-vs-untracked check |
| 3 | Read a note's body | CRM-note import (`touch:<crm>-note:<id>` keys) |
| 4 | Create a note on a contact | One-way sync |
| 5 | Create a task linked to a contact (optional) | Follow-ups as tasks |
| 6 | Associate a note with additional objects — deal/opportunity/company (optional) | Note placement where the tracker's users actually look |
| 7 | Delete a note (optional, rare) | Veto cleanup of an already-synced copy — where absent, the veto gives the manual step |
| 8 | Create a contact (optional, gated — this flavor only, ADR-0008) | Capturing a live-logged lead who is not in the CRM yet; off by default, per-contact confirmation, minimal fields |

- **Tier R (slots 1–3, read-only)**: powers the snapshot's tracked-vs-untracked check and CRM-note import into memory. A Tier R CRM is a full read source — never offer sync for it. Example: HubSpot's official Claude connector.
- **Tier W (slots 1–4, +5 where present)**: everything above plus one-way sync (notes, optional tasks).

A CRM not named in this file still works if its connector fills the slots — follow the principles and note format below, apply the untested-CRM honesty rule, and see "Add your CRM" at the end of this file to make it official.

## Principles (all CRMs)

1. **Sync is one-way, Fulcra → CRM.** The CRM is the user's system of record for pipeline; Fulcra holds the narrative files and typed records. Sync pushes touchpoint notes into the CRM, and nothing sync-related ever flows back. (The separate CRM-note IMPORT — slots 2–3 — reads existing notes into memory under `touch:<crm>-note:<id>` keys with the circularity guard; that is a read path with its own consent, not a reverse sync.) A failed CRM write never blocks or rolls back the Fulcra write — the Fulcra side is already complete before the CRM write starts.
2. **Contact matching: email first, then name.** Search the CRM's people/contacts for the person's email address. If the email is unknown or finds nothing, search by full name. If neither finds exactly one plausible match, skip the CRM write and tell the user — do not guess between multiple matches.
3. **Never create contacts silently.** Contact creation exists only as the gated slot-8 flow (ADR-0008): off by default, enabled only by a recorded `## Preferences` choice in `/sales/handoff.md`, confirmed per contact, minimal fields (name, email, company — nothing else), the touchpoint note attached immediately, and live-logged leads only — commit/backfill imports never create contacts. Outside that flow, if the person is not in the CRM, skip the write and say so; the user can add them there and say "retry the CRM sync for <person>".
4. **Never edit CRM fields, stages, amounts, owners, or lists.** The only writes are: a note attached to an existing contact, and (where the CRM supports it and the user wants it) tasks for follow-ups. No other CRM object is created or modified.
5. **Dedupe by key before every write.** Every touchpoint has a deterministic key in one of the canonical forms defined in `conventions.md` — date-form `touch:<person-slug>:<YYYY-MM-DD>` (with `-2`/`-3` ordinals), calendar `touch:cal:<event-id>`, transcript `touch:<transcript-id>`, CRM-note `touch:<crm>-note:<note-id>`, or messaging-thread `touch:<tool>-thread:<id>`. Before writing a note, fetch the matched contact's existing notes and scan their titles for the key string. If any title contains it, skip the write and tell the user that touchpoint is already in the CRM. Never assume a write happens exactly once.
6. **Degrade gracefully.** If a CRM tool is missing or a call errors, say exactly which step failed and what was skipped, confirm that the Fulcra write succeeded, and offer to retry. Retrying is safe because of the dedupe scan. Never fake success and never report a CRM write that was not confirmed.

## Note format (all CRMs)

- **Title**: human-readable prefix + the dedupe key as a bracketed suffix:

  `Call with Jordan Lee — 2026-08-20 [touch:jordan-lee:2026-08-20]`

  The bracketed key suffix is the load-bearing part — it must contain the exact key string, because the dedupe scan looks for that string in existing note titles. The prefix is for humans and may vary.

- **Body**:

  ```
  Summary: 2-5 sentences on what was discussed and any decisions.
  Follow-ups:
  - Send the Q3 memo
  Source: sales-memory | user account, calendar 2026-08-20 | 2026-08-20T17:30-04:00
  ```

  Write `Follow-ups: none` when there are none. The `Source:` line is the provenance trio from `conventions.md` (producer | evidence | timestamp) and marks the note as written by this skill, not by a human.

- **Follow-ups as tasks**: where the CRM supports tasks linked to a contact, offer to create one task per follow-up (content = the follow-up text) in the same pass as the note. Tasks are only ever created alongside a new note write — when the dedupe scan skips the note, it skips the tasks too, so tasks need no key of their own.

- **Note placement (slot 6 — optional, designed, untested)**: where the CRM's note primitive supports additional parents or associations, the note MAY also be associated to the most relevant *existing* deal/opportunity object — usually the account's deal or opportunity row (HubSpot deals; Attio deal-list records; Affinity opportunities). Which object is relevant is asked when ambiguous, never guessed (park in the review queue if unclear). Associating a note is not editing the object: fields, stages, amounts, and owners remain untouched (ADR-0004), and nothing is ever created to have somewhere to put the note.

## Attio (Tier W — engine-tested in the sibling packet)

The engine's reference implementation: tested against a live Attio workspace via the official Attio connector — in the sibling packet (see its testing matrix; design evidence only per ADR-0007). Under this flavor Attio is UNTESTED like every other CRM — say so on first use and verify the first write by reading it back.

| Slot | Attio tool |
|---|---|
| Search contacts | `search-records` (people object) |
| List a contact's notes | `search-notes-by-metadata` (filtered to the record) |
| Read a note body | `get-note-body` |
| Create a note | `create-note` (person record as parent) |
| Create a task | `create-task` (linked to the contact) |
| Delete a note | none — removal is manual in the Attio UI (the veto says exactly which note) |
| Create a contact (slot 8, gated) | `create-record` (people object) — only via the ADR-0008 flow; untested until a testing.md row exists |

- **Contact lookup**: search people records by email, then by name (connector tool: `search-records` on the people object, or the equivalent contact-search tool your connector exposes).
- **Idempotency mechanics**: Attio notes have **no custom fields**, so there is nowhere structured to put an idempotency key. The key therefore lives in two plain-text places: the note **title suffix** (`[touch:jordan-lee:2026-08-20]`) and the **`Source:` body line**. The dedupe check is the title: before writing, list the matched person record's existing notes (`search-notes-by-metadata` filtered to that record, or the equivalent) and scan each title for the exact key string. Found → skip, tell the user. Not found → write.
- **Note creation**: create the note with the matched person record as its parent (`create-note`), title and body per the format above. Attio note bodies accept markdown.
- **Tasks**: Attio supports tasks linked to records — create one per follow-up (`create-task`) linked to the contact, if the user wants tasks.
- Connector tool names can vary slightly between connector versions; match by capability (contact search, list notes on a record, create note, create task) if the names above are not present.
- The Attio connector has no delete tool. If the user wants a synced note removed, they delete it in the Attio UI.

## HubSpot (Tier R via the official connector; Tier W untested)

Same principles; not yet verified against a live workspace.

- **Important**: Claude's official HubSpot connector is **read-only** (verified 2026-08-21) — it cannot create notes, so it cannot carry this sync. HubSpot sync applies only when the user has a separate **write-capable HubSpot MCP server** connected. Detect by capability (can it create a note engagement?), not by connector name; with only the read-only connector present, say so and skip HubSpot sync entirely. The read-only connector still fills slots 1–3, so HubSpot works fully as a Tier R read source (tracked-vs-untracked check, CRM-note import).
- **Closest primitive**: a note engagement associated with a contact.
- HubSpot notes may not have a separate title field. If the tool's note primitive has no title, put the key as the **first line of the note body**, and run the dedupe scan against whatever note field the tools return when listing a contact's notes. The rule generalizes: the key must live in a field the tools can both write and read back.
- On first use, verify the round trip: after creating the first note, read the contact's notes back and confirm the key is findable. If it is not, stop syncing and tell the user dedupe cannot be guaranteed with this setup.

## Notion (Tier W — designed for, untested)

Same principles; not yet verified. Notion's official connector is read/write.

- Many founders run their lead tracker as a Notion database of contacts. **Ask the user which database holds their leads and customers** — never guess; it is searched for the contact match only.
- **Scope (keeps the "notes and tasks only" promise honest)**: the ONLY write is a block appended to the matched contact's existing page — the note-equivalent. Never create pages or rows in the user's databases, and never add properties to their schema; both count as creating CRM objects, which this sync never does. If the user's setup has no per-contact page to append to, say so and skip Notion sync rather than inventing structure.
- Put the key in the first line of the appended block. Dedupe scan = read the contact page's existing blocks and look for the key. Verify the first write by reading it back (as with any title-less primitive).

## Affinity (Tier W — designed for, untested)

Same principles; not yet verified.

- Affinity has an official **read/write** Claude connector (verified 2026-08-21) that supports creating notes on records. Affinity is the VC-native CRM — rarely a seller's system of record — so it sits last in this repo's founder-first testing order (see ROADMAP).
- **Closest primitive**: a note attached to a person. If the note primitive has no title field, use the first-line-of-body placement and read-back verification described under HubSpot.

## Say so in conversation

When syncing to ANY CRM under this packet — Attio included — state the status honestly before the first write, in words like: "CRM sync was engine-tested with Attio in the sibling packet; under this packet <CRM> is untested — I'll verify the first write by reading it back." Then actually do the read-back. Never present an untested integration as tested, and never present the sibling's evidence as this packet's.

## Add your CRM — the 10-minute promotion protocol

(For adding a messaging app, a notetaker, or a calendar surface instead, see `extending.md` — same spirit, per-source slots.)

Anyone with a CRM connector can add their CRM to this file and promote it to `tested`:

1. **Map the slots.** List your connector's tools and fill the capability slots above. Slots 1–3 only → your CRM is Tier R (still valuable — say so in its section). No slot-1 tool → the CRM can't participate; stop here.
2. **Run the write test** (Tier W): pick one contact you own, ask the skill to log a touchpoint and sync it. Verify: the note lands with the key in its title (or first body line for title-less primitives), and the body follows the Note format above.
3. **Run the dedupe test**: sync the same touchpoint again. Expected: the title scan finds the key and the skill writes nothing, saying so.
4. **Run the import + circularity test** (Tier R and W): create one hand-written note on that contact, ask the skill to import CRM notes for them. Expected: the hand-written note imports under `touch:<crm>-note:<id>`; the skill-written sync note is refused (its title carries `[touch:` — the circularity guard).
5. **Record it, sanitized**: add a dated table to `docs/testing.md` (no real names — "a contact in a test workspace"), using the row format of the sibling packet's testing matrix — this repo's testing.md gains its first CRM rows through exactly this protocol.
6. **PR the registry entry**: a section in this file titled `## <CRM> (Tier R|W — tested)` with the slot table, quirks (title-less notes? no delete tool? markdown support?), and a pointer to your testing.md rows. CI validates links; a maintainer sanity-checks claims against rule 2 of `CONTRIBUTING.md` (every claim maps to a demonstrated behavior).

A failed step is still a contribution: file an issue with the step, the tool called, and what came back.
