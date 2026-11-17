---
document:
  status: DRAFT
  role: PRIMARY_SOURCE
---

# Managed agents

Read to select, create, configure, publish or archive an agent. Task-level settings are in
[cards](../cards.md#settings); [runners](runners.md) covers machine selection. The `agent_*`
MCP tools perform these operations; HTTP equivalents are in [api-curl](../api-curl.md).

An agent names the harness, model, execution profile, standing instructions and selection
purpose used by AI tasks. A task's `settings.agent` is its managed **id**, never a harness name.

## Select an agent

Read `agent_list` before choosing an id. Match the task to the **published description** when
present. Descriptions guide selection and are never sent to turns; `instructions` are standing
orders sent beside task prompts. An unknown agent id is refused when writing task settings.
Only active, published agents can start new task conversations. Availability of the harness,
model and profile is checked on the runner.

## Fields

| Field | Rule |
|---|---|
| `id` | Service-assigned identity, used by task settings. |
| `name` | Required, unique across the store without regard to case. |
| `harness` | `claude`, `codex`, `muse` or `pi`. |
| `profileId` | Execution profile reference resolved by the runner, without credentials. Pi requires one unless a default Pi profile is configured. |
| `model` | Explicit model identifier before publication; Pi may resolve its local profile default. |
| `effort` | Published default effort. The task’s explicit effort takes priority. |
| `description` | When to choose this agent. An empty string clears it. |
| `instructions` | Standing orders for turns; independent of the selection description. |
| `published` | Latest immutable Git publication, with its commit, tag and APM lock digest; absent before the first publication. |
| `archived` | Hide from default listing and prevent new task conversations; false restores it. |
| `revision` | Used for conditional patches and publication. |

## Drafts and publications

Each agent is an independent Git/APM project. Its native `.apm/agents/*.agent.md` definition
contains model, default effort, description and instructions; `agent.json` contains identity,
display name, harness and profile reference. Dashboard’s Agentic resources panel edits the source
files and performs Git pull/push. New agents start with the assembly’s default resources.

Creation and configuration patches edit a **draft**. Set the native model and default effort, then publish explicitly with `agent_publish`.
Publication resolves APM dependencies and creates a Git commit and version tag.
Until publication, new conversations use the preceding publication or cannot start if none
exists. `agent_versions` reads the publication history, oldest first, including archived agents.

Each new conversation captures its publication and commit: model, execution profile reference and standing
instructions persist across replays, retries and manual messages. Later edits and publications
do not replace that snapshot. Saved harness and model values remain readable even if current
adapters no longer offer them. A new configuration does not retroactively update existing chats.

Archive with an agent patch setting `archived: true`, and restore with false. Request archived
agents explicitly when looking for one omitted from the default listing. Existing conversations
retain their captured publication. Published agents cannot be deleted; archive them to preserve
history. Deleting an unpublished agent leaves tasks that name it needing another agent.

The runner restores the committed package with APM’s frozen install. Every task execution
records its agent commit, resolved harness, model, effort, runner, conversation and turn.
The task’s Executions view exposes that immutable configuration.
