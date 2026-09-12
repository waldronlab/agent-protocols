# Writing a protocol

[`CONTRIBUTING.md`](CONTRIBUTING.md) covers the mechanics — where files go, how to bump a version, how
to record a review. [`PROTOCOL_STANDARD.md`](https://github.com/waldronlab/agent-protocol-standard/blob/main/PROTOCOL_STANDARD.md)
defines the format. This file is about the part neither of those can check for you: deciding what a
protocol should contain, and where one protocol ends and the next begins.

## A protocol is prose, not code

Write what to do and why, precisely enough that two people — or two agents, working in two languages —
get the same answer. Not an R script with comments. If a step can only be understood by reading an
implementation, it is not finished.

This is harder than it sounds, and it is the point. Code hides decisions inside library defaults:
which linkage, which tie-breaker, which denominator, what happens to a missing value. Prose forces
each one into the open, where it can be argued with. Most of the work of writing a protocol is
discovering the decisions your reference implementation was making silently.

[`independent-filtering-variance`](protocols/independent-filtering-variance/protocol.md) is the model
to imitate for tone and level of detail.

## One method, one citation, and cite its origin

An atomic protocol carries exactly one `method_citation`: the primary literature where the method was
**originally published**. Not the paper you took the analysis from. Not the paper that made the method
popular. The one that proposed it.

This takes real work and it is part of the task, not a formality. The failure mode is stopping one
step too early. In a recent batch, leave-one-dataset-out validation looked like it originated in
Pasolli et al. 2016 — a microbiome machine-learning paper — until the question got pushed further
back to Riester et al. 2014, which introduced the term and the procedure in ovarian cancer expression
data. Pasolli is the first *microbiome* use, which is a different and much narrower claim. Both facts
belong in the protocol; only one of them belongs in `method_citation`.

Expect to find that a method you assumed was invented by the lab's own paper is decades older, and
occasionally the reverse. Where a lab paper genuinely did propose the method, citing it is correct —
that is the exception, not the pattern.

## The four citation and DOI fields

Four frontmatter fields hold a DOI, and each name says what it identifies. The `*_citation` fields
reference other works; the `*_doi` fields identify things.

| Field | Identifies | Points at |
|---|---|---|
| `method_citation` | the **method** this protocol performs | the primary literature that first proposed it |
| `protocol_citation` | a publication **describing or validating this protocol** | the procedure as written here, including its parameterization |
| `artifact_doi` | **this document**, as a citable artifact | a DOI minted for it, e.g. from protocols.io |
| `collection_doi` | the **repository or collection** housing it | a Zenodo record |

`method_citation` and `protocol_citation` are the pair worth dwelling on, because both can point at a paper
containing the method. The difference is what the paper is credited for, and it runs along a specific axis:
**`method_citation` describes the method in general; `protocol_citation` describes precise usage** — the
parameter values, thresholds and choices this protocol fixes.

**That means one method can be the basis of several protocols.** Random forest classification is one method
with one origin, but two published parameterizations of it are genuinely different procedures that produce
different results from the same inputs. Each is its own protocol. They share a `method_citation` and are told
apart by their `protocol_citation`.

This is worth holding onto, because it is what keeps the one-method-one-citation rule from forcing unlike
procedures into a single document. If two candidate protocols perform the same method but fix different
parameters from different published sources, the rule is not telling you to merge them. Sibling protocols are
a legitimate shape.

**The test: if you rewrote the protocol's steps, would the DOI still be right?**

- `method_citation` — yes. A method's origin does not change when you revise how you describe it. It changes
  only if you change *which method* the protocol performs, which makes it a different protocol.
- `protocol_citation` — no. It is coupled to the protocol's content. Revise the steps away from what the
  paper describes and the paper no longer describes this protocol.

A worked case: a protocol for random forest classification of microbiome profiles using the hyperparameters
from Pasolli et al. 2016. Breiman 2001 proposed random forests — `method_citation`. Pasolli 2016 published
this particular parameterization for this kind of data — `protocol_citation`. Neither field is
optional-by-preference here; they record two different facts, and dropping either loses something a reader
needs.

**A composite protocol has no `method_citation`.** It proposes no method — it composes protocols that do, and
inherits their citations. A paper describing the pipeline as a whole is a `protocol_citation`.

Most atomic protocols will have `method_citation` and nothing else. Reach for `protocol_citation` only when a
paper really does describe or validate the procedure as you have written it — not merely when a paper
used the method.

## If you cannot name one paper that proposed everything the protocol does, it is more than one protocol

This is the most useful test available, because it is mechanical. Run it before you start writing.

It repeatedly splits protocols that felt like one thing:

- **Microbe set enrichment** looked like a single protocol with a method parameter. ORA, PADOG and
  CBEA have three citations, three null hypotheses, and three different input types — one thresholded
  list, one labelled abundance matrix, one compositional matrix. Three protocols, plus a composite
  that benchmarks them.
- **Prevalence filtering and the centred log-ratio transformation** were drafted as one preprocessing
  protocol. Downstream analyses need to be runnable with
  the transformation and without it. Splitting them made the transformation an optional slot rather
  than a mandatory step.
- **PERMANOVA and ANOSIM** bundle two tests by two authors with two null hypotheses. Taking a distance matrix as
  input rather than an abundance table removed a third bundled method at the same time: Aitchison
  distance is Euclidean distance on CLR output, so it belongs to the transformation protocol.
- **Random forest and leave-one-dataset-out validation** are used independently of each other in both
  directions. One protocol says how to fit the model; another says how to evaluate it.

A recurring theme: a bundled protocol may *feel* atomic if one paper happened
to do all of it at once. But that is a fact about the paper, not about the methods.

## When to stop splitting

The test above only pushes one direction, so it needs a floor. Without one it regresses forever: the
paper behind CLR cites prior mathematics, the paper behind PERMANOVA cites permutation testing, and
nothing is ever atomic.

**The regress does not actually happen, because a citation is not a method boundary.** The question is
not whether this protocol's `method_citation` has citations of its own. It is whether the protocol *performs*
more than one separately-nameable method as separable steps. Aitchison's paper rests on prior work, but
the protocol does not perform "take a logarithm" as a decision anyone makes independently — CLR without
the logarithm is not CLR with a different option selected, it is not CLR. Arithmetic inside a method is
not itself a method.

**An atomic protocol is a composable unit that is useful as a whole in building analyses.** That is the
floor, and it resolves into three questions. Split only if all three answer yes:

1. **Would two different analyses use this on its own?** If a piece only ever appears inside one larger
   thing, splitting it buys no reuse.
2. **Is there a plausible substitute?** The sharpest of the three. Split at the seams where real
   analyses vary. CLR has substitutes — arcsin-square-root, or no transformation at all — so the
   transformation is a unit. "Divide by the geometric mean" has no substitute within CLR, so it is not.
3. **Does it have a name practitioners use?** A named method is citable; a step inside one is not. This
   is usually why the naming question and the citation question give the same answer.

**A calibration that is easier to apply than any of them:** the right grain is roughly what a methods
section names in a clause. "Abundances were CLR-transformed" is one clause and one protocol. No methods
section writes "we divided by the geometric mean and took logarithms", because naming CLR already
implies it. If you would have to spell it out, it is a protocol. If naming something else already
implies it, it is a step.

**Splitting is not free, and that is what makes this a real limit rather than a preference.** Every
split adds a `protocols_used` edge, and a version bump has to propagate through every composite that
depends on it — permanent bookkeeping, not a one-time cost. It also adds a review burden and forces a
reader to assemble the picture from more pieces. A split has to buy reuse or substitutability. If it
buys neither, it is cost with no return.

Where the three questions are genuinely close, lean toward splitting: under-splitting hides a decision,
over-splitting only creates friction, and hiding decisions is the worse failure for a standard whose
purpose is making them explicit. But that is a tiebreaker, not a licence to skip the questions.

**Expect some cases to stay arguable.** Over-representation analysis is essentially Fisher's exact test
applied to sets; not splitting the test out is a judgement about what is independently composable in
this domain, not something the rules derive. A frequency table paired with a binomial test of direction
is defensibly one protocol — the counting rules and the test's unit of independence are the same
decision — and defensibly two. Make the call, and say in `## Notes` why.

## Alternatives inside a protocol are allowed — sometimes

A protocol may offer a genuine choice when the alternatives come from the same source, take the same
inputs, produce the same output type, and are interpreted the same way. The choice is then a tuning
knob, not a different method.

`independent-filtering-variance` is the precedent: `filter: variance | mean`, both proposed in the
same paper, one `method_citation` covering both, with the protocol stating that exactly one must be chosen,
that they must not be applied in sequence, and that the choice is recorded before results are seen.

The `method_citation` field is the tell. If the alternatives need two citations, they are two protocols.

This matches how protocols work elsewhere in science: Nature Protocols and protocols.io support
branch points within a protocol freely, but for procedural variants of one method — not for swapping
in a different method by a different author. OECD Test Guidelines draw the same line, covering several
assay variants in one guideline only where they are validated as concordant.

## Say what the protocol does *not* do

Every protocol needs an explicit out-of-scope statement, and it is often more valuable than the steps.
It is what lets a reader compose protocols without wondering whether two of them overlap, and it is
what stops a protocol from quietly growing into a pipeline.

`independent-filtering-variance` ends by stating that it does not define or perform hypothesis testing,
p-value calculation, multiple-testing adjustment, differential expression, or interpretation of
discoveries. That sentence does more work than any of its steps.

## Write down what must be recorded

A protocol that produces a number without a record of how it was produced is not reproducible, whatever
the steps say. Name the parameters, thresholds, versions, and intermediate counts that an execution must
report: the data version, the value of every parameter, how many features or samples survived each
filtering step, which tie-breaker was used, what happened to missing values.

Where a choice must be made before seeing results — a filter, a threshold, a candidate list — say so as
a step, and require that it be recorded. Prespecification that is only implied is prespecification that
did not happen.

## When there is no primary source

Some methods are old enough, or folkloric enough, to have no identifiable first publication. When that
is genuinely the case, say so in the pull request rather than reaching for a convenient recent paper —
citing an application paper to fill the field is exactly the error this guide is trying to prevent.
`method_citation` is optional in the standard; a purely descriptive or reporting protocol may have nothing to
cite at all.

Be careful about the scope of that permission. It covers protocols that **do not claim a method** —
a study-characteristics table, a corpus summary, a cohort-assembly procedure. An atomic protocol that
*does* perform a named method is a different case: if you cannot find its source, that is a question to
escalate, not a field to leave blank. The standard requires exactly one `method_citation` for an atomic
protocol, so quietly omitting it produces something that should not validate.

If you hit this, raise it as an issue in
[`waldronlab/agent-protocol-standard`](https://github.com/waldronlab/agent-protocol-standard/issues).
The standard may need a way to express "classical method, no primary source", and that is a decision
about the format, which is not made in this repository.

## Before you open the pull request

- For an atomic protocol claiming a method: it describes one method, and you can name the paper that
  proposed it. For a composite: it adds no steps of its own and carries no `method_citation`. For a
  descriptive or reporting protocol: it claims no method, and says so.
- It is a unit someone would compose into more than one analysis, with a plausible substitute.
- `method_citation` names the method's origin; `protocol_citation`, if present, describes this procedure.
- If a sibling protocol shares your `method_citation`, `## Notes` says how yours differs.
- Someone could execute it without reading any code.
- Every *optional* parameter has a stated default and a reason. Required inputs — a dataset, a
  covariate, a candidate list — are named as required, with no invented default.
- The out-of-scope section exists and is specific.
- The validator passes:

  ```sh
  git clone https://github.com/waldronlab/agent-protocol-standard.git
  Rscript agent-protocol-standard/scripts/validate-protocol.R protocols
  ```

  Note that the validator checks conformance, not correctness. It cannot tell you that you cited the
  wrong paper, bundled two methods, or left a decision implicit. Those are what review is for.

The format itself is pre-1.0 (`spec_version: 0.1.0`) and still changing. If a rule here gets in the way
of describing a method honestly, that is worth raising as an issue rather than working around — the
standard is young enough to be corrected.
