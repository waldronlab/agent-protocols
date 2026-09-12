# Writing a protocol

[`CONTRIBUTING.md`](CONTRIBUTING.md) covers mechanics: where files go, how to bump a version, how to
record a review. [`PROTOCOL_STANDARD.md`](https://github.com/waldronlab/agent-protocol-standard/blob/main/PROTOCOL_STANDARD.md)
defines the format. This file covers what neither can check for you — what goes in a protocol, and
where one protocol ends and the next starts.

## Prose, not code

Write what to do and why, precisely enough that two people, or two agents working in two languages,
get the same answer. Not an R script with comments. If a step only makes sense once you've read an
implementation, it isn't done.

That's harder than it sounds, and it's the point. Code hides decisions in library defaults: which
linkage, which tie-breaker, which denominator, what happens to a missing value. Prose drags each one
into the open where someone can argue with it. Most of the work is finding the decisions your
reference implementation was making silently.

Copy [`independent-filtering-variance`](protocols/independent-filtering-variance/protocol.md) for tone
and level of detail.

## Cite the method's origin, not its users

An atomic protocol carries exactly one `method_citation`: the primary literature where the method was
first published. Not the paper you took the analysis from. Not the paper that made it popular. The one
that proposed it.

This is real work, not a formality, and the failure mode is stopping one step early. Leave-one-dataset-out
validation looked like it came from Pasolli et al. 2016, a microbiome ML paper, until we pushed further
back to Riester et al. 2014, which introduced the term and the procedure in ovarian cancer expression
data. Pasolli is the first *microbiome* use — a much narrower claim. Both facts belong in the protocol,
only one belongs in `method_citation`.

Expect to find that a method you assumed came from one of our own papers is decades older. Sometimes
it's the other way round, and citing our paper is right. That's the exception.

## The four citation and DOI fields

Each name says what it identifies. `*_citation` points at other work; `*_doi` identifies a thing.

| Field | Identifies | Points at |
|---|---|---|
| `method_citation` | the **method** this protocol performs | the primary literature that proposed it |
| `protocol_citation` | a publication **describing or validating this protocol** | the procedure as written here, parameters included |
| `artifact_doi` | **this document**, as a citable artifact | a DOI minted for it, e.g. from protocols.io |
| `collection_doi` | the **repository or collection** housing it | a Zenodo record |

The first two get confused, because both can point at a paper containing the method. The difference:
`method_citation` is the method in general, `protocol_citation` is precise usage — the parameter values,
thresholds and choices this protocol fixes.

So one method can be the basis of several protocols. Random forest classification is one method with one
origin, but two published parameterizations are different procedures that give different answers from the
same data. Each is its own protocol. They share a `method_citation` and differ in `protocol_citation`.

Worth remembering, because it's what stops the one-method-one-citation rule from forcing unlike procedures
into one document. If two candidate protocols run the same method with different parameters from different
sources, the rule isn't telling you to merge them. Siblings are fine.

If you're unsure which field a DOI belongs in, ask whether it would still be right if you rewrote the
steps. A method's origin survives a rewrite, so that's `method_citation`. A paper describing this
procedure doesn't, because it would no longer be the procedure described — `protocol_citation`.

Worked case: random forest classification of microbiome profiles with Pasolli's hyperparameters. Breiman
2001 proposed random forests, so that's `method_citation`. Pasolli 2016 published this parameterization
for this kind of data, so that's `protocol_citation`. Two different facts; drop either and a reader loses
something.

A composite has no `method_citation`. It proposes no method — it composes protocols that do and inherits
their citations. A paper describing the whole pipeline is a `protocol_citation`.

Most atomic protocols will have `method_citation` and nothing else. Use `protocol_citation` only when a
paper really does describe the procedure as you've written it, not just use the method.

## If one paper didn't propose everything the protocol does, it's more than one protocol

The most useful test available, because it's mechanical. Run it before you start writing.

It keeps splitting things that felt like one protocol:

- **Microbe set enrichment.** Looked like one protocol with a method parameter. ORA, PADOG and CBEA have
  three citations, three null hypotheses, and three different input types — a thresholded list, a labelled
  abundance matrix, a compositional matrix. Three protocols, plus a composite that benchmarks them.
- **Prevalence filtering and CLR.** Drafted as one preprocessing protocol. Downstream analyses need to run
  with the transformation and without it, so splitting made CLR an optional slot instead of a mandatory step.
- **PERMANOVA and ANOSIM.** Two tests, two authors, two nulls. Taking a distance matrix as input instead of
  an abundance table dropped a third bundled method while we were at it: Aitchison distance is Euclidean
  distance on CLR output, so it belongs to the transformation protocol.
- **Random forest and LODO.** Each gets used without the other. One protocol says how to fit the model,
  another says how to evaluate it.

The pattern: a bundled protocol feels atomic when one paper happened to do all of it at once. That's a fact
about the paper, not about the methods.

## When to stop splitting

That test only pushes one way, so it needs a floor. Without one it never terminates — the CLR paper cites
prior mathematics, the PERMANOVA paper cites permutation testing, and eventually you're citing Euclid.

It doesn't actually regress, because a citation isn't a method boundary. The question isn't whether your
`method_citation` has citations of its own. It's whether the protocol *performs* more than one
separately-nameable method as separable steps. Aitchison's paper rests on prior work, but the protocol
doesn't perform "take a logarithm" as a decision anyone makes on its own. CLR without the logarithm isn't
CLR with a different option selected, it's not CLR. Arithmetic inside a method isn't a method.

An atomic protocol is a composable unit that's useful as a whole in building analyses. That's the floor,
and it comes down to three questions. Split only if all three are yes:

1. **Would two different analyses use this on its own?** If it only ever shows up inside one bigger thing,
   splitting buys no reuse.
2. **Is there a plausible substitute?** The sharpest of the three. Split where real analyses vary. CLR has
   substitutes — arcsin-square-root, or no transformation — so the transformation is a unit. "Divide by the
   geometric mean" has no substitute inside CLR, so it isn't.
3. **Does it have a name people use?** A named method is citable; a step inside one isn't. Usually why the
   naming question and the citation question give the same answer.

Easier calibration: the right grain is what a methods section names in a clause. "Abundances were
CLR-transformed" is one clause and one protocol. Nobody writes "we divided by the geometric mean and took
logarithms", because naming CLR already covers it. If you'd have to spell it out, it's a protocol. If naming
something else already implies it, it's a step.

Splitting isn't free, which is what makes this a real limit and not a preference. Each split adds a
`protocols_used` edge, and every version bump then has to propagate through the composites that depend on it.
That's permanent bookkeeping. It also adds review burden and makes a reader assemble the picture from more
pieces. A split has to buy reuse or substitutability; if it buys neither, it's pure cost.

When the three questions are close, lean toward splitting. Under-splitting hides a decision, over-splitting
just creates friction, and hiding decisions is the worse failure for a standard that exists to make them
explicit. That's a tiebreaker, not permission to skip the questions.

Some cases stay arguable. ORA is basically Fisher's exact test applied to sets, and not splitting the test
out is a judgement about what's independently composable here, not something the rules give you. A frequency
table plus a binomial test of direction is defensibly one protocol — same counting rules, same unit of
independence — and defensibly two. Make the call and say why in `## Notes`.

## Alternatives inside one protocol, sometimes

A protocol can offer a real choice when the alternatives come from the same source, take the same inputs,
produce the same kind of output, and mean the same thing. Then it's a tuning knob, not a different method.

`independent-filtering-variance` is the precedent: `filter: variance | mean`, both from the same paper, one
`method_citation` covering both, and the protocol says exactly one must be chosen, that they must not be
applied in sequence, and that the choice is recorded before anyone looks at results.

`method_citation` is the tell. If the alternatives need two citations, they're two protocols.

This is how protocols work elsewhere too. Nature Protocols and protocols.io allow branch points freely, but
for procedural variants of one method, not for swapping in a different method by a different author. OECD
Test Guidelines draw the same line — several assay variants in one guideline only where they're validated as
concordant.

## Say what the protocol doesn't do

Every protocol needs an explicit out-of-scope statement, and it's often worth more than the steps. It lets a
reader compose protocols without wondering whether two of them overlap, and it stops a protocol quietly
growing into a pipeline.

`independent-filtering-variance` ends by saying it doesn't define or perform hypothesis testing, p-value
calculation, multiple-testing adjustment, differential expression, or interpretation of discoveries. That
sentence does more work than any of its steps.

## Say what has to be recorded

A protocol that produces a number with no record of how isn't reproducible, whatever the steps say. Name the
parameters, thresholds, versions and intermediate counts an execution has to report: data version, the value
of every parameter, how many features or samples survived each filtering step, which tie-breaker was used,
what happened to missing values.

Where a choice has to be made before seeing results — a filter, a threshold, a candidate list — make it a
step and require it to be recorded. Prespecification that's only implied didn't happen.

## When there's no primary source

Some methods are old enough, or folkloric enough, that there's no identifiable first publication. If that's
genuinely the case, say so in the pull request instead of reaching for a convenient recent paper. Citing an
application paper to fill the field is the exact error this guide is trying to prevent.

Be careful how far that goes. It covers protocols that claim no method — a study-characteristics table, a
corpus summary, a cohort-assembly procedure. An atomic protocol that *does* run a named method is different:
if you can't find the source, escalate it, don't leave the field blank. The standard wants exactly one
`method_citation` for an atomic protocol, so quietly omitting it produces something that shouldn't validate.

Either way, open an issue in
[`waldronlab/agent-protocol-standard`](https://github.com/waldronlab/agent-protocol-standard/issues). The
standard may need a way to say "classical method, no primary source", and that's a decision about the format,
which doesn't get made in this repository.

## Before you open the pull request

- Atomic protocol claiming a method: one method, and you can name the paper that proposed it. Composite: no
  steps of its own, no `method_citation`. Descriptive or reporting protocol: claims no method, and says so.
- It's a unit someone would use in more than one analysis, and there's a plausible substitute for it.
- `method_citation` is the method's origin. `protocol_citation`, if present, describes this procedure.
- If a sibling protocol shares your `method_citation`, `## Notes` says how yours differs.
- Someone could run it without reading any code.
- Every *optional* parameter has a stated default and a reason. Required inputs — a dataset, a covariate, a
  candidate list — are named as required, with no invented default.
- The out-of-scope section exists and is specific.
- The validator passes:

  ```sh
  git clone https://github.com/waldronlab/agent-protocol-standard.git
  Rscript agent-protocol-standard/scripts/validate-protocol.R protocols
  ```

  It checks conformance, not correctness. It can't tell you that you cited the wrong paper, bundled two
  methods, or left a decision implicit. That's what review is for.

The format is pre-1.0 (`spec_version: 0.1.0`) and still moving. If a rule here gets in the way of describing
a method honestly, raise it as an issue rather than working around it. The standard is young enough to fix.
