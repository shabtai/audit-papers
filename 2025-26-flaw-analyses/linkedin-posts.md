# LinkedIn posts

Two posts, one per paper. Each: hook → most-impressive example with technical detail → plain-language sidebar for non-statisticians → one-line summary for the rest of the audit.

---

## Post 1 — Aung et al., *Nature Genetics* 2025 (NSCLC spatial signatures)

**Paper:** *"Spatial signatures for predicting immunotherapy outcomes using multi-omics in non-small cell lung cancer"* — Aung et al., *Nat Genet* 2025. https://doi.org/10.1038/s41588-025-02351-7 (PMID 41073787)

A published Nature Genetics paper announces 8 hazard ratios across 3 cohorts as evidence that spatial signatures predict NSCLC immunotherapy response. The paper's own released code does not produce the headline numbers.

The most direct contradiction: `validate_finalSigs_pStroma_RNA.R` line 157, the script that validates the response signature on the UQ cohort, contains this literal label inside its plotting code:

```r
label = "HR = 1.1 (0.43-3) \n p = 0.6 (Log-Rank-1-sided) \n cutpoint = tertile"
```

HR = 1.1. p = 0.6. The 95% CI spans 0.43 to 3.0, well across the null.

The paper's abstract reports HR = 0.38 for the same comparison.

Both numbers come from the same script run by the same author on the same deposited data. The script self-documents one result. The abstract claims another. There is no path in the released artifact from HR=0.38 to HR=1.1 except by running additional analysis that wasn't committed.

This is one of 12 MAJOR-impact findings produced by an independent natural-joints audit. Other findings in the same paper:

- **5 of 9 abstract HRs collapse to non-significance** under bug correction, two-sided p-value testing, or end-to-end reproduction.
- **A one-character bug** at 11 R script sites (`Surv(as.numeric(time, event))`) silently drops the censoring vector — making the Cox model treat every patient as a progression event. Correcting this single bug moves Yale tumor HR 3.8 → 2.2 and Yale stroma HR 0.4 → 0.30; both lose significance.
- **Circular validation**: the cell-to-gene signature's "good models" are filtered by their held-out test-cohort HR before being assembled into the final signature. `good_models = which((hzrs$x)>=1.5)`. The validation result is the model-selection criterion.
- **The Greek validation cohort (n=79) has zero code in the repository.** The third "independent" cohort exists only in manuscript text.
- **`lower.limits = 0` in LASSO forces the resistance signature to be non-negative; `upper.limits = 0` forces the response signature to be non-positive.** The direction of the signature is set by a keyword argument, not by the data.
- **One-sided p-values for "validation", two-sided for "discovery"** — direction picked post-hoc by the sign of the observed HR. Both abstract validation p-values (0.05, 0.036) become 0.10 and 0.072 when correctly two-sided.
- **Reading the released code shows it tries to load coefficient files that were never committed.** The headline cell-to-gene HR=5.1 is computed off `final_coeffs_..._0p1.csv` and `hzrs_..._CW.csv`, neither of which exists anywhere in the repository.

---

**Plain language for non-statisticians:**

Clinical biomarker studies follow this shape: *(1) you find some pattern in a discovery dataset; (2) you check that the same pattern works in a separate validation dataset; (3) if it works in both, the biomarker is real.* The paper presents 3 cohorts (Yale, UQ, Greek) supposedly walking through this shape.

What the audit found:

- The "discovery" pattern only reaches statistical significance because of a coding bug that treats every patient as if they had a bad outcome. Fix the bug and the pattern is within the noise.
- The "validation" code, when actually run, gives a number that says the signature does not work in the validation cohort. The abstract quotes a different number that nothing in the released code produces.
- The third cohort isn't in the released code at all.
- The genes that make up the final signature are picked from analysis runs that already showed the desired result, then "validated" against the same dataset that picked them.

A faithful description would be: *we found a tentative pattern in 32 patients at Yale; it doesn't transfer to UQ when you run our actual code; we cannot show our work for Greece.* The paper's description, in contrast, is "three-cohort validation of two complementary spatial signatures."

---

**One-line summary for this audit:** *Of 8 hazard ratios named in the abstract, at least 5 either collapse on bug correction, are unreproducible from the public code+data, or are products of post-hoc statistical choices. 37 total findings, all novel — zero prior public reports.*

Full report (with reproduction code per finding): https://shabtai.github.io/audit-papers/2025-26-flaw-analyses/aung2025/flaws.html

Audit conducted with natural-joints (https://www.natural-joints.com/) — independent code & data audits of published research.

#Bioinformatics #Reproducibility #DataScience #StatisticalRigor #ScientificIntegrity

---
---

## Post 2 — Thompson et al., *Nature Genetics* 2025 (CIN chemo-resistance signatures)

**Paper:** *"Predicting resistance to chemotherapy using chromosomal instability signatures"* — Thompson et al., *Nat Genet* 2025. https://doi.org/10.1038/s41588-025-02233-y (PMID 40551015)

A published Nature Genetics paper tests an "anthracycline-resistance biomarker" in a cohort where 87% of the patients also received platinum chemotherapy, and the biomarker's training labels were derived from platinum-response priors. The platinum cotherapy is not adjusted for in the survival model. **The analysis does not rule out a platinum-response confounder** — it does not separate "this biomarker predicts anthracycline response" from "this biomarker predicts platinum response, observed in a platinum-co-treated cohort." Both interpretations are compatible with the published result.

The data (OV04 doxorubicin pilot study, Fig 2c, n=30):

```
plat_cotherapy distribution in OV_tissue_doxorubicin_predictions.csv:
True     26   (86.7% — patients who received concurrent platinum)
False     4   (13.3% — patients who got doxorubicin without platinum)
```

The Cox model (`OV04_Survival_Analysis.R:198`):

```r
cox <- coxph(Surv(PFS, censoring) ~ prediction*primary_PFS + tumour_stage +
                                    age_recoded + maintenance + wgii,
             data=doxo_data)
```

Covariates: tumour_stage, age_recoded, maintenance, wgii. The column `plat_cotherapy` is in the CSV but never appears anywhere in the script — not as a covariate, not in any sensitivity analysis, not in any robustness check.

The paper's Methods explicitly describes how the training labels for the anthracycline biomarker were assigned (verbatim):

> *"platinum-resistant patients are expected to have an 18% response rate to doxorubicin monotherapy ... whereas sensitive patients have a 28% response rate."*

So the biomarker was trained against labels derived from platinum response in patient-derived models, then validated in a cohort where 87% of patients also received platinum cotherapy, with that cotherapy not adjusted for in the survival model.

The 4 patients who received doxorubicin without platinum are too few for a clean restricted analysis. A platinum-adjusted Cox is not reported. Whether the biomarker tracks anthracycline response specifically — separately from platinum response — is therefore not established by this experiment. It might. The experimental design just doesn't tell us either way.

**This isn't a typo defense.** The plat_cotherapy values are in the deposit; the Cox model is in the committed script; the literature priors for label assignment are in the paper Methods. All three are intact and all three line up the same way: an anthracycline biomarker trained on platinum-response labels and validated in a platinum-co-treated cohort.

This was surfaced because the semantic enricher resolved `plat_cotherapy` as *"Boolean flag indicating whether the patient received platinum-based therapy as a concurrent treatment (cotherapy) alongside doxorubicin"* — a meaning that's not derivable from the True/False values alone. An auditor looking at the Cox formula without that column-meaning has no reason to ask "what about platinum cotherapy?" — the enrichment is what made the question askable.

---

**Plain language for non-statisticians:**

The paper proposes a genetic test that — they claim — predicts whether ovarian-cancer patients will respond to anthracycline (doxorubicin) chemotherapy. The validation experiment uses 30 patients from the OV04 trial.

Two facts about those 30 patients are not foregrounded in the paper:

1. **Training labels came from a different drug.** The genetic test was developed using lab-grown tissue from patients whose response to *platinum* (a different drug) had been observed. Patients whose platinum response was poor were labeled "resistant" in training; patients with better platinum response were labeled "sensitive." Doxorubicin response itself was not directly measured for training.

2. **Validation patients mostly received platinum too.** Of the 30 validation patients, 26 received platinum chemotherapy alongside doxorubicin. Only 4 received doxorubicin without platinum.

So the test was trained on labels that reflect platinum behavior, and validated in patients who almost all got platinum. The validation experiment's Cox model adjusts for things like tumor stage, age, and maintenance therapy — but not for the platinum cotherapy.

In a clinical context: imagine testing a "blood-pressure medication response predictor" in a group where 87% of patients also took a diuretic — and not accounting for the diuretic in the analysis. You might conclude the predictor works on blood-pressure medication when actually it's reading the diuretic. Same shape of error here.

A clean validation would be: take only the 4 patients who got doxorubicin without platinum and check if the biomarker still separates them. But 4 patients is far too small. The data necessary to disentangle the two effects isn't in this cohort.

The honest read of Fig 2c is: *the biomarker predicts outcome in a cohort where most patients received platinum cotherapy that we didn't adjust for, using a biomarker whose labels were assigned from platinum response. **The analysis does not rule out a platinum-response confounder** — it does not separate anthracycline-specific from platinum-cotherapy effects. The biomarker may genuinely predict anthracycline response, or it may primarily reflect platinum response. The data in Fig 2c does not let us tell.*

---

**One-line summary for this audit:** *Of 25 audit findings (all novel, none previously reported), the most consequential is methodological: the anthracycline biomarker's training labels were derived from platinum response and its validation cohort is 87% platinum-co-treated with cotherapy unadjusted — the analysis does not rule out a platinum-response confounder. Alongside this — 5 of 9 main headline hazard ratios lose statistical significance under proper multiple-testing correction, the OV04 platinum pilot HR collapses to non-significance when a 7-patient hand-coded covariate is removed, treating RECIST-confirmed cancer progression as "censored" instead of as an event shifts two of four phase-3 HRs by more than 30%, and patients with all-missing biomarker values silently receive deterministic prediction labels based on which file they were stored in.*

Full report (with reproduction R code per finding): https://shabtai.github.io/audit-papers/2025-26-flaw-analyses/thompson2025/flaws.html

Audit conducted with natural-joints (https://www.natural-joints.com/) — semantic data-layer enrichment plus multi-agent code/honesty/facts review. The plat_cotherapy finding above was specifically surfaced by enrichment: the column's True/False values say nothing on their own; the resolved meaning *"received platinum-based therapy as a concurrent treatment alongside doxorubicin"* is what made the audit agent look for it in the Cox model.

#Bioinformatics #Reproducibility #ClinicalResearch #StatisticalRigor #ScientificIntegrity

---

Repo index: https://shabtai.github.io/audit-papers/2025-26-flaw-analyses/
Top-4 deep dive (with the methodology used to filter "real bugs" from reproducibility issues that could be excused by demo data): https://shabtai.github.io/audit-papers/2025-26-flaw-analyses/top-bugs.html
