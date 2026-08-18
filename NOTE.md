# A written rule has no failure mode

*Working note, 15 August 2026. Merlini, with Fede (@babyblueviper1) and Pavlo (@pipavlo82).*

## 0 · Where this started

Three days ago, on an admin page nobody was looking at. We were onboarding a post-quantum key
for an agent, and the question was narrow: if we rotate the key and cut a new anchor epoch,
does the earlier agent survive it?

Three days later the answer is yes. Almost none of what follows is about that.

## 1 · The rule

On 4 August the group wrote down an invariant:

> Never let a collapsed value hide which state produced it. The marker travels **with** the
> value.

It came from a specific defect and it is correct. Everyone agreed. It got cited by name in
reviews. Two of us applied it, independently, in three different places, and got it right
each time.

Then, between 14 and 15 August, we reintroduced it five times.

## 2 · The instrument, not the system

The onboarding test ran on live data: a fresh agent authorized, a new anchor epoch cut — epoch
3, 29 bindings, tx on Base Sepolia — and every agent in that root recomputed against its own
current binding. **29 of 29 up to date, across five registries.** Rotation, authorization,
anchoring and the panel all held.

We reported two defects that afternoon. Both were ours.

The first: a checker that matched agents on `agent_id` alone, across an anchor spanning five
registries, and so compared agent 7 of one registry against agent 7 of another and called the
difference staleness. It produced "4 of 12 stale" with confidence. The panel it was checking
filtered on registry *and* id, correctly, in code read twenty minutes earlier.

The second: a claim that owner authorization was "published nowhere", from checking two of
three endpoints and generalising. It is published in full, on the third.

Neither was a wrong answer. Both were **correct answers to an adjacent question, delivered with
numbers attached.**

## 3 · The refinement that made it tractable

Fede's contribution, and the one that turned a mood into a method:

> A negative control catches that a check discriminates *at all*. A scope bug only shows up
> against a control built at the **same granularity as the real claim** — a two-registry
> fixture, not a one-registry one. The control has to match the claim's dimensionality, or it
> passes cleanly while still checking the adjacent thing.

The test of a rule is what it finds within the hour. This one found a live defect.

Our `/binding` endpoint published an owner authorization that verified perfectly — valid
signature, correct owner, nothing forged — **for a key that had been rotated out.** A verifier
following our own printed instruction would reach a true conclusion about the wrong key.

The population is the point: **28 of 29 agents sit at key epoch 0**, where the authorized key
and the in-force key are the same, and the endpoint is entirely correct. One agent had two
epochs. Any control built against a single-epoch agent passes forever.

## 4 · The audit, and the sharper rule

Fede then asked the right follow-up: any record written once at bind time, sitting beside a
field that tracks current state, is structurally the same risk — go and look for others.

We did. **Zero further instances**, and the reason is worth more than a longer list would have
been. Four such pairs exist in that schema. Three are already correct:

| pair | why it holds |
|---|---|
| mint-time capability assignment ↔ expiring entitlement | authority is re-checked at **call** time |
| historical signature ↔ current key | the signature **carries the key it was signed with** |
| source binding at mint ↔ mutating owner | both sides are **re-read live** |
| owner authorization ↔ in-force key | *the one that broke* |

So "write-once beside mutating" is not the risk condition — three of four such pairs are fine.
The condition is whether the record **carries its own scope forward into the published claim**.
The signature carries its key. The source binding recomputes. The ENS link deletes itself on
transfer. The authorization stored its key and then published a message that named no epoch and
compared nothing.

Which is the 4 August rule exactly. Applied correctly three times, missed once, by the same
hands.

## 5 · Five in two days

Counted honestly, because the count is the finding:

1. A currency verdict collapsing *resolver unreachable* and *no local ipfs* into one
   `UNDETERMINED`.
2. Our authorization endpoint, above.
3. A `review_unavailable` marker reachable from four distinct causes, all printing one string.
4. **`UNDETERMINED` itself** — introduced to stop a two-way collapse, and immediately
   collapsing two causes of its own.
5. A claim that a cited ledger entry "did not resolve", from searching the wrong system.

Item 4 is the one that should end the argument. The remedy reintroduced the disease, one level
down, in the same commit that cured it.

## 6 · The conclusion we did not want

We had the rule. It was written, dated, agreed, and cited. We applied it correctly in at least
three places. And we reintroduced it five times in forty-eight hours.

That is not a discipline problem, and treating it as one predicts more of the same. It is a
property of the instrument:

> **A written rule has no failure mode. It cannot go red.**

Nothing in either codebase asked *does this marker have more than one cause behind it?* So not
reintroducing it depended entirely on whether someone happened to read the right line on the
right afternoon. Five in two days is what that dependency looks like when you measure it.

So the rule became a check: enumerate the terminal markers, trace the distinct branches that
reach each, fail when a marker answers more than one question with no reason travelling beside
it. It runs in CI, and it has its own controls, because a check that has only ever run against
clean code has not been shown to work — it has been shown to be silent, which is what a broken
one also looks like.

## 7 · The check was wrong too

On its first run against real code it flagged sample documents inside a self-test as a
collapsed marker. They were inert data, not two branches producing a state.

Its controls at that moment varied exactly one axis — reason present versus absent — and passed
cleanly while the tool was wrong on a second axis they never varied: emitted state versus inert
data. The same failure Fede had named the day before, inside the tool built to catch that
failure.

His summary is the best sentence anyone produced this week:

> Writing the rule down, then building the check for the rule, and still needing a real run
> against real code to find the axis the check itself wasn't varying. **Three layers deep and
> the pattern still got through the first two.**

The practical form: a new check must be run against code it did not come from before it is
trusted. The fixture that motivated it encodes only what you already understood, so it can only
confirm you.

## 8 · What actually got fixed

Three artifacts, each of which knew something it did not say:

- an authorization that knew which key epoch it covered, and published a bare signature;
- an enforcer that knew whether a signature had been *checked and failed* or *never checked*,
  and returned the same rejection for both;
- a binding that knew its own signature scheme — the anchored statement declares it — and
  published a bare public key, leaving verifiers to infer the family from key length.

All three shipped with the same shape of fix: a named field for the fact, a required reason
where a state has causes, and the legacy boolean kept only as a **derived projection** so it
can never be set independently. That shape is Fede's; Pavlo's formulation of why is the one to
keep — *don't collapse unresolved into true or false; make the third state explicit, and keep
legacy booleans as projections only.*

## 9 · The honest accounting

Most of what this week found was wrong with our instruments, not our system. The substrate
held: rotation survived, earlier agents came through unchanged, consent was recomputable by a
third party throughout.

That is a good result and it is not the interesting one. The interesting one is that a group
which writes invariants down, agrees them, and cites them by name still reintroduced one five
times in two days — and only stopped when something mechanical was watching.

Write the rule. Then build the thing that can fail.

## 10 · The same shape, one layer up (17–18 August)

Two days after this note closed, the pattern arrived again — not in a marker on a value, but in a
verdict on a check. Recorded here because it is the same rule at a higher layer, and because it
arrived twice more independently, which is this note's own kind of evidence.

A verdict is a value too. **A green that does not carry a bound witness of the procedure that
produced it collapses with a mere assertion of the same green.** "The check passed" and "I said the
check passed" are byte-identical unless something binds the computation that witnessed it into the
result — exactly as *authorized* and *authorized-for-the-key-that-was-rotated-out* were
byte-identical until the epoch travelled with the message (§4).

Two instances, from two threads, one invariant:

- **A run record** — recompute-kit `predicate_conformance.v0` (PR #14, as design/spec). A commit's
  position in the DAG proves a record existed no later than X; it cannot prove *this execution
  consumed that input.* So the spec splits the claim: `record_identity` (weaker — precommit before
  the recorded result, and the leg the gate actually implements) versus `execution_witness` (an
  attestation binding executed revision + consumed hash + result). Only the second lets a green mean
  "produced by this check" — and it is deliberately **out of scope in PR #14** until a real repo/CI
  witness surface exists. The design names the boundary; the witness leg is specified, not yet built.
- **A verification result** — observation-conditions-note §9, a candidate kept *named, not counted.*
  An agent applying this group's own recompute discipline stated a spec's hashes matched *without
  recomputing them* — the canonical bytes differed — and it was caught only when the recompute was
  re-run. Pavlo's boundary is the sharp one: a hash/diff produced *after* an assertion proves a
  *later* recompute occurred, not how the *original* green was made. The close is the same fix as
  the run record's: bind the witnessing computation — the recomputed digest, the mutation result —
  into the verdict.

Same fix, two objects: **bind the witness into the result, or it inherits provenance it did not
earn.** An independent convergence — Fede reached it from the DAG side, the §9 candidate from the
agent-drift side — not a coincidence of vocabulary.

**External corroboration, from the testing-discipline side.** Robert C. Martin's
*negative-test-experiment* wrote Hunt the Wumpus eight times — four disciplines × a forced
complexity metric on/off — producing eight distinct source trees, no shared checksum, and **every
one passes acceptance 25/0.** *"Acceptance does not distinguish the four."* He rates design by
reading the trees, *"not mutation, not acceptance"*: the verdict carries no trace of the discipline
or the design that produced it, reached empirically and independently. Two of his measurements
sharpen this note's own instruments:

- **Mutation-kill is a floor, not a ranking.** All eight programs killed ~94.5–99.2% of *covered*
  mutants (the experiment records uncovered sites separately) — the layered design and the single-file
  listing alike. A green mutation suite means "this check is not decorative," never "this is good."
- **A forced metric goes decorative** — a counter-case, kept separate. Forcing the complexity metric
  below its threshold *"multiplies names… CC ≤ 3 is visible; the hunt is not"*: coverage up, design
  flat. A control optimised *for* becomes a target that satisfies the letter — §7's "the check was
  wrong too," seen from the metric side.

The narrow, reads-only form of this stays in observation-conditions-note, deliberately (that note's
scope call). What belongs here is the general shape only: **a result must carry a bound witness of
the procedure that produced it.** Write the rule, then build the thing that can fail — and make the
green able to say it really ran.

*Found and first-drafted by Merlini (the citation, and the two-instance mapping); Pavlo sharpened
the §9 boundary and the run-record split. By offer — additions welcome.*

## 11 · The two-axis test for a load-bearing check (18 August)

§10 says a result must carry a bound witness of the procedure that produced it. The obvious
corollary — *prove the check can fail* — turned out to carry **two independent obligations**, and a
suite can satisfy one while silently failing the other. Both were caught the same week, one in each
of us:

- **Axis 1 — WHERE you run it.** recompute-kit PR #14's conformance suite ran `bun gate.ts --grade`,
  which printed results and exited 0 *unconditionally*; a validating path existed in the same file,
  but CI never called it. Every vector "passed" by construction — an evaluator returning wrong
  results but well-formed JSON stayed green forever. Right injection point, wrong entrypoint watched.
- **Axis 2 — WHERE you inject the fault.** A first attempt to show a checker could fail forced its
  comparison function to always return `true` — which proves only that the comparator *ran*,
  trivially true of any comparison-based check. The real test injects a claim-relevant semantic fault
  into the *domain logic that produces the answer* (flip one `!=` in the thing under test), leaves the
  comparison intact, and requires the pinned expected to go red. Right entrypoint, wrong injection
  layer.

**The rule, as the working group fixed it:** *A check is load-bearing on a claim only when a
claim-relevant semantic fault, injected upstream in the claim-producing logic, is carried through to
a red outcome by the exact command CI actually invokes. Passing either axis alone — the right
entrypoint with a plumbing-only injection, or the right injection point verified through a sibling CI
never calls — is insufficient.*

It closes the gap between "this checker can fail" and "the deployed verification path can detect this
class of wrong answer." A green that clears only one axis is the decoration §1 named, wearing a
mutation test as cover.

*Formulated with Fede (babyblueviper1) and Pavlo (pipavlo82) on the working-group thread,
18 August; Pavlo's "claim-relevant" scoping keeps the obligation to exactly what the check claims to
verify. The two instances above are ours — one miss each.*
