# This packet is a deliberate product fork of the fulcra-dealflow-memory engine

This packet serves founders selling their own product. Its siblings — [fulcra-dealflow-memory](https://github.com/keng009/fulcra-dealflow-memory) (investors managing deal flow; the engine's origin) and [fulcra-raise-memory](https://github.com/keng009/fulcra-raise-memory) (founders raising) — serve different ICPs with different vocabularies (leads and accounts here, instead of founders-and-startups or investors-and-funds) and, over time, different features. This repo was derived 2026-08-31 from the engine at contract v3.1 plus review rounds 5–8 (snapshot-first flow, per-source dedupe keys, the veto invariant, review queue, sweep watermarks, commit ledger, CRM and messaging adapters) with a fresh public history (ADR-0001) and a disjoint Fulcra namespace (`/sales/`, `Sales Touchpoint` — the siblings use `/dealflow/` + `Dealflow Touchpoint` and `/raise/` + `Raise Touchpoint`) so all three products can run on one account.

**Decision**: the sibling repos diverge intentionally. There is no shared module, no cross-repo byte-alignment rule, and no obligation to keep contracts identical. Engine-level fixes (dedupe, veto, healing) get cherry-picked between siblings by judgment when they apply; product-level features do not.

**Status**: accepted (2026-08-31, Nick).

**Consequences**: byte-alignment (CONTRIBUTING rule 1) remains strictly INTRA-repo; when adopting an engine fix from a sibling, adapt it to this contract rather than copying bytes; this repo's claims stand on its own [testing.md](../testing.md) — the siblings' live tests are evidence for the shared engine's design, never for this packet's behavior.
