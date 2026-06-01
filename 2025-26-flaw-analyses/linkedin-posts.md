# LinkedIn posts

Two posts, one per paper. Each: hook → most-impressive example with technical detail → plain-language sidebar for non-statisticians → one-line summary for the rest of the audit.

---

## Post 1 — Aung et al., *Nature Genetics* 2025 (NSCLC spatial signatures)

**Paper:** *"Spatial signatures for predicting immunotherapy outcomes using multi-omics in non-small cell lung cancer"* — Aung et al., *Nat Genet* 2025. https://doi.org/10.1038/s41588-025-02351-7 (PMID 41073787)

A one-character bug in this published Nature Genetics paper vacates two of its headline statistical claims.

The script: `lasso_cox_cellFract_tumor_resistanceLASSO_model_PFS.R`

The line:

```r
y_df = Surv(as.numeric(cellFrac$PFS_5Years_months, cellFrac$PFS_5Years_Index))
```

This looks like it builds `Surv(time, event)`. It doesn't. Two R semantic quirks compose:

1. `as.numeric()` silently drops extra positional arguments. `as.numeric(time, event)` evaluates to `as.numeric(time)`. The event-status column is gone before `Surv()` ever sees it.

2. `Surv(time)` with one argument defaults to "all events observed". Every patient gets `status = 1` — no censoring, no warning, no error.

The Cox model treats every patient as having had a progression event, regardless of who was actually censored.

When the bug is corrected (one closing parenthesis):

- **Resistance signature, Yale cohort**: HR 3.8 (p=0.004) → 2.58 (p=0.053). **Significance lost.**
- **Response signature, Yale cohort**: HR 0.4 (p=0.019) → 0.30 (p=0.057). **Significance lost.**

The pattern appears 11 times in the cell-fraction scripts; the same author uses the correct `Surv(t, e)` form in 6 other scripts in the same repo. It's a copy-paste artifact, not a systemic understanding gap — but it's exactly where the abstract's cell-type signatures are derived and validated.

---

**Plain language for non-statisticians:**

Clinical survival studies track how long patients survive without their cancer progressing. Some patients have a known progression event during the study. Others leave the study early or remain progression-free at study end — those patients are called "censored." A correct analysis tells the model: *for censored patients, we know they survived AT LEAST this long; we don't know whether they would have progressed later.*

The bug tells the model the opposite: *every patient progressed at exactly the time their record ended.* This treats people who were just being followed up as if they all had bad outcomes.

When you do this, the difference between "patients flagged as resistant by the biomarker" and "patients flagged as sensitive" looks bigger and statistically cleaner than it really is. Fixing the bug shrinks both gaps to within the noise — meaning the data doesn't actually let you tell the two groups apart with confidence.

In a clinical context: this is the difference between "we have evidence the biomarker predicts immunotherapy response" and "we don't have enough evidence yet." The paper claims the former. The corrected analysis only supports the latter.

---

**One-line summary for this audit:** *One copy-paste typo in `Surv()` discards censoring across 11 call sites and silently removes statistical significance from two of the paper's named Yale-cohort hazard ratios.*

Full report (with reproduction code): https://github.com/shabtai/2025-26-flaw-analyses/blob/main/aung2025/flaws.html

Audit conducted with natural-joints (https://www.natural-joints.com/) — independent code & data audits of published research papers using semantic data-layer enrichment plus multi-agent code/honesty/facts review.

#Bioinformatics #Reproducibility #DataScience #StatisticalRigor #ScientificIntegrity

---
---

## Post 2 — Thompson et al., *Nature Genetics* 2025 (CIN chemo-resistance signatures)

**Paper:** *"Predicting resistance to chemotherapy using chromosomal instability signatures"* — Thompson et al., *Nat Genet* 2025. https://doi.org/10.1038/s41588-025-02233-y (PMID 40551015)

A published Nature Genetics paper tests an "anthracycline-resistance biomarker" in a cohort where 87% of the patients also received platinum chemotherapy — and the biomarker's training labels were derived from platinum-response priors. The platinum confounder is not adjusted for. So the validation result is consistent with the biomarker being a repackaged platinum-response classifier.

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

The 4 patients who received doxorubicin without platinum are too few to support a clean restricted analysis. The platinum-adjusted analysis is not done. The headline finding sits on top of an uncorrected confounder that's stronger than any covariate in the Cox formula.

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

The honest read of Fig 2c is: *the biomarker predicts outcome in a cohort where most patients received platinum cotherapy that we didn't adjust for, using a biomarker whose labels were assigned from platinum response. Whether the biomarker tracks anthracycline response specifically — separately from platinum — is not established by this experiment.*

---

**One-line summary for this audit:** *Of 25 audit findings (all novel, none previously reported), the most concerning is structural: the anthracycline biomarker is trained on platinum-response-derived labels and validated in a 87%-platinum-co-treated cohort with cotherapy unadjusted; alongside this — 5 of 9 main headline hazard ratios lose statistical significance under proper multiple-testing correction, the OV04 platinum pilot HR collapses to non-significance when a 7-patient hand-coded covariate is removed, treating RECIST-confirmed cancer progression as "censored" instead of as an event shifts two of four phase-3 HRs by more than 30%, and patients with all-missing biomarker values silently receive deterministic prediction labels based on which file they were stored in.*

Full report (with reproduction R code per finding): https://shabtai.github.io/audit-papers/2025-26-flaw-analyses/thompson2025/flaws.html

Audit conducted with natural-joints (https://www.natural-joints.com/) — semantic data-layer enrichment plus multi-agent code/honesty/facts review. The plat_cotherapy finding above was specifically surfaced by enrichment: the column's True/False values say nothing on their own; the resolved meaning *"received platinum-based therapy as a concurrent treatment alongside doxorubicin"* is what made the audit agent look for it in the Cox model.

#Bioinformatics #Reproducibility #ClinicalResearch #StatisticalRigor #ScientificIntegrity

---

Repo index: https://shabtai.github.io/audit-papers/2025-26-flaw-analyses/
Top-4 deep dive (with the methodology used to filter "real bugs" from reproducibility issues that could be excused by demo data): https://shabtai.github.io/audit-papers/2025-26-flaw-analyses/top-bugs.html
