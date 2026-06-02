# 2025–26 Bioinformatics Paper Flaw Analyses

Independent audits of recent high-profile bioinformatics papers, restricted to findings where the "the public deposit is a subset, not the production cohort" defense does not apply — i.e., bugs that live in the committed source, contradictions in the paper text, or methodological choices independent of any specific row-level data.

Browse the HTML at [`index.html`](index.html).

## Papers

### Thompson et al., Nature Genetics 2025 — CIN chemo-resistance signatures
- DOI [10.1038/s41588-025-02233-y](https://doi.org/10.1038/s41588-025-02233-y) · PMID 40551015
- Code [macintyrelab/Thompson2025_ChemoResistancePrediction](https://github.com/macintyrelab/Thompson2025_ChemoResistancePrediction)
- Audit: [thompson2025/flaws.html](thompson2025/flaws.html)
- **Verdict:** PARTIAL — headlines reproduce; 25 verified novel findings about *what they mean*, including a published HR (Fig 2c) not derivable from committed code; 5 of 9 main HRs vacate under Bonferroni; RECIST coding flip moves breast taxane HR by +66%, breast anthra by −36%.

### Aung et al., Nature Genetics 2025 — NSCLC spatial immunotherapy signatures
- DOI [10.1038/s41588-025-02351-7](https://doi.org/10.1038/s41588-025-02351-7) · PMID 41073787
- Code [tznaung/NSCLC_SpatialOmics](https://github.com/tznaung/NSCLC_SpatialOmics)
- Audit: [aung2025/flaws.html](aung2025/flaws.html)
- **Verdict:** BREAKS HEADLINE — `Surv(as.numeric(time, event))` typo (11 occurrences) discards censoring; Yale resistance HR 3.8 (p=0.004) → 2.58 (p=0.053); Yale response HR 0.4 (p=0.019) → 0.30 (p=0.057). Both lose significance.

## Verification standard

Every finding shown was independently verified — by re-running the published code against the deposited data, by comparing paper text against script source, or by structural proof from code + data values.

No access to restricted clinical data used; everything is reproducible from public artifacts.

## License

Audit content is released under CC-BY-4.0.

Reproduction scripts are released under MIT.
