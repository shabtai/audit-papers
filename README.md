# audit-papers

Independent code &amp; data audits of published research papers, conducted with
[natural-joints.com](https://www.natural-joints.com/).

## Current audits

- **Wang et al., _Nature Biomedical Engineering_ 2025** — _A collaborative large language
  model for drug analysis_ (DrugGPT) ([PMID&nbsp;40987953](https://pubmed.ncbi.nlm.nih.gov/40987953/)).
  45 findings across five methodological-validity axes; released artifacts do not support
  the three headline benchmark comparisons (88.2% USMLE/MedQA, 88.0% PubMedQA, 97.1% DDI).
  See [`druggpt-2025/`](druggpt-2025/).
- **Monkman et al., _Nature Communications_ 2026** — _Spatial metabolic phenotyping
  of NSCLC immune-checkpoint-inhibitor response_
  ([10.1038/s41467-025-56818-6](https://doi.org/10.1038/s41467-025-56818-6)).
  37 findings; two empirically-confirmed blockers (silent deletion of four metabolic
  markers from the headline neighbourhood construct; +18.6 pp inflation on
  the OS-24-month landmark from informative censoring).
  See [`nsclc-monkman-2026/`](nsclc-monkman-2026/).
- **Deutsch et al., _Nature_ 2026** — _Expanding the human proteome with microproteins
  and peptideins_ ([10.1038/s41586-026-10459-x](https://doi.org/10.1038/s41586-026-10459-x)).
  See [`index.html`](index.html).
- **Thompson et al., _Nature Genetics_ 2025** — _Predicting resistance to chemotherapy
  using chromosomal instability signatures_
  ([10.1038/s41588-025-02233-y](https://doi.org/10.1038/s41588-025-02233-y)).
  25 verified novel findings; Fig 2c HR=20.020 is a hardcoded literal not derivable from
  the committed Cox model (recomputed HR=344); 5 of 9 main HRs vacate under Bonferroni;
  RECIST coded as `censoring=0` shifts 2 of 4 phase-3 HRs by &gt;30%; OV04 platinum HR
  depends on a script-injected 7-patient hand-coded list.
  See [`2025-26-flaw-analyses/thompson2025/flaws.html`](2025-26-flaw-analyses/thompson2025/flaws.html).
- **Aung et al., _Nature Genetics_ 2025** — _Spatial signatures for predicting
  immunotherapy outcomes using multi-omics in NSCLC_
  ([10.1038/s41588-025-02351-7](https://doi.org/10.1038/s41588-025-02351-7)).
  One-character bug `Surv(as.numeric(time, event))` discards censoring across 11 call
  sites; Yale resistance HR 3.8→2.58 (p 0.004→0.053) and response HR 0.4→0.30 (p
  0.019→0.057) — both lose significance.
  See [`2025-26-flaw-analyses/aung2025/flaws.html`](2025-26-flaw-analyses/aung2025/flaws.html).

Cross-paper deep dive on the [top 4 most impressive bugs](2025-26-flaw-analyses/top-bugs.html)
across Thompson + Aung, with the demo-data-defense methodology used to filter findings.

## Conventions

**Every published HTML page must include the GoatCounter analytics snippet**
inside `<head>` (placed immediately after `<title>`):

```html
<script data-goatcounter="https://natural-joints.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>
```

This is what surfaces visit counts on the natural-joints GoatCounter dashboard.
Pages missing the tag are invisible to analytics. Verify with
`grep -L goatcounter **/*.html` before publishing.
