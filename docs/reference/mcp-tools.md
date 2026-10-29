# MCP tools

Roadmap's MCP server exposes the Knowledge Base as **seven tools, one per role**,
each taking an `action`. A client sees seven tools instead of dozens, and an agent
that only looks things up never has to read the vocabulary for changing them.

Every tool takes an optional `workspace` argument. `kb_read`, `kb_write` and
`kb_flow` also take an optional `branch`: the call reads and writes in that
[branch](../knowledge-base/branches.md).

Errors come back as readable text that names what's missing or wrong
(`Not written: Node type 'web-service' requires base_url.`), not as exceptions, so
an agent can correct itself from the message.


## Two endpoints

| Endpoint | Transport | Tools |
|---|---|---|
| `/mcp/sse` | SSE: a long-lived session | one MCP tool per tool on this page |
| `/mcp` | stateless HTTP: one POST per call | one tool, `roadmap`, that runs the others by name |

Both serve the same tools. On `/mcp`, call `roadmap` with a `command` and its
`args`:

```json
{"command": "kb_read", "args": {"action": "status"}}
```

`{"command": "help"}` lists every command with its arguments, and
`{"command": "help", "args": {"command": "kb_admin"}}` shows one in full. A command
the endpoint doesn't know is answered with the closest names.

The workspace works the same on both: `/mcp/<workspace>/sse` or `/mcp/<workspace>`
picks it for the connection, and every command also takes `workspace`, which wins
over the connection's. Connecting a client: see
[Connect an AI client](../get-started/pro.md#connect-an-ai-client).

## `kb_read`

Navigate and query.

| Action | Does | Needs |
|---|---|---|
| `status` | counts, the sources you may write to, the node types that carry content | — |
| `find` | search nodes by `query` (id, title, uri, reference fields), narrowed by `kind` / `source` | — |
| `get` | one node in full, with its content | `source`, `kind`, `id` |
| `neighbours` | what a node links to and how, optionally filtered by `relation` | `source`, `kind`, `id` |
| `legacy` | content nodes not yet in the [document format](../knowledge-base/documents.md) | — |

## `kb_write`

Change the graph.

| Action | Does | Needs |
|---|---|---|
| `add_node` | add or replace a node; re-using an id updates it | `kind`, `id` |
| `update` | change part of a content node: `fields` are merged (`null` removes one), `body` or `document` replaced | `kind`, `id`, `content_json` |
| `set_state` | move a unit of work one state along its lifecycle | `kind`, `id`, `state` |
| `delete_node` | remove a node and every link that touches it | `source`, `kind`, `id` |
| `link` | relate two nodes; `relation` says how (`contains` places a node in a tree). Both nodes must exist: write them first with `add_node`, or the link is refused (`Not written: no node ...`) | `kind`, `id`, `dst_kind`, `dst_id` |
| `unlink` | remove that relation; the nodes stay | same as `link` |

`run` on `add_node`, `update`, `link` and `set_state` stamps the write with a
[flow run](../knowledge-base/flows.md#runs-and-provenance).

```python
# a pointer into Jira: identity only
kb_write(action="add_node", kind="ticket", id="PAY-3187", source="atlassian")

# something the Knowledge Base owns
kb_write(action="add_node", kind="web-service", id="payments-api",
         title="Payments API",
         content_json='{"fields": {"base_url": "https://pay.internal/api"}}')

kb_write(action="link", kind="web-service", id="payments-api",
         dst_kind="ticket", dst_id="PAY-3187", dst_source="atlassian",
         relation="changed-by")
```

## `kb_admin`

Administer node types, personas, packs and sources.

| Action | Does | Needs |
|---|---|---|
| `list_types` | every node type, its form, and how many nodes use it | — |
| `get_type` | one type in full, including its template and states | `id` |
| `save_type` | create or replace a type: `description`, `category` (`topic` / `folder` / `work`), `persona`, `parent_type`, `states_json`, `fields_json` or `template`, and `icon` (a Bootstrap icon name such as `bi-database`, shown on the explorer's chips); what isn't given is kept from the existing type | `name` (or `id`), `form` |
| `delete_type` | remove a type; refused while nodes use it unless `force` | `id` |
| `list_personas` | the personas, in order | — |
| `save_persona` | add a persona at the end, or change one. The `id` comes from the name, so saving with an existing persona's name changes that persona and replaces its description | `name` (or `id`), `description` |
| `list_packs` | the [packs](../knowledge-base/packs.md) in the pack registry | — |
| `install_pack` | install a pack, or part of it with `items_json` = `{"personas": [...], "node_types": [...], "replace": [...]}`; what a type needs comes along, and what you have is kept unless named in `replace` | `id` |
| `clear_source` | drop everything ingested from one source | `source` |

```python
kb_admin(action="save_type", name="Web service", form="structured",
         fields_json='{"base_url": {"label": "Base URL", "type": "url", "required": true},'
                     ' "protocol": "Protocol"}')
```

Personas can't be deleted or reordered with `kb_admin`. Remove one in the settings
screen; to reorder them, edit the `knowledgebase.personas` list in `config.yaml` (see
[Setting up personas and node types](../knowledge-base/setup.md#deleting-and-reordering-personas)).

A work type saved without `states_json` goes through the default lifecycle:
`planned`, `in-progress`, `done`. Those are the ids `set_state` takes.
`get_type` still shows `"states": []` for such a type.

## `kb_flow`

Take a [flow](../knowledge-base/flows.md) on.

| Action | Does | Needs |
|---|---|---|
| `list` | the flows, and whether each is ready | — |
| `describe` | what one reads, who it writes for, which types it may use, what stops it | `id` (flow) |
| `brief` | **take it on**: opens a run and returns the whole prompt plus the run id | `id` (flow), optional `work_item` |
| `progress` | report the stage the run has started, with a one-line `message` | `id` (run), `stage` |
| `get` | what a run has produced so far, and the stages it reported | `id` (run) |
| `finish` | close a run with its `outcome`: `ok`, or `failed` with a `message`; `commit` records the git commit it made | `id` (run) |

## `kb_orchestrate`

Run the [state machines](../knowledge-base/orchestration.md).

| Action | Does | Needs |
|---|---|---|
| `list` | the orchestrations, with how many instances are where | — |
| `describe` | states, transitions, a Mermaid diagram, problems | `id` |
| `save` | create or replace one from `orchestration_json` | `orchestration_json` |
| `delete` | remove one and its instances | `id` |
| `start` | instances in the initial state; `branch` picks the branch (`main` forces main) | `id`, optional `subject` |
| `next` | **take the next piece of work**: the oldest ready instance's flow brief | optional `id` |
| `status` | the instances: state, status, run, branch | optional `id` |
| `signal` | send `signal` (and `message`) to a waiting instance | `id` (instance), `signal` |
| `retry` / `cancel` | put a stuck instance back in its state / stop one | `id` (instance) |
| `clear` | forget every instance of one orchestration | `id` |

## `kb_chat`

Host a [chat](../knowledge-base/chat.md) about the knowledge base.

| Action | Does | Needs |
|---|---|---|
| `join` | the chat's subject, the conversation so far, and how to host it | `id` (the chat code) |
| `wait` | the person's new messages; returns empty after `timeout` seconds (at most 50), call again | `id` |
| `say` | reply: short markdown `text`, and `show_json`, up to four pieces to show: `{"node": "source/kind/id", "section": "<title or number>"}` | `id` |
| `history` | the whole conversation | `id` |
| `branch` | put the chat on branch `text` (empty: main) | `id` |

## `kb_branch`

[Branches](../knowledge-base/branches.md) of the knowledge base.

| Action | Does | Needs |
|---|---|---|
| `list` | the branches, open ones first, with their work item's state and how much they change | — |
| `get` | one branch: its work item and state, git branch, commits, change counts | `id` |
| `diff` | what it changes, node by node | `id` |
| `compare` | one topic on main beside the same topic in the branch, as a diff | `id`, `node` |
| `context` | the branch as **as is → to be**, grouped by persona; `personas` (comma-separated) narrows it | `id` |
| `open` | open a branch for a unit of work; `chat` puts that chat on it; `git_branch` names the code's branch | `name`, optionally `work_item`, `description`, `chat`, `git_branch`, `id` |
| `update` | change `name` / `description` / `work_item` / `git_branch` | `id` |
| `commit` | record a commit made on the git branch | `id`, `commit`, `message` |
| `merge` | apply it to main and close it; `force` merges despite conflicts | `id` |
| `abandon` / `reopen` | close without merging; open again | `id` |

## Git Repos and workspaces

| Tool | Does | Needs |
|---|---|---|
| `list_git_repos` | the configured [Git Repos](../get-started/git.md): name and path | — |
| `get_git_repo_info` | one repo's checkout path, its branches and its recent commits | `repo_name` |
| `list_workspaces` | the workspaces on this server | — |
