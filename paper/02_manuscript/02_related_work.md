# 2. Related work

## 2.1 Supervised underwater image enhancement

The supervised family learns the mapping from degraded to clean images from
paired data. Water-Net (Li et al., 2020) introduced it together with the UIEB
dataset, using a colour-estimation module and an attention-driven
colour-correction network. FUnIE-GAN (Islam et al., 2020) targeted real-time
deployment on robots with a conditional GAN and its own EUVP dataset. U-Shape
Transformer (Peng et al., 2023) introduced the LSUI dataset and a
channel-attention U-Net variant. UIEC²-Net (Wang et al., 2021) fuses features
in RGB and LAB. Five A+ (Jiang et al., 2023) demonstrated that 9K parameters
suffice for competitive enhancement, which matters for embedded ocean
platforms. TOPAL (Jiang et al., 2022) uses a target-oriented perceptual
adversarial fusion. Diffusion entered the field with WF-Diff (Zhao et al.,
2024), and the state-space models followed with WaterMamba (Guan et al., 2024).
Recent methods continue upward in reported accuracy: UniUIR (Zhang et al.,
2025) reports 25.11 dB PSNR on UIEB, WWE-UIE (Cheng et al., 2026) 24.32 dB
and Phaseformer (Khan et al., 2025) 25.98 dB. Each reports under splits that
none of them share, which is the comparability problem of §1 quantified at the
top of the reporting distribution.

## 2.2 Physics-based and zero-shot methods

The classical alternatives assume no training data. Retinex theory (Jobson et
al., 1997) decomposes the image into illumination and reflectance and rescales
the former; multi-scale Retinex with colour restoration remains a standard
reference point. The underwater dark channel prior (Drews Jr. et al., 2013)
adapts dark-channel dehazing to wavelength-selective attenuation. Both
generalise to any water body and both rank near the bottom of the paired-set
metrics (§4.2, [Tables](tab:uieb|tab:lsui|tab:euvp)) — Retinex reaches 15.099 dB PSNR on UIEB against
23.654 for UIEC²-Net — while Retinex' strong UCIQE shows that reconstructing the
reference and scoring well perceptually are different tasks.

PhysINR-Zero sits between the families. It is trained, but on synthetic
degradations of terrestrial photographs, so the zero-shot boundary is with
respect to real underwater data rather than with respect to training per se.
Two design choices separate it from the classical priors: the water-type prior
is *soft* — the field emits a distribution over the ten Jerlov types rather
than committing to one — and the adaptation is per-image, which is what lets a
single initialisation serve turbid coastal water and clear oceanic water.

## 2.3 Coordinate networks and test-time adaptation

Coordinate neural networks represent a scene as a function of position. In
low-level vision they are used mostly for implicit scene representation; the
application here is different — the coordinate field carries *physical*
quantities sampled at the image grid, not colour. Test-time adaptation through
meta-learning follows first-order MAML (Finn et al., 2017), which trades the second-order gradient
of full MAML for a first-order approximation at a substantial runtime saving.

## 2.4 Benchmark and protocol work

The comparability problem has produced two kinds of response. The first is the
re-run: Cong et al. (2024) re-ran seventeen methods under one protocol "to
prevent differences in metric values caused by different toolboxes or
implementations", unifying the split ratio and the metric code; Guo et al.
(2025) re-benchmarked thirty-seven methods on a mixed test pool. The second is
the metric critique. Li and Cavallaro (2022) conclude that "none of the
no-reference quality measures satisfactorily rates the quality of enhanced
underwater images", analysing UIQM, UCIQE and CCF directly. Yang and Sowmya
(2015) — whose UCIQE definition this paper uses — and Panetta et al. (2016) —
whose UIQM definition it uses — supplied the measures that later work came to
dispute. WWE-UIE (Cheng et al., 2026) drops UIQM in favour of NIQE. The
colour-cast diagnostic this benchmark reports alongside them is an internal
HSV saturation measure, not one of the published measures under critique.
Most
directly, the UIEB benchmark paper itself states "We discard the non-reference
metrics designed for underwater image enhancement" (Li et al., 2020, §IV).
HydroBench follows the re-benchmarking response and inherits the critique: it
implements both families of metric with stated conventions, and the
no-reference orderings in §4.2 are read with the field's own reservations.
