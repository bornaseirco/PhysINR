# 4. Results

<!-- Numbers in this section are generated from experiments/merged_results/ and
     experiments/physinr_ablations/. Every table/figure below is traced to its
     source file in paper/07_provenance/figures_provenance.json. Do not edit a
     number by hand: rerun experiments/merge_results.py and
     experiments/make_figures.py. -->

## 4.1 Experimental setup

The evaluation covers five benchmark sets with 519 test images in total: UIEB
(134, seeded 70/15/15), LSUI (120), EUVP (120), RUIE (100) and UCCS (45).
UIEB, LSUI and EUVP are paired, so they admit full-reference metrics — PSNR,
SSIM, MS-SSIM and LPIPS with the AlexNet and VGG backbones — alongside the
no-reference set. RUIE and UCCS ship no reference images and are scored on
no-reference metrics only (UCIQE, UIQM and its components, and the
benchmark's HSV colour-cast diagnostic). Ten baselines spanning classical,
learning-based and restoration-oriented methods are compared against
PhysINR-Zero: nine run their published models, and one (WF-Diff) runs a
labelled analytic fallback.

One baseline could not be run as published: WF-Diff's weights are distributed
only through Baidu Pan, which requires an authenticated regional account, and
the authors' follow-up repository ships no checkpoint ("uploaded after paper
acceptance"). It runs as the labelled analytic fallback of §3.7. It is marked
`[fallback]` in the tables and excluded from the ranking discussion.

A second recent method is absent for a different reason. UniUIR (Zhang et al.,
TIP 2025) publishes no checkpoint at all: its repository ships an empty
`checkpoints/` directory, its project page links only to the paper and the
code, and no mirror exists on HuggingFace, Zenodo, ModelScope, Kaggle or Gitee
(each searched directly, September 2026). The repository also depends on the
CUDA-only `mamba-ssm`/`causal-conv1d` kernels, which have no build for this
host. Rather than train an unverified reimplementation and present it as the
authors' model, UniUIR is excluded outright and is discussed in §2 as a
reported-result comparison only. Its absence is stated here so the method
count is unambiguous.

Three further baselines became available after the benchmark was first
assembled and are now re-scored as their published models; every number in the
tables therefore comes from a released network. UIEC^2-Net runs from the
released checkpoint, whose DataParallel `module.` prefix is a naming artefact.
Five A+ runs from the weights its authors commit to their repository --- the
same release their own inference script defaults to, with the α = 0.1 loss
weights. WaterMamba needs the two host-specific accommodations recorded in
§3.8.

PhysINR-Zero adapts per image and never sees a real underwater photograph
during meta-training. Meta-training ran 4,000 tasks in 31 min 09 s and
converged to a meta-loss of 1.483 on held-out synthetic tasks. At test time the
field carries 10,296 parameters; adaptation costs 0.92–8.50 s/image depending on
frame resolution, measured on the Apple-Silicon host described in §3.8.

## 4.2 Comparison against baselines

Full-reference metrics separate the methods sharply, and PhysINR-Zero does not
lead them. On UIEB, UIEC²-Net reaches 23.654 ± 4.083 dB PSNR against
18.346 ± 3.660 dB for PhysINR-Zero ([Table](tab:uieb), [Fig.](fig:box-uieb)); the difference is
significant (Δ = 5.31 dB, p = 7.2e-22, r = −0.83). Five A+ (22.956 dB) and
Water-Net (22.712 dB) follow it, and Five A+ also leads SSIM (0.910 vs 0.827)
while the two split the LPIPS backbones — UIEC²-Net on AlexNet (0.127 vs
0.228), Five A+ on VGG (0.143 vs 0.256). PhysINR-Zero is seventh of eleven on
UIEB PSNR, ahead of the analytic WF-Diff fallback, FUnIE-GAN, Retinex and UDCP.
FUnIE-GAN is the closest method below at 16.471 dB; PhysINR-Zero is
1.88 dB above it and the difference is significant (p = 2.3e-07, r = 0.45). The
nearest method that is measurably above is WaterMamba at
20.906 dB (Δ = −2.56, p = 4.3e-06, r = −0.40). The same ordering holds on
| LSUI and EUVP: the supervised networks that were trained on underwater image
| pairs recover more of the reference signal than a method that was not. On
| LSUI, PhysINR-Zero is seventh of eleven (19.188 dB against
25.819 for UIEC²-Net, Δ = 6.63); on EUVP, ninth of eleven (15.603 against
20.071 for FUnIE-GAN, Δ = 4.47), where it also sits below the analytic
WF-Diff fallback column by 0.18 dB ([Tables](tab:lsui|tab:euvp); Supplementary Figs. S2–S3).

The no-reference metrics now tell the opposite story. PhysINR-Zero ranks first
of eleven on UCIQE on UIEB (32.186 against 31.663 for Retinex; the margin is
significant, p = 1.6e-03, r = 0.27) and repeats the lead on LSUI (31.618),
EUVP (30.988) and RUIE (31.803); on UCCS it is second (31.237 against 31.467
for Retinex). [Fig.](fig:qualitative)
shows the split on one test frame: PhysINR-Zero and Retinex carry the colour,
the supervised networks carry the structure. An earlier
transmission prior of the same construction ranked second on four of the five
sets; §4.4 reports the design study that produced the shipped prior, and the
display stage of §3.6 is what moved the colour measures from mid-field to the
top of the column.

The reason is UICM, the colour-measure component, and it is worth following
into the metric's definition. UICM is the standard deviation of two
log-chromaticity ratios, α = (g − b)/(g + b) and β = (r − g)/(r + g), so it
rewards chromatic *spread* across the image — how many distinct hues the frame
carries, not whether the white balance is right. Measured on the UIEB images
themselves, the reference photographs average 0.290 and the degraded inputs
0.174: recovering colour means *raising* chroma spread, which is why a
colour-cast amplifier can buy UCIQE without getting closer to the reference.
The recalibrated level head of §3.1.1 moves the prior's output the other way:
from 0.303 for the earlier prior — above the reference, i.e. over-saturated —
to 0.252, just under it. The full-reference metrics confirm which of the two is
closer to the photograph: PSNR rises 1.04 dB and LPIPS-AlexNet falls 0.134 over
that change, while UCIQE falls 2.94 because one of its three terms rewards the
over-saturation the other metrics penalise. The display stage of §3.6 is the
deliberate counterpart: it stretches the rendered frame's channel ranges back
out at output time, so the shipped model's UICM lands at 0.294 — the
reference's own level (0.290) restored through display contrast rather than
through the prior's level.

The component scores confirm the reading. UICM is where PhysINR-Zero is most
competitive among the learned methods — second of eleven on UIEB (0.294),
behind only UDCP (0.519) and just ahead of Retinex (0.283) — and the same
position holds on the other sets (second on LSUI, EUVP and RUIE, third on
UCCS). UISM and UIConM, the sharpness and contrast measures, sit fourth on
UIEB (0.291, 0.315); both are still led by UDCP and
Retinex, the two classical priors whose analytic sharpening is unbounded. The
optimisation is doing what its loss asks. The colour-balance group operates
directly on CIELAB chroma statistics, which is where UICM is measured;
sharpness enters only indirectly through the structure loss, and the prior
group's curve and magnitude terms penalise α — the nonlinearity exponent that
controls contrast amplification — in the direction opposite to what UISM
rewards. A method optimising a different objective would be expected to move
UISM; §4.3 tests this by varying the loss.

The colour-cast diagnostic, CCF, turns out to be the weakest diagnostic of
the set. Computed on the UIEB images themselves, the reference photographs
average 0.897 and the degraded inputs 0.900 — the metric cannot separate the
clean image from the one it was derived from. Every method lands in the
0.81–0.97 band, and the ranking it produces is not the ranking any
full-reference metric produces. This is a property of the metric, not of the
methods, and it is one more reason the no-reference numbers in this section
are reported alongside the paired ones rather than instead of them.

Per-image adaptation is what keeps the no-reference scores competitive, and
reducing it is the single most damaging ablation (§4.3).

## 4.3 Ablation study

Fifteen variants were evaluated on the first 40 UIEB test images. Because
`load_split` takes the front of the split, the first 40 entries of a
full-split column are exactly the images a `--limit 40` variant saw, so the
full model's column is truncated to those 40 values to keep the Wilcoxon tests
paired. Its mean on this subset (18.408 dB) is close to the 134-image mean
quoted in §4.2.

One variant moves PSNR detectably: five adaptation steps instead of twelve
costs 0.91 dB, at Holm-adjusted p = 0.010 (r = −0.61)
([Table](tab:ablation), [Fig.](fig:ablation)). The largest change in the
study is bigger — dropping the colour-balance group costs 1.02 dB — but it
does not survive the correction over fifteen comparisons (p = 0.633). The
largest of the remaining thirteen is 0.43 dB, from widening the deviation
bound to δ_max = 4, at p = 0.093. For context, the between-image standard
deviation of the full model's PSNR on this subset is 4.28 dB — every effect
this design detects or suspects is a small fraction of the spread it is
measured against.

The no-reference side tells a different story: three variants are
significant on UCIQE. In ascending Holm-adjusted p order they are: five
adaptation steps instead of twelve (Δ = +0.59, p = 1.4e-04, r = 0.76);
dropping the colour-balance group (Δ = +1.16, p = 6.1e-04, r = 0.71); and
twenty adaptation steps (Δ = −0.32, p = 0.019, r = −0.57). The full
fifteen-variant test panel is [Table](tab:ablation-sig). The two that raise
UCIQE move the output the way §4.2 predicts. Removing the colour-balance
group is the larger of the two, and its direction is the predicted one: the
group operates on CIELAB chroma statistics, UCIQE's colour-measure term
rewards chromatic spread, and the component view confirms the channel — the
same variant raises UICM by 0.10 (Holm-adjusted p = 4.9e-05) while UIQM
itself stays flat (largest variant Δ = +0.08, adjusted p = 1.000). Five
steps raises UCIQE through under-adaptation: fewer updates leave the render
less converged onto the data term, and the residual colour range is what the
metric rewards.

The adaptation-length row is the most informative in the study, because it
is a sweep rather than an ablation and its two ends move both metric
families, in opposite directions: five steps costs 0.91 dB of PSNR and buys
0.59 of UCIQE, both significant; twenty steps buys back 0.18 dB of PSNR
(non-significant) and gives up 0.32 of UCIQE (significant). The remaining
paired-metric damage confirms which variant is the costly one: five steps
also degrades SSIM (−0.02) and MS-SSIM (−0.02), both significant, and the
δ_max = 4 variant degrades SSIM and both LPIPS variants. The adaptation
budget is thus the one degree of freedom both families track, in opposite
directions. Twelve steps is neither end's optimum; it is a compromise, and
§4.2 already showed the direction in which the families disagree.

The negative result is that the remaining twelve variants are statistically
indistinguishable from the full model on the PSNR–UCIQE pair. Dropping any
one of the physical-model, cycle-consistency, Jerlov-prior, anti-trivial or
spatial-smoothness losses moves PSNR by at most 0.14 dB and UCIQE by at most
0.02, both at p = 1.000 ([Table](tab:ablation)). The coordinate-stride and
τ_max sweeps are likewise flat: stride 1 against stride 2 gives Δ = 0.00 and
stride 3 gives −0.15, and τ_max = 2 or 4 against 8 changes PSNR by at most
0.05 dB — all at p = 1.000. The deviation bound is flat at the tight end:
tightening δ_max to 0.5 raises PSNR 0.07, at p = 1.000. Its wide end
(δ_max = 4) is the exception discussed above, guarded by SSIM and LPIPS
rather than by PSNR; the default of 1.5 therefore sits on a plateau rather
than at a boundary optimum. Following §8 of the technical specification, an
optimum at a sweep boundary would be reported as a constraint, not a result.

Three readings are consistent with that pattern, and the experiment cannot
fully separate them. Six of the seven loss groups may be genuinely redundant
on UIEB — the dataset's turbidity range may be narrow enough that the
constraints do not bind. They may be individually small but jointly
necessary, which a one-at-a-time design cannot detect. Or the ablation may
be underpowered: forty images is a small sample for effects below about
0.5 dB, and the colour-balance deficit — the largest PSNR effect in the
study at 1.02 dB — still sits at adjusted p = 0.633, so the line between no
effect and an effect not yet detected is drawn by the sample as much as by
the design. What the study does establish is a division of labour. Six of
the seven loss groups are, singly, invisible to both metric families; the
two knobs that do move metrics — the colour-balance group and the adaptation
budget — act through colour handling and per-image optimisation rather than
through the physics terms, and where they touch both families at once, they
move the paired and no-reference families in opposite directions.

## 4.4 The prior's design space: why the level has to be learned

The score tables report the shipped model; this section reports how its prior
was arrived at, because the candidate designs fail in opposite directions and
the failure mode is invisible to the §4.2 metrics — a result about the
evaluation protocol as much as about the method.

Four candidate designs were compared on the same 20 UIEB test frames and 12
UCCS colour-cast frames at 480 px, 12 adaptation steps, under one protocol,
plus the raw input as a floor, plus an intermediate that the design passed
through on the way from A to B:

- **A — water-type lookup (the legacy prior).** τ_prior = (w · β) · d with no
  measurement and no learned level. This is the prior of the published
  checkpoints, reproduced exactly by the legacy configuration.
- **Intermediate — content-dependent water-type head.** Replacing A's fixed
  lookup with a head that predicts the water type from image content. This is
  reported because it fixed A's saturation and still failed, and the failure
  is what motivated the measured-level priors.
- **B — measured only.** τ_prior = τ_data with the estimator of (6), which
  normalises the best channel to t = 1 by construction.
- **C — measured + learned level, uncalibrated depth.** The head of §3.1.1,
  supervised on a sampler drawing depth in 0.5–12.0 m.
- **D — measured + learned level, calibrated depth (the shipped model).**
  The same head, supervised on the 0.5–4.0 m range of §3.1.1.

The metric that separates them is not in the §4.2 panel. Saturation raises
contrast and colour measures even as it destroys content, so a broken output
can *outscore* a faithful one on UCIQE. CIELAB chroma, lightness L\* and ΔE
against the reference photograph do not have that defect, and the
per-channel clipping fraction — the share of pixels within 1/255 of either
rail — is what sees the saturation directly.

[Table](tab:revision) reports the five priors on that protocol.

**A over-amplifies.** On a UIEB test frame the legacy prior implies τ_G = 1.11
(t = 0.34) where the water plausibly sits near 0.1–0.3. The inverse therefore
divides by 0.34, amplifies the green channel three-fold and pins **70.5 % of
its pixels at 1.0 before a single adaptation step runs** (1.6 % of the red
channel at 0). The saturated regions are flat, which reads as the posterised
neon green/cyan wash. The mean clipping over the 20 UIEB frames is 16.5 % of
the green channel against the input's own 0.3 %. Its UCIQE is the *highest* of
the five priors --- 29.11 against 19.56 for the raw input --- because
rail-pinned pixels supply exactly the chroma spread and contrast the metric
rewards. This is the metric-inflation trap of §5.3 in its extreme form.

**The intermediate fixes the saturation and fails anyway.** Predicting the
water type from image content rather than looking it up removes the blowout
entirely: green clipping falls from 16.5 % to 0.3 %, matching the input, and
the UCCS cast falls from 43.1 to 26.8 — *below* the 35.2 the input carries.
The output is less casted than what went in. But the same restraint costs the
metric that rewarded the blowout: UCIQE drops to 20.51 on UIEB (against A's
29.11) and to 16.39 on UCCS, barely above the input's 15.60. The head learned
to be conservative because the training signal could not tell it where the
right level was. The defect moved rather than disappearing — A over-saturates,
the content head under-restores, and neither knows the water's actual optical
thickness — which is what motivated measuring it instead of predicting it,
as B, C and D do.

**B under-amplifies.** Removing the blowout by measurement alone trades it for
the opposite defect. The range identity (5) is scale-free, so normalising the
best channel to t = 1 sets it above the real value of ≈ 0.59 on every frame,
under-amplifying by ≈ 1.7×. The output is washed out and dark: L\* 43.95
against the reference's 49.07, mean red 0.336 against 0.413. Chroma lands on
the reference's 11.84 almost exactly (11.37) — but it arrives there by
*desaturating* rather than by restoring. The score-table metrics pass while
this happens: cast, clipping and neon fraction all read as fixed.

**C over-amplifies differently.** The learned head is the right mechanism, but
supervised on the wrong distribution. At 0.5–12.0 m the sampler's synthetic
best-channel transmission averages 0.32 with a 10th percentile of 0.011 —
roughly twice the optical thickness real photographs carry (0.59, p10 0.50).
The head learned the murk it was taught: on real UIEB frames it predicts
t_best = 0.058 where the truth is 0.588, a **10.2× over-amplification**. It is
the worst of the five on PSNR, on SSIM and on ΔE (0.659 and 15.84).

**D is the first prior that is simultaneously correct.** The recalibrated
0.5–4.0 m range of §3.1.1 moves the synthetic t_best mean to 0.579 against the
measured 0.59, and the head's residual error collapses. Chroma 10.55 against
the reference's 11.84, L\* 49.76 against 49.07, mean red 0.401 against 0.413,
ΔE 11.22 (the lowest of the five), and SSIM 0.7781 — above the input's own
0.7229 and above every other prior. The neon fraction sits at the reference
level on every frame where the reference is at zero.

The reading is that the prior's **level** and its **ratio** are different
estimation problems with different difficulty. The level cannot be solved from
a single frame at all — it has to be carried by the training distribution.
That is why the sampler's depth range is derived from the optics of §3.1.1
rather than chosen for computational convenience.

**Honest caveat.** The aggregate means are strong, but 6 of the 20 UIEB
frames deviate materially: four over-saturate where the reference is flat
(chroma error 13–21) and two — the murkiest inputs in the set — under-restore
to L\* ≈ 17.5 against a reference of 46.8. The calibrated range matches the
real distribution's *mean* (0.579 vs 0.59) but not its *spread* (real p10
0.50), and the residual errors concentrate at the tails.

[Fig.](fig:revision) shows the corresponding frames: the five priors on
the same inputs, side by side.

<!-- TABLE/FIGURE MAP (generated; keep in sync with experiments/build_latex.py
     TABLES and FIGURES. Floats are numbered in the order they appear in the
     LaTeX, which is the order of those two lists. Prose references use the
     [Table](tab:label) xref syntax and resolve via \ref, so they cannot go
     stale; the numbers below are for human readers only.)
Table 1  comparison_UIEB.md        (11 methods, n=134)
Table 2  comparison_LSUI.md        (11 methods, n=120)
Table 3  comparison_EUVP.md        (11 methods, n=120)
Table 4  comparison_UCCS.md + comparison_RUIE.md side by side
                                    (11 methods, n=45 / n=100, no-reference only)
Table 5  revision_v4.md            (prior design study, five sides, §4.4)
Table 6  ablation_UIEB.md          (16 methods, n=40)
Table 7  ablation_UIEB_significance.tex (paired Wilcoxon, Holm)
Fig. 1   method_diagram.pdf
Fig. 2   qualitative_UIEB.pdf
Fig. 3   revision_v4_{uieb,uccs}.pdf  (prior design study, §4.4, side by side)
Fig. 4   box_UIEB_{psnr,ssim,ms_ssim,lpips_alex}.pdf
Fig. 5   ablation_UIEB_psnr.pdf
Supplementary (supplementary.pdf, Figs. S1--S4):
S1 meta_train_curve.pdf; S2 box_LSUI_*; S3 box_EUVP_*; S4 radar_*
-->
