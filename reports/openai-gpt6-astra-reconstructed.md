# OpenAI: "Rethinking skills and prompts for GPT-6 Astra" (reconstructed)

Source: https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra
Author: Eric Provencher (OpenAI Developer Relations). Published ~11 September 2026.

PROVENANCE WARNING: the original page and every mirror were blocked by this
container's network policy. This file is reconstructed from search-engine
snippets and press coverage (the-decoder, aiweekly, digitaltoday, note.com/npaka,
pasqualepillitteri, tenten, theneuron). Treat it as a SECONDARY source. Where you
cite it in the report, say "per press coverage of the OpenAI post". Do not invent
details beyond what is listed here.

## Thesis
- Coding-agent best practices are changing fast. With more capable models, what
  used to require a lot of handholding and scaffolding no longer does.
- Scaffolding written for older/weaker models (GPT-5.6 Sol, Luna) can now STUNT
  Astra's performance or bloat its context: it consumes context, triggers
  irrelevant skills, prompts redundant work, or causes unnecessary pauses.
- "Excessive defensive prompt scaffolding degrades agent performance rather than
  improving accuracy."

## Skills
- Every skill's name + description is injected into model context. Codex caps
  this list at ~2% of the context window (or 8,000 characters when the window is
  unknown). If descriptions are too long or there are too many skills, Codex
  forcibly SHORTENS descriptions, which makes routing less reliable: "harder for
  the model to judge which skill to choose."
- Keep descriptions as short as possible while making clear WHEN to use the
  skill. Prune descriptions to primary function + specific triggers. Example: a
  Postgres migration skill should fire only when creating/modifying a migration
  or checking its rollout, not on anything Postgres-related.
- Make triggers specific so the model does not misfire on the wrong instructions.
- Treat skills as ROUTERS, not textbooks. Use progressive disclosure for complex
  workflows: load guidance only when it is actually relevant instead of dumping
  everything into context.

## AGENTS.md / project instructions
- Should work like a TABLE OF CONTENTS: separate permanent rules from contextual
  guidance, and point to the latter rather than inlining it.

## Prompts and rules
- Clearly define what "done" looks like BEFORE a task starts. Control comes from
  defining done and the boundaries, not from piling up prohibitions that the
  model then interprets rigidly.
- Replace rigid micro-step instructions ("Do A first, then B, then C") with
  crisp definitions of done. Detailed step lists were needed to stabilise weaker
  models; for capable models they limit judgement.
- The fix is NOT more guardrails: remove the old ones and rewrite the few that
  remain so they point a direction instead of listing prohibitions.
- "Fewer rules means giving better instructions. Removing redundant rules frees
  up attention for the ones that actually matter."

## Three recommended shifts (as summarised by coverage)
1. Prune skill descriptions to primary functions and triggers.
2. Implement progressive disclosure for complex workflows.
3. Replace rigid micro-step instructions with crisp definitions of done.
