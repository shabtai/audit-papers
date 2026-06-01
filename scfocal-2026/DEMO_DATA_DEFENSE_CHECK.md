# scFOCAL audit — demo-data defense check

Application of `METHODOLOGY_demo_data_defense.md` to the 13 real-bugs findings from this audit. The methodology
asks: can any of these findings be excused by the authors saying "what we deposited is just illustrative / sample
/ placeholder data; the real analysis used different data and produced the published numbers correctly"?

## Pass criteria (per the methodology)

A finding survives the demo-data defense if at least one of these holds:

- **A.** The bug is in the deposited *source code* (which the project publishes as the actual pipeline).
- **B.** The published paper numbers match what the deposited code, applied to the deposited data, produces.
- **C.** The deposited values reproduce paper-headlined numbers to enough precision that "this is demo data" requires implausible coincidences.

## Per-finding validation

| # | Strongest demo-data defense | Why it fails | Defense criteria failed |
|---|---|---|---|
| **F1** Pseudoreplication | "Demo limma output; real analysis used `lme4` mixed-effects" | Bug is in committed `discordanceLimma.R:55-66` (no block arg in `model.matrix(~0 + patient_group)`). Deposit shows the exact pseudoreplication signature (AveExpr=9.806±0.014 across 1,674 drugs, 205 P=0 underflow, 81.5% adj.p<.05). Mixed-effects on real data would not produce these signatures. F24's `library(lme4)` never-called confirms the fix wasn't applied. | A + C |
| **F3** Mean-cutoff dichotomization | "Demo code uses mean; real code uses external biology threshold" | Dichotomization in committed `GBM_Combinations_ISOSCELES_2020L1000.R:117-136`. Author's own line-125 comment (`# should the cut-off be set dynamically?`) shows this IS the real code's choice. | A |
| **F11** Only-GBM41 loop | "Committed `patient_ids <- 'GBM41'` is demo; real code iterated 13" | Committed source code is wrong. The repo README positions `scFOCAL-dataProcessing` as "Processing pipelines used in Suter et al." The deposited s3b output covers 13 patients × 7 k-means, which the committed code cannot produce — either the public code is wrong OR the public code is misrepresented. | A + B |
| **F12** `200nM.R` bugs | "Demo script; real DESeq2 used proper design" | Bugs in committed source (positional replicate hardcode, padj/pval typo, circular LFC cutoff). Published 841-gene CT-179 signature size is consistent with the raw-p path, not the adj-p path. | A |
| **F22** 75th-percentile cutoff | "Demo QC; real QC used proper cutoff" | Code-level bug in committed `Panobinostat_analysis/01_*.R:273-275`. Hidden "real" code would contradict the repo's purpose statement. | A |
| **F2** Combination_Index misnomer | "Demo column; real CI computed differently" | `combinationScore = logFC * Resistant_Cell_Connectivity` is in committed source. Cross-validation `Combination_Index ≡ |logFC × mRCC|` holds with Pearson r=1.000000 and max abs err 4.99×10⁻⁸ across all 411 deposited rows — to 7-8 significant figures. Demo data of that exact precision matching the formula is implausible. | A + C |
| **F9a** Degenerate t-test (N=1, SD=0) | "Fig 5D demo numbers; real Vehicle had N=10 throughout" | Deposited Vehicle timeline cross-validates against deposited Fig 5E survival data, which reproduces paper medians (Vehicle=19d, CT-179=24d, ABT-414=38d, Combo=63d) to the day. Demo data wouldn't reproduce 4 medians exactly. The N=1 SD=0 rows ARE what produced the published "<0.000001". | C |
| **F24** `library(lme4)` fossil | "Demo imports; real code doesn't load lme4" | Observation about committed source. Hidden "real" code with different imports contradicts the repo. | A |
| **F7-B** FDR=3.2×10⁻²³⁸ matches one-sample arithmetic | "Deposit is demo; real two-sample test ran on raw Seurat object" | Published FDR=3.2×10⁻²³⁸ is 148 orders of magnitude off proper two-sample (~10⁻⁹⁰), within 1 OoM of one-sample arithmetic on N=10,394. The published FDR magnitude IS the bug. Defense requires the published number to coincidentally match one-sample arithmetic by 148 OoM — not credible. | B + C |
| **F9** BLI imaging dropout | "Demo Vehicle N timeline; real had N=10 throughout" | Dropout pattern cross-validated against Fig 5E survival data (also deposited, also matches paper medians exactly). Both deposited tables would have to be demo AND mutually consistent in their specific dropout-vs-death timing pattern AND both happen to reproduce paper medians to the day. Implausible. | C |
| **F6** ρ=0.7 aggregate | "Deposit is demo per-line ρs; real per-line was consistent" | Aggregate ρ across the 24 deposited per-line ρ values matches paper headline ρ≈0.7 under standard correlation-of-aggregates arithmetic. Demo data wouldn't aggregate to the paper's specific headline. | C |
| **F17** CT-179 monotherapy NS | "Demo survival data; real CT-179 was significant" | Per-mouse death times in deposit reproduce all 4 arms' median survival to the day (19/24/38/63). Log-rank test on the deposited data gives p=0.611 for CT-179 vs Vehicle. Demo data wouldn't match. | C |
| **F4** varlitinib rank 31 | "Demo CI values; real ranking has varlitinib in top-3" | Rank 31 is deterministic from deposited values, which (per F2) are exactly `|logFC × mRCC|`. If Fig 5 is built from different "real" CI values, the paper's source data for Fig 5 is hidden — itself a Nat Comms data-availability violation. Either way the bug stands. | B + C |

## Result

**13 of 13 findings survive the demo-data defense.**

| Defense outcome | Findings | Count |
|---|---|---|
| Bug in committed source code (A) — defense doesn't apply | F1, F3, F11, F12, F22, F24, F2 | 7 |
| Deposit reproduces paper headlines to the day / to many digits (C) | F9, F9a, F6, F17 | 4 |
| Arithmetic discriminator works regardless of demo/real status (B) | F7-B | 1 |
| Cross-validation between code and data (B) | F4 | 1 |

## Collective requirements of the demo-data defense

For all 13 findings to be excused, the authors would need to simultaneously claim:

1. The 4 deposited median survival values (Vehicle=19, CT-179=24, ABT-414=38, Combo=63), the BLI N timeline, the per-line ρ distribution, the limma significance signatures, the cross-table mouse-imaging-vs-death pattern, and the FDR magnitude **all happen to reproduce the paper's published numbers to many significant figures by coincidence**.
2. The committed GitHub code is "demo" but the repo README positions it as the production pipeline (`scFOCAL-dataProcessing`: "Processing pipelines used in Suter et al.").
3. The "real" code (with mixed-effects per-patient random effects, biology-derived cutoffs, proper two-sample Wilcoxon tests, full-patient loops over all 13 patients, padj-consistent filter chains) exists somewhere but was never published.

That is not a credible defense — it is a chain of admissions that none of the deposited code or data produced the published results, which constitutes a complete reproducibility failure on its own and a violation of Nat Comms data availability policy.

## Bottom line

The audit's 13-finding "real bugs" tier is robust to the demo-data defense. The audit headline stands:

> The paper's ISOSCELES discordance pipeline overstates its own significance evidence by 100–10²³⁸×, its central in-vivo validation cannot be independently reproduced because the DMSO arm cells are not deposited, and its lead in-vivo BLI panel was computed on selectively-imaged mice with degenerate single-mouse t-tests — but the qualitative discoveries (alisertib targets NPC-like cells, CT-179 + Depatux-M combo extends survival) and the survival data itself survive after the statistical inflation is corrected.
