# Configuration

The Knowledge Base is configured per workspace, in the `knowledgebase` section of
the workspace's `config.yaml`. The settings page edits it under *Settings →
Knowledge Base*, and `kb_admin` / `kb_orchestrate save` write parts of it over MCP.

| Tab | Section | Holds |
|---|---|---|
| Layer 1 | `sources` | the information sources that feed the ground layer |
| Layer 2 | `personas`, `node_types` | who knowledge is written for, and the types written for them |
| Flows | `flows` | configured paths into the knowledge base |
| Orchestration | `orchestrations` | state machines that run the flows |

## Sources

```yaml
knowledgebase:
  sources:
    - id: atlassian         # nodes from this source are keyed by it
      type: atlassian       # the information source it was added from
      label: Atlassian
      enabled: true
```

The source picker offers the settings sections filed under *Information Sources*,
so a newly installed source plugin shows up with no change to the Knowledge Base.
Credentials stay on the source's own settings tab.

## Personas

```yaml
knowledgebase:
  personas:                # the order here is the order of the lanes
    - id: architects
      name: Architects
      description: structure and decisions
```

## Node types

```yaml
knowledgebase:
  node_types:
    - id: database
      name: Database
      category: topic          # topic | folder | work
      form: structured         # structured | unstructured
      persona: architects      # "" means everyone
      fields:
        - { name: engine, label: Engine, type: text, required: true }
        - { name: diagram, label: Database diagram, type: mermaid }
    - id: test-scenario
      name: Test scenario
      category: topic
      form: unstructured
      persona: testers
      template: "## Given\n\n## When\n\n## Then\n"
    - id: feature
      name: Feature
      category: work
      parent_type: project
      states:
        - { id: analysis, label: Analysis }
        - { id: design, label: Design }
        - { id: development, label: Development }
        - { id: done, label: Done }
```

Field types: `text`, `long-text`, `number`, `boolean`, `url`, `date`, `plantuml`,
`mermaid`, `wireframe`.

## Flows

```yaml
knowledgebase:
  flows:
    - id: delphi-tip-analysis
      name: Delphi tip analysis
      kind: analyse               # analyse | ingest | act
      enabled: true
      source: {kind: repo, ref: legacy-delphi, scope: {paths: ["src/**"]}}
                                  # kind: repo | kb_source | context | web
      targets:
        personas: [architects, programmers]
        node_types: []            # empty = every type of those personas, plus shared
        allow_new_folders: true
      work_item:                  # what the output is about
        required: false
        kind: project
        id: delphi-migration      # a default; empty = each run supplies one
        on_start: ""              # states the item moves to, when not orchestrated
        on_finish: ""
        on_fail: ""
      prompt:
        instructions: "Read the unit sources and record, for each one, ..."
        stages:
          - {id: read, label: Read the sources, description: "..."}
      triggers: [manual, mcp]     # manual | mcp | source
      limits: {max_nodes: 200, max_input_chars: 200000}
```

`version` (on the flow) and `prompt.version` are bumped by Roadmap when the
targets or the prompt change, and are stamped on everything a run writes.

## Orchestrations

```yaml
knowledgebase:
  orchestrations:
    - id: feature-delivery
      mode: each                  # single | each
      subject_type: feature       # each: the work type that gets instances
      initial: design             # empty = the first state
      max_visits: 5
      states:
        - {id: design, flow: technical-design}
        - {id: review}                                # no flow: waits for a signal
        - {id: done, final: true, merge_branch: true}
      transitions:
        - {state: design, on: ok, to: review}
        - {state: review, on: approved, to: done}
        - {state: "*", on: cancelled, to: done}
```

See [Orchestration](../knowledge-base/orchestration.md) for what each piece means.
