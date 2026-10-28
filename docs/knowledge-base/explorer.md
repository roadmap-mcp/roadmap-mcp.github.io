# The explorer

The **Knowledge Base** page (`/knowledgebase`) shows the knowledge layer: the work
breakdown first, then one lane per persona, each drawn as a tree.

!!! info "The page reads; it doesn't write"
    There is no form for adding a topic. Topics are written by agents over MCP or by
    an ingest: the knowledge comes from somewhere else, so a form would only be a
    slower way to enter it. Deleting stays, because deciding that something a
    machine wrote is wrong is a human call.

## Finding your way

- **The portal.** The page opens on one block per lane: its name, how many topics
  or work items it holds, the persona's description, the top of its tree, and for
  the breakdown the whole project's progress. Opening a block browses that lane:
  its tree on the left, the selected node on the right.
- **The breadcrumb.** Home (the portal), the lane, the folders holding the
  selected node, and the node itself. Each step but the last is a link.
- **The URL says where you are**: `?lane=work|<persona id>|graph`, then
  `node=source/kind/id` and `tab=`. Any lane or topic can be linked to, and the
  browser's back button walks back along the links you followed.
- **Type filters.** A persona's tree has one toggle per topic type it holds. Click
  to switch a type off or on, Ctrl-click to keep only that one, *All* to bring them
  all back.
- **The ground layer** is behind the graph icon in the header: the raw graph of
  every node, pointers included, filterable by kind. It's what the knowledge is
  built *from*, so it isn't the default view.

## A node

Selecting a node opens its pane, titled with its name and its type in brackets
("Sign-in [Use case]").

| Tab | Shows |
|---|---|
| *Details* | the short fields, then the document: a numbered table of contents (a click scrolls to the section) and every section with its diagrams and sketches drawn |
| *Progress* | units of work only: the state, progress, every flow run about it with the stages it reported and how it ended, and the log of state changes |
| *Links* | the node's relations, grouped as the lanes are, with one toggle per relation type to filter the list; a link opens the node at the other end |
| *Changes* | in a [branch](branches.md), for a new or changed topic: what the branch did to it |

A pointer node has **Refresh from source**: it re-reads the item from the system
that owns it and shows the live content under *From the source*, marked as not
stored.

A diagram has a **maximize** button that opens it full screen.

## The work breakdown

Each row shows its state and a progress bar (behind the header's progress toggle),
and a spinner marks an item with a flow run in progress. The page re-reads the
status every ten seconds, so a running flow's stages come in as they're reported.

A row's **⋯** menu:

- moves the item one state forward or back;
- lists the [flows](flows.md) bound to it, each with *Copy reference*: paste-ready
  MCP instructions for an AI, with the work item filled in, so whatever the run
  writes is linked to that item;
- *Copy context reference* (when the item has `context` links): instructions for an
  implementing AI to read the item and the topics it should start from.

## Branches and chat

The **branch picker** in the header switches the whole page to a
[branch](branches.md). The **Chat** button, or the chat icon on a selected node,
opens a [chat](chat.md) with an AI about the knowledge base.
