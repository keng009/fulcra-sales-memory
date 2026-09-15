# Cross-harness install matrix

The skills are written to run on any agent harness that can load skill instructions and reach the Fulcra MCP server ([AGENTS.md](../AGENTS.md) exists for exactly this). Claims stay per-harness and evidence-backed: **reading the memory** from another assistant is proven at the platform level (any MCP client reads the same files/records); **running this flavor's skills** on a harness needs a green row here. This flavor's behaviors are live-tested from the development harness (Claude's desktop app, Code tab — see [testing.md](testing.md)); the zip-upload install into Claude's chat app is the one row still waiting on a human run. The engine's cross-harness evidence lives in the sibling's docs/harness-matrix.md (in [the sibling repo](https://github.com/keng009/fulcra-dealflow-memory)) and is design evidence only (ADR-0007).

| Harness | Install path | Last attempt | Result | Blockers / notes |
|---|---|---|---|---|
| Claude (claude.ai, custom skill upload) | zip → Customize → Skills → + Create skill → Upload a skill | — | ⚪ Untried | The release gate ([#1](https://github.com/keng009/fulcra-sales-memory/issues/1)) |
| Claude Code / Claude Cowork (desktop app, Code tab) | skill folder in-session | 2026-09-15 | 🟢 Verified — snapshot → commit → veto, Attio sync/import guards, unattended auto-log, and a scheduled sweep (task built by the skill's rule 7, run succeeded) all executed from the repo folder | Development harness; scheduled tasks run while the app is open |
| Hermes / OpenClaw / Codex / Grok | agent reads repo (AGENTS.md) | — | ⚪ Untried | Sibling's Hermes attempt (2026-08-28) hit a harness crash + suspected MCP-server bug — retest here after that verdict |
| Gemini | — | — | ⚪ Not targeted | No Fulcra MCP path established |

Legend: 🟢 verified · 🟡 partially verified · 🔴 attempted, inconclusive/failed · ⚪ untried.
