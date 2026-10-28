# Store schema

Each workspace's knowledge base is one SQLite file:

| Workspace | Config | Store |
| --- | --- | --- |
| `default` | `<base>/config.yaml` | `<base>/knowledgebase/ground.db` |
| `acme` | `<base>/workspaces/acme/config.yaml` | `<base>/workspaces/acme/knowledgebase/ground.db` |

Tables are created on open (`CREATE TABLE IF NOT EXISTS`), and a missing column is
added with an `ALTER`. Everything that varies by source lives in a JSON `details`
column, so a new source never changes the schema.

| Table | Key | Holds |
|---|---|---|
| `nodes` | `(source, kind, id)` | title, uri, `updated_at`, `ingested_at`, `details` JSON, `content` JSON |
| `links` | `(src..., dst..., kind)` | `attributes` JSON; the two ends may be in different sources |
| `ingest_runs` | `id` | a run log per source: status, counts, message |
| `flow_runs` | `id` | one run of a flow: flow and prompt version, trigger, work item, where the source stood, stages reported, status, counts |
| `chats`, `chat_messages` | `id` / `(chat, seq)` | a chat: subject, branch, the AI's last call; the messages with the pieces they showed |
| `orchestration_instances` | `id` | one walk through an orchestration: subject, branch, state, status, open run, history |
| `branches` | `id` | name, description, work item, git branch and commits, status (`open` / `merged` / `abandoned`), what its merge applied |
| `branch_nodes` | `(branch, source, kind, id)` | a branch's copy of a node; `deleted` hides it in the branch; `base_at` records when main's version was written |
| `branch_links` | `(branch, src..., dst..., kind)` | links the branch added, or removed (`deleted`) |

Reads go through two SQL views, `v_nodes` and `v_links`, built for the branch the
request is in, so the same queries serve main and every branch. Merging a branch
applies its copies to `nodes` and `links` in one transaction.

`content` is null on a pointer node, and a write that carries no content leaves the
stored content alone, so refreshing a node's label can never wipe what somebody
wrote. Re-ingesting updates rows in place, so ids stay stable and links keep
working.
