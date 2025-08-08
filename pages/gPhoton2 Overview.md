# gPhoton Pipeline Overview

## gPhoton1

The original gPhoton software (henceforth referred to as _gPhoton1_) calculated photometry by performing spacio-temporal queries on a database of ~1.1 trillion timestamped and sky-mapped photon events.

The gPhoton1 paper [Million, et. al 2016](https://doi.org/10.3847/1538-4357/833/2/292)

## gPhoton2

Perhaps the most significant change in gPhoton2 is a that the pipeline now generates images a per-observation basis and performs photometric measurements on these images. This is much more similar to how the original mission performed these measurements.[^myref]

[^myref]: It is also much more similar to how photometry is _generally_ measured on astronomical images.