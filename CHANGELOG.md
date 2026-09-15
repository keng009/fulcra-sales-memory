# Changelog

User-visible changes to the skill packet. Format follows [Keep a Changelog](https://keepachangelog.com/); versions are [release tags](https://github.com/keng009/fulcra-sales-memory/releases) with ready-to-upload zips attached. Live-behavior evidence for every claim: [docs/testing.md](docs/testing.md).

## [Unreleased] — 0.1.0

### Added
- Initial public packet, forked 2026-08-31 from the [fulcra-dealflow-memory](https://github.com/keng009/fulcra-dealflow-memory) engine (contract v3.1 plus review rounds 5–8) and re-flavored for founders selling their own product (ADR-0007): `/sales/` namespace, `Sales Touchpoint` type, lead/customer/account vocabulary, sales-stage `stage_noted` vocabulary (`lead` → `closed-won`/`closed-lost`), "Leads going cold" reporting, "Account history check".
- The two skills: `sales-demo` (guided 10-minute session, snapshot-first) and `sales-memory` (ongoing capture/recall/report with the same snapshot-first Show→Save→Tend flow as the siblings — identical time-to-value by design).
- Engine features carried from the fork point: stable per-source dedupe keys (`touch:cal:<event-id>`, transcript ids, CRM-note ids, thread ids), the veto-set-first invariant and tombstone list, commit ledger before every one-yes, review queue, sweep watermarks with failure-safe advancement, backfill hygiene, dual-surface calendar detection, CRM capability tiers + "Add your CRM" protocol, messaging capture registry (`message` channel; WhatsApp/Telegram/Signal/iMessage/LinkedIn/Slack paste tier), interruption-safe sample cleanup, snapshot-time read disclosure.
- **ADR-0008 — gated CRM contact creation (this flavor only)**: optional capability slot 8 lets the skill add a missing live-logged lead as a minimal CRM contact — off by default, enabled only by a recorded `## Preferences` choice, confirmed per contact, never from imports or backfill. The siblings' "never create contacts" rail otherwise holds.
- CI validation, release packaging, ADRs 0001–0008, five-rule CONTRIBUTING, docs/why-fulcra.md, README chooser, docs/mcp-operations.md, docs/harness-matrix.md (all untried for this flavor), messaging browser-observation tier, scheduled-sweep Tend behavior, CRM slot 6 note placement — designed features carry designed/untested labels until testing.md rows exist.

- **Multi-pipeline support** (first sales-specific feature earned from real usage): sellers with more than one business register pipelines in `handoff.md` (`## Pipelines`); relationships carry a `Pipeline:` line and records an optional `pipeline` field; reports, going-cold, and filtered asks ("what moved in <name>") group per pipeline. Fully progressive — single-business users see none of it.

- **Browser-observation account-risk posture** (engine-level, all siblings): inbox-only, two windows a day, read-only-what-is-new, the user's own browser profile only, stop-on-first-warning with backoff, and an explicit no-decoy-activity rule — restraint, not mimicry, plus the honest ToS disclosure. **LinkedIn lead-sequence discipline** (this flavor): judgment-based ICP screening with a fit score, connect → thank-you (+1 day) → signal-anchored action message (+7 days) tracked as dated follow-ups, 30-day drop candidates in Report, re-screen instead of discard, multi-channel touches — drafts only.

- First live run recorded (testing.md 2026-09-15): snapshot → one-yes commit → veto on a real account with two registered pipelines — folder init, type creation, 8 files + 9 records, backfill hygiene, read-back, tombstone.

- **Connecting Fulcra guide** (`references/connect-fulcra.md`): both setup paths — Claude's app (account → Customize → Connectors → verify) and agent harnesses via Fulcra's official `fulcra-get-started` / `fulcra-connect` skills — with the honest line that no skill can create the account or authorize the connector itself. Both preflights and the README install steps point to it; CI now requires the pointer in every skill.

- **`crm-setup` skill** (third skill in the packet): Connect → Inspect → Build → Hand off. Per-CRM connector paths (Attio; HubSpot and Notion designed-for), workspace inspection before any proposal, a pipeline whose stages mirror `sales-memory`'s vocabulary (the connector builds what it can; attribute/stage edits are the user's two-minute UI step, read back afterward), the stage map recorded under `## Preferences`, then sync handed to `sales-memory`. Structure only — never contacts, deals, or notes; ledger before every write. CI validates its frontmatter and rails; releases ship `crm-setup.zip`.
- **Attio live-tested under this flavor** (testing.md 2026-09-15): email-first contact matching resolving a duplicate-record case, sync note write in the contract format, dedupe re-run skip, and the import path's cross-key and circularity guards against real third-party notes.

### Release gate
- First release requires the live runs listed in [docs/testing.md](docs/testing.md) — this flavor ships engine-proven but flavor-untested until then, and the README says so.
