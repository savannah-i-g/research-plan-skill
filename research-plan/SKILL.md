---
name: research-plan
description: Multi-phase research-and-planning workflow for substantial design work. Phase 1 — parallel codebase exploration → design synthesis via Plan agent → user-locked decisions → ExitPlanMode → numbered doc tree → memory save. Phase 2 — parallel web research → findings folded back into design with traceability. Use when the user asks for extensive planning, deep design with grounding, research-backed plans for new languages / libraries / large features, or a "plan of how you will plan" for ambitious work. Skip for small fixes, single-file edits, or anything that fits in under ~30 LOC of change.
---

# research-plan — substantial design with web grounding

A multi-phase workflow for planning work that is too big to "just code"
and benefits from both deep codebase exploration and external research
grounding. Produces an approved plan + a populated documentation tree +
a research trail with citations + a memory entry for future sessions.

## When this skill applies

Trigger conditions:
- The user asks for "extensive planning" / "research and plan" / "design with grounding"
- The work introduces a new language feature, library suite, or substantial system
- The user explicitly says "research as much as you need"
- The user grants plan-mode + a clear "no codebase touches" until approved

Trigger anti-conditions (don't use this skill):
- Bug fix in a single file
- Feature that fits in <100 LOC
- Pure exploration ("what does X do?") without a design output
- The user is in active debugging — different workflow

The discipline of this skill is **costly**. ~3-5 parallel agent calls,
~8-15 web searches, ~12-20 documentation files. Worth it for landmark
work; overkill for routine work.

## The two-burst structure

This workflow has **two distinct bursts** separated by `ExitPlanMode`:

```
Burst 1 — PLAN (in plan mode, codebase read-only)
  ├── frame the meta-plan
  ├── parallel codebase exploration
  ├── synthesise + stress-test the design
  ├── refine with user redirects
  ├── lock 2-3 load-bearing decisions
  └── ExitPlanMode → create doc tree + memory entry

Burst 2 — RESEARCH (post-approval, web allowed)
  ├── parallel web research
  ├── capture findings under Research/ subdir
  ├── fold findings back into design docs
  └── mark traceability on each applied finding
```

The seam matters: `ExitPlanMode` is **not** the end of the work; it's
the boundary between the two bursts. The user approves the plan; *then*
research grounds it; *then* the design docs absorb the research.

## Phase 1 — Frame the meta-plan (~1 message)

Before any tool calls, write a short user-facing message describing
**how you will plan**. Five-phase outline (these are your phases, not
the user's):

> Phase 1 — Map the ground (parallel Explore agents)
> Phase 2 — Synthesise the load-bearing decision
> Phase 3 — Library / module / module decomposition
> Phase 4 — Integration story + "things you may not have covered"
> Phase 5 — Lock decisions (AskUserQuestion) + ExitPlanMode

Then immediately:
1. Create the plan file at the plan-mode path (a skeleton with all the
   section headers you'll fill in).
2. Fire 2-3 Explore agents in parallel for codebase exploration.
3. Pre-load deferred tool schemas you'll need at the end (`AskUserQuestion`,
   `ExitPlanMode`) via `ToolSearch select:...`.

Don't dump the plan into chat. The plan lives in the plan file.

## Phase 2 — Parallel codebase exploration

**Use the Explore agent type, up to 3 in parallel, single message.**

Each Explore agent gets a *distinct* brief — different areas of the
codebase, different concerns. Bad: three agents searching for the same
thing. Good: one for type system, one for math / physics infra, one
for build conventions.

Each brief should include:
- **Plain-language context** — what you're planning and why
- **Specific files / paths** to investigate
- **Concrete questions** (numbered) rather than "tell me everything"
- **Word-count cap** ("under ~1500 words")
- **What format** you want findings in ("structured map with paths + line numbers")
- **What to skip** (memory entries you already have, things already known)

While the Explore agents run, draft your initial design intuition in
the plan file. Don't wait passively.

## Phase 3 — Synthesise + stress-test

Once Explore agents return:

1. **Read the critical files** they identified — header type enums,
   integration-seam source, anything you'll cite by file:line.
2. **Launch a Plan agent** for an opinionated second opinion.

The Plan agent gets a different shape of prompt than Explore:
- **Stipulated context** (the verified findings from Explore) — don't
  make Plan re-explore
- **The decision points you want stress-tested**, numbered 1-N
- **Your current lean** on each, so Plan can confirm or challenge
- **"Be opinionated. Be a peer."** — explicit invitation to push back
- **Word-count cap** (~2500 words)
- **What you want back**: per-item verdict (confirm / challenge / refine) + reasoning + concrete proposed changes

The Plan agent will surface things you missed. Some will be right;
some will be wrong for your user's actual framing. **You judge.** The
Plan agent doesn't see the conversation, so it can't know that
(e.g.) the user redirected mid-planning. Trust-but-verify Plan agent
output — its summary describes what it argued, not necessarily what's
correct for this context.

## Phase 4 — Refine with user redirects

The user may redirect mid-planning. When they do:

- **Acknowledge the redirect explicitly** ("flipping the centre of gravity")
- **Re-evaluate the Plan agent's advice** — some may now be wrong for the new framing
- **Rewrite the plan file** to reflect the new framing, not patch around the old one
- Keep refinements from earlier phases that still apply; discard the rest

This is the phase where the plan grows from a draft to a real artifact.

## Phase 5 — Lock decisions via AskUserQuestion

`AskUserQuestion` is for **2-3 truly load-bearing decisions**, never
for "is this plan OK?" That's what `ExitPlanMode` is for.

Good AskUserQuestion candidates:
- Determinism default (portable vs fast)
- Ship strategy (bundled subset vs external-import-only)
- Scope of an optional feature (off-by-default vs included vs excluded)

Bad AskUserQuestion candidates:
- "Does this plan look good?" → that's ExitPlanMode
- "Which name should we use?" → propose one; let the user redirect
- Anything you could decide yourself with the user-redirect-clause as backup

Question shape:
- 2-4 options per question (max)
- First option is your recommendation, marked `(Recommended)`
- Each option has a clear description of what it commits to
- Questions are self-contained — the user can't see the plan file at
  this point, so phrase questions so they make sense without context

After answers: update the plan's "Decisions" section to lock them
verbatim with the date. Then call `ExitPlanMode`.

## Phase 6 — Create the doc tree

After ExitPlanMode, the plan file at `~/.claude/plans/...` is the
approval record. The **canonical going-forward docs** live in a fresh
subdirectory of the project's `Docs/` (or equivalent), structured as:

```
Docs/Plan_<Topic>/
├── 00-index.md              — read order + decisions-locked + at-a-glance
├── 01-context.md            — why this exists; problem statement
├── 02-<spine.md>            — the load-bearing decision (depends on topic)
├── 03-...                   ...
├── NN-roadmap.md            — landing order by LOC + deps + risk
└── Research/                — created in Burst 2; empty at this point
```

Aim for **10-16 numbered topic files**. Each:
- Has a clear, single-concern title
- Cross-references siblings (`NN-name.md` / `(NN-name.md#section)`)
- Includes file:line citations into the codebase
- Includes a "File map" section showing what source files would land
- Avoids timescale framing (use LOC + dependency + risk per
  `feedback_no_timescale_estimates.md` if available)

Write in parallel batches of 4-5 `Write` calls per message.

## Phase 7 — Memory save

For substantial planning work, save a project memory:

- **Path:** `~/.claude/projects/<encoded-path>/memory/project_plan_<topic>.md`
- **Type:** `project`
- **Body:** what the plan is, where the docs live, the decisions locked,
  the headline architectural moves, the load-bearing files cited

Then add a line to that project's `MEMORY.md` index pointing at it.

Why this matters: future conversations need to be able to pick up the
work without re-deriving it. The memory entry is the resume cue.

## Phase 8 — Web research (Burst 2)

This phase only happens when the user explicitly invites it. The user
will typically say something like "feel free to research while I'm
away" or "ground this in current literature".

### Research-only invariants
- Codebase still untouched (the no-touch rule from plan mode extends
  through research)
- All findings go to `Docs/Plan_<Topic>/Research/`
- Each finding has citations (URLs)
- Findings are not silently applied — they're captured first

### Query strategy

Plan **batches** of 4 parallel WebSearch queries per message. Three to
four batches total covers most plan-sized research needs (~12-16
searches). Group by topic:

- Batch 1 — foundational facts (formats, sizes, current standards)
- Batch 2 — comparable systems / state-of-the-art
- Batch 3 — adjacent / future-work references
- Batch 4 — anything specific the earlier batches surfaced

Use `WebFetch` for specific URLs the searches surface when the search
summary isn't enough. Don't fetch every URL — search summaries are
usually sufficient.

Use the **current year** in queries (per the `WebSearch` tool's own
guidance). Outdated information is worse than no information.

### Research doc shape

```
Research/
├── 00-index.md                   — overview + headline findings
├── 01-<topic-A>.md
├── 02-<topic-B>.md
...
```

Each topic file:
- **Headline numbers / facts** in a table near the top
- **Plan implications** — does this confirm or refine an existing
  design call?
- **Sources** — markdown links at the bottom, one per URL
- Aim for ~100-150 lines each

The `Research/00-index.md` has two sections that earn their keep:
- **Would-change-the-plan items** — numbered list of findings that
  refine the design
- **Confirmations (plan stands)** — findings that validate existing
  decisions

## Phase 9 — Apply research findings

When the user approves applying findings (or invites it explicitly):

1. For each "would-change-the-plan" item, make a **targeted Edit** to
   the relevant design doc.
2. Keep edits surgical — find the exact paragraph, splice in the new
   reference, don't rewrite surrounding text.
3. Use parallel `Edit` calls (different files are independent).
4. **Mark applied** in `Research/00-index.md` with a checkmark, the
   file edited, and a one-line note.

The trail matters: a reader six months later can see what was applied,
when, and why.

## Discipline notes

- **No emojis.** Default for technical documentation; reverse only if
  the user explicitly requests them.
- **No timescale framing.** Don't size work in days/weeks unless the
  project explicitly uses calendar tracking. LOC + dependency + risk.
- **No "v0.X will fix X" deferral promises.** If something's deferred,
  describe what blocks it (a dependency, a missing feature, a use
  case), not when. Apply per `feedback_no_version_deferrals.md` if the
  project has that feedback memory.
- **File-line citations everywhere.** `path/to/file.c:123` makes plans
  reviewable; vague references rot.
- **Parallel tool calls when independent.** Multiple `Write` to
  different files, multiple `WebSearch`, multiple Explore agents — all
  in one message. Sequential only when there's a real dependency.
- **Don't fork load-bearing libs.** If a library needs one new feature,
  extend it additively (new enum value, new optional flag). Forking
  invites drift.
- **Three agents max per phase.** Quality > quantity. If you need a
  fourth agent's perspective, ask whether you actually need it or
  whether you're spraying.
- **Match the project's existing doc conventions.** Look at sibling
  `Docs/Plan_*` directories if they exist. Match their style.
- **Don't narrate.** Update the plan file; don't recite it back in chat.

## Adaptation guidance

This skill scales down by skipping phases, not by skipping rigour:

- **Tight scope** (~1 lib, ~1 KFL feature): one Explore agent, no Plan
  agent, ~6-8 doc files. Skip the research phase unless invited.
- **Medium scope** (a feature touching 3-4 libs): two Explore agents,
  one Plan agent, ~10 doc files. Optional research phase.
- **Large scope** (this skill's design target — solar-system astro
  suite, language version, etc.): three Explore agents, one Plan agent,
  16+ doc files, full research phase, memory entry.

The phases stay in order; their depth varies. Never collapse the
plan/research seam — Burst 1 must finish before Burst 2 starts.

## Output the user sees

- A short opener describing your plan (Phase 1 message)
- Periodic 1-2 sentence updates ("synthesising 3 Explore findings",
  "writing the runtime doc")
- The `AskUserQuestion` for load-bearing decisions
- The `ExitPlanMode` for plan approval
- A short summary after each major milestone (plan landed, docs landed,
  research landed, revisions landed)

Nothing else. The artifacts are the work; the chat is the interface.

## Common failure modes

- **Asking too many questions.** Cap at 3 in `AskUserQuestion`. The
  rest gets recommended in the plan with the user-redirect clause.
- **Dumping the plan into chat.** Don't. The plan file is the artifact.
- **Letting the Plan agent decide.** It can't see the conversation.
  Use it as input; *you* decide.
- **Doing research before planning.** You research the wrong things
  because you don't yet know what's load-bearing. Plan first, research
  second.
- **Silently applying research.** Always mark what was applied + why +
  in which file. Future-you needs the trail.
- **Building the doc tree before approval.** Plan mode forbids it.
  Wait for ExitPlanMode.
- **Forgetting the memory entry.** Future sessions need the resume
  cue. Save it.
