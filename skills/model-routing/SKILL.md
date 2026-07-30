---
name: model-routing
description: Route bounded orchestration work to the cheapest adequate Codex model. Use when decomposing agent work, choosing a model or thinking level, launching Pi workers, or deciding whether to escalate, stop, or hand off; other orchestration skills should consult it before launching workers.
---

# Model routing

Use a **bounded** unit: one deliverable, the minimum required context, an observable acceptance check, and a stop condition. Decompose before routing; escalate only from concrete failure evidence.

## Route the unit

Use this table as the single source of truth:

| Model | Best-fit work | Starting effort |
|---|---|---|
| `gpt-5.6-luna` | Bounded extraction, classification, summaries, mechanical checks, and low-risk scripted edits | `low` |
| `gpt-5.6-terra` | Documentation, structured writing, product copy, and content synthesis | `low`; `medium` for multi-source synthesis |
| `gpt-5.4-mini` | Narrow low-risk implementation, focused test fixes, scaffolding, and mechanical refactors with clear acceptance commands | `low`; `medium` when several files interact |
| `gpt-5.5` | Default implementation worker for coding, tests, prototypes, routine debugging/review, and compatibility tooling | `low` for mechanical work; otherwise `medium`; `high` after a reasoning failure |
| `gpt-5.6-sol` | Control-plane work, architecture/security/legal adjudication, and bounded high-risk final review | `medium`; `high` when ambiguity warrants it |

Implementation work starts on Mini when narrow and low-risk, otherwise GPT-5.5. Sol is reserved for coordination and bounded adjudication; a write-capable Sol exception needs explicit user direction or evidence that `gpt-5.5:high` failed for a reasoning-related cause and the unit cannot be decomposed further; record the reason in the routing line. Compatibility-sensitive implementation starts on GPT-5.5, with a separate Sol review when risk warrants it.

For Codex models, use explicit `--thinking low|medium|high`. `low` is the lowest predictable explicit tier for the listed catalog. Raise one effort rung after a concrete miss; switch models when capability or risk was misrouted. Use `xhigh` or `max` only after `high` fails for a reasoning-related cause, and verify support in the current catalog. Verify the catalog before the first launch:

```bash
pi --list-models gpt-5
```

Route unavailable Luna, Terra, or Mini work to GPT-5.5 when adequate. Route unavailable GPT-5.5 work to Mini when adequate. Route unavailable Sol work to `gpt-5.5:high` with reduced assurance; otherwise stop and request rerouting.

## Bound the worker

Before launch, define all of these in the worker prompt or routing record:

- one deliverable;
- minimum paths, issues, and spec sections;
- acceptance command or observable artifact;
- expected duration and context budget;
- assigned tools and explicit scope boundaries;
- stop condition and handoff output.

For segmented implementation, target **under 80k active context / roughly 30% of a 272k window**. At **120k active context or 45%, whichever comes first**, preserve the worktree, write a concise handoff, and start a fresh worker for the remaining unit. Handoff earlier when investigation repeats, scope expands, unrelated coordination begins, or twice the expected duration passes without approaching acceptance. Compaction is a signal to hand off, not a way to extend an oversized unit.

## Execute and validate

1. Split the request into units, each with one deliverable, risk, minimum context, acceptance check, budget, and handoff. Shared-worktree units run serially; independent read-only analysis may run in parallel. **Completion:** every unit has an owner, route, budget, and acceptance check.
2. Keep tiny dependent work in the coordinator and remove fan-out whose duplicated context costs more than its parallelism saves. **Completion:** each dispatched worker has a distinct payoff and no unnecessary dependency.
3. Prepare a narrow prompt and minimum tool set. Give read-only workers read/search tools where possible; assign orchestration or tracker work only to a worker whose single deliverable is that work. When exactly one skill is required, load it with `--no-skills --skill <path>` and invoke it as `/skill:<name>`. **Completion:** the prompt names the deliverable, boundaries, tools, and handoff.
4. Apply the route at **process creation**. Launch every worker in a fresh, named Pi session with explicit provider, model, thinking level, and tools. **Completion:** before task execution, the process is named and its exact launch command matches the routing record.

   Direct launch:

   ```bash
   pi --provider openai-codex \
     --model <model-id> \
     --thinking <level> \
     --tools <minimum-required-tools> \
     --name "<short worker label>" \
     "<bounded task or /skill:name invocation>"
   ```

   Herdr launch, before sending the task to the pane:

   ```bash
   herdr pane run "$pane" \
     "pi --provider openai-codex --model <model-id> --thinking <level> --tools <minimum-required-tools> --name '<short worker label>'"
   ```

5. Monitor the budget using the footer/session totals and transcript at checkpoints. Interrupt before the hard context ceiling and hand off when a stop condition fires. **Completion:** the worker either reaches acceptance or leaves a concise handoff with preserved worktree state.
6. Validate material risk once in a separate bounded pass. Use GPT-5.5 for ordinary code; use Sol for security, architecture, legal, or difficult compatibility adjudication. Give the validator the diff, acceptance criteria, and evidence rather than the implementation transcript. **Completion:** validation records pass/fail findings and unresolved risk.
7. Synthesize worker reports against repository evidence. **Completion:** the coordinator records the final result, acceptance evidence, and any handoff or unresolved risk.

For long compatibility work, use separate bounded phases: template adaptation, automated validation, consumer rendering, and verdict. A single worker owns only the phase assigned to it.

Routing is complete when every unit has a route, one owner, explicit boundaries and stop conditions, a named process where dispatched, acceptance evidence, and (for material risk) a separate validation result.

Keep the routing record compact:

```text
worker | deliverable | risk | model:effort | context/time budget | validation
```

When Pi or the model catalog changes, recalibrate against [`references/pi-thinking-efficiency-research.md`](references/pi-thinking-efficiency-research.md) and current Pi docs/source, replacing stale rules in place.
