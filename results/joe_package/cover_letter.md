# Cover letter — IEEE Journal of Oceanic Engineering

Dear Editor-in-Chief,

We submit our manuscript, **"PhysINR-Zero: zero-shot underwater image
enhancement with physics-constrained coordinate fields, evaluated under a
leakage-guarded re-benchmarking protocol,"** for consideration as a regular
paper in the IEEE Journal of Oceanic Engineering.

**Why this journal.** Our subject — measurement protocol and enhancement
method evaluation for underwater imagery used in ocean engineering — has a
long record in this journal: the underwater image formation model of
McGlamery (1980), the UIQM quality measure of Panetta et al. (2016), the
UWTBench benchmark of Panetta et al. (2022), the attenuated colour-channel
correction of Zhang et al. (2022) and the spatial–frequency interaction
transformer of Zhou et al. (2025). Our work continues that tradition, and the
benchmark conventions it introduces follow the journal's own two-track
protocol practice.

**What the paper contributes.**

1. **PhysINR-Zero**, an enhancement method that has seen no underwater
   photograph at all: a physics-constrained coordinate field (10,296
   parameters) inverting a Jerlov-based forward model, meta-trained on
   synthetic degradations of terrestrial photographs, adapted per image
   without references.
2. **HydroBench**, a leakage-guarded re-benchmarking protocol that re-scores
   ten published baselines under one implementation, one split and one metric
   set across five test sets (519 images), with per-model provenance records
   and labelled fallbacks.
3. A measured split between the metric families, with its mechanism: rendered
   through a reference-free display stage, the zero-shot method leads the
   field's no-reference colour measure (first of eleven on UCIQE on four of
   the five sets) while trailing the supervised leaders by 4.5–6.6 dB PSNR on
   the paired sets. The no-reference metrics that rank it well are the ones
   the field itself distrusts — the colour component rewards chroma spread —
   and the split is traceable to the loss terms in a fifteen-variant ablation.

**Verification.** Every number in the manuscript is audited against the
released results (467 prose literals checked, zero mismatches); rank claims
carry Holm-corrected Wilcoxon tests with effect sizes; bootstrap confidence
intervals and a power analysis bound what the 40-image ablation can resolve;
all 31 references were verified against Crossref. The statistical
re-verification scripts ship with the source package. An AI-use disclosure is
included in the Acknowledgment section, per IEEE policy.

All authors approved the submission and declare no conflicts of interest. The
manuscript is not under consideration elsewhere. We suggest no reviewers.

Sincerely,

Ali Hosseini (corresponding author) and Saeed Sedighi
Department of Computer Engineering, Shi.C. (Shiraz Branch), Islamic Azad
University, Shiraz, Iran
ali.hosseini7148@iau.ac.ir; saeed.sedighi@iau.ac.ir
