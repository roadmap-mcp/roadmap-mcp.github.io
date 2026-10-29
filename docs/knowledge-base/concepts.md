# Concepts

## Personas

A **persona** is who knowledge is written *for*: a role (*Architects*, *Testers*,
*Business analysts*) or a specialism (*PrimeNG specialist*). Each persona gets a
**lane** in the explorer: a tree of the topics written for it.

A persona has a name and a description of who it is and what its topics cover. The
description is optional, but write one: it tells people and AI agents what belongs
in that persona's lane.
Its `id` is derived from the name the first time it's saved, and topics refer to
it by that id, so renaming a persona later keeps its tree. The order personas are
configured in is the order of the lanes.

## Node types

Every node type has one of **three categories**, and the category decides what
else it carries:

| Category | Lives in | Also carries | What it is |
| --- | --- | --- | --- |
| `work` | the work breakdown | `parent_type`, `states` | a project, a phase, a feature, a step |
| `folder` | a persona's lane | `persona` | how that persona's topics are browsed |
| `topic` | a persona's lane | `persona` | the page somebody wanted to read |

- A **topic** is the leaf: a use case, a database, a REST endpoint, a screen, a
  test scenario. Use one type per kind of information. *Database* and *Web service*
  are two types, not one "technical asset" type with a discriminator.
- A **folder** turns a persona's graph of topics into a tree a person can browse.
  A folder can carry content of its own, so an *overview of services* can stand
  above the services and say what the set of them is for.
- A **unit of work** never has a persona. The work breakdown belongs to no one
  lane. It cuts across all of them, which is exactly what makes it useful (see
  [The work breakdown](#the-work-breakdown)).

A type with no persona is **shared**: its topics appear in every lane.

### Structured and unstructured

A type has one of two forms:

- **Structured**: a short list of `fields` (`text`, `long-text`, `number`,
  `boolean`, `url`, `date`, and the diagram kinds `plantuml`, `mermaid` and
  `wireframe`), each optionally `required`, with a `hint`. Every instance answers
  the same questions, so a set of them can be compared and queried.
- **Unstructured**: free text, with an optional `template`, so the tenth test
  scenario has the same headings as the first.

!!! tip "The type is the specification"
    When an AI writes a topic, the type's description, fields and template are
    what it's asked to fill in. A vague type description produces vague topics.

## The work breakdown

Work types form a hierarchy through `parent_type`: a project holds features, a
feature holds steps. The breakdown is the first lane in the explorer.

Every work type has a **lifecycle**: an ordered list of `states`. If a type
declares none, it gets *Planned → In progress → Done*. The order is the state
machine. A person moves an item one state at a time, forward or back; the last
state means finished. A flow or an [orchestration](orchestration.md) moves items
as its work goes on.

Progress rolls up. A leaf's progress is how far along its lifecycle it is. An item
with units of work under it averages theirs: it's in the first state while none
has started, and in the last once all are finished. The state is stored with a log
of every move: when, from what, and by which run or person.

A unit of work can also have a **context**: `context` links to the topics someone
doing the item should read first (its use cases, its screens). It's the work
breakdown's version of a Roadmap context: a starting set of pointers, and the
topics' own links lead on from there.

## Relations

Links are typed. Two relation types have a fixed meaning:

| Relation | Means | Cardinality | Read by |
|---|---|---|---|
| `contains` | where a node **sits** | one parent | the explorer's lane trees |
| `about` | which work it **concerns** | many | "what do we know about this item?" |

`contains` alone decides what nests under what. Every other relation
(`documented-in`, `implements`, `uses`, `contradicts`, ...) is knowledge *about*
two things, shown on a node's *Links* tab.

A topic written *about* a unit of work isn't *held by* it. Roadmap writes the
`about` links itself: everything written under a [flow run](flows.md#runs-and-provenance)
is linked to that run's work item, so an AI never has to repeat it per node.

## Sources

Every node belongs to a **source**: one of the configured information sources
(an Atlassian site, for example), or **`manual`**, the source for anything an agent
or a person adds directly. `manual` is always writable. Removing a configured
source removes its nodes and links, so the graph never keeps orphans.

A pointer node's content is **fetched on demand** from its source. The explorer's
*Refresh from source* re-reads it, updates the node's label, and shows the live
content without storing it. Atlassian (Jira issues, Confluence pages) has a fetcher
today.
