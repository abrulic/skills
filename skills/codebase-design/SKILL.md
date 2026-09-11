---
name: codebase-design
description: >
  Shared vocabulary for designing deep modules: a lot of behaviour behind a small
  interface, placed at a clean seam, testable through that interface. Use when designing
  or improving a module's interface, deciding where a seam goes, making code more
  testable, or when another skill needs the deep-module vocabulary.
---

# Codebase design

Design **deep modules**: a lot of behaviour behind a small interface, placed at a clean
seam, testable through that interface. Use these terms exactly.

Vocabulary from Ousterhout's *A Philosophy of Software Design*.

## Glossary

**Module**: anything with an interface and an implementation. Scale-agnostic: a function,
class, package, or slice. Not "unit", "component", or "service" unless those are the
domain words.

**Interface**: everything a caller must know: types *and* invariants, ordering, error
modes, required config, performance characteristics. Not just the TypeScript `interface`
or the parameter list.

**Implementation**: what's inside. Distinct from **Adapter** (the role that fills a seam).

**Depth**: leverage at the interface. **Deep** = small interface + lots of implementation.
**Shallow** = interface nearly as complex as the implementation (avoid).

**Seam** (Michael Feathers): a place you can change behaviour *without editing there*.
Where the seam goes is its own design step.

**Adapter**: a concrete thing that satisfies an interface at a seam.

**Leverage**: what callers get from depth. One implementation pays back across N call
sites and M tests.

**Locality**: what maintainers get from depth. Change, bugs, and verification concentrate
in one place.

## Deep vs shallow

When designing an interface, ask:

- Can I reduce the number of methods?
- Can I simplify the parameters?
- Can I hide more complexity inside?

**The deletion test.** Imagine deleting the module. If complexity vanishes, it was a
pass-through. If complexity reappears across N callers, it was earning its keep.

**The interface is the test surface.** Callers and tests cross the same seam. If you want
to test *past* the interface, the module is probably the wrong shape.

**One adapter = a hypothetical seam; two adapters = a real one.** Don't introduce a seam
until something actually varies across it.

## Testable by construction

1. Accept dependencies, don't create them inside.
2. Return results, don't mutate/side-effect as the only output.
3. Keep the surface area small.

Taste rules for the code inside the module: `code-quality-standards`.
How to test at the seam: `test-discipline`. Which layer: `test-strategy`.
