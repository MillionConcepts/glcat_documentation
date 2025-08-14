# Image file metadata

Many of the FITS header keywords are specified in the FITS standard. We have attempted to utilize FITS standard keywords whenever it is appropriate and practical. We will not repeat the descriptions of these keywords from the FITS standard. We will, however, describe the keywords that are specific to the _gPhoton2_ image files, or those that may be used in a confusing or unusual way.

### Stub Header

Regardless of image type, _if_ the FITS file is monolithically compressed using RICE compression, then the first FITS HDU will be a stub header. This is a requirement of the FITS standard when RICE compression is used. The stub header will be as follows, exactly:

```{aside}
_gPhoton2_ appends `-r` to FITS filenames when the files have been RICE compressed.
```

```
SIMPLE  =                    T / file does conform to FITS standard             
BITPIX  =                   16 / number of bits per data pixel                  
NAXIS   =                    0 / number of data axes                            
EXTEND  =                    T / FITS dataset may contain extensions            
COMMENT   FITS (Flexible Image Transport System) format is defined in 'Astronomy
COMMENT   and Astrophysics', volume 376, page 359; bibcode: 2001A&A...376..359H
```

## Full-depth images

### Count image header (CNT)

```{card}
| Keyword | Description |
|---------|-------------|
| `XTENSION` | The extension type, which is always `IMAGE` for images. |
| `BITPIX` | The data type of the image, which is always `-32` for 32-bit floating point images. |
| `NAXIS` | The number of dimensions of the image, which is always `2` for 2D images. |
| `NAXIS1` | The length of the first axis of the image, which is the width in pixels. |
| `NAXIS2` | The length of the second axis of the image, which is the height in pixels. |
| `PCOUNT` | The number of additional parameters in the header, which is always `0` for images. |
| `GCOUNT` | The number of groups in the image, which is always `1` for single images. |
| `EXTNAME` | The name of the extension, which is `CNT` for count images. |
| `TELESCOP` | The name of the telescope, which is always `GALEX`. |
| `ECLIPSE` | The GALEX eclipse number, which is a unique identifier for the observation. |
| `LEG` | The observation leg number, which is a unique identifier for the observation leg. |
| `BANDNAME` | The name of the band, which is either `NUV` or `FUV`. |
| `BAND` | The band number, which is `1` for NUV and `2` for FUV. |
| `TIMESYS` | The time system used, which is always `UTC`. |
| `TIMEUNIT` | The time unit used, which is always `s` (seconds). |
| `DATE-BEG` | The start date and time of the observation in ISO 8601 format. |
| `DATE-END` | The end date and time of the observation in ISO 8601 format. |
| `TSTART` | The start time of the observation in seconds since the epoch. |
| `TSTOP` | The end time of the observation in seconds since the epoch. |
| `TELAPSE` | The total duration of the observation in seconds (TSTOP - TSTART). |
| `XPOSURE` | The effective exposure time of the observation in seconds, corrected for dead time and shutter effects. |
| `T0_0` | The start time of the observation in seconds since the epoch, for the first frame. |
| `T1_0` | The end time of the observation in seconds since the epoch, for the first frame. |
| `EXPT_0` | The exposure time for the first frame in seconds. |
| `PHOTCNT` |The total number of photons counted in the observation. |
| `N_FRAME` | The number of frames in the observation, which is always `1` for single images. |
| `CDELT1` | The pixel size in the first axis in degrees. This is the angular size of each pixel in the right ascension direction. |
| `CDELT2` | The pixel size in the second axis in degrees. This is the angular size of each pixel in the declination direction. |
| `CTYPE1` | The coordinate type for the first axis, which is always `RA---TAN` (Right Ascension, tangent projection). |
| `CTYPE2` | The coordinate type for the second axis, which is always `DEC--TAN` (Declination, tangent projection). |
| `CRPIX1` | The reference pixel for the first axis, which is the pixel number of the reference point in the right ascension direction. |
| `CRPIX2` | The reference pixel for the second axis, which is the pixel number of the reference point in the declination direction. |
| `CRVAL1` | The right ascension of the reference pixel in degrees. This is the celestial coordinate of the reference point in the right ascension direction. |
| `CRVAL2` | The declination of the reference pixel in degrees. This is the celestial coordinate of the reference point in the declination direction. |
| `EQUINOX` | The equinox of the celestial coordinates, which is always `2000`. |
| `EPOCH` | The epoch of the celestial coordinates, which is always `2000`. |
| `ORIGIN` | The original producer of the FITS file, which is always `Million Concepts` for GLCAT. |
| `VERSION` | The version of the _gPhoton2_ software used to create the file. |
| `DATE` | The date and time when the FITS file was created in ISO 8601 format. |
```

```{dropdown} Example FITS header for a count (CNT) image
XTENSION= 'IMAGE   '                                                            
BITPIX  =                  -32 / data type of original image                    
NAXIS   =                    2 / dimension of original image                    
NAXIS1  =                 3065 / length of original image axis                  
NAXIS2  =                 3099 / length of original image axis                  
PCOUNT  =                    0                                                  
GCOUNT  =                    1                                                  
EXTNAME = 'CNT     '                                                            
TELESCOP= 'GALEX   '                                                            
ECLIPSE =                23456                                                  
LEG     =                    0                                                  
BANDNAME= 'NUV     '                                                            
BAND    =                    1                                                  
TIMESYS = 'UTC     '                                                            
TIMEUNIT= 's       '                                                            
DATE-BEG= '2007-09-20T22:07:21'                                                 
DATE-END= '2007-09-20T22:35:46'                                                 
TSTART  =        874361241.995                                                  
TSTOP   =        874362946.995                                                  
TELAPSE =                1705.                                                  
XPOSURE =             1500.229                                                  
T0_0    =        874361241.995                                                  
T1_0    =        874362946.995                                                  
EXPT_0  =             1500.228                                                  
PHOTCNT =             35691827                                                  
N_FRAME =                    1                                                  
CDELT1  = -0.000416666666666667                                                 
CDELT2  = 0.000416666666666667                                                  
CTYPE1  = 'RA---TAN'                                                            
CTYPE2  = 'DEC--TAN'                                                            
CRPIX1  =                1533.                                                  
CRPIX2  =                1550.                                                  
CRVAL1  =     323.492794193632                                                  
CRVAL2  =    -2.06396933577621                                                  
EQUINOX =                2000.                                                  
EPOCH   =                2000.                                                  
ORIGIN  = 'Million Concepts'                                                    
VERSION = 'v3.0.0a0'                                                            
DATE    = '2025-07-26T01:07:33+00:00'                                           
COMMENT Sky-projected histograms of photon counts.
```

### Flag map header (FLAG)

```{card}
| Keyword | Description |
|---------|-------------|
| `HARDEDGE` | The "hard edge" flag radius in pixels. This defines a wide boundary where edge effects are significant enough to warrant blanket exclusion of data from analyses. |
| `SOFTEDGE` | The "soft edge" flag radius in pixels. This defines a narrower boundary where edge effects are likely to be present, and analyses should use caution. |
```

```{dropdown} Example FITS header for a flag (FLAG) map
XTENSION= 'IMAGE   '                                                            
BITPIX  =                    8 / data type of original image                    
NAXIS   =                    2 / dimension of original image                    
NAXIS1  =                 3065 / length of original image axis                  
NAXIS2  =                 3099 / length of original image axis                  
PCOUNT  =                    0                                                  
GCOUNT  =                    1                                                  
EXTNAME = 'FLAG    '                                                            
TELESCOP= 'GALEX   '                                                            
ECLIPSE =                23456                                                  
LEG     =                    0                                                  
BANDNAME= 'NUV     '                                                            
BAND    =                    1                                                  
TIMESYS = 'UTC     '                                                            
TIMEUNIT= 's       '                                                            
DATE-BEG= '2007-09-20T22:07:21'                                                 
DATE-END= '2007-09-20T22:35:46'                                                 
TSTART  =        874361241.995                                                  
TSTOP   =        874362946.995                                                  
TELAPSE =                1705.                                                  
XPOSURE =             1500.229                                                  
T0_0    =        874361241.995                                                  
T1_0    =        874362946.995                                                  
EXPT_0  =     1500.228                                                  
PHOTCNT =             35691827                                                  
N_FRAME =                    1                                                  
CDELT1  = -0.000416666666666667                                                 
CDELT2  = 0.000416666666666667                                                  
CTYPE1  = 'RA---TAN'                                                            
CTYPE2  = 'DEC--TAN'                                                            
CRPIX1  =                1533.                                                  
CRPIX2  =                1550.                                                  
CRVAL1  =     323.492794193632                                                  
CRVAL2  =    -2.06396933577621                                                  
EQUINOX =                2000.                                                  
EPOCH   =                2000.                                                  
HARDEDGE=                  370                                                  
SOFTEDGE=                  350                                                  
ORIGIN  = 'Million Concepts'                                                    
VERSION = 'v3.0.0a0'                                                            
DATE    = '2025-07-26T01:07:33+00:00'                                           
COMMENT Sky-projected bitmap of flags, where any flagged photon in that bin defi
COMMENT  nes the flag for that whole bin. Flag 1: Mission hotspot mask. Flag 2:
COMMENT P ost-CSP ghost flag. Flag 4: Wide edge flag. Flag 8: Narrow edge flag.
COMMENT Ex : Flag 12 means both narrow and wide edge flags are set.      
```

### Coverage map header (COVERAGE)

The COVERAGE header contains the same keywords as the CNT header, so all keyword descriptions can be found in the count image header keyword table above. No additional unique keywords are present in COVERAGE headers.

```{dropdown} Example FITS header for a coverage (COVERAGE) map
XTENSION= 'IMAGE   '                                                            
BITPIX  =                    8 / data type of original image                    
NAXIS   =                    2 / dimension of original image                    
NAXIS1  =                 3065 / length of original image axis                  
NAXIS2  =                 3099 / length of original image axis                  
PCOUNT  =                    0                                                  
GCOUNT  =                    1                                                  
EXTNAME = 'COVERAGE'                                                            
TELESCOP= 'GALEX   '                                                            
ECLIPSE =                23456                                                  
LEG     =                    0                                                  
BANDNAME= 'NUV     '                                                            
BAND    =                    1                                                  
TIMESYS = 'UTC     '                                                            
TIMEUNIT= 's       '                                                            
DATE-BEG= '2007-09-20T22:07:21'                                                 
DATE-END= '2007-09-20T22:35:46'                                                 
TSTART  =        874361241.995                                                  
TSTOP   =        874362946.995                                                  
TELAPSE =                1705.                                                  
XPOSURE =             1500.229                                                  
T0_0    =        874361241.995                                                  
T1_0    =        874362946.995                                                  
EXPT_0  =     1500.228                                                 
PHOTCNT =             35691827                                                  
N_FRAME =                    1                                                  
CDELT1  = -0.000416666666666667                                                 
CDELT2  = 0.000416666666666667                                                  
CTYPE1  = 'RA---TAN'                                                            
CTYPE2  = 'DEC--TAN'                                                            
CRPIX1  =                1533.                                                  
CRPIX2  =                1550.                                                  
CRVAL1  =     323.492794193632                                                  
CRVAL2  =    -2.06396933577621                                                  
EQUINOX =                2000.                                                  
EPOCH   =                2000.                                                  
ORIGIN  = 'Million Concepts'                                                    
VERSION = 'v3.0.0a0'                                                            
DATE    = '2025-07-26T01:07:33+00:00'                                           
COMMENT Aspect-derived maps in sky-projected space. Coverage varies due to dithe
COMMENT  r patterns. Full coverage during observation: 1. Partial coverage durin
COMMENT g  observation: 2. 
```

### Dose map header (DOSE)

```{card}
| Keyword | Description |
|---------|-------------|
| `PARTAREA` | **???** |
| `FULLAREA` | **???** |
```

```{dropdown} Example FITS header for a dose (DOSE) image
XTENSION= 'IMAGE   '                                                            
BITPIX  =                  -32 / data type of original image                    
NAXIS   =                    2 / dimension of original image                    
NAXIS1  =                 3200 / length of original image axis                  
NAXIS2  =                 3200 / length of original image axis                  
PCOUNT  =                    0                                                  
GCOUNT  =                    1                                                  
EXTNAME = 'DOSE    '                                                            
TELESCOP= 'GALEX   '                                                            
ECLIPSE =                23456                                                  
LEG     =                    0                                                  
BANDNAME= 'NUV     '                                                            
BAND    =                    1                                                  
TIMESYS = 'UTC     '                                                            
TIMEUNIT= 's       '                                                            
DATE-BEG= '2007-09-20T22:07:21'                                                 
DATE-END= '2007-09-20T22:35:46'                                                 
TSTART  =        874361241.995                                                  
TSTOP   =        874362946.995                                                  
TELAPSE =                1705.                                                  
XPOSURE =             1500.229                                                  
T0_0    =        874361241.995                                                  
T1_0    =        874362946.995                                                  
EXPT_0  =     1500.228                                                  
PHOTCNT =             35691827                                                  
N_FRAME =                    1                                                  
PARTAREA=     601646.689310881                                                  
FULLAREA=      6770353.1720786                                                  
ORIGIN  = 'Million Concepts'                                                    
VERSION = 'v3.0.0a0'                                                            
DATE    = '2025-07-26T01:07:33+00:00'                                           
COMMENT Detector-space histogram of photon counts.   
```

## Movie images

### Count movie Header (CNT)

```{card}
| Keyword | Description |
|---------|-------------|
| `T0_N` | The start time in seconds since the epoch for frame N, where N is the frame number. |
| `T1_N` | The end time in seconds since the epoch for frame N, where N is the frame number. |
| `EXPT_N` | The effective exposure time in seconds for frame N, where N is the frame number. |
```

```{dropdown} Example FITS header for a coverage (COVERAGE) map
XTENSION= 'IMAGE   '                                                            
BITPIX  =                  -32 / data type of original image                    
NAXIS   =                    3 / dimension of original image                    
NAXIS1  =                 3065 / length of original image axis                  
NAXIS2  =                 3099 / length of original image axis                  
NAXIS3  =                   15 / length of original image axis                  
PCOUNT  =                    0                                                  
GCOUNT  =                    1                                                  
EXTNAME = 'CNT     '                                                            
TELESCOP= 'GALEX   '                                                            
ECLIPSE =                23456                                                  
LEG     =                    0                                                  
BANDNAME= 'NUV     '                                                            
BAND    =                    1                                                  
TIMESYS = 'UTC     '                                                            
TIMEUNIT= 's       '                                                            
DATE-BEG= '2007-09-20T22:07:21'                                                 
DATE-END= '2007-09-20T22:37:21'                                                 
TSTART  =        874361241.995                                                  
TSTOP   =        874363041.995                                                  
TELAPSE =                1800.                                                  
XPOSURE =             1500.229                                                  
T0_0    =        874361241.995                                                  
T1_0    =        874361361.995                                                  
EXPT_0  =               103.74                                                  
T0_1    =        874361361.995                                                  
T1_1    =        874361481.995                                                  
EXPT_1  =              105.746                                                  
T0_2    =        874361481.995                                                  
T1_2    =        874361601.995                                                  
EXPT_2  =              105.919                                                  
T0_3    =        874361601.995                                                  
T1_3    =        874361721.995                                                  
EXPT_3  =              106.049                                                  
T0_4    =        874361721.995                                                  
T1_4    =        874361841.995                                                  
EXPT_4  =              102.603                                                  
T0_5    =        874361841.995                                                  
T1_5    =        874361961.995                                                  
EXPT_5  =              106.205                                                  
T0_6    =        874361961.995                                                  
T1_6    =        874362081.995                                                  
EXPT_6  =              106.241                                                  
T0_7    =        874362081.995                                                  
T1_7    =        874362201.995                                                  
EXPT_7  =              106.246                                                  
T0_8    =        874362201.995                                                  
T1_8    =        874362321.995                                                  
EXPT_8  =              106.235                                                  
T0_9    =        874362321.995                                                  
T1_9    =        874362441.995                                                  
EXPT_9  =              106.206                                                  
T0_10   =        874362441.995                                                  
T1_10   =        874362561.995                                                  
EXPT_10 =              106.137                                                  
T0_11   =        874362561.995                                                  
T1_11   =        874362681.995                                                  
EXPT_11 =              106.084                                                  
T0_12   =        874362681.995                                                  
T1_12   =        874362801.995                                                  
EXPT_12 =              105.953                                                  
T0_13   =        874362801.995                                                  
T1_13   =        874362921.995                                                  
EXPT_13 =              105.753                                                  
T0_14   =        874362921.995                                                  
T1_14   =        874363041.995                                                  
EXPT_14 =               21.112                                                  
PHOTCNT =             35691827                                                  
N_FRAME =                   15                                                  
CDELT1  = -0.000416666666666667                                                 
CDELT2  = 0.000416666666666667                                                  
CTYPE1  = 'RA---TAN'                                                            
CTYPE2  = 'DEC--TAN'                                                            
CRPIX1  =                1533.                                                  
CRPIX2  =                1550.                                                  
CRVAL1  =     323.492794193632                                                  
CRVAL2  =    -2.06396933577621                                                  
EQUINOX =                2000.                                                  
EPOCH   =                2000.                                                  
ORIGIN  = 'Million Concepts'                                                    
VERSION = 'v3.0.0a0'                                                            
DATE    = '2025-07-26T01:12:11+00:00'                                           
COMMENT Sky-projected histograms of photon counts. 
```

### Flag movie Header (FLAG)

The FLAG movie header contains the same keywords as the movie CNT header (described above) plus the FLAG-specific keywords (`HARDEDGE` and `SOFTEDGE`) described in the flag map header section. No additional unique keywords are present in FLAG movie headers.

```{dropdown} Example FITS header for a coverage (COVERAGE) map
XTENSION= 'IMAGE   '                                                            
BITPIX  =                    8 / data type of original image                    
NAXIS   =                    3 / dimension of original image                    
NAXIS1  =                 3065 / length of original image axis                  
NAXIS2  =                 3099 / length of original image axis                  
NAXIS3  =                   15 / length of original image axis                  
PCOUNT  =                    0                                                  
GCOUNT  =                    1                                                  
EXTNAME = 'FLAG    '                                                            
TELESCOP= 'GALEX   '                                                            
ECLIPSE =                23456                                                  
LEG     =                    0                                                  
BANDNAME= 'NUV     '                                                            
BAND    =                    1                                                  
TIMESYS = 'UTC     '                                                            
TIMEUNIT= 's       '                                                            
DATE-BEG= '2007-09-20T22:07:21'                                                 
DATE-END= '2007-09-20T22:37:21'                                                 
TSTART  =        874361241.995                                                  
TSTOP   =        874363041.995                                                  
TELAPSE =                1800.                                                  
XPOSURE =             1500.229                                                  
T0_0    =        874361241.995                                                  
T1_0    =        874361361.995                                                  
EXPT_0  =               103.74                                                  
T0_1    =        874361361.995                                                  
T1_1    =        874361481.995                                                  
EXPT_1  =              105.746                                                  
T0_2    =        874361481.995                                                  
T1_2    =        874361601.995                                                  
EXPT_2  =              105.919                                                  
T0_3    =        874361601.995                                                  
T1_3    =        874361721.995                                                  
EXPT_3  =              106.049                                                  
T0_4    =        874361721.995                                                  
T1_4    =        874361841.995                                                  
EXPT_4  =              102.603                                                  
T0_5    =        874361841.995                                                  
T1_5    =        874361961.995                                                  
EXPT_5  =              106.205                                                  
T0_6    =        874361961.995                                                  
T1_6    =        874362081.995                                                  
EXPT_6  =              106.241                                                  
T0_7    =        874362081.995                                                  
T1_7    =        874362201.995                                                  
EXPT_7  =              106.246                                                  
T0_8    =        874362201.995                                                  
T1_8    =        874362321.995                                                  
EXPT_8  =              106.235                                                  
T0_9    =        874362321.995                                                  
T1_9    =        874362441.995                                                  
EXPT_9  =              106.206                                                  
T0_10   =        874362441.995                                                  
T1_10   =        874362561.995                                                  
EXPT_10 =              106.137                                                  
T0_11   =        874362561.995                                                  
T1_11   =        874362681.995                                                  
EXPT_11 =              106.084                                                  
T0_12   =        874362681.995                                                  
T1_12   =        874362801.995                                                  
EXPT_12 =              105.953                                                  
T0_13   =        874362801.995                                                  
T1_13   =        874362921.995                                                  
EXPT_13 =              105.753                                                  
T0_14   =        874362921.995                                                  
T1_14   =        874363041.995                                                  
EXPT_14 =               21.112                                                  
PHOTCNT =             35691827                                                  
N_FRAME =                   15                                                  
CDELT1  = -0.000416666666666667                                                 
CDELT2  = 0.000416666666666667                                                  
CTYPE1  = 'RA---TAN'                                                            
CTYPE2  = 'DEC--TAN'                                                            
CRPIX1  =                1533.                                                  
CRPIX2  =                1550.                                                  
CRVAL1  =     323.492794193632                                                  
CRVAL2  =    -2.06396933577621                                                  
EQUINOX =                2000.                                                  
EPOCH   =                2000.                                                  
HARDEDGE=                  370                                                  
SOFTEDGE=                  350                                                  
ORIGIN  = 'Million Concepts'                                                    
VERSION = 'v3.0.0a0'                                                            
DATE    = '2025-07-26T01:12:17+00:00'                                           
COMMENT Sky-projected bitmap of flags, where any flagged photon in that bin defi
COMMENT  nes the flag for that whole bin. Flag 1: Mission hotspot mask. Flag 2: 
COMMENT P ost-CSP ghost flag. Flag 4: Wide edge flag. Flag 8: Narrow edge flag. 
COMMENT Ex : Flag 12 means both narrow and wide edge flags are set.    
```