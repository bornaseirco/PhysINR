# 3. Materials and methods

## 3.1 Physical forward model

The method inverts a forward model of underwater image formation
(McGlamery, 1980; [Fig.](fig:method)). For channel *c* of a pixel, the model
is

    J_c = bounded_nonlinear(I_c, α_c)                          (1)
    I'_c = J_c · t_c + A_c · (1 − t_c) + r_c                   (2)
    t_c = exp(−τ_c)                                            (3)

where *J* is the scene radiance after a bounded nonlinear correction, *τ* is
the optical thickness per channel, *A* the ambient (backscatter) light level
and *r* a residual. The corrected radiance is recovered by inverting (2):

    Ĵ_c = (I_c − Â_c · (1 − t̂_c)) / (t̂_c + ε).               (4)

The nonlinear correction in (1) uses the monotone remap *g(v) = v(1−v)(2v−1)*
bounded by α ∈ [0, 2] with a soft shoulder, which compresses recovered radiance
that falls outside the displayable range back into it without clipping.
The optical thickness is anchored to a water-type prior. Ten Jerlov water
types (Jerlov, 1976, Table XXVII) supply a per-channel diffuse attenuation
coefficient β_c, giving a prior of β_c · *d* for a global depth *d*. The
field may deviate from it by at most δ_max = 1.5 in optical thickness, through
a tanh activation, so the correction is bounded, smooth and saturating rather
than free. The oceanic type-I row of Jerlov's table lies below pure-seawater
attenuation at 9 of 15 tabulated wavelengths, which is not physically possible
for a natural water body. That row is substituted with the pure-seawater
attenuation curve of Austin and Petzold (1986), the standard reference for
clear oceanic K_d. The remaining nine rows are used as published, and the 1976
edition is used throughout — the widely reproduced 1968 table differs from it
by up to 34 % in individual entries, and the two editions must not be mixed.

**The largest acknowledged physical approximation.** Each camera channel is
read at one representative wavelength — 608, 545 and 455 nm for R, G and B —
rather than integrating the channel's spectral response against the attenuation
spectrum. The wavelengths are the peaks of each sRGB channel's response,
computed from the CIE 1931 colour-matching functions and the IEC 61966-2-1
primaries under D65, so the single-point estimator weights the broadband
response where it is heaviest. A broadband spectral integration would be more
accurate, but it requires auxiliary spectral measurement that the method
deliberately does not assume it has: it must work on ordinary sRGB imagery.
This approximation is stated here rather than buried in implementation, because
it is the one place the physics is weakest for exactly the channel that matters
most in turbid coastal water.

## 3.1.1 The transmission prior: level and ratio

The prior's **spectral ratio** and its **level** are estimated by separate
mechanisms, because they have different identifiability. Image formation is
affine in the scene radiance, so the ambient wash shifts the image without
compressing it:

    range(I_c) = t_c · range(J_c).                                          (5)

The per-channel percentile range (1st–99th) therefore recovers the transmission
*ratios* directly,

    t_data,c = range_c / max_c range_c',      τ_data,c = −ln t_data,c       (6)

with the least-attenuated channel as the scene reference, a 0.25 floor on the
reference, and t_data = 1 when the frame carries no contrast at all. The
estimator is parameter-free and needs no water type. On synthetic tasks the
correlation between τ_data and the true per-channel τ is 0.42–0.52, and the
mean error in the recovered ratio is 0.05 — the ratio is well identified.

The **level** is not. The range identity (5) is scale-free: it fixes t_R : t_G
: t_B but leaves the common scale undetermined, and no single-frame statistic
identifies it. On the UIEB test set the correlation between every image
statistic examined and the true best-channel transmission is only −0.33 to
−0.40, and the normalisation in (6) forces the best channel to t = 1 by
construction where the real value averages 0.59. An estimator-only prior
therefore under-amplifies by ≈ 1.7× on every frame — a deficit adaptation
cannot correct, because the wrong constant is written into the prior and the
inner loop has no signal to correct it against.

The level is instead **learned**, and supervised on the synthetic task's own
ground truth. A linear head on the same 41-dimensional content descriptor as
the water-type head predicts the best-channel transmission, and the outer
meta-loop supervises it against the sampled task's own best-channel
transmission, t_best = the maximum over channels c of exp(−β_c d).
The head is zero-initialised, so sigmoid(0) = 0.5 reproduces the
estimator-only starting point and training only has to move it. Its output
enters the prior additively as τ_level = −ln t_best, clamped to (1e-3, 1) —
deliberately wide rather than at the t_min floor, because censoring the level
at t_min truncates the synthetic target and biases the head toward the
saturation region. Amplification safety is enforced separately, by the floor
of §3.1.2 on the transmission entering the inverse.

The prior is therefore

    τ_prior = w_τ · τ_data + τ_level + τ_Jerlov,                             (7)

where the Jerlov component explains only the residual the measurement cannot
see — the absolute offset of the reference channel and the spectral shape
beyond what contrast reveals — so on synthetic tasks its supervision target
becomes τ_true − τ_data and the two terms cannot double-count.

**The depth range the level head is calibrated on.** The head learns whatever
distribution of t_best the sampler draws, so the sampler's depth range must
match the optical thickness real photographs carry. Sampling scene depth
uniformly in 0.5–12.0 m with the table's attenuation coefficients draws a
synthetic t_best with mean 0.32 and a 10th percentile of 0.011 — far murkier
than real frames, whose best channel sits at t ≈ 0.59 with a 10th percentile
of 0.50. A head supervised on the former predicts exactly the murk it was
taught. The range used here, 0.5–4.0 m, is derived from the optics rather
than from any evaluation set: solving exp(−β_best · d) = 0.59 with the
clear-water table coefficients gives d ≈ 4 m, and the resulting synthetic
t_best mean is 0.579 against the measured 0.59. The evaluation set was used
only to *check* the calibration, never to set it, so no real underwater image
entered the meta-training distribution.

## 3.1.2 Bounded amplification

The division in (4) is floored at t_min = 0.25 (amplification capped at 4×),
applied identically in the re-degradation operator so cycle consistency is
preserved exactly. The floor is a guard against pathological amplification —
1/t reaches 3,000 at the τ = 8 bound — and does not by itself prevent a 3–4×
saturation; the measured prior of (6) plus the learned level are what keep the
inverse in range.

**The failure the floor guards against is documented, not hypothetical.** An
earlier prior that set the thickness level from the water-type assignment
alone — τ_prior = (w · β) · d, with no measurement and no learned level —
over-estimated τ on real photographs by whole units. On a UIEB test frame that
configuration implied an effective τ_G = 1.11 where the water plausibly sits
near 0.1–0.3, so the inverse divided by t = 0.34, amplified the green channel
three-fold and pinned **70.5 % of its pixels at 1.0 before a single adaptation
step ran** (1.6 % of the red channel at 0). The saturation regions are flat,
which reads as the posterised neon green/cyan wash visible in the qualitative
figure, and the adaptation loop spent its twelve steps fighting the blow-out
rather than restoring the scene. The mechanism is worth stating precisely,
because it is not the water-type lookup that breaks: the Jerlov term alone
gives τ = (1.62, 0.64, 0.86), in which green is the *least* attenuated
channel. The blowout is carried by the spatial field, whose learned deviation
from the prior adds +0.47 to green and removes 1.49 from red. A prior with no
data-driven anchor places no constraint the field has to respect, so the field
learns whatever the loss rewards — and on synthetic tasks that is an
over-thick green channel. The failure is architectural, not numerical: the
inverse computes exactly what (4) specifies, for a level no single-frame
measurement ever supported. The legacy configuration (no measured prior, no
level head, no floor) reproduces that behaviour exactly, and its checkpoint's
numbers are reported as such in §4.

## 3.2 The coordinate field

The model is parameterised by a coordinate field evaluated at the image grid.
A small feature extractor — three depthwise-separable convolution blocks with
16, 24 and 32 channels, at most 4 × total downsampling, and LeakyReLU(0.1) —
produces a spatial feature map. The field then maps position plus local
feature to ten physical outputs: per-channel optical thickness (3),
per-channel nonlinearity (3) and per-channel ambient light (3). One
water-type logit completes the ten, and its softmax over the ten Jerlov types
makes the water-type assignment soft rather than committed. Each output is
bounded into its physical range. Three linear heads read a 41-dimensional
content descriptor: the pooled field logit, the full-resolution channel means,
their log-ratios, the estimated per-channel background light and pooled
extractor features. They produce the water-type weights, the scene depth and
the prior level of §3.1.1, so all three are functions of the image rather than
of the spatial mean of a single scalar.
parameter budget is 25,000 by design; the trained model carries 10,296.

The field is a continuous function of position. During adaptation it may be
evaluated on a strided grid — the default stride is 2 — and the enhanced image
is rendered once at stride 1. This is an approximation in *sampling*, not a
different model. Pixel *i·s* receives the identical coordinate it would receive
at stride 1. The feature map is resampled to the full grid once and then
*indexed* at the sampled positions, so *f(x)* remains a function of position
rather than of the grid it was computed on. The residual discrepancy on shared
pixels is ~1e-5 in optical thickness and arises only from the two global
poolings being estimated from the sample. The stride ablation (§4.3) tests the
default rather than assuming it.

## 3.3 Synthetic degradation sampling

Meta-training tasks are synthesised from clean photographs. Each task samples
a scene depth uniformly in 0.5–4.0 m, the range §3.1.1 derives from the optical
thickness real photographs carry rather than a guess about survey geometry. A
water type is drawn by softmax over the ten Jerlov types, resampled per task,
with ±15 % jitter on each channel's attenuation and a local optical-thickness
perturbation up to 0.35. The sampler adds an ambient light level in 0.15–0.85
with a ±0.1 per-channel colour shift, blur σ in 0–1.2 and noise σ in 0–0.04.
The degraded image is produced by the *same* forward operator the model
inverts, so the task distribution and the model's inductive bias are matched by
construction. This is a deliberate choice that makes the synthetic family
self-consistent, and it is also the reason the benchmark, not the meta-training
loss, is the test that matters: a synthetic family that matches the model
cannot also serve as an unbiased check on it.

## 3.4 Meta-training

The outer loop is first-order MAML (Finn et al., 2017) over synthetic tasks.
The clean pool is preloaded at a maximum side of 384 px — twice the crop size,
so a portrait source is never scaled below the resolution the inner loop
samples. Of 900 candidate photographs, 898 survived the duplicate check against
every evaluation set, and 882 were large enough for a 192-px crop. Each of the
4,000 tasks samples a batch of four 192-px crops and degrades each. The inner
loop runs four steps at learning rate 0.01 on the reference-free loss. A
meta-step then updates the *pre-adaptation* parameters against the outer loss,
which compares the adapted output with the synthetic clean crop.
exactly the loss that test time sees; the outer loop sees the clean crop, whose
ground truth was manufactured by the sampler rather than collected. The
distinction is kept in the claims: the method is zero-shot with respect to real
underwater data and meta-trained with synthetic supervision. Parameters are
projected back into their bounds after each step. In addition to the task
mixture, the outer loop supervises the three content heads of §3.2 against the
sampled task's own ground truth: the water-type mixture, the scene depth, and
the best-channel transmission that sets the prior level of §3.1.1. The inner
adaptation loop remains unsupervised and sees exactly the loss that test time
sees. The supervision reaches the *initialisation*, never the test image,
so the zero-shot claim is unchanged. The reported run trained in 31 min 09 s
over 4,000 tasks and converged to a meta-loss of 1.483 on held-out synthetic
tasks (Supplementary Fig. S1).

**The zero-shot boundary is enforced, not assumed.** The most efficient way to
destroy the claim is also the most tempting shortcut — pointing meta-training at
a benchmark dataset's reference directory, the cleanest imagery in the
repository. The training CLI therefore refuses any source path that resolves
inside a dataset carrying a test split, deriving that list from the committed
manifests so a newly added dataset cannot leak through a list nobody updated.
A perceptual-hash deduplication pass then drops near-duplicate training sources
against every evaluation split, writing the kept and dropped counts to the run's
provenance record. The reported pool is DIV2K (Agustsson and Timofte, 2017): 882
terrestrial photographs. The pass dropped two of them, whose perceptual hashes
matched EUVP test images — a real collision between the two collections, caught
rather than assumed away.

## 3.5 Loss functions

Seven loss groups are each weighted in the configuration: physics
reconstruction, cycle consistency, colour balance, structure, spatial
smoothness, a Jerlov group and a prior group. Physics reconstruction is
Charbonnier on the re-degraded image plus an optical-thickness range penalty;
cycle consistency is a re-degradation round-trip on image and gradient. Colour
balance matches mean and standard deviation over CIELAB a/b under D65 with a
transmission-masked chroma prior. Structure adds SSIM, edge, entropy and noise
terms, and spatial smoothness is edge-aware total variation on thickness,
ambient light and the nonlinearity. The Jerlov group is ramped entropy of the
soft water-type assignment, and the prior group's identity, ratio, curve and
magnitude terms penalise trivial solutions. A cosine warm-up over five steps
ramps four of the groups from a
5 % floor — the floor rather than zero, because a zeroed group multiplying a
NaN would silently mask a numerical fault — while the physics and cycle groups
are held at maximum throughout.

## 3.6 Test-time adaptation

At inference the meta checkpoint is loaded and the field parameters are reset to
the meta initialisation for each image. Twelve adaptation steps are taken at a
field learning rate of 3e-3 and an extractor learning rate of 3e-4 — the
extractor at 0.1 × the field rate — each step evaluating the reference-free loss
on the strided grid and projecting parameters back into bounds. The enhanced
image is rendered at stride 1 in a single pass. No gradient reaches the
benchmark's reference image, and if a ground truth exists it is never loaded by
this path. Adaptation costs 0.92–8.50 s per image depending on resolution.

**Display normalisation.** The physical inverse is deliberately conservative —
the t_min floor of §3.1.2 caps the amplification at 4× — so the rendered frame
keeps a compressed histogram: the water tint sits in the middle of the channel's
range instead of spanning it, and the output reads as barely different from the
input even where the radiance estimate has moved. Supervised competitors learn
this missing display transform from their training pairs. The pipeline adds it
as a final, reference-free stage: each channel's [$p_{1}$, $p_{99}$] percentile
range in the rendered frame is mapped linearly to [0, 1]. The stage touches the render
only — never the adaptation loss and never the re-degradation cycle — so the
reference-free objective, the physics and the meta-initialisation are unchanged,
and it carries no parameters. On the UIEB visual-check frames it is worth
+1.6 dB mean PSNR and moves frame contrast to the reference's own level; on the
full UIEB split it moves the split mean by the margin reported in §4.2.

## 3.7 Benchmark protocol

Five benchmark sets supply 519 test images. UIEB (Li et al., 2020),
LSUI (Peng et al., 2023) and EUVP (Islam et al., 2020) are paired and admit
full-reference metrics — PSNR, SSIM (Wang et al., 2004), MS-SSIM (Wang et al.,
2003) and LPIPS with the AlexNet and VGG backbones (Zhang et al., 2018) —
alongside the no-reference set. RUIE (Dalian University of Technology, n.d.),
which contains the UCCS colour-cast subset, ships no references and is scored
on no-reference metrics only: UCIQE (Yang and Sowmya, 2015), UIQM with its UICM,
UISM and UIConM components (Panetta et al., 2016) and an HSV
saturation-histogram colour-cast diagnostic implemented for this benchmark. The
diagnostic reports the fraction of pixels whose saturation is far from grey; a
high value means a single hue dominates the frame, the failure the enhancement
supposed to fix. It is a diagnostic rather than a quality measure and is not
attributed to any published metric. Benchmark and enhancement work published
in the ocean-engineering venue this paper targets follows the same two-track
convention: paired metrics where references exist, no-reference measures and
diagnostics where they do not. The examples span a decade --- the underwater
object-tracking benchmark of Panetta et al. (2022), the attenuated
colour-channel correction of Zhang et al. (2022) and the spatial–frequency
interaction transformer of Zhou et al. (2025).
UIEB is the only set with reference images used for ablation, so a removed loss
group can be judged on both full-reference and no-reference grounds. Every
dataset is verified against a sha256 manifest and every split is checked for
disjointness; UIEB's test split is a seeded 70/15/15 partition of its 890
paired images.

**Baselines.** Ten methods spanning classical priors (UDCP, Retinex), a GAN
(FUnIE-GAN) and supervised networks (Water-Net, UIEC²-Net, Five A+, TOPAL,
U-Shape Transformer, WaterMamba, WF-Diff) are scored by the same code path on
the same split with the same metric implementations. Nine run their published
models — seven from released network checkpoint files and two classical priors
from their reference implementations. WF-Diff's checkpoint is distributed only
through an authenticated regional file service and the authors' follow-up
repository ships none; it is scored through an explicitly labelled analytic
fallback, marked `[fallback]` in the tables and excluded from the ranking
discussion. Three baselines that were initially unavailable were recovered to
their published models during this study — UIEC²-Net, Five A+ and WaterMamba —
rather than left as approximations, which is recorded in the run metadata of
every column.

**Significance.** Scores are compared by paired Wilcoxon signed-rank against
PhysINR-Zero with Holm–Bonferroni correction within each metric family. A
comparison table is rendered only when its columns are paired, so an incomplete
run cannot produce a table that looks complete.

## 3.8 Implementation and reproducibility

The evaluation host is an Apple-Silicon workstation with 8 CPU cores and the
Metal Performance Shaders GPU backend; there is no CUDA. Two baselines required
host-specific accommodations, both recorded in their run metadata. WaterMamba's
WaterMamba's selective-scan kernel is CUDA-only and cannot build here, so the
scan runs through a chunked pure-PyTorch implementation of the same recurrence.
That is the recurrence the `mamba_ssm` authors' own reference implementation
checks the kernel against, agreeing with it to floating-point associativity.
Its channel projections are sized to the 256 × 256 resolution of the released
configuration, so larger frames are reconstructed from overlapping
cosine-windowed tiles.

Every run writes scores, per-method metadata, warnings and a manifest summary to
`results.json`; a `*.provenance.json` file records git commit, seed, device,
source images with sha256 hashes and the full configuration, so a reported run's
exact settings are recoverable from its artefacts. Figures and tables carry their
own provenance manifest mapping each artefact to the results file that produced
it. Per-(image, method) results are checkpointed and the runner is
`--resume`-capable, so an interrupted run continues rather than restarting.
