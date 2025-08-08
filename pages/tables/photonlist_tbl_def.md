# Photonlist table description

| Column Name | Units | Description |
|-------------|-------|-------------|
| t | seconds | The timestamp in GALEX time units. These typically have a maximum resolution of 0.05 seconds. A small number of observations were collected in a special mode that enabled high resolution; these were mostly for diagnostics and are not recommended for scientific analyses. |
| q | | Quantum efficiency. |
| ya | | |
| col | | |
| row | | |
| flags | | |
| ra | | |
| dec | | |
| detrad | | |
| mask | | |
| response | | |

## Flag column values
This is a description of the photonlist flags for completeness. All the vast majority of users of the photonlist files need to know, however, is that photon events with a flag value of _zero_ are considered "good" and any photon event with a flag value _other than zero_ should not be used in routine analyses.

```{aside}
One important exception is that _flagged_ photons still contribute to the global countrate and therefore affect the detector dead time. So the detector dead time correction **[CITE CODE]** uses _all_ photon events, regardless of flaggedness.
```

The photon flags use a binary bitmask.[^bitmask] Although there are only three valid values of flag in _gPhoton2_ photonlist files.

[^bitmask]: Binary bitmasks are common and familiar to computer scientists and programmers, but we have found that they are not as familiar to astronomers. So here is a very brief description for those who need it. The idea of a binary bitmask is that each digit of a binary number encodes the state for a separate item. To decode them, just make sure to represent the number in binary and then "read" the values. For example, a flag value of `6` is `0b110` in binary. This means that the second and third flags are triggered (with values of `1`) but not the first flag (with a value of `0`). A flag value of `65` is `0b1000001` in binary, so the first and seventh flags are set. (But if you are talking to a computer scientist, make sure to refer to them as the zeroth and sixth flags.)

| Flag (Decimal) | Flag (Binary) | Description |
|----------------|---------------|-------------|
| 0  | `0b0000` | Notionally "good" photon event data with a presumed valid aspect correction. |
| 7  | `0b0111` | |
| 12 | `0b1100` | |

### Brief note on photon flags
The power user of _gPhoton1_ will note that there were a lot more photonlist flags defined previously. We have eliminated most of them due to pointlessness. Also, one notable difference between _gPhoton1_ and _gPhoton2_ is that the former performed hotspot masking **[LINK]** as part of photon-level flagging. Indeed, _gPhoton1_ declined to even perform aspect correction on photon events covered by the hotspot mask. However, it turns out that some hotspots are transient, and this overzealous masking inhibited some categories of investigation. (See, for example, the study of time-resolved flares on GJ65 by https://doi.org/10.3847/1538-4357/ac5037). So _gPhoton2_ applies the _mask_ as well as the _flat_ **[CITE]** as part of image integration, after the photon events have been aspect-corrected.