---
paths:
  - "**/*.py"
  - "**/*.ipynb"
---

# Design conventions

How to decide where code goes and what shape it takes. Use the
vocabulary below precisely — in comments, commit messages and review —
so that "this function is too shallow" is a claim someone can check,
not a matter of taste.

## Vocabulary

- **Module** — anything with an interface and an implementation. A
  function, a class and a package are all modules.
- **Interface** — everything a caller must know to use the module
  correctly: the signature, and also invariants (unit vectors in,
  `(lon, lat)` order), ordering constraints, error modes and cost (an
  O(n²) edge build is interface).
- **Implementation** — everything behind the interface that a caller
  does not have to know.
- **Depth** — how much behaviour a caller gets per unit of interface
  they must learn. Deep is a small interface over a large
  implementation. Shallow is the reverse.
- **Seam** — a place where behaviour can change without editing that
  place. Where a module's interface lives.
- **Adapter** — a concrete thing that satisfies an interface at a seam.
- **Leverage** — what depth gives callers: one implementation that pays
  back across many call sites.
- **Locality** — what depth gives maintainers: change and verification
  in one place.

## Two tests

**The deletion test.** Delete the module and inline it at every call
site. If the complexity goes away, it was a pass-through and deleting it
was right. If the same complexity comes back at each caller, it earns
its place. `arc_visible` passes: without it, each caller repeats the
edge-crossing loop and the midpoint check for chords inside the polygon.

**One adapter is a hypothetical seam; two is a real one.** Do not add a
seam until something actually varies across it. For example, do not add
a pluggable obstacle or pathfinder interface while there is one obstacle
source and one search. Add it when a second one (real borders next to
the hand-made triangle, cuGraph next to networkx) actually exists.

## Shallow on purpose is allowed

A module may be shallow when it exists for **locality**, not leverage —
for example `gcd_km`, which keeps the one place where `R_EARTH` turns an
angle into kilometres. When you write a shallow module on purpose, say
so in its docstring and say what is wrong with making it deeper.
Otherwise the next reader thinks the thinness is a mistake and "fixes"
it.

## Prefer general-purpose over special-purpose

A module that answers a general question is usually deeper than several
that answer specific ones. The sphere helpers take unit vectors and know
nothing about airports or countries, so they work for any point set.
Keep it that way. The counter-case: do not merge two different
procedures into one function steered by flags. Two honest special
cases are better than special cases behind a general interface.

## Errors: define them out of existence where you can

Prefer a design where an error cannot occur over one that detects and
reports it. Convert to unit vectors at the boundary (on load), so no
inner function gets degrees by mistake. Where an error is possible,
make it name itself: an `assert` with a message such as
`'UAAA is inside the obstacle'`, or a named exception, not a bare
`KeyError` three calls later.

## Comments carry what the code cannot

Write the comment that says *why*, the constraint, or the thing that
will surprise the next reader — not what the next line does. Good
examples here: why the ring is clockwise (d3 fill side), why `angle()`
uses `atan2` (stable near 0 and π), why the midpoint check exists
(chords between vertices of the same polygon cross no edge).
