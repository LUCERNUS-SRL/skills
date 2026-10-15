---
name: clean-code
description: Clean Code smells and the pass that finds them. Use when writing, refactoring or reviewing code, or when another skill needs a pass.
---

These are the smells of Robert C. Martin's _Clean Code_ and Martin Fowler's _Refactoring_, corrected where applying them literally makes code worse. Write code with them in mind; find them with a **pass**.

The project's conventions, linter config and `AGENTS.md` come first, then [Guardrails](#guardrails), then [Smells](#smells).

## A pass

A **pass** reads the whole scope and checks every function and class in it against every smell that applies. It reports **findings**, in the shape of [FINDING-FORMAT.md](FINDING-FORMAT.md), and changes no code.

The **scope** is what you are given: a diff, the code the user names, or what another skill sets. For a diff, the scope is every function the diff touches, in full (the Boy Scout Rule: leave touched code cleaner), plus the class-level smells of any class that gained a member. Read each changed function at its call sites too: a name, a parameter list or a return shape is judged where it is used.

Read for bugs as well: error paths, retries and partial failure, concurrency, security.

Report a finding only when you would fix it if the code were yours. A number in a smell is a **threshold**: it prompts a look, and a finding needs the smell itself. A fix that changes what callers observe (return values, errors, ordering, side effects) is a **contract change**.

Done when every function and class in scope has been checked against every smell that applies, and anything left unchecked is listed with why.

## Smells

### Names

- **Mysterious Name**: a reader must open the body to learn what a name holds or does, or the name lists every step (`validateAndSaveAndNotify`). → Rename it to its intent, in the domain's words. If no honest name comes, the design is murky: split it.
- **Inconsistency**: one concept named or handled several ways (`fetch`, `get` and `load`; two error styles side by side). → Align with the convention already dominant in the code.
- **Hidden Side Effect**: a name hides a change the function makes (`saveDraft` that also sends an email). → Move the effect to the caller; when it cannot be separated, name it.

### Functions

- **Mixed Levels of Abstraction**: a business decision sits beside string parsing. → Extract the detail behind a name at the caller's level, and order functions so each caller sits above its callees.
- **Long Function**: over about 20 lines, running separable phases. → Split at the phases. A long function that reads clearly at one level stays whole.
- **Deep Nesting**: deeper than 2 levels, burying the main path. → Guard clauses, an extracted block, or a named condition.
- **Command-Query Separation**: a function both changes state and answers a question about it. → Split it into a command and a query. Returning an error, a status or a created id is no query, and atomic pairs stay combined (`pop`, compare-and-set, get-or-create).
- **Feature Envy**: a function uses another object's data more than its own. → Move it onto the data it envies.

### Parameters

- **Long Parameter List**: more than 3 parameters, often hiding a concept: parameters that travel together (Data Clumps) are a missing type. → Introduce the type.
- **Flag Argument**: callers pass a literal boolean to pick the behaviour. → One function per behaviour.
- **Output Argument**: the function mutates an argument to hand back a result. → Return the value, unless the language idiom or performance calls for a caller-owned buffer.
- **Primitive Obsession**: a string or number stands in for a domain concept with its own rules (an email, a money amount, an order id). → Give the concept a small type.

### Comments

- **Missing Why**: a reason the code cannot show (a constraint, an invariant, a unit, a workaround, a public API contract) is written nowhere. → Add the comment; a workaround links its issue.
- **Stale Comment**: restates what the code says, or no longer matches it. → Delete the restatement (renaming first if the code needed it), update the mismatch. A comment that says why stays.
- **Commented-Out Code**: → Delete it; version control remembers.

### Errors

- **Swallowed Error**: an empty catch, or an error result nobody reads. → Handle it, or let it propagate with context: the operation and the input that failed.
- **Sentinel Value**: failure or absence signalled by a special value (`-1`, `false`, `null`) the type system does not force callers to check. → The language's error mechanism, or an optional or empty collection where absence is normal.
- **Defensive Clutter**: checks for what the type system already rules out, or repeats of a check in the same module. → Delete them. Data crossing a trust boundary (user input, network, files, serialized data) is untyped, so its checks stay; authorization, injection and bounds checks always stay.

### Classes and modules

- **Divergent Change**: one class changes for unrelated reasons (I/O, formatting and policy in one place). → Split it along the reasons.
- **Shotgun Surgery**: one logical change forces edits scattered across many modules. → Gather what changes together into one module.
- **Constructor Over-injection**: more than 4 injected dependencies, serving more than one responsibility. → Split that responsibility out.
- **Message Chains**: `a.b().c().d()` reaches through collaborators' internals. → Ask the nearest object for what you need, or move the code down the chain. Data structures, fluent builders and pipelines are exempt.
- **Refused Bequest**: a subclass ignores or overrides most of what it inherits. → Replace the inheritance with composition.
- **Repeated Switches**: the same switch on the same type in several places. → Polymorphism, or one shared map. Exhaustive matches over a closed type the compiler checks are exempt.
- **Dependency Rule**: business rules depend on a framework, a database or other I/O. → Reverse it: the detail implements an interface the business rules own.
- **Tangled Concurrency**: threads, locks or async coordination woven into business logic. → Move the coordination apart and keep shared mutable state minimal.

### Duplication and clutter

- **Duplicated Code**: the same knowledge (a rule, a constant, a formula) in two places of one module or context. → Unify it now. Across services or bounded contexts each keeps its own copy. Code that only looks alike waits for a third copy, and is unified only when the copies change for the same reason.
- **Wrong Abstraction**: shared code grows flags or branches to serve diverging callers. → Inline it back into the callers, then extract what is truly shared.
- **Magic Number**: a literal whose meaning its use does not show (`86400`). → A named constant. 0, 1 and 2 in arithmetic or indexing are exempt.
- **Dead Code**: code nothing reaches. Search the whole repo first, non-code files included (config, scripts, SQL, docs); reflection, dependency injection, serialization, framework entry points and config strings count as reaching it. → Delete what is private. A public or exported member is a contract change.

### Tests

Test code is held to every smell here, with two exceptions: tests keep literal expected values, and prefer readable repetition over shared setup that hides the inputs.

- **Insufficient Tests**: a behaviour change without a test, or a bug fix without a test that fails without the fix. → Add the test.
- **Erratic Test**: passes or fails by run order, other tests, timing or environment. → Make it independent and repeatable.
- **Eager Test**: one test checks several behaviours. → One test per behaviour, each named for what it specifies.
- **Fragile Test**: breaks when a refactor leaves behaviour unchanged, because it reaches past the public interface. → Test behaviour through the public interface.
- **Missing Edge Case**: the edges (empty, one, many, limits) or the neighbourhood of a found bug go untested. → Add those cases.
- **Top-Heavy Test**: a behaviour tested at a costlier level than it needs (an end-to-end test for a rule the code's API can prove), or a test too slow to run after every small change. → Move it to the cheapest level that proves it. Integration and end-to-end tests check wiring and stay few.
- **Over-Mocking**: mocks for collaborators inside a boundary. → Mock only at architectural boundaries: database, network, clock, third-party services.

## Guardrails

These outrank every smell: a fix that creates one is no fix.

- **Lazy Element**: a function or class that does not pull its weight: its name says nothing its body does not, or it cannot be followed without opening its caller. → Inline it.
- **Speculative Generality**: an abstraction (interface, base class, wrapper, factory) without two real callers or a real seam such as a test boundary, I/O or a plugin point. → Inline it until a real need shows.
- **Number Gaming**: a fix that only gets code under a threshold (code moved into a helper, unrelated parameters bundled into an object) and leaves it no easier to understand. → Revert it.
