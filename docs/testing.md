# Live-test matrix

Dated, sanitized record of what has actually been tested against live services. "Tested" claims in the README trace here. No user data appears below.

## Engine provenance (inherited design evidence, NOT this repo's behavior)

This packet was forked 2026-08-31 from [fulcra-dealflow-memory](https://github.com/keng009/fulcra-dealflow-memory) at contract v3.1 plus review rounds 5–8. That repo's [testing matrix](https://github.com/keng009/fulcra-dealflow-memory/blob/main/docs/testing.md) carries dated live evidence for the shared engine design: Fulcra dual-write path, per-destination self-healing dedupe, snapshot→commit→veto end to end, stable calendar-event-id keys (first + second commit, zero duplicates), CRM-note import + circularity guard against Attio, injected partial-failure healing, and a live sweep run (first-run window, park-once, watermark-last ordering).

Per ADR-0007, that evidence supports the **design** of this packet's engine — it is never evidence for this packet's **behavior**. The flavors differ (`/sales/` namespace, `Sales Touchpoint` type, seller-facing vocabulary), and every claim about this packet must trace to a row below.

## This flavor — nothing live-tested yet

| Surface | Status |
|---|---|
| `sales-demo` full session (zip upload → snapshot/capture → save → prep brief) | **Untested** |
| `sales-memory` snapshot → commit → veto on a real account | **Untested** |
| `/sales/` folder init, `Sales Touchpoint` create-if-absent, dual write + read-back | **Untested** |
| CRM adapters (any tier) under this flavor | **Untested** |
| Gated CRM contact creation, slot 8 (ADR-0008 — this flavor's one engine divergence) | **Untested** |
| Messaging capture (paste tier) under this flavor | **Untested** |

First release is gated on at least: one full `sales-demo` session through Claude's actual zip-upload UI, and one `sales-memory` snapshot→commit→veto run on a real account, both recorded here (dated, sanitized).
