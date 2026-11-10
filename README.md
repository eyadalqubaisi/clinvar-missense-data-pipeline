# ClinVar Missense Variant Data Pipeline

A reproducible Python pipeline that turns the raw ClinVar archive — 9.3 million rows — into a clean, labelled dataset of 203,229 human missense variants ready for machine learning.

Built by **Eyad Alqubaisi**, B.Sc. Computer Science, King Khalid University.

## Why this project exists

Published variant classifiers routinely report impressive benchmark numbers, but those numbers depend entirely on how the training data was assembled. A single overlooked column can duplicate half the dataset and leak test data into training. This pipeline documents every filtering decision and the row count it costs, so the resulting dataset can be audited rather than trusted blindly.

## Pipeline stages

| # | Stage | Rows remaining | Share |
|---|---|---|---|
| 0 | Raw ClinVar variant_summary | 9,354,555 | 100% |
| 1 | Keep GRCh38 assembly only | 4,643,623 | 50% |
| 2 | Keep single nucleotide variants | 4,292,033 | 46% |
| 3 | Parse protein change from HGVS name | 3,468,710 | 37% |
| 4 | Apply the missense definition | 2,575,252 | 28% |
| 5 | Map to a binary clinical label | 219,382 | 2.3% |
| 6 | Review-status filter and deduplication | **203,229** | 2.2% |

## The three decisions that matter

**1. Genome assembly.** ClinVar lists every variant once per reference assembly. Keeping both GRCh37 and GRCh38 duplicates the entire dataset — and, worse, places the same variant in both the training and test splits. Pinning to GRCh38 removes 4.7 million duplicate rows.

**2. The missense definition.** A missense variant must change one amino acid into a *different* one. Three conditions enforce this: a protein change was parsed, the new residue is not a stop codon (Ter), and the new residue differs from the original. The third condition is the one most often forgotten — it excludes synonymous variants, which alter DNA but leave the protein unchanged.

**3. Label quality over label quantity.** 84% of missense variants in ClinVar carry an uncertain or conflicting interpretation. Training on them teaches the model noise, so they are dropped. Submissions with no assertion criteria are dropped as well.

## Final dataset

| Property | Value |
|---|---|
| Variants | 203,229 |
| Unique genes | 15,733 |
| Benign (label 0) | 141,110 |
| Pathogenic (label 1) | 62,119 |
| Pathogenic share | 30.6% |
| Duplicate variant IDs | 0 |

Output columns: variant_id, GeneSymbol, aa_ref, aa_pos, aa_alt, substitution, ClinicalSignificance, ReviewStatus, label.

## What the data shows

- **Class imbalance.** Benign outnumbers pathogenic roughly two to one, so accuracy is a misleading metric here — PR-AUC is the appropriate choice.
- **Gene concentration.** FBN1, TTN, KMT2D, BRCA1 and BRCA2 dominate the counts. This reflects clinical testing volume, not biology, and it means a random train/test split would let the model memorise genes instead of learning. A gene-level group split is required.
- **Substitution patterns.** Arg to Gln, Ala to Thr and Arg to His are the most frequent substitutions, reflecting the structure of the genetic code itself.

## Running it

The notebook runs end to end on Google Colab with no setup and no GPU. It downloads the ClinVar archive directly from NCBI, so no data files need to be stored in this repository.

Stack: Python, pandas, NumPy, matplotlib, regular expressions.

## Repository contents

| File | Description |
|---|---|
| clinvar_missense_pipeline.ipynb | The full pipeline, stage by stage |
| clinvar_missense_clean.csv | The final clean dataset |
| fig_class_balance.png | Benign vs pathogenic distribution |
| fig_top_genes.png | Top 15 genes by variant count |
| fig_top_substitutions.png | Top 15 amino acid substitutions |

## Background

This pipeline is an independent reimplementation of the data-construction and cleaning stage of my six-member B.Sc. graduation project on missense variant pathogenicity classification, where dataset assembly and cleaning were my responsibility. All code here is written from scratch and scoped to run on free compute.

## Next steps

- Gene-family-aware splitting to prevent paralog leakage
- Feature engineering from gnomAD allele frequencies and dbNSFP conservation scores
- Baseline XGBoost model with probability calibration

## Data source

ClinVar, National Center for Biotechnology Information (NCBI), downloaded November 2026. ClinVar is updated weekly, so row counts will shift slightly on re-runs.
