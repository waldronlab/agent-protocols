# Writing a protocol

Your primary focus should be **clarity, absence of ambiguity, scientific correctness, and accurate citation**. CI will validate formatting, but cannot validate the science for you.

[`CONTRIBUTING.md`](CONTRIBUTING.md) covers mechanics: where files go, how to bump a version, how to
record a review. [`PROTOCOL_STANDARD.md`](https://github.com/waldronlab/agent-protocol-standard/blob/main/PROTOCOL_STANDARD.md)
defines the format. This file covers what neither can check for you — what goes in a protocol, and
where one protocol ends and the next starts.

## Prose, not code

Write what to do and why, precisely enough that two people, or two agents working in two languages,
get the same answer. Not an R script with comments. If a step only makes sense once you've read an
implementation, more detail is needed.

Note that code hides decisions such as default settings in library defaults, but such defaults can 
change with package versions or between different implementations of a method. A protocol should specify 
the procedure is enough detail not to be affected by such implementation differences.

See [`independent-filtering-variance`](protocols/independent-filtering-variance/protocol.md) for an 
example of appropriate tone and level of detail.

## The four citation and DOI fields

Each name says what it identifies. `*_citation` points at other work; `*_doi` identifies a thing.

| Field | Identifies | Points at |
|---|---|---|
| `method_citation` | the **method** this protocol performs | the primary literature that proposed it |
| `protocol_citation` | a publication **describing or validating this protocol** | the procedure as written here, parameters included |
| `artifact_doi` | **this document**, as a citable artifact | a DOI minted for it, e.g. from protocols.io |
| `collection_doi` | the **repository or collection** housing it | a Zenodo record |

Protocols carry one `method_citation` and zero or one `protocol_citation`:

- Atomic protocols can have method_citation and nothing else. Use protocol_citation only when a paper really does describe the procedure as you've written it, which is in some way distinguished from the general method.
- Composite protocols automatically inherit the `method_citation` from the protocols they comprise, provide one only if a primary publication describes the workflow as you've defined it. 
- When to use `protocol_citation`: If a paper describes running a specific sequence of existing methods (e.g., a published pipeline or a specific analysis workflow). There would be no `protocol_citation` only if you are writing a first definition of the protocol.
`method_citation` is the method in general, `protocol_citation` is precise usage: the parameter values,
thresholds and choices this protocol specifies. Be sure to cite the primary literature that proposed the method
or defined the protocol, not a later paper that used the method or protocol.

One method can be the basis of several protocols; for example, Random Forest classification is a method with one
origin, with many published parameterizations that give different answers from the
same data. Each is its own protocol; they share a `method_citation` but differ in `protocol_citation`.

If you're unsure which field a DOI belongs in, ask whether it would still be right if you rewrote the
steps. A method's origin survives a rewrite, so that's `method_citation`. A paper describing this
procedure doesn't, so that's `protocol_citation`. 

## If one paper didn't propose everything the protocol does, it's more than one protocol

Examples from previous reviews of protocol proposals:

- **Benchmarking of microbe set enrichment methods.** Looked like one protocol as described in the BugSigDB publication. 
However it involves three different atomic enrichment protocols, and a composite benchmarking protocol.
- **Centered Log Ratio transformation (CLR) followed by filtering on microbial prevalence.** Seen multiple times in preprocessing workflows, but these are distinct protocols because they have different citations and can be used independently of each other.

## When to stop splitting up steps into new protocols

Splitting the steps of a protocol into more protocols can unnecessarily increase complexity. 
Split a step of a protocol out into its own protocol only if **all three** are yes:

1. **Would different analyses use the step on their own?** If a step only ever shows up this protocol,
   splitting buys no reuse.
2. **Is there a plausible substitute for the step?**  Split where real analyses vary. CLR is a good atomic 
protocol because it has substitutes: arcsin-square-root, log-transformation, or no transformation. 
"Divide by the geometric mean"has no substitute inside CLR, so this does not motivate splitting this step
into its own protocol.
3. **Does it have a name people use?** A named method is citable; a step inside one isn't. Usually why the
   naming question and the citation question give the same answer.

Another way to think about it is: in a methods section, would you say "we did X", ie name the procedure? If so, it's a protocol. Or would you say, "we did X as part of Y". If so, X is likely a step in protocol Y.

Some cases are arguable. Geometric mean _does_ have other uses, but for microbiome researchers I (Levi)
can only think of its use inside CLR, and it wouldn't be much burden to redefine it elsewhere if needed, so I 
wouldn't split it out. Make the call and say why in `## Notes`.

## Alternatives inside one protocol, sometimes

A protocol can offer a choice when the alternatives come from the same source, take the same inputs,
produce the same kind of output, and mean the same thing. Then it's a tuning knob, not a different protocol.

[independent-filtering-variance](agent-protocols/protocols/independent-filtering-variance) is the precedent: `filter: variance | mean`, both from the same paper, one
`method_citation` covering both, and the protocol says exactly one must be chosen, that they must not be
applied in sequence, and that the choice is recorded before anyone looks at results.

`method_citation` is the tell. If the alternatives need two citations, they're two protocols.

This is how protocols work elsewhere too. Nature Protocols and protocols.io allow branch points freely, but
for procedural variants of one method, not for swapping in a different method by a different author. OECD
Test Guidelines draw the same line — several assay variants in one guideline only where they're validated as
concordant.

## Say what the protocol doesn't do

Provide an explicit out-of-scope statement.  This lets an agent
compose protocols without wondering whether two of them overlap, and it stops a protocol from quietly growing into a pipeline when the protocol is often followed by other downstream analysis.

For example, [independent-filtering-variance](agent-protocols/protocols/independent-filtering-variance) ends by saying it doesn't define or perform hypothesis testing, p-value
calculation, multiple-testing adjustment, differential expression, or interpretation of discoveries. 

## Say what has to be recorded

Name the parameters, thresholds, versions and intermediate counts an execution has to report: for example the values
of every parameter, how many features or samples survived each filtering step, which tie-breaker was used,
what happens to missing values.

Where a choice has to be made — a filter, a threshold, a candidate list — make it a step and require it to be recorded. No choice should be implied.

## When you can't find a source

Rare, but some methods are old or folkloric enough to have no identifiable first publication. Say so instead of reaching for a convenient recent paper.

A missing `method_citation` validates today, so nothing stops you leaving it out. Descriptive protocols that
claim no method — a study table, a corpus summary — legitimately have nothing to cite. For a protocol that
does run a named method, an empty field is a question to raise, not an answer.

## Before you open the pull request
 
 Check:
- For an atomic protocol claiming a method, that it is one method, and you can name the paper that proposed
  it. Composite: its steps name the constituent protocols and the order, without restating what those
  protocols already say. 
- It's a unit that can be reused in different analyses.
- `method_citation` is the method's origin. `protocol_citation`, if present, describes this procedure.
- If a sibling protocol shares your `method_citation`, `## Notes` says how yours differs.
- An expert in the field could have a complete picture of the protocol without reading any code.
- Every configurable parameter has a stated default. Required inputs (data, software, parameters) are stated in the Materials section.
- The out-of-scope section exists and is specific.

The protocol standard is still pre-release and experimental. If a rule here gets in the way of describing
a protocol unambiguously, raise it as an issue rather than working around it.
