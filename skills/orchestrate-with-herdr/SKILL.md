---
name: orchestrate-with-herdr
description: Orchestrate bounded work in named Herdr tabs through a concise Matt consultation, model routing, and autonomous monitoring.
disable-model-invocation: true
---

# Orchestrate with Herdr

Use this skill as the **orquestrator**: a control loop that keeps work moving until the goal has acceptance evidence or a human decision is necessary.

## Establish control

1. Verify the session is Herdr-managed, then read the installed Herdr CLI help needed for this run. Follow the `herdr` skill for exact commands, IDs, and safe pane control. **Completion:** `HERDR_ENV=1`, and the current agent/pane and workspace are known.
2. Name the current Herdr agent `orquestrator` with `herdr agent rename`, targeting the caller's pane or current agent explicitly. **Completion:** `herdr agent get orquestrator` identifies this coordinator.
3. Split a background sibling pane, preserving `$PWD` and using `--no-focus`. Start a fresh Pi agent named `matt` there with `--model openai-codex/gpt-5.6-terra`, then submit exactly this shape of prompt:

   ```text
   /skill:ask-matt <concise question about the next action>
   ```

   Tell Matt to inspect the actual source of truth before recommending anything: read the relevant map or specification and inspect the open issues. Give only the paths, tracker, goal, and minimum decision-relevant context needed to begin that legwork; do not substitute a summary for the artifacts. Wait for the answer and read it before planning. **Completion:** Matt has inspected the map/specification and open issues, then recommended the next action, including a literal example when one is useful.

## Plan before dispatch

4. Turn Matt's recommendation into bounded units and dependencies. For each unit, capture its deliverable, acceptance check, stop condition, and whether it is independent, sequential, or HITL. Keep Matt's examples literal unless repository evidence requires adaptation. **Completion:** every planned unit has an owner condition and an acceptance check.
5. Invoke the installed model-router skill *before creating worker tabs or Pi sessions* (in this repository, `/skill:model-routing`). Route every worker through it, including model and thinking level. Exclude `gpt-5.6-sol`: choose the cheapest adequate non-Sol route, escalating from evidence rather than defaulting to Sol. **Completion:** every dispatchable unit has a non-Sol route and no worker topology has been created yet.
6. For each now-eligible unit, create a background Herdr tab with a clear label, start a fresh named Pi session using its fully qualified routed model (`openai-codex/<model>`), and give it only its bounded prompt. **Completion:** each active unit has one named tab, one fresh named Pi agent, and its routed launch command.

When Matt recommends a skill, make that worker's task begin with:

```text
/skill:<skill-name> <Matt's recommendation>
```

Do not dilute it with a retelling. Add only the minimum paths, acceptance check, or blocker needed to make Matt's recommendation executable.

## Monitor to the goal

Maintain a compact dependency record: `unit | tab/agent | depends on | state | acceptance | next action`.

At each checkpoint:

1. Inspect active named agents with Herdr. Prefer lifecycle-aware `agent wait`, `agent get`, and `agent read` over blind sleeps or a polling script.
2. For a settled agent, read its result and verify its acceptance evidence. Mark it complete only when that evidence is present.
3. When a completed unit unlocks a dependency, route that newly eligible unit first, then create its tab and fresh Pi session.
4. When an agent is blocked, inspect its transcript. Resolve ordinary, reversible orchestration questions from Matt's plan and repository evidence; for a HITL approval, choice, credential, or user-owned action, preserve the task and ask the user for that exact decision. Never approve or perform the HITL action on the user's behalf.
5. On a failed acceptance check, give the same worker a concise evidence-based correction when it remains within its stop condition; otherwise preserve its handoff and route a fresh replacement worker.
6. Repeat until all units have acceptance evidence, a preserved handoff is required, or a genuine HITL decision blocks progress.

If the user is AFK, remain in this loop. Do not send interim progress updates or return control merely because a worker settles; return only for a genuine HITL decision or the final result.

### Sequential example: Wayfinder map tickets

Matt recommends resolving map tickets in dependency order: reproduce ticket A, implement and validate A, then resolve ticket B against A's result. Route A before creating its tab; start A and wait on its named agent. On completion, read its evidence and run its acceptance command. Only then route B, create B's tab, and start a fresh B session with A's concise handoff. If either worker asks for product intent, leave it blocked and ask the user the narrow question; otherwise continue through every unlocked ticket without returning early.

### Parallel example: independent investigation and implementation

Matt recommends a read-only investigation and an independent implementation. Route both before creating either tab, then start one fresh worker in each clearly named tab. Monitor both agents by lifecycle state. When the investigation finishes, feed only its relevant finding to a dependent worker if it unlocks one; do not interrupt the independent implementation. A blocked HITL task remains blocked even if another worker is done. Finish by validating each unit's own acceptance evidence and the combined goal.

Orchestration is complete only when the stated goal has evidence, or the user has been given the exact remaining HITL decision with the affected work preserved.
