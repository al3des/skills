---
name: orchestrate-with-herdr
description: Orchestrate conversation-specified Wayfinder and Implement tickets in Herdr.
disable-model-invocation: true
---

# Orchestrate with Herdr

Use this skill as the **orchestrator**: a control loop for conversation-specified tickets in named Herdr tabs.

## Establish control

1. Verify the session is Herdr-managed, then read the installed Herdr CLI help needed for this run. Follow the `herdr` skill for exact commands, IDs, and safe pane control. **Completion:** `HERDR_ENV=1`, and the current agent/pane and workspace are known.
2. Name the current Herdr agent `orchestrator` with `herdr agent rename`, targeting the caller's pane or current agent explicitly. **Completion:** `herdr agent get orchestrator` identifies this coordinator.
## Plan before dispatch

3. Read the conversation history for a **supported ticket directive**: an association between `wayfinder` or `implement` and one ticket URL, with any stated dependency or parallelism. When history supplies no supported ticket directive, report that it lacks a supported ticket workflow, suggest `/ask-matt`, and stop. **Completion:** either at least one supported ticket directive is identified, or the user has the exact suggested next action.
4. Turn every supported ticket directive into a bounded unit. Record its skill, ticket URL, dependencies, state, acceptance evidence, and stop condition. Place every other handoff in an out-of-scope record; it remains undispatched. **Completion:** every conversation-specified handoff is accounted for as either a supported unit or out-of-scope work.
5. Before creating worker tabs or Pi sessions, invoke `/skill:model-routing` for every eligible supported unit. Capture its compact routing record, including model and thinking level; choose the cheapest adequate non-Sol route and escalate from evidence. **Completion:** every eligible supported unit has a recorded non-Sol route, and no worker topology exists.
6. For each eligible unit, create a background Herdr tab with a clear label and start a fresh named Pi session using the routed provider, model, thinking level, and tools. Send exactly one task prompt:

```text
/skill:<wayfinder|implement> <ticket-url>
```

**Completion:** each active unit has one named tab, one fresh named Pi agent, a launch command matching its routing record, and the exact skill-and-ticket prompt.

## Monitor to the goal

Maintain a compact dependency record: `unit | tab/agent | depends on | state | acceptance | next action`.

At each checkpoint:

1. Inspect active named agents with Herdr. Prefer lifecycle-aware `agent wait`, `agent get`, and `agent read` over blind sleeps or a polling script.
2. For a settled agent, read its result and verify its acceptance evidence. Mark it complete only when that evidence is present.
3. When a completed unit unlocks a dependency, route that newly eligible unit first, then create its tab and fresh Pi session.
4. When an agent is blocked, inspect its transcript. Resolve ordinary, reversible orchestration questions from the supported directive and repository evidence. For a HITL approval, choice, credential, or user-owned action, keep the task preserved and ask the user for the exact decision; the user retains that action.
5. On a failed acceptance check, give the same worker a concise evidence-based correction within its stop condition. Outside that condition, preserve its handoff and route a fresh replacement worker.
6. Once a worker's result, acceptance evidence, and needed handoff are captured, close the Herdr tab or pane created for it. Retain the `orchestrator` session, active workers, retry candidates, and HITL-blocked work. **Completion:** completed topology is closed, while active or blocked work remains available.
7. Repeat the checkpoint until every supported unit has acceptance evidence, a preserved handoff is required, or a genuine HITL decision blocks progress. **Completion:** the dependency record accounts for every supported unit.

If the user is AFK, continue the monitoring loop. Return with the precise HITL decision when one is needed; otherwise return the final evidence and the out-of-scope record after the loop completes.

Orchestration is complete when every supported unit has acceptance evidence, a preserved handoff is required, or the user has received the exact remaining HITL decision with the affected work preserved.
