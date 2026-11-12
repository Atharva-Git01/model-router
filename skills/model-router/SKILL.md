---
name: model-router
description: Routes each task to the cheapest Claude model that can do it well - Haiku for small mechanical work, Sonnet for ordinary work, Opus only for hard problems - by handing the work to a subagent through the Agent tool's `model` parameter. The short rule in ~/.claude/CLAUDE.md applies to every task; load this skill for the full rubric whenever a task's tier is unclear, a request mixes easy and hard parts, you are about to write a subagent brief, a handed-off task failed or came back harder than expected, or the user types /model-router, asks which model should do something, asks how routing works, or questions why a model was chosen.
argument-hint: "[task to route]"
---

# Model Router

The user wants every task done by the cheapest model that can do it well: Haiku for small jobs, Sonnet for ordinary work, Opus only when the task is hard. The main session is the router. It decides the tier, hands the work to a subagent running on that model, checks what comes back, and reports. The always-on summary of this policy is the "Model routing" section of `~/.claude/CLAUDE.md`; this file is the full version, with the reasoning.

If you are a subagent (you were started by the Agent tool), none of this applies to you. Do the task you were given yourself.

## The three tiers

| Tier | `model` | What belongs here |
|---|---|---|
| Small | `haiku` | Mechanical and fully specified. You could write the exact steps in the brief, no design decisions are involved, and the result is easy to check. |
| Medium | `sonnet` | Needs judgment but follows known patterns. The path is clear even if the details are not. This is the default tier. |
| Hard | `opus` | Ambiguous, novel, cross-cutting, or expensive to get wrong. Someone has to work out what to do before doing it. |

Size is not difficulty. A rename across 500 files is small: it is mechanical and a search proves it worked. A three-line fix to a race condition is hard.

**Signs a task is small**

- The user said exactly what to change and where.
- It is one operation repeated many times.
- It is read-only: find, list, count, look up, summarise, extract, convert.
- It is run-and-report: run a command, a test suite or a script and say what happened.
- A simple check (a search, a test, a diff) proves the result.

**Signs a task is hard**

- A serious attempt already failed (at Sonnet level, or by the user themselves), or the problem keeps coming back.
- The cause is unknown, the symptom is intermittent, or it spans several systems.
- It touches authentication, permissions, payments, personal data, cryptography, live-data migrations, or anything that cannot be undone.
- The requirements are vague and the task includes deciding what to build.
- It cuts across many modules with no existing pattern to copy. The plan is the hard part; the building usually is not.
- It turns on concurrency, performance, or subtle correctness.
- The user will act on the answer and a wrong one is costly: legal, financial, medical, or a strategic decision.

Everything else is medium: a feature in an existing codebase, an ordinary bug with a clear reproduction, tests, a refactor inside one module, a component or page, a new document or report, a normal code review, a data analysis, writing in the user's voice. Summarising existing material is small; writing something new is medium.

## Deciding

Work down this list and stop at the first line that fits.

1. The user named a model ("use opus for this") or said to do it yourself: do that.
2. The user invoked a skill by slash command, or the task's skill needs the user's answers along the way (questions, approval gates): run that skill in the main session as written, and hand off its production steps by tier where the skill leaves open who does them.
3. There is nothing to hand off: conversation, a clarifying question, a short answer or explanation, or something one quick tool call settles. Answer directly.
4. Any sign of hard: Opus. On a big job, Opus takes the planning and the hard parts only; see "Requests with mixed parts".
5. Clearly small: Haiku.
6. Otherwise: Sonnet.

When torn between small and medium, choose Sonnet, because a Haiku mistake on a judgment call costs a second run. When torn between medium and hard, start with Sonnet and move up if it struggles, because the user wants Opus kept for tasks that are clearly hard.

Decide from the request and what is already in the conversation. If that is not enough, take one quick look (a file listing, a single search) and no more. Reading the codebase in order to classify the task is doing the task on the expensive model; the exploring belongs to whichever model gets the work.

## Handing off

Call the Agent tool with `model` set to `haiku`, `sonnet` or `opus` and a self-contained brief. A few things make this work well:

- **Start a fresh agent, not a fork.** A fork always runs on the session's own model and ignores `model`, so it cannot route anything.
- **Skip the handoff when you are already the right model.** If the tier's model is the one this session runs on, do the work yourself. A same-model subagent only adds a cold start. If the session runs on a model outside the three tiers (Fable, for example), every tier is a handoff.
- **Run independent parts in parallel** by making several Agent calls in one message.
- **Send follow-ups to the same agent** with SendMessage while it is still relevant, so it keeps its context. Start a new agent only when the tier changes.
- **Settle open questions first.** A subagent cannot talk to the user. Ask the user yourself before handing off, or tell the subagent to come back with its questions rather than guess.
- **Name the skill in the brief** when the task calls for one that can run without the user (for example, CLAUDE.md sends UI component work to the ux-ui-kit skills). The subagent can load skills itself; loading a task skill in the main session only to hand the work off pays for it twice.
- **For read-only searching**, the Explore agent type skips CLAUDE.md and starts lighter than a general-purpose agent. Pass `model` to it the same way.

If this session cannot start subagents, do the work yourself. When the task sits above the session's model, say so in one line so the user can switch models if they want.

### Why the rules are shaped this way

Two costs sit behind every handoff, and they explain most of the rules above.

A subagent starts cold. Before it does anything it loads the system prompt, the tool list, CLAUDE.md and the skill list, which came to about 70,000 tokens when measured on this machine in November 2026. On Haiku that is negligible. On Sonnet or Opus it is the price of reading a small codebase. So a Haiku handoff is worth it for almost any real tool work, while a Sonnet or Opus handoff should carry a task big enough to be worth the start-up.

A subagent's reply is not shown to the user. Whatever it says, the main session has to say again. Handing off pays when the result lands in files or actions, and saves nothing when the result is a paragraph of chat. For a long written deliverable, have the subagent write it to a file and relay a short summary with the path.

For scale, API list prices in November 2026 per million input tokens were roughly $0.10 for Haiku, $2 for Sonnet and $4 for Opus. Prices change, so treat the ratios as a guide: Haiku is far cheaper than the other two, and Opus costs about twice what Sonnet does.

## Writing the brief

The subagent knows nothing about this conversation, so the brief is everything it has. Cover these points, in as few words as the task allows:

```
Goal: what to produce, and what it is for.
Context: decisions already made, constraints, and preferences it cannot discover on its own.
Where: absolute paths, names, commands.
Steps: see the tier notes below.
Done when: a check it can run or observe.
Report back: what changed and where, the result of the check, and anything left unverified. No file dumps.
If this needs decisions or files beyond the above, stop and say why instead of guessing.
```

Fit the detail to the tier:

- **Haiku** follows instructions well and should not be asked to decide. Give exact steps and an exact check, one job per brief.
- **Sonnet** wants the goal, the constraints and the existing pattern to follow. Leave the details to it.
- **Opus** wants the problem, everything known so far (symptoms, what was tried, constraints) and what a good answer looks like. Ask for its reasoning and the trade-offs it weighed. On a large job, ask for a plan the cheaper tiers can carry out.

The last line of the template matters most for the lower tiers. It turns a misjudged task into a clean signal to move up, where otherwise you would get a confident wrong answer.

## Requests with mixed parts

A big job is usually a mix even when the request is one sentence, such as "migrate the whole API". Sending all of it to Opus spends the expensive model on work Sonnet and Haiku do as well. Split the request and route each part. Design comes before building, and independent parts run in parallel.

Example: "Add subscription billing to the app."

1. Opus designs the data model, the webhook handling and the failure cases, and writes the plan to a file.
2. Sonnet implements the plan, one agent per independent module.
3. Haiku does the mechanical follow-through: renames, import updates, boilerplate, doc tables.

Do not split a task whose parts share so much context that each handoff would have to re-explain all of it. One coherent medium task stays one Sonnet task.

## When the work comes back

A subagent's report is a claim, so check it in proportion to the risk: read the diff, run the test, open the file.

- If it failed, or stopped to say the task is harder than briefed, move up one tier and pass along what was tried and what went wrong.
- If the brief was the problem (missing context, a wrong path), fix the brief and retry once on the same tier.
- Do not rerun the same tier with the same brief.
- If Opus cannot do it either, stop and tell the user what is blocking.

## What the user sees

Before each handoff, one line: `Routing: small → Haiku (rename across 3 files)`. For a split request, one line that lists the parts. No routing line when you answer directly. If a task was moved up a tier, say so in the final report.

## Overrides

- "Use opus / sonnet / haiku for this" sets the tier for that task.
- "Do it yourself" or "don't delegate" means no handoff.
- If the user corrects a route ("that should have gone to Sonnet"), follow the correction for the rest of the session. If it sounds like a standing preference ("always use Opus for anything in the payments folder"), offer to add it to the Model routing section of `~/.claude/CLAUDE.md`.

## Examples

| Request | Route | Why |
|---|---|---|
| "Rename `getUsr` to `getUser` everywhere in src/" | Haiku | One operation repeated; a search proves it |
| "Find where the upload size limit is set" | Haiku, Explore agent | Read-only lookup |
| "Run the tests and tell me what fails" | Haiku | Run and report |
| "Turn this CSV into JSON grouped by country" | Haiku | Fully specified conversion |
| "Add a dark-mode toggle to the settings page, with tests" | Sonnet | Ordinary feature that follows existing patterns |
| "Fix the failing date-parsing test" | Sonnet | Ordinary bug with a clear reproduction |
| "Build a pricing table component for the app" | Sonnet, brief names the design-code skill | Standard build; the skill carries the method |
| "Write this week's five reel captions" | Sonnet | Writing in the user's voice needs judgment |
| "Checkout double-charges some customers under load. Find out why and fix it" | Opus | Money, intermittent, cause unknown |
| "Design the database and sync strategy for offline mode" | Opus | Architecture with long-lived consequences |
| "That fix didn't work, it still crashes" | One tier above whoever tried; Opus if Sonnet made the attempt | A failed attempt moves the task up |
| "Migrate the whole API from REST to GraphQL" | Opus plans, Sonnet builds per module, Haiku does the mechanical edits | Mixed parts |
| "What's the difference between `let` and `const`?" | Answer directly | Nothing to hand off |

## When invoked as /model-router

- With a task after the command: route that task as described above and carry it out.
- With nothing after it: show the tier table, say which model this session is running on and which tier it therefore handles itself, and list the override phrases.
- With a question about a past route: explain which signs decided it.
