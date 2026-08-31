# Gated CRM contact creation (sales flavor only)

The engine rail the siblings carry is absolute: CRM sync never creates contacts. That fits their ICPs — an investor's or raising founder's CRM is a curated list, and their counterparties are already in it. Founder-led sales is different: a new inbound lead often is not in the CRM yet, and capturing them there is core value, not scope creep. Refusing to ever create a contact would make this flavor's sync useless exactly when a seller needs it most.

**Decision**: this flavor adds capability **slot 8 — create a contact** — behind three gates, all of which must pass:

1. **A recorded preference, off by default.** The first time a live-logged lead has no CRM match, the skill asks once whether to enable adding missing leads, and records the answer in `/sales/handoff.md` under `## Preferences` as `- crm-contact-creation: ask-each-time` or `- crm-contact-creation: never`. No line, `never`, or no answer → the write is skipped and the skill says so (the engine's original behavior).
2. **Per-contact confirmation.** Even at `ask-each-time`, every creation is confirmed for that specific person — name, email, company shown before the yes. Never silent, never bulk.
3. **Live captures only.** Commit, backfill, and import paths never create contacts — only a touchpoint the user is logging live qualifies. Bulk lead-list creation is explicitly out of scope.

A created contact is minimal — name, email, company, nothing else — and the touchpoint note is attached immediately, so no orphan contacts exist. Everything else in ADR-0004 holds unchanged: fields, stages, amounts, owners, and lists are never edited; `stage_noted` stays narrative.

**Status**: accepted (2026-08-31, Nick — "make it optional for the user to choose").

**Consequences**: `crm-sync.md` carries slot 8 and the amended "never create contacts silently" principle; the full skill's Capture step 9 implements the gate; the handoff template gains `## Preferences`; the conventions file documents the preference line. Slot 8 is untested until a testing.md row exists (issue #4), and each CRM's section must say so.
