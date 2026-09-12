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

## If you cannot name one paper that proposed everything the protocol does, it is more than one protocol

This is the most useful test available, because it is mechanical. Run it before you start writing.

It repeatedly splits protocols that felt like one thing:

- **Microbe set enrichment** looked like a single protocol with a method parameter. ORA, PADOG and
  CBEA have three citations, three null hypotheses, and three different input types — one thresholded
  list, one labelled abundance matrix, one compositional matrix. Three protocols, plus a composite
  that benchmarks them.
- **Prevalence filtering and the centred log-ratio transformation** were drafted as one preprocessing
  protocol. Different sources, and — more practically — downstream analyses need to be runnable with
  the transformation and without it. Splitting them made the transformation an optional slot rather
  than a mandatory step.
- **PERMANOVA and ANOSIM** bundle two tests by two authors with two nulls. Taking a distance matrix as
  input rather than an abundance table removed a third bundled method at the same time: Aitchison
  distance is Euclidean distance on CLR output, so it belongs to the transformation protocol.
- **Random forest and leave-one-dataset-out validation** are used independently of each other in both
  directions. One protocol says how to fit the model; another says how to evaluate it.

The recurring shape: the thing that makes a bundled protocol *feel* atomic is that one paper happened
to do all of it at once. That is a fact about the paper, not about the methods.

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
