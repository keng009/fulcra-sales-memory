# Fresh public history; internal documents stay external

This repo was created 2026-08-31 as a product fork of the [fulcra-dealflow-memory](https://github.com/keng009/fulcra-dealflow-memory) engine (see ADR-0007). Like its siblings — and for the same reasons — it publishes from a fresh `git init`: the engine's development history references private business context and named counterparties, and spec/plan documents for both packets live in a private internal repo, never here. History therefore starts at the finished fork rather than showing derivation commits.

**Status**: accepted (2026-08-27, Nick).

**Consequences**: development context a contributor might want lives in the tracker (issues record findings, dispositions, and decisions — see CONTRIBUTING rule 4), not in archaeology of this repo's history; nothing in any commit may reference the internal documents; once outside contributors hold commits here, re-publishing with rewritten history is off the table — all releases are additive.
