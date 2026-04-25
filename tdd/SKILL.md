---
name: tdd
description: Use ONLY when the user explicitly invokes `/tdd <feature>`, says "let's TDD this", "test-drive this", or otherwise asks for the TDD cycle. Enforces strict Red → Green → Refactor: plan tests up front, write one failing test, make it compile, make it pass with minimal code, simplify, refactor. Default mode is `auto` (no pauses). Do NOT use for retrofitting tests onto existing code, one-off "write a test for X" requests, quick fixes, or exploratory spikes — those are not TDD and the discipline gets in the way.
---

# Test-Driven Development

TDD is a design technique that uses tests as a tool. Design emerges from usage, not speculation. Short feedback loops let you course-correct immediately. The resulting architecture is testable by design, not retrofitted. **Do not rush toward feature completion** — correctness and design matter more than speed. Add only what tests demand.

## Mode

When invoked, announce: `Using TDD skill in mode: [auto|human]`

- **auto** (default): proceed through the cycle without pausing for confirmation. Still report each phase clearly.
- **human**: pause for confirmation after test planning, after each Red, after each Green, and after Refactor.

The user can override mode with `/tdd <feature> human` or by saying "switch to human mode" mid-cycle.

## Phase markers

Prefix each major phase line in your output with the marker, then a space:

- 🔴 Red — failing test
- 🌱 Green — minimal code to pass
- 🌀 Refactor — clean up while green
- 🧹 Refactoring sub-step (during refactor planning/execution)

## Setup (once per session/feature)

Before the first cycle, do these in parallel where possible:

1. **Restate the feature in one sentence** and identify the smallest first vertical slice. If the user gave a big feature, propose the first slice; in `human` mode wait for confirmation, in `auto` mode proceed.
2. **Find the project's test command**:
   - Check `CLAUDE.md` / `AGENTS.md` for documented test commands.
   - Check memory for project-specific testing rules (forbidden modules, required tag exclusions, etc.). **Honor these strictly.**
   - Otherwise infer from build files (`package.json`, `Makefile`, `pyproject.toml`, `build.gradle`, `Cargo.toml`, etc.).
   - If ambiguous, ask once. Don't guess.
3. **Find naming/style convention** by reading one or two existing tests in the same module. Match `itShould...` / `test_should_...` / `it("...")` / etc.

## Test Planning

Before any test code, list the tests as `[TEST]` comments. Plan the whole behavior, then implement one test at a time.

```
[TEST] Empty buffer reports zero size
[TEST] Push one item, size is one
[TEST] Push and pop returns the item
[TEST] Pop from empty buffer fails
...
```

**Walk ZOMBIES** explicitly to check completeness — see [references/zombies.md](references/zombies.md):
- **Z**ero / empty cases covered?
- **O**ne / single-item cases covered?
- **M**any cases covered?
- **B**oundary transitions covered? (both directions)
- **I**nterface clarity verified?
- **E**xceptions / errors covered?

In `human` mode, stop after planning and wait for confirmation before implementation. In `auto` mode, proceed.

## Cycle (per `[TEST]`)

Repeat for each `[TEST]` comment until all are passing.

### 🔴 Red — two-step

1. **Replace the next `[TEST]` comment directly with a real failing test.** No intermediate placeholders.
2. Test body uses given-when-then structure with **blank lines** between sections (do not add `// given` / `// when` / `// then` comments).
3. **Predict failure** explicitly: state what should fail and why.
4. Run tests — first failure should be a **compile/import error** (the production class/method doesn't exist yet). If something compiles when it shouldn't, the test isn't testing the right thing.
5. Add the **minimum** to make it compile (empty method body, return null/zero/default). No real behavior yet.
6. **Predict assertion failure** explicitly.
7. Run tests — should now compile but the assertion should fail. If it accidentally passes, the test is too weak; strengthen it.

### 🌱 Green — minimal code to pass

1. Write the **minimum** code to make the test pass. Bias hard toward small.
   - "Fake it" (return a hardcoded value matching the test's expectation) is a legitimate green step.
   - "Obvious implementation" is fine when the implementation is genuinely trivial.
   - Resist adding behavior the test doesn't require — no null checks, validation, error handling, logging, or extra branches unless a test demands them.
2. **Predict** that the new test passes and that all previously-passing tests still pass.
3. Run the relevant test suite. Default to running **all tests in the affected module/package**, scoped per project rules (forbidden modules, excluded tags, etc.). Don't run a single-test command if regressions could hide elsewhere.

### Simplify

For each line/expression you just added, ask: **"Does a failing test require this?"**

- If no test requires it → delete it, OR add a `[TEST]` comment to write the test that would justify it.
- Run tests after each simplification.
- Repeat until every line is justified by a test.

### 🌀 Refactor

Refactor is **mandatory consideration**, optional execution. Always look. If nothing's worth changing, say so and move to the next test.

1. Reflect on the domain:
   - Missing concept that would make the code more expressive?
   - Object/function waiting to be extracted?
   - A better way to model the problem?
2. You may **introduce abstractions** (new classes, helper functions, named values) as long as you add **no new behavior**. Tests must stay green; no new untested code.
3. Look for: duplication, unclear names, methods doing too much, magic numbers/strings, "fake it" shortcuts from green that should now be real, project-specific anti-patterns (Tell-Don't-Ask violations, etc., per CLAUDE.md).
4. Say `🧹 Starting refactoring stage` and list planned refactorings.
5. Implement **one at a time**, run tests after each.
6. When done (or if none needed), say `🧹 Refactoring complete`.

Then go back to the next `[TEST]` comment.

## Final Evaluation (after all planned tests pass)

1. **Gap analysis**: review the production code and ask what tests are missing. Walk ZOMBIES one more time. If gaps exist, add new `[TEST]` comments and run the cycle for each.
2. **Hardcoded values**: anything still hardcoded that shouldn't be? Triangulate it out via a new test, then refactor.
3. **Expressiveness**: re-read the production code and tests. Anything to clarify, rename, or extract? If yes, go to refactor phase.

## Discipline rules (non-negotiable)

1. **All code changes follow TDD** — even mid-stream "just add this one thing" requests. Write the test first.
2. **One test at a time** — focus on the simplest, lowest-hanging fruit.
3. **Predict failures and successes** explicitly before each run.
4. **Two-step red** — compile fail, then assertion fail. Don't skip step one.
5. **Minimal green** — just enough to pass.
6. **No comments in production code** unless asked.
7. **Run the right scope of tests every cycle** — the broadest scope project rules allow, to catch regressions.
8. **Refactor at the first opportunity** when tests are green. Don't accumulate debt across cycles.
9. **Test behavior, not implementation** — assert on responses, return values, or observable state, not on which methods got called.
10. **Push back when something seems wrong, unclear, or misspecified.** The user wants TDD precisely because it surfaces design problems early — flag them.
11. **Honor project-specific test rules** from CLAUDE.md and memory (forbidden modules, excluded tags, naming conventions).
12. **Don't skip pauses in human mode.** That's the entire point of human mode.

## When the cycle gets stuck

- **Test is too big to make pass in a small step**: revert the test and write a smaller one. Beck calls this "shrink the step."
- **Don't know how to make it pass**: write a simpler test that you do know how to pass, get to green, then triangulate up.
- **Refactor breaks tests**: revert the refactor immediately. Refactors must keep all tests green.
- **Test fails for the wrong reason** (compile error in test itself, missing import, infrastructure failure): fix the test infrastructure, then re-run. Don't proceed until the failure is for the *intended* reason.

## Anti-patterns

- ❌ Writing the implementation first and then "TDD-ing" by writing tests after — this is retrofitted testing, not TDD.
- ❌ Bundling multiple behaviors into one test (`assert A && assert B && assert C` for unrelated properties).
- ❌ Going Red → Green → "I'll refactor later" and starting the next cycle without refactoring. Refactor while context is fresh, or explicitly note technical debt as a `[TEST]` or TODO.
- ❌ Using TDD for code with no logic to verify (pure DTOs, simple wiring, framework boilerplate). Tell the user it's not a TDD candidate and propose a different approach.
- ❌ Running a single-test command when regressions could hide elsewhere — slow but correct beats fast and broken.
- ❌ Skipping the compile-fail step when adding a new class/method — the empty stub forces you to define the interface from outside.
