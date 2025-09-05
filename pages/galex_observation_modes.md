---
title: Observation Modes
site:
  hide_outline: true
  hide_toc: true
---

# Observation Modes

The “observation mode” or “plan type” for each {term}`eclipse` identifies the
general type of observation that was carried out for that eclipse.
These roughly correspond to the different classes of “science survey”
described in [chapter 2 of the official GALEX technical documentation][galex-tech-ch2].

Observation modes are divided into two classes: imaging and
spectroscopic.  gPhoton 2 currently does not support processing of
the data collected in spectroscopic modes.

[galex-tech-ch2]: http://www.galex.caltech.edu/researcher/techdoc-ch2.html

## Imaging modes

| Mode | Description | Typical Exposure Time | Typical Depth (AB mag) | Notes |
|------|-------------|-----------------------|------------------------|-------|
| AIS  | All-Sky Imaging Survey     | ~140 sec   | ~20.5  | Typically 12 {term}`leg`s per eclipse. |
| MIS  | Medium Imaging Survey      | ~1500 sec  | ~23    | Typically one {term}`leg` per eclipse. |
| DIS  | Deep Imaging Survey        | ~30000 sec | ~25    | Repeated MIS-like or AIS-like visits to the same field |
| NGS  | Nearby Galaxy Survey       | 1000–1500 sec | Varies | Targeted observations of nearby galaxies |
| CAI  | Calibration Imaging        | ~140 sec   | ?      | Images of stars with known ultraviolet brightness |
| DTI  | ?                          | ~650 sec   | ?      |  ?  |
| ETI  | ?                          | Varies     | ?      |  ?  |
| GII  | Guest Investigator Imaging | Varies     | Varies | Guest investigator programs |
| UIS  | Ultra Imaging Survey       | —         | —     | Targeted deep imaging (never actually used) |

## Spectroscopic modes

| Mode | Description | Notes |
|------|-------------|-------|
| WSS  | Wide Spectroscopic Survey        | Spectroscopic counterpart of AIS |
| MSS  | Medium Spectroscopic Survey      | Spectroscopic counterpart of MIS |
| DSS  | Deep Spectroscopic Survey        | Spectroscopic counterpart of DIS |
| CAS  | Calibration Spectroscopy         | Spectroscopic counterpart of CAI |
| ETS  | ?                                | Spectroscopic counterpart of ETI ? |
| GIS  | Guest Investigator Spectroscopic | Guest investigator programs |

## Observation subtypes

The “observation subtype” or “plan subtype” for each eclipse gives a
little bit more detail about what kind of observation was carried out
during that eclipse.

:::{list-table}
:align: center
:header-rows: 1

* - Subtype
  - Used with modes
  - Description
* - ais
  - AIS
  - All AIS eclipses have subtype “ais”.
* - cal
  - CAI CAS MIS
  - Calibration-related observations?
* - deep
  - DIS DSS MSS WSS
  - Targets selected originally for DIS?
* - dto
  - DTI MIS
  - ???
* - etc
  - ETI ETS MIS
  - “etcetera”?
* - gip
  - GII GIS
  - Targets selected via Guest Investigator Program?
* - ioc
  - MIS NGS
  - ???
* - mis
  - MIS
  - Majority of MIS eclipses have subtype “mis”.
* - ngs
  - MIS NGS
  - Targets selected originally for NGS?
:::


## GALEX standardized aperture radii

| Index | Aperture Radius | Note |
|-------|-----------------|------|
| 1     | 1.5" | |
| 2     | 2.3" | Minimum recommended for scientific analyses. |
| 3     | 3.8" | |
| **4** | **6.0"** | **Recommended as best general purpose aperture.** |
| 5     | 9.0" | |
| 6     | 12.8" | |
| 7     | 17.3" | Maximum recommended for most scientific analyses. |
| 8     | 30.0" | Useful outer background annuli radius in uncrowded fields. |
| 9     | 60.0" | Not recommended except for unusual calibration needs. |
| 10     | 90.0" | Not recommended except for unusual calibration needs. |
