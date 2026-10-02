# Skill Wars: evaluation brief (shared by both analyst agents)

## Goal (refined from the user's request)
The user is a developer choosing between, or learning from, two public skill
libraries for coding agents:

- https://github.com/mattpocock/skills  (local clone: repos/mattpocock-skills)
- https://github.com/obra/superpowers    (local clone: repos/obra-superpowers)

They want a CONCISE, evidence-based comparison of the two: differences,
similarities, and what each is actually capable of. They have four specific
worries, and want each answered with evidence (file:line quotes, counts):

1. LENGTH: are the skills too long for their job? Do they blow the context
   budget, or bury the instruction that matters?
2. "DUMBING DOWN": do they treat the model as a junior who cannot be trusted
   (micro-step scripts, rationalisation tables, "you cannot think your way out
   of this"), in a way that current guidance says degrades capable models?
3. OUTDATED TECHNIQUES: ALL-CAPS commands, ALWAYS/NEVER walls, repeated
   "IMPORTANT", prohibition lists, mandatory ceremony that was built for weaker
   models and may now be counterproductive.
4. BUREAUCRACY / OVER-CONSTRAINT: mandatory process regardless of task size,
   gates and approvals that add friction without adding safety, skills that
   reduce the model's degrees of freedom on judgement tasks.

Evaluate BOTH ways: (a) within the repo, skill vs skill (which skills are the
good citizens, which are the hotspots), and (b) later, the two repos against each
other (the orchestrator does the cross-comparison from your reports; make your
report precise enough to support it).

## Reference sources (read all three before judging)
- refs/claude-code-best-practices.md   (Anthropic, Claude Code best practices)
- refs/agent-skills-best-practices.md  (Anthropic, Skill authoring best practices)
- refs/openai-gpt6-astra-reconstructed.md (OpenAI post, RECONSTRUCTED from press
  coverage because the origin was blocked; cite as secondary)

## Rubric (apply to every skill, score 1-5 where 5 = fully aligned with guidance)
A. Concise / context cost. Body under 500 lines; assumes the model is smart;
   no explanation of things the model already knows; every paragraph earns its
   tokens. (Anthropic skills doc "Concise is key", "Token budgets"; OpenAI
   "skills as routers not textbooks".)
B. Description quality. Short, third person, states WHAT and WHEN, specific
   triggers, does not claim everything. Count characters. (Anthropic "Writing
   effective descriptions"; OpenAI 2%/8000-char budget and forced truncation.)
C. Degrees of freedom. Specificity matches fragility: judgement tasks get
   heuristics, fragile sequences get exact steps. Flag rigid micro-steps on
   judgement tasks. (Anthropic "Set appropriate degrees of freedom"; OpenAI
   "replace micro-steps with definitions of done".)
D. Progressive disclosure. SKILL.md routes to reference files; references one
   level deep; long references have a table of contents; nothing loaded that
   the task does not need. (Anthropic patterns 1-3, "Avoid deeply nested
   references"; OpenAI "load guidance only when relevant".)
E. Definition of done / verification. Says what done looks like, gives the
   model a check it can run, demands evidence over claims. (Claude Code doc
   "Give Claude a way to verify its work"; OpenAI "define done before starting".)
F. Scaffolding style. ALL-CAPS, ALWAYS/NEVER, IMPORTANT, prohibition lists,
   rationalisation/"red flag" tables, threats. Count them. Anthropic: emphasis
   on ONE line works, on many lines none stands out; a bloated instruction file
   gets ignored. OpenAI: fewer rules, point a direction instead of prohibiting.
G. Proportionality. Does the skill scale down for small tasks? (Claude Code
   doc: "If you could describe the diff in one sentence, skip the plan"; plan
   mode "adds overhead".) Mandatory ceremony on trivial tasks is bureaucracy.
H. Evaluation discipline. Are there evals/tests for skill BEHAVIOUR (not just
   plugin plumbing)? Tested across models? (Anthropic "Build evaluations first",
   "Test with all models".)
I. Consistency. Terminology, naming (gerund vs imperative), structure, voice
   consistent across the collection.
J. Portability and dependencies. Which harnesses, what the skill assumes about
   tools, hooks, subagents, issue trackers.

## What your report must contain (write it to reports/<repo>.md, in English)
1. Facts: commit hash and date, skill count by category, total words in
   SKILL.md files, total words in reference files, hooks, plugin manifests,
   harness support, how skills are triggered (bootstrap hook? description
   only? user-invoked only?), eval/test setup.
2. Inventory table, one row per SKILL.md: name | invocation (user / model /
   both) | body lines | body words | description chars | reference files
   (count + max nesting depth) | ALLCAPS-word count | ALWAYS/NEVER/MUST count
   | your A-J scores. Produce the numbers with scripts (wc, grep, awk), not by
   eye. Include the script(s) you used in an appendix so the orchestrator can
   re-run them.
3. Architecture and philosophy in the maintainer's own words (quote README,
   AGENTS.md/CLAUDE.md, any "writing skills" guidance), then your reading of
   how the pieces fit: entry points, flow for "build feature X", flow for "fix
   typo", flow for "fix this bug", subagent usage, what the user must set up.
4. Hotspots: the 5 worst passages for each of the user's four worries, each
   with file:line and a short quote, and why it conflicts with which guidance.
5. Good citizens: the 5 best passages/skills and why.
6. Capability map: what a developer actually gets from this library (what it
   automates, what it enforces, what it leaves to the model/user).
7. Verdict on each of the four worries, for this repo, in 3-5 sentences each,
   with the evidence that supports it. Be fair: where the maintainer argues
   against the guidance (e.g. superpowers AGENTS.md says its philosophy
   deliberately differs from Anthropic's and is eval-tuned), state that
   argument and assess whether the repo backs it with evidence.
8. Open questions you could not settle from the files.

Style: concise, concrete, no marketing language, no hedging padding. Prefer
tables and short bullets. Quote at most ~25 words per quote. Every claim about
a file carries a path and line number.

## Process rules
- Read every SKILL.md fully and every reference file it links. Skim release
  notes/changelog only for philosophy shifts. Ignore node_modules/.git.
- Do not modify the cloned repos.
- Do not browse the web; everything you need is on disk. If you believe a
  claim needs a source you do not have, say so in "Open questions".
- When done, reply with a 15-line summary and the path of the report. You
  will then receive follow-up questions from the orchestrator to check your
  understanding; keep your context, you will need it.
