# Setting up personas and node types

Before the Knowledge Base can hold knowledge, it needs to know who the knowledge is
for and what kinds of thing it is made of: the **personas** and the **node types**.
[Concepts](concepts.md) explains what they are. This page sets them up.

You can set them up in the settings screen, or over MCP with the `kb_admin` tool.
Both write the same configuration.

## In the settings screen

Go to *Settings → Knowledge Base → Layer 2*. On the left is a tree in three groups:

- **Personas**, each with the number of types written for it. Expand one to see its
  folder and topic types.
- **Shared by everyone**: types with no persona.
- **Work breakdown**: the work types, nested under their parent type.

Select a persona or a type to see it on the right, and click **Edit** to change it.

**Add a persona** with **+ Add persona**: give it a **Name** and a **Description**
of what it needs. Click **Save**; that saves straight away.

**Add a node type** with **+ Add node type**, or with the **+** next to *Shared by
everyone* or *Work breakdown*:

| Field | What it is |
|---|---|
| Name | The type's name, such as *Database*. |
| Is a | *Topic*, *Folder* or *Unit of work*. |
| Written for | The persona, or *Everyone* for a shared type. For a unit of work this is **Under**: *Top level*, or the work type it sits under. |
| Form | *Structured* (fields) or *Unstructured* (free text). |
| Description | What belongs in one. The AI writes to it, so be specific. |
| Fields | Structured types: **Add field**, then a Label, a Key, a Type, Required and a Hint for each. Fill in the Key yourself. |
| Template | Unstructured types: the text a new topic starts from. |
| Lifecycle | Units of work: **Add state** for each state, in order. Without any, it goes through *Planned*, *In progress*, *Done*. |

Click **Save** to save the type. Personas and types can't be deleted or reordered
here; see [below](#deleting-and-reordering-personas).

## Over MCP

Connect an AI client first (see [Connect an AI client](../get-started/pro.md#connect-an-ai-client)).
The examples below are the tool calls; you can also just ask the AI client to make
them.

### 1. Add the personas

One persona per role or specialism you write knowledge for:

```python
kb_admin(action="save_persona", name="Architects",
         description="Structure and decisions: the services, their data and how they talk.")
kb_admin(action="save_persona", name="Testers",
         description="What to test and how: suites and scenarios.")
```

Each persona's `id` comes from its name (`architects`, `testers`); types and topics
refer to it by that id. A new persona goes at the end, and the order is the order of
the lanes in the explorer. `kb_admin(action="list_personas")` shows them.

Saving again with the same name changes that persona, and replaces its description.

### 2. Add topic types

A topic type is one kind of page: a database, a web service, a test scenario. Give
it the persona it is written for, and a description of what belongs in one: the
type is the specification the AI writes to.

A **structured** type has fields every topic of that type fills in:

```python
kb_admin(action="save_type", name="Database", form="structured", persona="architects",
         description="One database the applications use, and what is in it.",
         fields_json='{"engine": {"label": "Engine", "type": "text", "required": true},'
                     ' "diagram": {"label": "Database diagram", "type": "mermaid"}}')
```

Field types: `text`, `long-text`, `number`, `boolean`, `url`, `date`, `plantuml`,
`mermaid`, `wireframe`. A field can also have a `hint` for whoever fills it in.

An **unstructured** type is free text, with an optional template a new topic starts
from:

```python
kb_admin(action="save_type", name="Test scenario", form="unstructured", persona="testers",
         description="One scenario: the situation, the action, the expected result.",
         template="## Given\n\n## When\n\n## Then\n")
```

Leave out `persona` for a type everyone shares: its topics appear in every lane.

### 3. Add folder types

A folder type turns a persona's topics into a tree you can browse, such as *Service
area* above the services that belong to it:

```python
kb_admin(action="save_type", name="Service area", category="folder", form="unstructured",
         persona="architects", description="A group of related services, and what they are for.")
```

### 4. Add the work breakdown

Work types describe the units of work, such as a project and the features in it.
They never have a persona. `parent_type` puts a type under another one, and
`states_json` gives it a lifecycle:

```python
kb_admin(action="save_type", name="Project", category="work", form="unstructured")
kb_admin(action="save_type", name="Feature", category="work", form="unstructured",
         parent_type="project",
         states_json='[{"id": "analysis", "label": "Analysis"},'
                     ' {"id": "development", "label": "Development"},'
                     ' {"id": "done", "label": "Done"}]')
```

A work type without `states_json` goes through `planned`, `in-progress` and `done`.

Roadmap doesn't check these rules when you save a type, so keep to them yourself:
only work types have a `parent_type` and states, and a work type has no `persona`.

### 5. Check the result

- `kb_admin(action="list_types")` lists every type, its form and how many nodes use
  it; `kb_admin(action="get_type", id="database")` shows one in full.
- The **Knowledge Base** explorer shows the work breakdown first, then a lane per
  persona, in order (see [The explorer](explorer.md)).
- Everything is stored in the `knowledgebase` section of `config.yaml` (see
  [Configuration](../reference/configuration.md#personas)).

## Changing and deleting types

Saving a type again with the same name replaces what you give and keeps the rest.
Changing a type doesn't rewrite the topics already written: a removed field stays in
their content until they are written again.

```python
kb_admin(action="delete_type", id="test-scenario")
```

Deleting a type is refused while topics use it, unless you add `force=True`; those
topics keep their content but no longer match a type.

## Deleting and reordering personas

There is no action for this, over MCP or in the settings screen. Stop Roadmap, edit
the `personas` list under `knowledgebase` in `config.yaml`, and start it again:

```yaml
knowledgebase:
  personas:                # the order here is the order of the lanes
    - id: testers
      name: Testers
      description: What to test and how.
    - id: architects
      name: Architects
      description: Structure and decisions.
```

Types written for a persona you delete become shared: their topics appear in every
lane.
