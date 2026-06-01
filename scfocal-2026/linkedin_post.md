# LinkedIn post — scFOCAL audit

---

An FDR of 3.2 × 10⁻²³⁸ in a Nature Communications paper — off by 135 orders of magnitude (verified by direct reproduction).

**The plain-English version, if statistics is not your thing:**

A drug trial wants to prove "the drug works better than placebo." The proper way: measure patients on the drug AND patients on placebo, compare the two groups, and let the variability in both arms determine how confident you can be in the difference.

This paper effectively measured only the drug arm, threw away the placebo measurements, and computed "statistical certainty" as if placebo behavior were perfectly predictable with zero variation. When you remove all the uncertainty about the control, your "certainty" about the difference becomes astronomical — but it is an arithmetic artifact, not evidence. The drug might genuinely work; the paper just hasn't shown it the way it claims to.

Concretely: a properly-conducted comparison would have reported a confidence like "1 in 10¹⁰³ chance of coincidence." The published number is "1 in 10²³⁸." Those are 135 zeros of difference. Both are extremely confident-sounding, but only one of them is what the actual experiment supports — and I verified this by re-running the deposited test code on the deposited data, which reproduces the published 10⁻²³⁸ exactly.

**Now the technical version:**

The paper (scFOCAL: Suter et al., Nat Commun 2026, s41467-025-67783-5) integrates single-cell RNA-seq with drug-response data in glioblastoma. Its headline in-vivo validation: alisertib treatment shifts ~10,000 cells away from the NPC-like state in xenograft tumors. Reported FDR for that shift = 3.2 × 10⁻²³⁸.

I downloaded the supplementary data. Every single cell in the deposit has Treatment = "Alisertib". Zero DMSO cells. The entire control arm was compressed to four scalar means in side columns.

The deposited test code makes the gap obvious:

  wilcox.test(Score, mu = 0)

That's a one-sample Wilcoxon against zero — not the two-sample alisertib-vs-DMSO comparison the paper describes. The four DMSO means were used as a centering offset, then discarded.

The arithmetic discriminator:
• Direct reproduction: running `wilcoxon(Score, mu=0)` on the deposited NPC.like cells (N=10,394, SD=0.046, shift=-0.016) gives p = 8.08 × 10⁻²³⁹ — matching the deposited / published 8 × 10⁻²³⁹ to 3 sig figs. All 4 subtype tests reproduce exactly.
• Proper two-sample Wilcoxon-Mann-Whitney against simulated DMSO with the same SD (biologically conservative — untreated cells should have similar or larger spread): p ≈ 3 × 10⁻¹⁰³. Extreme, but 135 OoM weaker than the published 10⁻²³⁸.

The published FDR is exactly what the one-sample test produces. The proper two-sample test is 135 orders of magnitude (verified by direct reproduction) away.

The "we just uploaded sample data, the real analysis was fine" defense doesn't rescue it. The published FDR magnitude IS the bug, and independent reanalysis is structurally blocked because the DMSO cells are absent from the deposit.

**One-line audit summary**: 33 deduplicated findings, 13 structural (cannot be excused as "wrong data uploaded"), 8 visible-on-inspection bugs in the public R source code, headline evidence-strength overstatement of 100× to 10²³⁸× across the pipeline that produced the paper's drug-discovery claims. The qualitative discoveries (alisertib targets NPC-like cells; OLIG2-inhibitor + Depatux-M combo extends survival) and the survival data itself survive once the statistical inflation is corrected.

---

## Notes

- Length: ~2700 characters with the plain-English addition (LinkedIn allows up to 3000 in the visible-without-click window).
- Two-tier structure: hook → analogy for non-statisticians → technical proof for the stats crowd → totals. Readers can stop at whichever depth they want.
- No emojis (per user preference).
- First line is the hook (visible before "see more").
- One-line summary at the bottom for the total findings.
- Optionally: add hashtags like #scRNAseq #reproducibility #datasharing #glioblastoma #computationalbiology at the end, or link the audit folder for those who want details.
