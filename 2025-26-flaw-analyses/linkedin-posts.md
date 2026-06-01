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

A published Nature Genetics figure shows a hazard ratio that the paper's own committed R code cannot produce.

The figure: Fig 2c, OV04 anthracycline pilot study. Published HR = **20.020** (95% CI 1.059–378.6, p=0.010).

The script (`OV04_Survival_Analysis.R`, lines 198–209):

```r
# Line 198: fit the Cox model the comment will reference
cox <- coxph(Surv(PFS, censoring) ~ prediction*primary_PFS + tumour_stage +
             age_recoded + maintenance + wgii, data=doxo_data)

# Lines 201-204: comments documenting how Fig 2c was generated
# We used intEST function of the interactionRCS package to obtain the HR of the prediction
# over the different primary_PFS at 6 months
# HR at 6 months = 20.020 (1.059-378.635)
# We plot this HR for the main figure

# Lines 207-209: the actual figure-prep code
dox_cox_df$`exp(coef)`[dox_cox_df$variate=="prediction"] = 20.020
dox_cox_df$`lower .95`[dox_cox_df$variate=="prediction"] = 1.059
dox_cox_df$`upper .95`[dox_cox_df$variate=="prediction"] = 378.635
```

There is no `intEST()` call anywhere in the repository. The `interactionRCS` package is never loaded. The three numbers are literal numeric assignments — hand-typed into the script.

Running the committed `coxph()` formula on the deposited CSV gives HR = **344** (95% CI 4.57–25,922). The 17× discrepancy isn't a "demo data vs production data" issue — the literals run on any input and produce the same triple regardless of cohort.

The same hardcoded-triple pattern recurs at `Pancancer_survival_analysis.R:988-990` for a different OV04 cohort. The script's documented method is not the script's actual computation.

---

**Plain language for non-statisticians:**

In drug-resistance research, the headline statistic is the *hazard ratio* — roughly, how much faster patients in one group reach a bad outcome compared to another. HR = 1.0 means no difference. HR = 2.0 means twice the risk. HR = 20 would mean the resistant group hits the bad outcome about twenty times faster — an extraordinary claim that should change clinical practice.

Standard scientific practice is: the figure shows the number, and the code shows how the number was calculated from the data. If you can't compute the number from the code, you can't really check it.

Here, the script *claims* it used a specific statistical function (`intEST`) to derive HR=20.020. But that function is never called. Instead, the script hand-types the number 20.020 directly into the figure dataframe — bypassing any actual computation. Anyone running the published code gets a different number; the figure shows the hand-typed value.

The deeper issue: the published number could have come from somewhere. Maybe a quick calculation on a different computer. Maybe an earlier version of the analysis. We can't tell — because the work that allegedly produced 20.020 isn't anywhere in the public record.

A more cautious figure would have shown the Cox model's actual output. That output has a 95% confidence interval spanning 4.6 to about 25,000 — meaning the data is consistent with a tiny effect, an enormous effect, and almost anything in between. With 30 patients and 28 progression events, the data simply doesn't pin down the effect tightly. The published "HR=20" looks dramatic; the honest version would be "we can't tell."

---

**One-line summary for this audit:** *Of 25 audit findings (all novel, none previously reported), 5 are major: a published HR is hardcoded into the script and never computed from the model; 5 of 9 main headlines lose statistical significance under proper multiple-testing correction; the OV04 platinum pilot HR collapses to non-significance when a 7-element hand-coded patient list is removed from the model; treating RECIST-confirmed cancer progression as "censored" instead of as an event shifts two of four phase-3 hazard ratios by more than 30%; patients with all-missing biomarker values silently receive deterministic prediction labels based on which file they were stored in.*

Full report (with reproduction R code per finding): https://github.com/shabtai/2025-26-flaw-analyses/blob/main/thompson2025/flaws.html

#Bioinformatics #Reproducibility #ClinicalResearch #StatisticalRigor #ScientificIntegrity

---

Repo index: https://github.com/shabtai/2025-26-flaw-analyses
Top-4 deep dive (with the methodology used to filter "real bugs" from reproducibility issues that could be excused by demo data): https://github.com/shabtai/2025-26-flaw-analyses/blob/main/top-bugs.html
