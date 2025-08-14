# Basic Photometry

## Flux Calculation

The `aperture_sum[...]` columns report the sum of response-corrected photon events within the aperture centered on the `[xy]center` position in the image. This calculation uses the ["exact" sub-pixel interpolation capability of `photutils`](https://photutils.readthedocs.io/en/stable/api/photutils.aperture.aperture_photometry.html) to estimate the fractional contributions of pixels that overlap the edge of the aperture.[^myref]

[^myref]: The effective aperture area is therefore exactly `pi*r^2`.

The cumulative counts for the _full depth_ observation is given in the column named `aperture_sum`. If gPhoton2 processing were requested in {term}`movie mode <movie>`, then the cumulative counts for each _frame_ will be in the columns labeled `aperture_sum_[N]` where `N={0,1,2,...,M-1}` where M is the total number of frames in the movie.

The exposure times use a parallel naming convention. The cumulative _effective_ exposure time for the observation is given in the `expt` columns and the cumulative _effective_ exposure time for each frame will be in `expt_[N]`. [The _effective_ exposure time has been corrected for detector dead time.]

To calculate the flux (in counts-per-second), divide the appropriate aperture_sum value by its corresponding expt value. The standard error (i.e. "1-sigma") on this measurement is the square root of the `aperture_sum` value divided by the expt. [sqrt(aperture_sum)/expt]

![](#aperture_photometry)

![](#dose_maps)


## Background Estimation 

### Background

[Describe the way that the mission estimated background and why we don't like it.]

### Annulus Method

An estimate of the background can be derived from the count rate of an annulus around the source position. The annulus must be small enough to capture the local background, but large enough that it is not unduly contaminated by light from the target source. Ideally, the annulus should be free from contaminating light from artifacts and other sources, and also large enough to capture significant signal. The selection of an optimal annulus for background estimation will therefore vary source-by-source, and may require visual inspection. However, we recommend that a generally good starting point for analysis is an annulus with an inner radius of 17.3" and an outer radius of 30". These radii correspond to APER5 and APER6.

To calculate the background flux, take the difference of the aperture_sum values at the two radii, and divide this by the effective area of the annulus. (So, in this case, PI*(30**2) - PI*(17.3**2) ~= 1887.19 arcseconds.) Then multiply that by the area of the aperture. (If you are using APER4=12.8", then this is ~=514.71 arcseconds.) We will call this the cumulative background counts within the aperture, or aperture_sum_bg (for ease of reference). [Restating here with code might be a good idea.] Divide this by the appropriate effective exposure time to get the estimated contribution of background within the aperture, in unites of counts-per second.

This value can be subtracted from the flux computed above to get an estimate of the background subtracted flux within the aperture. The standard error on this measurement would be sqrt(aperture_sum_bg)/expt. To calculate the error on this background-subtracted flux, add the flux and background standard errors in quadrature.

## Aperture Correction

The aperture correction accounts for flux not measured because it fell outside of the aperture, in the tails of the PSF. For any 2D Gaussian with a given FWHM, the fraction that falls outside of a circle centered on its peak will be a constant (e.g. 10%). The correction can therefore be applied by multiplying the measured flux in linear units (e.g. 1.1x the counts-per-second) or by subtracting a value in logarithmic (magnitude) units (e.g. AB Magnitude - 0.1). In well-behaved imaging systems, the PSF is consistent across the detector. In somewhat well-behaved imaging systems, the PSF is at least constant across time and scene composition. The GALEX detectors were not well-behaved in these ways. As discussed elsewhere [LINK], the GALEX point spread function varies as a complicated function of (at minimum) detection position, observation mode, observation depth, source brightness, and field brightness.

There were two aperture correction tables published. [... We don't believe either of them.] The aperture correction table are therefore—and only every believed to be—only average estimates.

[Were the aperture corrections also derived from LDS749b? That is another reason to not trust them. But saturation and non-representative observing mode.]

