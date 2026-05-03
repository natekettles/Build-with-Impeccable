---
name: build
description: Feature development pipeline - research, plan, track, and implement major features.
argument-hint: "[subcommand] [name]"
metadata:
  author: Shpigford
  version: "1.1"
---

# build

Feature development pipeline - research, plan, track, and implement major features.

## Instructions

This command manages a 4-phase feature development workflow for building major features. Parse `$ARGUMENTS` to determine which subcommand to run.

**Arguments provided:** $ARGUMENTS

### Auto-mode override

This skill overrides auto-mode's "prefer assumptions over questions" guidance. The whole point of `/build` is to validate assumptions before writing code — silent assumptions are exactly what this workflow exists to prevent.

Rules when auto-mode is active:
- You MUST still call AskUserQuestion at every decision point this skill prescribes (feature name, scope, phase size, ambiguities, existing-doc handling, etc.).
- To keep it cheap: always pre-compute your recommendation and label that option `Recommended: <choice>` in the AskUserQuestion `label` field. Put your reasoning in the `description`. The user can then one-click through.
- Routine mechanical work (reading files, running searches, writing the doc once decisions are made) still proceeds without asking — auto-mode applies there as normal.
- If a decision is genuinely low-stakes and reversible (e.g. minor wording in a doc you're about to write), you may skip the Ask. Anything that shapes scope, approach, or phase boundaries must be asked.

### Argument Parsing

Parse the first word of $ARGUMENTS to determine the subcommand:

- `product [name]` → Run the full design+build orchestrator (shape → research → implementation → progress → phases)
- `research [name]` → Run the Research phase
- `implementation [name]` → Run the Implementation phase
- `progress [name]` → Run the Progress phase
- `phase [n] [name]` → Run Phase n of the implementation
- `status [name]` → Show current status and suggest next step
- (empty or unrecognized) → Show usage help

If the feature name is not provided in arguments, you MUST use AskUserQuestion to prompt for it.

---

## Subcommand: Help (empty args)

If no arguments provided, display this help:

```
/build - Feature Development Pipeline

Subcommands:
  /build product [name]         End-to-end orchestrator: shape brief → research → plan → phases
  /build research [name]        Deep research on a feature idea
  /build implementation [name]  Create phased implementation plan
  /build progress [name]        Set up progress tracking
  /build phase [n] [name]       Execute implementation phase n
  /build status [name]          Show status and next steps

Recommended workflow (UI-bearing features):
  /build product chat-interface

  Runs /impeccable shape (saves BRIEF.md), then research, implementation,
  progress, and walks you through phases. UI phases auto-invoke the
  impeccable mock + browser-iteration loop for production-grade craft.

Manual workflow (existing pipeline still works):
  /build research chat-interface
  /build implementation chat-interface
  /build progress chat-interface
  /build phase 1 chat-interface
```

Then use AskUserQuestion to ask what they'd like to do:

- question: "What would you like to do?"
- header: "Action"
- multiSelect: false
- options:
  - label: "Run the full product orchestrator (Recommended)"
    description: "Run /build product — shape brief, then research, plan, and phased build"
  - label: "Start new feature research"
    description: "Begin deep research on a new feature idea"
  - label: "Continue existing feature"
    description: "Work on a feature already in progress"
  - label: "Check status"
    description: "See what step to do next for a feature"

---

## Subcommand: product

End-to-end orchestrator that pairs the **impeccable** plugin's design rigor with this skill's phased build pipeline. This is the recommended entry point for any UI-bearing feature.

### Why this exists

`/impeccable shape` produces an excellent design brief. `/impeccable craft` builds beautifully but in one monolithic push — no phased plan, no technical-research artifact, no progress tracking across sessions. This pipeline keeps shape's brief, then uses the build pipeline (RESEARCH → IMPLEMENTATION → PROGRESS → phases) as the structural spine, and re-introduces craft's mock + browser-iteration loop inside each UI phase.

### Step 1: Get Feature Name

If feature name not in arguments, use AskUserQuestion to prompt for it (same shape as research phase).

### Step 2: Shape the Design Brief

Invoke the **impeccable** skill's shape command on the user's behalf — call it as `/impeccable shape {name}` (or, if running in a harness where slash commands cannot be re-entered, follow the procedure in `impeccable/.claude/skills/impeccable/reference/shape.md`: discovery interview, optional visual probes, design brief, explicit user confirmation).

**Wait for explicit user confirmation of the brief.** Do not proceed until confirmed. PRODUCT.md, DESIGN.md, or a self-authored summary do NOT count as a confirmed brief. If the user disagrees with any part, loop back through shape's discovery questions.

### Step 3: Persist the Brief

Once the brief is confirmed, write it verbatim to `docs/{name}/BRIEF.md` with this structure:

```markdown
# {Feature Name} Design Brief

> Confirmed by user on {YYYY-MM-DD}. Source: /impeccable shape.

## 1. Feature Summary
## 2. Primary User Action
## 3. Design Direction
[Color strategy + theme scene sentence + 2-3 anchor references]
## 4. Scope
[Fidelity / breadth / interactivity / time intent]
## 5. Layout Strategy
## 6. Key States
## 7. Interaction Model
## 8. Content Requirements
## 9. Recommended References
[List of impeccable reference files: spatial-design.md, typography.md, motion-design.md, etc.]
## 10. Open Questions
## 11. Register
[brand or product — copy from PRODUCT.md if present, infer otherwise]
```

The persisted BRIEF.md satisfies impeccable's `shape=pass` gate on future sessions and gives later phases a stable artifact to read.

### Step 4: Run Research

Invoke the `/build research {name}` flow (the full Research subcommand below). Research now reads BRIEF.md first and grounds technical research in the brief's register, anchor references, scope, and anti-goals.

### Step 5: Run Implementation Planning

Invoke the `/build implementation {name}` flow. Each phase is tagged `[ui]` or `[non-ui]` based on whether it touches user-facing surfaces. Show the tagged plan to the user and let them override any tag before approving.

### Step 6: Run Progress Setup

Invoke the `/build progress {name}` flow. The progress doc's Quick Reference block now includes a link to BRIEF.md.

### Step 7: Phase Walkthrough

For each phase in order:

1. Run `/build phase {n} {name}`.
2. After the phase completes, ask the user via AskUserQuestion whether to continue to the next phase, pause, or revise the implementation plan.

UI phases auto-invoke the impeccable craft mock + browser-iteration loop (see Phase subcommand, Step 6 below). No prompt — it just happens. Non-UI phases skip the impeccable loop and run as the existing build pipeline.

### Step 8: Done

When all phases are complete, surface the standard completion summary plus a final reminder to run `/impeccable polish {name}` or `/impeccable audit {name}` for a last-pass quality check before shipping.

---

## Subcommand: research

### Step 1: Get Feature Name

If feature name not in arguments, use AskUserQuestion:

- question: "What's a short identifier for this feature? (lowercase, hyphens ok - e.g., 'chat-interface', 'user-auth', 'data-export'). Use 'Other' to type it."
- header: "Feature name"
- multiSelect: false
- options:
  - label: "I'll type the name"
    description: "Enter a short, kebab-case identifier for the feature"

### Step 2: Check for Existing Research

Check if `docs/{name}/RESEARCH.md` already exists.

If it exists, use AskUserQuestion:

- question: "A RESEARCH.md already exists for this feature. What would you like to do?"
- header: "Existing doc"
- multiSelect: false
- options:
  - label: "Overwrite"
    description: "Replace existing research with fresh exploration"
  - label: "Append"
    description: "Add new research below existing content"
  - label: "Skip"
    description: "Keep existing research, suggest next step"

If "Skip" selected, suggest running `/build implementation {name}` and exit.

### Step 3: Read BRIEF.md if it exists

If `docs/{name}/BRIEF.md` exists, read it before gathering further context. The brief carries the user's confirmed answers about purpose, audience, scope, design direction, key states, and anti-goals — use them to shortcut the discovery questions in Step 4 and to ground every research decision (architecture choices, library shortlists, integration points) in the brief's register, anchor references, and constraints.

If BRIEF.md does not exist, this research run is operating without a confirmed design brief. That is allowed (for backend-only features it's fine), but if the feature has a UI surface, suggest once: *"This feature has a UI surface — consider running `/build product {name}` instead so the research is grounded in a confirmed shape brief."*

### Step 4: Gather Feature Context

Use AskUserQuestion to understand the feature. Skip questions already answered by BRIEF.md.

- question: "Describe the feature you want to build. What problem does it solve? What should it do? (Use 'Other' to describe)"
- header: "Description"
- multiSelect: false
- options:
  - label: "I'll describe it"
    description: "Provide a detailed description of the feature"

### Step 5: Research Scope

Use AskUserQuestion:

- question: "What aspects should the research focus on?"
- header: "Focus areas"
- multiSelect: true
- options:
  - label: "Technical implementation"
    description: "APIs, libraries, architecture patterns"
  - label: "UI/UX design"
    description: "Interface design, user flows, interactions"
  - label: "Data requirements"
    description: "What data to store, schemas, privacy"
  - label: "Platform capabilities"
    description: "OS APIs, system integrations, permissions"

If BRIEF.md is present, skip this question for any focus area the brief already covers (UI/UX is covered by Sections 3, 5, 6, 7 of the brief; data requirements may be covered by Section 8). Use the brief's content directly rather than re-asking.

### Step 6: Conduct Deep Research

Now conduct DEEP research on the feature:

1. **Codebase exploration**: Understand existing patterns, similar features, relevant code
2. **Web search**: Research best practices, similar implementations, relevant APIs
3. **Technical deep-dive**: Explore specific technologies, libraries, frameworks
4. **Use AskUserQuestion FREQUENTLY**: Validate assumptions, clarify requirements, get input on decisions

Research should cover:
- Problem definition and user needs
- Technical approaches and trade-offs
- Required data models and storage
- UI/UX considerations
- Integration points with existing code
- Potential challenges and risks
- Recommended approach with rationale

### Step 7: Write Research Document

Create the directory if needed: `docs/{name}/`

Write findings to `docs/{name}/RESEARCH.md` with this structure:

```markdown
# {Feature Name} Research

## Overview
[Brief description of the feature and its purpose]

## Problem Statement
[What problem this solves, why it matters]

## User Stories / Use Cases
[Concrete examples of how users will use this]

## Technical Research

### Approach Options
[Different ways to implement this, with pros/cons]

### Recommended Approach
[The approach you recommend and why]

### Required Technologies
[APIs, libraries, frameworks needed]

### Data Requirements
[What data needs to be stored/tracked]

## UI/UX Considerations
[Interface design thoughts, user flows]

## Integration Points
[How this connects to existing code/features]

## Risks and Challenges
[Potential issues and mitigation strategies]

## Open Questions
[Things that still need to be decided]

## References
[Links to relevant documentation, examples, articles]

## Brief Alignment
[If BRIEF.md exists: confirm how each Recommended Reference, anchor reference, and key state from the brief is reflected in the research above. Note any deviations and why.]
```

### Step 8: Next Step

After writing the research doc, inform the user:

"Research complete! Document saved to `docs/{name}/RESEARCH.md`

**Next step:** Run `/build implementation {name}` to create a phased implementation plan."

---

## Subcommand: implementation

### Step 1: Get Feature Name

If feature name not in arguments, use AskUserQuestion to prompt for it (same as research phase).

### Step 2: Verify Research Exists

Check if `docs/{name}/RESEARCH.md` exists.

If it does NOT exist:
- Inform user: "No research document found at `docs/{name}/RESEARCH.md`"
- Suggest: "Run `/build research {name}` first to create the research document."
- Exit

### Step 3: Check for Existing Implementation Doc

Check if `docs/{name}/IMPLEMENTATION.md` already exists.

If it exists, use AskUserQuestion:

- question: "An IMPLEMENTATION.md already exists. What would you like to do?"
- header: "Existing doc"
- multiSelect: false
- options:
  - label: "Overwrite"
    description: "Create a fresh implementation plan"
  - label: "Append"
    description: "Add new phases below existing content"
  - label: "Skip"
    description: "Keep existing plan, suggest next step"

If "Skip" selected, suggest running `/build progress {name}` and exit.

### Step 4: Read Research and Brief

Read `docs/{name}/RESEARCH.md` to understand:
- The recommended approach
- Technical requirements
- Data models needed
- UI/UX design
- Integration points

If `docs/{name}/BRIEF.md` exists, read it too. Use the brief's Key States, Layout Strategy, Interaction Model, and Recommended References to inform phase boundaries. A phase that delivers a brief-defined state (default / empty / loading / error / success / edge case) is usually a good shippable unit.

### Step 5: Design Implementation Phases

Break the research into practical implementation phases. Each phase should:
- **Deliver something functional and testable** — the user should be able to go use/test what was built after each phase. Combine infrastructure work with the UI or functionality that uses it so every phase produces a shippable result. Never stack multiple infrastructure-only phases in a row.
- Be independently valuable (deliver something usable)
- Be small enough to complete in a focused session
- Build on previous phases
- Have clear success criteria
- **Carry a `[ui]` or `[non-ui]` tag.** A phase is `[ui]` if it touches any user-facing surface (a screen, component, copy, motion, or interaction). It is `[non-ui]` only if every task is backend, infrastructure, schema, or tooling with no rendered surface. When in doubt, tag it `[ui]`. The phase runner uses these tags to decide whether to invoke the impeccable mock + browser-iteration loop.

Use AskUserQuestion to validate phase breakdown:

- question: "How granular should the implementation phases be?"
- header: "Phase size"
- multiSelect: false
- options:
  - label: "Small phases (1-2 hours)"
    description: "Many focused phases, easier to track progress"
  - label: "Medium phases (half day)"
    description: "Balanced approach, moderate number of phases"
  - label: "Large phases (full day)"
    description: "Fewer phases, each delivering significant functionality"

### Step 6: Conduct Phase Research

For each phase you're planning, do targeted research:
- Web search for implementation specifics
- Review relevant code in the codebase
- Identify dependencies between phases

Use AskUserQuestion for any uncertainties about phase ordering or scope.

### Step 7: Write Implementation Document

Write to `docs/{name}/IMPLEMENTATION.md` with this structure:

```markdown
# {Feature Name} Implementation Plan

## Overview
[Brief recap of what we're building and the approach from research]

## Prerequisites
[What needs to be in place before starting]

## Phase Summary
[Quick overview of all phases]

---

## Phase 1: [Phase Title] [ui|non-ui]

### Objective
[What this phase accomplishes]

### Rationale
[Why this phase comes first, what it enables]

### Tasks
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

### Success Criteria
[How to verify this phase is complete]

### Files Likely Affected
[List of files that will probably need changes]

---

## Phase 2: [Phase Title] [ui|non-ui]

[Same structure as Phase 1]

---

[Continue for all phases]

---

## Post-Implementation
- [ ] Documentation updates
- [ ] Testing strategy
- [ ] Performance validation

## Notes
[Any additional context or decisions made during planning]
```

### Step 8: Show Phase Tags for Override

After writing the implementation doc, surface the phase list with their `[ui|non-ui]` tags and ask the user via AskUserQuestion whether any tag should be flipped before approving the plan. Example:

```
Phase 1: Data model + ingest pipeline           [non-ui]
Phase 2: Listing screen with filters            [ui]
Phase 3: Detail screen with state coverage      [ui]
Phase 4: Bulk export endpoint                   [non-ui]
```

If the user requests a flip, update IMPLEMENTATION.md and re-show.

### Step 9: Next Step

Inform the user:

"Implementation plan complete! Document saved to `docs/{name}/IMPLEMENTATION.md`

**Next step:** Run `/build progress {name}` to set up progress tracking."

---

## Subcommand: progress

### Step 1: Get Feature Name

If feature name not in arguments, use AskUserQuestion to prompt for it.

### Step 2: Verify Implementation Doc Exists

Check if `docs/{name}/IMPLEMENTATION.md` exists.

If it does NOT exist:
- Inform user: "No implementation document found at `docs/{name}/IMPLEMENTATION.md`"
- Suggest: "Run `/build implementation {name}` first."
- Exit

### Step 3: Check for Existing Progress Doc

Check if `docs/{name}/PROGRESS.md` already exists.

If it exists, use AskUserQuestion:

- question: "A PROGRESS.md already exists. What would you like to do?"
- header: "Existing doc"
- multiSelect: false
- options:
  - label: "Overwrite"
    description: "Start fresh progress tracking"
  - label: "Keep existing"
    description: "Keep current progress, suggest next step"

If "Keep existing" selected, read the progress doc and suggest the next incomplete phase.

### Step 4: Read Implementation Document

Read `docs/{name}/IMPLEMENTATION.md` to extract:
- All phase titles
- Tasks within each phase
- Success criteria

### Step 5: Create Progress Document

Write to `docs/{name}/PROGRESS.md` with this structure:

```markdown
# {Feature Name} Progress

## Status: Phase 1 - Not Started

## Quick Reference
- Brief: `docs/{name}/BRIEF.md` (if present — confirmed design brief from /impeccable shape)
- Research: `docs/{name}/RESEARCH.md`
- Implementation: `docs/{name}/IMPLEMENTATION.md`

---

## Phase Progress

### Phase 1: [Title from Implementation] [ui|non-ui]
**Status:** Not Started

#### Tasks Completed
- (none yet)

#### Decisions Made
- (none yet)

#### Blockers
- (none)

---

### Phase 2: [Title]
**Status:** Not Started

[Same structure]

---

[Continue for all phases]

---

## Session Log

### [Date will be added as work happens]
- Work completed
- Decisions made
- Notes for next session

---

## Files Changed
(Will be updated as implementation progresses)

## Architectural Decisions
(Major technical decisions and rationale)

## Lessons Learned
(What worked, what didn't, what to do differently)
```

### Step 6: Next Step

After creating progress doc:

"Progress tracking set up! Document saved to `docs/{name}/PROGRESS.md`

**Next step:** Run `/build phase 1 {name}` to begin implementation."

---

## Subcommand: phase

### Step 1: Parse Arguments

Parse arguments to extract:
- Phase number (if provided)
- Feature name (if provided)

If neither provided, prompt for both using AskUserQuestion.

### Step 2: Get Feature Name

If feature name not determined, use AskUserQuestion to prompt for it.

### Step 3: Verify All Docs Exist

Check that all three docs exist:
- `docs/{name}/RESEARCH.md`
- `docs/{name}/IMPLEMENTATION.md`
- `docs/{name}/PROGRESS.md`

If any missing, inform user which doc is missing and suggest the appropriate `/build` command to create it.

### Step 4: Get Phase Number

If phase number not in arguments:

Read `docs/{name}/IMPLEMENTATION.md` to extract available phases.

Use AskUserQuestion to let user select:

- question: "Which phase would you like to work on?"
- header: "Phase"
- multiSelect: false
- options: [dynamically generated from phases found in IMPLEMENTATION.md, marking completed ones]

### Step 5: Read All Context

Read all available documents to fully understand:
- `docs/{name}/BRIEF.md` if it exists (the confirmed design brief — register, anchor references, key states, recommended impeccable references)
- `docs/{name}/RESEARCH.md` (research and rationale)
- `docs/{name}/IMPLEMENTATION.md` (the specific phase tasks, success criteria, and the phase's `[ui|non-ui]` tag)
- `docs/{name}/PROGRESS.md` (current progress and decisions made)

Note the phase's `[ui|non-ui]` tag from IMPLEMENTATION.md. It controls Step 6 below.

### Step 6: Deep Research on Phase

Before writing any code, conduct thorough technical research:

1. **Web search (required — use Agent tool)**: Launch sub-agents with `WebSearch` to research specific implementation details for this phase. Search for:
   - Best practices and common patterns for the specific tech/approach
   - Known pitfalls, gotchas, and edge cases
   - Recent changes to APIs or libraries you'll be using
   - Run multiple searches in parallel covering different aspects of the phase

2. **Documentation lookup (required — use Context7 MCP)**: Use `mcp__context7__resolve-library-id` and `mcp__context7__query-docs` to fetch current documentation for any libraries, frameworks, or APIs involved in this phase. Do NOT rely on training data for library specifics — docs change. Look up:
   - API signatures and configuration options you'll be using
   - Migration guides if upgrading or integrating new versions
   - Setup/integration instructions for any new dependencies

3. **Codebase exploration**: Use the Agent tool (Explore type) to understand relevant existing code — patterns, conventions, and integration points for this phase.

4. **Use AskUserQuestion** to clarify any ambiguities about the phase requirements.

### Step 6.5: Impeccable Loop (UI phases only)

**Skip this entire step for `[non-ui]` phases.** For `[ui]` phases, run automatically without prompting:

1. **Load impeccable context.** Read PRODUCT.md and DESIGN.md from the project root if they exist. If PRODUCT.md is missing, empty, or placeholder, suggest the user run `/impeccable teach` and pause. Identify the register (brand vs product) — copy from BRIEF.md Section 11 if present, otherwise infer from PRODUCT.md.

2. **Confirm brief gate.** If `docs/{name}/BRIEF.md` does NOT exist for this feature, do not proceed with the impeccable loop — instead, suggest: *"This is a UI phase but no BRIEF.md exists. Run `/impeccable shape {name}` to produce a confirmed brief, save it as docs/{name}/BRIEF.md, then resume this phase."* Then pause.

3. **Load named impeccable references.** Open and read each reference file listed in BRIEF.md Section 9 ("Recommended References"). At minimum always consult `spatial-design.md` and `typography.md`. Reference files live at `impeccable/.claude/skills/impeccable/reference/<name>.md` — check both that path and `~/.claude/skills/impeccable/reference/` and any harness-installed copy. Apply the shared design laws (color strategy, theme via scene sentence, typography, layout, motion, absolute bans, copy rules) from `impeccable/.claude/skills/impeccable/SKILL.md`.

4. **North-star mock (capability-gated).** If the brief's Scope is mid-fi, high-fi, or production-ready AND the harness has built-in image generation (e.g. Codex `image_gen`, the `image-gen` skill, or equivalent), generate 1–3 high-fidelity north-star comps for this phase's surface, get user approval on direction, then build a **mock fidelity inventory** of major visible ingredients (hero silhouette, signature motifs, nav/CTA treatment, section sequence, typography, density, color treatment, motion cues). For each ingredient, decide implementation: semantic HTML/CSS/SVG, generated asset, sourced asset, icon library, canvas/WebGL, or accepted omission. If image generation is unavailable, state that in one line and skip — do NOT ask the user to install APIs or tooling.

5. **Production bar — apply during Step 7 build.** Real or realistic content. Preserve the mock's major ingredients. Semantic-first markup. Calibrated spacing. Intentional typography. Realistic state coverage (default, hover, focus-visible, active, disabled, loading, error, success, empty, overflow, long text, short text, first-run). Accessible interactions. Coherent icons. Optimized media. Premium motion. Maintainable code. (Full checklist in `impeccable/.claude/skills/impeccable/reference/craft.md` Step 5.)

### Step 7: Execute Phase Work

Begin implementing the phase:

1. Work through each task in the phase.
2. For `[ui]` phases, hold the Step 6.5 production bar throughout — not as a final pass, but during initial implementation.
3. Use AskUserQuestion frequently for implementation decisions.
4. Follow the "Always Works" philosophy — test as you go.
5. Document decisions in PROGRESS.md as you make them.

### Step 7.5: Runtime / Visual Verification (mandatory before marking complete — all phases)

"Done" means observed-working, not "I wrote the code that should do it." Tool-success messages and PostToolUse hook confirmations tell you a file was written, not that it renders or runs correctly. Before moving to Step 7.6 (UI phases) or Step 8 (non-UI phases), you must independently verify the change at runtime — through whichever of these channels is appropriate for the phase:

1. **User-facing UI changes** — render the affected surface and capture a screenshot. Pick the lightest-weight tool that actually shows the rendered result:
   - Web pages / dev servers → `mcp__Claude_in_Chrome__navigate` + `mcp__Claude_in_Chrome__read_page` or `mcp__Claude_Preview__preview_start` + `mcp__Claude_Preview__preview_screenshot`. If those MCPs aren't available, fall through to the `browser-harness` skill or headless Playwright.
   - Native macOS / iOS apps → build, launch, and capture via `mcp__computer-use__screenshot` (request access first). For iOS, use the simulator. For Mac, run the actual app.
   - HTML artifacts / docs / decks / exported files → open the file and inspect the rendering, confirming any external resources (images, stylesheets, fetched data) actually resolve in the rendering context, not just on disk.
   - CLI / API / library work → execute the affected command or hit the endpoint, capture stdout/stderr, and confirm the observable output matches the phase's success criteria.
2. **Tests** — for any phase whose success criteria reference automated tests, run the relevant test target (unit, integration, snapshot, screenshot). Paste the actual pass/fail count into PROGRESS.md, not a paraphrase.
3. **Both light and dark mode** when the phase touches visual design — a screenshot in only one theme is half a verification.
4. **Console / log inspection** when a runtime is involved — sample the browser console (`mcp__Claude_in_Chrome__read_console_messages`), the preview console logs (`mcp__Claude_Preview__preview_console_logs`), or platform-equivalent log output for unhandled errors.

If verification is genuinely impossible (no browser MCP available, no simulator, can't run the affected code in this environment), state that explicitly in PROGRESS.md: *"Built but unverified — reason: ..."* — and surface it in the Step 9 next-step message so the user knows what's outstanding. Never imply success you haven't observed.

Record what you verified directly in PROGRESS.md under a **Verification** subsection for the phase: which channel(s) you used, the result, and a path/URL to any screenshot saved.

### Step 7.6: Browser-Iteration Loop (UI phases only)

**Skip this entire step for `[non-ui]` phases.** For `[ui]` phases, after Step 7.5 has confirmed the result is rendering and running, run the design-quality critique:

1. **Required viewport pass.** Re-capture the rendered state at mobile narrow, tablet/small laptop, and desktop wide (or whatever viewports the brief specifies). Step 7.5's screenshots may have only covered one viewport.

2. **Critique-and-fix loop.** Write a short critique for yourself, patch the implementation, re-inspect. Continue until no material issues remain against this checklist:
   - Does the live result match BRIEF.md (every section)?
   - Does it match the approved mock's fidelity inventory? Missing major ingredients are P0 defects.
   - Does it pass the **AI slop test**? If someone could say "AI made this" without doubt, it needs more design intention. Run the category-reflex check: if someone could guess theme + palette from the category alone, rework.
   - Does it violate any of the absolute bans (side-stripe borders, gradient text, glassmorphism as default, hero-metric template, identical card grids, modal as first thought)?
   - Are all states intentional? (empty, error, loading, edge cases)
   - Does it adapt compositionally across viewports, not just shrink?
   - Spacing consistency, optical alignment, type hierarchy, contrast, image quality, icon coherence, focus treatment, motion timing — all deliberate?
   - Performance basics — no oversized images, no avoidable layout thrash, no blocking animations?

The exit bar is not "it works." It is: the rendered result looks intentional at all checked viewports, every expected state is handled, no placeholders remain unless explicitly accepted, and the implementation would be defensible in a high-end studio review.

At least one critique-and-fix pass is required after the first browser inspection, unless the first pass has no material defects.

### Step 8: Update Progress Document

As you work, update `docs/{name}/PROGRESS.md`:

- Mark tasks as completed
- Record decisions made and why
- Note any blockers encountered
- List files changed
- Add architectural decisions
- Add the **Verification** subsection produced in Step 7.5 (channels used, result, screenshot/log paths)
- Update the session log with today's work

Update the phase status:
- "In Progress" when starting
- "Completed" when all tasks done, success criteria met, **and Step 7.5 verification has been performed and recorded**

### Step 9: Next Step

After completing the phase:

1. Read PROGRESS.md to determine next incomplete phase
2. Inform user of completion and suggest next action:

"Phase {n} complete! Progress updated in `docs/{name}/PROGRESS.md`

**Next step:** Run `/build phase {n+1} {name}` to continue with [next phase title]."

Or if all phases complete:

"All phases complete! The {feature name} feature implementation is done.

Consider:
- Running tests to verify everything works
- Updating documentation
- Creating a PR for review"

---

## Subcommand: status

### Step 1: Get Feature Name

If feature name not in arguments, use AskUserQuestion to prompt for it.

### Step 2: Check Which Docs Exist

Check for existence of:
- `docs/{name}/BRIEF.md`
- `docs/{name}/RESEARCH.md`
- `docs/{name}/IMPLEMENTATION.md`
- `docs/{name}/PROGRESS.md`

### Step 3: Determine Status and Next Step

Based on which docs exist:

**No docs exist:**
"No documents found for feature '{name}'.
**Next step:** Run `/build product {name}` for the full design+build pipeline, or `/build research {name}` to start with research only."

**Only BRIEF.md exists:**
"Design brief confirmed for '{name}'.
**Next step:** Run `/build research {name}` (or `/build product {name}` to continue the orchestrated flow)."

**BRIEF.md and RESEARCH.md exist (or only RESEARCH.md):**
"Research complete for '{name}'.
**Next step:** Run `/build implementation {name}` to create implementation plan."

**RESEARCH.md and IMPLEMENTATION.md exist (BRIEF.md may also exist):**
"Research and implementation plan complete for '{name}'.
**Next step:** Run `/build progress {name}` to set up progress tracking."

**RESEARCH, IMPLEMENTATION, and PROGRESS all exist:**
Read PROGRESS.md to find current phase status.
"Feature '{name}' is in progress.
**Current status:** [Phase X - status]
**Next step:** Run `/build phase {next incomplete phase} {name}` to continue."

If all phases complete:
"Feature '{name}' implementation is complete!"

---

## Important Guidelines

### Use AskUserQuestion Liberally

Throughout all phases, use AskUserQuestion whenever:
- There's ambiguity in requirements
- Multiple approaches are possible
- You need to validate an assumption
- A decision will significantly impact the implementation
- You're unsure about scope or priority

This applies even in auto-mode — see the **Auto-mode override** section near the top. Pre-compute a recommendation and label that option `Recommended: <choice>` so the user can one-click through.

### Deep Research Expectations

"Deep research" means:
- Multiple web searches on different aspects
- Thorough codebase exploration
- Reading relevant documentation
- Considering multiple approaches
- Understanding trade-offs

Don't rush through research - it's the foundation for good implementation.

### Progress Tracking

Keep PROGRESS.md updated in real-time during phase work:
- Don't wait until the end to update
- Record decisions as they're made
- Note blockers immediately
- This creates valuable context for future sessions

### Functional Phases First

Every phase should produce something the user can see, test, or use. If a phase requires infrastructure (data models, APIs, services), bundle the minimum viable UI or integration that exercises it. The litmus test: "Can the user do something new after this phase?" If the answer is no, merge it with the next phase or restructure. Back-to-back phases that are all plumbing with no user-facing result are a planning smell — fix the plan, don't ship scaffolding.

### Scope Management

A key purpose of this workflow is preventing scope creep:
- Each phase should have clear boundaries
- If new requirements emerge, note them for future phases
- Don't expand the current phase's scope mid-implementation
- Use AskUserQuestion to validate if something is in/out of scope

### Always Works Philosophy

When implementing phases:
- Test changes as you make them
- Don't assume code works — verify it. "Verify" means observed-working through the appropriate channel (rendered screenshot, executed test, hit endpoint), not "I wrote the code that should do it."
- Tool-success messages and PostToolUse hook confirmations are not verification — they confirm the file was written, not that the change renders or runs correctly.
- If something doesn't work, fix it before moving on
- The goal is working software, not just written code

Step 7.5 of the `phase` subcommand is the enforcement point — every phase must produce a recorded **Verification** entry in PROGRESS.md before its status flips to Completed.

### Pairing with the Impeccable Plugin

The `/build` pipeline pairs with the **impeccable** plugin to combine technical rigor with design rigor:

- **`/impeccable shape`** produces the design brief (register, anchor references, color strategy, scene sentence, scope, key states, recommended references). This is what RESEARCH.md alone has historically been missing.
- **`/build` pipeline** produces the technical research, phased plan, and progress tracking that monolithic build steps in `/impeccable craft` lack.
- **`/build product`** orchestrates both: shape brief saved to `docs/{name}/BRIEF.md` → research → implementation → progress → phases. UI phases automatically run impeccable's mock + browser-iteration loop without prompting.

When working a UI-bearing feature, prefer `/build product` over running build and impeccable separately. When working a pure backend feature, the original `/build research → implementation → progress → phase` flow is still the right entry point — the impeccable loop self-skips for `[non-ui]` phases.

Do NOT replace the design brief with PRODUCT.md/DESIGN.md, a self-authored summary, or research notes. The brief must be user-confirmed via `/impeccable shape`. PRODUCT.md and DESIGN.md are project anchors that *reduce* repeated questions during shape, not substitutes for the per-feature brief.
