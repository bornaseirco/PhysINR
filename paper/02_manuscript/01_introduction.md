# 1. Introduction

Underwater imagery is the primary sensing modality for civilian ocean
engineering: hull and infrastructure inspection, environmental monitoring,
seafloor mapping and the perception stack of autonomous underwater vehicles.
All of these degrade the image in the same way, because water attenuates light
selectively by wavelength and scatters it toward the camera. The result is a
colour cast toward blue-green, reduced contrast and a veiling backscatter that
removes exactly the texture an inspector or a detector needs. Enhancement
before analysis is not optional on most field imagery.

Two families of method address it. Supervised deep networks — Water-Net
(Li et al., 2020), FUnIE-GAN (Islam et al., 2020), U-Shape Transformer
(Peng et al., 2023) — learn the correction from paired degraded/clean images.
The comparison also covers UIEC²-Net (Wang et al., 2021), Five A+ (Jiang et
al., 2023), TOPAL (Jiang et al., 2022), WF-Diff (Zhao et al., 2024) and
WaterMamba (Guan et al., 2024).
They recover reference structure well, and they bind to the data they were
trained on: a network trained on one water type, one camera and one depth range
is being applied to a different one. The classical priors — the underwater dark
channel prior (Drews Jr. et al., 2013) and multi-scale Retinex (Jobson et al.,
1997) — need no training data and generalise anywhere, but reconstruct
reference structure poorly (§4.2). Between the two there is an operating point
that is easy to state and hard to reach: a method that learns, but not from
underwater photographs.

The second problem is measurement. Published results in this field are not
comparable across papers, and the spread is not marginal. Re-runs of Water-Net
on UIEB by papers that cite it span 16.305 to 19.81 dB — a 2.8 dB range, wider
than the improvement most new methods claim over their predecessor. U-Shape
Transformer is credited with UIEB PSNR 21.25, 22.91 and 21.55 by three
different papers. Thirty-seven such contradictions were catalogued across
recent surveys, and on no-reference metrics the disagreement is proportionally
worse: one method's UIQM on the same test set differs by a factor of 1.5 to 1.9
between two 2026 surveys. The causes are structural, not careless. Test
populations differ — UIEB alone is variously used as its 90 paired images, its
60-image challenge subset, a 190-image or a 45-image selection; LSUI is split
3879/400 by one group, 504 test images by another, 427 by a third. Metric
implementations are re-written per paper and preprocessing varies from 256 × 256
resize to full resolution. The field has reached this conclusion independently:
the comprehensive survey of Cong et al. (2024) re-ran seventeen methods under a
single protocol for exactly this reason, and GuidedHybSensUIR (Guo et al.,
2025) re-benchmarked thirty-seven. Any claim about a new method's standing
therefore has to be made on a protocol that fixes all three: split,
implementation and metric set.

This paper asks two questions and answers both empirically. Can a method that
has seen no underwater photograph at all be competitive on standard benchmarks?
And where, precisely, does it fall short? The answer to the second question
turns out to be more interesting than the first, because the method fails
predictably, in one metric family and not the other, and the failure pattern is
traceable to the loss it optimises. The venue framing matters here. Benchmark
and enhancement work published in the ocean-engineering literature has long
treated measurement protocol as part of the method --- from the underwater
image formation model of McGlamery (1980) to the quality measures of Panetta
et al. (2016) and the enhancement benchmarks of Panetta et al. (2022) and
Zhang et al. (2022) in the IEEE Journal of Oceanic Engineering. This paper
continues that convention: the protocol is specified, provenance-labelled and
audited before any method is compared under it.

We present PhysINR-Zero, an enhancement method whose learned weights are
meta-trained on synthetic degradations of ordinary terrestrial photographs and
applied to underwater images without further training. The scene is represented
as a coordinate field of physical quantities — per-channel optical thickness, a
bounded nonlinear intensity correction and ambient light — which inverts a
forward model built on the Jerlov water-type classification. At test time the
field is adapted to each image against reference-free losses; the method never
loads a reference image, and the code path is structured so that it cannot. The
model carries 10,296 parameters.

The contributions are:

1. **A zero-shot physics-constrained enhancement method.** PhysINR-Zero
   meta-trains on synthetic degradations sampled from the same forward operator
   it inverts, so task distribution and inductive bias are matched by
   construction. The zero-shot boundary is enforced in code, not assumed: the
   training CLI refuses a source directory that resolves inside any benchmark
   dataset, and a perceptual-hash deduplication pass drops near-duplicate
   training sources against every evaluation split, with the counts written to
   the run's provenance record.
2. **A leakage-guarded re-benchmarking protocol.** Every method in the
   comparison — classical prior, GAN, transformer and state-space model — is
   scored by one code path on five benchmark sets (519 test images) with eleven
   metrics, split-disjointness verified against sha256 manifests and a table
   rendered only when the columns are paired. Three baselines whose published
   weights were initially unavailable were recovered to their released models
   rather than left as approximations; the one method whose checkpoint cannot be
   obtained is marked as a fallback and excluded from the ranking, rather than
   reported as its own result.
3. **A measured split between perceptual quality and signal reconstruction.**
   PhysINR-Zero trails the supervised leaders by 4.5–6.6 dB PSNR on the
   paired sets, while improving on every full-reference metric relative to an
   earlier prior of its own design. A reference-free display stage stretches
   the conservative inverse's compressed output ranges, which moves the
   no-reference colour measures to the top of the field — first of eleven on
   UCIQE on four of the five sets — while the paired metrics keep their
   ordering. The split follows the loss
   terms, and a CIELAB fidelity panel against the reference photographs shows
   the no-reference metrics rewarding the chroma spread that a colour cast
   *is* — a result about the evaluation protocol as much as about the method.
4. **A bounded negative result.** Of fifteen ablation variants, twelve are
   statistically indistinguishable from the full model on both metric
   families. Three register on the no-reference side only — colour-balance
   removal and the two adaptation-length ends move UCIQE significantly — and
   exactly one, five adaptation steps, also moves PSNR detectably, costing
   0.91 dB while buying 0.59 of UCIQE. That bounds the individual
   contribution of the physics and regularisation loss groups rather than
   establishing that they are unnecessary, and it locates the significant
   variation in colour handling and adaptation budget rather than in
   reconstruction.

The honest summary is in §5: measured against supervised baselines on paired
benchmarks, PhysINR-Zero is a perceptual-quality method rather than a
signal-reconstruction method, and the no-reference metrics that rank it well are
themselves distrusted by the field, including by the authors of the benchmark
that supplies its test images. Both halves of that sentence are reported.
