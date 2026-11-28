---
document:
  status: DRAFT
  role: PRIMARY_SOURCE
---

# Managed agents

Read to select, install, configure, publish or delete an agent. Task settings are in
[cards](../cards.md); [runners](runners.md) covers machine selection. The `agent_*` MCP tools
perform these operations; HTTP equivalents are in [api-curl](../api-curl.md).

An agent provides standing instructions and assembles resources for related tasks. A harness
executes each turn and offers its model and permission options. An agent can support several
harnesses and supply execution defaults. A task's `settings.agent` is the agent's stable **id**.
agentic-resource-manager discovers its Git working tree, including manual clones.

## Select an agent

Read `agent_list` before choosing an id. Match the task to the **published description** when
present. Descriptions guide selection; `instructions` are standing orders sent beside task prompts.
Published agents can start task conversations. The runner checks harness, model and profile
availability. Choose the agent while defining the task.

A missing agent retains its task reference. Change the task's agent or reinstall its repository
to restore the same identity. Other settings of an existing task remain editable while it is missing.

## Fields

| Field | Rule |
|---|---|
| `repository` | Git URL for `agent_create`; cloning retains the repository's identity and publications. |
| `id` | Generated for a new local agent; retained across cloning and reinstallation. |
| `name` | Required for a new local agent; unique without regard to case. |
| `harness` | `claude`, `codex`, `muse` or `pi`; required for a new local agent. |
| `profileId` | Runner execution profile reference, without credentials. Pi uses its configured default when unset. |
| `model` | Explicit before publication; Pi may resolve its local profile default. |
| `effort` | Published default effort; the task's explicit effort takes priority. |
| `description` | Selection guidance; an empty string clears it. |
| `instructions` | Standing orders for turns. |
| `published` | Latest Git publication, with commit, tag and APM lock digest. |
| `revision` | Used for conditional patches and publication. |

## Source and publication

`agent_create` with a Git `repository` URL clones the project and resolves dependencies with APM.
With no repository URL, supply a name and harness to initialize a local Git repository.
Native `.apm/agents/*.agent.md` definitions contain model, effort, description and instructions;
`agent.json` contains identity, display name, harness and profile. `apm.yml` declares dependencies.
Dashboard's agent editor and Agentic resources panel edit these working-tree files.

Skills and MCP modules with a Git remote use Git references in `apm.yml`. **Local** in the agent
editor temporarily uses the working tree. Clear those checkboxes before publishing. **Use local
draft** in **New conversation** captures the draft and its current resources for a test conversation.

Set model and default effort, then call `agent_publish`. Publication installs dependencies,
compiles, commits, tags and pushes the release when a remote exists. If the tool returns
`pushConfirmation`, explain that the listed local dependencies require Kiao. After the owner
confirms, repeat the call with the returned token as `confirmation`. `agent_versions` reads the
history, oldest first. Existing conversations retain their selected publication; changing a task's
agent forks its conversation onto that agent.

## Delete a local clone

Call `agent_delete` with only the id to inspect `deletion` risks and its `confirmation` fingerprint.
Present the warnings about missing remotes, local changes, unpushed refs or unavailable remotes.
After the owner confirms, call `agent_delete` with that fingerprint. A repository change requires
reviewing its current state again. Tasks retain their agent reference after deletion.

Workers install committed packages through APM's frozen install and retain a verified local cache.
Every task execution records the agent commit, resolved harness, model, effort, runner,
conversation and turn. Dashboard's Executions view exposes that immutable configuration.
