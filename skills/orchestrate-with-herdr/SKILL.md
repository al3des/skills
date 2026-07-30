---
name: orchestrate-with-herdr
description: Run bounded Herdr work through Matt consultation, model routing, and autonomous monitoring.
disable-model-invocation: true
---

# Orchestrate with Herdr

Use this skill as the **orchestrator**: a control loop for bounded work in named Herdr tabs.

## Establish control

1. Verify the session is Herdr-managed, then read the installed Herdr CLI help needed for this run. Follow the `herdr` skill for exact commands, IDs, and safe pane control. **Completion:** `HERDR_ENV=1`, and the current agent/pane and workspace are known.
2. Name the current Herdr agent `orchestrator` with `herdr agent rename`, targeting the caller's pane or current agent explicitly. **Completion:** `herdr agent get orchestrator` identifies this coordinator.
3. Split a background sibling pane, preserving `$PWD` and using `--no-focus`. Start a fresh Pi agent named `matt` there with `--model openai-codex/gpt-5.6-terra`. Send a bounded prompt beginning with:

   ```text
   /skill:ask-matt <concise question about the next action>
   ```

   Include only paths, tracker, goal, and decision-relevant context. Ask Matt to inspect the relevant map/specification and open issues before recommending action. Wait for and read the response before planning. **Completion:** Matt's response identifies the inspected artifacts and issue IDs, recommends the next action from that evidence, and includes a literal example when useful.

## Plan before dispatch

4. Turn Matt's recommendation into bounded units and dependencies. Record each unit's owner, deliverable, dependencies, acceptance check, stop condition, and classification: independent, sequential, or HITL. Preserve Matt's literal examples unless repository evidence requires adaptation. **Completion:** every planned unit has all recorded fields and an acceptance check.
5. Before creating worker tabs or Pi sessions, invoke `/skill:model-routing` for every dispatchable unit. Capture its compact routing record, including model and thinking level; choose the cheapest adequate non-Sol route and escalate from evidence. **Completion:** every dispatchable unit has a recorded non-Sol route, and no worker topology exists.
6. For each eligible unit, create a background Herdr tab with a clear label, start a fresh named Pi session using the routed provider, model, thinking level, and tools, and give it its bounded prompt. **Completion:** each active unit has one named tab, one fresh named Pi agent, and a launch command matching its routing record.

When Matt recommends a skill, make that worker's task begin with:

```text
/skill:<skill-name> <Matt's recommendation>
```

Preserve Matt's recommendation verbatim, then append only the paths, acceptance check, or blocker needed to execute it.

## Monitor to the goal

Maintain a compact dependency record: `unit | tab/agent | depends on | state | acceptance | next action`.

At each checkpoint:

1. Inspect active named agents with Herdr. Prefer lifecycle-aware `agent wait`, `agent get`, and `agent read` over blind sleeps or a polling script.
2. For a settled agent, read its result and verify its acceptance evidence. Mark it complete only when that evidence is present.
3. When a completed unit unlocks a dependency, route that newly eligible unit first, then create its tab and fresh Pi session.
4. When an agent is blocked, inspect its transcript. Resolve ordinary, reversible orchestration questions from Matt's plan and repository evidence. For a HITL approval, choice, credential, or user-owned action, keep the task preserved and ask the user for the exact decision; the user retains that action.
5. On a failed acceptance check, give the same worker a concise evidence-based correction within its stop condition. Outside that condition, preserve its handoff and route a fresh replacement worker.
6. Once a worker's result, acceptance evidence, and needed handoff are captured, close the Herdr tab or pane created for it. Close Matt's pane after its recommendation is captured unless a later follow-up is planned. Retain the `orchestrator` session, active workers, retry candidates, and HITL-blocked work. **Completion:** completed topology is closed, while active or blocked work remains available.
7. Repeat the checkpoint until every unit has acceptance evidence, a preserved handoff is required, or a genuine HITL decision blocks progress. **Completion:** the dependency record accounts for every planned unit.

If the user is AFK, continue the monitoring loop. Return with the precise HITL decision when one is needed; otherwise return the final evidence after the loop completes.

For sequential and parallel patterns, read [`references/orchestration-patterns.md`](references/orchestration-patterns.md) when Matt's plan includes those dependencies.

Orchestration is complete only when the stated goal has acceptance evidence, a preserved handoff is required, or the user has received the exact remaining HITL decision with the affected work preserved.
