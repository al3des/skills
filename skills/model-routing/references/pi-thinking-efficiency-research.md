# Pi thinking modes and orchestration efficiency research

Scope: primary sources only: Pi's installed README, docs, and source from `@earendil-works/pi-coding-agent`. This report guides future revisions of the bundled `model-routing` skill. Source paths and model capabilities reflect the Pi installation inspected when the skill was authored and must be rechecked as Pi evolves.

## Documented facts

### Thinking level semantics and support

- Pi exposes thinking levels `off`, `minimal`, `low`, `medium`, `high`, `xhigh`, and `max` via CLI/settings/RPC; `xhigh` and `max` are only exposed when the selected model supports them. Sources: `README.md` CLI reference (`--thinking`) and examples; `docs/rpc.md#set_thinking_level`; `docs/models.md#thinking-level-map`.
- Default thinking level is `medium` in source (`DEFAULT_THINKING_LEVEL = "medium"`). Source: `dist/core/defaults.js:1`.
- Model configs mark whether a model supports reasoning with `reasoning: true/false`; per-model `thinkingLevelMap` maps Pi levels to provider values or marks unsupported levels with `null`. Omitted entries mean standard levels through `high` use provider default mapping; extended `xhigh`/`max` are unsupported unless explicitly mapped. Source: `docs/models.md#model-configuration` and `docs/models.md#thinking-level-map`.
- For Anthropic adaptive-thinking models, Pi maps `minimal`/`low` to provider effort `low`, `medium` to `medium`, `high` to `high`, and defaults other values to `high` unless the model has an explicit `thinkingLevelMap` string. Source: `dist/.../pi-ai/dist/api/anthropic-messages.js:586-604`.
- For Anthropic budget-based thinking, Pi enables `thinking: { type: "enabled", budget_tokens, display }`; when thinking is off and the model permits off, Pi sends `thinking: { type: "disabled" }`. Source: `dist/.../pi-ai/dist/api/anthropic-messages.js:747-773`.
- For OpenAI-compatible APIs, Pi clamps unsupported levels through `clampThinkingLevel`, then sends provider-specific fields: OpenAI-style `reasoning_effort`, OpenRouter `reasoning: { effort }`, Together `reasoning: { enabled }` plus optional `reasoning_effort`, Qwen `enable_thinking`, DeepSeek `thinking`, etc. Source: `docs/models.md#openai-compatibility`; `dist/.../pi-ai/dist/api/openai-completions.js:407-574`.
- In the installed `openai-codex` catalog, `gpt-5.5` maps Pi `minimal` to provider `low`, supports `xhigh`, and does not expose `max`. GPT-5.6 Luna, Terra, and Sol also map `minimal` to `low` and explicitly expose both `xhigh` and `max`. Consequently, `minimal` is not a lower-effort Codex setting than `low` for these four models. Source: `dist/.../pi-ai/dist/providers/openai-codex.models.js:61-140`.
- In the OpenAI Codex Responses adapter, Pi omits the request `reasoning` object when thinking is `off`; it does not send an explicit zero-reasoning effort. Therefore this source does not establish `off` as a lower-token Codex mode. Source: `dist/.../pi-ai/dist/api/openai-codex-responses.js:324-374`.

### Custom thinking budgets and output budget interaction

- `thinkingBudgets` can be set in settings as custom token budgets per thinking level. Source: `docs/settings.md#thinkingbudgets`.
- Source defaults for budget-based thinking are `minimal: 1024`, `low: 2048`, `medium: 8192`, `high: 16384`; custom budgets override these. `xhigh` and `max` are clamped to `high` for budget lookup. Source: `dist/.../pi-ai/dist/api/simple-options.js:33-51`.
- Pi reserves at least 1024 non-thinking output tokens by lowering the thinking budget if the computed `maxTokens` would otherwise be consumed by thinking. Source: `dist/.../pi-ai/dist/api/simple-options.js:43-50`.
- For Anthropic budget-based thinking, Pi adds thinking budget to requested output cap up to model max, then clamps to context; it finally caps `thinkingBudgetTokens` to `maxTokens - 1024`. Source: `dist/.../pi-ai/dist/api/anthropic-messages.js:620-626`.
- The OpenAI Codex Responses adapter receives the selected reasoning effort but does not consume `thinkingBudgets`; those numeric settings therefore do not cap reasoning tokens for this project's `openai-codex` models. Source: `dist/.../pi-ai/dist/api/openai-codex-responses.js:324-374`; compare the budget use in `dist/.../pi-ai/dist/api/anthropic-messages.js:620-626`.

### Context, compaction, and session reuse

- Pi auto-compacts when `contextTokens > contextWindow - reserveTokens`; default `reserveTokens` is 16,384 and default `keepRecentTokens` is 20,000. Sources: `docs/compaction.md#when-it-triggers`; `docs/settings.md#compaction`.
- Compaction keeps recent messages and summarizes older ones; it is lossy, but full history remains in the JSONL session and can be revisited through `/tree`. Sources: `README.md#compaction`; `docs/compaction.md#how-it-works`.
- Repeated compactions summarize from the previous kept boundary, preserving messages that survived prior compaction in the next summarization pass. Source: `docs/compaction.md#how-it-works`.
- Session files are trees. `/tree` keeps alternatives in one file; `/fork` starts a separate session from a prior user prompt; `/clone` duplicates the current active branch into a new session. Sources: `docs/sessions.md#tree-fork-and-clone`; `README.md#sessions`.
- `buildSessionContext()` extracts the active branch context, current model, and thinking level; `custom` entries do not participate in LLM context, while `custom_message`, compaction summaries, and branch summaries do. Source: `docs/session-format.md#context-building`.

### Cache behavior

- The interactive footer shows total token/cache usage (`↑`, `↓`, `R`, `W`), latest cache hit rate (`CH`), cost, context usage, and current model. Source: `README.md#interactive-mode`.
- `PI_CACHE_RETENTION=long` requests extended prompt cache: Anthropic 1h and OpenAI 24h. Source: `README.md#environment-variables`.
- Cache retention defaults to `short`; `PI_CACHE_RETENTION=long` switches to `long`; explicit `cacheRetention` can disable cache with `none`. Sources: `dist/.../pi-ai/dist/api/anthropic-messages.js:16-34`; `dist/.../pi-ai/dist/api/openai-completions.js:90-99`.
- Anthropic cache markers are placed on the system prompt, the last user message/conversation block, and the last tool definition when supported. Sources: `docs/models.md#anthropic-messages-compatibility`; `dist/.../pi-ai/dist/api/anthropic-messages.js:691-773,954-996`.
- OpenAI Chat Completions uses `prompt_cache_key` for direct OpenAI when cache retention is not `none`, and `prompt_cache_retention: "24h"` when retention is `long` and supported. Source: `dist/.../pi-ai/dist/api/openai-completions.js:449-460`.
- Pi can send session-affinity headers from the session id for compatible providers. Source: `docs/models.md#anthropic-messages-compatibility`; `docs/models.md#openai-compatibility`; `dist/.../pi-ai/dist/api/openai-completions.js:421-438`.
- Pi's cache-miss accounting treats compaction and branch summaries as legitimate context resets, but model switches are not exempt and can re-bill the prompt. It ignores misses at or below 1024 tokens as noise. Source: `dist/core/cache-stats.js:1-101`.

### Tools and context loading

- Default built-in tools are `read`, `write`, `edit`, and `bash`; CLI tool allow/deny controls include `--tools`, `--exclude-tools`, `--no-builtin-tools`, and `--no-tools`; built-in tool list also includes `grep`, `find`, and `ls`. Source: `README.md#tool-options`.
- Context files `AGENTS.md`/`CLAUDE.md` load from global, parent directories, and current directory unless `--no-context-files` is used. Source: `README.md#context-files`.
- Skills use progressive disclosure: Pi scans names/descriptions into the system prompt at startup; the full `SKILL.md` is loaded only when the task matches or `/skill:name` forces it. Source: `docs/skills.md#how-skills-work`.
- Skill discovery can be disabled with `--no-skills`; explicit `--skill` paths still load. Source: `docs/skills.md#locations`.

### Monitoring and orchestration control

- `/session` shows session file, ID, messages, tokens, and cost; RPC `get_session_stats` returns assistant usage totals and current context-window estimate used for compaction/footer display. Sources: `README.md#commands`; `docs/rpc.md#get_session_stats`.
- RPC emits `message_update` deltas including thinking/text/toolcall events, tool execution start/update/end, queue updates, compaction start/end, retry events, and `agent_settled` when no automatic retry/compaction retry/queued continuation remains. Source: `docs/rpc.md#events`.
- Steering messages are delivered after the current assistant turn's tool calls, before the next LLM call; follow-ups are delivered only when the agent has no more tool calls/steering messages. Delivery can be `one-at-a-time` or `all`. Sources: `README.md#message-queue`; `docs/rpc.md#steer`; `docs/rpc.md#set_steering_mode`.

## Recommendations for `model-routing/SKILL.md`

- Standardize `low` as the lowest explicit effort for these Codex models: their catalog maps `minimal` to `low`, while `off` merely omits an explicit reasoning setting. Use `medium` for ordinary multi-step work and `high` for ambiguous, difficult, or high-risk work.
- Treat `xhigh`/`max` as evidence-driven escalations after `high`. Verify support first: GPT-5.5 exposes `xhigh` but not `max`; the three GPT-5.6 variants expose both.
- Control Codex effort with `--thinking`; omit a numeric `thinkingBudgets` policy because the Codex Responses adapter does not use it.
- Prefer fresh sessions for unrelated tasks and reuse a session for tightly related iterative work where accumulated context and session-keyed prompt caching remain useful. Use `/clone` or `/fork` only when inherited context is still relevant to the variant.
- Compact deliberately when a cohesive session approaches its context limit; compaction is lossy and itself requires summarization. A fresh bounded worker is more efficient for unrelated work.
- Preserve project context rules while reducing optional load: narrow tool schemas with `--tools`, and use `--no-skills --skill <required-path>` when a worker needs one explicit skill rather than autonomous discovery of all skills.
- Use `/session`, the footer, or RPC `get_session_stats` on representative runs to compare input/output/cache use and recalibrate repeated routes.
- Prefer steering for urgent corrections before the next LLM call; prefer follow-up for independent next tasks. Batch only dependency-free instructions.
