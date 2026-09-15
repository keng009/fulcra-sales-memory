# Connecting a CRM — per-CRM paths and what the skill will and won't do

The CRM counterpart of the Fulcra connection guide. Same honesty: no skill can create a CRM account or authorize its connector — those are the user's clicks — but once the connector is present, the skill can inspect the workspace, build lean structure, and verify it.

## What `crm-setup` does and does not do, in any CRM

| Will | Won't |
|---|---|
| Detect the connector and verify the workspace | Create the account or authorize the connector |
| Read objects, lists, attributes, existing stages | Assume the workspace is empty |
| Recommend a stage set mirroring `sales-memory`'s vocabulary | Rename, reorder, or delete existing structure |
| Create an optional list (Attio) on a ledger + one yes | Create or edit contacts, companies, deals, notes |
| Read every write back and record the mapping in Fulcra | Write anything the ledger didn't list |

## Attio (designed against the live connector; structure build untested until a testing.md row)

**Account and connector**
1. Create a workspace at [attio.com](https://attio.com) (free tier is enough to start).
2. In Claude: **Customize → Connectors → add Attio**, sign in, pick the workspace.
3. Verify: ask *"who am I in Attio?"* — the connector's `whoami` returns the workspace name and your access level.

**What the connector can do for structure**: list objects, lists, and attribute definitions; **create lists** (`create-list`: name, snake_case slug, parent object, access); add records to lists; create records, notes, and tasks. **What it cannot do**: create or edit attributes or status stages — those are set in the Attio UI (Settings → Objects → *object* → attribute). The skill therefore recommends stage names, waits, and reads them back.

**Recommended lean structure**: keep the standard People / Companies / Deals objects; use Deals' required *Deal stage* as the pipeline with stages **Lead, Qualified, Demo, Proposal, Negotiation, Closed won, Closed lost**; no custom attributes needed. Notes attach to People (what `sales-memory` syncs) and can also be associated to a Deal (slot 6 in `crm-sync.md`, designed-for).

**Quirks that matter**: notes have no custom fields (the dedupe key lives in the note title); the connector has no note delete (a vetoed sync note is removed by hand in the UI); the same person can exist twice with different emails — `sales-memory` matches by email first for exactly that reason.

## HubSpot (designed-for, untested)

**Account and connector**: a HubSpot account (free CRM tier works); in Claude, **Customize → Connectors → add HubSpot** (the official connector — write-capable as of 2026-09; verify its current tool list on first use). Verify with the connector's organization or owner lookup.

**Structure**: HubSpot's Deals object and its default *Sales Pipeline* carry deal stages; the connector can create deals and notes (engagements) but pipeline stages are edited in HubSpot settings (Objects → Deals → Pipelines). Same pattern: recommend the seven stages, the user sets them, the skill reads the pipeline back and maps. Notes may lack a title field — `sales-memory` puts the key on the first body line for that case (see `crm-sync.md`).

## Notion (designed-for, untested)

**Account and connector**: a Notion workspace; in Claude, **Customize → Connectors → add Notion** and grant the pages/databases the skill may see. Verify with a search.

**Structure**: a database of contacts (one page per person) with a *Status* property using the seven stages. The connector reads and appends to pages but does not create databases or properties — the user creates the database from the recommended property list; the skill reads it back. `sales-memory`'s only Notion write is a block appended to the matched contact's page.

## When there is no CRM at all

That is a complete setup. `sales-memory` with Fulcra alone is the system of record for conversations; going-cold, prep briefs, and momentum reports need no CRM. Set one up when a second person needs the pipeline or when the user wants a board view — and come back to this skill then.
