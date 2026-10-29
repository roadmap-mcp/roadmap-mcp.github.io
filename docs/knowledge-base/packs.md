# Packs

A **pack** is a ready-made set of personas, the node types written for them, and the
work types that go with them. Install one instead of
[setting everything up yourself](setup.md), then adapt what it gives you.

Packs are published in a **pack registry**, the way plugins are in the plugin
registry. Installing one only adds personas and types to the workspace's
configuration; it doesn't install any code. Installing a pack, in the settings
screen or over MCP, needs the [admin role](../get-started/users.md#roles). Packs are installed per workspace: a new
workspace starts empty.

## The packs

| Pack | Persona | What it holds |
|---|---|---|
| `functional-analysis` *Functional analysis* | Analysis | functional areas, use cases, screen descriptions, use case diagrams, screen flows |
| `legacy-migration` *Legacy migration (Delphi to Java)* | Technical design | reverse-engineering topics (screens, units, database) and migration designs, plus a work breakdown of migration project, phase and step, with a seven-state lifecycle for a step |
| `development` *Development* | Development | development areas, implementation records, guidelines, sequence diagrams |
| `e2e-testing` *End-to-end testing (Playwright)* | Testing | test suites and test scenarios |
| `primeng-specialist` *PrimeNG specialist* | PrimeNG specialist | sections, components, guides |

## Install a pack in the settings screen

1. Go to *Settings → Knowledge Base → Layer 2* and click **Packs**. The registry's
   packs are listed, each with its version, description and how many personas and
   types it holds.
2. Open a pack. Every persona and type in it has a checkbox, and a state:
    - **new**: you don't have it yet;
    - **installed**: you have it, the same as in the pack;
    - **differs from yours**: you have one with the same id, but it isn't the same.
3. Tick what you want. The whole pack is ticked to start with; **All** and **None**
   tick or untick everything. Ticking a type also ticks what it needs: its persona
   and the work types above it. Unticking a persona or a work type unticks what
   depends on it.
4. Click **Install N item(s)**. The result says what happened, such as *2 installed*
   or *1 kept as you had them*, and the ticks are cleared.

A persona or type you already have is kept as you have it, unless you tick
**replace mine** on it. A replaced persona keeps its place in the order of the lanes.

**Install from file** installs a pack file from your disk the same way: it shows the
same list and states. A file that isn't a valid pack is refused with the reason, for
example a type written for a persona the pack doesn't carry.

## Install a pack over MCP

```python
kb_admin(action="list_packs")
```

lists the registry's packs: id, name, version, description and counts.

```python
kb_admin(action="install_pack", id="e2e-testing")
```

installs the whole pack and reports, for each persona and type, whether it was
installed, unchanged, kept or replaced, starting with the pack and its version:
``Pack `e2e-testing` (v1.0.0): installed: ...``. To install part of a pack, or to replace
items you already have, add `items_json`:

```python
kb_admin(action="install_pack", id="functional-analysis",
         items_json='{"node_types": ["use-case"], "replace": ["use-case"]}')
```

What a type needs comes along, as in the settings screen; here that is the
*Analysis* persona. An unknown pack id is refused:
`Not done: No pack 'x' in the registry. Packs: ...`.

## Export your own pack

**Export**, in the Layer 2 header, writes personas and types you have as a pack:

1. Fill in a **Name**, an **Id** (filled in from the name), a **Version** and a
   **Description**.
2. Tick the personas and types to include. What a ticked type needs, its persona
   and the work types above it, goes in too.
3. Click **Download pack**. You get `<id>.json`.

Install it in another workspace with **Install from file**, or share it.

## Another pack registry

The app reads the registry at
`https://raw.githubusercontent.com/roadmap-mcp/roadmap-dist/main/kb-packs/index.json`.
To use another one, set `ROADMAP_KB_PACK_REGISTRY_URL` before you start Roadmap (see
[Environment variables](../get-started/pro.md#environment-variables)).
