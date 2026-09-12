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
that proposed it. This is important to ensuring accuracy of citation of original literature. 

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

Example: "We employed random forest classification (`method_citation`) as implemented by Pasolli et al. 2016. (`protocol_citation`)."

Most composites have no `method_citation` — they sequence existing protocols, propose nothing new, and
inherit their constituents' citations. A paper describing that kind of pipeline is a `protocol_citation`.
But a string of methods can be published as a method in its own right, and then `method_citation` is right:
a two-stage hierarchical meta-analysis is a method, and so is `humann4-database-build`. Ask whether the
paper proposes the combination as an approach people would cite, or just describes running the steps.

Atomic protocols can have `method_citation` and nothing else. Use `protocol_citation` only when a
paper really does describe the procedure as you've written it, which is in some way distinguished from the general method.

## If one paper didn't propose everything the protocol does, it's more than one protocol

Examples from previous reviews of protocol proposals:

- **Microbe set enrichment.** Looked like one protocol with a method parameter. ORA, PADOG and CBEA have
  three citations, three null hypotheses, and three different input types — a thresholded list, a labelled
  abundance matrix, a compositional matrix. Three protocols, plus a composite that benchmarks them.
- **Prevalence filtering and CLR.** Drafted as one preprocessing protocol. Downstream analyses need to run
  with the transformation and without it, so splitting made CLR an optional slot instead of a mandatory step.

The pattern: a bundled protocol may feel atomic when one paper happened to do all of it at once, but that's a fact
about the paper, not about the methods.

## When to stop splitting

An atomic protocol is a composable unit that's useful as a whole in building analyses. Do not split further than necessary. Split only if all three are yes:

1. **Would two different analyses use this on its own?** If it only ever shows up inside one bigger thing,
   splitting buys no reuse.
2. **Is there a plausible substitute?**  Split where real analyses vary. CLR has
   substitutes — arcsin-square-root, or no transformation — so the transformation is a unit. "Divide by the
   geometric mean" has no substitute inside CLR, so it isn't.
3. **Does it have a name people use?** A named method is citable; a step inside one isn't. Usually why the
   naming question and the citation question give the same answer.

Another way to think about it is: in a methods section, would you say "we did X", ie name the procedure? If so, it's a protocol. Or would you say, "we did X as part of Y". If so, X is likely a step in protocol Y.

For example, "Abundances were
CLR-transformed" is one clause and one protocol. Nobody writes "we divided by the geometric mean and took
logarithms", because naming CLR already covers it. CLR can be considered an atomic protocol with two steps. 

Some cases are arguable.  Make the call and say why in `## Notes`.

## Alternatives inside one protocol, sometimes

A protocol can offer a choice when the alternatives come from the same source, take the same inputs,
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

Provide an explicit out-of-scope statement.  It lets an agent
compose protocols without wondering whether two of them overlap, and it stops a protocol from quietly growing into a pipeline when the protocol is often followed by other downstream analysis.

For example, `independent-filtering-variance` ends by saying it doesn't define or perform hypothesis testing, p-value
calculation, multiple-testing adjustment, differential expression, or interpretation of discoveries. 

## Say what has to be recorded

Name the parameters, thresholds, versions and intermediate counts an execution has to report: for example the values
of every parameter, how many features or samples survived each filtering step, which tie-breaker was used,
what happens to missing values.

Where a choice has to be made — a filter, a threshold, a candidate list — make it a step and require it to be recorded. No choice should be implied.

## When you can't find a source

Rare, but some methods are old or folkloric enough to have no identifiable first publication. Say so in the
pull request instead of reaching for a convenient recent paper.

A missing `method_citation` validates today, so nothing stops you leaving it out. Descriptive protocols that
claim no method — a study table, a corpus summary — legitimately have nothing to cite. For a protocol that
does run a named method, an empty field is a question to raise, not an answer.

Raise it in [`waldronlab/agent-protocol-standard`](https://github.com/waldronlab/agent-protocol-standard/issues)
— the standard may need a way to say "classical method, no primary source", and format decisions don't get
made in this repository.

## Before you open the pull request
 
 Check:
- For an atomic protocol claiming a method, that it is one method, and you can name the paper that proposed
  it. Composite: its steps name the constituent protocols and the order, without restating what those
  protocols already say. Descriptive or reporting protocol: claims no method, and says so.
- It's a unit someone would use in more than one analysis, and there's a plausible substitute for it.
- `method_citation` is the method's origin. `protocol_citation`, if present, describes this procedure.
- If a sibling protocol shares your `method_citation`, `## Notes` says how yours differs.
- An expert in the field could have a complete picture of the protocol without reading any code.
- Every *optional* parameter has a stated default and a reason. Required inputs — a dataset, a covariate, a
  candidate list — are named as required, with no unstated default.
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
