# obra/superpowers: analyst report

Paths are relative to the clone (`repos/obra-superpowers/`) unless prefixed `refs/`. Reference shorthand: **AS** = `refs/agent-skills-best-practices.md` (Anthropic, skill authoring), **CC** = `refs/claude-code-best-practices.md` (Anthropic, Claude Code), **OAI** = `refs/openai-gpt6-astra-reconstructed.md` (press coverage of the OpenAI post, so a secondary source). Every number comes from the scripts in the Appendix. Token figures are estimates (chars/4 to chars/3.5) because no tokenizer is available offline.

---

## 1. Facts

| Item | Value |
|---|---|
| Commit | `8ca22dba9a94f28898bbce59f2537ff4d87c747d`, 2026-09-25 11:06 -0700, "Release v6.4.2: leaner plans from writing-plans (#2384)". The clone is shallow (1 commit). |
| Version | 6.4.2 (`.claude-plugin/plugin.json:4`, plus 10 other manifests kept in sync by `.version-bump.json`) |
| Skills | **15** SKILL.md files. README categories (`README.md:343-364`): Testing 1 (test-driven-development), Debugging 3 (systematic-debugging, verification-before-completion, diagnosing-superpowers), Collaboration 9 (brainstorming, writing-plans, executing-plans, dispatching-parallel-agents, requesting-code-review, receiving-code-review, using-git-worktrees, finishing-a-development-branch, subagent-driven-development), Meta 2 (writing-skills, using-superpowers) |
| SKILL.md size | 3,828 body lines and **25,371 body words** (25,796 with frontmatter; `wc -w` reports 25,414). 2 skills exceed 500 lines: subagent-driven-development (564) and writing-skills (677). |
| Reference files | **45 markdown files, 27,802 words**, plus 14 scripts/assets (9,214 words). 15 markdown files are over 100 lines with no table of contents. One of them is `writing-skills/anthropic-best-practices.md` (1,150 lines), a vendored copy of AS with "Claude" replaced by "agent". |
| Unreferenced files shipped inside skills | `brainstorming/spec-document-reviewer-prompt.md`; `systematic-debugging/{CREATION-LOG,test-academic,test-pressure-1..3}.md`; `using-superpowers/references/gemini-tools.md`. That last file is loaded only through `GEMINI.md:2`. |
| Hooks | `hooks/hooks.json:3-15` registers SessionStart with matcher `startup\|clear\|compact`, which runs `hooks/run-hook.cmd session-start` (a polyglot Windows/Unix wrapper). `hooks/hooks-cursor.json` is the Cursor variant. Muse runs a SessionStart hook (`.muse-plugin/plugin.json:75-85`). Hermes uses `pre_llm_call` on the first turn (`.hermes-plugin/__init__.py:91-104`). Pi uses `session_start` plus `session_compact` (`.pi/extensions/superpowers.ts:23-56`). OpenCode transforms messages (`.opencode/plugins/superpowers.js:235-263, 337-360`). These are the only hooks. **No Stop or PreToolUse hook enforces anything**; all enforcement is prose. |
| Manifests | `.claude-plugin/` (plugin + marketplace), `.codex-plugin/` (`"hooks": {}`), `.agents/plugins/marketplace.json` (Codex marketplace), `.cursor-plugin/`, `.devin-plugin/`, `.kimi-plugin/`, `.muse-plugin/` (plugin + marketplace), `.hermes-plugin/`, `.opencode/`, `.pi/` (via `package.json:15-22`), `gemini-extension.json` + `GEMINI.md` |
| Harnesses | 16 install sections (`README.md:10-25`): Claude Code, Antigravity, Codex App, Codex CLI, Cursor, Devin, Factory Droid, Gemini, Copilot CLI, Grok Build, Kimi, OpenCode, Pi, Qwen, Hermes, Muse |
| How skills trigger | (1) A **bootstrap**: the full `using-superpowers/SKILL.md` is injected every session and tells the model to invoke any skill with a "1% chance" of applying (`skills/using-superpowers/SKILL.md:11`). (2) Each skill's description. (3) The user naming a skill. No skill sets `disable-model-invocation`, so every skill is model- and user-invocable. **Codex and Devin get no bootstrap** and rely on descriptions only (`.codex-plugin/plugin.json:24`; `RELEASE-NOTES.md:173`; `tests/devin/test-devin-plugin.sh:2-8`). |
| Tests in repo | 54 test files. Most cover plugin plumbing (brainstorm server, manifests, OpenCode loading, version bump). 7 scripts drive a real LLM, all through Claude Code `claude -p`: triggering tests (`tests/explicit-skill-requests/*`, including one Haiku runner), an SDD recall quiz plus an integration run, and a worktree RED/GREEN test that claims "Validated: 50/50 runs" (`tests/claude-code/test-worktree-native-preference.sh:19`). |
| Evals | Skill-behaviour evals live in the external **superpowers-evals** repo ("Quorum"), which is **not cloned** here (`docs/testing.md:25-37`; `AGENTS.md:102-104`). Some eval write-ups are in `docs/superpowers/specs/*` and `RELEASE-NOTES.md`. |

### What lands in context on every session

On Claude Code, `hooks/session-start:11,27` cats the whole SKILL.md, frontmatter included, and wraps it in `<EXTREMELY_IMPORTANT>You have superpowers. … </EXTREMELY_IMPORTANT>`. Inside that wrapper is the skill's own `<EXTREMELY-IMPORTANT>` block, so the text is double-wrapped. It is re-injected after every `/clear` and every compaction (`hooks/hooks.json:5`).

| Harness | Injected text (chars / words) | Est. tokens | When |
|---|---|---|---|
| Claude Code, Cursor, Copilot, Muse, Antigravity | 3,405 / 520 | 850-970 | startup, clear, compact |
| OpenCode V1 / V2 | 3,737 / 579 and 4,155 / 642 | 930-1,190 | first user message; survives compaction |
| Pi | 4,367 / 676 | 1,090-1,250 | start + after compaction |
| Hermes | 5,677 / 828 | 1,420-1,620 | first turn only |
| Gemini (`GEMINI.md` includes SKILL.md + gemini-tools.md) | 7,774 / 1,110 | 1,940-2,220 | every session |
| Kimi (skill body + `skillInstructions`) | 4,941 / 762 | 1,240-1,410 | session start |
| Codex, Devin | 0 (no bootstrap) | 0 | n/a |
| Skill index (15 names + descriptions, every harness) | 2,689 / 380 | 670-770 | always |

Codex budget (OAI:23-27): 15 descriptions total 2,302 chars, or 2,615 including names. That is about **33% of the 8,000-character fallback** and about 16% of 2% of a 200k window. Longest descriptions: diagnosing-superpowers 375, receiving-code-review 234, verification-before-completion 225. Truncation is unlikely when superpowers is the only plugin, but it takes a third of the fallback budget away from everything else installed.

---

## 2. Inventory

Columns: inv = invocation; lines/words = SKILL.md body without frontmatter; desc = description chars; refs = markdown files in the skill directory / maximum link depth from SKILL.md; CAPS = ALL-CAPS words in prose (code blocks, inline code, filenames and acronyms excluded); A/N/M = case-insensitive count of always/never/must in prose. Scores A-J are my judgement (5 = aligned with the guidance).

| Skill | inv | lines | words | desc | refs | CAPS | A/N/M | A | B | C | D | E | F | G | H | I | J |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| brainstorming | both | 281 | 2,582 | 198 | 2 / 1 | 15 | 3 | 2 | 2 | 2 | 3 | 3 | 2 | 2 | 3 | 3 | 3 |
| diagnosing-superpowers | both | 116 | 991 | 375 | 19 / 2 | 0 | 7 | 4 | 3 | 4 | 4 | 5 | 4 | 3 | 2 | 4 | 4 |
| dispatching-parallel-agents | both | 163 | 843 | 106 | 0 / 0 | 2 | 1 | 3 | 4 | 4 | 4 | 3 | 5 | 4 | 1 | 4 | 3 |
| executing-plans | both | 369 | 3,235 | 170 | 0 / 0 (+2 scripts, reuses SDD's) | 6 | 9 | 2 | 4 | 3 | 3 | 5 | 3 | 3 | 3 | 2 | 4 |
| finishing-a-development-branch | both | 221 | 1,246 | 101 | 0 / 0 | 0 | 3 | 3 | 4 | 4 | 4 | 4 | 4 | 3 | 2 | 4 | 5 |
| receiving-code-review | both | 201 | 879 | 234 | 0 / 0 | 5 | 1 | 3 | 3 | 2 | 4 | 3 | 2 | 3 | 1 | 3 | 3 |
| requesting-code-review | both | 91 | 404 | 107 | 1 / 1 | 0 | 3 | 4 | 4 | 3 | 5 | 4 | 3 | 2 | 2 | 2 | 4 |
| subagent-driven-development | both | **564** | **4,854** | 85 | 3 / 1 (+3 scripts) | 6 | 27 | 1 | 4 | 3 | 3 | 5 | 3 | 3 | 4 | 4 | 3 |
| systematic-debugging | both | 279 | 1,422 | 91 | 8 / 1 (5 unreferenced) | 28 | 3 | 2 | 3 | 2 | 3 | 3 | 1 | 1 | 3 | 3 | 4 |
| test-driven-development | both | 326 | 1,459 | 79 | 1 / 1 | 4 | 7 | 2 | 3 | 1 | 4 | 4 | 1 | 1 | 4 | 4 | 4 |
| using-git-worktrees | both | 163 | 1,035 | 196 | 0 / 0 | 3 | 3 | 3 | 4 | 4 | 4 | 4 | 4 | 3 | 4 | 4 | 3 |
| using-superpowers (bootstrap) | hook + both | 61 | 465 | 154 | 7 / 1 | 23 | 2 | 2 | 2 | 1 | 4 | 2 | 1 | 1 | 2 | 3 | 4 |
| verification-before-completion | both | 116 | 542 | 225 | 0 / 0 | 8 | 2 | 3 | 3 | 3 | 4 | 5 | 2 | 3 | 2 | 4 | 5 |
| writing-plans | both | 200 | 1,619 | 84 | 0 / 0 | 5 | 2 | 3 | 4 | 3 | 3 | 4 | 4 | 3 | 4 | 4 | 4 |
| writing-skills | both | **677** | 3,795 | 97 | 4 / 2 | 48 | 13 | 1 | 4 | 2 | 3 | 3 | 1 | 1 | 4 | 2 | 4 |
| **Total / mean** | | 3,828 | 25,371 | 2,302 | 45 | 153 | 86 | 2.5 | 3.4 | 2.7 | 3.7 | 3.8 | 2.7 | 2.4 | 2.7 | 3.3 | 3.8 |

Scaffolding counters (SKILL.md prose; script `skill_metrics.py` plus `rationalization_rows.py`):

| Skill | IMPORTANT | STOP | XML-style tags | "human partner" | Excuse/Thought→Reality rows | Red-Flag bullets | "No exceptions" |
|---|---|---|---|---|---|---|---|
| using-superpowers | 2 | 3 | 2 (`<SUBAGENT-STOP>`, `<EXTREMELY-IMPORTANT>`) | 1 | 12 | 0 | 0 |
| brainstorming | 0 | 2 | 1 (`<HARD-GATE>`) | 8 | 7 | 0 | 0 |
| test-driven-development | 0 | 1 | 4 (`<Good>`/`<Bad>`) | 3 | 11 | 13 | 2 |
| systematic-debugging | 0 | 6 | 0 | 2 | 8 | 11 | 0 |
| verification-before-completion | 0 | 1 | 0 | 0 | 8 | 8 | 1 |
| writing-skills | 1 | 2 | 2 | 2 | 8 | 0 | 3 |
| executing-plans | 0 | 0 | 0 | 7 | 12 | 0 | 0 |
| finishing-a-development-branch | 0 | 0 | 0 | 9 | 10 | 0 | 0 |
| subagent-driven-development | 0 | 0 | 0 | 4 | 9 | 0 | 0 |
| diagnosing-superpowers | 0 | 0 | 0 | 1 | 6 | 0 | 0 |
| using-git-worktrees | 0 | 0 | 0 | 1 | 5 | 0 | 0 |
| requesting-code-review | 0 | 0 | 0 | 0 | 2 | 7 | 0 |
| receiving-code-review | 0 | 0 | 0 | 9 | 0 | 0 | 0 |
| writing-plans, dispatching-parallel-agents | 0 | 0 | 0 | 1 / 0 | 0 | 0 | 0 |
| **Total** | 3 | 15 | 9 | 48 (73 across all of `skills/`) | **98 rows in 12/15 skills** | 39 | 6 |

Other collection facts:
- 14 of 15 descriptions start "Use when". The exception is brainstorming: "You MUST use this before any creative work" (`skills/brainstorming/SKILL.md:3`).
- 13 of 15 bodies exceed the repo's own "<500 words" target for ordinary skills (`skills/writing-skills/SKILL.md:217-220`). The bootstrap is 465 words against its own "<150 words" target for getting-started workflows.
- There are 4 "Iron Law" ALL-CAPS code blocks: `test-driven-development:34`, `systematic-debugging:17`, `verification-before-completion:17` and `writing-skills:379`.

---

## 3. Architecture and philosophy

### In the maintainer's words
- "Mandatory workflows, not suggestions." (`README.md:323`)
- The original plan reader is "an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing" (`README.md:42`). v6.4.2 replaced this framing inside the skill (`RELEASE-NOTES.md:11`), but the README still says it.
- "Our internal skill philosophy differs from Anthropic's published guidance… We have extensively tested and tuned our skill content for real-world agent behavior." (`AGENTS.md:42`)
- "Do not modify carefully-tuned content (Red Flags tables, rationalization lists, 'human partner' language) without evidence" (`AGENTS.md:100`). Also: "'your human partner' is deliberate, not interchangeable with 'the user'" (`AGENTS.md:108`).
- On the bootstrap: "Without it, the skills are dead weight — present on disk but never invoked." (`AGENTS.md:76`). Contrast "Codex reliably triggers skills on its own, and the bootstrap hook made the UX worse rather than better." (`RELEASE-NOTES.md:173`)
- On skill writing: "Writing skills IS Test-Driven Development applied to process documentation." (`skills/writing-skills/SKILL.md:10`). Also "LLMs respond to the same persuasion principles as humans." (`skills/writing-skills/persuasion-principles.md:5`)
- The newer, more nuanced doctrine: "The form that bulletproofs one failure type measurably backfires on another." (`skills/writing-skills/SKILL.md:463`). Also "never reach for the prohibition by default" (`:472`).

### writing-skills compared with Anthropic and OpenAI guidance

| Topic | writing-skills | AS / OAI | Assessment |
|---|---|---|---|
| Description content | "describes ONLY when to use (NOT what it does)" (`:99`), backed by one anecdote (`:154-156`) | "include both what the Skill does and when to use it" (AS:203); OAI agrees on "primary function + specific triggers" (OAI:28-29) | Direct conflict. The repo's own descriptions are inconsistent: verification, receiving-code-review and using-git-worktrees describe the workflow anyway. |
| Evals first | Baseline (RED) before writing; micro-tests with a no-guidance control (`:577-587`) | "Build evaluations first" (AS:744-756) | Aligned, and more rigorous than AS on controls and variance. |
| What success means | "Agent follows rule under maximum pressure" (`:411`); "Not bulletproof if: Agent argues skill is wrong" (`testing-skills-with-subagents.md:276-278`) | Evaluate task outcomes (AS:758-771) | The metric is compliance, not outcome quality. A skill that is wrong for a context still scores as "bulletproof". |
| Emphasis and persuasion | Authority, "YOU MUST", commitment (`persuasion-principles.md:15,45`) | Emphasise a single line (CC:188); fewer rules (OAI:48-51). AS:819 does allow "MUST filter" as an iteration step. | Mostly opposed. The newer "Match the Form" section (`:461-476`) moves toward the guidance. |
| Length | <150 / <200 / <500-word targets (`:217-220`) | Under 500 lines (AS:257) | The stated targets are stricter than AS, but the repo does not meet them. |
| Testing across models | Not addressed in the skill | "Test with all models you plan to use" (AS:134-144) | Release notes cite Claude, Codex and Sonnet 5 runs. The evals themselves are not visible. |

### How the pieces fit
**Entry point.** The bootstrap: "Invoke relevant or requested skills BEFORE any response or action — including clarifying questions" (`using-superpowers:20`). It also says "Before entering plan mode… invoke the brainstorming skill first" (`:22`) and "follow the skill exactly. If it has a checklist, create a todo per item" (`:24`). The routing examples are "Let's build X" → brainstorming and "Fix this bug" → systematic-debugging (`:30-31`).

**Flow 1: "Let's make a react todo list"** (the repo's own acceptance test, `AGENTS.md:78-82`)
1. **brainstorming.** The request is classified as architectural: "A new todo-list project is architectural and requires the written spec and planning handoffs." (`brainstorming:93-95`). Steps: intent question (`:19-24`), write-back of the understanding (`:25-28`), clarifying questions one at a time (`:132, :206`), an optional visual companion offered in its own message (`:131, :275`), 2-3 approaches, design sections with approval after each (`:134`), spec written to `docs/superpowers/specs/…` and committed (`:135, :241-244`), self-review, then a spec review gate (`:256-261`).
2. **writing-plans.** The plan goes to `docs/superpowers/plans/…` with a mandatory header, Global Constraints and Review Focus (`writing-plans:54-90`). The handoff says "Please review the plan. Which execution approach would you prefer?" (`:189-194`).
3. **using-git-worktrees**, entered from the setup step of SDD or executing-plans (`subagent-driven-development:126-129`). The agent asks consent unless a preference is declared (`using-git-worktrees:41-45`). If the worktree directory is not git-ignored, it edits `.gitignore` and commits (`:86`). It then runs `npm install` and baseline tests, and asks the user if they fail (`:104-130`).
4. **subagent-driven-development** (per task: implementer subagent, task-reviewer subagent, fix loop of up to 5 rounds, ledger) **or executing-plans** (inline, TDD loaded, `task-start` and `task-done` scripts). Either way it ends with a whole-branch review on "the most capable available model" (`subagent-driven-development:192-194`). The run does not stop between tasks except for four named classes (`subagent-driven-development:27-31`).
5. **test-driven-development** applies inside every task. **verification-before-completion** applies before every success claim.
6. **finishing-a-development-branch** re-runs tests, confirms the base branch if unknown (`:48-51`), presents "exactly these 3 options" and waits (`:55-82`).

Controller-side skill text loaded for this flow: about 12,000 words (11,981 for SDD, 12,432 for Native; `flow_cost.py`). On top of that, each task dispatches templates of 960 words (implementer) and 1,330 words (task reviewer).

**Flow 2: "fix this typo in the README"**
- The bootstrap is already paid. The 1% rule and the Red Flags table push toward invoking something: "The skill is overkill | Simple things become complex. Use it." (`using-superpowers:47`). Opting out requires "your human partner has explicitly told you to" (`:65`).
- verification-before-completion is likely, because its description covers "before committing" (`:3`).
- brainstorming fires only if the model reads a typo as "creative work… modifying behavior" (`brainstorming:3`). If it does, the lightest path is Bounded, which is still a hard gate: "STOP and wait for an explicit yes" (`:126`).
- No skill defines a "trivial change" exit. CC:106-110 ("If you could describe the diff in one sentence, skip the plan") has no counterpart. The repo has no negative-trigger test for this case (grep of `tests/` finds none).
- Likely load: about 1,100 words (bootstrap plus verification). Worst case adds brainstorming's 2,600 words and an approval round-trip.

**Flow 3: "fix this bug"**
- systematic-debugging comes first (`using-superpowers:31`). Four phases with "You MUST complete each phase before proceeding" (`systematic-debugging:46`), and a failing test is required before the fix (`:172-177`), which hands off to TDD.
- verification-before-completion applies before the fix is claimed (`:189`).
- After 3 failed fixes the agent must "Discuss with your human partner before attempting more fixes" (`:210`).
- There is no approval gate before the fix unless brainstorming also fires. Its description, "modifying behavior", plausibly matches a bug fix, and its Bounded path lists "a one-file fix" (`brainstorming:72`).
- Load: about 4,000 words (`flow_cost.py`).

**Every point where the model must stop for the human:**

| # | Gate | Where |
|---|---|---|
| 1 | Intent question when purpose is missing; write-back and invite correction | `brainstorming:19-28` |
| 2 | Clarifying questions, one per message | `brainstorming:132, 206` |
| 3 | Visual-companion offer, "MUST be its own message", then wait | `brainstorming:275` |
| 4 | Spike: approve probe. Bounded: approve the in-chat design ("STOP and wait for an explicit yes") | `brainstorming:44-45, 126` |
| 5 | Architectural: approval after each design section | `brainstorming:134, 220` |
| 6 | Written-spec review gate | `brainstorming:46-49, 256-261` |
| 7 | Plan review plus choice of execution method | `writing-plans:181-198` |
| 8 | Worktree consent (unless a preference is declared) | `using-git-worktrees:41-45` |
| 9 | Baseline tests fail: ask whether to proceed | `using-git-worktrees:130` |
| 10 | No implementation on main/master without explicit consent | `subagent-driven-development:128-129`, `executing-plans:112-113` |
| 11 | Mid-run: only destructive or irreversible, security, external side effect, or "plan so broken that every path forward is a guess" | `subagent-driven-development:27-31`, `executing-plans:39-43` |
| 12 | Final review done by self-review (no subagent): the human decides whether that is enough | `executing-plans:253-258` |
| 13 | TDD exceptions (prototype, generated, config) need permission | `test-driven-development:24-27, 330` |
| 14 | 3+ failed fixes: discuss architecture | `systematic-debugging:194-210` |
| 15 | Unclear review items: ask before implementing any; conflicts with partner's decisions | `receiving-code-review:42-48, 82-83` |
| 16 | Base branch unknown: confirm | `finishing-a-development-branch:48-51` |
| 17 | Integration menu (merge / PR / keep) | `finishing-a-development-branch:55-82` |
| 18 | Discard only by typed `discard`; worktree removal refused → ask | `finishing-a-development-branch:134-146, 177-196` |
| 19 | diagnosing: intake before analysis; bundle and issue only after approval | `diagnosing-superpowers:101-109` |

The todo-list request alone hits gates 1, 2, 5, 6, 7, 8 and 17 unconditionally: at least **7 human round-trips**, and more with one question per message and several design sections.

**Hard rules (selection):**
- 1% rule, skills before any response (`using-superpowers:11-20`).
- No implementation before the path's approvals (`brainstorming:38-56`).
- "take the heavier one… Nothing downgrades mid-task" (`brainstorming:86-88`).
- After architectural brainstorming, "the ONLY skill you invoke… is writing-plans — never frontend-design" (`brainstorming:184-186`).
- "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST… Delete it. Start over." (`test-driven-development:34-45`). The README restates it: "Deletes code written before tests." (`README.md:317`)
- "Never fix bugs without a test." (`test-driven-development:321`)
- "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST" (`systematic-debugging:17`)
- "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE" (`verification-before-completion:17`)
- Never "You're absolutely right!"; no gratitude (`receiving-code-review:29-32, 139-148`)
- Review mandatory after each task and before merge (`requesting-code-review:14-17`)
- Never parallel implementers (`subagent-driven-development:282`); never fix in the controller (`:408`); always name the model (`:204-206`)
- "NO SKILL WITHOUT A FAILING TEST FIRST", applying to "documentation updates" too (`writing-skills:379-393`)

**Subagents.** SDD dispatches 1 implementer and 1 reviewer per task, re-reviews per fix round, and a final reviewer, with tiered models (`subagent-driven-development:184-219`). Subagents may not spawn subagents (`implementer-prompt.md:50-60`). diagnosing-superpowers fans out 7 analysts in parallel (`:36-43`). Child sessions are told to ignore the bootstrap through `<SUBAGENT-STOP>` (`using-superpowers:6-8`). The OpenCode port does not rely on that text, because "the <SUBAGENT-STOP> note… relies on model compliance, which is not reliable" (`.opencode/plugins/superpowers.js:148-149`).

**User setup.**
- Install per harness (`README.md:52-305`).
- On Codex, set `multi_agent = true` and optionally a default subagent model (`using-superpowers/references/codex-tools.md:3-8, 70-79`).
- A git repository. Node for the visual companion and graph rendering. Git Bash on Windows (`RELEASE-NOTES.md:137`).
- `.superpowers/` in `.gitignore` (`brainstorming/visual-companion.md:58`).
- Telemetry opt-out via `SUPERPOWERS_DISABLE_TELEMETRY` (`README.md:397-399`).

### Philosophy shifts (from RELEASE-NOTES and docs)
- **v3.2.2 (Oct 2025)** added the EXTREMELY-IMPORTANT block, the 1% rule and the rationalization table. The stated rationale is "observed agent behavior"; no numbers are given (`RELEASE-NOTES.md:1097-1110`).
- **v6.0-6.4** moved the other way:
  - Compression campaign that removed persuasion and recap prose, with each cut micro-tested (`:128-133`).
  - Bootstrap trimmed, with the Red Flags table kept "unchanged" (`:165-167`).
  - Codex bootstrap removed (`:173`).
  - Brainstorming "Ceremony now scales to the task" (`:88`).
  - "Controllers no longer stall… only destructive or irreversible actions still stop" (`:92`).
  - executing-plans "no longer stops every few tasks" (`:31`).
  - Leaner plans with a capable reader: "plans took a quarter of the time and about a third of the tokens" (`:5, :11`).
- The direction is toward the guidance in the execution layer. The bootstrap and the discipline skills (TDD, debugging, verification) have stayed largely untouched.

---

## 4. Hotspots (5 per worry)

### Worry 1: length
| # | Where | Quote / fact | Conflicts with |
|---|---|---|---|
| 1 | `skills/subagent-driven-development/SKILL.md` (whole) | 564 lines, 4,854 words. Model Selection, waiting policy and fix loop are all inline (`:184-244, :354-429`) even though templates are split out | AS:257 (<500 lines); OAI:33-35 ("routers, not textbooks") |
| 2 | `skills/writing-skills/SKILL.md` + `anthropic-best-practices.md` | 677 lines + a vendored 1,150-line copy of AS with no TOC; 15 references >100 lines lack a TOC | AS:257, AS:404-406 (TOC for >100 lines) |
| 3 | `skills/using-superpowers/SKILL.md` via `hooks/session-start:27` | 520 words / ~900 tokens re-injected at "startup\|clear\|compact" (`hooks/hooks.json:5`), against the repo's own "<150 words" target (`writing-skills:218`); the Gemini variant is 1,110 words | CC:172-174 (load only what applies broadly); AS:22 (only metadata preloaded) |
| 4 | `skills/executing-plans/SKILL.md:68-106, 323-373` | 3,235 words for the "cheapest" mode; a 39-line dot graph and a 51-line example re-encode the prose | AS:24-30 ("Does this paragraph justify its token cost?") |
| 5 | `skills/brainstorming/SKILL.md:110-182` | The same flow is encoded three times: checklist (`:110-139`), dot graph (`:142-182`) and "Terminal states" prose (`:184-189`); 2,582 words for the most-triggered skill | AS:24-30; CC:174 ("Would removing this cause Claude to make mistakes?") |

### Worry 2: "dumbing down"
| # | Where | Quote | Conflicts with |
|---|---|---|---|
| 1 | `README.md:42` | "an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing" | AS:24 ("Claude is already very smart"); stale relative to `writing-plans:10` |
| 2 | `skills/test-driven-development/SKILL.md:39-45` | "Don't look at it / Delete means delete / Implement fresh from tests. Period." | AS:61-63 (match freedom to fragility); OAI:45-47 |
| 3 | `skills/using-superpowers/SKILL.md:35-50` | "These thoughts mean STOP—you're rationalizing", then "I need more context first" → "Skill check comes BEFORE clarifying questions." | CC:337-350 (have Claude ask questions); OAI:16-18 (scaffolding "causes unnecessary pauses") |
| 4 | `skills/verification-before-completion/SKILL.md:53, 58, 70` | "Expressing satisfaction before verification ('Great!', 'Perfect!', 'Done!')"; "Tired and wanting work over"; "Exhaustion ≠ excuse" | AS:24-30 (it explains what the model already knows); it anthropomorphises the model |
| 5 | `skills/writing-skills/persuasion-principles.md:5, 15` + `testing-skills-with-subagents.md:276-279` | "LLMs respond to the same persuasion principles as humans"; "Imperative language: 'YOU MUST'"; "Not bulletproof if: Agent argues skill is wrong" | OAI:19-20 ("Excessive defensive prompt scaffolding degrades agent performance"). The cited study is about persuading models to comply with *objectionable* requests (`persuasion-principles.md:173`), not about skill outcomes |

Runner-up: `systematic-debugging:52-56` ("Read Error Messages Carefully - Don't skip past errors… Read stack traces completely").

### Worry 3: outdated techniques
| # | Where | Quote | Conflicts with |
|---|---|---|---|
| 1 | `skills/using-superpowers/SKILL.md:10-16` (inside the hook's own `<EXTREMELY_IMPORTANT>`, `hooks/session-start:27`) | "even a 1% chance… you ABSOLUTELY MUST invoke"; "YOU DO NOT HAVE A CHOICE. YOU MUST USE IT." | CC:188 ("If you emphasize many lines, none of them stands out"); OAI:17-18 ("triggers irrelevant skills") |
| 2 | Four Iron Laws + "Violating the letter…" ×3 | `test-driven-development:14, 34`; `systematic-debugging:12, 17`; `verification-before-completion:12, 17`; `writing-skills:379` | OAI:48-49 ("point a direction instead of listing prohibitions") |
| 3 | 98 Excuse/Thought→Reality rows in 12 of 15 skills, plus 39 Red-Flag bullets; edits frozen by `AGENTS.md:100` | e.g. `systematic-debugging:249` "Emergency, no time for process" → "Systematic debugging is FASTER" | CC:174, 186 (bloat gets ignored; prune); OAI:50-51 |
| 4 | `skills/systematic-debugging/CREATION-LOG.md:41, 51-53` (shipped in the skill directory) | "'ALWAYS' / 'NEVER' (not 'should' / 'try to')"; "'NEVER fix symptom' appears 4 times" (deliberate redundancy) | CC:188; AS:24-30 |
| 5 | `skills/receiving-code-review/SKILL.md:29-30, 139-148` | "NEVER: 'You're absolutely right!' (explicit instruction-file violation)"; "❌ ANY gratitude expression"; "DELETE IT." | OAI:48-49 (prohibition lists); `:30` points at the maintainer's personal instruction file, which users do not have |

Runner-up: `writing-skills:616-631` ("you MUST STOP… MANDATORY… IMPORTANT: Create a todo for EACH checklist item").

### Worry 4: bureaucracy and over-constraint
| # | Where | Quote | Conflicts with |
|---|---|---|---|
| 1 | `skills/using-superpowers/SKILL.md:20, 65` + `README.md:323` | "BEFORE any response or action — including clarifying questions"; "Only skip skill workflows… when your human partner has explicitly told you to" | CC:106-110 (plan mode "adds overhead"; skip for small fixes) |
| 2 | `skills/brainstorming/SKILL.md:86-88, 93-95` | "When in doubt between two paths, take the heavier one… Nothing downgrades"; "A new todo-list project is architectural" | CC:110 ("describe the diff in one sentence, skip the plan"); CC:574 ("Sometimes you should skip planning") |
| 3 | `brainstorming:46-49, 134, 206, 256-261` + `writing-plans:181-194` | Stacked approvals: each design section, the spec, the plan, the execution method; "Only one question per message" | OAI:16-18 ("unnecessary pauses"); CC:345-350 (one interview, then a spec) |
| 4 | `skills/test-driven-development/SKILL.md:18-27` + `writing-skills:382-393` | "Always: … Refactoring"; exceptions need permission; "Not for 'documentation updates'" | AS:61-132 (high freedom where many approaches are valid) |
| 5 | `systematic-debugging:39-42` + `test-pressure-1.md:11, 38-40` | "Manager wants it fixed NOW (systematic is faster than thrashing)"; the test scenario treats a 35-minute investigation during a "$15,000/minute" outage as the compliant choice | AS:131-132 ("open field" = trust judgement); no mitigate-first carve-out |

Runners-up: `requesting-code-review:84-85` ("Never: Skip review because 'it's simple'"); `using-git-worktrees:86` (auto-commits a `.gitignore` change); `using-git-worktrees:114-115` (`poetry install` for any `pyproject.toml`).

Counter-evidence: `subagent-driven-development:17-31` and `executing-plans:27-43` deliberately remove mid-run check-ins ("a session parked on a question costs their whole day"). `finishing-a-development-branch:132-146` scopes destructive actions tightly.

---

## 5. Good citizens

| # | Passage | Why |
|---|---|---|
| 1 | `writing-plans:140-161, 175` ("What a Step Contains"; "Proportion") | "A plan longer than the code it describes has written the code instead." It defines done per step, describes the reader as capable (`:10`), and is measured: 9/9 planted-defect probes, a quarter of the time, a third of the tokens (`RELEASE-NOTES.md:5-12`). Matches OAI:45-47 and AS:61. |
| 2 | `executing-plans:32-43, 207-221` (rulings, four stops, completion contract) | Grants judgement ("decide them"), records it ("Ruling: … what it costs if wrong"), and defines done with evidence. Matches CC:27-52 and OAI:42-44. |
| 3 | `writing-skills:461-476, 577-587` (Match the Form to the Failure; Micro-Test Wording) | Evidence-led ("fully separated distributions"), always with a no-guidance control. Prefers positive recipes and refuses nuance clauses. Matches OAI:48-51 and AS:744-756. |
| 4 | `diagnosing-superpowers` (SKILL.md 991 words routing to 19 files) | A real router: "Every finding cites `path:line`. No citation, no finding." (`:15-17`). Analyst prompts specify return formats instead of steps (`prompts/analyst-common.md:23-38`). Matches AS patterns 1-2 and OAI:33-35. |
| 5 | `test-driven-development/writing-good-tests.md:8-63, 157-169` | A positive catalogue ("Every test names the break it catches"), a mutation check, and "prose for humans earns no test at all" (`:51-52`). Directional rather than prohibitive. |

Honourable mentions:
- `finishing-a-development-branch:177-198`: never `--force`; shows what would be lost. This is low freedom on a "narrow bridge" (AS:131), where it belongs.
- `verification-before-completion:38-48`: a claim → required-evidence table, essentially CC:52 as a table.

---

## 6. Capability map

| Layer | What you get |
|---|---|
| **Automates** | Bootstrap injection on most of the 16 listed harnesses. All except Codex and Devin; I verified the mechanism in-repo for Claude Code, Cursor, Copilot, Muse, Antigravity, OpenCode, Pi, Hermes, Gemini and Kimi. Factory Droid, Grok and Qwen presumably reuse the Claude plugin hook, but that is unverified. Skill routing. Spec and plan documents under `docs/superpowers/{specs,plans}`. Worktree creation, preferring the harness's native tool. Per-task subagent implement/review loops with model tiering, a 5-round circuit breaker and a compaction-proof ledger (`scripts/sdd-workspace`, `task-brief`, `review-package`, `task-start`, `task-done`). Final whole-branch review. Merge/PR/keep menu. A browser "visual companion" for mockups (a Node server with session-key auth). Session forensics with a scrubbed bug bundle and a GitHub issue draft. `find-polluter.sh` test bisection. Graphviz rendering of skill flowcharts. |
| **Enforces (in prose only)** | TDD order, root cause first, evidence before claims, design and plan approvals, review after every task, no parallel implementers, no controller-side fixes, an explicit model on every dispatch. Nothing is enforced by a hook (CC:227-231 recommends hooks for "zero exceptions" rules). |
| **Leaves to model/user** | Path classification (spike, bounded or architectural), the actual design, the choice of execution method, every merge/push decision, test commands, CI. There is no issue-tracker integration other than diagnosing-superpowers filing issues against superpowers itself. |
| **Assumes** | git. A subagent tool, for SDD (the fallback is executing-plans). bash (scripts). Node (visual companion). `gh`, optionally. A "human partner" available for at least 7 gates per feature. |

---

## 7. Verdicts

**Length.**
- The always-on cost is moderate: about 520 words (~900 tokens) per session, compaction or `/clear` on Claude Code, up to 1,110 words on Gemini, and zero on Codex.
- The per-workflow cost is heavy. A feature flow loads about 12,000 words of skill text into the controller, and two skills break the 500-line limit (SDD 564, writing-skills 677).
- Progressive disclosure works for prompt templates and diagnosing-superpowers. It fails elsewhere: three-fold encodings in brainstorming and executing-plans, 15 long references without a TOC, and test fixtures shipped in skill directories.
- The repo misses its own word targets: 13 of 15 skills exceed 500 words (`writing-skills:217-220`).
- Key instructions are sometimes buried. TDD's best addition, "the project's suite defines green" (`test-driven-development:185-193`), sits mid-file between boilerplate examples.

**"Dumbing down."**
- Present and concentrated in the older discipline layer: using-superpowers, TDD, systematic-debugging, verification and writing-skills. These police the model's "thoughts", forbid looking at deleted code, and anthropomorphise fatigue.
- writing-skills teaches authors to apply persuasion on the model and to treat disagreement with a skill as a test failure.
- The newer layer treats the model as capable and asks it to make and record rulings: writing-plans v6.4.2, SDD and executing-plans "rulings, not stalls", and diagnosing-superpowers.
- The README still advertises the "junior with poor taste, no judgement" framing (`README.md:42`) that the skill itself dropped.

**Outdated techniques.**
- Measured by count, the library leans heavily on them: double-wrapped EXTREMELY-IMPORTANT tags with ALL-CAPS sentences in the always-on bootstrap, 4 Iron Laws, 98 rationalization rows in 12/15 skills, 39 red-flag bullets, and 153 ALL-CAPS words (48 in writing-skills alone).
- The maintainer's defence (`AGENTS.md:42, 100`) is backed in one place: removing TDD's rebuttal prose cut test-first compliance from 8/10 to 5/10 on Claude and Codex (`RELEASE-NOTES.md:131`). The repo's own data also shows prohibitions backfiring on shaping tasks (`writing-skills:472`).
- No in-repo evidence supports the bootstrap's caps and 1% rule, or the "human partner" phrasing. They date from an unquantified v3.2.2 change (`RELEASE-NOTES.md:1110`), and the Codex bootstrap was removed because it "made the UX worse" (`:173`).
- Verdict: partly evidence-backed for discipline under pressure, unproven for the always-on layer.

**Bureaucracy and over-constraint.**
- High at the front of the workflow. A new todo app requires at least 7 human round-trips before and after code.
- The only scale-down mechanism (brainstorming's three paths) is biased upward ("take the heavier one"; todo app = architectural), and even the lightest path is a hard approval gate.
- Opting out requires the human to say so explicitly (`using-superpowers:65`). There is no "one-sentence diff" exit (contrast CC:110).
- Once a plan is approved, execution is deliberately low-friction: four stop classes only, rulings logged, no progress check-ins. This is a sound design that matches CC and OAI.
- Net: the gates add safety at integration and destructive steps, but add friction without demonstrated benefit at design time for small or familiar tasks. No in-repo eval measures over-triggering or the cost of gates on trivial work.

**On the maintainer's claim that the philosophy is "extensively tested and tuned".**
- The repo shows real eval craft:
  - Controls, n=5-25 per arm, hand-read results (`docs/superpowers/specs/2026-06-10-positive-instruction-redesign-design.md:8-18`; `2026-07-06-sdd-plan-scoped-workspace-eval-results.md`).
  - 50/50 worktree runs.
  - Cost studies.
- The evidence covers recent, narrow changes. Twice it shows guidance was unnecessary for current models: "Current-generation opus does not produce plan placeholders… with or without the banned-patterns list" (`positive-instruction-redesign-design.md:153-155`), and the Codex bootstrap removal.
- The specific content AGENTS.md freezes (Red Flags tables, rationalization lists, "human partner") has, in this repo, only the TDD 8/10 vs 5/10 datum.
- The rest would have to be in the uncloned superpowers-evals repo.
- Verdict: partially backed.

---

## 8. Open questions

1. **superpowers-evals is not cloned.** I cannot see its scenarios, models, pass rates, or whether the bootstrap's caps, 1% rule, "human partner" phrasing or Red Flags were ever A/B tested on current models. diagnosing-superpowers' eval records are "kept by the maintainer outside the repo" (`docs/testing.md:21`).
2. **Codex metadata.** The Codex package seeds `skills/*/agents/openai.yaml` from a prior package (`scripts/package-codex-plugin.sh:33-36`). Those files are not in the repo, so the descriptions Codex actually shows, and whether truncation happens, are unknown.
3. **Brainstorming on bugs and typos.** Whether brainstorming fires on "fix this bug" or a README typo is model-dependent. The repo has positive trigger tests only, with no negative or over-trigger tests.
4. **Subagent bootstrap on Claude Code.** Does the SessionStart hook reach Claude Code subagents? `<SUBAGENT-STOP>` (`using-superpowers:6-8`) suggests the maintainers saw it happen on some harness. OpenCode handles it structurally.
5. **New projects without git.** For a brand-new project with no git repository (the todo-list case), no skill says to run `git init`. The worktree and finishing skills assume git.
6. **Kimi.** I assumed the `sessionStart` skill is injected without frontmatter; the manifest does not say.
7. **Persuasion research.** `persuasion-principles.md` applies Meincke et al. (2025), an objectionable-request compliance study, to engineering skills. No in-repo replication on skill outcomes exists.
8. **Stale docs.** README claims that contradict current skills ("2-5 minutes each… complete code", `README.md:313`; "two-stage review (spec compliance, then code quality)", `:360`) and stale recall tests (`tests/claude-code/test-subagent-driven-development.sh:42-46, 135-145` still expect two reviewers and pasted task text) may mislead. Whether these tests still pass is unknown.
9. **Token counts.** All token counts are estimates. The "94% PR rejection rate" (`AGENTS.md:7`) cannot be verified from files.

---

## Appendix: scripts (re-run from the scratchpad root)

```bash
cd <analysis-workspace>   # scripts were run from the analysis workspace; they are reproduced below
python3 scripts/obra/skill_metrics.py repos/obra-superpowers          # inventory + counters
python3 scripts/obra/rationalization_rows.py repos/obra-superpowers   # table rows / red-flag bullets
python3 scripts/obra/ref_files.py repos/obra-superpowers              # every non-SKILL file, TOC check
python3 scripts/obra/bootstrap_cost.py repos/obra-superpowers         # injected text per harness (runs hooks/session-start)
python3 scripts/obra/flow_cost.py repos/obra-superpowers              # words loaded per flow
cd repos/obra-superpowers && git log -1 --format='%H %ci %s'; wc -l -w skills/*/SKILL.md
```


### scripts/obra/skill_metrics.py

```python
#!/usr/bin/env python3
"""Per-skill metrics for a skills/ tree (obra/superpowers layout).

Usage: python3 skill_metrics.py <repo-root> [--json]

Counts are computed on the SKILL.md BODY (frontmatter excluded) unless noted.
Prose counts (ALLCAPS, ALWAYS/NEVER/MUST, IMPORTANT, STOP, XML tags) EXCLUDE
fenced code blocks (``` ... ```), because those hold dot graphs, shell, and
example code; a separate "_incode" column reports what was skipped.
"""
import os, re, sys, json, glob
from collections import deque

ROOT = sys.argv[1]
SK = os.path.join(ROOT, "skills")

# Acronyms / technical tokens that are uppercase but are not emphasis.
ACRONYMS = set("""TDD API APIS YAGNI DRY SHA SHAS JSON HTML CSS URL URLS PR PRS CLI
CPU UI UX SDD SDO JWT XML YAML HTTP HTTPS SSH GIT TODO TODOS TBD MCP SVG PDF
RED GREEN REFACTOR OK ID IDS UUID UUIDS JS TS ENV PID ISO BASE HEAD FIX_BASE
MERGE_BASE PLAN_FILE BRIEF_FILE REPORT_FILE DIFF_FILE BASE_SHA HEAD_SHA FIX_BASE_SHA
DESCRIPTION PLAN_OR_REQUIREMENTS MODEL FINDINGS GLOBAL_CONSTRAINTS CASE RANGE BUNDLE
PUBLIC_REPOS PROPRIETARY GIT_DIR GIT_COMMON BRANCH WORKTREE_PATH MAIN_ROOT LOCATION
BRANCH_NAME STATE_DIR SESSION_DIR CODEX_CI NODE_ENV IDENTITY APP DONE DONE_WITH_CONCERNS
BLOCKED NEEDS_CONTEXT ADDRESSED CLEAN MISSED README CHANGELOG LLM LLMS AI VCS CI
DNS SPA REST SQL NPM AWS GCP OAUTH JSONL ENOTEMPTY RSS N/A NA MD YYYY-MM-DD A/B/C/D
RED-GREEN RED-GREEN-REFACTOR""".split())

CAPS_RE = re.compile(r"\b[A-Z][A-Z'_/-]*[A-Z]\b")          # 2+ chars, all caps
ANM_RE = re.compile(r"\b(always|never|must)\b", re.I)
ANM_UP_RE = re.compile(r"\b(ALWAYS|NEVER|MUST)\b")
IMPORTANT_RE = re.compile(r"\bIMPORTANT\b")
STOP_RE = re.compile(r"\bSTOP\b")
XML_RE = re.compile(r"<(/?)([A-Z][A-Za-z_-]+)>")               # <HARD-GATE>, <Good>, ...
PARTNER_RE = re.compile(r"human partner", re.I)
RF_HEAD_RE = re.compile(r"^#+ .*red flag", re.I)
RT_TABLE_RE = re.compile(r"^\|\s*(Excuse|Thought)\s*\|\s*Reality\s*\|", re.I)
NOEXC_RE = re.compile(r"no exceptions", re.I)

def split_frontmatter(text):
    m = re.match(r"^---\n(.*?)\n---\n?(.*)$", text, re.S)
    if not m:
        return {}, text, 0
    fm_raw, body = m.group(1), m.group(2)
    fm = {}
    for line in fm_raw.splitlines():
        if ":" in line:
            k, v = line.split(":", 1)
            fm[k.strip()] = v.strip().strip('"').strip("'")
    fm_lines = fm_raw.count("\n") + 3   # --- + lines + ---
    return fm, body, fm_lines

def strip_code(text):
    out, incode, code = [], False, []
    for line in text.splitlines():
        if line.lstrip().startswith("```"):
            incode = not incode
            code.append(line)
            continue
        (code if incode else out).append(line)
    return "\n".join(out), "\n".join(code)

INLINE_CODE_RE = re.compile(r"`[^`\n]*`")
MDFILE_RE = re.compile(r"\b[\w./-]+\.(md|json|sh|js|ts|dot)\b")

def prose_counts(text):
    prose, code = strip_code(text)
    xml_open = len([m for m in XML_RE.finditer(prose) if m.group(1) == ""])
    prose_nc = MDFILE_RE.sub(" ", XML_RE.sub(" ", INLINE_CODE_RE.sub(" ", prose)))
    caps_all = CAPS_RE.findall(prose_nc)
    caps = [w for w in caps_all if w.strip("'/-_") not in ACRONYMS and len(w.strip("'/-_")) >= 2]
    return {
        "allcaps": len(caps),
        "allcaps_incode": len([w for w in CAPS_RE.findall(code) if w not in ACRONYMS]),
        "anm_ci": len(ANM_RE.findall(prose)),
        "anm_upper": len(ANM_UP_RE.findall(prose)),
        "important": len(IMPORTANT_RE.findall(prose)),
        "stop": len(STOP_RE.findall(prose)),
        "xml_tags": xml_open,
        "human_partner": len(PARTNER_RE.findall(text)),
        "rf_headings": sum(1 for l in prose.splitlines() if RF_HEAD_RE.match(l)),
        "rt_tables": sum(1 for l in prose.splitlines() if RT_TABLE_RE.match(l)),
        "no_exceptions": len(NOEXC_RE.findall(text)),
        "caps_words": caps,
    }

def words(text):
    return len(text.split())

def ref_graph(skill_dir):
    """BFS from SKILL.md over mentions of other files in the same skill dir
    (by relative path or basename) and cross-skill '../<skill>/...' paths.
    Returns {relpath: depth} for reachable files and list of unreferenced files."""
    files = []
    for dp, dn, fn in os.walk(skill_dir):
        for f in fn:
            files.append(os.path.relpath(os.path.join(dp, f), skill_dir))
    text_of = {}
    for f in files:
        try:
            text_of[f] = open(os.path.join(skill_dir, f), encoding="utf-8").read()
        except Exception:
            text_of[f] = ""
    def mentions(src, tgt):
        t = text_of.get(src, "")
        if tgt == "SKILL.md":
            return False
        base = os.path.basename(tgt)
        return (tgt in t) or re.search(r"(?<![\w/.-])" + re.escape(base) + r"(?![\w-])", t) is not None
    depth = {"SKILL.md": 0}
    q = deque(["SKILL.md"])
    while q:
        cur = q.popleft()
        if not cur.endswith((".md", ".dot")):   # only follow prose files
            continue
        for g in files:
            if g not in depth and mentions(cur, g):
                depth[g] = depth[cur] + 1
                q.append(g)
    unref = sorted(f for f in files if f not in depth)
    return depth, unref, files

def main():
    rows = []
    tot = dict(skill_words=0, ref_md_words=0, ref_md_files=0, script_files=0, script_words=0)
    for sd in sorted(glob.glob(os.path.join(SK, "*"))):
        p = os.path.join(sd, "SKILL.md")
        if not os.path.isfile(p):
            continue
        text = open(p, encoding="utf-8").read()
        fm, body, fm_lines = split_frontmatter(text)
        pc = prose_counts(body)
        depth, unref, files = ref_graph(sd)
        md_refs = [f for f in files if f != "SKILL.md" and f.endswith(".md")]
        other = [f for f in files if f != "SKILL.md" and not f.endswith(".md")]
        md_words = sum(words(open(os.path.join(sd, f), encoding="utf-8").read()) for f in md_refs)
        oth_words = sum(words(open(os.path.join(sd, f), encoding="utf-8", errors="ignore").read()) for f in other)
        reach_md = {f: d for f, d in depth.items() if f != "SKILL.md"}
        tot["skill_words"] += words(text)
        tot["ref_md_words"] += md_words
        tot["ref_md_files"] += len(md_refs)
        tot["script_files"] += len(other)
        tot["script_words"] += oth_words
        inv = "both" if fm.get("disable-model-invocation", "false") != "true" else "user"
        rows.append(dict(
            name=fm.get("name"), invocation=inv,
            file_lines=text.count("\n") + (0 if text.endswith("\n") else 1),
            body_lines=body.count("\n") + (0 if body.endswith("\n") else 1),
            body_words=words(body), file_words=words(text),
            desc_chars=len(fm.get("description", "")),
            desc_starts_use_when=fm.get("description", "").lower().startswith("use when"),
            ref_md=len(md_refs), ref_md_words=md_words, other_files=len(other),
            ref_max_depth=max(reach_md.values()) if reach_md else 0,
            ref_md_max_depth=max([d for f, d in reach_md.items() if f.endswith(".md")] or [0]),
            unreferenced=unref,
            **{k: v for k, v in pc.items() if k != "caps_words"},
            caps_sample=sorted(set(pc["caps_words"]))[:40],
        ))
    if "--json" in sys.argv:
        print(json.dumps(dict(rows=rows, totals=tot), indent=1))
        return
    hdr = ["name","inv","body_lines","body_words","desc_chars","ref_md","other_files",
           "ref_md_max_depth","ref_max_depth","allcaps","anm_ci","anm_upper","important","stop","xml_tags",
           "human_partner","rf_headings","rt_tables","no_exceptions"]
    print("\t".join(hdr))
    for r in rows:
        print("\t".join(str(r[h if h != "inv" else "invocation"]) for h in hdr))
    sums = {h: sum(r[h] for r in rows) for h in hdr[2:]}
    print("TOTAL\t-\t" + "\t".join(str(sums[h]) for h in hdr[2:]))
    print()
    print("totals:", json.dumps(tot))
    print("desc chars total:", sum(r["desc_chars"] for r in rows),
          "| name+desc chars total:", sum(r["desc_chars"] + len(r["name"]) for r in rows))
    print("skills over 500 body lines:", [r["name"] for r in rows if r["body_lines"] > 500])
    print("descriptions starting 'Use when':", sum(r["desc_starts_use_when"] for r in rows), "/", len(rows))
    print()
    for r in rows:
        if r["unreferenced"] and r["unreferenced"] != []:
            print("unreferenced from SKILL.md graph:", r["name"], r["unreferenced"])

if __name__ == "__main__":
    main()
```

### scripts/obra/rationalization_rows.py

```python
#!/usr/bin/env python3
"""Count rows in Excuse|Reality / Thought|Reality tables and bullets under
'Red Flags' headings in each SKILL.md (outside fenced code).
Usage: python3 rationalization_rows.py <repo-root>"""
import os, re, sys, glob
sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
R = os.path.abspath(sys.argv[1]); sys.argv = [sys.argv[0], R]
import skill_metrics as m
T = re.compile(r"^\|\s*(Excuse|Thought)\s*\|\s*Reality\s*\|", re.I)
tot_rows = tot_bul = 0
for p in sorted(glob.glob(f"{R}/skills/*/SKILL.md")):
    prose = m.strip_code(open(p, encoding="utf-8").read())[0].splitlines()
    rows = bullets = 0
    for i, l in enumerate(prose):                      # table rows anywhere
        if T.match(l):
            j = i + 2
            while j < len(prose) and prose[j].startswith("|"):
                rows += 1; j += 1
    sec = False
    for l in prose:                                    # bullets under Red Flags
        if l.startswith("#"):
            sec = re.match(r"^#+ .*red flag", l, re.I) is not None
        elif sec and re.match(r"^\s*[-*] ", l):
            bullets += 1
    tot_rows += rows; tot_bul += bullets
    print(f"{os.path.basename(os.path.dirname(p)):<32} table_rows={rows:>3} red_flag_bullets={bullets:>3}")
print(f"TOTAL table_rows={tot_rows} red_flag_bullets={tot_bul}")
```

### scripts/obra/ref_files.py

```python
#!/usr/bin/env python3
"""List every non-SKILL.md file under skills/: lines, words, TOC presence for
long markdown, and the same emphasis counters as skill_metrics.py.

Usage: python3 ref_files.py <repo-root>
"""
import os, sys, re, glob
sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
sys.argv = [sys.argv[0], sys.argv[1]]
import skill_metrics as m

R = os.path.abspath(sys.argv[1])
TOC_RE = re.compile(r"^#+\s*(contents|table of contents)\b", re.I | re.M)
tot = dict(md=0, md_words=0, rt=0, rf=0, hp=0, caps=0, anm=0)
print("file\tlines\twords\tlong_no_toc\tallcaps\tanm_ci\thuman_partner\trf_head\trt_tables")
for p in sorted(glob.glob(os.path.join(R, "skills", "**", "*"), recursive=True)):
    if not os.path.isfile(p) or os.path.basename(p) == "SKILL.md":
        continue
    rel = os.path.relpath(p, R)
    t = open(p, encoding="utf-8", errors="ignore").read()
    lines = t.count("\n")
    if p.endswith(".md"):
        c = m.prose_counts(t)
        long_no_toc = lines > 100 and not TOC_RE.search(m.strip_code(t)[0])
        print(f"{rel}\t{lines}\t{len(t.split())}\t{'YES' if long_no_toc else ''}\t{c['allcaps']}\t{c['anm_ci']}"
              f"\t{c['human_partner']}\t{c['rf_headings']}\t{c['rt_tables']}")
        tot["md"] += 1; tot["md_words"] += len(t.split()); tot["rt"] += c["rt_tables"]
        tot["rf"] += c["rf_headings"]; tot["hp"] += c["human_partner"]; tot["caps"] += c["allcaps"]
        tot["anm"] += c["anm_ci"]
    else:
        print(f"{rel}\t{lines}\t{len(t.split())}\t(non-md)")
print("TOTALS (md refs):", tot)
```

### scripts/obra/bootstrap_cost.py

```python
#!/usr/bin/env python3
"""Reconstruct the always-on text superpowers injects per harness and size it.

Usage: python3 bootstrap_cost.py <repo-root>

- Claude Code: runs hooks/session-start exactly as the plugin does (env
  CLAUDE_PLUGIN_ROOT set) and decodes hookSpecificOutput.additionalContext.
  Fires on SessionStart matcher startup|clear|compact (hooks/hooks.json).
- Cursor / Copilot / Muse / Antigravity: same script, same text, other JSON key.
- OpenCode V1/V2, Pi, Hermes: rebuilt from their injector source (wrapper +
  frontmatter-stripped body + per-harness tool mapping).
- Gemini: GEMINI.md @-includes SKILL.md + references/gemini-tools.md.
- Kimi: manifest sessionStart skill (assumed body without frontmatter) +
  skillInstructions string.
- Codex, Devin: no bootstrap; only the skills' name+description index.
Token figures are ESTIMATES (no tokenizer offline): chars/4 and chars/3.5.
"""
import json, os, re, subprocess, sys

R = os.path.abspath(sys.argv[1])
SK = os.path.join(R, "skills", "using-superpowers", "SKILL.md")
raw = open(SK, encoding="utf-8").read()
body = re.match(r"^---\n[\s\S]*?\n---\n([\s\S]*)$", raw).group(1)

def size(label, text, note=""):
    c = len(text); w = len(text.split())
    print(f"{label:<34} chars={c:>6} words={w:>5} est_tokens={c//4:>5}-{int(c/3.5):>5}  {note}")
    return c

# 1. Claude Code (and Cursor/Copilot/Muse/Antigravity) via the real hook script
env = dict(os.environ, CLAUDE_PLUGIN_ROOT=R)
env.pop("CURSOR_PLUGIN_ROOT", None); env.pop("COPILOT_CLI", None); env.pop("MUSE_PLUGIN_ROOT", None)
out = subprocess.run(["bash", os.path.join(R, "hooks", "session-start")], env=env,
                     capture_output=True, text=True, check=True).stdout
cc = json.loads(out)["hookSpecificOutput"]["additionalContext"]
size("Claude Code SessionStart hook", cc, "per startup|clear|compact")
open(os.path.join(os.path.dirname(__file__), "claude-code-bootstrap.txt"), "w").write(cc)

# 2. OpenCode V1 / V2: pull the exported mapping constants with node
js = os.path.join(R, ".opencode", "plugins", "superpowers.js")
maps = json.loads(subprocess.run(
    ["node", "--input-type=module", "-e",
     f"import {{V1_MAPPING, V2_MAPPING}} from '{js}'; console.log(JSON.stringify([V1_MAPPING, V2_MAPPING]))"],
    capture_output=True, text=True, check=True).stdout)
for name, mp in zip(("OpenCode V1", "OpenCode V2"), maps):
    t = ("<EXTREMELY_IMPORTANT>\nYou have superpowers.\n\n**IMPORTANT: The using-superpowers skill content is "
         "included below. It is ALREADY LOADED - you are currently following it. Do NOT use the skill tool to load "
         "\"using-superpowers\" again - that would be redundant.**\n\n" + body + "\n\n" + mp + "\n</EXTREMELY_IMPORTANT>")
    size(name, t, "first user message; re-added after compaction (V2)")

# 3. Pi: wrapper + marker + body + piToolMapping() (extract template literal)
ts = open(os.path.join(R, ".pi", "extensions", "superpowers.ts"), encoding="utf-8").read()
pimap = re.search(r"function piToolMapping\(\): string \{\n\treturn `([\s\S]*?)`;", ts).group(1).replace("\\`", "`")
t = ("<EXTREMELY_IMPORTANT>\nsuperpowers:using-superpowers bootstrap for pi\n\nYou have superpowers.\n\nThe using-superpowers "
     "skill content is included below and is already loaded for this Pi session. Follow it now. Do not try to load "
     "using-superpowers again.\n\n" + body.strip() + "\n\n" + pimap + "\n</EXTREMELY_IMPORTANT>")
size("Pi", t, "session_start + session_compact")

# 4. Hermes: wrapper + body + loader section + hermes-tools.md
ht = open(os.path.join(R, "skills", "using-superpowers", "references", "hermes-tools.md"), encoding="utf-8").read().strip()
sd = "~/.hermes/plugins/superpowers/skills"
t = ("<EXTREMELY_IMPORTANT>\nsuperpowers:using-superpowers bootstrap for hermes\n\nYou have superpowers.\n\n"
     "The using-superpowers skill content is included below and is already loaded for this Hermes session. "
     "Follow it now. Do not try to load using-superpowers again.\n\n" + body.strip() +
     "\n\n## Loading Superpowers Skills on Hermes\n\nSuperpowers skills are registered with Hermes' native skill loader: "
     'invoke one with `skill_view("superpowers:skill-name")` (for example `skill_view("superpowers:brainstorming")`). '
     "If a namespaced lookup returns 'not found', read the skill file directly instead:\n"
     f'`read_file("{sd}/skill-name/SKILL.md")`\n\nThe superpowers skills directory is: `{sd}`\n\n' + ht + "\n</EXTREMELY_IMPORTANT>")
size("Hermes", t, "first turn only (no post-compaction hook)")

# 5. Gemini: GEMINI.md context file includes two files
gt = open(os.path.join(R, "skills", "using-superpowers", "references", "gemini-tools.md"), encoding="utf-8").read()
size("Gemini (GEMINI.md @includes)", raw + gt, "context file, every session")

# 6. Kimi
km = json.load(open(os.path.join(R, ".kimi-plugin", "plugin.json")))
size("Kimi (body + skillInstructions)", body + km["skillInstructions"], "sessionStart skill (assumed body)")

# 7. Description index (all harnesses with native skills; the only cost on Codex/Devin)
idx = []
for d in sorted(os.listdir(os.path.join(R, "skills"))):
    p = os.path.join(R, "skills", d, "SKILL.md")
    if os.path.isfile(p):
        fm = re.match(r"^---\n([\s\S]*?)\n---", open(p, encoding="utf-8").read()).group(1)
        name = re.search(r"^name:\s*(.*)$", fm, re.M).group(1).strip()
        desc = re.search(r"^description:\s*(.*)$", fm, re.M).group(1).strip().strip('"')
        idx.append(f"- {name}: {desc}")
size("Skill index (15 name+desc lines)", "\n".join(idx), "Codex/Devin: only always-on cost")
print("Codex budget: 8,000 chars when window unknown; 2% of window otherwise "
      "(2% of 200k tokens ~ 4,000 tokens ~ 16,000 chars).")
```

### scripts/obra/flow_cost.py

```python
#!/usr/bin/env python3
"""Sum the SKILL.md (and template) words a CONTROLLER context loads per flow.

Usage: python3 flow_cost.py <repo-root>
Word counts = whitespace-split words of the whole file (frontmatter included),
since the Skill tool loads the whole file. Bootstrap = the injected hook text.
Flows follow the skill text (see report section 3); "maybe" skills are listed
separately because whether they fire depends on the model's reading of the
1%-rule bootstrap.
"""
import os, sys, subprocess, json
R = os.path.abspath(sys.argv[1])
def w(rel):
    return len(open(os.path.join(R, rel), encoding="utf-8").read().split())
def s(name):
    return w(f"skills/{name}/SKILL.md")

env = dict(os.environ, CLAUDE_PLUGIN_ROOT=R)
boot = json.loads(subprocess.run(["bash", f"{R}/hooks/session-start"], env=env,
       capture_output=True, text=True).stdout)["hookSpecificOutput"]["additionalContext"]
B = len(boot.split())

flows = {
 "build feature, SDD (controller)": ["brainstorming", "using-git-worktrees", "writing-plans",
                                     "subagent-driven-development", "finishing-a-development-branch"],
 "build feature, Native/inline":    ["brainstorming", "using-git-worktrees", "writing-plans",
                                     "executing-plans", "test-driven-development",
                                     "verification-before-completion", "finishing-a-development-branch"],
 "fix this bug":                    ["systematic-debugging", "test-driven-development",
                                     "verification-before-completion"],
 "fix README typo (likely)":        ["verification-before-completion"],
}
maybe = {
 "fix this bug": ["brainstorming (bounded path, 'modifying behavior')"],
 "fix README typo (likely)": ["brainstorming (if read as 'creative work')"],
}
for f, sk in flows.items():
    tot = B + sum(s(x) for x in sk)
    print(f"{f:<36} bootstrap={B} + skills={sum(s(x) for x in sk):>5} -> {tot:>5} words  ({', '.join(sk)})")
    if f in maybe:
        print(f"{'':<36} maybe: {maybe[f]}")

# Per-task subagent prompt payload (template words, before filling) for SDD
tmpl = {"implementer": "skills/subagent-driven-development/implementer-prompt.md",
        "task-reviewer": "skills/subagent-driven-development/task-reviewer-prompt.md",
        "re-review": "skills/subagent-driven-development/re-review-prompt.md",
        "final code-reviewer": "skills/requesting-code-review/code-reviewer.md",
        "spec-doc reviewer (orphan)": "skills/brainstorming/spec-document-reviewer-prompt.md"}
for k, v in tmpl.items():
    print(f"template {k:<28} {w(v):>5} words")
# Note: subagents do not get the bootstrap on Claude Code? The SessionStart hook
# runs per session; the <SUBAGENT-STOP> block tells dispatched subagents to ignore it.
```

### Key outputs (as run on commit 8ca22db)

```text
name	inv	body_lines	body_words	desc_chars	ref_md	other_files	ref_md_max_depth	ref_max_depth	allcaps	anm_ci	anm_upper	important	stop	xml_tags	human_partner	rf_headings	rt_tables	no_exceptions
brainstorming	both	281	2582	198	2	5	1	2	15	3	1	0	2	1	8	1	1	0
diagnosing-superpowers	both	116	991	375	19	0	2	2	0	7	0	0	0	0	1	1	1	0
dispatching-parallel-agents	both	163	843	106	0	0	0	0	2	1	0	0	0	0	0	0	0	0
executing-plans	both	369	3235	170	0	2	0	1	6	9	0	0	0	0	7	0	1	0
finishing-a-development-branch	both	221	1246	101	0	0	0	0	0	3	0	0	0	0	9	0	1	0
receiving-code-review	both	201	879	234	0	0	0	0	5	1	1	0	0	0	9	0	0	0
requesting-code-review	both	91	404	107	1	0	1	1	0	3	0	0	0	0	0	1	1	0
subagent-driven-development	both	564	4854	85	3	3	1	1	6	27	0	0	0	0	4	0	1	0
systematic-debugging	both	279	1422	91	8	2	1	2	28	3	3	0	6	0	2	1	1	0
test-driven-development	both	326	1459	79	1	0	1	1	4	7	0	0	1	4	3	1	1	2
using-git-worktrees	both	163	1035	196	0	0	0	0	3	3	1	0	0	0	1	0	1	0
using-superpowers	both	61	465	154	7	0	1	1	23	2	2	2	3	2	1	1	1	0
verification-before-completion	both	116	542	225	0	0	0	0	8	2	1	0	1	0	0	1	1	1
writing-plans	both	200	1619	84	0	0	0	0	5	2	1	0	0	0	1	0	0	0
writing-skills	both	677	3795	97	4	2	2	2	48	13	5	1	2	2	2	1	1	3
TOTAL	-	3828	25371	2302	45	14	10	13	153	86	15	3	15	9	48	8	12	6

totals: {"skill_words": 25796, "ref_md_words": 27802, "ref_md_files": 45, "script_files": 14, "script_words": 9214}
desc chars total: 2302 | name+desc chars total: 2615
skills over 500 body lines: ['subagent-driven-development', 'writing-skills']
descriptions starting 'Use when': 14 / 15

unreferenced from SKILL.md graph: brainstorming ['scripts/server.cjs', 'spec-document-reviewer-prompt.md']
unreferenced from SKILL.md graph: systematic-debugging ['CREATION-LOG.md', 'test-academic.md', 'test-pressure-1.md', 'test-pressure-2.md', 'test-pressure-3.md']
unreferenced from SKILL.md graph: using-superpowers ['references/gemini-tools.md']

brainstorming                    table_rows=  7 red_flag_bullets=  0
diagnosing-superpowers           table_rows=  6 red_flag_bullets=  0
dispatching-parallel-agents      table_rows=  0 red_flag_bullets=  0
executing-plans                  table_rows= 12 red_flag_bullets=  0
finishing-a-development-branch   table_rows= 10 red_flag_bullets=  0
receiving-code-review            table_rows=  0 red_flag_bullets=  0
requesting-code-review           table_rows=  2 red_flag_bullets=  7
subagent-driven-development      table_rows=  9 red_flag_bullets=  0
systematic-debugging             table_rows=  8 red_flag_bullets= 11
test-driven-development          table_rows= 11 red_flag_bullets= 13
using-git-worktrees              table_rows=  5 red_flag_bullets=  0
using-superpowers                table_rows= 12 red_flag_bullets=  0
verification-before-completion   table_rows=  8 red_flag_bullets=  8
writing-plans                    table_rows=  0 red_flag_bullets=  0
writing-skills                   table_rows=  8 red_flag_bullets=  0
TOTAL table_rows=98 red_flag_bullets=39

Claude Code SessionStart hook      chars=  3405 words=  520 est_tokens=  851-  972  per startup|clear|compact
OpenCode V1                        chars=  3737 words=  579 est_tokens=  934- 1067  first user message; re-added after compaction (V2)
OpenCode V2                        chars=  4155 words=  642 est_tokens= 1038- 1187  first user message; re-added after compaction (V2)
Pi                                 chars=  4367 words=  676 est_tokens= 1091- 1247  session_start + session_compact
Hermes                             chars=  5677 words=  828 est_tokens= 1419- 1622  first turn only (no post-compaction hook)
Gemini (GEMINI.md @includes)       chars=  7774 words= 1110 est_tokens= 1943- 2221  context file, every session
Kimi (body + skillInstructions)    chars=  4941 words=  762 est_tokens= 1235- 1411  sessionStart skill (assumed body)
Skill index (15 name+desc lines)   chars=  2689 words=  380 est_tokens=  672-  768  Codex/Devin: only always-on cost
Codex budget: 8,000 chars when window unknown; 2% of window otherwise (2% of 200k tokens ~ 4,000 tokens ~ 16,000 chars).

build feature, SDD (controller)      bootstrap=520 + skills=11461 -> 11981 words  (brainstorming, using-git-worktrees, writing-plans, subagent-driven-development, finishing-a-development-branch)
build feature, Native/inline         bootstrap=520 + skills=11912 -> 12432 words  (brainstorming, using-git-worktrees, writing-plans, executing-plans, test-driven-development, verification-before-completion, finishing-a-development-branch)
fix this bug                         bootstrap=520 + skills= 3495 ->  4015 words  (systematic-debugging, test-driven-development, verification-before-completion)
                                     maybe: ["brainstorming (bounded path, 'modifying behavior')"]
fix README typo (likely)             bootstrap=520 + skills=  580 ->  1100 words  (verification-before-completion)
                                     maybe: ["brainstorming (if read as 'creative work')"]
template implementer                    960 words
template task-reviewer                 1330 words
template re-review                      710 words
template final code-reviewer            915 words
template spec-doc reviewer (orphan)     235 words
```
