# Shapefiles to geojson files
Matt McKnight  
May 16, 2017  

# SHAPEFILES TO GEOJSON FILES




### Begin by setting working directory and reading into R the layer file for each census tract file


```r
setwd( "../SHAPEFILES/" )

Mad_county <- readOGR(dsn="Madison-ct", layer="tl_2010_36053_tract10")
```

```
## OGR data source with driver: ESRI Shapefile 
## Source: "Madison-ct", layer: "tl_2010_36053_tract10"
## with 16 features
## It has 12 fields
## Integer64 fields read as strings:  ALAND10 AWATER10
```

```r
On_county <- readOGR(dsn="Onondaga-ct", layer="tl_2010_36067_tract10")
```

```
## OGR data source with driver: ESRI Shapefile 
## Source: "Onondaga-ct", layer: "tl_2010_36067_tract10"
## with 140 features
## It has 12 fields
## Integer64 fields read as strings:  ALAND10 AWATER10
```

```r
Osw_county <- readOGR(dsn="Oswego-ct", layer="tl_2010_36075_tract10")
```

```
## OGR data source with driver: ESRI Shapefile 
## Source: "Oswego-ct", layer: "tl_2010_36075_tract10"
## with 30 features
## It has 12 fields
## Integer64 fields read as strings:  ALAND10 AWATER10
```

### Then simply use the "geojson_write" command to convert orginal shapefiles into a singles geojson file


```r
geojson_write( Mad_county, geometry="polygon", file="Mad_county.geojson" )
```

```
## <geojson>
##   Path:       Mad_county.geojson
##   From class: SpatialPolygonsDataFrame
```

```r
class(Mad_county)
```

```
## [1] "SpatialPolygonsDataFrame"
## attr(,"package")
## [1] "sp"
```

```r
geojson_write( On_county, geometry="polygon", file="On_county.geojson" )
```

```
## <geojson>
##   Path:       On_county.geojson
##   From class: SpatialPolygonsDataFrame
```

```r
class(On_county)
```

```
## [1] "SpatialPolygonsDataFrame"
## attr(,"package")
## [1] "sp"
```

```r
geojson_write( Osw_county, geometry="polygon", file="Osw_county.geojson" )
```

```
## <geojson>
##   Path:       Osw_county.geojson
##   From class: SpatialPolygonsDataFrame
```

```r
class(Osw_county)
```

```
## [1] "SpatialPolygonsDataFrame"
## attr(,"package")
## [1] "sp"
```

### Geojson files can then be plotted using the plot function with the names of the variables found in the respective shapefile.


```r
plot(Mad_county)
title(main = "Census Tracts in Madison County, NY")
```

![](Shapefiles_to_Geojson_Files_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
names(Mad_county)
```

```
##  [1] "STATEFP10"  "COUNTYFP10" "TRACTCE10"  "GEOID10"    "NAME10"    
##  [6] "NAMELSAD10" "MTFCC10"    "FUNCSTAT10" "ALAND10"    "AWATER10"  
## [11] "INTPTLAT10" "INTPTLON10"
```

```r
plot(On_county)  
title(main = "Census Tracts in Onondaga County, NY")
```

![](Shapefiles_to_Geojson_Files_files/figure-html/unnamed-chunk-3-2.png)<!-- -->

```r
names(On_county)
```

```
##  [1] "STATEFP10"  "COUNTYFP10" "TRACTCE10"  "GEOID10"    "NAME10"    
##  [6] "NAMELSAD10" "MTFCC10"    "FUNCSTAT10" "ALAND10"    "AWATER10"  
## [11] "INTPTLAT10" "INTPTLON10"
```

```r
plot(Osw_county)  
title(main = "Census Tracts in Oswego County, NY")
```

![](Shapefiles_to_Geojson_Files_files/figure-html/unnamed-chunk-3-3.png)<!-- -->

```r
names(Osw_county)
```

```
##  [1] "STATEFP10"  "COUNTYFP10" "TRACTCE10"  "GEOID10"    "NAME10"    
##  [6] "NAMELSAD10" "MTFCC10"    "FUNCSTAT10" "ALAND10"    "AWATER10"  
## [11] "INTPTLAT10" "INTPTLON10"
```
