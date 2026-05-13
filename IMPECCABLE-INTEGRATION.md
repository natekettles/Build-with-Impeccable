# Impeccable Integration

This is a fork of Shpigford's `/build` skill with three additions. The headline change: UX and UI work is delegated to the `/impeccable` skill, so every UI-bearing feature gets impeccable's design judgment baked into the same pipeline that scopes, plans, and ships it.

## Why combine them

`/build` is a feature pipeline: research, phased plan, progress tracking, end-to-end implementation. `/impeccable` is a design discipline: shape briefs, north-star mocks, browser-iteration craft, polish, audit. On their own, `/build` ships features that look like AI made them, and `/impeccable` produces beautiful surfaces with no plan or progress trail. Combined, the same workflow that scopes a feature also produces a confirmed design brief, builds against a fidelity inventory, and runs a critique-and-fix loop on every UI phase. One pipeline, both rigours.

## What changed

### 1. Invocation of /impeccable for all UX & UI

The headline change. A new `product` subcommand orchestrates the full design + build flow, and every UI-tagged phase auto-invokes impeccable's mock + browser-iteration loop without prompting.

The new `product` orchestrator (excerpt):

```diff
+## Subcommand: product
+
+End-to-end orchestrator that pairs the **impeccable** plugin's design rigor with this skill's phased build pipeline. This is the recommended entry point for any UI-bearing feature.
+
+### Step 2: Shape the Design Brief
+
+Invoke the **impeccable** skill's shape command on the user's behalf - call it as `/impeccable shape {name}` ...
+
+**Wait for explicit user confirmation of the brief.** Do not proceed until confirmed.
+
+### Step 3: Persist the Brief
+
+Once the brief is confirmed, write it verbatim to `docs/{name}/BRIEF.md` ...
+
+### Step 5: Run Implementation Planning
+
+Each phase is tagged `[ui]` or `[non-ui]` based on whether it touches user-facing surfaces.
+
+### Step 7: Phase Walkthrough
+
+UI phases auto-invoke the impeccable craft mock + browser-iteration loop (see Phase subcommand, Step 6.5 below). No prompt - it just happens. Non-UI phases skip the impeccable loop and run as the existing build pipeline.
```

Inside the per-phase runner, a new Step 6.5 fires the impeccable loop on UI phases:

```diff
+### Step 6.5: Impeccable Loop (UI phases only)
+
+**Skip this entire step for `[non-ui]` phases.** For `[ui]` phases, run automatically without prompting:
+
+1. **Load impeccable context.** Read PRODUCT.md and DESIGN.md ...
+2. **Confirm brief gate.** If `docs/{name}/BRIEF.md` does NOT exist for this feature, do not proceed ...
+3. **Load named impeccable references.** Open and read each reference file listed in BRIEF.md Section 9 ...
+4. **North-star mock (capability-gated).** ... generate 1-3 high-fidelity north-star comps ...
+5. **Production bar - apply during Step 7 build.** Real or realistic content. Preserve the mock's major ingredients ...
```

And a follow-up Step 7.6 runs the critique-and-fix browser loop:

```diff
+### Step 7.6: Browser-Iteration Loop (UI phases only)
+
+1. **Required viewport pass.** Re-capture the rendered state at mobile narrow, tablet/small laptop, and desktop wide.
+2. **Critique-and-fix loop.** Write a short critique for yourself, patch the implementation, re-inspect. Continue until no material issues remain ... AI slop test, absolute bans, intentional states, compositional adaptation across viewports.
```

When in the build pipeline this fires: the `product` orchestrator runs `/impeccable shape` up front (Step 2) and persists the brief; later, every `[ui]`-tagged phase runs Step 6.5 (load references, generate mock, apply production bar) before code is written, then Step 7.6 (viewport pass + critique loop) after the runtime verification gate. Non-UI phases skip both steps entirely.

### 2. Auto-mode override for AskUserQuestion

```diff
+### Auto-mode override
+
+This skill overrides auto-mode's "prefer assumptions over questions" guidance. The whole point of `/build` is to validate assumptions before writing code - silent assumptions are exactly what this workflow exists to prevent.
+
+Rules when auto-mode is active:
+- You MUST still call AskUserQuestion at every decision point this skill prescribes ...
+- To keep it cheap: always pre-compute your recommendation and label that option `Recommended: <choice>` ...
+- Routine mechanical work ... still proceeds without asking.
```

Auto-mode normally prefers silent assumptions over user questions. That defeats the point of `/build`, which exists specifically to surface scope and approach decisions before code is written. The override forces AskUserQuestion at prescribed decision points and keeps it cheap by requiring a pre-computed `Recommended:` option for one-click pass-through.

### 3. Explicit verification step

```diff
+### Step 7.5: Runtime / Visual Verification (mandatory before marking complete - all phases)
+
+"Done" means observed-working, not "I wrote the code that should do it." Tool-success messages and PostToolUse hook confirmations tell you a file was written, not that it renders or runs correctly. Before moving to Step 7.6 ... or Step 8 ..., you must independently verify the change at runtime ...
+
+1. **User-facing UI changes** - render the affected surface and capture a screenshot ...
+2. **Tests** - run the relevant test target ... Paste the actual pass/fail count into PROGRESS.md, not a paraphrase.
+3. **Both light and dark mode** when the phase touches visual design.
+4. **Console / log inspection** when a runtime is involved.
+
+If verification is genuinely impossible ... state that explicitly in PROGRESS.md: *"Built but unverified - reason: ..."*. Never imply success you haven't observed.
```

A hard gate before any phase flips to Completed. The default `/build` flow trusted the model's "I wrote it, so it works" reflex; Step 7.5 forces a runtime channel (browser screenshot, executed test, hit endpoint) and a written `Verification` entry in PROGRESS.md. The Always Works Philosophy section is updated to point at Step 7.5 as its enforcement point.

## See the full diff

Run `diff -u` against Shpigford's upstream `/build` SKILL.md to see every change in context.
