# Shapefiles to geojson files
Matt McKnight  
May 16, 2017  

# SHAPEFILES TO GEOJSON FILES

Geojson files are useful to use in analysis because they combine all the files necessary for spatial data into one file. Simple but effective. This tutorial will show how US census data for this project was downloaded, merged, and converted into geojson files.



# DOWNLOAD SHAPEFILES 
### Begin by using functions from the tigris package to directly download selected census tract shapefiles.


```r
# How to download shapefiles from the census directly using tigris functions
# https://rdrr.io/cran/tigris/man/blocks.html

Mad_county <- tracts(state = "NY", county = "053", year = "2015")

On_county <- tracts(state = "NY", county = "067", year = "2015")

Osw_county <- tracts(state = "NY", county = "075", year = "2015")

# Drop NAs
Mad_county <- na.omit(Mad_county)
On_county <- na.omit(On_county)
Osw_county <- na.omit(Osw_county)
```


# MERGE DATA
### Add Syracuse MSA census tract data to census tract shape files


```r
# Read in census data
source_data("https://github.com/lecy/analyzing-nonprofit-service-areas/blob/master/DATA/acs_2015_syr.Rda?raw=true")
```

```
## [1] "acs_2015_syr"
```

### Clean census data GEOID

The data downloaded from the census API includes values in GEOID cells that need to be removed prior to merging with shapefiles.

```r
# Search term: deleting part of charactor in factor variable in r
# http://stackoverflow.com/questions/5487164/r-how-to-replace-parts-of-variable-strings-within-data-frame 

census_dat <- as.data.frame(sapply(acs_2015_syr,gsub,pattern="14000US",replacement=""))
```

### Merge 3 spatial files


```r
syr_msa <- union(Mad_county, On_county)
syr_msa <- union(syr_msa, Osw_county)
plot( syr_msa )
```

![](Shapefiles_to_Geojson_Files_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

### Clean census data GEOID to match shapefile GEOID

Depending on what year is downloaded, the GEOIDs from census attribute data may not match with GEOID values in TIGER shapefiles. Below shows a procedure that may be used to ensure the values in both sets of data are identical and ready to be merged.

```r
# Combine 3 GEOID variables in syr_msa in to one column for merge

# Example

x <- c(1,1,1,NA,NA,NA,NA,NA,NA)
y <- c(NA,NA,NA,1,1,1,NA,NA,NA)
z <- c(NA,NA,NA,NA,NA,NA,1,1,1)
df <- data.frame(x,y,z)
# unite isn't quite what we are looking for because it adds the NAs to the single created column.
unite(df, a, x, y, z, sep = "", remove = FALSE)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["a"],"name":[1],"type":["chr"],"align":["left"]},{"label":["x"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["y"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["z"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"1NANA","2":"1","3":"NA","4":"NA"},{"1":"1NANA","2":"1","3":"NA","4":"NA"},{"1":"1NANA","2":"1","3":"NA","4":"NA"},{"1":"NA1NA","2":"NA","3":"1","4":"NA"},{"1":"NA1NA","2":"NA","3":"1","4":"NA"},{"1":"NA1NA","2":"NA","3":"1","4":"NA"},{"1":"NANA1","2":"NA","3":"NA","4":"1"},{"1":"NANA1","2":"NA","3":"NA","4":"1"},{"1":"NANA1","2":"NA","3":"NA","4":"1"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
#But, a trick can be applied by dropping NAs:
df <- df[!is.na(df)]
# Values become a single vector that can be added back to a data.frame
df
```

```
## [1] 1 1 1 1 1 1 1 1 1
```

```r
# Actual 
GEOID <- data.frame(syr_msa$GEOID.1, syr_msa$GEOID.2, syr_msa$GEOID)
head(GEOID)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["syr_msa.GEOID.1"],"name":[1],"type":["fctr"],"align":["left"]},{"label":["syr_msa.GEOID.2"],"name":[2],"type":["fctr"],"align":["left"]},{"label":["syr_msa.GEOID"],"name":[3],"type":["fctr"],"align":["left"]}],"data":[{"1":"36053030101","2":"NA","3":"NA"},{"1":"36053030200","2":"NA","3":"NA"},{"1":"36053030103","2":"NA","3":"NA"},{"1":"36053030102","2":"NA","3":"NA"},{"1":"36053031000","2":"NA","3":"NA"},{"1":"36053031100","2":"NA","3":"NA"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
GEOID_tot <- GEOID[!is.na(GEOID)]

GEOID$GEOID_tot <- GEOID_tot
GEOID_fixed <- GEOID_tot[c(
1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128, 129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,147,148, 149,150,151,16,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168, 169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186
 )]

GEOID$GEOID_fixed <- GEOID_fixed

syr_msa$GEOID_full <- GEOID$GEOID_fixed

#check to see if census GEOID is the same as shapefile MSA GEOID_full column:
test <- data.frame(census_dat$GEOID, syr_msa$GEOID_full)
a <- as.vector(test$census_dat.GEOID)
b <- as.vector(test$syr_msa.GEOID_full)
a <- sort(a)
b <- sort(b)
test <- data.frame(a,b)
identical(test[['census_dat.GEOID']],test[['syr_msa.GEOID_full']])
```

```
## [1] TRUE
```


### Merge census attribute data with shapefile data


```r
# Merge data with MSA shp file
syr_merged <- geo_join(syr_msa, census_dat, "GEOID_full", "GEOID")
plot( syr_merged )
title(main = "Syracuse MSA, NY")
```

![](Shapefiles_to_Geojson_Files_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
head(syr_merged)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["STATEFP.1"],"name":[1],"type":["chr"],"align":["left"]},{"label":["COUNTYFP.1"],"name":[2],"type":["chr"],"align":["left"]},{"label":["TRACTCE.1"],"name":[3],"type":["chr"],"align":["left"]},{"label":["GEOID.1"],"name":[4],"type":["chr"],"align":["left"]},{"label":["NAME.1"],"name":[5],"type":["chr"],"align":["left"]},{"label":["NAMELSAD.1"],"name":[6],"type":["chr"],"align":["left"]},{"label":["MTFCC.1"],"name":[7],"type":["chr"],"align":["left"]},{"label":["FUNCSTAT.1"],"name":[8],"type":["chr"],"align":["left"]},{"label":["ALAND.1"],"name":[9],"type":["chr"],"align":["left"]},{"label":["AWATER.1"],"name":[10],"type":["chr"],"align":["left"]},{"label":["INTPTLAT.1"],"name":[11],"type":["chr"],"align":["left"]},{"label":["INTPTLON.1"],"name":[12],"type":["chr"],"align":["left"]},{"label":["STATEFP.2"],"name":[13],"type":["chr"],"align":["left"]},{"label":["COUNTYFP.2"],"name":[14],"type":["chr"],"align":["left"]},{"label":["TRACTCE.2"],"name":[15],"type":["chr"],"align":["left"]},{"label":["GEOID.2"],"name":[16],"type":["chr"],"align":["left"]},{"label":["NAME.2"],"name":[17],"type":["chr"],"align":["left"]},{"label":["NAMELSAD.2"],"name":[18],"type":["chr"],"align":["left"]},{"label":["MTFCC.2"],"name":[19],"type":["chr"],"align":["left"]},{"label":["FUNCSTAT.2"],"name":[20],"type":["chr"],"align":["left"]},{"label":["ALAND.2"],"name":[21],"type":["chr"],"align":["left"]},{"label":["AWATER.2"],"name":[22],"type":["chr"],"align":["left"]},{"label":["INTPTLAT.2"],"name":[23],"type":["chr"],"align":["left"]},{"label":["INTPTLON.2"],"name":[24],"type":["chr"],"align":["left"]},{"label":["STATEFP"],"name":[25],"type":["chr"],"align":["left"]},{"label":["COUNTYFP"],"name":[26],"type":["chr"],"align":["left"]},{"label":["TRACTCE"],"name":[27],"type":["chr"],"align":["left"]},{"label":["GEOID"],"name":[28],"type":["chr"],"align":["left"]},{"label":["NAME"],"name":[29],"type":["chr"],"align":["left"]},{"label":["NAMELSAD"],"name":[30],"type":["chr"],"align":["left"]},{"label":["MTFCC"],"name":[31],"type":["chr"],"align":["left"]},{"label":["FUNCSTAT"],"name":[32],"type":["chr"],"align":["left"]},{"label":["ALAND"],"name":[33],"type":["chr"],"align":["left"]},{"label":["AWATER"],"name":[34],"type":["chr"],"align":["left"]},{"label":["INTPTLAT"],"name":[35],"type":["chr"],"align":["left"]},{"label":["INTPTLON"],"name":[36],"type":["chr"],"align":["left"]},{"label":["GEOID_full"],"name":[37],"type":["chr"],"align":["left"]},{"label":["NAME.3"],"name":[38],"type":["fctr"],"align":["left"]},{"label":["GEOID.3"],"name":[39],"type":["fctr"],"align":["left"]},{"label":["state"],"name":[40],"type":["fctr"],"align":["left"]},{"label":["county"],"name":[41],"type":["fctr"],"align":["left"]},{"label":["tract"],"name":[42],"type":["fctr"],"align":["left"]},{"label":["mdn_hous_val"],"name":[43],"type":["fctr"],"align":["left"]},{"label":["tenure_tot"],"name":[44],"type":["fctr"],"align":["left"]},{"label":["tenure_own"],"name":[45],"type":["fctr"],"align":["left"]},{"label":["tenure_rent"],"name":[46],"type":["fctr"],"align":["left"]},{"label":["tot_occup"],"name":[47],"type":["fctr"],"align":["left"]},{"label":["occupied"],"name":[48],"type":["fctr"],"align":["left"]},{"label":["vacant"],"name":[49],"type":["fctr"],"align":["left"]},{"label":["tot_pop"],"name":[50],"type":["fctr"],"align":["left"]},{"label":["mhh_income"],"name":[51],"type":["fctr"],"align":["left"]},{"label":["poverty"],"name":[52],"type":["fctr"],"align":["left"]},{"label":["labor_part"],"name":[53],"type":["fctr"],"align":["left"]},{"label":["unemploy"],"name":[54],"type":["fctr"],"align":["left"]},{"label":["high_sch"],"name":[55],"type":["fctr"],"align":["left"]},{"label":["ged"],"name":[56],"type":["fctr"],"align":["left"]},{"label":["tot_race"],"name":[57],"type":["fctr"],"align":["left"]},{"label":["white"],"name":[58],"type":["fctr"],"align":["left"]},{"label":["black"],"name":[59],"type":["fctr"],"align":["left"]},{"label":["am_ind"],"name":[60],"type":["fctr"],"align":["left"]},{"label":["asian"],"name":[61],"type":["fctr"],"align":["left"]},{"label":["islander"],"name":[62],"type":["fctr"],"align":["left"]},{"label":["other"],"name":[63],"type":["fctr"],"align":["left"]},{"label":["mixed"],"name":[64],"type":["fctr"],"align":["left"]},{"label":["not_hispanic"],"name":[65],"type":["fctr"],"align":["left"]},{"label":["hispanic"],"name":[66],"type":["fctr"],"align":["left"]},{"label":["porp_white"],"name":[67],"type":["fctr"],"align":["left"]},{"label":["porp_m"],"name":[68],"type":["fctr"],"align":["left"]}],"data":[{"1":"36","2":"053","3":"030101","4":"36053030101","5":"301.01","6":"Census Tract 301.01","7":"G5020","8":"S","9":"2775435","10":"0","11":"+43.0982901","12":"-075.6497099","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"36053030101","38":"Census Tract 301.01, Madison County, New York","39":"36053030101","40":"36","41":"053","42":"030101","43":"81100","44":"982","45":"419","46":"563","47":"1206","48":"982","49":"224","50":"2759","51":"42813","52":"792","53":"1991","54":"147","55":"439","56":"182","57":"2759","58":"2499","59":"77","60":"63","61":"16","62":"0","63":"0","64":"4","65":"2659","66":"100","67":"0.905762957593331","68":"0.0942370424066691"},{"1":"36","2":"053","3":"030200","4":"36053030200","5":"302","6":"Census Tract 302","7":"G5020","8":"S","9":"79435467","10":"46792","11":"+43.1170389","12":"-075.7631608","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"36053030200","38":"Census Tract 302, Madison County, New York","39":"36053030200","40":"36","41":"053","42":"030200","43":"110900","44":"1508","45":"1229","46":"279","47":"1796","48":"1508","49":"288","50":"3656","51":"40682","52":"709","53":"3057","54":"61","55":"1193","56":"188","57":"3656","58":"3335","59":"66","60":"2","61":"11","62":"0","63":"0","64":"232","65":"3646","66":"10","67":"0.912199124726477","68":"0.087800875273523"},{"1":"36","2":"053","3":"030103","4":"36053030103","5":"301.03","6":"Census Tract 301.03","7":"G5020","8":"S","9":"48107412","10":"211866","11":"+43.0704480","12":"-075.6735612","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"36053030103","38":"Census Tract 301.03, Madison County, New York","39":"36053030103","40":"36","41":"053","42":"030103","43":"134800","44":"1354","45":"1050","46":"304","47":"1426","48":"1354","49":"72","50":"3733","51":"56987","52":"158","53":"3143","54":"49","55":"720","56":"35","57":"3733","58":"3362","59":"60","60":"99","61":"30","62":"0","63":"0","64":"46","65":"3597","66":"136","67":"0.900616126439861","68":"0.0993838735601393"},{"1":"36","2":"053","3":"030102","4":"36053030102","5":"301.02","6":"Census Tract 301.02","7":"G5020","8":"S","9":"6232906","10":"0","11":"+43.0945118","12":"-075.6649580","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"36053030102","38":"Census Tract 301.02, Madison County, New York","39":"36053030102","40":"36","41":"053","42":"030102","43":"95300","44":"2070","45":"1069","46":"1001","47":"2329","48":"2070","49":"259","50":"4760","51":"31916","52":"943","53":"3922","54":"201","55":"879","56":"221","57":"4760","58":"4573","59":"0","60":"63","61":"52","62":"0","63":"0","64":"0","65":"4688","66":"72","67":"0.960714285714286","68":"0.0392857142857143"},{"1":"36","2":"053","3":"031000","4":"36053031000","5":"310","6":"Census Tract 310","7":"G5020","8":"S","9":"202183361","10":"1699120","11":"+42.8408125","12":"-075.5015238","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"36053031000","38":"Census Tract 310, Madison County, New York","39":"36053031000","40":"36","41":"053","42":"031000","43":"115000","44":"2024","45":"1637","46":"387","47":"2713","48":"2024","49":"689","50":"5230","51":"48250","52":"659","53":"4372","54":"166","55":"1309","56":"136","57":"5230","58":"4978","59":"18","60":"10","61":"3","62":"0","63":"15","64":"75","65":"5099","66":"131","67":"0.951816443594646","68":"0.0481835564053538"},{"1":"36","2":"053","3":"031100","4":"36053031100","5":"311","6":"Census Tract 311","7":"G5020","8":"S","9":"201540058","10":"507738","11":"+42.8079912","12":"-075.3432945","13":"NA","14":"NA","15":"NA","16":"NA","17":"NA","18":"NA","19":"NA","20":"NA","21":"NA","22":"NA","23":"NA","24":"NA","25":"NA","26":"NA","27":"NA","28":"NA","29":"NA","30":"NA","31":"NA","32":"NA","33":"NA","34":"NA","35":"NA","36":"NA","37":"36053031100","38":"Census Tract 311, Madison County, New York","39":"36053031100","40":"36","41":"053","42":"031100","43":"71500","44":"940","45":"804","46":"136","47":"1139","48":"940","49":"199","50":"2560","51":"45278","52":"401","53":"1969","54":"77","55":"702","56":"73","57":"2560","58":"2551","59":"0","60":"0","61":"0","62":"0","63":"0","64":"9","65":"2560","66":"0","67":"0.996484375","68":"0.00351562500000002"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


# GEOJSON FILES
### Convert shapefiles to geojson files for the Syracuse MSA


```r
geojson_write( syr_merged, geometry="polygon", file="syr_merged.geojson" )
```

```
## <geojson>
##   Path:       syr_merged.geojson
##   From class: SpatialPolygonsDataFrame
```


### Create create shapefile with centroids for every census tract and convert to geojson format


```r
syr_merged_cen = gCentroid(syr_merged,byid=TRUE)

geojson_write( syr_merged_cen, geometry="point", file="syr_merged_cen.geojson" )
```

```
## <geojson>
##   Path:       syr_merged_cen.geojson
##   From class: SpatialPoints
```


### Drop missing values, convert shapefiles for poverty organizations in a geojson file



```r
load("../DATA/pov_orgs_gps.Rda")

pov_orgs_gps_nona <- na.omit(pov_orgs_gps)

geojson_write( pov_orgs_gps_nona, geometry="point", file="../SHAPEFILES/pov_orgs.geojson" )
```

```
## <geojson>
##   Path:       ../SHAPEFILES/pov_orgs.geojson
##   From class: data.frame
```
 
