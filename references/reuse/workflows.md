---
document:
  status: DRAFT
  role: PRIMARY_SOURCE
---

# Workflows

A **workflow** names a group of cards. Dashboard draws a rectangle around its members.
Links determine execution order.

| Operation | Tool |
|---|---|
| Create and assign members by their card IDs | `workflow_create` with `flowNodes` |
| Read or rename a workflow | `workflow_get` or `workflow_patch` |
| Copy or paste a group and its cards | `workflow_copy` or `workflow_paste` |
| Change a card's group | `board_patch_card` with `workflow` |
| Delete a group and retain its cards | `workflow_delete` |

Each card belongs to at most one workflow on its board. Creating a group with already-grouped
members moves them. Use an empty workflow ID to ungroup a card.

Use `workflow_get` to read the saved `values` that members share across plays. New and pasted
workflows start with empty shared values. [Structured data](../structured-data.md)

For reuse on any board, see [copying](copying.md) and [description templates](templates.md).
