---
title: The Aggregated Aspect and Metadata Tables
site:
  hide_outline: true
---

# The Aggregated Aspect and Metadata Tables

gPhoton 2 relies on four large tables of precalculated data.
These contain information about each GALEX {term}`eclipse`
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

[GALEX archive]: https://archive.stsci.edu/missions-and-data/galex
[MAST]: https://archive.stsci.edu/
[gdrive-aspect]: https://drive.google.com/drive/u/1/folders/1aPfLKsZM8x5Pxji0Lh3dUblpo9dyt1IW

## Contents of the aggregated metadata tables

There are four files of aggregated metadata; each contains a single
table in [Apache Parquet][] format.

All four tables contain celestial coordinates; these are uniformly
equatorial (J2000), with values in decimal degrees.  Declination
ranges from −90° to 90°.  Right ascension _normally_ ranges from −180°
to 180°, but values greater than 180° can appear in a few places, such
as the `ra0` and `ra_max` columns of `metadata.parquet` and
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
  - Number of _planned_ observation targets ({term}`leg`s) for this eclipse.
    Always at least 1; never higher than 12.
* - `observed_legs`
  - integer[^null-unsupport]
  - Leg recalculation
  - Number of distinct targets actually observed.
    This is _usually_ equal to `planned_legs` but can be smaller or bigger.
    When not null, always at least 1 and never higher than 18.
* - `has_aspect`
  - boolean
  - `aspect.parquet`
  - True if gPhoton 2 supports processing this eclipse’s data.
* - `eclipse_start`
  - float
  - scst file header, `TRANGE0`
  - Start time for this eclipse, as seconds since the {term}`GPS epoch`;
    always strictly less than the time of the first
    [aspect fix](#aspect) for the eclipse.
* - `eclipse_duration`
  - float
  - scst file header, `TRANGE1`
  - Duration of this eclipse, in seconds;
    `eclipse_start + eclipse_duration` is always strictly greater than
    the time of the last [aspect fix](#aspect) for the eclipse.<br>
    Caution: for the majority of eclipses this number is nonsense
    (substantially larger than the maximum physically plausible
    duration of an eclipse).
* - `ok_exposure_time`
  - float[^null-unsupport]
  - Leg recalculation
  - Total time during which usable data was being collected;
    always less than `eclipse_duration`;
    always a whole number of seconds.
    Unlike `eclipse_duration` this number is always in a sensible range.
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
  - RA of the centroid of the telescope’s aggregate field of view
    during this eclipse.  Always greater than `ra_min` and less than
    `ra_max`.  In rare cases (when the field of view crossed the ±180°
    meridian), `ra0` can be greater than 180.
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
  - Declination of the centroid of the telescope’s aggregate field of
    view during this eclipse.  Always greater than `dec_min` and less
    than `dec_max`.
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

This table contains one row for each _observed_ {term}`leg` of each
eclipse that gPhoton 2 can process.  Eclipses for which `has_aspect`
is false in [](#metadata) do not appear in this table.  It provides
reference values needed to process the [aspect fixes](#aspect) for
each leg.

The data in this table was entirely produced by the [heuristic leg
recalculation](/glcat-leg-recalc) process.

:::{list-table} Columns of `boresight.parquet`
:label: boresight-schema
:align: center
:header-rows: 1

* - Column
  - Type
  - Contents
* - `eclipse`
  - integer
  - Serial number of the eclipse.
* - `leg`
  - integer
  - 1-based index of the leg within the eclipse.
* - `time`
  - float
  - Starting time of the leg, as seconds since the {term}`GPS epoch`;
    always strictly less than the time of the first _usable_
    [aspect fix](#aspect) for the leg.
* - `duration`
  - float
  - Duration of the leg, in seconds.  `time + duration` is always
    strictly greater than the time of the last _usable_ [aspect fix](#aspect)
    for the leg.
* - `ra_min`
  - float
  - {abbr}`RA (equatorial right ascension)` of the western edge of a
    box enclosing the telescope’s aggregate field of view during this
    leg.
* - `ra0`
  - float
  - RA of the centroid of the telescope’s aggregate field of view
    during this leg.  Always greater than `ra_min` and less than
    `ra_max`.  In rare cases (when the field of view crossed the ±180°
    meridian), `ra0` can be greater than 180.
* - `ra_max`
  - float
  - RA of the eastern edge of a box enclosing the telescope’s
    aggregate field of view during this leg.  Always greater than
    `ra_min`.  In rare cases (when the field of view crossed the ±180°
    meridian), `ra_max` can be greater than 180.
* - `dec_min`
  - float
  - Declination of the southern edge of a box enclosing the
    telescope’s aggregate field of view during this leg.
* - `dec0`
  - float
  - Declination of the centroid of the telescope’s aggregate field of
    view during this leg.  Always greater than `dec_min` and less
    than `dec_max`.
* - `dec_max`
  - float
  - Declination of the northern edge of a box enclosing the
    telescope’s aggregate field of view during this leg.
    Always greater than `dec_min`.
* - `planned_ra`
  - float
  - RA of the _planned_ target of observation for this leg.
    Normally close to `ra0`, but can be substantially different
    (whenever the telescope did not carry out the plan as intended).
* - `planned_dec`
  - float
  - Declination of the _planned_ target of observation for this leg.
    Normally close to `dec0`, but can be substantially different.
* - `full_exposure_area`
  - float
  - Solid angle, in square degrees, covered by the region of the sky
    that the telescope observed for the entire usable duration of the
    leg (the “full exposure region”).  For legs where the telescope
    was programmed nominally and carried out its program successfully,
    this will be slightly smaller than 1.25 square degrees.  It cannot
    be greater than 1.25.
* - `full_exposure_uncircularity`
  - float
  - A measure of how much the full exposure region deviates from a
    perfect circle.  Specifically, this is the reciprocal of the
    [isoperimetric quotient][isop] of the full exposure region,
    minus 1.  Zero would indicate the full exposure region _is_ a
    perfect circle (this never happened), and larger values mean the
    telescope’s pointing had more and more of a _directional_ wander
    to it.
* - `partial_exposure_area_ratio`
  - float
  - A measure of how much smaller the full exposure region is than the
    ideal.  Specifically, this is the solid angle covered by the
    region of the sky that was observed for only _part_ of the usable
    duration of the leg, divided by the `full_exposure_area`.  Zero
    would indicate that the telescope’s pointing did not wander at all
    (this never happened), larger values mean more wander of some kind.

:::

[isop]: https://mathworld.wolfram.com/IsoperimetricQuotient.html

(leg-aperture)=
### `leg-aperture.parquet`

This table contains one row for each _observed_ {term}`leg` of each
eclipse that gPhoton 2 can process.  Eclipses for which `has_aspect`
is false in [](#metadata) do not appear in this table.  It provides
information about GALEX’s overall field of view for each leg.  It’s
separate from [](#boresight) because the field of view information is
bulky and not always required.

The “geometry” columns of this table each store
a vector representation of a region of the sky,
in [WKB format](https://libgeos.org/specifications/wkb/).
The geometry type is always either `Polygon` or `MultiPolygon`.

:::{note}
RA coordinates of regions in this table never exceed +180°; when the
field of view crosses the ±180° meridian of right ascension, the
region is split at that meridian, with some of the subpolygons having
RA coordinates close to +180° and others close to −180°. This is
different from the handling of RA values in [](#metadata) and
[](#boresight).
:::

The data in this table was entirely produced by the [heuristic leg
recalculation](/glcat-leg-recalc) process.

:::{list-table} Columns of `leg-aperture.parquet`
:label: leg-aperture-schema
:align: center
:header-rows: 1

* - Column
  - Type
  - Contents
* - `eclipse`
  - integer
  - Serial number of the eclipse.
* - `leg`
  - integer
  - 1-based index of the leg within the eclipse.
* - `partial`
  - geometry
  - The region of the sky that was observed for _at least_ part of the leg.
* - `full_400`
  - geometry
  - The region of the sky that was observed for all of the leg.
* - `full_370`
  - geometry
  - The region of the sky that was observed _by the central part of
    the detector plate_ for all of the leg.  Data from the edges of
    the detector plate is often contaminated by
    [artifacts](/image-artifacts); this region is somewhat less
    likely to be contaminated.
* - `full_350`
  - geometry
  - Similar to `full_370` but using an even more restrictive
    definition of “the central part of the detector.”  [TODO:
    Document what 400, 370, and 350 actually mean. -zw]

:::

(aspect)=
### `aspect.parquet`
