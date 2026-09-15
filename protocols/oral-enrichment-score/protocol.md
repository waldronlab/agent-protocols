---
name: oral-enrichment-score
description: Calculate the oral enrichment score by summing the relative abundances of species prevalent in oral-cavity microbiomes within gut metagenomic profiles.
version: 1.0.0
authors:
  - name: Otto Infield-Harm
    orcid: 0009-0009-8629-6124
  - name: Levi Waldron
    orcid: 0000-0003-2725-0694
date: 2026-09-15
status: draft
license: CC-BY-4.0
type: atomic

protocol_citation: "10.1038/s41467-025-66888-1"
method_origin_citation: "10.1038/s41467-025-66888-1"

artifact_doi: ~
collection_doi: ~

upstream_repositories:
  - "https://github.com/waldronlab/curatedMetagenomicData"
database_urls:
  - "https://bioconductor.org/packages/curatedMetagenomicData"

protocols_used: []
key_packages: []
category: metagenomics
tags: [microbiome, metagenomics, oral-enrichment, dysbiosis, gut, oral]
---

# Oral Enrichment Score

This protocol defines a reproducible oral enrichment score (OES) for a gut microbiome
sample. The score is the summed relative abundance of species that are prevalent in
oral-cavity microbiomes, using a prevalence threshold of 1% as in Manghi et al.

## Materials

- **Input data:**
  - A species-level relative-abundance table for oral-cavity samples. Each sample must
    have a body-site label identifying it as an oral-cavity sample.
  - A species-level relative-abundance table for gut samples to be scored. The oral and
    gut tables must use the same taxonomic naming scheme and abundance scale.
  - Treat each sample as one observation and each species as one abundance field, or
    use an equivalent long-format representation with explicit sample and species
    identifiers. Do not interpret metadata fields as species abundances.
- **Sample metadata:**
  - A body-site field that distinguishes oral-cavity samples from gut samples.
  - Optional sample identifiers and study identifiers, which should be retained in the
    output.
- **Reference method:**
  - Manghi et al., Nature Communications, DOI
    [10.1038/s41467-025-66888-1](https://doi.org/10.1038/s41467-025-66888-1).
  - The cMD 3 resource ([Bioconductor package](https://bioconductor.org/packages/curatedMetagenomicData))
    may be used as a source of uniformly processed profiles and metadata.

## Steps

### Step 1: Confirm the input abundance scale and taxonomic identifiers

Use species-level relative abundances for both the oral reference samples and the gut
samples to be scored. Confirm that each sample's species abundances use the same
normalization and that species identifiers match between tables. Express abundances as
proportions from 0 through 1, not as percentages from 0 through 100. If the input is
in percentages, divide every abundance by 100 before continuing.

Do not calculate the OES from CLR-transformed, log-transformed, or raw read-count
values. The score is defined from relative abundances before compositional
transformation.

Require every abundance used in the calculation to be numeric, finite, and
non-negative. Treat an explicit numeric zero as zero. Do not treat a missing value,
blank value, or non-numeric value as zero; resolve it or stop the calculation.

If the two tables contain different species naming conventions, harmonize the
identifiers before continuing. Do not silently treat an unresolved identifier as a
different species; stop and report any unresolved identifiers that could affect the
oral signature or its gut-sample abundances.

### Step 2: Select oral-cavity reference samples

From the reference abundance table, retain only samples whose metadata body-site
value is exactly `oralcavity`, or whose mapping to that value has been explicitly
documented in advance. Do not infer oral-cavity status from a study name, specimen
name, or free-text description alone.

Exclude samples with missing body-site labels from construction of the oral reference
set. Keep a record of the number of oral-cavity samples retained and excluded. If no
oral-cavity samples remain, stop; an oral-associated species signature cannot be
constructed.

The original study used 857 oral-cavity samples from curatedMetagenomicData 3. A
different reference collection may be used, but its source, body-site definition,
sample count, and study composition must be recorded because they determine the
resulting species signature.

### Step 3: Construct the oral-associated species signature

For every species in the harmonized oral-cavity table:

1. Count the number of retained oral-cavity samples in which the species has a
   finite relative abundance greater than zero.
2. Divide that count by the total number of retained oral-cavity samples.
3. Retain the species if its prevalence is at least 1% of the oral-cavity samples.

The retained species form the **oral-associated species signature**. With the cMD 3
reference data and the paper's processing choices, this threshold produced a
signature of 305 species.

Use “at least 1%” as an inclusive threshold. Do not round prevalence before applying
the threshold. If the oral reference set contains N samples, retain a species when
its number of non-zero samples is at least the ceiling of 0.01 × N (the smallest
whole number greater than or equal to 0.01 × N).

Save the signature as a versioned reference artifact containing, at minimum:

- species identifier,
- number of oral-cavity samples with non-zero abundance,
- total oral-cavity sample count,
- calculated prevalence,
- prevalence threshold,
- reference dataset identifier and date.

### Step 4: Align the gut abundance table to the signature

Before scoring, confirm that the gut table is a complete species-level profile for
each sample: a species absent from a sample is represented by an omitted species
entry or an explicit zero, rather than by an unknown or missing measurement. Then,
for each gut sample to be scored:

1. Match species identifiers to the oral-associated species signature.
2. Assign abundance zero to a signature species when it is absent from a complete
   species profile or is explicitly present with zero relative abundance.
3. Do not assign zero to a signature species when its value is missing, invalid, or
   cannot be matched because of an unresolved identifier. Resolve the problem and
   repeat the alignment before scoring that sample.
4. Retain all sample identifiers and relevant metadata alongside the aligned
   abundance values.

The score cannot be interpreted as oral enrichment if taxonomic profiles were
generated with incompatible reference databases or taxonomic resolutions.

### Step 5: Calculate the oral enrichment score

For each gut sample, sum the relative abundances of all species in the
oral-associated species signature:

```text
For each gut sample:
    OES = sum(relative abundance for every species in the oral-associated signature)
```

The output must contain one OES value per gut sample. The score is non-negative and
is on the same relative-abundance scale as the input table. A score of zero means
that no retained oral-associated species contributed a measured non-zero abundance
under the input profiling and matching rules; it does not prove that no oral-origin
microbes are present.

### Step 6: Record score provenance and quality checks

For every score calculation, record:

- the oral reference dataset and version,
- the number of oral reference samples,
- the prevalence threshold,
- the number of species in the signature,
- the abundance profiler and database version,
- the number of gut samples scored,
- the number and proportion of gut samples with OES equal to zero,
- the number of unmatched or unresolved species identifiers.

Confirm that all OES values are finite and non-negative. Investigate any score
greater than the sum of all non-negative species abundances in its corresponding gut
profile or any unexpected change in the fraction of zero scores before using the
results downstream. If abundance profiles are normalized to sum to one, an OES
greater than one is necessarily an error.

## Notes

- The paper selected the 1% prevalence threshold after comparing thresholds from 1%
  through 50%. The 1% signature had the highest reported discrimination of control
  versus non-control populations and gave more than 80% of individuals a non-zero
  score in their evaluation.
- OES is an association measure based on taxonomic relative abundance. In
  cross-sectional stool data it does not directly demonstrate oral-to-gut
  transmission, strain transfer, viability, or causality.
- The score is sensitive to the oral reference population, taxonomic profiler,
  reference database, prevalence definition, and treatment of missing species.
  These choices must remain fixed when comparing samples or cohorts.
- For statistical analysis, the paper used the score directly for rank-based
  comparisons and used `log(OES + 0.0001)` for covariate-adjusted regression. The
  pseudocount is an analysis choice, not part of the OES definition.
- An alternative score based on Shannon entropy over the oral-associated species was
  evaluated in the paper, but it is not the primary OES defined by this protocol.
- This protocol calculates the score only. Disease association testing, covariate
  adjustment, meta-analysis, and machine-learning evaluation should be documented
  as separate protocols.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-09-15)

#### Changes
- Initial protocol creation based on the oral enrichment score procedure in Manghi et al.
- Clarified table orientation, abundance units, invalid-value handling, body-site
  selection, missing-species treatment, and score validation.

#### Reviews
*No reviews yet.*
