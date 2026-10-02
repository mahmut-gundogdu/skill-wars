# Skill Wars: mattpocock/skills vs obra/superpowers

A developer-facing comparison of two popular skill libraries for coding agents, judged against the 2026 guidance on writing skills and prompts from Anthropic and OpenAI. Turkish version: [README.tr.md](README.tr.md).

| | [mattpocock/skills](https://github.com/mattpocock/skills) | [obra/superpowers](https://github.com/obra/superpowers) |
|---|---|---|
| Analysed commit | `d81f3a1` (2026-09-29, v1.2.3) | `8ca22db` (2026-09-25, v6.4.2) |
| What it is | A toolbox of 37 small skills (27 shipped) you mostly call by hand | A mandatory, end-to-end development methodology of 15 skills |
| How skills fire | Description only. 16 of 27 shipped skills fire only when typed | A hook injects a bootstrap every session that orders the model to use skills |
| Always-on context cost | 2,150 chars of descriptions | ~900 tokens of bootstrap plus 2,689 chars of descriptions, re-injected after every `/clear` and compaction |
| SKILL.md size | median 534 words, max 160 lines, none over 500 lines | median 1,246 words, two skills over 500 lines (564 and 677) |
| Reference files | 23 files, 11.4k words | 45 files, 27.8k words |
| Shouting (ALL-CAPS emphasis words) | 8 | 153 |
| Rationalisation ("excuse → reality") rows | 0 | 98, in 12 of 15 skills |
| Human approval gates in a feature flow | 2 to 4, all typed by you | at least 7 (19 distinct gates exist) |
| Behaviour evals in repo | none | partial: in-repo LLM tests plus an external eval repo (not inspected) |
| Harnesses | Claude Code plugin, anything else via `npx skills` | 16 listed harnesses with native plugins |
| Setup | run `/setup-matt-pocock-skills` once per repo, configure an issue tracker | install and go |

## Verdict in one paragraph

The two libraries encode the same engineering canon (interview before building, spec, red-green TDD, root-cause debugging, subagent review, worktrees) but with opposite theories of the model. **superpowers** treats the model as something to be disciplined: a session-start bootstrap, four ALL-CAPS "Iron Laws", 98 rows of pre-written rebuttals to the model's own excuses, and approval gates at every design step. This is exactly the style the Anthropic and OpenAI guidance now warns against, and the repo says so itself: its contributor guide states that its philosophy "differs from Anthropic's published guidance". **mattpocock/skills** treats the model as capable and keeps the human as the index: skills are short, emphasis is carried by bold words rather than caps, and most skills cost nothing until you type them. Its weakness is proportionality: `tdd`, `diagnosing-bugs` and the interview skills impose fixed ceremony that does not scale down for small tasks, and the repo has no evals at all. Neither library proves it "works with any model". If you want an opinionated autopilot and accept the tax, pick superpowers. If you want composable tools under your own control, pick mattpocock/skills and make its two auto-firing discipline skills user-invoked.

## Method

- Each repo was cloned and analysed by a separate Opus 5.5 agent against a shared brief and rubric. Every number was produced by a script, and every claim carries a `path:line`. The orchestrator spot-checked 28 citations across both reports against the files (all matched), then put seven verification questions to each agent on flows, gates, evidence and the strongest case for the other side; every answer was consistent with the files. The full reports are in [reports/](reports/).
- Yardsticks: Anthropic's [Claude Code best practices](https://code.claude.com/docs/en/best-practices), Anthropic's [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices), and OpenAI's [Rethinking skills and prompts for GPT-6 Astra](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra). The OpenAI post was blocked by the analysis environment's network policy, so its recommendations were reconstructed from search results and press coverage ([reports/openai-gpt6-astra-reconstructed.md](reports/openai-gpt6-astra-reconstructed.md)) and are cited as a secondary source.
- The guidance distilled to ten criteria: concise body, specific description, degrees of freedom matched to fragility, progressive disclosure, a definition of done, restrained emphasis, proportional ceremony, evals, consistency, portability. Scores are the analysts' judgement on a 1 to 5 scale and are only roughly comparable across the two repos.

## What each library is

**mattpocock/skills.** A collection organised in buckets (engineering, productivity, misc, in-progress). The one design axis is who may invoke a skill: user-invoked skills orchestrate (`/grill-me`, `/to-spec`, `/to-tickets`, `/implement`, `/triage`, `/wayfinder`) and are unreachable by the model; model-invoked skills hold reusable discipline (`grilling`, `tdd`, `code-review`, `diagnosing-bugs`, `codebase-design`, `pr`). Nothing chains them; you type the next step. A router skill, `ask-matt`, recommends a path and stops. A setup skill writes a per-repo issue-tracker config and glossary layout that the engineering skills read. The maintainer's own `writing-for-agents` skill reads like an independent derivation of the Anthropic and OpenAI advice: hunt no-ops, disclose by branch, end every step on a completion criterion, prompt the positive.

**obra/superpowers.** A methodology. A SessionStart hook injects the full `using-superpowers` skill, wrapped in `<EXTREMELY_IMPORTANT>`, which tells the model that with "even a 1% chance a skill might apply" it "ABSOLUTELY MUST invoke the skill", before any response, including clarifying questions. The flow is brainstorming (design in approved sections, written spec), writing-plans, a worktree, then either subagent-driven development (an implementer and a reviewer subagent per task, up to five fix rounds, a final whole-branch review) or inline execution, with TDD, verification-before-completion and a merge menu at the end. Nothing is enforced by a hook; all enforcement is prose. The project ships a `writing-skills` skill that teaches authors to baseline-test a skill, then "bulletproof" it with rationalisation tables and red-flag lists.

## Similarities

- Same canon: interview first, spec, TDD with red before green, root cause before fix, review by a fresh subagent, worktrees for parallel work, YAGNI and deep modules.
- Both ship a meta-skill on writing skills, which the Anthropic skills doc says is unnecessary.
- Both document their own over-triggering problems: superpowers has no negative-trigger tests, and mattpocock's docs record `diagnosing-bugs` firing on plain problem descriptions.
- Both have drift between README and skills: superpowers' README still describes the plan reader as a "junior engineer with poor taste" that v6.4.2 removed from the skill, and still advertises "2-5 minute tasks" and a two-stage review; mattpocock's router still routes to a hand-off that was deleted, and its `tdd` description promises "red-green-refactor" while the body says refactoring is not part of the loop.
- Both load roughly 25k words of SKILL.md text in total; the difference is how much of it reaches a given session.
- Neither has evidence that its skills work across models. mattpocock's docs record weaker models breaking the grilling gate and GPT-5.6-Sol over-firing diagnosis; superpowers' own notes record the Codex bootstrap being removed because it "made the UX worse".

## Differences

| Axis | mattpocock/skills | obra/superpowers |
|---|---|---|
| Theory of the model | Capable; "forcing the point harder restricts the agent's creativity for little gain" | Must be disciplined; "YOU DO NOT HAVE A CHOICE. YOU MUST USE IT." |
| Enforcement | Advice, human-sequenced, no hooks | Bootstrap hook plus "Mandatory workflows, not suggestions" |
| Emphasis style | Bold leading words; 0 `IMPORTANT`, 0 ALL-CAPS `ALWAYS`/`NEVER`/`MUST` | 4 Iron Laws, 39 red-flag bullets, 98 rebuttal rows, XML-style `<HARD-GATE>` tags |
| Small-task path | No skill fires on a typo; cost is 2,150 chars | No exit; the red-flag table answers "the skill is overkill" with "Use it" |
| Ceremony on a new feature | Interview until "the frontier is empty" (46 questions is "ordinary"), spec, tickets | Intent, questions one per message, design sections approved one by one, spec gate, plan gate, worktree consent, merge menu |
| Execution | `/implement` per ticket with `/clear` between, or `implement-spec` with parallel subagents in worktrees | Deliberately low friction once the plan is approved: only four stop classes, rulings logged |
| Verification | Strong in `diagnosing-bugs` and `wizard`; weak elsewhere (criterion E mean 3.6) | Strong throughout: evidence tables, "no completion claims without fresh verification" (E mean 3.8) |
| Evals | None; "the check is a manual run" | Real eval craft (controls, 5 to 25 runs per arm) for recent changes; the frozen content has one supporting datum |
| Setup burden | Setup skill, tracker auth (`gh`/`glab`), labels created by hand | None beyond install |
| Description rule | Model-invoked: what plus when; user-invoked: a human one-liner kept out of model context | "Describes ONLY when to use (NOT what it does)", which conflicts with Anthropic's "what and when"; 5 of its own 15 descriptions break the rule anyway |
| Stance on Anthropic guidance | Converges independently | Explicit, deliberate divergence, PRs that "comply" are rejected |

## The four worries

### 1. Too long?

**mattpocock: mostly no.** No body exceeds 160 lines, a planning session loads about 4k tokens and a per-ticket session about 3k. The hotspots are local: `wayfinder` is 1,954 words in one file with its key rule ("never resolve more than one ticket per session") at line 105, `ask-matt` restates every skill at 1,895 words, and four reference files over 100 lines lack a table of contents.

**superpowers: yes, per workflow.** The bootstrap alone is 520 words against the repo's own 150-word target, paid on every start, clear and compaction. A feature flow loads about 12,000 words of skill text into the controller; two skills break the 500-line guidance limit; 13 of 15 skills exceed the repo's own 500-word target; brainstorming encodes the same flow three times (checklist, graph, prose); 15 long reference files have no table of contents, including a 1,150-line vendored copy of Anthropic's guide.

### 2. Does it dumb the model down?

**mattpocock: low.** There are no rationalisation tables, red-flag lists or threats anywhere. The residue is exhortation ("Be aggressive. Be creative. Refuse to give up." followed by a list of standard repro techniques) and one blanket "Never trust your parametric knowledge" in `teach`.

**superpowers: yes, in the older layer.** `test-driven-development` says of code written before a test: "Delete it. Start over. Don't look at it." `using-superpowers` polices the model's thoughts: "I need more context first" is listed as a rationalisation. `verification-before-completion` explains that "Exhaustion ≠ excuse". The `writing-skills` skill teaches authors to apply persuasion research (a study on getting models to comply with objectionable requests) and counts a model arguing that a skill is wrong as a test failure. The newer layer is the opposite: v6.4.2 `writing-plans` describes a capable reader, and the execution skills ask the model to make and log its own rulings.

### 3. Outdated techniques?

**mattpocock: low on shouting, moderate on prohibitions.** 8 caps tokens in 24k words. But 110 "don't/never/avoid" tokens, banned-word lists and a six-bullet "Don't" list contradict the maintainer's own rule that "steering by prohibition drags the forbidden behaviour into context".

**superpowers: heavily.** Double-wrapped `EXTREMELY-IMPORTANT` blocks with ALL-CAPS sentences in the always-on text, the "1% rule", four Iron Laws, 98 rebuttal rows and a creation log that records "NEVER fix symptom" being repeated four times on purpose. The maintainers defend this as eval-tuned. The repo backs that in one place: deleting TDD's rebuttal prose dropped test-first compliance from 8/10 to 5/10 on Claude and Codex. Nothing in the repo tests the bootstrap's caps, the 1% rule or the "human partner" phrasing, and the repo's own data twice found guidance unnecessary for current models.

### 4. Bureaucracy and over-constraint?

**mattpocock: the main weakness.** `tdd` will not write a test at a seam the human has not confirmed, has no "user is away" escape, and is also run inside `implement-spec`'s background subagents where no human can confirm anything; `diagnosing-bugs` runs six gated phases and its broad trigger ("something broken/throwing/failing/slow") makes it fire on ordinary bug reports; grilling has no question cap and the README says to use it for every change; `to-spec` demands "extremely extensive" user stories. The maintainer documents each complaint (issues #746, #607, #578) without a fix. What limits the damage: 16 of 27 shipped skills never fire unless typed, there is no bootstrap, and the router reserves spec and tickets for multi-session work.

**superpowers: high at the front, low at the back.** A React todo list is classified as "architectural" and needs at least seven round trips before and after code. Even a one-file bug fix is "bounded", and a bounded task's approval "is as hard a gate as an architectural one". "When in doubt between two paths, take the heavier one. Nothing downgrades mid-task." Opting out requires the human to say so explicitly. There is no "one-sentence diff, skip the plan" exit, which the Claude Code guide recommends. Once a plan is approved, execution is deliberately uninterrupted, which matches the guidance well.

## Scorecard (1 = ignores guidance, 5 = follows it)

| Criterion | mattpocock (27 shipped) | superpowers (15) |
|---|---|---|
| A. Concise body | 3.9 | 2.5 |
| B. Description quality | 3.8 | 3.4 |
| C. Degrees of freedom | 4.2 | 2.7 |
| D. Progressive disclosure | 4.2 | 3.7 |
| E. Definition of done | 3.6 | 3.8 |
| F. Emphasis and scaffolding | 4.3 | 2.7 |
| G. Proportionality | 3.7 | 2.4 |
| H. Evaluation discipline | 1.6 | 2.7 |
| I. Consistency | 3.9 | 3.3 |
| J. Portability | 3.5 | 3.8 |

Best skills: mattpocock `wizard`, `to-questionnaire`, `grilling`, `domain-modeling`; superpowers `diagnosing-superpowers`, `finishing-a-development-branch`, `using-git-worktrees`, and the v6.4.2 `writing-plans`. Worst: mattpocock `wayfinder`, `diagnosing-bugs`, `teach`; superpowers `using-superpowers`, `brainstorming`, `systematic-debugging`, `writing-skills`.

## What a developer actually gets

| | mattpocock/skills | obra/superpowers |
|---|---|---|
| Artifacts | Specs and dependency-ordered tickets on GitHub, GitLab or local files; triage state machine; architecture HTML report; PR bodies; bash setup wizards; cited research notes; glossary and ADRs | Design specs and plans under `docs/superpowers/`; per-task review ledgers; a browser "visual companion" for mockups; session forensics with a scrubbed bug bundle |
| Orchestration | Parallel implementers in worktrees (`implement-spec`); two-axis review in parallel subagents | Implementer plus reviewer subagent per task with model tiering and a five-round circuit breaker; final whole-branch review; seven-analyst diagnosis |
| Enforced deterministically | Nothing in the plugin (opt-in git-guardrail and pre-commit installers exist) | Nothing; prose only |
| Left to you | Choosing and sequencing skills, closing tickets, verifying review findings, deciding whether TDD or diagnosis is worth it | Approving each design section, spec, plan and merge; everything else runs |
| Known gaps | `code-review` subagents can re-invoke the skill (one report reached 50+ agents); `implement-spec` has no path for a failed implementer or a merge conflict | Stale README and recall tests; `requesting-code-review` uses `HEAD~1` where another skill forbids it; no test for skills firing when they should not |

## Recommendations

- **Choose superpowers** if you want an autopilot that will interview you, plan, and then run for hours unattended, and you are on a harness with subagents. Expect about 900 tokens of bootstrap per session and a long design phase. If you fork it, the first cut is the bootstrap: keep the invoke-before-acting rule, the two routing examples, the platform pointers and the "user instructions take precedence" line (about 130 words), and drop the `EXTREMELY-IMPORTANT` block and the red-flag table.
- **Choose mattpocock/skills** if you want small tools you compose by hand with near-zero context tax. Run the setup skill, then set `disable-model-invocation: true` on `tdd` and `diagnosing-bugs` locally if they fire on tasks that do not need them, and add the CLAUDE.md lines the docs suggest ("ask one question at a time").
- **For either**: both maintainers assert cross-model behaviour they have not measured. Test on the models you use. The guidance's own loop applies: baseline without the skill, then add only what changes behaviour.
- **What the guidance would cut from both**: prohibition lists, exhaustive specs, and any instruction a capable model already follows by default.

## Files

- [reports/mattpocock-skills.md](reports/mattpocock-skills.md) and [reports/obra-superpowers.md](reports/obra-superpowers.md): the full analyst reports with inventory tables, hotspots with `path:line`, verdicts and the scripts that produced every number.
- [reports/brief.md](reports/brief.md): the brief and rubric both analysts worked from.
- [reports/openai-gpt6-astra-reconstructed.md](reports/openai-gpt6-astra-reconstructed.md): the reconstructed OpenAI recommendations, with provenance warning.
