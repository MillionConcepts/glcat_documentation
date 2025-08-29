# Source Identification and Photometry

## Past Methods: SExtractor
Stars in GALEX images are treated as "point" sources in sky-projected space. Thorough identification of all point sources in a GALEX image is a critical step of catalog production. The methodology for point source identification has evolved from the `SExtractor` (Source-Extractor) program used by the mission's original pipeline to produce the MCAT. SExtractor was also used to identify "extended" sources (galaxies) and to run aperture photometry on output source catalogs. Separate tooling is now used in gPhoton2 for point source identification, extended source identification, and aperture photometry. 

## Point Source Identification 
A wide range of observation targets, exposure times, and two different bands makes a single solution with one set of detection thresholds an imperfect solution for GALEX. Historically, gPhoton used `photutils` implementation of the DAOFIND algorithm, `DAOStarFinder`, to identify point sources in count images. `DAOStarFinder` identifies sources with values over a user-set threshold and with a shape close to a 2D Gaussian Kernel set to be the approximate size of stars in the image. This works pretty well, but has issues with over or under source identification in sparse images, as well as images with variable background. 

### Method Overview 
The current methodology for point source identification is as follows: 
1. **Masking:** The full-depth count image is masked to only show the are in view for the whole observation ("full" coverage in the coverage backplane). Hotspots and coldspots are also masked.
2. **Background Estimation:** We estimate the background of the count image by dividing the image into 15 × 15 pixel boxes and, within each box, applying 3σ sigma-clipping to exclude bright sources or outliers before computing the median background value with `Background2D`. These local estimates are then smoothed with a 21 × 21 pixel filter. The background image is subtracted directly from the input count image and the root mean square (RMS) of the background is also computed and passed on to form the threshold image.
3. **Threshold Image:** The threshold image will be used in step (5) as the cutoff for the count image, above which sources can be identified. The threshold image is derived from the background RMS image from the previous step, but with two modifications. Values below a "minimum" for the threshold are set to the minimum value. The image is then multiplied by a "multiplier" based on the number of photons used to make the image. Threshold setting is discussed more in the next section.
5. **Image Convolution:** The count image and hotspot mask are convolved using a 2D gaussian kernel with a FWHM of 3 pixels and a size of 3 x 3 pixels.
6. **Source Detection:** The convolved count image and mask are fed to `photutils` `detect_sources` which uses detects sources above the threshold image using image segmentation. A minimum of 2 connected pixels were required to qualify as a source. A segmentation image with the same shape as the count image is returned with pixels belonging to a source marked with an integer value representing each source.
   
```{image} figures/segmentation.png
:alt: segments
:width: 700px
:align: center
```
*Figure: A segmentation image produced by `detect_sources` for eclipse 23456. Bright / large sources are labeled first and then other sources are indexed from top to bottom, as you can see in the color of the sources in the segmentation image.*

7. **Source Deblending:** The segmentation image is deblended using `deblend_sources` which uses multi-thresholding to separate connected sources. To be deblended, the new source must have at least 3 connected pixels and the local peaks must have at least a 7.5 magnitude difference.
8. **Source Outlining:** Then an image holding the outline of each source in the segmentation image is made using `outline_segments`. This is used for identifying point sources within extended sources (see below).
9. **Source Catalog:** The segmentation image is converted to a catalog of sources using `SourceCatalog`. The `SourceCatalog` class returns many properties of each source that are unvetted, such as ellipticity, eccentricity, and the flux of all pixels identified as a source. Properties other than the source location are not propagated to the band catalogs of GLCAT. 

### Threshold Setting 
We also tune the threshold image used in source finding to adapt with the number of photons in each eclipse. The number of photons is a good metric for 1) the exposure time of an eclipse and 2) how crowded the FOV is. Previous iterations of source finding used exposure time for scaling the threshold, but that required setting different thresholds for FUV and NUV images and did not account for what the FOV contained. Only photons that make it into the count image are used in calculating the threshold. However, the count is still slightly inflated by the unavoidable inclusion of photons in hotspots and the partial coverage area around the edge of the image, which are masked during source finding. 

```{image} figures/e4356_sourcethresh.png
:alt: threshold
:width: 700px
:align: center
```
*Figure: The estimated background of an eclipse's FUV band strongly reflects the mission flat.*

The background RMS image works well as a threshold when there are enough photons to have a complete background, but at low exposure times the background RMS is frequently at or close to 0. Therefore, we set a minimum value that replaces all values below the minimum in the background RMS image. This minimum value scales with photon count and is only really important at lower exposure times and/or lower photon counts, as well as areas in which the detector is locally less sensitive (see figure above). The transition point at which the minimum seems to no longer be as important is around 800 seconds exposure time, or around 6-7 million photons. This is approximately the same as the number of photons required to fill every pixel of a GALEX count image when we assume the radius of the image is 1450 pixels: $\pi \cdot 1450^2 \approx 6{,}605{,}198.55$ photons. 

#### Minimum Threshold and Threshold Multiplier  
The minimum threshold value is based on the relationship between photon count and the median value of the background RMS image for ~1200 eclipses (see figure below). The minimum is higher than the median value of the background RMS at low photon counts and lower at high photon counts. 
$$
\text{minimum} = 0.5 \cdot \frac{b \, \bigl(\sqrt{\text{photons}} - c \bigr)}{1 + \lvert b \, (\sqrt{\text{photons}} - c) \rvert} + 0.53,
\quad b = 0.00182538272,\; c = 1058.67793
$$

The "multiplier" is also higher at low photon counts and decreases with higher photon counts where the background is more developed. The higher multiplier is most important for the intermediate photon count eclipses where there are still “holes” in the background and the background is more dictated by instrument sensitivity.

$$
\text{multiplier} = -0.14 \cdot \ln(\text{photons}) + 4.0
$$


```{image} figures/min_mult_sourcefinding.png
:alt: minmult
:width: 500px
:align: center
```
*Figure: How photon count scales with the median RMS of the background of 1200 eclipses, plotted with the multiplier and minimum threshold equations based on this relationship.*

There is one scenario where we modify the multiplier and minimum to adapt for the FOV. We increase the threshold multiplier for background-dominant arrays where detector sensitivity inequalities may be more obvious. Background-dominant arrays generally have a photon count to exposure time ratios of less than 15000, so relatively low photon counts for a given exposure time. We increase the minimum by .05, and by an additional .1 and .75 for the multiplier if the multiplier is below 1.8. Otherwise, discrepancies in the background resulting from the detector result in whole regions being marked as thousands of false point sources. 

The minimum is applied first, followed by the multiplier. 

    bkg_rms[bkg_rms < minimum] = minimum
    threshold = np.multiply(multiplier, bkg_rms)

## Extended Source Identification 

Extended source identification has a wider range of targets it looks to identify than point source. Extended sources generally consist of 1) multiple point sources and 2) are not point-source shaped (~small circle) but the scale can differ significantly from most of the FOV (1.2 degrees) to tens of pixels (1.5 arcsec/pixel). The new method employed by gPhoton2 for extended source identification takes the theory of an “extended source is a collection of point sources” to the extreme by identifying bright areas (not necessarily stars) with loose DaoStarFinder settings and clustering them with DBSCAN. `Photutils` `DaoStarFinder` uses the DAOFIND algorithm (Stetson 1987) to identify point sources by finding local maximums and attempting to fit a gaussian to them. 

For extended sources we run `DAOStarFinder` twice on the count image. The "minimum" value used for the threshold in point source identification is used as the threshold with `DAOStarFinder` but divided by 80 for one run and 110 for another. This much lower threshold allows more sources than really exist to be identified-- we really want any local maximums to have a point assigned to them. The FWHM of the gaussian kernels used by the two star finder runs are 5 and 3 pixels, respectively. The results of the loose DAOStarFinder runs are thousands to hundreds of thousands of ‘stars’. Usually (with the exception of very sparse fields) there are many more sources ID’d by DaoStarFinder than by the image segmentation used in point source finding. 

We feed these ‘stars’ to a grouping algorithm, `DBSCAN`, because dense clusters of many points point to an area likely being an extended source. The specific implementation of DBSCAN we use is from `sci-kit learn`. DBSCAN makes no presumptions about the shape of clusters or the number of clusters in a set of points, which is good because extended sources have different shapes and there may be multiple or none in an image.  DBSCAN looks at each point in your dataset, and if a point is within distance "epsilon" of another point, those points are in the same group. All points within a group are within epsilon distance of another point in the group, but not all points are at most epsilon apart. With this method, bright areas like galaxies end up with large groups of points that are identified as a cluster by DBSCAN if the epsilon is set to an appropriate distance. 

### Setting an Appropriate Epsilon for DBSCAN  

The maximum distance between points for them to be grouped as a cluster by DBSCAN is epsilon. Originally we had epsilon set as a static value for NUV and FUV (40 and 50 pixels respectively). This was pretty good, but could go ballistic in eclipses with crowded FOV or bright backgrounds. 

The theory for setting an ideal epsilon value for an eclipse is: 
If the number of points is equally distributed about the FOV, the minimum epsilon value for all points to be considered part of the same group (aka the highest distance between all points' closest neighbor) is too high for the epsilon setting for extended source finding. Therefore the ideal epsilon value is just below the highest distance between all points' closest neighbor. 

Using this theory as guide, we tested a variety of epsilon values on eclipses with a number of DAO sources ranging from close to 0 to 6,000,000 sources. Based on how well extended sources were identified in the results, we chose an ideal epsilon value for each eclipse. The points in the figure below show the results of that test.  

```{image} figures/extended_epsilon.png
:alt: epsilon
:width: 500px
:align: center
```
*Figure: The ideal epsilon value is strongly correlated with the number of points identified by DAOStarFinder.*

We used the results of the ideal epsilon test to hand-fit an ideal epsilon equation for all eclipses based on the number of DAO sources identified, shown by the orange line in the figure above. The equation for the epsilon value used by gPhoton2 is as follows, where 1500 is approximately the radius of the count image: 
$$
\epsilon = \left(\frac{1500^{2}}{\text{Num. DAO points}}\right)^{0.6}
$$
Notably this is pretty close to the hypothetical best epsilon value based on the distance between equally distributed points around the image (we are approximating here, this can be a complicated packing-fraction problem if we want it to be, but we don't). 
$$
\epsilon = \sqrt{\frac{1500^{2}}{\text{Num. DAO points}}}
$$

Because we don’t want only a handful of DAO points to be identified as an extended source when they’re scattered across the image, we set a cap on the epsilon value at low point counts. When the number of points < 1514, epsilon is set to 80. 1514 is approx when the equation for epsilon hits 80. Additionally, over 250,000 DAO points, we set epsilon to 5 and cut the number of points used in DBSCAN to 250,000, evenly distributed. 

### Outlining Extended Sources 

Large stars frequently have several DAO points clustered on them. To try and filter as many of those out of the extended source results as possible, we cut out DBSCAN groups with 11 or less DAO points. Any resulting DBSCAN groups have a convex hull drawn around all points in the group, resulting in a set of vertices defining the shape of the extended source in image coordinates. These vertices are propagated to the extended source catalog file. 

### Extended Source Properties 

After creating a catalog of extended source outlines, we check what point sources identified in Point Source Identification [link that] are present in an extended source using the "outline segmentation image" generated as part of image segmentation. All sources identified within an extended source are tagged in their point source catalog with that extended source's ID. 

Two metrics are then added to the extended source catalog: the star count (`source_count`) and density of stars within the extended source (`area_density`). `Area_density` is the sum of the `area` column in the point source catalog for all stars identified as within an extended source divided by the `hull_area` from the extended source catalog. Both the star count and the star density of an extended source are good metrics for filtering extended sources. Large extended sources with lots of stars will have high star counts and relatively high density, while only 1 or 2 stars in an extended source with high density means it was likely a single star falsely identified as an extended source. We filter out the second type of extended source by only keeping those with area densities above .15 and at least 4 unique stars, or at least 30 unique stars with whatever density. 


```{image} figures/extended_e8719.png
:alt: e8719
:width: 800px
:align: center
```
*Figure: High source counts and a large area for the spiral galaxy in the middle differentiate it from other "extended" sources that are most likely point sources in the NUV, and are not identified as extended in the FUV.*


### Artifacts as Extended Sources 

Some artifacts, like edge reflections, look a lot like extended sources. It is important to double check in the image that your extended source is not an artifact. Proximity to the edge of the image is also a good proxy for viewing the image, although it will not cover all artifact types. In the figure above you can see one edge reflection in the NUV is identified as an extended source. 

## Aperture Photometry 

gPhoton2 intakes a list of requested aperture sizes. Aperture sizes and point source locations in projected image coordinates are used to make a set of circular apertures (`photutils` `CircularAperture` class). `Photutils` aperture photometry tool (`aperture_photometry`) takes the circular apertures and an input image and returns a photometry table with the sum of the the image values within the aperture of each source (aka the `aperture_sum`). The photometry table also includes the sky coordinates of each source and the uncertainty of the aperture sum, `aperture_sum_err`. 

gPhoton2 uses `aperture_photometry` with the `exact` method for determing aperture sum when the aperture only partially overlaps pixels. This means the exact fractional overlap between aperture and pixel is used as a weight when calculating the aperture sum. While this method is the slowest way to use `aperture_photometry`, we use it for all aperture photometry in gPhoton2. 