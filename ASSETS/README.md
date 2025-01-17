
# SHAPEFILES TO GEOJSON FILES

Geojson files are useful to use in analysis because they combine all the files necessary for spatial data into one file. Simple yet versatile. Step 2 will show how US census data for this project was downloaded, merged, and converted into geojson files.

```{r setup, include=FALSE}

# global chunk options
knitr::opts_chunk$set(echo = TRUE, warning=F, message=F, fig.width=10)

library( rgdal )
library( geojsonio )
library( raster )
library( tidyr )
library( plyr )
library( tigris )
library( rgeos )
library( repmis )
library( maptools )

```


# DOWNLOAD SHAPEFILES 

Begin by using functions from the tigris package to directly download selected census tract shapefiles. Using this package only requires using a function to download shapefiles into R, making it very simple to use.

```{r}

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

Add Syracuse MSA census tract data to census tract shape files

```{r}

# Read in census data

source_data("https://github.com/lecy/analyzing-nonprofit-service-areas/blob/master/ASSETS/acs_2015_syr.Rda?raw=true")

```

## Clean census data GEOID

The data downloaded from the census API includes values in GEOID cells that need to be removed prior to merging with shapefiles.

```{r}

head(acs_2015_syr$GEOID)

acs_2015_syr$GEOID <- gsub(acs_2015_syr$GEOID, pattern="14000US", replacement="")
census_dat <- acs_2015_syr

head(acs_2015_syr$GEOID)
```

## Merge 3 spatial files

When the shapefiles are joined, they do not create a full list variable of all GEOIDs. This step creates one GEOID variabe combined from all three shapefiles.

```{r}

# Spatial join
syr_msa <- union(Mad_county, On_county)
syr_msa <- union(syr_msa, Osw_county)

# Create complete 

GEOID <- data.frame(syr_msa$GEOID.1, syr_msa$GEOID.2, syr_msa$GEOID)
head(GEOID)
GEOID_tot <- GEOID[!is.na(GEOID)]
GEOID$GEOID_tot <- GEOID_tot
head(GEOID)

syr_msa$GEOID_full <- GEOID_tot

# syr_msa <- cbind( Mad_county, On_county )
# syr_msa <- cbind( syr_msa, Osw_county )
# Does not work, Error: arguments imply differing number of rows: 16, 140

plot(syr_msa)

```

## Merge census attribute data with shapefile data

Merge Syracuse MSA data from [Step 1](Step_1_-_Download_and_Clean_Data.html). Occasionally, some census tract may need to be dropped because they only cover bodies of water with no other census information. In this MSA, one body of water census tract is deleted from the data. 

```{r}

# Merge data with MSA shp file

syr_merged <- merge(syr_msa, census_dat, 
                    by.x = "GEOID_full", by.y = "GEOID")                      

plot(syr_merged)
title(main = "Syracuse MSA, NY")
head(syr_merged)
nrow(syr_merged)

# Drop census tract 9900, water census tract in Oswego county
syr_merged_clean <- syr_merged[!(syr_merged$tract=="990000"),]
nrow(syr_merged)

plot(syr_merged_clean)
title(main = "Syracuse MSA, NY")

```


# GEOJSON FILES

Convert shapefiles to geojson files for the Syracuse MSA. This function also saves the created geojson file to a chosen folder. 

```{r}
geojson_write(syr_merged_clean, geometry="polygon", file="../ASSETS/syr_merged.geojson")

```

Create a shapefile with centroids for every census tract and convert to geojson format. Save file.

```{r}

syr_merged_cen = gCentroid(syr_merged_clean, byid=TRUE)

geojson_write(syr_merged_cen, geometry="point", file="../ASSETS/syr_merged_cen.geojson")

```

Load poverty nonprofit data. Drop missing values, convert shapefiles for poverty organizations in a geojson file. Save file.

```{r}

load("../ASSETS/pov_orgs_gps.Rda")

# Missing values are dropped to insure smooth transformation into a spatial object

pov_orgs_gps_nona <- na.omit(pov_orgs_gps)

geojson_write(pov_orgs_gps_nona, geometry="point", file="../ASSETS/pov_orgs.geojson")

```
 


