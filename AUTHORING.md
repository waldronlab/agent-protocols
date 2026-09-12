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

An atomic protocol carries exactly one citation: the primary literature where the method was
**originally published**. Not the paper you took the analysis from. Not the paper that made the method
popular. The one that proposed it.

This takes real work and it is part of the task, not a formality. The failure mode is stopping one
step too early. In a recent batch, leave-one-dataset-out validation looked like it originated in
Pasolli et al. 2016 — a microbiome machine-learning paper — until the question got pushed further
back to Riester et al. 2014, which introduced the term and the procedure in ovarian cancer expression
data. Pasolli is the first *microbiome* use, which is a different and much narrower claim. Both facts
belong in the protocol; only one of them belongs in `citation`.

Expect to find that a method you assumed was invented by the lab's own paper is decades older, and
occasionally the reverse. Where a lab paper genuinely did propose the method, citing it is correct —
that is the exception, not the pattern.

## `citation`, `publication_doi`, and the other two DOI fields

Four frontmatter fields hold a DOI, and they are not interchangeable. The axis that separates them is
**what is being identified** — the method, this document, a paper about this document, or the
collection it lives in.

| Field | Identifies | Points at |
|---|---|---|
| `citation` | The **method** this protocol performs | The primary literature that first proposed it |
| `publication_doi` | A **peer-reviewed paper that describes or validates this protocol** | A publication of the procedure as written here |
| `protocol_doi` | **This protocol artifact** | A DOI minted for this document, e.g. from protocols.io |
| `repository_doi` | The **collection** housing it | A Zenodo record for the repository |

`citation` and `publication_doi` are the pair that get confused, because both can point at a paper that
contains the method. The difference is what the paper is being credited for. `citation` credits an
invention; `publication_doi` credits a description of this specific procedure.

**The test: if you rewrote the protocol's steps, would the DOI still be right?**

- `citation` — yes. A method's origin does not change when you revise how you describe it. It changes
  only if you change *which method* the protocol performs, which makes it a different protocol.
- `publication_doi` — no. It is coupled to the protocol's content. Revise the steps away from what the
  paper describes and the paper no longer describes this protocol.

A worked case: a protocol for random forest classification of microbiome profiles, using the
hyperparameters from Pasolli et al. 2016. Breiman 2001 proposed random forests, so it goes in
`citation`. Pasolli 2016 is the published description of this particular parameterization for this
particular kind of data, so it goes in `publication_doi`. Neither field is optional-by-preference here
— they record two different facts, and dropping either loses information a reader needs.

Most atomic protocols will have `citation` and nothing else. Reach for `publication_doi` only when a
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
not whether this protocol's citation has citations of its own. It is whether the protocol *performs*
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
same paper, one citation covering both, with the protocol stating that exactly one must be chosen,
that they must not be applied in sequence, and that the choice is recorded before results are seen.

The citation field is the tell. If the alternatives need two citations, they are two protocols.

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
`citation` is optional in the standard; a purely descriptive or reporting protocol may have nothing to
cite at all.

If you hit this, raise it as an issue in
[`waldronlab/agent-protocol-standard`](https://github.com/waldronlab/agent-protocol-standard/issues).
The standard may need a way to express "classical method, no primary source", and that is a decision
about the format, which is not made in this repository.

## Before you open the pull request

- The protocol describes one method, and you can name the paper that proposed it.
- It is a unit someone would compose into more than one analysis, with a plausible substitute.
- `citation` names the method's origin; `publication_doi`, if present, describes this procedure.
- Someone could execute it without reading any code.
- Every parameter has a stated default and a reason.
- The out-of-scope section exists and is specific.
- The validator passes:

  ```sh
  git clone https://github.com/waldronlab/agent-protocol-standard.git
  Rscript agent-protocol-standard/scripts/validate-protocol.R protocols
  ```

  Note that the validator checks conformance, not correctness. It cannot tell you that you cited the
  wrong paper, bundled two methods, or left a decision implicit. Those are what review is for.
