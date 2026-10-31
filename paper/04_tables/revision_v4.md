<!-- Prior design study: four candidate transmission priors on the
     same 20 UIEB test frames and 12 UCCS colour-cast frames, 480 px
     max side, 12 adaptation steps, one protocol for every column.
     Clipping is the fraction of pixels within 1/255 of either rail,
     averaged over the three channels; clip (G) is the green channel
     alone, the one the water-type-lookup prior saturates. Cast is
     the CIELAB chroma magnitude. The published checkpoint runs under
     its stored configuration (legacy); the intermediate row is the
     content-dependent prior that preceded the measured-level fix;
     B/C/D are the three priors of results §4.5. The adaptation loop
     is chaotic at the metric margin (±0.02 clipping, ±0.35 dB PSNR
     across repeat runs), so only larger differences are
     interpretable. -->

| Variant | clip (all) | clip (G) | cast | PSNR | SSIM | UCIQE |
| --- |---|---|---|---|---|---|
| **UIEB, n = 20** | | | | | | |
| **raw input** | 0.012 | 0.003 | 13.814 | 16.179 | 0.7229 | 19.557 |
| A: water-type lookup | 0.113 | 0.165 | 19.602 | 15.444 | 0.7066 | 29.108 |
| intermediate: content head | 0.012 | 0.003 | 10.483 | 16.323 | 0.7151 | 20.508 |
| B: measured only | 0.007 | 0.003 | 11.445 | 16.178 | 0.7263 | 20.071 |
| C: + level, uncalibrated | 0.105 | 0.075 | 18.813 | 14.579 | 0.6592 | 28.494 |
| **D: + level, calibrated** | 0.064 | 0.060 | 10.548 | 15.348 | 0.7781 | 25.483 |
| **UCCS, n = 12** | | | | | | |
| **raw input** | 0.002 | 0.000 | 35.185 | --- | --- | 15.599 |
| A: water-type lookup | 0.108 | 0.261 | 43.076 | --- | --- | 25.532 |
| intermediate: content head | 0.000 | 0.000 | 26.815 | --- | --- | 16.385 |
| B: measured only | 0.000 | 0.000 | 30.997 | --- | --- | 16.137 |
| C: + level, uncalibrated | 0.023 | 0.023 | 34.525 | --- | --- | 24.875 |
| **D: + level, calibrated** | 0.004 | 0.004 | 31.185 | --- | --- | 21.842 |
