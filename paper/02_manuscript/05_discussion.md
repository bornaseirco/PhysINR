# 5. Discussion

## 5.1 What the results establish, and what they do not

The headline finding is a split, and it is the most consistent pattern in the
data. The two metric families now disagree about the method in a structured
way — it leads the no-reference colour measure on four of the five sets while
staying in the lower half of the paired rankings — and that split is itself the
result, because an earlier transmission prior of the same construction sat at
rank 2 on UCIQE and rank 9 on PSNR. The supervised leaders show the inverse
pattern: UIEC²-Net leads the paired metrics on UIEB and places fourth on UCIQE.

What this does *not* establish is that PhysINR-Zero enhances underwater images
better than the supervised baselines. It establishes something narrower and
more defensible: a method that has never seen a real underwater photograph
tops the field's no-reference colour measure on four of the five sets. The
same output recovers reference structure 4.5–6.6 dB below the supervised
leaders on the paired sets. A deployment that needs perceptual
plausibility — a human inspecting a feed, a colour-dependent annotation — and
a deployment that needs geometric or radiometric fidelity — a detector, a
mosaic, a spectral measurement — would draw opposite conclusions from
[Table](tab:uieb).

The colour-fidelity study of §4.4 is the qualification on that trade. The
no-reference metrics that rank the method well are the ones the field itself
distrusts (§5.3). The prior-design study shows the mechanism in its extreme
form: the water-type-lookup prior saturates 70 % of the green channel and still
posts the *highest* UCIQE of the five candidate designs, because rail-pinned
pixels supply the chroma spread and contrast the metric rewards. The shipped
prior scores lower on UCIQE and higher on every measure of
fidelity to the reference photograph; the CIELAB panel of §4.4 is what a
reader should consult for what the images actually are.

## 5.2 The mechanism is traceable
The split follows the loss terms directly. The colour-balance group operates
on CIELAB chroma statistics — mean and standard deviation over the a/b
opponent channels — which is exactly where UICM is measured. Sharpness enters
the loss only indirectly, through the structure group, and the prior group's
curve and magnitude terms penalise the nonlinearity exponent that controls
contrast amplification in the direction opposite to what UISM rewards. The
ablation confirms the correspondence causally (§4.3, [Table](tab:ablation)): the group whose
terms live on the colour metric moves that metric significantly while its
paired-metric cost does not survive correction; the adaptation-length knob,
by contrast, moves both families at once — in opposite directions. An
evaluator who needs sharpness rather than colour can read the loss and
predict the deficit before running anything.

## 5.3 The no-reference rankings should be read with the field's own reservations

The UCIQE ranking that places this method first of eleven on four of
the five sets is precisely the metric the field distrusts.
no-reference measures "satisfactorily rates the quality of enhanced underwater
images", and the UIEB benchmark paper that supplies this study's test images
states that its authors "discard the non-reference metrics designed for
underwater image enhancement" as inconsistent with human perception
(Li et al., 2020, §IV). The measures used here are their published definitions
with their published coefficients, so the rankings are reproducible; but a
reader who believes the critique should treat §4.2's no-reference orderings as
what the metrics say, not as what the images are. The metric's ordering makes
the point by itself: on UIEB its podium — PhysINR-Zero (32.186), Retinex
(31.663, 0.52 behind, Holm-adjusted p = 1.6e-03, r = 0.27) and UDCP (29.717)
— is occupied by the methods ranked seventh, tenth and eleventh on PSNR, and
on EUVP the same three hold it against PSNR ranks of ninth, tenth and
eleventh. That inversion is itself evidence the metric rewards something
other than reconstruction; §4.4 and the UICM decomposition of §4.2 identify
what.

## 5.4 Comparability

The re-run of three baselines onto their published models changed the
leaderboards, and not always in the same direction. On UIEB, replacing the
analytic approximations with the released networks moved PhysINR-Zero's UCIQE
rank, because the analytic fallbacks scored better on the colour metrics than
the real models do — the clearest possible demonstration of why the
comparability problem matters. An approximation standing in for a method
produces a different leaderboard, in a direction that flatters whichever method
the approximation happened to resemble. The published record reports such rows
as the method's own result without qualification; this study labels them and
excludes them from the ranking discussion. The recovery also shows that "no
checkpoint available" is frequently a retrieval problem rather than a property
of a method — three of four cases were resolved by tracking down released
weights or substituting an equivalent scan.

## 5.5 Limitations

- **Physical approximation.** Each camera channel is read at one
  representative wavelength rather than integrating its spectral response
  (§3.1) — the weakest part of the physics, bounded but not eliminated by the
  water-type softmax.
- **Synthetic–real gap.** The training tasks are sampled from the same forward
  operator the model inverts. That makes the family self-consistent and also
  means the meta-training loss cannot serve as a check on the method; the
  benchmark is that check, and the synthetic family's coverage of real water
  conditions is the untested assumption. The prior's level is set by the
  sampler's depth range, which §3.1.1 calibrates to the *mean* optical
  thickness real photographs carry but not to its *spread*; the residual tails
  — flat scenes over-saturating, the murkiest inputs under-restoring — are
  quantified in §4.4, and matching the spread is the direct next experiment on
  the sampler.
- **Single host, single seed.** All timings and scores come from one
  Apple-Silicon host using the Metal backend, with the accommodations of §3.8
  for two CUDA-only components. The results were not replicated on a second
  architecture or across seeds, so seed-to-seed variance is not quantified and
  the two host-specific substitutions are validated only against their own
  reference implementations.
- **Coverage of the field.** Five of eighteen prominent 2024–2026 methods ship
  compiled CUDA kernels with no Apple-Silicon build path and could not be
  included. The eleven-method field is therefore a selected, not exhaustive,
  cross-section.
- **Ablation power.** Fifteen variants on forty images is a small sample for
  effects below about 0.5 dB with Holm correction over fifteen comparisons, and
  the one-at-a-time design cannot detect joint necessity of loss groups.
- **No downstream evaluation.** Colour metrics are a proxy for what an
  inspection or perception pipeline needs. Whether the enhancement improves a
  detector or segmenter's performance on these images was not measured in this
  study and is the most direct next experiment.

# 6. Conclusions

A physics-constrained coordinate field, meta-trained on synthetic degradations of
terrestrial photographs and adapted per image without references, splits the
eleven-method field the same way on every set: first of eleven on UCIQE on
four of the five sets and second on the fifth, 0.23 behind Retinex, against
seventh on PSNR on UIEB and LSUI and ninth on EUVP — sixth on SSIM and on
LPIPS-AlexNet on UIEB, second on the colour-measure component behind only
UDCP, fourth on the sharpness and contrast components. It does not
reconstruct reference structure at the supervised leaders' level, trailing
them by 4.5–6.6 dB PSNR across the paired sets. The deficit pattern follows
the loss terms it optimises, and the ablation confirms the causal link:
removing the colour-balance group is the largest UCIQE change in the study,
it moves the colour component with it, and its paired-metric cost does not
survive correction.

An earlier transmission prior of the same construction scored differently —
second on UCIQE on four of the five sets against ninth on PSNR. §4.4 shows the
trade was taken deliberately: the recalibrated level head moves the output
closer to the reference photograph on every full-reference metric, and the
UCIQE it gives up is concentrated in a term that rewards chroma spread rather
than colour correctness.

Of the fifteen ablation variants, three differ significantly from the full
model on UCIQE, and one of them is also the only variant that moves PSNR
detectably: five adaptation steps costs 0.91 dB while buying 0.59 of UCIQE —
both significant, opposite directions — making the adaptation budget the one
lever both metric families track, against each other. Dropping the
colour-balance group raises UCIQE by 1.16, the largest no-reference change in
the study, while its 1.02 dB PSNR cost does not survive correction; twenty
adaptation steps lowers UCIQE by 0.32. The remaining twelve variants are
statistically indistinguishable from the full model on both families (§4.3).
That bounds the individual contribution of each physics and regularisation
loss group without establishing that the groups are redundant — forty images
cannot resolve sub-0.5 dB effects, and a one-at-a-time design cannot detect
joint necessity — and it locates the significant variation in colour handling
and adaptation budget rather than in the physics terms.

The method's practical claim is a cost claim. Ten thousand parameters, under
nine seconds of per-image adaptation, and no underwater training data mean the
enhancement is available at a new site on the first image rather than after a
paired-data collection campaign, at a known cost in pixel fidelity. The
benchmark's contribution is that the cost is measured against ten baselines on
one split with one metric implementation and labelled provenance — and that a
method whose checkpoint cannot be obtained is reported as unavailable rather
than as its own result.
