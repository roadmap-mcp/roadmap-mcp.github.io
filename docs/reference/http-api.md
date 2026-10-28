# HTTP API

Anything an agent can do over MCP, a person can do with curl. All paths are under
**`/api/knowledgebase`**.

Two headers choose where a request reads and writes:

| Header | Query alternative | Chooses |
|---|---|---|
| `X-Roadmap-Workspace` | `?ws=` | the workspace (default: `default`) |
| `X-Roadmap-KB-Branch` | `?branch=` | the [branch](../knowledge-base/branches.md) (default: main) |

## The graph

| Method | Path | Does |
|---|---|---|
| `GET` | `/available-sources` | installed information sources not yet added |
| `GET` | `/node-types` | the configured node types |
| `GET` | `/status` | node and link totals, per-source counts, recent runs |
| `GET` | `/graph?source=&kind=&query=&limit=` | nodes plus the links among them |
| `GET` | `/nodes?source=&kind=&query=&limit=` | matching nodes |
| `GET` | `/nodes/{source}/{kind}/{id}` | one node with the links that touch it |
| `POST` | `/nodes` | upsert `{nodes: [], links: [], run}`; re-posting an id updates it |
| `POST` | `/links` | upsert `{links: [], run}` |
| `POST` | `/nodes/{source}/{kind}/{id}/refresh` | re-read the node from its source; writes back the label only |
| `DELETE` | `/nodes/{source}/{kind}/{id}` | remove a node and its links |
| `POST` | `/sources/{source}/clear` | drop everything from one source |

Content is checked against its type, and a refusal is a `400` whose message says
what to do instead.

```bash
curl -s -X POST http://127.0.0.1:8090/api/knowledgebase/nodes \
  -H "Content-Type: application/json" -d '{
    "nodes": [
      {"source":"manual","kind":"services","id":"all","title":"All services"},
      {"source":"manual","kind":"web-service","id":"orders","title":"Orders API"}
    ],
    "links": [
      {"src_source":"manual","src_kind":"services","src_id":"all",
       "dst_source":"manual","dst_kind":"web-service","dst_id":"orders",
       "kind":"contains"}
    ]}'
```

## Work

| Method | Path | Does |
|---|---|---|
| `GET` | `/work-status` | every unit of work's state and progress, open runs and their last stage, the last run's outcome |
| `POST` | `/nodes/{source}/{kind}/{id}/state` | move a unit of work one state |

## Flows and runs

| Method | Path | Does |
|---|---|---|
| `GET` | `/flows` | the flows, with their problems |
| `POST` | `/flows/preview` | render the prompt of a flow, saved or not |
| `POST` | `/flows/{id}/brief` | open a run and return the prompt |
| `POST` | `/flows/{id}/run` | run it on the server (`501` without the flows plugin) |
| `GET` | `/flows/runs` | the run history |
| `GET` | `/flows/runs/{id}` | one run |
| `POST` | `/flows/runs/{id}/progress` | report a stage |
| `POST` | `/flows/runs/{id}/finish?status=` | close a run with its outcome |
| `DELETE` | `/flows/runs/{id}/nodes` | undo a run |

## Orchestrations

| Method | Path | Does |
|---|---|---|
| `GET` | `/orchestrations` | the configuration, problems, instance counts and diagrams |
| `GET` | `/orchestrations/instances?orchestration=&status=&subject=` | the instances |
| `POST` | `/orchestrations/{id}/start` `{subject}` | start instances |
| `POST` | `/orchestrations/next` `{orchestration}` | the next piece of work (`204` when nothing is ready) |
| `POST` | `/orchestrations/instances/{id}/signal` `{signal, message}` | signal a waiting instance |
| `POST` | `/orchestrations/instances/{id}/retry`, `/cancel` | retry a stuck instance / stop one |
| `DELETE` | `/orchestrations/{id}/instances` | forget every instance |

## Chats

| Method | Path | Does |
|---|---|---|
| `POST` | `/chats` `{focus, title}` | open a chat about node `focus`, or the whole knowledge base |
| `GET` | `/chats/{id}?since=n` | the chat and the messages after `n` |
| `POST` | `/chats/{id}/messages` `{text}` | the person says something |
| `POST` | `/chats/{id}/close`, `/reopen` | end it / open it again |
| `POST` | `/chats/{id}/branch` `{branch}` | move it to a branch |

## Branches

| Method | Path | Does |
|---|---|---|
| `GET` | `/branches?status=` | the branches, with change counts and the work item's state |
| `POST` | `/branches` `{name, description, work_item, chat, git_branch, id}` | open one |
| `GET` | `/branches/{id}` | one branch |
| `GET` | `/branches/{id}/diff` | what it changes, node by node |
| `GET` | `/branches/{id}/compare/{source}/{kind}/{node}` | one topic as is and to be: a line diff (`lines`) and every changed diagram as a pair (`diagrams`) |
| `GET` | `/branches/{id}/context?personas=` | the branch as "as is → to be" markdown |
| `POST` | `/branches/{id}/merge` `{force}` | merge it (`409` with the reason when refused) |
| `POST` | `/branches/{id}/abandon`, `/reopen` | close without merging / open again |
