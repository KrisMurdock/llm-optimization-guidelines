# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## 5. General Rules for Implementing Optimization Guidelines

These rules apply to any task where an optimization document under `optimize_guideline/` is used to guide later implementation. They were abstracted from cases where another model followed a written optimization route but produced code that looked plausible while missing the real objective.

### 5.1 Convert the Guideline Into Explicit Acceptance Criteria

Before coding, restate the optimization document as concrete checks:
- Which user-visible behavior must change?
- Which existing production path must call the new logic?
- Which data contract must be preserved or extended?
- Which old behavior must be removed, kept as fallback, or put behind a feature flag?
- Which tests or manual checks prove completion?

Do not treat headings, module names, or suggested file names as the goal. The goal is the behavior described by the document.

### 5.2 Do Not Stop at Scaffolding

Creating files, classes, prompts, configs, schemas, or helper modules is not enough. New code must be reachable from the real execution path.

Before claiming completion, verify:
- The production entrypoint calls the new implementation.
- The returned result is actually consumed by the caller.
- The UI/API/job/task layer sees the new state or output.
- The old path is intentionally removed, feature-gated, or documented as fallback.

If new code can be deleted without changing runtime behavior, it is dead scaffolding.

### 5.3 Preserve One Source of Truth

Many implementation failures come from duplicating responsibilities.

Avoid:
- Computing the same score in two places.
- Generating the same report section twice.
- Keeping parallel schema definitions that drift.
- Letting frontend and backend each invent defaults.
- Letting an LLM regenerate values that were already produced deterministically.

For every key artifact, identify the owner:
- data source
- normalized data contract
- score or decision
- generated content
- final assembly
- persistence
- frontend display

All other layers should consume that owner, not recreate it.

### 5.4 Keep Names and Contracts Exact

String-keyed contracts are fragile. A minor rename can silently break the whole optimization.

Before changing or adding any dimension, phase, status, field, section key, route, or event name:
- Search all references with `rg`.
- Update producer, consumer, schema, prompt, tests, and UI together.
- Check for duplicates.
- Check aggregate invariants such as weights summing to `1.0`.
- Add a contract test if the value crosses module boundaries.

Examples of fragile keys include rubric dimensions, progress `phase_key`, API fields, section keys, task statuses, and JSON output fields.

### 5.5 Match Callback and Async Contracts Exactly

If the guideline asks for progress, streaming, subtasks, or background execution, verify the callback signature and async behavior before wiring it.

Rules:
- If a callback is async, `await` it.
- If a function may be sync or async, detect `inspect.isawaitable(...)`.
- Do not pass a new callback argument shape unless an adapter converts it to the existing contract.
- Preserve frontend-facing fields such as `phase_key`, `sub_tasks`, `status`, `progress`, and `detail`.

Progress that is emitted but never awaited, persisted, or rendered does not count.

### 5.6 Validators Need Evidence, Not Vibes

Any validation, review, rubric, consistency check, or quality gate must receive the evidence required to make the judgment.

Do not ask a validator to verify facts using only:
- summaries
- generated prose
- key claims without source data
- truncated JSON blobs with unknown missing fields

Pass compact but sufficient evidence:
- normalized primary data
- relevant dependency outputs
- source paths or evidence IDs
- score reasons
- missing-data notes
- allowed value ranges or schema

If data must be shortened, summarize it structurally instead of slicing raw JSON by character count.

### 5.7 Critical Failures Must Not Become Warnings

Retry loops often hide serious bugs by accepting bad output after the last retry.

Use this rule:
- pass: accept
- minor/major after retries: accept only if the product can safely show a warning
- critical after retries: fail or create a fallback that explicitly says the result is unavailable

Do not mark factual contradictions, parse failures, missing required data, broken persistence, or unreachable code as `passed_with_warnings`.

### 5.8 Fallbacks Must Be Honest

Fallbacks should preserve user trust. They must not masquerade as successful optimized output.

Bad fallback:
- Wrap invalid model output into a fake valid result.
- Fill missing data with generic confident prose.
- Show zero values as real metrics.
- Continue with stale cached results without labeling them.

Good fallback:
- Mark status as `fallback`, `failed`, or `insufficient_data`.
- Explain what failed.
- List missing inputs.
- Avoid strong conclusions.
- Keep the UI/report/API honest about degraded quality.

### 5.9 Deterministic Assembly Beats LLM Rewriting

When the system already has validated artifacts, do not ask an LLM to rewrite facts, numbers, scores, or final structured tables.

Prefer deterministic code for:
- ordering sections
- joining validated outputs
- copying score tables
- inserting fixed disclaimers
- rendering known metrics
- mapping backend statuses to frontend labels

Use an LLM only where judgment or natural language synthesis is necessary, and protect generated output with schema parsing and validation.

### 5.10 End-to-End Verification Is Required

Passing isolated unit tests is not enough for optimization work. Always prove the new behavior through the full path the user cares about.

For any optimization, check:
- trigger: the user action or backend job starts the new path
- execution: the new logic runs
- data: expected fields are present and correctly shaped
- persistence: output is saved if the product expects it
- display: frontend or final artifact shows the new result
- regression: old critical behavior still works or is intentionally replaced

If possible, add one test that fails when the new code is not wired into production.

### 5.11 Keep Optimization Scope Surgical

Optimization documents can be broad. Implement in phases, but each phase must be complete.

Avoid:
- partially implementing many suggestions with no finished user value
- unrelated refactors while following a guideline
- changing infrastructure to mask a feature issue
- modifying generated artifacts without checking the real source of truth

For each phase, define:
- included behavior
- excluded behavior
- files touched
- verification command or manual check

### 5.12 Preserve Existing User-Facing Semantics Unless Explicitly Changed

When optimizing internals, do not accidentally regress the interface contract.

Before changing UI steps, API responses, report sections, progress phases, or persisted fields:
- Compare old and new user-visible behavior.
- Keep backward compatibility or provide migration/fallback.
- Make removed behavior explicit in the implementation notes.

An optimization that improves internals but breaks the user's existing workflow is not complete.

### 5.13 Use Specific Examples Only as Examples

Project-specific cases, such as modular research-report generation, rubric scoring, deep-analysis progress, PDF save flow, or historical backtest UI, are examples of these rules. Do not overfit future work to those exact modules.

The transferable lesson is:
- wire new code into the real path
- keep one source of truth
- preserve contracts exactly
- validate with evidence
- fail honestly
- verify end to end
