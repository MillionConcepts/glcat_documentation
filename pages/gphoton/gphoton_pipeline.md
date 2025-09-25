# gPhoton Pipeline Overview

gPhoton is a software package that enables flexible and complex analyses of data from the Galaxy Evolution Explorer (GALEX) mission. This software reproduces many of the capabilities of the original mission data processing pipeline and also adds some new ones.

Although a snapshot copy of the GALEX mission data processing pipeline still exists, it is not runnable on any existing modern infrastructure.[^mission_pipeline]

[^mission_pipeline]: The mission pipeline was initially developed >20 years ago to operate on a specific internal network at Caltech, with custom configurations to enable the use of a custom distributed compute task and data management system. Reproducibility or reuse of the software were not objectives of this development.

## gPhoton1

The original gPhoton software (henceforth referred to as _gPhoton1_) calculated photometry by performing spacio-temporal queries on a database of ~1.1 trillion timestamped and sky-mapped photon events.

The [_gPhoton1_ paper](https://doi.org/10.3847/1538-4357/833/2/292) provides a thorough overview. There is also quite a lot of useful information in the [_gPhoton1_ user guide](https://github.com/cmillion/gPhoton/blob/master/docs/UserGuide.md).

The _gPhoton1_ software is no longer actively maintained. As of writing, the _gPhoton1_ software is still mostly usable, and MAST continues to host and maintain the external API for the database of calibrated photon events. However, **we no longer recommend that people use _gPhoton1_**. We now recommend that all investigators use _gPhoton2_, which is described below.

## gPhoton2

Perhaps the most significant change in gPhoton2 is a that the pipeline now generates images a per-observation basis and performs photometric measurements on these images. This is much more similar to how the original mission performed these measurements.[^myref]

The GFCAT paper [](10.3847/1538-4365/ace717)

[^myref]: It is also much more similar to how photometry is _generally_ measured on astronomical images.