---
document:
  status: DRAFT
  role: PRIMARY_SOURCE
---

# Task description templates

Read when authoring or filling a task description template. [Structured data](../structured-data.md)
defines placeholders evaluated when an AI task runs.

## Editor description templates

Dashboard loads `.temp.md` files from the store's flat `kiao-user-config/flow-node-templates/`
folder. They use the template-lang format; consult that skill when editing template syntax.
Selecting and instantiating a template **replaces** the task description with the result.

`AI.temp.md` and `Human.temp.md` are recognized by task type. When missing, Dashboard seeds them
on first opening a board: AI wraps the description with the default standing wording (be concise,
work at the strategic level, commit without pushing), while Human keeps the description alone.
Existing store templates can differ; inspect them when matching the editor's wording.

| Variable | Value |
|---|---|
| `${flowNodeDescription}` | Current task description, so the template can wrap it. |
| `${flowNodeTitle}` | Task title. |

Templates are applied by the **editor**, not by Play. Cards created through MCP or HTTP keep the
body sent by the caller. To produce the same wording as the editor, read the relevant store
template and compose the intended body before creating or patching the card.

## Instantiate copied descriptions

Use **Copy as JSON** on a card, or **Copy workflow as JSON** on a workflow, then choose
**Instantiate here** on the destination board. For a group of cards, fill each templated description
in turn. Dashboard creates the copy after the last confirmation.

A Markdown description can contain variables such as `${project:text}`. Turn on
**Multi-part template** to author a TemplateLang collection with optional sections. Use
[copying](copying.md) for how links and execution settings travel with the copy. The MCP and
HTTP creation and paste operations store the descriptions supplied by the caller.
