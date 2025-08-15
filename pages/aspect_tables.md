---
title: The Aggregated Aspect and Metadata Tables
site:
  hide_outline: true
  hide_toc: true
  hide_title_block: true
---

gPhoton 2 relies on four large tables of precalculated data.  These
contain information about each [{abbr}`GALEX (Galexy Evolution
Explorer)`][] eclipse[^eclipse] which depends only on how the GALEX
satellite was programmed for that eclipse (this is the “metadata”) and
what part of the sky it was actually pointed at during each part of
the eclipse (the “aspect data”).  Everything in these tables can,
in principle, be recalculated from the [GALEX archive][] on
[{abbr}`MAST (Mikulski Archive for Space Telescopes)`][];
however, this is an expensive process and there is little
scientific value in repeating it.

The aspect and metadata tables are not distributed along with the
gPhoton source code. [They are currently downloadable from Google
Drive][gdrive-aspect], but may be moved to MAST, or another more
convenient location, in the future.

[^eclipse]: The GALEX satellite was in a medium-altitude orbit of
    Earth.  Because it was designed to observe very faint stars and
    galaxies, it could only collect data during the part of each orbit
    that passed within Earth’s shadow.  Each such period of active
    observation is referred to as an eclipse.

% This server does not offer HTTPS
[GALEX]: http://www.galex.caltech.edu/

[GALEX archive]: https://archive.stsci.edu/missions-and-data/galex
[MAST]: https://archive.stsci.edu/
[gdrive-aspect]: https://drive.google.com/drive/u/1/folders/1aPfLKsZM8x5Pxji0Lh3dUblpo9dyt1IW

## Contents of the aggregated metadata tables

There are four files of aggregated metadata; each contains a single
table in [Apache Parquet][] format.

(metadata)=
### `metadata.parquet`

This table contains one row for every eclipse that has _something_
archived on MAST, whether or not gPhoton 2 can currently process
the type of data that GALEX collected during that eclipse.  Notably,
spectroscopic “grism” observations are currently not supported, but
appear in this table anyway.

Most of the data in this table is aggregated from the “{abbr}`SCST
(spacecraft state)`” files on MAST.  Some columns contain summary data
from the [heuristic leg recalculation](#leg-recalc) process,
described below.

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
  - Serial number of the eclipse
* - `plan_type`
  - string
  - scst file header, `MPSTYPE`
  - Code for the type of observation planned for this eclipse (see [](#plan-types))
* - `plan_subtype`
  - string
  - scst file header, `MPSPLAN`
  - Code for the subtype of observation planned for this eclipse (see [](#plan-subtypes))
* - `visit`
  - integer
  - scst file header, `VISIT`
  - Visit code for this eclipse (needs elaboration)
* - `plan_id`
  - integer
  - scst file header, `PLANID`
  - Code for the specific observation plan for this eclipse (needs elaboration)
* - `planned_legs`
  - integer
  - scst file header, `MPSNPOS`
  - Number of _planned_ observation targets for this eclipse
* - `observed_legs`
  - integer (may be null)
  - Leg recalculation
  - Number of distinct targets actually observed
* - `has_aspect`
  - boolean
  - `aspect.parquet`
  - True if gPhoton 2 supports processing this eclipse’s data
* - `eclipse_start`
  - float
  - Leg recalculation
  - Start time for this eclipse, as seconds since the GPS epoch; always strictly less than the time of the first [aspect fix](#aspect-schema) for the eclipse
* - `eclipse_duration`
  - float
  - Leg recalculation
  - Duration of this eclipse, in seconds; `eclipse_start + eclipse_duration` is always strictly greater than the time of the last [aspect fix](#aspect-schema) for the eclipse
* - `ok_exposure_time`
  - float (may be null)
  - Leg recalculation
  - Total time during which usable data was weing collected; always less than `eclipse_duration`; always a whole number of seconds
* - `ra0`
  - float (may be null)
  - Leg recalculation
  - {abbr}`RA (equatorial right ascension)` of the centroid of where the telescope was pointed during this eclipse, in decimal degrees
* - `ra_min`
  - float (may be null)
  - Leg recalculation
  - RA of one corner of a bounding box for where the eclipse was pointed
* - `ra_max`
  - float (may be null)
  - Leg recalculation
  - RA of the diagonally opposite corner of the bounding box
* - `dec0`
  - float (may be null)
  - Leg recalculation
  - {abbr}`Dec (equatorial declination)` of the centroid
* - `dec_min`
  - float (may be null)
  - Leg recalculation
  - Dec of one corner of the bounding box
* - `dec_max`
  - float (may be null)
  - Leg recalculation
  - Dec of the diagonally opposite corner
* - `nuv_det_on_time`
  - float
  - scst file header, `NHVNOMN`
  - Total time the {abbr}`NUV (near-ultraviolet)` detector was operational during the eclipse
* - `nuv_det_temp`
  - float (may be NaN)
  - scst file header, `NDTDET`
  - Temperature of one component of the NUV detector (°C)
* - `nuv_tdc_temp`
  - float (may be NaN)
  - scst file header, `NDTTDC`
  - Temperature of another component of the NUV detector (°C)
* - `nuv_has_raw6`
  - boolean
  - MAST index
  - Whether MAST has archived a “raw6” file containing GALEX’s observations in the NUV band
* - `fuv_det_on_time`
  - float
  - scst file header, `FHVNOMN`
  - Total time the {abbr}`FUV (far-ultraviolet)` detector was operational during the eclipse
* - `fuv_det_temp`
  - float (may be NaN)
  - scst file header, `FDTDET`
  - Temperature of one component of the FUV detector (°C)
* - `fuv_tdc_temp`
  - float (may be NaN)
  - scst file header, `FDTTDC`
  - Temperature of another component of the FUV detector (°C)
* - `fuv_has_raw6`
  - boolean
  - MAST index
  - Whether MAST has archived a “raw6” file containing GALEX’s observations in the FUV band

:::

Nulls appear in the columns that “may be null” only when the eclipse
collected data of a type that gPhoton 2 does not currently support.
These eclipses are completely absent from the other three tables.
Whenever one of these columns contains a null, all the rest of them
will also contain nulls, and the `has_aspect` column will be false;
conversely, if `has_aspect` is true, then none of these columns will
be null.

The `nuv_det_temp` and `nuv_tdc_temp` columns contain NaN values when,
and only when, `nuv_det_on_time` is zero.  Similarly, the `fuv_det_temp`
and `fuv_tdc_temp` columns contain NaN values when, and only when,
`fuv_det_on_time` is zero.  Note that `nuv_has_raw6` and `fuv_has_raw6`
are not so neatly correlated with `nuv_det_on_time` and `fuv_det_on_time`.
Meaningful data is available in a band only when the `has_raw6` column
for that band is true *and* the `det_on_time` column for that band is
positive.

(plan-types)=
### `plan_type` codes

The `plan_type` column holds a three-letter code that identifies the
general type of observation that was carried out during each eclipse.
These correspond to the different classes of “science survey”
documented in [chapter 2 of the official GALEX technical
documentation][galex-tech-ch2]; see that document for a slightly more
detailed description of each survey class.

Whether gPhoton 2 supports analysis of data from an eclipse (see above)
depends only on its `plan_type`.

:::{list-table} Plan types supported by gPhoton 2
:label: plan-types-supported
:align: center
:header-rows: 1

* - Code
  - Official name
  - Description
* - AIS
  - All-sky Imaging Survey
  - Short-duration exposures intended to cover as much of the sky as possible.
    (The galactic plane and several other ultraviolet bright spots had to be avoided.)
    AIS eclipses invariably are split into multiple “legs” (see [](#boresight)).
* - MIS
  - Medium Imaging Survey
  - Exposures as long as will fit in one eclipse ($\approx 1\,500\,\text{s}$),
    targets selected to match other broad sky surveys.
* - DIS
  - Deep Imaging Survey
  - Repeated long exposures of targets known to be interesting in the ultraviolet.
* - NGS
  - Nearby Galaxy Survey
  - A selective sample of nearby galaxies.
* - CAI
  - Calibration Imaging
  - Images of white dwarf stars of known ultraviolet brightness, for calibration.
* - DTI
  - ???
  - ???
* - ETI
  - ???
  - ???
* - GII
  - Guest Investigator Imaging
  - Targets selected by researchers outside the GALEX team.  Many subtypes.

:::

:::{list-table} Plan types currently not supported by gPhoton 2
:label: plan-types-unsupported
:align: center
:header-rows: 1

* - Code
  - Official name
  - Description
* - MSS
  - Medium Spectroscopic Survey
  - ???
* - DSS
  - Deep Spectroscopic Survey?
  - ???
* - WSS
  - Wide? Spectroscopic Survey?
  - ???
* - CAS
  - Calibration Spectroscopy
  - Similar to CAI, but data was collected in spectroscopic mode
* - ETS
  - ???
  - ???
* - GIS
  - Guest Investigator Spectroscopy?
  - Similar to GII, but data was collected in spectroscopic mode?

:::

[galex-tech-ch2]: http://www.galex.caltech.edu/researcher/techdoc-ch2.html

### `plan_subtype` codes

(boresight)=
### `boresight.parquet`

(leg-aperture)=
### `leg-aperture.parquet`

(aspect)=
### `aspect.parquet`

[Apache Parquet]: https://parquet.apache.org/

(leg-recalc)=
## Heuristic Leg Recalculation
