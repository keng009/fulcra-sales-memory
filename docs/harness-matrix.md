# Cross-harness install matrix

The skills are written to run on any agent harness that can load skill instructions and reach the Fulcra MCP server ([AGENTS.md](../AGENTS.md) exists for exactly this). Claims stay per-harness and evidence-backed: **reading the memory** from another assistant is proven at the platform level (any MCP client reads the same files/records); **running this flavor's skills** on a harness needs a green row here. This packet is flavor-untested everywhere — the engine's evidence lives in the sibling's docs/harness-matrix.md (in [the sibling repo](https://github.com/keng009/fulcra-dealflow-memory) — on the v0.3.0 branch until its PR #28 merges) and is design evidence only (ADR-0007).

| Harness | Install path | Last attempt | Result | Blockers / notes |
|---|---|---|---|---|
| Claude (claude.ai, custom skill upload) | zip → Customize → Skills → + Create skill → Upload a skill | — | ⚪ Untried | The release gate ([#1](https://github.com/keng009/fulcra-sales-memory/issues/1)) |
| Claude Code / Claude Cowork | skill folder in-session | — | ⚪ Untried for this flavor | Development harness |
| Hermes / OpenClaw / Codex / Grok | agent reads repo (AGENTS.md) | — | ⚪ Untried | Sibling's Hermes attempt (2026-08-28) hit a harness crash + suspected MCP-server bug — retest here after that verdict |
| Gemini | — | — | ⚪ Not targeted | No Fulcra MCP path established |

Legend: 🟢 verified · 🟡 partially verified · 🔴 attempted, inconclusive/failed · ⚪ untried.
