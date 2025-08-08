# Image File Formats



## Dose Maps
Some text.

![Dose map example](figures/dose_maps.png)
*Figure: Example of a dose map image.*

[image headers](tables/image_metadata.md)

## Count Maps

![](figures/cnt_images.png)
*Figure: Example of a count map image.*

## Flag Maps

```
COMMENT Sky-projected bitmap of flags, where any flagged photon in that bin defi
COMMENT  nes the flag for that whole bin. Flag 1: Mission hotspot mask. Flag 2: 
COMMENT P ost-CSP ghost flag. Flag 4: Wide edge flag. Flag 8: Narrow edge flag. 
COMMENT Ex : Flag 12 means both narrow and wide edge flags are set.      
```

## Coverage

```
COMMENT Aspect-derived maps in sky-projected space. Coverage varies due to dithe
COMMENT  r patterns. Full coverage during observation: 1. Partial coverage durin
COMMENT g  observation: 2. 
```

## Integrated Images



### Movies

![](figures/movie_cnt.gif)
*Figure: 120-second movies*