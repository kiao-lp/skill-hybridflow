---
document:
  status: DRAFT
  role: PRIMARY_SOURCE
---

# Structured task data

Read when a task needs earlier results, shared workflow values, or typed JSON values and schemas.
These fields and contracts are shared by MCP and HTTP. [Operations](operations.md) covers safe
patches; [chains](chains.md) describes execution order and conversation options.

## Choose the handoff

Identify the fields each consumer needs, including artifact identity, revision and verdict when
acting on validated work. Use a minimal schema for results that drive later decisions. Shared
folders and conversation history provide context, but cannot identify the exact result approved.
Independent tasks need no output schema merely because they are linked.

Trace the result through each intervening step. Use predecessor placeholders for direct handoffs
and shared workflow values for fields that cross Human steps or separate plays. To recover the
stored result of a specific task, read its `output` with `board_get_card` and verify completion
and artifact identity. Check the freshness of shared values when replaying part of a workflow.

## Schema values

A schema is a Value card. This is the card object, placed in `flowNodes` for creation:

```json
{
  "title": "Validation result",
  "type": "value",
  "valueType": "json-schema",
  "value": {
    "type": "object",
    "properties": {
      "revision": { "type": "string" },
      "approved": { "type": "boolean" }
    },
    "required": ["revision", "approved"],
    "additionalProperties": false
  }
}
```

Keep the returned id. The schema and task use different type fields:

| Field | Meaning |
|---|---|
| Value `valueType: ""` | Untyped JSON. |
| Value `valueType: "json-schema"` | The value itself is a JSON Schema. |
| Value `valueType: "<schema-id>"` | JSON typed by another schema value on the **same board**; a value cannot type itself. |
| AI or Human task `outputType: "<schema-id>"` | Export typed by a schema value from **any board**. Use the id, never an inline schema or literal `"json-schema"`. |

`board_schema_values` discovers schemas across non-archived boards and cards, with `id`, `title`,
`value`, `boardSlug` and `boardName`. Over HTTP, use `GET /schema-values`; direct store HTTP
also supports `GET /schema-values/{id}`.

The value editor diagnoses schemas against the meta-schema and ordinary values against their
selected schema. Storage checks JSON syntax and type references; successful storage alone does
not prove schema validity or value conformance. AI exports are validated by the runner, and
Human submissions by the Dashboard editor.

## Configure producers and consumers

Select or create a schema, then patch the producer's `outputType` with its id. Link the consumer
and put the fields it needs in its description or criteria. Example producer patch:

```json
{"outputType":"<schema-id>","onSuccess":{"flowNode":"<consumer-id>","mode":"end"}}
```

Example consumer patch:

```json
{"body":"Act on revision {{input.revision}} only if approved is true: {{input.approved}}."}
```

Use actual ids and the revision of each edited card. Preserve existing `onSuccess` options when
replacing that object, and respect connection limits. `outputType: ""` clears the selection.

## AI export contract

At task start, the runner resolves and compiles the schema, then appends it and export
instructions to the prompt. Draft 2020-12 is the default; supported explicit drafts and local
`$ref` definitions work. External files and network resources are not fetched. An unavailable
or invalid schema fails the task before a turn is sent.

The **final message** must contain exactly one complete fenced block labelled `output`:

````markdown
```output
{"revision":"abc123","approved":true}
```
````

Its payload is the JSON value matching the schema: no comments, extra wrapper or string encoding.
Objects, arrays and scalars work when allowed by the schema; prose may accompany the block.
A `json` fence, tool result or progress message cannot export data. Missing, incomplete, duplicate,
malformed or schema-invalid exports mark the task `error` and stop succession. The `output`
export applies to successful AI tasks with an `outputType`.

## Predecessor inputs

The runner replaces references when a card starts. The saved card keeps its original
references. Human tasks also show predecessor output in the Work panel.

| Placeholder | Reads |
|---|---|
| `{{input.revision}}` | A property. |
| `{{input.person.name}}` | A nested property. |
| `{{input.items.0}}` | An array element, indexed from zero. |
| `{{input}}` | The whole JSON value. |

Strings insert directly; other values use JSON syntax. Replacement is one pass: placeholder-like
text within received strings stays data. Missing input, a missing property or an out-of-range
index marks the task `error` before a turn. Without input placeholders, a task can run without
predecessor data. Only `done`, non-archived predecessors supply it. Direct Play can consume the
predecessor's last successful export; verify freshness when restarting part of a workflow.
A Value's payload never becomes task input merely through placement beside it.

AI tasks can use references in the prompt fields and working directory. Human tasks can
use them in the title, description, criteria and working directory. Scheduler durations and
datetimes and Bitbucket identifiers and target statuses also accept references. A field can
combine literal text with references, for example `workspace/repo#{{input.pullRequestId}}`.
The runner checks the resulting value before using it. Notes can show shared workflow values.

## Shared workflow values

Group producers and consumers with `workflow_create` or set their `workflow` field with
`board_patch_card`. A workflow keeps its shared values across plays. `workflow_get` returns the
workflow's `id` and `name` plus saved custom fields under `values`. `board_get` exposes the same
fields in `workflows`. The equivalent HTTP reads use the board and workflow endpoints.

Put a complete fenced block in an AI task's `body` to stage one field before rendering its
prompt. Ask an AI task to return a block in its final reply when it computes the field.

````markdown
```currentWokflow.message
Ready for review
```
````

Workflow assignments work on any AI task in a workflow. Plain text becomes a string and valid
JSON keeps its JSON type. The runner commits fields with successful task completion. Final reply
values override assignments to the same fields in the description. Reserve `name` and `id` for
the built-in read-only metadata.

Use `{{currentWorkflow.message}}`, `{{currentWorkflow.name}}` or `{{currentWorkflow.id}}` in fields
that accept references. Use a dotted path for nested JSON or `{{currentWorkflow}}` for the whole
object. The runner also supplies the object to the AI agent. Missing values stop execution
and remain visible as references in Notes and in the Human Work panel. Exclusive gateways can read the
same fields in CEL, for example `currentWokflow.approved == true`. Both `currentWokflow` and
`currentWorkflow` work in fences, placeholders and conditions.

## Role references

Create a card with `type: "role"`, a display `title`, a `roleId` and a `people` array of
single-line strings. Every active Role ID is unique on its board and uses letters A–Z, digits,
underscores or hyphens. Use `{{role.reviewers}}` for the role with ID `reviewers` in any field
that accepts value references. It inserts the people in their stored order with a comma and a
space between entries. A role with no people inserts an empty string. Archived roles are
outside the lookup. In CEL, read `role.reviewers` or `role['release-reviewers']`.

## Data across control cards

Data passes independently of conversation `mode` and `compact`. Schedulers and parallel gateways
with one predecessor pass the value through. An exclusive gateway takes the output of the
predecessor that activated it. It forwards that value to its successor and exposes it as `input`
when evaluating CEL routes. Direct Play uses the most recently completed predecessor. A limiter
forwards the value to the branch selected by its counter. [Chains](chains.md)

A parallel join exposes an array in **board order**, with `null`
for any branch without an export. It has no input when no predecessor has output. Each outgoing
branch receives the Gateway's stored output and can read `{{input.0.revision}}`, for example.
Board order is not arrival order or the Gateway's outgoing branch order; establish the mapping
and recheck it after structural changes.

## Human results

Reaching a Human task marks it `todo`. Work opens Dashboard's WorkOnTask panel and moves `todo`
to `running`. With an `outputType`, Submit requires JSON matching the selected schema. Without
one, submission is raw text stored as a JSON string. A Human result is its own submission, never
an automatic copy of its predecessor's result.

Submit saves `output` and `done` and offers a Human successor as `todo`. Dashboard can start a
ready automatic successor when chat execution is available; see [Human steps](chains.md#human-steps).
The successor reads this Human output using the same input rules. A later task can combine the
approval in `input` with an earlier AI task's saved revision in `currentWokflow`. To consume a
specific producer's output directly, name that producer in the instructions.

## Output lifetime

The read-only `output` holds the last successful result. A new play clears it, as does changing
the card's type or output schema. Do not forge it with a patch or call `/played` manually.
Deleting a referenced schema retains its missing id on the task and refuses its next play.
A schema used by outputs or typed values cannot be retyped until its references are removed.
Before reporting a workflow ready, read back schema selections, consumer fields and the complete
path of each required result, including Human boundaries and joins.
