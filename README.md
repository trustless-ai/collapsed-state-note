# A written rule has no failure mode

A working-group note on collapsed states — and on what happened when we wrote the rule against
them down, agreed it, cited it by name, and then reintroduced it five times in forty-eight
hours.

**[Read the note → `NOTE.md`](NOTE.md)**

## What it argues

On 4 August 2026 the group recorded an invariant: *never let a collapsed value hide which state
produced it — the marker travels with the value.* It is correct, it was applied correctly in at
least three places, and between 14 and 15 August it came back five times, across two codebases,
from the two people who wrote it. Once **inside the fix for itself**: a third state introduced
to stop a two-way collapse immediately collapsed two causes of its own.

The conclusion is not that we were careless. It is that documentation is the wrong instrument
for this class of defect:

> A written rule has no failure mode. It cannot go red.

Nothing asked *does this marker have more than one cause behind it?*, so not reintroducing it
depended on someone reading the right line on the right afternoon. The note ends with the rule
rebuilt as a check that can fail — and with that check being wrong on its first real run, on an
axis its own controls never varied.

## What it is not

Not a specification, and nothing here is normative. It records three fixes to a live system and
one uncomfortable count. The system under test held throughout — rotation survived, earlier
agents came through unchanged, consent stayed independently recomputable. Almost everything the
week found was wrong with the *instruments*, which is the part worth writing down.

## Contributors

- **Merlini** — the arc, the audit, the checker, the three fixes
- **@babyblueviper1** — the control-dimensionality rule (*a control must match the claim's own
  dimensionality, or it passes while checking the adjacent thing*), and the closing observation
- **@pipavlo82** — the third-state formulation: *don't collapse unresolved into true or false;
  make the third state explicit, and keep legacy booleans as projections only*

## Related

- [`trustless-ai/composed-attestation-note`](https://github.com/trustless-ai/composed-attestation-note) — the seam rules for composing independent commitments
- [`trustless-ai/recompute-kit`](https://github.com/trustless-ai/recompute-kit) — `tools/check_collapsed_states.py`, the rule as a check that can go red

## Licence

[CC0 1.0 Universal](LICENSE). Public domain. Copy it, fork it, cite it or don't.
