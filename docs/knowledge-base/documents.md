# The document format

A topic's long text is a **document**: a tree of titled **sections** whose leaves
are **fragments**. The section titles are the outline, so the explorer can show a
numbered table of contents above the topic. Each fragment stays a paragraph, a
list, a table or a code block.

| Fragment | Holds | Drawn by |
|---|---|---|
| `markdown` | text | the explorer (raw HTML is shown as text) |
| `plantuml` | a diagram, `@startuml` ... `@enduml` | the configured PlantUML server |
| `mermaid` | a diagram (`erDiagram`, `flowchart LR`, ...), with no code fence | the browser |
| `wireframe` | a sketch of a screen, in PlantUML Salt | the browser |

```json
"content": {
  "fields": {"step": "Sign in and application shell"},
  "document": [
    {"type": "section", "title": "Backend", "children": [
      {"type": "markdown", "text": "One feature package, `auth`."},
      {"type": "section", "title": "REST endpoints", "children": [
        {"type": "markdown", "text": "- `POST /api/auth/sign-in` ..."}]}]}]}
```

How a document fits the type:

- On a **structured** type, `fields` holds the short values, and each `long-text`
  field becomes a top-level section titled with the field's label.
- On an **unstructured** type, the template's headings are the sections.
- A field of type `plantuml`, `mermaid` or `wireframe` is a section that must hold
  a fragment of that type. This is how a type asks for a diagram: a *Sequence
  diagram* on a design topic, a *Database diagram* as a Mermaid `erDiagram`.

Every diagram has a toggle to show its source. One that doesn't parse shows its
error and its source.

## Wireframes

A `wireframe` fragment is a screen drawn as a sketch, not a screenshot: what it
holds and where, with sample values. Its source is **PlantUML Salt**, which AI
agents already know, but it's drawn in the browser in a hand-drawn style, so a
screen's design is never sent to an outside server.

```
{+
  {* MyApp | Discover | Settings }
  <b>Sign in
  ==
  Email    | "you@example.com   "
  Password | "********          "
  [X] Remember me
  [ Log in ] | [Register]
}
```

| Salt | Draws |
|---|---|
| `[Text]` · `"text   "` · `^Choice^` | button · input (the spaces set its width) · dropdown |
| `[ ] a` / `[X] a` · `( ) a` / `(X) a` | checkbox · radio button |
| text, `<b>bold</b>`, `.` | label · bold label · empty cell |
| `--` `==` `..` `~~` on a row of their own | separator |
| `{` `{+` `{#` `{!` `{-` | grid · framed · table · column lines · row lines |
| `{^"Title"` · `{/ A \| B }` · `{* A \| B }` · `{T` | group box · tabs · menu bar · tree (`+`, `++`) |

Anything not recognised is drawn as text, so a sketch always draws. On write, the
source must be one balanced `{ ... }` block, optionally inside `@startsalt` ...
`@endsalt`.

## What is checked

Content is always checked against its type: a structured node missing a required
field, or naming an unknown one, is refused, and so is content on a node whose
kind isn't a type.

Writes made under a [flow run](flows.md) are held to the document format as well:
no `body`, no long-text field values, no markdown heading inside a fragment (a
heading is a section), and no diagram field without its diagram. A `plantuml`
fragment must run from `@start...` to `@end...`, and a `mermaid` one must be bare
source. Each refusal says why, so the agent can correct the write and try again.
