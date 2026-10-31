<!-- Revision v2 audit: the transmission-prior failure and its fix. 20 UIEB
test frames and 12 UCCS colour-cast frames, 480 px max side, 12 adaptation
steps, one protocol for every column. "Clipping" is the fraction of pixels
within 1/255 of either rail, averaged over the three channels; clip(G) is the
green channel alone, the one v1 saturates. The published checkpoint runs
under its stored configuration (legacy); "published+floor" enables only the
t_min = 0.25 transmission floor; "typehead" is the intermediate
content-dependent-prior checkpoint; v2 is the revision retrain (measured
level + floor + DC-free learned component). The adaptation loop is chaotic at
the metric margin -- repeat runs of the same column vary by about +-0.02 in
clipping and +-0.35 dB in PSNR -- so only differences larger than that are
 interpretable. -->

| Variant | clip (all) | clip (G) | PSNR | SSIM | cast (Lab) | UCIQE |
| --- | --- | --- | --- | --- | --- | --- |
| **UIEB, n = 20** | | | | | | |
| raw input | 0.012 | 0.003 | 16.18 | 0.7229 | 13.81 | 19.56 |
| v1 published | 0.112 | 0.169 | 15.38 | 0.7052 | 20.20 | 29.28 |
| v1 + floor only | 0.111 | 0.167 | 15.37 | 0.7050 | 20.55 | 29.30 |
| typehead (intermediate) | 0.013 | 0.003 | 16.31 | 0.7159 | 10.48 | 20.57 |
| **v2 (revision)** | **0.007** | **0.003** | 16.15 | **0.7243** | 11.42 | 20.04 |
| **UCCS, n = 12** | | | | | | |
| raw input | 0.002 | — | — | — | 35.19 | 15.60 |
| v1 published | 0.108 | — | — | — | 43.08 | 25.53 |
| v1 + floor only | 0.108 | — | — | — | 43.08 | 25.53 |
| typehead (intermediate) | 0.000 | — | — | — | 26.81 | 16.38 |
| **v2 (revision)** | **0.000** | — | — | — | 31.00 | 16.14 |
