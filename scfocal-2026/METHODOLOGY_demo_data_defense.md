# Methodology — Demo-data defense check

A filter for paper-audit findings. Applied AFTER the standard "real bug vs deposit issue" sort, this is the stronger second pass: if the authors said "the deposit is illustrative / sample / placeholder data; the real analysis used different data and produced the published numbers correctly," would the finding still hold?

A finding that survives this filter is safe to publish as a critique without giving the authors a clean rebuttal path. A finding that falls under this filter should be reframed as "deposit / reproducibility issue" rather than "analytical bug."

## The defense

> The deposit is illustrative / sample / placeholder data; the real analysis used different data and produced the published numbers correctly.

This is the strongest rhetorical position the authors can take when their deposit is shown to contain errors: separate the deposit from the analysis. It is sometimes legitimate (e.g., GEO-deposited summary tables vs raw counts on Synapse), and sometimes a retreat from accountability. The check below distinguishes the two.

## Pass criteria

A finding survives the demo-data defense if **at least one** of these is true:

- **A. Code-level bug.** The bug is in the deposited *source code* — which the project publishes as the actual pipeline, not as a demo. If they claim the committed code is "demo code," they are claiming the public repo doesn't reflect the analysis, which is itself a major transparency / reproducibility failure.
- **B. Code-and-data cross-validate.** The published numbers in the paper match what the deposited code, applied to the deposited data, produces. The deposit IS the analysis substrate, regardless of label.
- **C. Reproduction by coincidence is implausible.** The deposited numbers reproduce paper-headlined values to enough precision (multiple decimals, or to-the-day medians, or exact significance counts) that "this is demo data that happens to coincide with the paper" is statistically implausible.

A finding **falls** if none of A/B/C hold — i.e., if the deposited material could plausibly be a sample / placeholder / illustrative subset, and the published results could have come from a different, unpublished analysis.

## Procedure

For each finding from the prior "real bugs" tier:

1. **State the strongest demo-data defense** the authors could mount for this specific finding. Be charitable; assume they will frame it optimally.
2. **Check each criterion (A / B / C) in turn:**
   - For A: is the bug in committed source files (`.R`, `.py`, `.ipynb`, `.Rmd` in the public repo)? If yes → survives.
   - For B: does the published paper number match what the deposited code+data produces? If yes → survives. (Run the computation if needed.)
   - For C: do the deposited numbers reproduce paper-headlined values to many digits / to the day / exactly? If yes → survives, because reproduction by coincidence on N independent numbers is implausibly precise.
3. **Record which criteria (A/B/C) the finding passed on.** A finding passing on multiple criteria is stronger.
4. **If a finding fails all three:** demote from "real bug" tier to "deposit / reproducibility issue" tier. The critique should be reframed accordingly.

## Outputs to produce

Per-finding row in a validation table:
- Finding ID
- Strongest demo-data defense
- Concrete reason it fails (with cited file lines, deposited values, or arithmetic discriminator)
- Which of A / B / C the finding survives on

Collective check:
- For all surviving findings, articulate the joint claim the authors would have to make to excuse them simultaneously. This is usually a chain of compounding admissions (the code is demo + the data is demo + the published numbers are coincidences + the real pipeline is hidden) that no reviewer would accept.

## Why this matters

A naive "the deposit has bugs" critique invites the easy rebuttal: "the deposit was just for illustration." Reviewers and editors give this rebuttal weight by default. The demo-data defense check forces the critic to confront the rebuttal up front and either:

- **Strengthen the critique** by tying the bug to the published numbers themselves (criteria B/C) or to the source code (criterion A), OR
- **Reframe the critique** as a reproducibility / data-availability complaint rather than an analytical-bug claim, which is a different (often weaker, but still valid) accusation.

Both outcomes are better than letting the bug sit in a category where a clean rebuttal exists.

## When to skip the check

- For purely cosmetic findings (typos, header formatting): not worth the check; demote to "low-severity" regardless.
- For findings already known to be deposit-only by definition (e.g., "X file is missing from the deposit"): the demo-data defense doesn't apply because there is no analytical claim to defend.
- For findings about a paper that explicitly labels its deposit as "demo" or "sample": the burden shifts to the paper to disclose where the real analysis lives; the critic should pursue that, not run this check.

## Provenance

This methodology was developed during the scFOCAL audit (`audit-1780255732`, 2026-05-31, paper s41467-025-67783-5). Per-finding application is in `DEMO_DATA_DEFENSE_CHECK.md`; the 13 real-bugs findings to which it was applied are in `real_bugs.html`. All 13 survived; collective rebuttal-chain analysis is at the bottom of `DEMO_DATA_DEFENSE_CHECK.md`.
