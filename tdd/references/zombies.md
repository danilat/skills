# ZOMBIES — Test Case Discovery Heuristic

A checklist for ensuring your `[TEST]` plan is complete before you start implementing.

## Structure

**ZOM axis** (simple → complex):
- **Z** — Zero: initial state after creation (empty, not full, default values)
- **O** — One: first item, first transition
- **M** — Many: multiple items, more complex scenarios

**BIE considerations** (apply at each ZOM level):
- **B** — Boundary: transitions between states, both directions (empty↔not-empty, not-full↔full, off↔on)
- **I** — Interface: let tests reveal what methods, parameters, and return types are needed
- **E** — Exceptions: error conditions, and verify the object still works after errors

**S** — Simple scenarios, simple solutions (applies throughout)

## Key principles

**Test transitions, not just states.** Verify moving from empty to not-empty, *and back again*. State coverage isn't enough — the moves between states are where bugs live.

**Procrastinate deliberately.** Defer real implementation until tests demand it. Hard-coded return values are fine — tests will catch when you forget to generalize.

**Interface emerges from tests.** Don't design the API up front. Write tests, and the needed methods reveal themselves. Naming gets clearer when you've tried to use the thing.

**Exceptions come last.** Get happy paths working first, then test error conditions. Verify that failed operations don't corrupt the object.

## Example: Circular buffer (FIFO)

```
[TEST] New buffer is empty                               <- Z
[TEST] New buffer is not full                            <- Z
[TEST] Put one item, buffer is not empty                 <- O
[TEST] Put then get returns the item                     <- O + I
[TEST] Put then get, buffer is empty again               <- O + B (transition back)
[TEST] Put three items, get returns them in order        <- M
[TEST] Fill to capacity, buffer is full                  <- M + B
[TEST] Wrap around: fill, empty, refill works            <- M + B
[TEST] Put to full buffer fails                          <- E
[TEST] Get from empty buffer fails                       <- E
[TEST] After failed put, buffer still works              <- E (integrity check)
```

## How to use during planning

1. List your initial `[TEST]` ideas.
2. Walk Z → O → M and ask at each level:
   - Have I covered the empty / single / multi case?
   - Are there boundary transitions I missed (both directions)?
   - Does the interface feel right, or are tests forcing awkward usage?
   - What could go wrong, and does the object stay consistent after?
3. Add the missing `[TEST]` comments. It's normal to double or triple your initial list.

## Anti-patterns

- Listing only happy-path tests. Boundaries and exceptions are where most bugs live.
- Listing only state-checking tests, no transition tests.
- Designing the interface before writing any tests — the point is to let usage drive the shape.

---

*Source: [TDD Guided by ZOMBIES](https://blog.wingman-sw.com/tdd-guided-by-zombies) by James Grenning.*
