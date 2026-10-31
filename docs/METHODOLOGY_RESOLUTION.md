# PhysINR-Zero: methodology resolution

Every place the 23-section methodology specification fixes a quantity, the
code uses it. Every place it is *silent*, the code chooses a value. This
document records those choices so that a number in the manuscript can be
traced to a decision and a reason, and so that a reader who disagrees with a
choice can see exactly which one to vary.

The convention is `A<n>`: an assumption the implementation makes that the
methodology does not pin down. Each entry states the assumption, why this
value, and what would falsify it.

---

## A1 — `delta_max = 1.5` bounds the local departure from the Jerlov prior

**Where.** `PhysINRConfig.delta_max` (`physinr_zero/config.py`); used in
`PhysINR.forward` to form `tau = tau_jerlov + delta_max * tanh(q)`.

**Assumption.** The spatial optical-thickness field may deviate from the
water-type prior by at most 1.5 in optical thickness, per channel.

**Why this value.** The prior is `beta_c * d_bar`, a global triplet. Real
water columns are not uniform — slanted surfaces, suspended plumes, lighting
— so the field needs spatial freedom, but not so much that it can abandon the
physics. `delta_max` is expressed in the same units as `tau`, so it is
directly comparable to the prior's own scale: with `depth_max = 15` m and a
type-III water column the prior is around 1.8–4.5, so ±1.5 is a large but
not unbounded correction. The tanh keeps the deviation smooth and saturating.

**What would falsify it.** If the ablation over `delta_max ∈ {0.5, 1.5, 4.0}`
shows the metric optimum at a boundary, the value is not a choice but a
constraint being enforced by it, and the manuscript must say so.

---

## A2 — `alpha_max = 2.0`, `kappa_alpha = 0.25` bound the nonlinear correction

**Where.** `PhysINRConfig.alpha_max`, `kappa_alpha`; used by
`physics.bounded_nonlinear(v, alpha)`.

**Assumption.** The nonlinear intensity correction is a bounded monotone
remap of [0,1] onto [0,1] with slope at most ~`alpha_max` near the midpoint
and a soft-shoulder width set by `kappa_alpha`.

**Why these values.** The physical inverse recovers radiance that can lie
outside the displayable range; §8's correction compresses it back without
clipping. `alpha_max = 2.0` permits a doubling of local contrast at most —
enough to restore punch to a veiled image, not enough to invent contrast that
was not there. `kappa_alpha` controls where the shoulder begins; 0.25 puts it
near the extremes, leaving the midtones linear.

**What would falsify it.** If learned `alpha` saturates at `alpha_max` on a
large fraction of test images, the bound is active rather than merely safe,
and the correction is being limited rather than shaped.

---

## A3 — each camera channel is read at one representative wavelength

**Where.** `physics.RGB_WAVELENGTHS = {"r": 608, "g": 545, "b": 455}`,
`physics.jerlov_beta_matrix`.

**Assumption.** The attenuation coefficient of a camera channel can be
approximated by interpolating the Jerlov table at a single representative
wavelength rather than integrating the channel's spectral response against
the attenuation spectrum.

**Why this is an approximation, and why it is the honest one.** The Jerlov
table gives `K_d` at eight wavelengths. A real sRGB channel is a broad band,
and underwater attenuation varies steeply with wavelength, so a point sample
misrepresents a channel whose response straddles a strong gradient — the blue
channel in coastal water most of all.

The representative wavelengths are not arbitrary: they are the *peaks* of each
sRGB channel's spectral response, computed from the CIE 1931 2-degree
colour-matching functions and the IEC 61966-2-1 sRGB primaries under D65.
Reading `K_d` at the peak is the single-point estimator that a broadband
response weights most heavily.

Doing better requires a full spectral model (a 33-band or hyperspectral
radiative transfer calculation), which needs data this method deliberately
does not assume it has: it must work on ordinary sRGB imagery with no
auxiliary spectral measurement. **This is the largest acknowledged physical
approximation in the method and is stated as such in the paper.**

**What would falsify it.** A spectral-integrated `beta` that changes the
per-channel ratios enough to alter the soft water-type assignment on a
non-trivial fraction of images.

---

## A4 — the type-I row is corrected with Austin & Petzold (1986)

**Where.** `physics.AUSTIN_PETZOLD_TYPE_I`, `physics.USE_AUSTIN_PETZOLD_TYPE_I`.

**Assumption.** For Jerlov type I, the 1986 coastal-/oceanic attenuation data
of Austin & Petzold is a better single-point channel estimate than Jerlov's
own type-I row.

**Why.** Jerlov's published type-I values lie *below* pure-seawater
attenuation at 9 of 15 tabulated wavelengths, which is not physically
possible for a natural water body: type I is the clearest *oceanic* water,
not clearer than the pure sea water it contains. The Austin & Petzold curve
is consistent with the pure-water limit and is the standard reference for
clear oceanic `K_d`. The substitution is applied to the type-I row only; the
other nine rows are used as published.

**Edition warning (this caused a real error once).** The widely reproduced
`K_d0` table of Solonenko & Mobley is the *1968* edition of Jerlov, not the
1976 one, and individual entries differ by up to 34%. The two editions must
never be mixed. `physics.JERLOV_TABLE` is the 1976 edition (Table XXVII),
transcribed and cross-checked against two independent archived copies, and
`JERLOV_SOURCE` carries the citation.

**What would falsify it.** A primary source whose type-I row is internally
consistent with the pure-water limit and disagrees with Austin & Petzold.

---

## A5 — the outer meta-loss supervises against the synthetic clean image

**Where.** `MetaTrainer.train` (`physinr_zero/meta_train.py`).

**Assumption.** In meta-training, the outer loss may compare the adapted
output with the *synthetically* degraded source image, treated as the clean
target.

**Why this is legitimate.** No real reference image exists for the method's
operating regime — that is the problem being solved. The synthetic task is
generated *from* a clean photograph by a physically sampled degradation, so
the target is known by construction, not by annotation. The outer loss is
therefore supervised, but on data whose ground truth was manufactured rather
than collected. The inner loop remains reference-free (`L_total` only), which
is what the test-time procedure sees.

The distinction matters for the claims: the method is zero-shot with respect
to real underwater data, and meta-trained with synthetic supervision. Both
halves of that sentence are reported.

**What would falsify it.** Nothing about the code; the concern is a
mismatch between the synthetic degradation family and real underwater
images. That is tested empirically — the benchmark is the test — and the
risk is stated in the paper's limitations.

---

## A6 — the re-degradation residual is coupled to the thickness deviation

**Where.** `PhysINR.forward`, `residual = residual_max * tanh(q)`.

**Assumption.** The residual `r~_c` of the §9 re-degradation operator is
`q_c`, the same pre-activation that drives the local departure from the
Jerlov prior, scaled by `residual_max = 0.05`.

**Why.** §9 specifies a residual on the re-degradation operator but §4
allocates exactly ten field outputs (3 + 3 + 3 + 1), all of which are already
accounted for by `tau`, `alpha`, `ambient` and the water-type logit. There is
no free output slot for an independent residual, so either the output
dimensionality must grow — breaking the §4 parameter budget the method is
defined by — or the residual must be coupled to an existing quantity.

The coupling is not arbitrary. The residual exists to absorb discrepancies
the physical model cannot explain, and those discrepancies are largest
exactly where the optical-thickness field is being pulled away from the
Jerlov prior — the same condition `q` encodes. Coupling them costs no
parameters and gives the residual a physically motivated spatial structure
rather than an independent noise field the network would have to learn from
scratch.

**What would falsify it.** An independent residual field that improves
metrics enough to justify its parameters. That is a reportable ablation, not
a correction.

---

## A7 — adaptation evaluates the field on a strided coordinate grid

**Where.** `PhysINRConfig.loss_stride` (default 2),
`PhysINR.forward(coord_stride=...)`, `PhysINR._grid`, `strided_image`,
`TestTimeAdapter.adapt`, `MetaTrainer.train`.

**Assumption.** During adaptation the field may be evaluated at every
`loss_stride`-th pixel; the enhanced image is rendered at stride 1.

**Why this is a faithful implementation, not a shortcut.** §21 step 4a of the
inference procedure reads "sample normalised image coordinates `x`". It does
not say "evaluate at every pixel". Two properties make the subsampling an
approximation *in sampling* rather than a different model:

1. `PhysINR._grid` assigns pixel `i * stride` the coordinate
   `-1 + 2*i*stride/(H-1)` — the identical coordinate the stride-1 grid would
   have given that pixel. The strided forward is therefore the restriction of
   one continuous field to a subset of the pixels.
2. The local feature field is resampled to the full grid once and then
   *indexed* at the sampled coordinates (`_sample` returns `(B, H, W, C)`),
   so `f(x)` is a function of position. Resampling the feature map *to the
   smaller grid* would define a different `f(x)` and break this; the test
   suite pins it (`test_model.py::TestCoordinateSampling`).

The residual discrepancy on shared pixels is ~1e-5 in `tau`, and comes from
the two *global* poolings (the water-type logit mean and the ambient offset)
being estimated from the sample rather than the full pixel set.

**Measured cost** (MPS, 857×1280 image, 12 adaptation steps):

| stride | per step | per image | speedup |
|--------|----------|-----------|---------|
| 1      | 1.78 s   | 21.3 s    | 1.0×    |
| 2      | 0.63 s   | 7.6 s     | 2.8×    |
| 3      | 0.41 s   | 5.0 s     | 4.3×    |

The final full-grid render costs 0.22 s regardless of stride. The default is
2, chosen as the largest speedup that leaves the metrics within noise of
stride 1 (reported in the stride ablation, not assumed).

**What would falsify it.** The stride ablation: if stride 2 changes any
headline metric by more than the seed-to-seed variance at stride 1, the
default must return to 1 and the runtime cost accepted.

---

## A8 — the meta-training source is clean terrestrial photography

**Where.** `physinr_zero/cli.py` (`_assert_source_is_clean`,
`_evaluation_datasets`), `MetaTrainer`, the `datasets/clean_photos` pool.

**Assumption.** The clean images used to build synthetic tasks are
terrestrial/natural photographs containing no underwater content.

**Why this is enforced rather than assumed.** The most efficient way to
destroy the zero-shot claim is also the most tempting shortcut: point
meta-training at a benchmark dataset's *reference* directory, which is the
cleanest imagery in the repository. Those reference images are real
underwater photographs, so using them falsifies the claim outright.

The CLI therefore refuses any `--source` that resolves inside a HydroBench
dataset carrying a test split. The list of such datasets is derived from the
committed manifests, not hardcoded, so a newly added dataset cannot leak
through a list nobody updated. A `--allow-eval-source` escape hatch exists for
ablations that specifically study leakage, and writes an `OVERRIDE` line to
the log and the provenance; when it is used, the zero-shot claim must be
dropped for that checkpoint.

The pool used for the reported results is DIV2K (Agustsson & Timofte, NTIRE
2017): 800 natural 2K photographs. Its provenance — dataset, URL, image
count, sha256 — is recorded per run.

**What would falsify it.** Any evaluation image appearing in the
meta-training pool. The `deduplicate` pass guards this with a perceptual-hash
check against every evaluation test split, and the counts (kept/dropped) are
written to the run's provenance record.

---

## Ablation inventory

The assumptions above fall into two classes, and the manuscript treats them
differently:

- **Choices that the methodology leaves open** (A1, A2, A6, A7) — each is a
  knob and each gets an ablation row. A choice that turns out to be at a
  boundary of the swept range is reported as a constraint, not a result.
- **Structural facts that follow from the specification** (A3, A4, A5, A8) —
  these are not knobs. They are stated in the methods and the limitations,
  with the physical reasoning and the risk attached.

Every ablation reuses the same evaluation pipeline, the same metrics and the
same seeds as the main comparison, so a number in an ablation table is
comparable to the number next to it in the main table.
