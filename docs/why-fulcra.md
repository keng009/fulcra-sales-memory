# Why Fulcra?

These skills could not be honest instruction files without a place to put memory. This page says exactly what they use Fulcra for, what that buys a founder selling their own product, and where Fulcra is genuinely necessary versus merely convenient — because buyers and teammates will check claims, and so should you.

## What the skills actually use

| Fulcra capability | Used for |
|---|---|
| Data catalog (`get_data_catalog`) | The bootstrap and the demo's opening move: inspect what the account already holds before assuming or writing anything |
| Versioned file store (`list_files`, `read_file`, `write_file`) | The narrative memory: one relationship file per lead or customer, plus `INDEX.md`, `handoff.md` (incl. the vetoed-keys list), and the review queue. Every write creates a new version; deletes are soft |
| Custom data types (`create_data_type`, `record_data`, `get_records`) | The structured memory: one `Sales Touchpoint` record per conversation, queryable by time window |
| Calendar (`get_calendar_events`), where the account has it | One of the snapshot's sources (calendar can also come from a Claude-side connector — dual-surface, see the skills) |
| Timezone (`get_user_info`) | Every timestamp written |

## What that buys you while you sell

- **Memory that outlives the chat — and the quarter.** A pipeline runs for months across dozens of customer threads. Conversations end; the account doesn't. Everything logged is there next session, next month, and at renewal or re-engagement, when "why Brightpath went quiet last time, and what they said" is worth real money.
- **Cross-assistant memory — the part nothing else provides.** The same files and records are readable by any assistant connected to your account over MCP. Log a call in Claude, ask ChatGPT "what does Jordan care about?" — same memory, no re-teaching. Chat products' built-in memories are silos by design; this one is yours.
- **Momentum you can query.** "What moved this week" and the leads-going-cold list are one windowed query over typed records — not a re-read of every file (ADR-0006). In founder-led sales, cold threads are revenue quietly dying; a queryable memory surfaces them on time.
- **Reversibility you can bank on.** The one-yes batch commit (ADR-0005) is only honest because files are versioned and soft-deletable — the consent model leans directly on the storage's properties.
- **Neutral ground.** The memory sits in your account — not in your tracker vendor or any one AI company. Switch CRMs, switch assistants, bring on a fractional seller with different tools: the memory stays. The files are plain markdown you can read in the Fulcra portal without any AI at all.

## Where Fulcra is necessary — and where it isn't

Honestly drawn line:

| Job | Without Fulcra? |
|---|---|
| Capture one customer call in a single Claude session | Possible — chat memory or a pasted note would hold it for a while |
| Memory that persists reliably across a months-long pipeline with an exact, auditable format | Weak without it — chat memory is lossy, unstructured, and unqueryable |
| The same memory readable from Claude *and* ChatGPT *and* anything MCP-connected | **Not possible without an account-level store like Fulcra** — this is the load-bearing use |
| Time-windowed momentum reports and going-cold alerts across dozens of funds | Needs typed, queryable records — not a folder of notes, not chat memory |
| Versioned writes and soft deletes underpinning batch consent | A storage property; the skills inherit it rather than promising it themselves |
| Reading calendar/transcripts/CRM sources | Fulcra not required — those come from their own connectors (Fulcra-held calendar data works too) |
| Copying notes into your CRM or tracker | Fulcra not required for the write — but the dedupe keys and provenance that make the sync idempotent live in the Fulcra-side records |

So: a founder logging the occasional call in one assistant could limp along without Fulcra. The moment sales memory has to be durable, structured, queryable, reversible, and readable by more than one assistant — which is the premise — the account-level store stops being optional. Where the platform has limits, the skills say so rather than papering over them: typed records have no per-record delete (vetoes are tombstoned and excluded from reads), and reads can briefly lag writes (the skills carry a lag guard).
