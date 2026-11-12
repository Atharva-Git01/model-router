# Model routing (model-router)
Standing instruction for every task in every project: give each task to the cheapest model that can do it well. The user explicitly asks for subagents to be used for this, so do it without asking each time: pick the tier, then call the Agent tool (a fresh agent, not a fork) with `model` set to that tier and a self-contained brief. This section is for the main session only. If you were started by the Agent tool, you are the one doing the work: do the task yourself and ignore this section.

| Task | `model` | What belongs here |
|---|---|---|
| Small | `haiku` | Mechanical, fully specified, easy to check: renames, formatting, typo fixes, find / list / look up, summarise or extract from existing material, convert data, boilerplate from a clear pattern, run a command and report. Size is not difficulty: a 500-file rename is small. |
| Medium | `sonnet` | Needs judgment but follows known patterns: a feature in an existing codebase, an ordinary bug with a clear reproduction, tests, a refactor inside one module, a component or page, writing a new document or report, a normal code review, data analysis. The default tier. |
| Hard | `opus` | Only when clearly needed: architecture or system design, planning a change that cuts across many modules, a bug whose cause is unknown or that survived a serious attempt, anything touching auth, payments, personal data or irreversible operations, concurrency or performance problems, vague requirements that need a plan first. |

- Unsure between small and medium: Sonnet. Unsure between medium and hard: Sonnet first, then Opus if it struggles.
- If the tier's model is the one this session already runs on, do the work yourself; a same-model subagent only adds cost. If this session runs on a model outside the table (Fable, for example), hand off all three tiers.
- Answer directly, with no handoff: conversation, clarifying questions, short answers and explanations, and anything one quick tool call settles. A subagent's reply has to be relayed anyway, so nothing is saved.
- Decide from the request and what is already in context. Do not explore first in order to classify; exploring belongs to the model that gets the work.
- A big job is usually a mix, even when the request is one sentence ("migrate the whole API"), so split it: Opus plans and takes only the hard parts, Sonnet builds from the plan, Haiku does the mechanical follow-through. Run independent parts in parallel.
- Say the route in one line before handing off, for example `Routing: small → Haiku (rename across 3 files)`.
- Check what comes back before reporting it. If the subagent failed or says the task is harder than briefed, move up one tier and pass along what was tried.
- The user's words win: "use opus / sonnet / haiku for this" or "do it yourself" overrides the table.
- Skills still apply. A skill the user invokes by slash command, or one that needs the user's answers along the way, runs in the main session as written. Otherwise name the needed skill in the brief so the model doing the work loads it.
- Full rubric, brief template and examples: the **model-router** skill (`~/.claude/skills/model-router/SKILL.md`). Load it when the tier is unclear, before splitting a mixed request, after a failed handoff, or when the user types `/model-router`.
