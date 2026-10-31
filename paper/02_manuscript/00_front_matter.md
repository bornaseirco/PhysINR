# Front matter

## Title

PhysINR-Zero: zero-shot underwater image enhancement with physics-constrained
coordinate fields, evaluated under a leakage-guarded re-benchmarking protocol

## Authors

Ali Hosseini (a,\*), Saeed Sedighi (a)

(a) Department of Computer Engineering, Shi.C., Islamic Azad University,
Shiraz, Iran

**Corresponding author:** Ali Hosseini, ali.hosseini7148@iau.ac.ir
**Emails:** ali.hosseini7148@iau.ac.ir; saeed.sedighi@iau.ac.ir

## Abstract

Underwater imaging supports civilian ocean engineering — inspection,
environmental monitoring, marine robotics — yet enhancement methods are hard to
compare, and the strongest supervised ones need paired training data a new site
cannot supply. We present PhysINR-Zero, an enhancement method that carries
no underwater training data, and evaluate it with HydroBench, a protocol that
re-runs every method under one implementation, one split and one metric set.
PhysINR-Zero represents the scene as a coordinate field of physical quantities —
per-channel optical thickness, a bounded nonlinear correction and ambient light —
inverting a Jerlov-based forward model. Its 10,296 parameters are meta-trained on
synthetic degradations of terrestrial photographs by first-order MAML, then
adapted to each image against reference-free losses, then rendered through a
reference-free display stage that restores the contrast the conservative
inverse leaves in the frame. Across 519 images from five benchmark sets against
ten baselines it trails the supervised leaders by 4.5–6.6 dB PSNR on the paired
sets, while ranking first of eleven on the no-reference UCIQE measure on four
of the five sets — and the two families disagree for a measurable reason:
UICM, the colour-measure term of UCIQE, rewards chroma spread, the very signal
a colour cast carries. The method delivers perceptual quality without signal
reconstruction, on metrics the field distrusts, and the paper quantifies where
that trust should end.

## Keywords

Underwater image enhancement; zero-shot learning; physics-informed neural
network; test-time adaptation; image quality assessment; benchmarking
