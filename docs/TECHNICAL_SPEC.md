# PhysINR-Zero: technical specification

This is the implementation contract behind the manuscript. It maps the
methodology's 23 sections to modules, states every I/O boundary, and lists the
hyperparameters with their defaults. Anything a reader needs to reproduce a
number should be findable from here.

Assumptions where the methodology is silent are in
`docs/METHODOLOGY_RESOLUTION.md` (A1–A8); they are referenced here, not
restated.

---

## 1. Pipeline at a glance

```
clean terrestrial photos ──► synthesize ──► synthetic task (I_synth, params)
        (DIV2K)                  (§7)              │
                                                  ▼
                                     MetaTrainer.train   ◄── FOMAML
                                     (§20, meta-tasks)      inner: L_total
                                                  │        outer: vs clean
                                                  ▼
                                     weights/physinr_zero_meta.pt
                                                  │
   test image ────────────────────────────────────┐│
                                                  ▼▼
                                     TestTimeAdapter.adapt   ──► enhanced image
                                     (§21, reference-free)         + trace JSON
```

Zero-shot boundary: the arrow from DIV2K to the checkpoint is the **only** path
by which learned weights reach test time. No underwater image of any benchmark
dataset crosses it (enforced, not assumed — see A8).

---

## 2. Module map

| Module | Responsibility | Key I/O |
|---|---|---|
| `physinr_zero/config.py` | every hyperparameter, serialised into the checkpoint | `PhysINRConfig` → `to_dict()` / `PhysINRConfig(**d)` |
| `physics.py` | image formation, Jerlov table, bounded parameterisations | `jerlov_beta_matrix`, `transmission`, `regrade`, `inverse_degrade`, `bounded_nonlinear` |
| `model.py` | feature extractor + coordinate field; the forward physics pass | `PhysINR.forward(image, coord_stride)` → `PhysINROutput` |
| `synthesize.py` | synthetic degradation sampling for meta-training | `SyntheticDegradation.sample(clean)` → degraded image + true parameters |
| `losses.py` | the seven loss groups and the warm-up schedule | `LossComputer.__call__(x, out, y_clean=None, weights=None)` → `(total, terms)` |
| `meta_train.py` | FOMAML outer loop over synthetic tasks | `MetaTrainer.train(tasks, out_path)` → checkpoint + `MetaRecord` |
| `adapt.py` | per-image test-time adaptation and reset | `TestTimeAdapter.adapt(image, steps)` → `(enhanced, AdaptTrace)` |
| `cli.py` | `meta-train` / `enhance` / `smoke` / `info`; the source guard | exit codes: 0 ok, 2 bad input, 3 run failure |

HydroBench integration:

| Artifact | Role |
|---|---|
| `hydro_bench/methods/deep/physinr_zero/adapter.py` | `PhysINRZero(BaseAlgorithm)`; `training_sets=()` by design |
| `hydro_bench/configs/methods.yaml` | registered `mode: online`, `runner: process` |
| `experiments/run_physinr_zero.sh` | 4-stage orchestration (meta-train → comparison → 2 ablation sweeps) |
| `experiments/merge_results.py` | merges per-variant results into one table per dataset |
| `experiments/make_figures.py` | figures + tables + per-artefact provenance |

---

## 3. Algorithmic steps

### 3.1 Meta-training (`§20`)

```
preload clean images, max side 2*meta_crop (384)   # 900 cand. -> 898 dedup -> 882 usable
for t in 1..meta_tasks (4000):
    sample meta_batch=4 crops of meta_crop=192
    for each crop: sample a synthetic degradation (§3.3)  -> (I_synth, truth)
    inner loop (4 steps, lr 0.01): adapt field params on L_total(I_synth)
    outer: L_outer = ||regrade(adapted field) - clean crop||
    meta step on the *pre-adaptation* parameters (FOMAML first-order)
project parameters back into bounds; save state_dict + config + record
```

Outer loss uses the *clean* crop as target — synthetic supervision, A5. The
inner loop sees `L_total` only, which is exactly what test time sees.

### 3.2 Test-time adaptation (`§21`)

```
load meta checkpoint; reset per-image parameters to the meta initialisation
x = image / 255
for s in 1..adapt_steps (12):
    out = model(x, coord_stride = loss_stride)     # A7
    L, terms = losses(x, out)                      # no ground truth used
    step: field lr 3e-3, extractor lr 3e-4         # extractor = 0.1x field
    project into bounds
render at coord_stride = 1                        # full grid, one pass
parameters reset for the next image
```

No gradient reaches the benchmark's reference image; if a ground truth exists it
is never loaded by this path.

### 3.3 Synthetic degradation sampling (`§7`)

Per task, sampled uniformly and independently:

| Quantity | Range |
|---|---|
| scene depth `d` | 0.5–12.0 m |
| water type | softmax over the 10 Jerlov types, resampled per task |
| per-channel β jitter | ±15% of the type's β |
| local τ perturbation | up to 0.35 (spatial structure) |
| ambient light level | 0.15–0.85 |
| ambient colour shift | ±0.1 per channel |
| blur σ | 0.0–1.2 (separable Gaussian) |
| noise σ | 0.0–0.04 |

`I_synth = regrade(clean; τ, A, α, residual)` — the same forward operator the
model inverts, so the task distribution and the model's inductive bias are
matched by construction. This is a deliberate choice: it makes the synthetic
family self-consistent, and it is also the reason the benchmark, not
meta-training loss, is the test that matters (A5).

---

## 4. The physics forward pass (`§2`, `§4`, `§8`, `§9`)

```
features f(x)      : 3-block depthwise-separable conv, 16/24/32 channels,
                     total downsampling ≤4x, LeakyReLU(0.1)
field outputs (10) : tau_c (x3), alpha_c (x3), ambient_c (x3), water logit (x1)
bounded as         : tau  = tau_jerlov + delta_max * tanh(q)      in [0, tau_max]
                     alpha = alpha_max * softplus-bound,          in [0, alpha_max]
                     ambient = ambient_min + kappa_a * softplus   in [ambient_min, ambient_max]
                     water = softmax over 10 Jerlov types
tau_jerlov         : beta_c(type) * depth_field,  depth in [0, depth_max]
t_c                = exp(-tau_c)
J_c                = bounded_nonlinear(I_c, alpha_c)   (§8, g(v)=v(1-v)(2v-1))
regrade            : I' = J * t + A * (1 - t) + residual_max * tanh(q)
inverse (decode)   : J_hat = (I - A_hat*(1-t_hat)) / (t_hat + eps)
```

Parameter budget: **< 25,000** total (measured 9,813 at default), enforced by
`param_budget` and asserted in the test suite.

---

## 5. Loss groups (`§10`–`§16`)

Seven groups, each a documented weight in the config; a group is ablated by
zeroing its base weight (`drop_group`, §22). Composition is as in
`PhysINRLoss.__call__`; the term weights below are the config's inner weights
and the group weight multiplies the whole group.

| Group | Group weight | Composition |
|---|---|---|
| phys | `w_phys=1.0` | `recon + w_range*range` (Charbonnier on the re-degraded image, plus a τ range penalty) |
| cycle | `w_cycle=0.5` | `char + w_grad*grad` (re-degradation round-trip, image + gradient) |
| color | `w_color=0.5` | `mean + w_std*std` over CIELAB a/b (D65, BT.709), with a transmission-masked chroma prior |
| struct | `w_struct=0.5` | `w_ssim*ssim + w_edge*edge + w_entropy*entropy + w_noise*noise` |
| spatial | `w_spatial=0.05` | `tv_tau + tv_ambient + w_alpha_tv*tv_alpha + w_curv*curv` (all edge-aware) |
| jerlov | `w_jerlov=0.3` | `base + w_dev*dev + ww*water` (entropy of the soft water-type assignment, ramped) |
| prior | `w_prior=0.2` | `identity + w_ratio*ratio + w_curve*curve + w_magnitude*magnitude` (anti-trivial, §17) |

Warm-up (`§20`): a cosine ramp over `warmup_steps=5` applies to the four groups
`_SCHEDULED = (color, struct, spatial, jerlov)`; `phys` and `cycle` are held at
maximum. The ramp starts at a 5% floor rather than zero, because `0 * NaN = NaN`
would silently mask a numerical fault. The *total* therefore rises as groups
switch on, which is expected, and is why ablations compare checkpoints under a
fixed weight vector via the `weights=` override.

---

## 6. Hyperparameters (defaults)

Reproduced from `PhysINRConfig`; the authoritative list is the code. Every value
here is written into the checkpoint, so a reported run's exact configuration is
recoverable from its `.provenance.json`.

**Field / training.** `fourier_levels=4`, `fourier_ramp=4`, `mlp_hidden=64`,
`feature_channels=(16,24,32)`, `feature_strides=(2,2,1)`, `leaky_slope=0.1`,
`tau_max=8.0`, `delta_max=1.5` (A1), `alpha_max=2.0`, `kappa_alpha=0.25` (A2),
`ambient_min=0.1`, `ambient_max=0.9`, `kappa_ambient=0.05`, `residual_max=0.05`
(A6), `depth_max=15.0`, `num_water_types=10`, `param_budget=25000`.

**Adaptation.** `adapt_steps=12`, `lr_field=3e-3`, `lr_extractor=3e-4`,
`loss_stride=2` (A7), `eps=1e-4`, `max_side=1024`.

**Meta-training.** `meta_tasks=4000`, `meta_batch=4`, `meta_inner_steps=4`,
`meta_inner_lr=0.01`, `meta_outer_lr=1e-3`, `meta_crop=192`, `seed=1234`.

**Synthesis.** as §3.3.

---

## 7. Evaluation procedure

| Set | Split | Metrics | Why |
|---|---|---|---|
| UIEB | seeded 70/15/15 test | PSNR, SSIM, MS-SSIM, LPIPS(Alex/VGG) + all NR | paired; ablation substrate |
| LSUI | test | FR + NR | paired |
| EUVP | test | FR + NR | paired |
| RUIE | test | NR only | unpaired |
| UCCS | test | NR only | unpaired |

Ablations run on UIEB (40 images) — it is the set with reference images, so a
dropped loss group can be judged on both FR and NR.

Per-(image, method) results are checkpointed (`partial.json`, `--resume`), and
every variant writes to its own output directory because the runner overwrites
`results.json` on completion. Baselines are never re-run; their files are copied
unchanged by the merge, which records the source file of every column.

Significance: paired Wilcoxon signed-rank against the chosen baseline, Holm
corrected; a table is only rendered when columns are paired, so an incomplete
run cannot produce a table that looks complete.

---

## 8. Ablation inventory (`§22`)

| Knob | Values | Tests |
|---|---|---|
| loss groups | phys, cycle, color, struct, spatial, jerlov, prior | whether any group is load-bearing |
| `loss_stride` | 1, 2, 3 | A7: is the stride an approximation or a model change? |
| `adapt_steps` | 5, 12, 20 | where adaptation saturates |
| `tau_max` | 2, 4, 8 | A1-adjacent: is the bound active? |
| `delta_max` | 0.5, 1.5, 4.0 | A1: is the deviation bound active? |

An optimum at a sweep boundary is reported as a *constraint*, not a result
(A1, A2).

---

## 9. Reproducibility artefacts

Every run writes, alongside its outputs:

- `results.json` — scores, per-method metadata, warnings, manifest summary;
- `*.provenance.json` — git commit, seed, device, source images, sha256, config,
  and for meta-training the dedup record and `tasks_completed`;
- per-image adaptation traces (`--trace`) — loss terms per step, for the
  convergence figure;
- `figures_provenance.json` — which results file backs each figure and table.

The placeholder guard (`adapter._build`) refuses a checkpoint with
`tasks == 0`, so a plumbing checkpoint can never produce reported numbers.
