---
name: software-design
description: 'Applies the principles, trade-off frameworks, and red flags from John Ousterhout''s "A Philosophy of Software Design". USE FOR: designing, writing, refactoring, or reviewing software.'
---

# A Philosophy of Software Design

## How to Use These Principles

Every design rule has limits and exceptions; **any principle can create complexity if taken too far**. Red flags are **diagnostic signals to investigate whether a better alternative exists**, never mechanical rewrite commands.

### Decision & Implementation Loop
1. **Judge by Net Complexity, Weighted by Frequency**: Evaluate decisions by whether they reduce overall cognitive load, change amplification, and unknown unknowns—especially for the **most common callers and use cases**, not by raw line count or method count.
2. **Design It Twice for Non-Trivial Decisions**: When designing or refactoring a significant module, interface, or control flow, sketch at least two genuinely different designs and compare their interface cost, information hiding, and caller ergonomics before coding. (Do not invent multi-design ceremony for trivial localized edits.)
3. **Define the Abstraction Before the Body**: State what the module/method guarantees at its interface before writing its internals. If the contract cannot be stated simply and completely, reconsider the boundary—never omit essential contract details just to make a comment look short.
4. **Check Both Sides of Every Rule ("Taking It Too Far")**: Before splitting, combining, generalizing, or masking errors, verify the counter-conditions in the sections below so you do not trade one form of complexity for a worse one.
5. **Respect Task Scope & Existing Contracts**: Make strategic improvements where you touch code, but stay proportional to task scope, preserve existing caller contracts unless an API change is intended, and review the final diff to ensure comments and tests match the code.

---

## 1. The Nature of Complexity & Strategic Programming

**Complexity** is anything related to the structure of a software system that makes it hard to understand and modify. Effective complexity is weighted by how often developers encounter a part of the system.

- **Symptoms**:
  1. **Change amplification**: A seemingly simple change requires code modifications in many different places.
  2. **Cognitive load**: How much a developer needs to know in order to complete a task. (Fewer lines of code that require more mental context are more complex than longer, linear code that requires less context.)
  3. **Unknown unknowns**: It is not obvious which pieces of code must be modified to complete a task, or what information a developer must have to carry out the task successfully.
- **Causes**:
  1. **Dependencies**: A piece of code cannot be understood and modified in isolation; related code must be considered and/or modified when it changes.
  2. **Obscurity**: Important information is not obvious.
- **Strategic vs. Tactical Programming**: Tactical programming focuses only on getting the current task working quickly, accumulating complexity. Strategic programming treats working code as insufficient: invest continually in clean abstractions, while factoring in practical constraints like deadlines, disproportionate refactoring cost, and external compatibility.

---

## 2. Modules Should Be Deep

A **module** is any unit of code with an interface and an implementation (class, function/method, subsystem, or service).
- **Interface**: Everything a caller must know to use the module—both **formal** (signature, types, exceptions/error codes) and **informal** (high-level behavior, preconditions, side effects, ordering constraints).
- **Abstraction**: A simplified view of an entity that omits unimportant details. Avoid **false abstractions** that appear simple on the surface but still require callers to understand internal details to use them safely.
- **Deep vs. Shallow Modules**:
  - **Deep modules** provide powerful functionality behind a simple interface (small interface cost, large hidden implementation benefit).
  - **Shallow modules** have complicated interfaces relative to the small functionality they provide.
- **Common-Case Simplicity**: Design interfaces so the most common usage is as simple as possible, without forcing callers to deal with rarely used options (**Overexposure**).
- **Guardrail — Don't Confuse Size with Depth**: A small helper function is **deep** if its signature and contract are much simpler than its implementation. A large function or class is **shallow** if it exposes a complex interface with many parameters, preconditions, or side effects. Evaluate the ratio of interface cost to hidden complexity, not line count alone.

---

## 3. Information Hiding and Leakage

- **Information hiding**: Each module should encapsulate design decisions (data structures, formats, low-level mechanisms, invariants) within its implementation so they do not appear in its interface. Also practice information hiding *within* a class so private data details are localized to as few methods as possible.
- **Information leakage**: Occurs when the same design decision is reflected in multiple modules, creating a dependency between them (including through an interface, or via shared unencapsulated assumptions).
- **Temporal decomposition**: Structuring code strictly by the execution order of steps (e.g., separate modules for reading, modifying, and writing that all share knowledge of the same format). **Focus on the knowledge needed to perform each task, not the order in which tasks occur.**
- **Taking It Too Far**: Information hiding only makes sense when the information is **not needed outside the module**. If callers genuinely need a piece of information or control, expose it cleanly in the interface rather than hiding it.

---

## 4. General-Purpose Modules Are Deeper

- **Somewhat General-Purpose**: Implement **functionality** only for current needs (do not build speculative features), but make the **interface** general enough to represent the underlying abstraction cleanly and separate mechanism from caller-specific policy.
- **Diagnostic Questions**:
  1. *What is the simplest interface that will cover all my current needs?*
     - **Taking It Too Far**: Reducing the number of methods is only an improvement if each method stays simple. Combining methods by adding many extra parameters or mode flags makes the interface *more* complex, not simpler.
  2. *In how many situations will this method be used?*
     - **Taking It Too Far**: A method having only one caller is **not** automatically bad. A single-caller helper that hides real complexity behind a clean, general interface is good; the red flag is a method whose interface bakes in caller-specific policy so tightly that two similar operations require separate specialized methods.
  3. *Is this API easy to use for my current needs?* If callers must write significant plumbing code to use a general interface for today's task, the abstraction is wrong.

---

## 5. Different Layer, Different Abstraction

Each layer or call boundary should provide a **different abstraction** from the layers above and below it. Similarly, the internal representation of a class should differ from its external interface; if they are identical, the class is likely shallow.

- **Pass-Through Methods**: A method that does little except invoke another method with a similar or identical signature adds interface cost without adding functionality.
- **When Interface Duplication Is OK**: Methods with the same signature are justified when each provides significant independent functionality—such as a dispatcher selecting among multiple implementations of an interface.
- **Decorators**: Wrappers that duplicate a whole interface just to tweak one behavior are often shallow; consider adding the capability directly to the underlying class, merging with the use case, or combining wrappers first.
- **Pass-Through Variables**: Variables threaded through a long chain of intermediate methods only so a leaf method can read them.
  - **Taking It Too Far (Context Object Trade-offs)**: Storing pass-through state in a shared context object avoids polluting intermediate signatures, **but context objects have many of the disadvantages of global variables**: they obscure where data comes from, easily turn into grab-bags creating couplings across the system, and introduce **thread-safety hazards**. Prefer immutable context fields where possible, scope contexts per system/instance rather than globally, and weigh whether explicit parameters are actually clearer for a short call chain.

---

## 6. Pull Complexity Downwards

- Most modules have more users than developers; **it is more important for a module to have a simple interface than a simple implementation**.
- **Configuration Parameters**: Before exporting a parameter or policy knob, ask: *"Will callers be able to determine a better value than we can determine here?"* If not, compute a sensible default or dynamic policy internally.
- **Taking It Too Far**: Do not pull complexity downward indiscriminately (which turns a lower layer into a dumping ground for unrelated application policy). Pulling complexity downward makes the most sense when:
  1. The complexity being pulled down is closely related to the module's existing functionality,
  2. Doing so will result in many simplifications elsewhere in the application, AND
  3. Doing so results in a **simpler module interface**.

---

## 7. Better Together Or Better Apart?

Deciding whether to combine or separate code is a two-way trade-off. Compare both directions against net complexity:

### Bring Code Together When:
1. **Information is shared**: Both pieces depend on the same internal format, invariant, or representation.
2. **It will simplify the interface**: Combining modules eliminates intermediate interfaces, data structures, or state-shuttling when two components are almost always used together.
3. **It eliminates non-trivial duplication**: Factoring repeated code into a single method is most effective when the repeated snippet is non-trivial and the extracted method has a simple signature (if the snippet is only a line or two, or requires a complex signature, weigh whether extraction actually reduces net complexity).

### Separate Code When:
1. **General-purpose mechanism vs. special-purpose policy**: Keep lower-level, reusable mechanisms separate from higher-level, caller-specific policy.

### Splitting and Joining Methods
- **Length alone is not a reason to split**: A longer method with linear flow and a single cohesive abstraction is simpler than chopping it into conjoined pieces.
- **Two valid ways to split a method**:
  1. **Extract an independent subtask**: Factor out a child method whose interface is much simpler than its body. *Test*: You must be able to understand the child method without reading the parent, and understand the parent without reading the child's implementation.
  2. **Split an overly broad method into two independent sibling methods**: When one method tries to do two weakly related things, split it for callers—ideally most callers only need to invoke one of them (if callers must invoke both in sequence and pass state between them, weigh that interface cost carefully as splitting is usually not a good idea).
- **Bad split (Conjoined Methods)**: If a reader must flip back and forth between two methods to understand how either works, or if the split requires passing many local variables between them, keep them together.

---

## 8. Define Errors Out Of Existence

Exception and error handling often accounts for a large fraction of system complexity. Four exception-handling techniques—plus a related technique for eliminating special cases—reduce error-handling proliferation (choose according to semantics, recovery needs, and handler placement):

1. **Define errors out of existence**: Best when feasible—design semantics so normal behavior handles the situation naturally without raising an error (when creating/modifying an API contract).
2. **Mask exceptions/errors**: Detect and resolve the condition inside a low-level module (e.g., retrying a transient transport glitch) so callers never see it.
3. **Exception aggregation**: Propagate errors cleanly to a single high-level handler rather than writing bespoke recovery logic at every intermediate call site.
4. **Just crash / abort on unrecoverable invariants**: For internal invariant corruption or conditions from which the application cannot meaningfully recover, aborting immediately is simpler than threading broken-state recovery everywhere. (Whether a condition is recoverable depends on the system's reliability requirements.)
5. **Design special cases out of existence**: Structure data representations and normal-case logic so boundary conditions (empty ranges, no-op states) execute through the normal code path without special `if` branches.

- **Taking It Too Far**:
  - **Never mask errors that callers need to know about**: Just like information hiding, masking an error inside a module is valid **only** if the module can still fulfill its contract and callers do not need the failure information. Swallowing a failure that leaves the caller's operation incomplete causes silent data loss or corruption.
  - **Do not silently break existing error contracts**: Defining errors out of existence is a contract design technique; when modifying an existing API whose callers rely on specific error returns, preserve the contract unless updating the API and its callers intentionally.

---

## 9. Comments, Naming, and Modifying Existing Code

### Comments Describe Things That Aren't Obvious from the Code
- **Follow consistent conventions**: Document classes, data structures, method interfaces, and member variables using the codebase's conventions (omitting a declaration comment only in the rare case where the declaration is already so completely obvious that there is nothing non-obvious to add).
- **Don't repeat the code**: Never write comments that restate what the code next to them visibly does using the same words.
- **Lower-level comments add precision**: For variables, fields, and parameters, clarify units, boundary inclusivity, nullability, resource ownership, and invariants.
- **Higher-level comments enhance intuition**: For code blocks, explain *what* the block accomplishes conceptually and *why*, not *how* each statement executes.
- **Separate Interface vs. Implementation Comments**:
  - **Interface comments** tell callers everything needed to use the module (behavior, arguments, return values, preconditions, side effects, error conditions) and **nothing** about internal implementation details.
  - **Implementation comments** explain internal strategy, subtle invariants, and bug-fix rationale.
- **Cross-module design decisions**: Record protocols or invariants spanning multiple files in one central, discoverable place (such as a shared header or design-notes section) and leave brief pointer comments at each participating code site rather than duplicating the full explanation.
- **Comments as a Design Indicator**: If an interface comment must be long and complicated to be complete, treat that as a red flag to investigate whether the abstraction can be simplified—**never** "fix" it by deleting necessary contract documentation.

### Choosing Names
- **Create an image & be precise**: Names should convey a clear mental model of what the entity is and isn't.
- **Context & Scope Matter (Not a Blind Word Blacklist)**:
  - Generic names (`data`, `info`, `handle`, `process`, `result`) are red flags when they hide specific domain meaning. However, `result` is completely appropriate for the return-value variable of a method, and `i`/`j` are appropriate for short, obvious loop indices.
  - **Avoid names that are too specific**: A name that implies a narrower use case than the parameter actually supports (e.g., naming a general text-range parameter `selection`) misleads readers.
  - The wider the scope/distance between declaration and use, the more descriptive the name must be.

### Modifying Existing Code
- Put design and bug rationale in code comments, not just commit messages.
- Keep comments near the code they describe, avoid duplicating comments across multiple places, and **check the diffs before finishing** to verify all affected comments remain accurate.

---

## 10. Consistency, Obviousness, Software Trends, and Performance

- **Consistency**: Follow existing conventions in the file and repository for naming, style, interfaces, and invariants.
  - **Taking It Too Far**: Do not force dissimilar things into the same pattern just for superficial uniformity, and allow deliberate, coordinated convention updates when explicitly authorized and applied consistently.
- **Code Should Be Obvious**: Code is obvious when a reader's first guesses about its meaning and behavior are correct. Avoid non-obvious control flow, generic unnamed tuple/pair containers that obscure field meaning, and constructs that violate reader expectations.
- **Software Trends**:
  - **Inheritance**: Implementation inheritance creates tight parent–child dependencies and information leakage. Prefer composition or interface inheritance first; when implementation inheritance is used, cleanly separate parent state from subclass state.
  - **Increments Should Be Abstractions**: Develop software by building clean abstractions, not by tacking on tactical feature patches.
  - **Unit Tests & TDD**: Unit tests (especially coverage-focused tests of internal behavior) are essential to refactor confidently without introducing regressions. Avoid rigid test-first TDD when designing new modules (which focuses attention on getting individual features working rather than finding the best abstraction), **except when fixing bugs**: before fixing a bug, write a unit test that fails due to the bug and passes after the fix.
  - **Design Patterns**: Apply well-known patterns (such as Iterator or Observer) when they cleanly fit a problem, but never force a pattern when a simpler direct approach works.
  - **Getters and Setters**: Avoid routine getters and setters that merely expose private fields as shallow boilerplate; expose state only when needed part of the module's abstraction.
- **Designing for Performance**:
  - Simple code tends to be fast. Choose naturally clean, efficient data structures and algorithms initially, avoiding known-expensive operations when a clean alternative exists.
  - **Measure before tuning**: Before adding complexity for optimization, measure a baseline to identify the actual bottleneck, prioritize fundamental design improvements over local hacks, and **measure after the change**—if the added complexity did not yield a significant speedup, back it out.
  - **Design around the critical path**: When optimizing a proven hot path, identify the minimal code required for the common case and structure the module so the common case runs along one straight-line path, funneling special cases to a slower side path.

---

## 11. Summary of Design Principles

1. Complexity is incremental: you have to sweat the small stuff.
2. Working code isn't enough.
3. Make continual small investments to improve system design.
4. Modules should be deep.
5. Interfaces should be designed to make the most common usage as simple as possible.
6. It's more important for a module to have a simple interface than a simple implementation.
7. General-purpose modules are deeper.
8. Separate general-purpose and special-purpose code.
9. Different layers should have different abstractions.
10. Pull complexity downward.
11. Define errors (and special cases) out of existence.
12. Design it twice.
13. Comments should describe things that are not obvious from the code.
14. Software should be designed for ease of reading, not ease of writing.
15. The increments of software development should be abstractions, not features.

---

## 12. Summary of Red Flags

- **Shallow Module**: The interface for a class or method isn't much simpler than its implementation.
- **Information Leakage**: A design decision is reflected in multiple modules.
- **Temporal Decomposition**: The code structure is based on the order in which operations are executed, not on information hiding.
- **Overexposure**: An API forces callers to be aware of rarely used features in order to use commonly used features.
- **Pass-Through Method**: A method does almost nothing except pass its arguments to another method with a similar signature.
- **Repetition**: A nontrivial piece of code is repeated over and over.
- **Special-General Mixture**: Special-purpose code is not cleanly separated from general purpose code.
- **Conjoined Methods**: Two methods have so many dependencies that its hard to understand the implementation of one without understanding the implementation of the other.
- **Comment Repeats Code**: All of the information in a comment is immediately obvious from the code next to the comment.
- **Implementation Documentation Contaminates Interface**: An interface comment describes implementation details not needed by users of the thing being documented.
- **Vague Name**: The name of a variable or method is so imprecise that it doesn't convey much useful information.
- **Hard to Pick Name**: It is difficult to come up with a precise and intuitive name for an entity.
- **Hard to Describe**: In order to be complete, the documentation for a variable or method must be long.
- **Nonobvious Code**: The behavior or meaning of a piece of code cannot be understood easily.

