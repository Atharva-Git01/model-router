# model-router

A Claude Code skill that sends each task to the cheapest Claude model that can do it well.

| Task | Model | Typical work |
|---|---|---|
| Small | Haiku | Renames, formatting, lookups, summaries, data conversion, run-and-report |
| Medium | Sonnet | Features, ordinary bug fixes, tests, components, new documents, code reviews |
| Hard | Opus | Architecture, unknown-cause bugs, auth/payments/personal data, performance, vague requirements |

The main session decides the tier, then hands the work to a subagent through the Agent tool's `model` parameter. Big jobs are split (Opus plans, Sonnet builds, Haiku does the mechanical follow-through). Quick answers and conversation are handled directly, with no handoff.

## Install

1. Copy the skill folder into your personal skills directory:

   ```bash
   cp -r skills/model-router ~/.claude/skills/
   ```

2. Append the contents of `CLAUDE-snippet.md` to `~/.claude/CLAUDE.md`. This short standing rule is what makes Claude apply the routing to every task. The skill holds the full rubric, brief template and examples.

Start a new session afterwards. Type `/model-router` to see the tier table.

## Overrides

- "Use opus / sonnet / haiku for this" sets the model for that task.
- "Do it yourself" means no handoff.

## Notes

- CLAUDE.md is an instruction, not a lock. Claude follows it consistently but nothing forces it.
- Each subagent starts cold and loads your CLAUDE.md and skill list first (about 70,000 tokens on one setup). That is negligible on Haiku, so the saving is largest when the session itself runs on Sonnet.
- Written for Claude Code with the Agent tool and the `haiku`, `sonnet` and `opus` model aliases.
