# mattpocock/skills: evaluation report

Repo: https://github.com/mattpocock/skills, local clone `repos/mattpocock-skills`. All paths below are relative to the clone root unless prefixed `refs/`. Reference docs: `refs/agent-skills-best-practices.md` (Anthropic skills, "AS"), `refs/claude-code-best-practices.md` (Anthropic Claude Code, "CC"), `refs/openai-gpt6-astra-reconstructed.md` (OpenAI post, reconstructed from press coverage, secondary, "OA"). Every number comes from the scripts in the Appendix.

**TL;DR.** A mid-sized (37 skills, 27 shipped), low-shouting library. Its instruction style mostly follows current guidance: 8 emphatic ALL-CAPS tokens in 24k words, zero IMPORTANT, zero rationalisation tables. The maintainer's own `writing-for-agents` skill reads like an independent derivation of the Anthropic/OpenAI advice. The weak spots are **proportionality** and **evaluation**. Two model-invoked skills (`tdd`, `diagnosing-bugs`) and the grilling/spec chain impose fixed ceremony (human gates, exhaustive interviews, 6-phase diagnosis) that does not scale down. The repo has no behaviour evals, yet the README claims the skills "work with any model", and its own docs record model-specific failures that contradict that claim.

---

## 1. Facts

| Item | Value | Evidence |
|---|---|---|
| Commit | `d81f3a183412e71a5b1e84ca21bc1a35eea03a60`, 2026-09-29 13:37:40 +0100, "Merge pull request #1120 from mattpocock/release/v1.3" | `git log -1`; the clone is shallow (1 commit), so no history analysis |
| Version | 1.2.3 (13 unreleased changesets pending in `.changeset/`) | `package.json:3`, `.claude-plugin/plugin.json:3` |
| Skills (37 SKILL.md) | engineering 20 (11 user-invoked / 9 model-invoked), productivity 7 (5/2), misc 4 (0/4), in-progress 6 (6/0), deprecated 0 | script; bucket meanings `CLAUDE.md:1-7` |
| **Promoted** (shipped) | engineering + productivity = **27** (16 user / 11 model). `plugin.json` lists exactly these 27 (script: 0 missing, 0 extra) | `.claude-plugin/plugin.json:21-48`, `CLAUDE.md:9` |
| Words in SKILL.md | 25,611 incl. frontmatter (bodies 24,481); promoted 19,923 (bodies 19,097) | script |
| Words in reference files | 11,354 in 23 `.md` refs (all in promoted skills), plus 1,874 words of bundled scripts/config in 4 files (`template.sh`, `hitl-loop.template.sh`, `block-dangerous-git.sh`, `dependency-cruiser.config.cjs`) | script |
| Body size | median 534 words / 70 lines; max 1,954 words (`wayfinder`), max 160 lines (`pr`); **0 bodies over 500 lines**; 7 bodies are 12 lines or fewer | script |
| Always-loaded metadata | 11 promoted model-invoked descriptions = **2,150 chars** (2,260 with names), about 27% of OA's 8,000-char fallback budget. With misc: 15 skills, 3,003 chars. The 22 user-invoked descriptions are kept out of model context by design | script; `.agents/invocation.md:5` |
| Hooks | **None shipped.** No SessionStart or bootstrap hook. The only hook in the repo is one a misc skill installs on request (PreToolUse git blocker) | `skills/misc/git-guardrails-claude-code/SKILL.md:8,37-59` |
| Plugin manifests | `.claude-plugin/plugin.json`; `.claude-plugin/marketplace.json` (undocumented fallback); no Codex plugin (deferred); `agents/openai.yaml` in all 37 skill dirs | `.agents/install-block.md:59-61`, `.agents/adr/0002-ship-as-a-claude-code-plugin.md:19-23` |
| Harness support | Claude Code via the official-marketplace plugin; Codex and others via `npx skills@latest add`; `AGENTS.md` is a symlink to `CLAUDE.md` | `README.md:34-57`, `CHANGELOG.md:36` |
| How skills trigger | Model-invoked: description only. User-invoked: typed slash command only (`disable-model-invocation: true` plus `policy.allow_implicit_invocation: false`; script finds 0 mismatches across 37). Skill-to-skill: "Call the Skill tool with X", 17 edges. No skill can call a user-invoked skill | `.agents/invocation.md:5-8,16,22` |
| Evals / tests | **None for skill behaviour.** `package.json:7-11` scripts are changeset/version only; CI (`.github/workflows/release.yml:29-34`) only versions; manifest validation is a manual command (`CLAUDE.md:11`). Maintainer docs: "There is no automated eval here; the check is a manual run" | `docs/productivity/writing-for-agents.md:47` |
| Feedback loop actually used | GitHub issues are mined into each docs page's "Common questions" section | `.agents/writing-docs.md:48-56` |

---

## 2. Inventory (one row per SKILL.md)

`*` = promoted (ships in plugin). `inv` = invocation as configured in the files (`model` means model- and user-reachable). lines/words = SKILL.md body after frontmatter. `refs n/depth` = bundled files and the max link depth from SKILL.md (BFS over path mentions). `CAPS` = emphatic ALL-CAPS tokens outside code with acronyms removed (prototype's 1 is the literal name hint "PROTOTYPE, wipe me"; git-guardrails' 1 is the literal message "BLOCKED"). `A/N/M` = `always|never|must` in any case outside code. The CAPS forms ALWAYS/NEVER/MUST and IMPORTANT/CRITICAL each occur **0 times in all 37 skills**. Scores: 1-5, 5 = aligned with guidance (rubric in brief).

| skill | bucket | inv | lines | words | desc ch | refs (n/depth) | CAPS | A/N/M | A | B | C | D | E | F | G | H | I | J | mean |
|---|---|---|--:|--:|--:|---|--:|--:|--|--|--|--|--|--|--|--|--|--|--:|
| ask-matt | eng* | user | 90 | 1895 | 83 | 1/1 | 0 | 1 | 3 | 4 | 5 | 4 | 3 | 4 | 4 | 1 | 3 | 3 | 3.4 |
| code-review | eng* | model | 83 | 991 | 417 | 0/0 | 0 | 5 | 4 | 3 | 5 | 4 | 4 | 5 | 3 | 2 | 4 | 3 | 3.7 |
| codebase-design | eng* | model | 110 | 806 | 265 | 2/1 | 0 | 2 | 3 | 3 | 5 | 5 | 3 | 4 | 4 | 2 | 4 | 4 | 3.7 |
| diagnosing-bugs | eng* | model | 134 | 1378 | 156 | 1/1 | 0 | 5 | 3 | 2 | 3 | 3 | 5 | 4 | 2 | 2 | 3 | 4 | 3.1 |
| domain-modeling | eng* | model | 70 | 465 | 151 | 2/1 | 0 | 0 | 4 | 4 | 5 | 5 | 3 | 4 | 5 | 2 | 4 | 5 | 4.1 |
| grill-with-docs | eng* | user | 2 | 9 | 106 | 0/0 | 0 | 0 | 5 | 4 | 4 | 5 | 3 | 5 | 3 | 2 | 5 | 3 | 3.9 |
| implement | eng* | user | 10 | 50 | 60 | 0/0 | 0 | 0 | 5 | 4 | 4 | 4 | 3 | 5 | 3 | 2 | 2 | 4 | 3.6 |
| implement-spec | eng* | user | 35 | 419 | 57 | 0/0 | 0 | 1 | 4 | 4 | 4 | 4 | 4 | 5 | 4 | 1 | 4 | 2 | 3.6 |
| improve-codebase-architecture | eng* | user | 66 | 872 | 125 | 1/1 | 1 | 0 | 3 | 4 | 4 | 3 | 3 | 4 | 3 | 2 | 4 | 2 | 3.2 |
| pr | eng* | model | 160 | 546 | 27 | 1/0 (orphan: CREDITS.md) | 0 | 0 | 3 | 3 | 4 | 3 | 4 | 5 | 3 | 1 | 3 | 5 | 3.4 |
| prototype | eng* | model | 22 | 450 | 179 | 2/1 | 1 | 0 | 4 | 5 | 4 | 5 | 4 | 3 | 4 | 1 | 4 | 4 | 3.8 |
| research | eng* | model | 8 | 87 | 238 | 0/0 | 0 | 0 | 5 | 4 | 5 | 5 | 4 | 5 | 3 | 2 | 5 | 2 | 4.0 |
| retro | eng* | user | 39 | 665 | 44 | 0/0 | 1 | 0 | 4 | 4 | 5 | 4 | 3 | 4 | 4 | 1 | 4 | 3 | 3.6 |
| setup-matt-pocock-skills | eng* | user | 111 | 972 | 180 | 5/1 | 0 | 2 | 4 | 4 | 5 | 5 | 3 | 4 | 5 | 1 | 3 | 3 | 3.7 |
| tdd | eng* | model | 34 | 534 | 149 | 2/1 | 0 | 3 | 5 | 3 | 3 | 5 | 4 | 4 | 2 | 2 | 3 | 4 | 3.5 |
| to-spec | eng* | user | 70 | 462 | 149 | 0/0 | 3 | 0 | 4 | 4 | 3 | 4 | 3 | 3 | 3 | 1 | 4 | 3 | 3.2 |
| to-tickets | eng* | user | 100 | 846 | 247 | 0/0 | 3 | 3 | 4 | 3 | 4 | 4 | 4 | 3 | 4 | 2 | 3 | 3 | 3.4 |
| triage | eng* | user | 107 | 962 | 136 | 2/1 | 0 | 4 | 3 | 4 | 4 | 3 | 5 | 4 | 4 | 2 | 4 | 3 | 3.6 |
| wayfinder | eng* | user | 123 | 1954 | 197 | 0/0 | 0 | 8 | 2 | 4 | 3 | 3 | 4 | 3 | 4 | 1 | 4 | 2 | 3.0 |
| wizard | eng* | model | 40 | 621 | 313 | 1/1 | 0 | 4 | 4 | 4 | 5 | 5 | 5 | 4 | 4 | 2 | 5 | 4 | 4.2 |
| grill-me | prod* | user | 2 | 6 | 51 | 0/0 | 0 | 0 | 5 | 4 | 4 | 5 | 3 | 5 | 3 | 2 | 5 | 3 | 3.9 |
| grilling | prod* | model | 24 | 289 | 152 | 0/0 | 0 | 1 | 5 | 4 | 4 | 5 | 5 | 5 | 2 | 2 | 5 | 4 | 4.1 |
| handoff | prod* | user | 10 | 108 | 86 | 0/0 | 0 | 0 | 5 | 4 | 5 | 5 | 3 | 5 | 5 | 1 | 4 | 3 | 4.0 |
| teach | prod* | user | 134 | 1462 | 61 | 4/1 (orphan: GLOSSARY-FORMAT.md) | 0 | 3 | 3 | 4 | 3 | 3 | 2 | 4 | 4 | 2 | 3 | 3 | 3.1 |
| to-questionnaire | prod* | user | 49 | 447 | 88 | 0/0 | 0 | 4 | 4 | 4 | 5 | 4 | 5 | 5 | 5 | 1 | 4 | 5 | 4.2 |
| wait-what | prod* | user | 2 | 44 | 50 | 0/0 | 0 | 0 | 5 | 3 | 4 | 5 | 2 | 5 | 5 | 1 | 4 | 5 | 3.9 |
| writing-for-agents | prod* | model | 77 | 1757 | 103 | 1/1 | 0 | 8 | 3 | 5 | 5 | 4 | 3 | 5 | 4 | 1 | 4 | 5 | 3.9 |
| git-guardrails-claude-code | misc | model | 91 | 257 | 243 | 1/1 | 1 | 0 | 4 | 4 | 5 | 4 | 5 | 5 | 5 | 1 | 4 | 2 | 3.9 |
| migrate-to-shoehorn | misc | model | 114 | 385 | 168 | 0/0 | 0 | 2 | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 1 | 3 | 4 | 3.6 |
| scaffold-exercises | misc | model | 102 | 407 | 204 | 0/0 | 0 | 1 | 4 | 3 | 5 | 4 | 5 | 5 | 4 | 1 | 4 | 2 | 3.7 |
| setup-pre-commit | misc | model | 87 | 294 | 238 | 0/0 | 0 | 0 | 4 | 4 | 5 | 4 | 5 | 5 | 4 | 1 | 4 | 4 | 4.0 |
| claude-handoff | beta | user | 12 | 167 | 97 | 0/0 | 0 | 1 | 5 | 4 | 5 | 5 | 3 | 4 | 5 | 1 | 4 | 1 | 3.7 |
| loop-me | beta | user | 26 | 383 | 78 | 0/0 | 0 | 2 | 4 | 3 | 5 | 4 | 4 | 5 | 3 | 1 | 3 | 4 | 3.6 |
| setup-ts-deep-modules | beta | user | 97 | 1077 | 185 | 1/1 | 0 | 7 | 3 | 3 | 5 | 4 | 5 | 4 | 3 | 1 | 4 | 3 | 3.5 |
| writing-beats | beta | user | 62 | 826 | 111 | 0/0 | 0 | 4 | 3 | 3 | 4 | 4 | 3 | 4 | 4 | 1 | 3 | 5 | 3.4 |
| writing-fragments | beta | user | 74 | 580 | 55 | 0/0 | 0 | 2 | 4 | 3 | 5 | 4 | 2 | 4 | 4 | 1 | 3 | 5 | 3.5 |
| writing-shape | beta | user | 74 | 1008 | 77 | 0/0 | 0 | 3 | 3 | 3 | 4 | 4 | 3 | 4 | 3 | 1 | 3 | 5 | 3.3 |

Mean score by bucket: engineering 3.58, productivity 3.87, misc 3.80, in-progress 3.50. Promoted per-criterion means: A 3.93, B 3.78, C 4.22, D 4.22, **E 3.59, F 4.30, G 3.67, H 1.56**, I 3.85, J 3.48. (The A-J scores are my judgement; the script only merges them into the table.)

**Scoring notes that apply across the table**
- **H (evals)** is 1 everywhere because no behaviour evals exist. It is 2 where the docs page records observed cross-model behaviour from issues (field observation, not evals): e.g. `docs/productivity/grilling.md:62`, `docs/engineering/diagnosing-bugs.md:59`.
- **B for user-invoked skills**: the repo deliberately makes these descriptions human-facing one-liners with triggers stripped (`.agents/invocation.md:5`). They never enter model context, so I did not penalise missing "Use when". All 15 model-invoked descriptions contain "Use when"; none of the 22 user-invoked ones do (script).

**Supplementary metrics (script)**
- **References over 100 lines, none with a TOC** (AS:404-406 asks for one): `triage/AGENT-BRIEF.md` 207, `improve-codebase-architecture/HTML-REPORT.md` 123, `prototype/UI.md` 112, `triage/OUT-OF-SCOPE.md` 105.
- **Nesting depth** is 1 everywhere. Cross-links exist (`DESIGN-IT-TWICE.md` to `DEEPENING.md`, `LOGIC.md` to `UI.md`), but every target is also linked from SKILL.md, so AS:374-378 is respected.
- **Orphan references**: `pr/CREDITS.md` (attribution only, harmless) and `teach/GLOSSARY-FORMAT.md` (a functional loss, acknowledged in `docs/productivity/teach.md:38`, issue #559).
- **Prohibition tokens** (`don't|do not|never|avoid` outside code) in SKILL.md bodies: 110 total, 88 in promoted skills. Top: wayfinder 11, codebase-design 9, setup-ts-deep-modules 7, improve-codebase-architecture 6, wizard 6. In references: `prototype/LOGIC.md` 6, `HTML-REPORT.md` 5.
- **Bold**: 550 spans across SKILL.md bodies (ask-matt 87, wayfinder 53, codebase-design 44, diagnosing-bugs 44). Bold, not caps, is the house emphasis, mostly used to introduce a "leading word" rather than to shout.
- **Description voice**: second or first person appears in 6 descriptions: `ask-matt`, `improve-codebase-architecture`, `to-spec`, `wayfinder` and `to-questionnaire` ("you/your"), and `loop-me` ("me/I"). AS:206 asks for third person. All 6 are user-invoked, so none of them enters model context.
- **Naming** is mixed: gerund-led (`grilling`, `diagnosing-bugs`, `writing-for-agents`, `writing-*`), imperative (`implement`, `triage`, `teach`, `to-spec`, `to-tickets`), noun (`code-review`, `codebase-design`, `pr`, `wizard`, `research`, `retro`, `handoff`) and conversational (`ask-matt`, `grill-me`, `wait-what`, `loop-me`). AS:192: "Avoid ... Inconsistent patterns within your skill collection".
- **Flow load** (SKILL.md files plus the references a flow normally reads, words x 1.33 as approximate tokens):

| Flow | Words loaded | approx. tokens |
|---|--:|--:|
| Feature, planning session: grill-with-docs, grilling, domain-modeling, GLOSSARY-FORMAT, ADR-FORMAT, to-spec, to-tickets | 3,001 | ~4.0k |
| Feature, each ticket session: implement, tdd, tests.md, mocking.md, code-review | 2,186 | ~2.9k |
| Optional: codebase-design pulled in by tdd | 851 | ~1.1k |
| Bug: diagnosing-bugs | 1,402 | ~1.9k |
| Router: ask-matt plus PHASE-BOUNDARIES.md | 2,617 | ~3.5k |
| Setup, worst case (skill plus all 5 seed templates) | 2,819 | ~3.7k |

### Consistency and drift defects (inform the I scores)

| Defect | Evidence |
|---|---|
| The router still claims a hand-off that was removed | `skills/engineering/ask-matt/SKILL.md:48` ("Its post-mortem hands off to `/improve-codebase-architecture`") versus `.changeset/user-invoked-skill-invocation.md:8` (hand-off "Removed ... outright") and `skills/engineering/diagnosing-bugs/SKILL.md:130-138` (Phase 6 is Cleanup only). The same stale claim appears in `docs/engineering/diagnosing-bugs.md:21,54,93`. `CLAUDE.md:21` calls this "a router that lies". |
| Dangling instruction | `diagnosing-bugs/SKILL.md:120`: "Flag this for the next phase", but the next phase is now Cleanup |
| Docs say redaction is missing; the skill has it | `docs/engineering/diagnosing-bugs.md:71` ("open and unimplemented") versus `skills/engineering/diagnosing-bugs/SKILL.md:12-16` and `CHANGELOG.md:7-11` |
| Docs say skills name Claude Code's `Agent` tool; the text is now neutral | `docs/engineering/codebase-design.md:72` and `docs/engineering/improve-codebase-architecture.md:84` versus `codebase-design/DESIGN-IT-TWICE.md:21`, `improve-codebase-architecture/SKILL.md:27` and `CHANGELOG.md:13` |
| Docs claim a behaviour that code-review lacks | `docs/engineering/tdd.md:35,94` ("code-review checks ... only the agreed seams were used"); `code-review/SKILL.md` has no seam check |
| Trigger phrase contradicts the body | `tdd/SKILL.md:3` "red-green-refactor" versus `tdd/SKILL.md:38` "Refactoring is not part of the loop" (`docs/engineering/tdd.md:51`, issue #589) |
| Cross-skill convention broken | `.agents/invocation.md:16` requires "Call the Skill tool", not a bare `/skill`. `implement/SKILL.md:9,13` ("Use /tdd", "use /code-review") and `in-progress/loop-me/SKILL.md:8` use bare names |
| README misdescribes setup | `README.md:78` "(GitHub, Linear, or local files)" versus `setup-matt-pocock-skills/SKILL.md:44-47` (GitHub/GitLab/Local/Other). `README.md:80` "Ask you where you want to save any docs" versus `setup-matt-pocock-skills/SKILL.md:59` ("write it without asking") |
| Glossary out of date | `GLOSSARY.md:19` `ready-for-afk` versus `triage/SKILL.md:35` `ready-for-agent`. `GLOSSARY.md:13` "_Avoid_: ticket" versus the `to-tickets` skill name and body |
| Duplication, against its own single-source rule | ADR criteria in `domain-modeling/SKILL.md:68-74` and `domain-modeling/ADR-FORMAT.md:31-37`. Folder trees in `domain-modeling/SKILL.md:12-38`, `setup-matt-pocock-skills/domain.md:13-39` and `domain-modeling/GLOSSARY-FORMAT.md:32-58`. "One adapter ..." in `codebase-design/SKILL.md:65` and `codebase-design/DEEPENING.md:29`. The rule itself is `writing-for-agents/SKILL.md:78` |
| ADR mentions a deleted bucket | `.agents/adr/0002-ship-as-a-claude-code-plugin.md:9,14,17` (`personal/`) versus `CHANGELOG.md:141` |
| House rules violated | `.agents/writing-docs.md:75` "Never name the author" versus 5 docs lines naming Matt (e.g. `docs/engineering/tdd.md:75`, `docs/engineering/setup-matt-pocock-skills.md:59`). `CLAUDE.md:25` bans em-dashes "anywhere", including `CHANGELOG.md`, which still has 104 (script) |

---

## 3. Architecture and philosophy

### In the maintainer's words
- Positioning: "Approaches like GSD, BMAD, and Spec-Kit try to help by owning the process. But while doing so, they take away your control" (`README.md:17`).
- Claim: "These skills are designed to be small, easy to adapt, and composable. They work with any model." (`README.md:19`).
- Usage advice: grilling skills "help you align with the agent ... Use them _every_ time you want to make a change." (`README.md:103`).
- The one axis: "**User-invoked** skills ... their job is to orchestrate. **Model-invoked** skills ... hold the reusable discipline." (`README.md:186`). A user-invoked skill "may invoke model-invoked skills, but it can never reach another user-invoked skill" (`.agents/invocation.md:8`).
- Invocation economics: "Pick model-invocation only when the agent must reach the skill on its own ... If it only ever fires by hand, make it user-invoked and pay no context load" (`skills/productivity/writing-for-agents/SKILL-MECHANICS.md:12`).
- Hard vs soft dependencies on setup: the split "keeps soft-dependency skills token-light and avoids cargo-culting the setup pointer" (`.agents/adr/0001-explicit-setup-pointer-only-for-hard-dependencies.md:10`).
- On forcing compliance: "No instruction makes an agent comply 100% of the time, and forcing the point harder restricts the agent's creativity for little gain" (`docs/engineering/tdd.md:59`).
- On length: "Concision skills fail by growing: a 400-line skill still leaves the model verbose" (`CHANGELOG.md:78`; em-dash rendered as a colon).

### `writing-for-agents` vs Anthropic/OpenAI guidance (and whether the repo follows it)

| Principle | AS / CC / OA | `writing-for-agents` | Does the repo follow it? |
|---|---|---|---|
| Cut what the model knows | AS:24-30 "Claude is already very smart"; OA:48-51 fewer rules | No-op test: "an instruction the model already obeys by default pays load to say nothing" (`SKILL.md:81`) | Mostly. Exceptions: textbook testability in `codebase-design/SKILL.md:67-95`, pedagogy theory in `teach/SKILL.md:34-45` |
| Progressive disclosure | AS:251-259, 374-378; OA:33-35 | "inline what every branch needs, and push behind a pointer what only some branches reach" (`SKILL.md:39`) | Yes in `prototype` and `setup`. No in `wayfinder` (1,954 words, 0 refs) and `diagnosing-bugs` (134 lines, 0 md refs). No TOCs on 4 long refs (WFA never mentions TOCs) |
| Descriptions | AS:201-215 what + when, third person; OA:28-32 prune to function and triggers | "One trigger per branch ... Cut identity the body already carries" (`SKILL.md:16-18`) | Mostly. `pr` ("Use when writing a PR body.", 27 chars) cuts the *what* that AS asks for. `code-review` (417 chars) carries mechanism detail |
| Define done | CC:27-52; OA:42-47 | "Every step ends on a completion criterion ... strongest criteria are both checkable and exhaustive" (`SKILL.md:47,52`) | Strong in `diagnosing-bugs`, `wizard`, `to-questionnaire`, `grilling`. The "exhaustive" half pushes against proportionality (section 4) |
| Positive framing | OA:48-49 point a direction; CC:188 emphasis on one line only | "Negation ... drags the forbidden behaviour into context ... Prompt the positive" (`SKILL.md:74`) | Partly: 110 prohibition tokens, banned-word lists (`codebase-design/SKILL.md:12`, `HTML-REPORT.md:112`), and the `LOGIC.md:60-67` "Don't" list |
| Single source of truth | AS:601-617 consistent terms | `SKILL.md:78` | Partly (duplications listed in section 2) |
| Evaluate | AS:744-756 evals first; AS:134-144 test all models | "settle it by running the document, not by debate" (`SKILL.md:81`); docs: "There is no automated eval here" (`docs/productivity/writing-for-agents.md:47`) | No: no evals and no model matrix |
| Need a "writing skills" skill? | AS:791-793 "You don't need ... a 'writing skills' skill" | The repo ships one, arguing a model left alone "will produce something verbose" (`docs/productivity/writing-for-agents.md:41`) | Deliberate divergence, argued but unmeasured |
| Leading words | (not in AS, CC or OA) | A pretrained word "anchors a whole region of behaviour in the fewest tokens" (`SKILL.md:63`) | Used heavily (tight, red, tracer bullet, frontier, fog of war, seam). The skill itself coins about 15 new terms (context pointer, two loads, sediment, sprawl, legwork, cache...), which by its own `:63` "recruits no priors" |

Net: `writing-for-agents` is close to the Anthropic/OpenAI advice on concision, disclosure, done-criteria and positive framing, and adds a useful invocation-cost model. It diverges on evaluation: it substitutes manual runs for evals and has no cross-model testing. The repo follows it unevenly, best in the small skills and worst in `wayfinder`, `teach` and the prohibition lists.

### How the pieces fit (my reading)

**Entry points.** The human is the index. There are 16 typed commands in the plugin, plus 11 model-invoked skills whose 2,150 chars of descriptions are always loaded. No hook, no bootstrap skill and no "using-skills" meta-skill. `ask-matt` is a typed router that "recommends and stops" (`docs/engineering/ask-matt.md:5`). It cannot fire anything because user-invoked skills are unreachable by skills (`.agents/invocation.md:8`).

**Flow: "build feature X"** (`skills/engineering/ask-matt/SKILL.md:13-36`)
1. `/grill-with-docs` makes two Skill-tool calls, to `grilling` and `domain-modeling` (`grill-with-docs/SKILL.md:7`). It asks rounds of questions until "the frontier is empty", then waits for the user to confirm (`grilling/SKILL.md:8,28`), writing `GLOSSARY.md` and ADRs lazily (`domain-modeling/SKILL.md:40,60-74`).
2. Optional prototype detour, bridged both ways by `/handoff` (`ask-matt/SKILL.md:18-21`).
3. Multi-session build: `/to-spec` (publishes the spec, labels it `ready-for-agent`, `to-spec/SKILL.md:19`), then `/to-tickets` (user approval loop `:56`; published blockers-first `:60-65`), then `/implement` once per ticket with `/clear` between tickets (`ask-matt/SKILL.md:24`), or `/implement-spec` (subagents in worktrees). Single-session build: `/implement` directly (`ask-matt/SKILL.md:26`).
4. `implement` = tdd at pre-agreed seams, then typecheck, then the full suite once, then code-review, then commit (`implement/SKILL.md:7-15`). `pr` (model-invoked) shapes the PR body, then `/retro`.
5. Every user-invoked step is typed by the human; nothing chains them. Load: about 3.0k words for planning and about 2.2k words per ticket session.

**Flow: "fix typo".** No skill is designed to fire, and none of the 11 model-invoked descriptions matches a typo. The cost is only the 2,150 chars of always-on descriptions. The only push toward ceremony is advice to the human (`README.md:103`). `ask-matt` has no explicit "don't use a skill" branch; its smallest route is "`/implement` right here" (`ask-matt/SKILL.md:26`).

**Flow: "fix this bug".** `diagnosing-bugs` auto-fires on "diagnose"/"debug this" or on "something broken/throwing/failing/slow" (`diagnosing-bugs/SKILL.md:3`). It runs 6 gated phases: a red-capable loop (`:57-66`), reproduce and minimise (`:86`), 3-5 ranked hypotheses shown to the user, non-blocking if the user is AFK (`:90-98`), instrumentation, a regression test at a correct seam (`:116-128`), then a cleanup checklist (`:132-138`). That is about 1.4k words. If the user says "test-first", `tdd` may also fire with its seam-confirmation gate (`tdd/SKILL.md:22`). For an incoming report from someone else: `/triage` (verifies the claim, `triage/SKILL.md:74`), then an agent brief, then `/implement`.

**Subagent usage**

| Skill | Subagent pattern |
|---|---|
| `code-review` | Always 2 parallel subagents, Standards and Spec (`code-review/SKILL.md:11,58-70`) |
| `implement-spec` | Exploration, then one implementer per ticket in its own worktree, then a merger (`implement-spec/SKILL.md:23-36`) |
| `research` | One background agent (`research/SKILL.md:6`) |
| `grilling` | A subagent for facts (`grilling/SKILL.md:26`) |
| `improve-codebase-architecture` | Exploration subagent (`improve-codebase-architecture/SKILL.md:27`) |
| `codebase-design/DESIGN-IT-TWICE.md` | 3 or more design subagents (`:21`) |
| `wayfinder` | Research subagents (`wayfinder/SKILL.md:115`) |

Known recursion bugs, both still unfixed: code-review subagents re-invoke the skill, with "one report reached 50-plus agents" (`docs/engineering/code-review.md:56`); a research agent cost about 450k tokens across duplicate runs (`docs/engineering/research.md:35`).

**What a user must set up before the engineering skills work**
1. **Install.** Use the plugin (all 27) or skills.sh. With skills.sh you must pick `setup-matt-pocock-skills` (`README.md:55`) and the primitives the wrappers call: `grill-me` needs `grilling`, `grill-with-docs` also needs `domain-modeling` (`docs/productivity/grilling.md:70`), and `tdd` needs `codebase-design` (`docs/engineering/tdd.md:25`).
2. **Run `/setup-matt-pocock-skills` once per repo.** It writes:
   - `docs/agents/issue-tracker.md`, including the "Wayfinding operations" section;
   - `docs/agents/domain.md`;
   - `docs/agents/triage-labels.md`, only if `triage` is installed;
   - an `## Agent skills` block in `CLAUDE.md`, or `AGENTS.md` if no `CLAUDE.md` exists (`setup-matt-pocock-skills/SKILL.md:63-112`).
   
   Hard dependents tell you to run setup if it is missing: `to-spec:9`, `to-tickets:11`, `triage:43`, `wayfinder:25`, `implement-spec:9`, `code-review:13`. Soft dependents only read `GLOSSARY.md`/ADRs if present: `tdd:10`, `diagnosing-bugs:10`, `improve-codebase-architecture:25` (`.agents/adr/0001-explicit-setup-pointer-only-for-hard-dependencies.md:7-8`).
3. **Tooling.** An authenticated `gh` or `glab`. Create the triage labels yourself; setup does not (`docs/engineering/triage.md:74`, #616). GitHub sub-issues/dependencies for `wayfinder` (`setup-matt-pocock-skills/issue-tracker-github.md:41-43`). git worktrees for `implement-spec`.
4. **Codex caveat.** Setup picks `CLAUDE.md` whenever it exists, so on Codex move the block to `AGENTS.md` (`docs/engineering/setup-matt-pocock-skills.md:63`).
5. **Behaviour tuning lives in your own CLAUDE.md**, per the docs:
   - "When grilling, ask one question at a time." (`docs/productivity/grilling.md:46-50`);
   - a line forbidding implementation without permission, for weaker models (`docs/productivity/grilling.md:62`);
   - "browser tests after the behaviour works" (`docs/engineering/tdd.md:63`).

---

## 4. Hotspots (5 per worry)

### 4.1 LENGTH: too long, context cost, buried instruction
Context: at the budget level these are small. Nothing exceeds 160 lines, and the per-session loads in section 2 are 1-2% of a 200k window. The hotspots below are relative to the job, not absolute.

| # | Where | Quote / fact | Conflict |
|---|---|---|---|
| 1 | `skills/engineering/wayfinder/SKILL.md:105` | "never resolve more than one ticket per session" sits at line 105 of the largest body (1,954 words, 0 reference files) | AS:251-259 (SKILL.md as overview; split as it grows); OA:33-35 "routers, not textbooks"; the maintainer's own "Sprawl ... attention thins across the excess" (`writing-for-agents/SKILL.md:43`) |
| 2 | `skills/engineering/ask-matt/SKILL.md` (1,895 words) + `PHASE-BOUNDARIES.md` (699) | Restates every skill's purpose, which is also restated in `README.md:186-233`, the bucket READMEs and 28 docs pages; this already caused the stale claim at `:48` | AS:13-22 (context is a public good); `writing-for-agents/SKILL.md:78` "Duplication ... costs maintenance and tokens" |
| 3 | `skills/productivity/writing-for-agents/SKILL.md` (1,757 words in 77 lines) | Dense, abstract prose loaded whenever any skill or CLAUDE.md is edited (`:3`). It coins about 15 terms while warning that "a made-up word recruits no priors" (`:63`) | AS:24-30 (only add context Claude lacks); OA:16-20 (scaffolding bloats context) |
| 4 | `skills/productivity/teach/SKILL.md:34-45` | "Fluency can give the user an illusory sense of mastery, but storage strength is the real goal" (`:41`): pedagogy theory a frontier model knows, in a 1,462-word body | AS:24-59 ("Does Claude really need this explanation?") |
| 5 | `skills/engineering/triage/AGENT-BRIEF.md` (207 lines), `improve-codebase-architecture/HTML-REPORT.md` (123), `prototype/UI.md` (112), `triage/OUT-OF-SCOPE.md` (105) | Long references with no table of contents. AGENT-BRIEF carries 3 full good examples plus 1 bad one (`:72-207`) | AS:404-406 (TOC for references over 100 lines) |

### 4.2 "DUMBING DOWN": treating the model as a junior
The repo has almost none of the classic pattern: the rationalisation, red-flag, threat and "you cannot think your way out" counter is 0 in all 37 SKILL.md bodies. The only hits are a test-smell list headed "Red flags:" (`tdd/tests.md:38`) and the phrase "not an excuse" (`HTML-REPORT.md:108`). The 5 closest cases:

| # | Where | Quote | Conflict |
|---|---|---|---|
| 1 | `skills/engineering/diagnosing-bugs/SKILL.md:66` | "If you catch yourself reading code to build a theory before this command exists, stop: jumping straight to a hypothesis is the exact failure..." | Pre-empts the model's habits instead of trusting a stated done-criterion (OA:42-47). Mitigated because it sits next to a crisp criterion (`:57-64`) |
| 2 | `skills/engineering/diagnosing-bugs/SKILL.md:22,24-35` | "Be aggressive. Be creative. Refuse to give up." followed by a 10-item list of standard repro techniques (failing test, curl, CLI, Playwright, bisect...) | AS:24-30 (do not explain what Claude knows); exhortation adds no information |
| 3 | `skills/engineering/codebase-design/SKILL.md:67-95` | "Accept dependencies, don't create them." / "Return results, don't produce side effects." with TypeScript before/after; repeated in `tdd/mocking.md:20-35` | AS:24-59 (the "bad: too verbose" pattern); also duplication |
| 4 | `skills/productivity/teach/SKILL.md:30` | "Never trust your parametric knowledge." Blanket distrust that sits oddly with `:116` "your default posture should be to attempt to answer" | AS:24 "Claude is already very smart"; CC:570-578 (patterns are starting points) |
| 5 | `skills/engineering/improve-codebase-architecture/HTML-REPORT.md:121-123` | "Don't write *'easier to maintain'* or *'cleaner code'* ... No hedging, no throat-clearing ... If a bullet could be cut, cut it." | OA:48-51 (fewer rules); micro-management of prose style |

### 4.3 OUTDATED TECHNIQUES: caps, ALWAYS/NEVER walls, IMPORTANT, prohibition lists
Counts across 24,481 SKILL.md body words:
- 8 emphatic ALL-CAPS tokens: `to-spec` 3, `to-tickets` 3, `improve-codebase-architecture` 1, `retro` 1;
- 0 IMPORTANT or CRITICAL;
- 0 ALWAYS/NEVER/MUST in caps;
- 76 lower-case always/never/must;
- 110 prohibition tokens.

The hotspots are prohibition phrasing, not shouting.

| # | Where | Quote | Conflict |
|---|---|---|---|
| 1 | `skills/engineering/to-spec/SKILL.md:7,33,55` | "Do NOT interview the user" / "A LONG, numbered list of user stories" / "Do NOT include specific file paths" | CC:188 (emphasis stands out only on one line); OA:48-49 |
| 2 | `skills/engineering/to-tickets/SKILL.md:31,67` | "a narrow but COMPLETE path ... vertical, NOT a horizontal slice"; "Do NOT close or modify any parent issue." | Same |
| 3 | `skills/engineering/prototype/LOGIC.md:60-67` (also `UI.md:107-112`) | Six-bullet "Anti-patterns" list: "Don't add tests." "Don't wire it to the real database." "Don't generalise." ... | The maintainer's own rule: "steering by prohibition drags the forbidden behaviour into context ... Prompt the positive" (`writing-for-agents/SKILL.md:74`); OA:48-49 |
| 4 | `skills/engineering/codebase-design/SKILL.md:12`, `improve-codebase-architecture/SKILL.md:13`, `HTML-REPORT.md:112` | "**Never substitute:** component, service, unit (for module) · API, signature (for interface) · boundary (for seam) · layer, wrapper" | Banned-word lists prime the banned words (same `:74` rule). Partly mitigated because each list pairs with a positive term set |
| 5 | `skills/engineering/wayfinder/SKILL.md:17,79,91,105` | "never by a bare id, number, or slug"; "Always call the Skill tool twice"; "Don't pre-slice the fog"; "never resolve more than one ticket per session". The skill has the most prohibition tokens (11) and 8 always/never/must | OA:48-51; CC:174 ("Bloated ... files cause Claude to ignore your actual instructions") |

### 4.4 BUREAUCRACY / OVER-CONSTRAINT
This is the real concern in this repo.

| # | Where | Quote | Conflict |
|---|---|---|---|
| 1 | `skills/engineering/tdd/SKILL.md:22` (model-invoked) | "Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam." The docs call it "absolute" (`docs/engineering/tdd.md:35`), name it the "most-reported friction" (`:55`, #607) and admit the skill never decides "*whether* a change is worth the loop" (`:21`, #746 open) | CC:105-111 ("If you could describe the diff in one sentence, skip the plan"); AS:61-72 (high freedom for judgement tasks); OA:42-47 |
| 2 | `skills/engineering/diagnosing-bugs/SKILL.md:3,8,86` (model-invoked) | Description fires on "reports something broken/throwing/failing/slow". "Skip phases only when explicitly justified." "Do not proceed until you have reproduced **and** minimised." Docs: it "triggers the rather formal diagnosing-bugs skill ... considerable reply delays" (`docs/engineering/diagnosing-bugs.md:59`, #578, fix "has not landed") | CC:105-111; OA:16-18 ("unnecessary pauses", "redundant work") |
| 3 | `skills/productivity/grilling/SKILL.md:6,28` + `README.md:103` | "Interview the user relentlessly"; done when "every branch of the design tree visited"; "Use them *every* time you want to make a change." No cap, by policy (`.out-of-scope/question-limits.md:3-7`). "Forty-six questions across four rounds is an ordinary session" (`docs/productivity/grill-me.md:49`) | CC:105-111; CC:337-355 recommends interviews "for larger features" only |
| 4 | `skills/engineering/to-spec/SKILL.md:33,41` + `to-tickets/SKILL.md:56` | "A LONG, numbered list of user stories" ... "extremely extensive and cover all aspects of the feature"; "Iterate until the user approves the breakdown." | AS:13-22 (the bloated artifact is loaded by every later session); CC:355 ("The most useful specs are self-contained" and precise, not exhaustive) |
| 5 | `skills/engineering/implement-spec/SKILL.md:36` (+ `code-review/SKILL.md:11`, `implement/SKILL.md:13`) | "Fix all issues raised by the code review in a single implementer subagent." Code review always spawns 2 subagents, and inside `implement` it runs on an uncommitted (empty) diff (`docs/engineering/implement.md:65`) | CC:547-549 ("Chasing every finding leads to over-engineering"); the repo's own docs: "do not run it in a loop until it comes back clean" (`docs/engineering/code-review.md:72`) |

Also noted: `improve-codebase-architecture` users report "10's or 100's of questions" after its grilling loop was added, making it "borderline unusable" (`docs/engineering/improve-codebase-architecture.md:56`).

---

## 5. Good citizens (5)

| # | Where | Why it aligns |
|---|---|---|
| 1 | **The grilling family**: `grill-me/SKILL.md:7`, `grill-with-docs/SKILL.md:7` (one-line Skill-tool calls) over `grilling/SKILL.md` (24 lines) | One source of truth for the interview. A crisp definition of done: "The session is done when the frontier is empty" plus a confirmation gate (`:28`). The facts-vs-decisions split (`:26`) came from observed failures (`CHANGELOG.md:171`). Matches OA:42-47 and AS:13-30 |
| 2 | `skills/engineering/prototype/SKILL.md:12-17` | 22-line router: logic questions go to `LOGIC.md`, UI questions to `UI.md`. When the question is ambiguous it picks a default and "state[s] the assumption" (`:17`). Textbook AS:353-372 (conditional details) and OA:33-35 |
| 3 | `skills/engineering/wizard/SKILL.md:10,25,31,41-43` + `template.sh` | The deterministic UX lives in a script: "**Your job is only to scope the procedure and author its stages.**" Each step has its own "Done when". Verification is `bash -n`/shellcheck plus a static trace. The description names an explicit non-trigger (`:3`). Matches AS:107-127 (low freedom where fragile), AS:876-938 (utility scripts, solve don't defer), CC:27-52 |
| 4 | `skills/engineering/diagnosing-bugs/SKILL.md:57-64` (the passage, not the whole skill) | "name **one command** ... that you have **already run at least once** (show the invocation and its output, redacted)", with red-capable, deterministic, fast and agent-runnable checks. The best verification criterion in the repo. CC:27-52 ("Give Claude a check it can run", "show evidence") |
| 5 | `skills/engineering/setup-matt-pocock-skills/SKILL.md:15,36` + `.agents/adr/0001-explicit-setup-pointer-only-for-hard-dependencies.md:7-10` | "This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write." It skips sections that exploration already settled. Tracker specifics live in one per-repo doc that every skill reads ("When a skill says 'publish to the issue tracker'", `issue-tracker-github.md:28-30`). Matches OA:37-39 (contextual guidance behind pointers) and AS progressive disclosure |

Honourable mentions:
- `tdd` was cut from a workflow to a reference: "the step-by-step Workflow was largely restating the loop" (`CHANGELOG.md:209`).
- `code-review` fails fast on a bad ref or empty diff (`code-review/SKILL.md:23`) and keeps smells as labelled "judgement call[s]" that the repo's own standards override (`:40-41`).
- `to-questionnaire` gives every step a "Done when" (`:12-16`).
- The docs policy "say the unflattering thing where it is true" (`.agents/writing-docs.md:56`) produced unusually candid failure documentation.

---

## 6. Capability map

| Category | What the developer gets |
|---|---|
| **Automates (artifacts)** | Specs and dependency-ordered tickets published to GitHub, GitLab or local markdown (native blocking links where available). A triage state machine with agent briefs and an `.out-of-scope/` knowledge base. Architecture-review HTML reports. PR bodies (diagram + evidence + "door"). Interactive bash setup wizards. Cited research notes. Handoff docs. Teaching workspaces. `GLOSSARY.md` and ADRs maintained inline |
| **Automates (orchestration)** | `implement-spec`: a parallel ticket graph with implementer subagents in worktrees and merges into one integration branch. `code-review`: two-axis review in parallel subagents. `wayfinder`: a multi-session decision map on the tracker with claim, frontier and resolve operations |
| **Enforces deterministically** | Nothing in the shipped plugin: no hooks or scripts gate behaviour. Opt-in installers outside the plugin: a git-command blocking hook (`misc/git-guardrails-claude-code`), husky pre-commit (`misc/setup-pre-commit`), dependency-cruiser boundaries (`in-progress/setup-ts-deep-modules`, which "prove[s] the rules bite", `:79-87`) |
| **Enforces by instruction (human gates)** | `tdd` confirms seams (`:22`); `grilling` confirms shared understanding (`:28`); `to-tickets` needs breakdown approval (`:56`); `setup` confirms its drafts (`:63-70`); `triage` waits for direction (`:72`); `diagnosing-bugs` shows hypotheses, non-blocking (`:98`); `wizard` confirms its stage list (`:23`) |
| **Leaves to the user** | Choosing and sequencing skills (the human is the index; `ask-matt` only advises). Closing tickets and ticking acceptance criteria: `implement` never does (`docs/engineering/implement.md:53`). Verifying review findings (`docs/engineering/code-review.md:68`). Creating tracker labels (#616). Deciding whether TDD or diagnosis is worth it for a change (#746, #578). Re-running setup after updates (`docs/engineering/setup-matt-pocock-skills.md:59`) |
| **Leaves to the model** | Exploration strategy ("explore organically", `improve-codebase-architecture/SKILL.md:27`), frontier computation in grilling, test design within agreed seams, the content of every artifact |

---

## 7. Verdicts

**1. LENGTH.** Mostly not a problem.
- Budget: 0 bodies over 500 lines, median 534 words, and 2,150 chars of always-on description, well under OA's 8,000-char reference.
- Per-session load: about 2-4k tokens per flow (section 2). The plugin uses user-invocation to keep 16 of 27 descriptions out of context entirely, an explicit cost model (`SKILL-MECHANICS.md:9-12`).
- The real issues are local. Two single-file skills over 1,800 words (`wayfinder`, `ask-matt`) and a dense 1,757-word style guide. Key rules land mid-file (`wayfinder:105`, `diagnosing-bugs:66`). Four references over 100 lines have no TOC.
- A secondary length cost is *output*: `to-spec` demands "extremely extensive" user stories, which later sessions must then read.

**2. "DUMBING DOWN".** Low.
- No rationalisation tables, threats or "you cannot think your way out" lines anywhere (counter = 0).
- The maintainer argues against forcing: "forcing the point harder restricts the agent's creativity for little gain" (`docs/engineering/tdd.md:59`). The skills' main lever is pretrained "leading words" rather than step scripts (`writing-for-agents/SKILL.md:63`).
- Residual: exhortation and a textbook technique list in `diagnosing-bugs` (`:22,24-35`), testability basics in `codebase-design` (`:67-95`), and "Never trust your parametric knowledge" (`teach:30`). These are content bloat more than condescension.

**3. OUTDATED TECHNIQUES.** Low on shouting, moderate on prohibitions.
- 8 emphatic caps tokens, 0 IMPORTANT, 0 capitalised ALWAYS/NEVER/MUST. Emphasis is carried by bold leading words.
- But 110 prohibition tokens, banned-word lists, and a six-bullet "Don't" list (`prototype/LOGIC.md:60-67`) contradict the maintainer's own Negation rule (`writing-for-agents/SKILL.md:74`), which matches OA:48-49.
- The `to-spec`/`to-tickets` pair carries 6 of the 8 caps tokens and looks like an older layer that has not been revised to the house style.

**4. BUREAUCRACY / OVER-CONSTRAINT.** The main weakness.
- The skills define ceremony that does not scale with task size:
  - an "absolute" human gate before any test (`tdd:22`);
  - a 6-phase gated diagnosis that a broad description fires on any "broken" report (`diagnosing-bugs:3,8`);
  - interviews with no cap, recommended for "every" change (`README.md:103`, `grilling:6,28`);
  - exhaustive specs (`to-spec:41`);
  - fix-all-review-findings loops (`implement-spec:36`).
- The maintainer acknowledges each in the docs but has not fixed them (#746, #607, #578 open). CC:105-111 and OA:16-18 argue directly against this.
- Fairness: the architecture limits the damage. 16 of 27 shipped skills are user-invoked, there is no bootstrap hook and no meta-skill forcing skill use, and the router reserves spec/tickets for multi-session work (`ask-matt:22-26`). So ceremony mostly appears when the human types it. The exceptions are `tdd` and `diagnosing-bugs`, where ceremony can arrive uninvited.

**README claims tested**
- **"small"**: largely true. Median 534 words and 6 promoted skills of 12 lines or fewer, but 5 promoted bodies exceed 1,000 words and the promoted set totals about 30.5k words including references.
- **"easy to adapt"**: partly. skills.sh copies editable files, but there are 17 Skill-tool dependency edges ("you must install them too", `CHANGELOG.md:241`), and `npx skills update` overwrites local edits (`docs/engineering/code-review.md:52`).
- **"composable"**: true at the primitive level (`grilling` is reused by 5 skills). But user-invoked skills cannot compose (`.agents/invocation.md:8`), so the main flow is sequenced by hand, and the docs concede "a skill that names another skill does not reliably cause that skill to load" (`docs/productivity/grilling.md:73`).
- **"work with any model"**: unsupported, and contradicted by the repo's own docs. There are no evals and no model matrix (AS:134-144, 1164-1169), while the docs record:
  - weaker models breaking the grilling gate (`docs/productivity/grilling.md:62`);
  - over-firing on GPT-5.6-Sol: "calibrated against Claude Code's invocation behaviour" (`docs/engineering/diagnosing-bugs.md:59`);
  - "weaker models skip straight to interviewing" (`docs/engineering/improve-codebase-architecture.md:56`);
  - Codex producing "a single 30-line HTML card" (`docs/productivity/teach.md:80`);
  - to-tickets "worse on Codex" (`docs/engineering/to-tickets.md:65`);
  - "Does the model matter? More than for most skills." (`docs/productivity/grill-me.md:67-68`).
- **vs GSD/BMAD/Spec-Kit "owning the process"**: fair as a statement about mechanism (no hooks, no forced bootstrap, human-sequenced). Less fair as a statement about intent: `ask-matt` defines a full process (grill, spec, tickets, implement, review, retro), and the README recommends grilling before every change. It is process as advice, not process as enforcement.

---

## 8. Open questions

1. Do any private or off-repo evals exist? `writing-for-agents/SKILL.md:81` says to "settle it by running the document", but nothing on disk records such runs, models or results.
2. The "~150k tokens" smart-zone figure (`ask-matt/SKILL.md:38`, `PHASE-BOUNDARIES.md:21`) has no source in the repo.
3. Does every Claude Code surface actually keep user-invoked descriptions out of model context? Issue #693, cited in `docs/engineering/wizard.md:77`, says desktop/web drop them from the listing entirely. This affects the 2,150-char estimate and whether those skills work there at all.
4. Did switching to "Call the Skill tool with X" raise load reliability? It "is intended to raise the hit rate" (`.changeset/skill-tool-invocation-terminology.md:7`), but no measurement is given, and `docs/productivity/grilling.md:73` still calls it "a real and unfixed rough edge".
5. Codex's description budget for this set: the 2%/8,000-char figure is from a secondary, reconstructed source.
6. The current state of issues #746, #607, #578, #530, #589, #674, #616 and #693: known only from the docs pages, and I cannot browse. The docs also describe some unreleased changes (`docs/engineering/research.md:61`), and the release/v1.3 merge (version still 1.2.3, 13 pending changesets) may shift the numbers. The clone is shallow, so I could not check history.
7. Whether "leading words recruit priors" (`writing-for-agents/SKILL.md:63`) measurably changes behaviour versus plain instructions. It is asserted, not tested.
8. How the `code-review` name collision with Claude Code's built-in `/code-review` resolves today (`docs/engineering/code-review.md:52`).

---

## Appendix: scripts (re-runnable)

Run from the scratchpad directory (Python 3.11 + PyYAML):

```bash
python3 scripts/mattpocock_metrics.py repos/mattpocock-skills                 # all tables in sections 1-2
python3 scripts/mattpocock_metrics.py repos/mattpocock-skills --json > scripts/mattpocock_metrics.json
python3 scripts/mattpocock_table.py scripts/mattpocock_metrics.json             # inventory table with A-J scores
```

Raw output of the first command is saved at `scripts/mattpocock_metrics.out`.

Method notes:
- Body = everything after the YAML frontmatter.
- Emphasis, prohibition and CAPS counts run on prose with fenced code, inline code, link targets, URLs and file names removed, and with bold/italic markers stripped. This keeps `Do **not**` countable and stops `SKILL.md` from counting as caps.
- The CAPS acronym allowlist is in the script, and the per-skill token lists are printed so they can be audited.
- Reference depth = BFS from SKILL.md over mentions of sibling files' relative paths (markdown links or plain mentions).
- Cross-checks done by hand with grep:
  - wayfinder has 9 always/never/must by grep vs 8 by the script; the extra one is inside a fenced block (`wayfinder/SKILL.md:52`);
  - a raw `grep -rhoE '\b[A-Z]{3,}\b'` over `skills/` shows the only emphatic caps outside templates are the 8 listed in 4.3.

Reconciliation with `reports/_metrics-mattpocock.txt` (a quick pass not produced by me):
- **lines**: identical for all 37 skills.
- **words**: differ by at most about 2% (different tokenisation; mine is `str.split()` on the body).
- **desc chars**: mine is the parsed YAML string. Theirs counts raw escapes, so `code-review` is 417 here and 419 there (the `\"` escapes).
- **CAPS**: theirs counts every raw caps token, including file names and acronyms. For example, `teach` 14 comes from `MISSION`, `RESOURCES`, `FORMAT`, `HTML`, `NOTES`, `CLI`; `triage` 8 from `AGENT`, `BRIEF`, `OUT`, `OF`, `SCOPE`, `PR`. Mine counts emphatic tokens only.
- **ANM**: theirs counts the CAPS forms (0 everywhere), which matches my statement that the CAPS forms are absent. My A/N/M column is any case.
- **refs**: theirs includes SKILL.md itself (n+1); the `md` column matches mine exactly.

### `scripts/mattpocock_metrics.py`

```python
#!/usr/bin/env python3
"""Metrics for mattpocock/skills.  Usage: python3 mattpocock_metrics.py <repo_root> [--json]
Counts are computed on prose with fenced code blocks and inline `code` removed
(so example code and file names do not inflate emphasis counts), except
body_lines/body_words which are raw (everything after the YAML frontmatter)."""
import os, re, sys, json, yaml
from collections import deque, OrderedDict

ROOT = os.path.abspath(sys.argv[1])
SK = os.path.join(ROOT, "skills")
PROMOTED = {"engineering", "productivity"}
plugin = json.load(open(os.path.join(ROOT, ".claude-plugin/plugin.json")))
plugin_set = {os.path.normpath(p) for p in plugin["skills"]}

# ALL-CAPS tokens that are acronyms / proper names rather than emphasis.
ACRONYMS = set("""ADR ADRS API APIS HTML CSS JS JSON YAML URL URLS UI UX CLI CLIS PR PRS MR MRS TDD AFK HITL
CI OS DOM SDK ID IDS HTTP REST SQL ORM AWS DDD IIFE MCP MCPS TOC WSL RNG REPL HAR PM LR NNNN TS ESM SVG CDN
CDNS CRUD GLM GPT AI YAGNI TODO TODOS RPE PDF OK SKILL README CLAUDE AGENTS GLOSSARY MISSION RESOURCES NOTES
CONTEXT GIT LLM LLMS SHA HEAD IDE JWT DB QA KB ASD STE NN XX YY GH MD DX SRI CDN PNPM NPM UTC EOF""".split())
EMPH_WORDS = ["ALWAYS", "NEVER", "MUST", "IMPORTANT", "CRITICAL", "NOT", "ONLY", "STOP", "NO", "DON'T", "DO"]

def split_fm(text):
    m = re.match(r"^---\n(.*?)\n---\n", text, re.S)
    return yaml.safe_load(m.group(1)), text[m.end():]

def prose(text):
    out, fence = [], False
    for line in text.split("\n"):
        if re.match(r"^\s*(```|~~~)", line):
            fence = not fence; continue
        if fence: continue
        line = re.sub(r"`[^`]*`", " ", line)                      # inline code
        line = re.sub(r"\]\([^)]*\)", "]", line)                  # link targets
        line = re.sub(r"https?://\S+", " ", line)                 # bare urls
        line = re.sub(r"[\w.-]+\.(md|sh|json|yaml|cjs|ts|tsx|html)\b", " ", line)  # file names
        line = re.sub(r"</?[a-z-]+>", " ", line)                  # <template-tags>
        out.append(line)
    return "\n".join(out)

def caps_tokens(p):
    toks = re.findall(r"(?<![\w-])[A-Z]{2,}(?:['’][A-Z]+)?(?![\w-])", p)
    return [t for t in toks if t.replace("’", "'") not in ACRONYMS]

def counts(text):
    p0 = prose(text)
    nbold = len(re.findall(r"\*\*[^*\n]+\*\*", p0))
    p = re.sub(r"(?<!\w)_([^_\n]+)_(?!\w)", r"\1", p0.replace("**", ""))   # drop bold/italic markers
    caps = caps_tokens(p)
    return OrderedDict(
        caps=len(caps), caps_list=caps,
        anm_any=len(re.findall(r"\b(always|never|must)\b", p, re.I)),
        anm_caps=len(re.findall(r"\b(ALWAYS|NEVER|MUST)\b", p)),
        important=len(re.findall(r"\b(IMPORTANT|CRITICAL)\b", p)),
        prohib=len(re.findall(r"\b(don['’]t|do not|never|avoid)\b", p, re.I)),
        bold=nbold,
        redflag=len(re.findall(r"rationali[sz]|red flag|excuse|no exceptions|non-negotiable|you cannot think", p, re.I)),
        done=len(re.findall(r"done when|completion criterion|definition of done|it'?s working if|verify|verif", p, re.I)),
        gates=len(re.findall(r"\b(confirm|approve[sd]?|wait for|ask the user|check with the user|stop and)\b", p, re.I)),
        emdash=text.count("—"),
    )

def words(s): return len(s.split())

def ref_graph(sdir):
    """BFS from SKILL.md over mentions of sibling files' relative paths."""
    files = []
    for dp, _, fns in os.walk(sdir):
        for fn in fns:
            rel = os.path.relpath(os.path.join(dp, fn), sdir)
            if rel == "SKILL.md" or rel.startswith("agents" + os.sep): continue
            files.append(rel)
    texts = {f: open(os.path.join(sdir, f), errors="ignore").read() for f in files + ["SKILL.md"]}
    def mentions(src, tgt):
        t = texts[src]
        pat = re.escape(tgt) if "/" in tgt else r"(?<![\w-])" + re.escape(tgt) + r"(?![\w-])"
        return re.search(pat, t) is not None
    depth = {"SKILL.md": 0}; q = deque(["SKILL.md"])
    while q:
        cur = q.popleft()
        for f in files:
            if f not in depth and mentions(cur, f):
                depth[f] = depth[cur] + 1; q.append(f)
    reach = [d for f, d in depth.items() if f != "SKILL.md"]
    orphans = [f for f in files if f not in depth]
    md = [f for f in files if f.endswith(".md")]
    return dict(files=files, md=md, depth=depth, maxdepth=max(reach) if reach else 0, orphans=orphans,
                ref_words=sum(words(texts[f]) for f in md),
                other_words=sum(words(texts[f]) for f in files if not f.endswith(".md")),
                ref_lines={f: texts[f].count("\n") for f in md})

def skill_deps(body, known):
    deps = []
    for m in re.finditer(r"Skill tool[^.\n]*", body):
        deps += [n for n in re.findall(r"[\"`]([a-z0-9-]+)[\"`]", m.group(0)) if n in known]
    bare = [n for n in re.findall(r"(?<![\w/.])/([a-z][a-z0-9-]+)", body) if n in known]
    return sorted(set(deps)), sorted(set(bare))

rows = []
known = set()
for b in sorted(os.listdir(SK)):
    bd = os.path.join(SK, b)
    if os.path.isdir(bd):
        known |= {s for s in os.listdir(bd) if os.path.isfile(os.path.join(bd, s, "SKILL.md"))}

for b in sorted(os.listdir(SK)):
    bd = os.path.join(SK, b)
    if not os.path.isdir(bd): continue
    for s in sorted(os.listdir(bd)):
        sdir = os.path.join(bd, s); sm = os.path.join(sdir, "SKILL.md")
        if not os.path.isfile(sm): continue
        raw = open(sm).read(); fm, body = split_fm(raw)
        oy = os.path.join(sdir, "agents", "openai.yaml")
        oyd = yaml.safe_load(open(oy)) if os.path.isfile(oy) else {}
        dmi = bool(fm.get("disable-model-invocation"))
        implicit = (oyd.get("policy") or {}).get("allow_implicit_invocation", True)
        desc = fm.get("description", "")
        g = ref_graph(sdir); c = counts(body)
        sk_deps, bare = skill_deps(body, known - {s})
        rows.append(OrderedDict(
            bucket=b, name=fm["name"], dir=s, promoted=b in PROMOTED,
            in_plugin=os.path.normpath("./skills/%s/%s" % (b, s)) in plugin_set,
            invocation="user" if dmi else "model+user",
            codex_consistent=(dmi == (implicit is False)),
            body_lines=body.count("\n"), body_words=words(body), file_words=words(raw),
            desc_chars=len(desc), name_chars=len(fm["name"]),
            desc_use_when=bool(re.search(r"\buse when\b", desc, re.I)),
            desc_pronouns=re.findall(r"\b(you|your|I|me|my)\b", desc),
            n_refs=len(g["files"]), n_md_refs=len(g["md"]), ref_depth=g["maxdepth"],
            orphans=g["orphans"], ref_words=g["ref_words"], other_words=g["other_words"],
            ref_lines=g["ref_lines"], skill_tool_deps=sk_deps, bare_slash=bare, **c))

def agg(rs, k): return sum(r[k] for r in rs)
args = sys.argv[2:]
if "--json" in args:
    print(json.dumps(rows, indent=1)); sys.exit()

print("# Inventory (body = SKILL.md after frontmatter; counts exclude fenced/inline code)")
hdr = "bucket|name|inv|codexOK|plugin|lines|words|desc|useWhen|pron|refs(md)|depth|orphans|refWords|CAPS|A/N/M any|A/N/M CAPS|IMPORTANT|prohib|bold|redflag|done/verif|gates|emdash|SkillTool deps|bare /mentions"
print(hdr)
for r in rows:
    print("|".join(str(x) for x in [r["bucket"], r["name"], r["invocation"], r["codex_consistent"], r["in_plugin"],
        r["body_lines"], r["body_words"], r["desc_chars"], r["desc_use_when"], ",".join(r["desc_pronouns"]) or "-",
        "%d(%d)" % (r["n_refs"], r["n_md_refs"]), r["ref_depth"], ",".join(r["orphans"]) or "-", r["ref_words"],
        r["caps"], r["anm_any"], r["anm_caps"], r["important"], r["prohib"], r["bold"], r["redflag"], r["done"],
        r["gates"], r["emdash"], ",".join(r["skill_tool_deps"]) or "-", ",".join(r["bare_slash"]) or "-"]))

print("\n# CAPS tokens per skill (emphasis candidates, acronyms removed)")
for r in rows:
    if r["caps_list"]: print(r["name"], r["caps_list"])

print("\n# Reference file line counts (>100 lines should have a TOC per Anthropic guidance)")
for r in rows:
    for f, n in r["ref_lines"].items():
        print("%s/%s %d" % (r["name"], f, n))

print("\n# Reference .md files: same counters (CAPS, A/N/M any, prohib, bold, redflag)")
for r in rows:
    b = r["bucket"]
    for f in r["ref_lines"]:
        c = counts(open(os.path.join(SK, b, r["dir"], f)).read())
        print("%s/%s|caps=%d %s|anm=%d|prohib=%d|bold=%d|redflag=%d" % (r["name"], f, c["caps"], c["caps_list"], c["anm_any"], c["prohib"], c["bold"], c["redflag"]))

print("\n# Totals by bucket")
print("bucket|skills|user|model|SKILL.md words(file)|SKILL.md body words|ref .md words|other bundled words|max body lines")
for b in sorted({r["bucket"] for r in rows}):
    rs = [r for r in rows if r["bucket"] == b]
    print("|".join(str(x) for x in [b, len(rs), sum(r["invocation"] == "user" for r in rs),
          sum(r["invocation"] != "user" for r in rs), agg(rs, "file_words"), agg(rs, "body_words"),
          agg(rs, "ref_words"), agg(rs, "other_words"), max(r["body_lines"] for r in rs)]))
for label, rs in [("PROMOTED", [r for r in rows if r["promoted"]]), ("ALL", rows)]:
    print("%s|%d|%d|%d|%d|%d|%d|%d|%d" % (label, len(rs), sum(r["invocation"] == "user" for r in rs),
          sum(r["invocation"] != "user" for r in rs), agg(rs, "file_words"), agg(rs, "body_words"),
          agg(rs, "ref_words"), agg(rs, "other_words"), max(r["body_lines"] for r in rs)))

print("\n# Always-loaded metadata (name + description) for model-invoked skills")
for label, rs in [("PROMOTED", [r for r in rows if r["promoted"]]), ("ALL", rows)]:
    mi = [r for r in rs if r["invocation"] != "user"]
    print("%s: %d model-invoked, desc chars=%d, name+desc chars=%d; all-skill desc chars=%d; mean desc=%.0f, max desc=%d"
          % (label, len(mi), agg(mi, "desc_chars"), agg(mi, "desc_chars") + agg(mi, "name_chars"),
             agg(rs, "desc_chars"), agg(rs, "desc_chars") / len(rs), max(r["desc_chars"] for r in rs)))

print("\n# Consistency checks")
print("promoted skills missing from plugin.json:", [r["name"] for r in rows if r["promoted"] and not r["in_plugin"]])
print("non-promoted skills in plugin.json:", [r["name"] for r in rows if not r["promoted"] and r["in_plugin"]])
print("claude/codex invocation mismatch:", [r["name"] for r in rows if not r["codex_consistent"]])
print("dir name != frontmatter name:", [r["dir"] for r in rows if r["dir"] != r["name"]])
print("bodies over 500 lines:", [r["name"] for r in rows if r["body_lines"] > 500])
readme = open(os.path.join(ROOT, "README.md")).read()
print("promoted skills not linked in README:", [r["name"] for r in rows if r["promoted"] and ("/%s/SKILL.md" % r["dir"]) not in readme])

# Words loaded for representative flows (SKILL.md files incl. frontmatter + named reference files)
W = {r["name"]: r["file_words"] for r in rows}
def refw(skill, f):
    b = [r for r in rows if r["name"] == skill][0]["bucket"]
    return words(open(os.path.join(SK, b, skill, f)).read())
flows = OrderedDict([
  ("feature, planning session (grill-with-docs -> to-spec -> to-tickets, one context)",
     [("grill-with-docs", None), ("grilling", None), ("domain-modeling", None), ("domain-modeling", "GLOSSARY-FORMAT.md"),
      ("domain-modeling", "ADR-FORMAT.md"), ("to-spec", None), ("to-tickets", None)]),
  ("feature, per-ticket session (implement -> tdd -> code-review)",
     [("implement", None), ("tdd", None), ("tdd", "tests.md"), ("tdd", "mocking.md"), ("code-review", None)]),
  ("feature, + optional codebase-design pulled by tdd", [("codebase-design", None)]),
  ("bug, model-invoked diagnosing-bugs only", [("diagnosing-bugs", None)]),
  ("setup (skill + all seed templates it may read)",
     [("setup-matt-pocock-skills", None)] + [("setup-matt-pocock-skills", f) for f in
      ["domain.md", "issue-tracker-github.md", "issue-tracker-gitlab.md", "issue-tracker-local.md", "triage-labels.md"]]),
  ("router (ask-matt + PHASE-BOUNDARIES.md)", [("ask-matt", None), ("ask-matt", "PHASE-BOUNDARIES.md")]),
])
print("\n# Words loaded per flow (approx tokens = words*1.33)")
for k, items in flows.items():
    n = sum(W[s] if f is None else refw(s, f) for s, f in items)
    print("%s: %d words (~%d tokens)" % (k, n, n * 1.33))

# Prose-level checks across the repo
def count_in(globroot, pat, exts=(".md",)):
    n = 0
    for dp, _, fns in os.walk(globroot):
        if ".git" in dp or "node_modules" in dp: continue
        for fn in fns:
            if fn.endswith(exts): n += len(re.findall(pat, open(os.path.join(dp, fn), errors="ignore").read()))
    return n
print("\n# Em-dashes (CLAUDE.md:25 bans them)")
for p in ["skills", "docs", ".agents", ".changeset"]:
    print(p, count_in(os.path.join(ROOT, p), "—"))
for f in ["README.md", "CLAUDE.md", "CHANGELOG.md"]:
    print(f, open(os.path.join(ROOT, f)).read().count("—"))
print("\n# Behaviour evals / tests present?")
hits = [os.path.relpath(os.path.join(dp, fn), ROOT) for dp, _, fns in os.walk(ROOT) if ".git" not in dp and "node_modules" not in dp
        for fn in fns if re.search(r"(eval|test|spec)", fn, re.I) and not fn.endswith(".md")]
print("files named *eval*/*test*/*spec* (excl .md):", hits)
pj = json.load(open(os.path.join(ROOT, "package.json")))
print("package.json scripts:", pj.get("scripts"))
```

### `scripts/mattpocock_table.py`

```python
#!/usr/bin/env python3
"""Merge script metrics (mattpocock_metrics.json) with the analyst's A-J scores into a markdown table."""
import json, sys
rows = json.load(open(sys.argv[1]))
# A Concise, B Description, C Freedom, D Disclosure, E Done/verify, F Scaffolding, G Proportionality,
# H Evals, I Consistency, J Portability   (analyst judgement, 1-5, 5 = aligned with guidance)
S = {
 "ask-matt":                      [3,4,5,4,3,4,4,1,3,3],
 "code-review":                   [4,3,5,4,4,5,3,2,4,3],
 "codebase-design":               [3,3,5,5,3,4,4,2,4,4],
 "diagnosing-bugs":               [3,2,3,3,5,4,2,2,3,4],
 "domain-modeling":               [4,4,5,5,3,4,5,2,4,5],
 "grill-with-docs":               [5,4,4,5,3,5,3,2,5,3],
 "implement":                     [5,4,4,4,3,5,3,2,2,4],
 "implement-spec":                [4,4,4,4,4,5,4,1,4,2],
 "improve-codebase-architecture": [3,4,4,3,3,4,3,2,4,2],
 "pr":                            [3,3,4,3,4,5,3,1,3,5],
 "prototype":                     [4,5,4,5,4,3,4,1,4,4],
 "research":                      [5,4,5,5,4,5,3,2,5,2],
 "retro":                         [4,4,5,4,3,4,4,1,4,3],
 "setup-matt-pocock-skills":      [4,4,5,5,3,4,5,1,3,3],
 "tdd":                           [5,3,3,5,4,4,2,2,3,4],
 "to-spec":                       [4,4,3,4,3,3,3,1,4,3],
 "to-tickets":                    [4,3,4,4,4,3,4,2,3,3],
 "triage":                        [3,4,4,3,5,4,4,2,4,3],
 "wayfinder":                     [2,4,3,3,4,3,4,1,4,2],
 "wizard":                        [4,4,5,5,5,4,4,2,5,4],
 "grill-me":                      [5,4,4,5,3,5,3,2,5,3],
 "grilling":                      [5,4,4,5,5,5,2,2,5,4],
 "handoff":                       [5,4,5,5,3,5,5,1,4,3],
 "teach":                         [3,4,3,3,2,4,4,2,3,3],
 "to-questionnaire":              [4,4,5,4,5,5,5,1,4,5],
 "wait-what":                     [5,3,4,5,2,5,5,1,4,5],
 "writing-for-agents":            [3,5,5,4,3,5,4,1,4,5],
 "git-guardrails-claude-code":    [4,4,5,4,5,5,5,1,4,2],
 "migrate-to-shoehorn":           [4,4,4,4,4,4,4,1,3,4],
 "scaffold-exercises":            [4,3,5,4,5,5,4,1,4,2],
 "setup-pre-commit":              [4,4,5,4,5,5,4,1,4,4],
 "claude-handoff":                [5,4,5,5,3,4,5,1,4,1],
 "loop-me":                       [4,3,5,4,4,5,3,1,3,4],
 "setup-ts-deep-modules":         [3,3,5,4,5,4,3,1,4,3],
 "writing-beats":                 [3,3,4,4,3,4,4,1,3,5],
 "writing-fragments":             [4,3,5,4,2,4,4,1,3,5],
 "writing-shape":                 [3,3,4,4,3,4,3,1,3,5],
}
order = ["engineering", "productivity", "misc", "in-progress"]
print("| skill | bucket | inv | lines | words | desc ch | refs (n/depth) | CAPS | A/N/M | A | B | C | D | E | F | G | H | I | J | mean |")
print("|---|---|---|--:|--:|--:|---|--:|--:|--|--|--|--|--|--|--|--|--|--|--:|")
means = {}
for b in order:
    for r in [r for r in rows if r["bucket"] == b]:
        s = S[r["name"]]; m = sum(s) / 10; means.setdefault(b, []).append(m)
        refs = "%d/%d" % (r["n_refs"], r["ref_depth"]) + (" (orphan: %s)" % ",".join(r["orphans"]) if r["orphans"] else "")
        bk = {"engineering": "eng*", "productivity": "prod*", "misc": "misc", "in-progress": "beta"}[b]
        inv = "user" if r["invocation"] == "user" else "model"
        print("| %s | %s | %s | %d | %d | %d | %s | %d | %d | %s | %.1f |" % (r["name"], bk, inv, r["body_lines"], r["body_words"],
              r["desc_chars"], refs, r["caps"], r["anm_any"], " | ".join(map(str, s)), m))
print()
for b in order:
    print("mean score %s: %.2f" % (b, sum(means[b]) / len(means[b])))
cols = "ABCDEFGHIJ"
prom = [S[r["name"]] for r in rows if r["promoted"]]
print("promoted per-criterion means:", {c: round(sum(x[i] for x in prom) / len(prom), 2) for i, c in enumerate(cols)})
```
