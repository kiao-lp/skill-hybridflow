---
document:
  status: DRAFT
  role: PRIMARY_SOURCE
---

# Archive and delete

Read when archiving or deleting objects. Apply the shared [editing rules](../operations.md)
and refresh the affected references before removal.

Archiving a board or card is reversible. Board listings omit archived boards by default; card
archives hide cards and their links. Stored links from archived cards still occupy connection
slots. Unlink them explicitly if rewiring requires it.

Deletes are permanent. Delete only within the user's authorized scope; otherwise obtain agreement
on the objects to remove. Inspect references before deleting:

| Object | Effect |
|---|---|
| Board | Removes its cards and workflows. |
| Cards | Removes incoming links from surviving cards. Deleting a schema leaves missing schema ids on task outputs, refusing their next play until repaired. |
| Workflow | Ungroups its members and retains them. |
| Agent | Inspect clone deletion risks, obtain owner confirmation, then delete using its state fingerprint. Tasks keep the reference; reinstall its repository or choose another agent. |
| Runner | Check and replace any task pins to that runner. |
