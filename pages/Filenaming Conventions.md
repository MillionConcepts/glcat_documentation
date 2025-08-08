# gPhoton2 file types

All filename elements are lowercase.

## Photometry filenames

Examples of photometry filenames are `e23456-nd-f0060-b00-movie-photom-12_8.csv` or `e03456-fd-f0060-b02-image-photom-30_0.csv`.

They have a general format like:

```
e{eclipse_number}-{band_initial}d-f{frame_depth}-b{leg_number}-{integration_type}-photom-{aperture_radius}.csv
```

| Element | Description | Example |
|---------|-------------|---------|
| GALEX eclipse number | 5 digits, zero-padded | `e03456` |
| Band initial | one character, either `f`(uv) or `n`(uv) | `nd`[^imaging_modes] |
| Frame depth | Integration depth of each frame (4 digits, zero-padded), _or_ `ffull` if it is a full depth integration[^integration_types] | `f0060` |
| Leg number | Observation leg number (2 digits, zero-padded) | `b01` |
| Integration type | `image` or `movie`[^integration_types] | `movie` |
| Aperture radius | in arcseconds, _not_ zero-padded, with one decimal place using an underscore for the decimal | `12_8` |

[^imaging_modes]: This convention of `[nf]d` was used by the mission. The "d" in this case stands for "direct" as in "direct imaging." This corresponds to one of the three selectable filters for the mission. The others were "grism," which would be indicated by `[nf]g`. The final mode was "opaque," which was like putting the lens cap on the telescope. Although the "opaque" filter had holes in it, so it is not a lot of use for e.g. making straightforward measurements of detector background.
[^integration_types]: An integration type of `image` will always have a frame depth of `ffull` and vice versa. A "full depth" integration uses the maximum contiguous integration for a given observation + leg, which may be anywhere between a few dozen seconds and half an hour. The exposure time is not discernible from the filenames; this information is contained in the files.

## Image filenames

Example of image filenames are `e23456-nd-ffull-b00-image-r.fits` or `e23456-fd-f0120-b00-movie-r.fits`.

They have a general format like:

```
e{eclipse_number}-{band_initial}d-f{frame_depth}-b{leg_number}-{integration_type}-{compression_type}.fits
```

| Element | Description | Example |
|---------|-------------|---------|
| GALEX eclipse number | 5 digits, zero-padded | `e03456` |
| Band initial | one character, either `f`(uv) or `n`(uv) | `nd`[^imaging_modes] |
| Frame depth | Integration depth of each frame (4 digits, zero-padded), _or_ `ffull` if it is a full depth integration[^integration_types] | `f0060` |
| Leg number | Observation leg number (2 digits, zero-padded) | `b01` |
| Integration type | `image` or `movie`[^integration_types] | `movie` |
| Compression type | Monolithic file compression: `r` for RICE, `g` for GZIP, **???** for none | `r` |

## Photonlist filenames

The photonlist files contain a table for which each row corresponds to a single photon event recorded by GALEX. These are most similar to what the original mission called "x" (for e[x]tended) files. They are also similar in content to the _gPhoton1_ photon database, as well as the photon file data output directly by the _gPhoton1_ data processing pipeline (called simply `gPhoton`). The columns of the photonlist files are defined [here](tables/photonlist_tbl_def.md).

An example of a raw6 filename is `e23456-nd-b00.parquet`.

They have a general format like:

```
e{eclipse_number}-{band_initial}d-b{leg_number}.parquet

```
| Element | Description | Example |
|---------|-------------|---------|
| GALEX eclipse number | 5 digits, zero-padded | `e03456` |
| Band initial | one character, either `f`(uv) or `n`(uv) | `nd`[^imaging_modes] |
| Leg number | Observation leg number (2 digits, zero-padded) | `b01` |


## raw6 filenames

The "raw6" files contain the photon even data that was derived directly from downlinked telemetry. The data in these must be decoded, reprojected, and calibrated before they are of direct scientific interest to astronomers. This processing is one of the primary functions of the _gPhoton_ tools. During a _gPhoton2_ run, the software will pull any required raw6 files (for the processing being performed) from the MAST servers, unless they are already specified in local directories. Most scientists will never want to look at the raw6 files directly, but people who run _gPhoton2_ will end up with these files on their systems. Here is a description so that they know what they are looking at. The best documentation for the _content_ of these files is the [gPhoton2 source code](https://github.com/MillionConcepts/gPhoton2/).

```{aside}
We are not entirely certain why these are called "raw6" files. We think perhaps they were the sixth of a set of files derived directly from the raw telemetry streams (which we know were stored in files called `tlm` for telemetry).
```

An example filename is `e23456-fd-raw6.fits.gz`.

They have a general format like:
```
e{eclipse_number}-{band_initial}d-raw6.fits.gz
```

| Element | Description | Example |
|---------|-------------|---------|
| GALEX eclipse number | 5 digits, zero-padded | `e03456` |
| Band initial | one character, either `f`(uv) or `n`(uv) | `nd`[^imaging_modes] |
