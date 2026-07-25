---
name: model-routing
description: Route orchestrated work across OpenAI Codex GPT models and thinking levels. Use when splitting work among agents, spawning pi workers, choosing models or reasoning effort, or reducing context and token use in a multi-agent run; other orchestration skills should consult this before launching workers.
---

# Model routing

Climb a **budget ladder**: choose the cheapest adequate model and lowest explicit effort, then escalate only when evidence warrants it. Risk overrides surface simplicity.

## Assign the worker

| Model | Route here | Starting effort |
|---|---|---|
| `gpt-5.6-luna` | Bounded extraction, classification, summaries, support triage, mechanical low-risk checks | `low` |
| `gpt-5.6-terra` | Documentation, structured writing, product copy, content planning | `low`; `medium` for multi-source synthesis |
| `gpt-5.5` | Default general coding, routine review, coordination, workload planning, mixed everyday work | `low` for mechanical changes; otherwise `medium` |
| `gpt-5.6-sol` | Hard debugging, security, architecture, difficult reasoning, compatibility-sensitive code, legal or business risk review | `medium`; `high` when ambiguous or high-risk |

For these Codex models, Pi maps `minimal` to provider effort `low`; standardize on `low`. Thinking `off` merely omits an explicit reasoning setting, so `low` is the lowest predictable route. Numeric `thinkingBudgets` apply to budget-based adapters, while Codex effort is controlled with `--thinking`.

Use `xhigh` or `max` after `high` fails for a reasoning-related cause. GPT-5.5 supports through `xhigh`; Luna, Terra, and Sol also support `max`. When an output misses acceptance criteria, raise one effort rung if the model still fits the workload; switch models when capability or risk was misrouted.

## Orchestrate

1. Split the request into independent work units. Record each unit's deliverable, risk if wrong, minimum context, and acceptance check. **Gate:** every proposed worker owns one bounded unit.
2. Remove fan-out whose parallelism or isolation does not repay duplicated prompts and context. Keep small dependent work in the orchestrator. **Gate:** shared context serves distinct deliverables rather than duplicate attempts.
3. Assign model and effort from the budget ladder. GPT-5.5 is the baseline for unmatched work. **Gate:** every unit has one route, and every high-risk route uses Sol or records reduced assurance.
4. Before the first launch, run `pi --list-models gpt-5`. Route unavailable Luna or Terra work to GPT-5.5, unavailable GPT-5.5 work to Sol, and unavailable Sol work to `gpt-5.5:high` with reduced assurance. **Gate:** every selected model and effort is supported by the current catalog.
5. Launch each independent unit in a fresh named `pi` session. Reuse that worker session for tightly related turns; start fresh when prior history is unrelated. **Gate:** each worker has a visible identity and one cohesive context.
6. Give each worker only relevant paths, issue/spec references, acceptance criteria, and an output contract. Use a narrow tool allowlist. Preserve project context rules; when exactly one skill is required, load it with `--no-skills --skill <path>` and invoke it as `/skill:<name>`. **Gate:** every launch contains sufficient task context and excludes unrelated payload.
7. Validate material risk once: GPT-5.5 for ordinary validation, Sol for security, architecture, legal, or compatibility-sensitive validation. A Luna result that can affect code, external claims, or business decisions requires validation. Seek duplicate opinions when independent adjudication is the deliverable. **Gate:** each material low-tier result has one named validation path.
8. Synthesize once against repository evidence and acceptance criteria. For repeated routes, inspect the footer or `/session` totals (`↑`, `↓`, `R`, `W`, `CH`, context %) and move future work down the ladder when quality holds. **Gate:** acceptance is resolved and representative routes have evidence for the next calibration.

Routing is complete when every worker has a model, effort, bounded context, and acceptance check; duplicated context has been removed; and every material low-tier result has a validation path.

## Lean launch

```bash
pi --provider openai-codex \
  --model <model-id> \
  --thinking <level> \
  --tools <minimum-required-tools> \
  --name "<short worker label>" \
  "<bounded task or /skill:name invocation>"
```

For a read-only worker, prefer `--tools read,grep,find,ls`. Add `edit`, `write`, or `bash` when its deliverable requires them. Ask report-only workers for exactly: findings, evidence, recommendation.

Keep the routing record compact:

```text
worker | deliverable | risk | model:effort | validation
```

When Pi or the model catalog changes, recalibrate this policy against [`references/pi-thinking-efficiency-research.md`](references/pi-thinking-efficiency-research.md) and the current installed Pi docs/source, replacing stale rules in place.
