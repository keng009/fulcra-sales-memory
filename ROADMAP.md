# Roadmap

Where this packet is headed, by theme. The [issue tracker](https://github.com/keng009/fulcra-sales-memory/issues) is the source of record — this page is the map. Nothing here is a promise with a date; items gated on platform capabilities ship only when those are live (CONTRIBUTING rule 3).

## Now — first live run, then v0.1.0

The flavor's own surfaces are now live-tested from the development harness — snapshot → commit → veto, Attio sync and import guards, unattended auto-log, and a scheduled sweep run ([testing.md](docs/testing.md)). The release gate [#1](https://github.com/keng009/fulcra-sales-memory/issues/1) has one human step left: a full `sales-demo` session through Claude's real zip-upload flow. Plus a fresh-eyes review pass of the founder vocabulary and claims ([#2](https://github.com/keng009/fulcra-sales-memory/issues/2)). (CI is live: `validate` runs on every push and is a required check on main.)

## Adapters under this flavor

The capability-based adapter layers came over from the fork point; none is exercised under `/sales/` yet ([#3](https://github.com/keng009/fulcra-sales-memory/issues/3)). Seller-relevant order: HubSpot and Notion trackers first, then Attio and Affinity; messaging paste tier for the channels leads actually reply on (WhatsApp, LinkedIn, iMessage). The gated contact-creation slot (ADR-0008) has its own live-test gate ([#4](https://github.com/keng009/fulcra-sales-memory/issues/4)).

## Toward automatic — without losing consent

- **Scheduled message-sweep digest** ([#5](https://github.com/keng009/fulcra-sales-memory/issues/5)): the behavior is now specified in the full skill (Tend rule 5) — leads go cold in DMs, not email; what remains is the live scheduled run that promotes it from designed to tested.
- **Notes on the account's deal/opportunity object** ([#6](https://github.com/keng009/fulcra-sales-memory/issues/6)): each tracker's own object model, never touching fields or stages.
- Unattended auto-log: **accepted** as [ADR-0009](docs/adr/0009-unattended-auto-log.md) (2026-09-15) and implemented as Tend rule 6 — a standing, revocable, per-pipeline yes with high-confidence-only eligibility, receipts, and a dead-Fulcra stop. Designed/untested until the four testing.md scenarios run.

## Sales-specific features — earned, not guessed

This fork exists so seller features can diverge from the sibling packets (ADR-0007). The first shipped: **multi-pipeline support** (2026-09-02) — the maintainer sells for two businesses, so `/sales/` separates registered pipelines across capture, reports, and going-cold, invisibly for single-business users. Further candidates come from real sales usage — follow-up nudges from stored commitments, per-account brief drafting, inbound-signal capture patterns, a per-pipeline CRM mapping ([#7](https://github.com/keng009/fulcra-sales-memory/issues/7)) — and get scoped as issues when a real user needs them. Nothing lands as roadmap theater.

## The siblings

[fulcra-dealflow-memory](https://github.com/keng009/fulcra-dealflow-memory) (investors managing deal flow — the origin of this engine, with the live-test evidence) and [fulcra-raise-memory](https://github.com/keng009/fulcra-raise-memory) (founders raising) each have their own ROADMAP.md. Features diverge by ICP; engine fixes cherry-pick (ADR-0007).
