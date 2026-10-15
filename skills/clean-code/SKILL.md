---
name: clean-code
description: Clean Code smells. Use when writing, refactoring or reviewing code, or when another skill needs them.
---

These are the code smells and rules of Robert C. Martin's _Clean Code_ and Martin Fowler's _Refactoring_, corrected where applying them literally makes code worse.

Precedence: the project's conventions, linter config and `AGENTS.md` first, then [Guardrails](#guardrails), then [Rules](#rules).

A **smell** is a named rule in Rules or Guardrails: a judgement call. A **finding** is a smell spotted at a specific place in the code. A smell whose fix changes what existing callers observe (return values, errors, ordering, side effects) is a contract change: its fix names the callers to update.

The scope is what you are given: your own code once it works, a diff, the code the user names, or a scope another skill sets. Check every function and class in scope against the smells that apply to it, and list what you left unchecked.

## Rules

A **threshold** (~20 lines, more than 3 parameters, nesting deeper than 2, more than 4 injected dependencies) prompts a look; a finding needs a real smell.

### Names

- **Mysterious Name**: the reader has to open the body to learn what a name holds or does, or the name lists everything the function does (`validateAndSaveAndNotify`). Rename it to reveal intent, in the domain's vocabulary. If no single honest name comes, the design is murky: split the function.
- **Inconsistency**: similar things are named or done differently (`fetch`, `get` and `load` for one concept; two error-handling styles side by side). Inside the scope, align with the convention already dominant in the code; report other places.
- **Hidden Side Effect**: a name hides a change the function makes (`saveDraft` that also sends an email). Move the effect out to the caller; when it cannot be separated, name it.

### Functions

- **Mixed Levels of Abstraction**: high-level steps sit beside low-level detail (a business decision next to string parsing). Extract the detail behind a name at the caller's level, and order functions so each caller sits above its callees.
- **Long Function**: over ~20 lines, mixing levels of abstraction or running separable phases. Split along those lines. A long function that reads clearly at one level stays whole.
- **Deep Nesting**: deeper than 2 levels, burying the main path. Flatten with guard clauses, extract the inner block, or name the condition.
- **Command-Query Separation**: a function both changes state and answers a question about it. Split it into a command and a query. Returning an error, a status or a created id is not a query, and atomic pairs stay combined (`pop`, compare-and-set, get-or-create).

### Parameters

- **Long Parameter List**: more than 3, hiding a concept: parameters that travel together (Data Clumps) are a missing type; introduce it.
- **Flag Argument**: callers pass a literal boolean to pick the behaviour. Split into one function per behaviour.
- **Output Arguments**: the function mutates an argument to hand back a result. Return the value instead, unless the language idiom or performance calls for a caller-owned buffer.

### Comments

- **Missing Why**: a reason the code cannot show (a constraint, invariant, unit, workaround or public API contract) is nowhere written down. Add the comment; a workaround links its issue.
- **Stale Comment**: restates what the code says, or no longer matches the code under it. Delete the restatement (renaming first if the code needed it) and update the mismatch. A comment that says why stays.
- **Commented-Out Code**: delete it; version control remembers.

### Errors

- **Swallowed Error**: an empty catch, or an error result nobody reads. Handle it, or let it propagate with context: the operation and the input that failed.
- **Sentinel Value**: failure or absence signalled by a special value (`-1`, `false`, `null`) that the type system does not force callers to check. Use the language's error mechanism (exception, Result type, error return), or an optional or empty collection where absence is normal.
- **Defensive Clutter**: checks for conditions the type system already rules out, or exact repeats of a check in the same module. Data crossing a trust boundary (user input, network, files, serialized data) is not typed yet, so it is checked; authorization, injection and bounds checks always stay.

### Classes and modules

- **Divergent Change**: a class changes for unrelated reasons: it mixes distinct concerns (I/O, formatting, policy), or its history shows unrelated edits. Split it along the reasons.
- **Constructor Over-injection**: more than 4 injected dependencies, serving more than one responsibility. Split that responsibility out.
- **Message Chains**: `a.b().c().d()` reaches through collaborators' internals. Ask the nearest object for what you need, or move the code that uses the result down the chain; pass-through methods only create a Middle Man. Data structures, fluent builders and pipelines are exempt.
- **Repeated Switches**: the same switch on the same type recurs in several places. Replace it with polymorphism or one shared map. Exhaustive matches over a closed type that the compiler checks are exempt.
- **Dependency Rule**: business rules depend on a framework, database or other I/O detail. Reverse it: the detail implements an interface the business rules own. If a software architecture skill is available, use it for layering and SOLID beyond single responsibility.
- **Tangled Concurrency**: threads, locks or async coordination woven into business logic. Move the coordination apart and keep shared mutable state minimal. If a concurrency skill is available, use it for the design.

### Duplication and clutter

- **Duplicated Code**: the same knowledge (a rule, a constant, a formula) in two places of one module or context. Unify it now. Across services or bounded contexts a shared copy couples them, so each keeps its own. Code that only looks alike waits for its third copy, and is unified only when the copies change for the same reason.
- **Wrong Abstraction**: shared code grows flags or branches to serve diverging callers. Inline it back into the callers, then extract again what is truly shared.
- **Magic Number**: a literal whose use does not show its meaning (`86400`). Name it as a constant. 0, 1 and 2 in arithmetic or indexing are exempt.
- **Dead Code**: code nothing reaches. Search the whole repo first, non-code files included (config, scripts, SQL, docs); reflection, dependency injection, serialization, framework entry points and config strings count as reaching it. Delete what is private; a public or exported member goes to the user as a question.

### Tests

Test code is held to every smell in this skill, except that tests keep literal expected values and prefer readable repetition over shared setup that hides inputs. Tests are F.I.R.S.T. (fast, independent, repeatable, self-validating, timely). The test smells:

- **Insufficient Tests**: a behaviour change without a test, or a bug fix without a test that fails without the fix. Add the test.
- **Erratic Test**: passes or fails depending on run order, other tests, timing or environment. Make it independent and repeatable.
- **Eager Test**: one test checks several behaviours. Split it into one test per concept, each named for the behaviour it specifies.
- **Fragile Test**: breaks when a refactor leaves behaviour unchanged, because it reaches past the public interface or mirrors production classes. Test behaviour through the public interface.
- **Missing Edge Case**: the edges (empty, one, many, limits) or the neighbourhood of a found bug go untested. Add those cases.
- **Top-Heavy Test**: a behaviour tested at a costlier level than needed (a UI or end-to-end test for a business rule the code's API could prove), or a test too slow to run after every small move. Move it to the cheapest level that proves it, with the slow part behind a boundary mock. Integration and end-to-end tests check wiring (configuration, dependency injection, contracts) and stay few.
- **Over-Mocking**: mocks for collaborators inside a boundary. Mock only at architectural boundaries: database, network, clock, third-party services.

## Guardrails

These outrank every Rule: a fix that would create one is no fix.

- **Lazy Element**: a function or class that does not pull its weight: its name says nothing its body does not, or a reader cannot follow it without opening its caller. Inline it.
- **Speculative Generality**: an abstraction (interface, base class, wrapper, factory) without two real callers or a real seam such as a test boundary, I/O or a plugin point. Inline it until a real need shows.
- **Number Gaming**: Goodhart's law applied to thresholds: a fix that only gets code under a threshold (code moved into a helper, unrelated parameters bundled into an object) and leaves it no easier to understand. Revert it.

## Reporting

A review ends in a report; the code stays as it was. Report a finding only when you would fix it if the code were yours. A bug goes first, marked `Bug`. Repeats of one smell form one finding that lists every location. Each finding gives:

- **Location**: the path and the nearest function or class, plus the line when you have it.
- **Smell**: `possible <Smell>`, or `possible <Principle> violation` for Command-Query Separation and the Dependency Rule. Every finding carries the qualifier `possible`.
- **Why**: why the smell applies here.
- **Code**: the 1 to 3 lines that show it, quoted verbatim. Smells about missing code, such as Insufficient Tests or Missing Why, have nothing to quote.
- **Fix**: one line saying what to change, and for a contract change, the callers to update.

A default layout, to adapt as needed:

````
src/render.ts:42, render(): possible Flag Argument. `compact` switches between two layouts.
```ts
function render(items: Item[], compact: boolean) {
```
Split into `renderCompact(items)` and `renderFull(items)`.
````
