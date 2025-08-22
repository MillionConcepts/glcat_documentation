---
title: The Aggregated Aspect and Metadata Tables
site:
  hide_outline: true
---

# The Aggregated Aspect and Metadata Tables

gPhoton 2 relies on four large tables of precalculated data.
These contain information about each
[{abbr}`GALEX (Galexy Evolution Explorer)`][GALEX] eclipse[^eclipse]
which depends only on how the GALEX satellite was programmed for that
eclipse (this is the “metadata”) and what part of the sky it was
actually pointed at during each part of the eclipse (the “aspect
data”).  Everything in these tables can, in principle, be recalculated
from the [GALEX archive][] on
[{abbr}`MAST (Mikulski Archive for Space Telescopes)`][MAST];
however, this is an expensive process and there is little scientific
value in repeating it.

The aspect and metadata tables are not distributed along with the
gPhoton source code. [They are currently downloadable from Google
Drive][gdrive-aspect], but may be moved to MAST, or another more
convenient location, in the future.

[^eclipse]: The GALEX satellite was in a medium-altitude orbit of
    Earth.  Because it was designed to observe very faint stars and
    galaxies, it could only collect data during the part of each orbit
    that passed within Earth’s shadow.  Each such period of active
    observation is referred to as an eclipse.


[GALEX]: http://www.galex.caltech.edu/
[GALEX archive]: https://archive.stsci.edu/missions-and-data/galex
[MAST]: https://archive.stsci.edu/
[gdrive-aspect]: https://drive.google.com/drive/u/1/folders/1aPfLKsZM8x5Pxji0Lh3dUblpo9dyt1IW

## Contents of the aggregated metadata tables

There are four files of aggregated metadata; each contains a single
table in [Apache Parquet][] format.

All four tables contain celestial coordinates; these are uniformly
equatorial (J2000), with values in decimal degrees.  Declination
ranges from −90° to 90°.  Right ascension _normally_ ranges from
−180° to 180°, but values greater than 180° can appear in a few
places, such as the `ra_max` column of `metadata.parquet` and
`boresight.parquet`.

[Apache Parquet]: https://parquet.apache.org/

(metadata)=
### `metadata.parquet`

This table contains one row for every eclipse that has _something_
archived on MAST, whether or not gPhoton 2 can currently process
the type of data that GALEX collected during that eclipse.  Notably,
spectroscopic “grism” observations are currently not supported, but
appear in this table anyway.  The `has_aspect` column of this table
(see below) is true for all eclipses that gPhoton 2 supports, and
false for those that it doesn’t.  When `has_aspect` is false, several
other columns of `metadata.parquet` will be null, and the eclipse
in question will be completely absent from the other three tables.

Most of the data in this table is aggregated from the
“{abbr}`SCST (spacecraft state)`” files on MAST.
Some columns contain summary data
from the [heuristic leg recalculation](/glcat-leg-recalc)
that was performed for GLCAT.

:::{list-table} Columns of `metadata.parquet`
:label: metadata-schema
:align: center
:header-rows: 1

* - Column
  - Type
  - Source
  - Contents
* - `eclipse`
  - integer
  - scst file header, `ECLIPSE`
  - Serial number of the eclipse.
* - `plan_type`
  - string
  - scst file header, `MPSTYPE`
  - [Observation mode](/galex-observation-modes) for this eclipse.
* - `plan_subtype`
  - string
  - scst file header, `MPSPLAN`
  - [Observation subtype](/galex-observation-modes#observation-subtypes)
    for this eclipse.
* - `visit`
  - integer
  - scst file header, `VISIT`
  - Visit code for this eclipse (needs elaboration).
* - `plan_id`
  - integer
  - scst file header, `PLANID`
  - Code for the specific observation plan for this eclipse
    (needs elaboration).
* - `planned_legs`
  - integer
  - scst file header, `MPSNPOS`
  - Number of _planned_ observation targets for this eclipse.
    Always at least 1; in practice, never higher than 12.
* - `observed_legs`
  - integer[^null-unsupport]
  - Leg recalculation
  - Number of distinct targets actually observed.
    This is _usually_ equal to `planned_legs` but can be smaller or bigger.
* - `has_aspect`
  - boolean
  - `aspect.parquet`
  - True if gPhoton 2 supports processing this eclipse’s data.
* - `eclipse_start`
  - float
  - scst file header, `TRANGE0`
  - Start time for this eclipse, as seconds since the
    {abbr}`GPS (Global Positioning System)` epoch[^gps-epoch];
    always strictly less than the time of the first
    [aspect fix](#aspect-schema) for the eclipse.
* - `eclipse_duration`
  - float
  - scst file header, `TRANGE1`
  - Duration of this eclipse, in seconds;
    `eclipse_start + eclipse_duration` is always strictly greater than
    the time of the last [aspect fix](#aspect-schema) for the eclipse.
* - `ok_exposure_time`
  - float[^null-unsupport]
  - Leg recalculation
  - Total time during which usable data was being collected;
    always less than `eclipse_duration`;
    always a whole number of seconds.
* - `ra_min`
  - float[^null-unsupport]
  - Leg recalculation
  - {abbr}`RA (equatorial right ascension)` of the western edge of a
    box enclosing the telescope’s aggregate field of view during this
    eclipse.
* - `ra_max`
  - float[^null-unsupport]
  - Leg recalculation
  - RA of the eastern edge of a box enclosing the telescope’s
    aggregate field of view during this eclipse.  Always greater than
    `ra_min`.  In rare cases (when the field of view crossed the ±180°
    meridian), `ra_max` can be greater than 180.
* - `ra0`
  - float[^null-unsupport]
  - Leg recalculation
  - RA of the centroid of the telescope’s aggregate field of view.
    Always greater than `ra_min` and less than `ra_max`.  In rare
    cases (when the field of view crossed the ±180° meridian), `ra0`
    can be greater than 180.
* - `dec_min`
  - float[^null-unsupport]
  - Leg recalculation
  - Declination of the southern edge of a box enclosing the
    telescope’s aggregate field of view during this eclipse.
* - `dec_max`
  - float[^null-unsupport]
  - Leg recalculation
  - Declination of the northern edge of a box enclosing the
    telescope’s aggregate field of view during this eclipse.
    Always greater than `dec_min`.
* - `dec0`
  - float[^null-unsupport]
  - Leg recalculation
  - Declination of the centroid of the telescope’s aggregate field of view.
    Always greater than `dec_min` and less than `dec_max`.
* - `nuv_det_on_time`
  - float
  - scst file header, `NHVNOMN`
  - Total time the {abbr}`NUV (near-ultraviolet)` detector was
    operational during the eclipse.
* - `nuv_det_temp`
  - float[^null-nuv-off]
  - scst file header, `NDTDET`
  - Temperature (°C) of one component of the NUV detector.
* - `nuv_tdc_temp`
  - float[^null-nuv-off]
  - scst file header, `NDTTDC`
  - Temperature (°C) of another component of the NUV detector.
* - `nuv_has_raw6`
  - boolean
  - MAST index
  - Whether MAST has archived a “raw6” file containing GALEX’s
    observations in the NUV band.
* - `fuv_det_on_time`
  - float
  - scst file header, `FHVNOMN`
  - Total time the {abbr}`FUV (far-ultraviolet)` detector was
    operational during the eclipse.
* - `fuv_det_temp`
  - float[^null-fuv-off]
  - scst file header, `FDTDET`
  - Temperature (°C) of one component of the FUV detector.
* - `fuv_tdc_temp`
  - float[^null-fuv-off]
  - scst file header, `FDTTDC`
  - Temperature (°C) of another component of the FUV detector.
* - `fuv_has_raw6`
  - boolean
  - MAST index
  - Whether MAST has archived a “raw6” file containing GALEX’s
    observations in the FUV band.

:::

[^gps-epoch]: January 6, 1980, at 00:00:00 UTC.

[^null-unsupport]: These columns will all be null when (and only when)
    the `has_aspect` column is false, indicating that gPhoton 2
    currently doesn’t support the observation mode of this eclipse.

[^null-nuv-off]: These columns will be either null or NaN when
    `nuv_det_on_time` is zero, meaning that no data was collected in
    the NUV band for this eclipse.  Caution: `nuv_has_raw6` may be
    either true or false in _both_ cases.  Meaningful NUV data is only
    available when `nuv_has_raw6` is true _and_ `nuv_det_on_time` is
    positive.

[^null-fuv-off]: These columns will be either null or NaN when
    `fuv_det_on_time` is zero, meaning that no data was collected
    in the FUV band for this eclipse.  Caution: `fuv_has_raw6` may be
    either true or false in _both_ cases.  Meaningful FUV data is only
    available when `fuv_has_raw6` is true _and_ `fuv_det_on_time` is
    positive.

(boresight)=
### `boresight.parquet`

Thist

(leg-aperture)=
### `leg-aperture.parquet`

(aspect)=
### `aspect.parquet`
