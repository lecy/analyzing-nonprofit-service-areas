
### Step 1 - Download and Clean Data ###

# INTRODUCTION

Step 1 is presented to show the process of data collection for this project. Code from step 1 includes downloading and cleaning data from two different databases. First, information on nonprofit organizations in the Syracuse metropolitan area (MSA) is downloaded from IRS information. Coordinates for each nonprofit are then matched using a Google API. Second, U.S. Census information on demographics by census tract are downloaded from the census using an API. Data created from this code is already located in the asset folder of this repository and therefore need not be run. 

```{r setup, include=FALSE}

# global chunk options
knitr::opts_chunk$set(echo = TRUE, warning=F, message=F, fig.width=10)

library( ggmap )
library( rgdal )
library( devtools )
library( censusapi )
library( plyr )
library( gdata )
library( geojsonio )
library( acs )
library( tigris )
library( sf ) 
library( sp )  
library( rgeos )
library( spatialEco ) 
library( leaflet )
library( raster )
library( repmis )
library( TSP )
library( magrittr )

```


# NONPROFIT DATA

## Create NPO Data Frame

The IRS Business Master Files (BMF) includes all types of tax exempt organizations. Therefore, the first step is to subset the data to organizations whose missions focus more directly on providing services to poverty populations. The [NTEECC codes](http://nccs.urban.org/classification/national-taxonomy-exempt-entities) provide a simple way to subset the data. Here, we will subset the BMF in the Syracuse MSA using the [most recent BMF](http://nccs-data.urban.org/data.php?ds=bmf), August, 2016.

Choosing what organizations to include in the sample will follow empirical work from [Joassart-Marcelli and Woch (2003)](http://journals.sagepub.com/doi/abs/10.1177/0899764002250007) and [Peck 2008](http://journals.sagepub.com/doi/abs/10.1177/0899764006298963).Organizations from the following NTEECC 15 categories are excluded: Arts, Culture, and Humanities (A); Environment (C); Animal Related (D); Disease/Disorders (G); Medical Research (H); Public Safety, Disaster, and Relief (M); Recreation (N); International, Foreign Affairs, and National Security (Q); Philanthropy, Voluntarism, and Grant-Making Foundations (T); Science and Technology (U); Social Science (V); Public, Society Benefit (W); Religion Related (X); Mutual/Membership Benefit Organizations (Y); and Unknown (Z). 

Relevant codes from the remaining 11 NTEECC categories are included. All human services organizations are included (P). Also included are: Adult education (B60) and educational services (B90) in education; community clinics (E32) and Public Health Programs (E70) in health; substance abuse prevention and treatment (F20) and selected community mental health centers (F30) in mental health; crime prevention (I20), rehabilitation services for offenders (I40), abuse prevention (I70), and legal services (I80) from crime; employment assistance and training (J20), and vocational rehabilitation (J30) in employment; food programs (K30), nutrition (K40), and home economics (K50) from food, agriculture, and nutrition; 
housing development, construction &  management (L20), 	housing search assistance (L30), temporary housing (L40), and	housing support (L80), from housing and shelter; youth centers and clubs (O20), adult and child matching programs (O30), and youth development programs (O50) in youth development; and community and neighborhood development (S20) in community improvement and capacity building. In addtion, we add two categories from civil rights, social action, and advocacy which are neglected in past empirical research: civil rights (R20) and intergroup and race relations (R30) organizations. 

```{r eval=FALSE}

print("Don't run me")

# Import subset from NCCS BMF Data on nonprofits in Syr MSA

source_data("https://github.com/lecy/analyzing-nonprofit-service-areas/blob/master/DATA/syr_orgs.Rda?raw=true")


# Subset - Extract poverty orgs

pov_orgs <- subset(syr_orgs, 
                        grepl("B6.", NTEECC) | 
                        grepl("B92", NTEECC) |
                        grepl("E32", NTEECC) |  
                        grepl("E7.", NTEECC) |  
                        grepl("F2.", NTEECC) |
                        grepl("F32", NTEECC) |  
                        grepl("I2.", NTEECC) |
                        grepl("I4.", NTEECC) |  
                        grepl("I7.", NTEECC) |
                        grepl("I8.", NTEECC) |  
                        grepl("J2.", NTEECC) |
                        grepl("J3.", NTEECC) |  
                        grepl("K3.", NTEECC) |  
                        grepl("K4.", NTEECC) |  
                        grepl("K5.", NTEECC) |
                        grepl("L2.", NTEECC) |
                        grepl("L3.", NTEECC) |  
                        grepl("L4.", NTEECC) |  
                        grepl("L8.", NTEECC) |  
                        grepl("O2.", NTEECC) |  
                        grepl("O3.", NTEECC) |
                        grepl("O5.", NTEECC) |  
                        grepl("P", NTEECC)   |
                        grepl("R2.", NTEECC) |  
                        grepl("R3.", NTEECC) |
                        grepl("S2.", NTEECC) 
                       )

head(pov_orgs)

```


## Geocode Poverty Organizations

Selected organizations can be geocoded and used for mapping and spatial analysis. To download coordinates from Google, you need to register for a [Google API Key](https://developers.google.com/maps/documentation/javascript/get-api-key). Below, we use a Google API key to download latitude and longitude coordinates.

```{r eval=FALSE}

print("Don't run me")

# download specific ggmap, register Google key

devtools::install_github("dkahle/ggmap")

register_google(key = 'AIzaSyAMrECdXqpOt443ms6nzh118uUsqC2lE6M',
                account_type = "premium", day_limit = 15000)

# subset orgs for gecoding address information

pov_orgs_gps <- subset(pov_orgs, select=c(EIN:INCOME))

pov_orgs_gps$whole_address <- do.call(paste, c(pov_orgs_gps[ 
                                      c("ADDRESS", "CITY","STATE",
                                        "ZIP5")], sep = ", ")) 

pov_orgs_gps <- subset(pov_orgs_gps, select=c(whole_address, EIN:INCOME))

for (i in 1:nrow(pov_orgs_gps)) {
  latlon = geocode(pov_orgs_gps[i,1]) 
  pov_orgs_gps$lon[i] = as.numeric(latlon[1])
  pov_orgs_gps$lat[i] = as.numeric(latlon[2])
}

# Make sure you find a way to then take any PO Boxes and place them in the 
# center of centroids, and only the ones that are NAs.

save(pov_orgs_gps, file="../ASSETS/pov_orgs_gps.Rda")

```


# CENSUS DATA

## Census API Package

Use a [US census key](http://api.census.gov/data/key_signup.html) to query data from the US census. Many types of data and years of data [are available](https://www.census.gov/data/developers/data-sets.html) using US Census APIs. Here, data from the American Community Survery 5 year estimate is collected using the devtools package. 26 variables of interest are downloaded from the U.S. census. 

```{r eval=FALSE}

print("Don't run me")

# Census API 
# install.packages("devtools")
# devtools::install_github("hrecht/censusapi")

censuskey <- "f8064a17832b439e690be95ab0e0ef9a78617584"

var.list <- c("NAME", 
              "B25077_001E", "B25003_001E", "B25003_002E", "B25003_003E",
              "B25002_001E", "B25002_002E", "B25002_003E", "B01003_001E",
              "B19013_001E", "B17020_002E", "B12006_001E", "B23025_005E",                      "B15003_017E", "B15003_018E",
              "B03001_001E", "B03002_003E", "B03002_004E", "B03002_005E",
              "B03002_006E", "B03002_007E", "B03002_008E", "B03002_009E", 
              "B03001_002E", "B03001_003E", 
              "GEOID" 
              )

acs_2015 <- getCensus( name="acs5",
                       vintage = 2015,
                       key = censuskey,
                       vars= var.list, 
                       region ="tract:*", 
                       regionin ="state:36" 
                     )


# subset to Syracuse

acs_2015_syr <- acs_2015[which (acs_2015$county == "067" | 
                                acs_2015$county == "075" |
                                acs_2015$county == "053"), ]

```

## Rename Variables

Variables are renamed with more intelligible names.

```{r eval=FALSE}

print("Don't run me")

# Race variables are single race, non-hispanic. For example, "white" is really "white non-hispanic." This is true for every race category listed except, obviously, hispanic.


rename( acs_2015_syr, 
       c("B25077_001E" = "mdn_hous_val", 
         "B25003_001E" = "tenure_tot", 
         "B25003_002E" = "tenure_own", 
         "B25003_003E" = "tenure_rent",
         "B25002_001E" = "tot_occup", 
         "B25002_002E" = "occupied",
         "B25002_003E" = "vacant", 
         "B01003_001E" = "tot_pop", 
         "B19013_001E" = "mhh_income",
         "B17020_002E" = "poverty",
         "B12006_001E" = "labor_part",
         "B23025_005E" = "unemploy",
         "B15003_017E" = "high_sch",
         "B15003_018E" = "ged",
         "B03001_001E" = "tot_race",
         "B03002_003E" = "white",             
         "B03002_004E" = "black", 
         "B03002_005E" = "am_ind",
         "B03002_006E" = "asian", 
         "B03002_007E" = "islander",
         "B03002_008E" = "other", 
         "B03002_009E" = "mixed",
         "B03001_002E" = "not_hispanic",
         "B03001_003E" = "hispanic")
        )

acs_2015_syr <- rename.vars(acs_2015_syr,
            c("B25077_001E", "B25003_001E", "B25003_002E", "B25003_003E",
              "B25002_001E", "B25002_002E", "B25002_003E", "B01003_001E",
              "B19013_001E", "B17020_002E", "B12006_001E", "B23025_005E",                      "B15003_017E", "B15003_018E",
              "B03001_001E", "B03002_003E", "B03002_004E", "B03002_005E",
              "B03002_006E", "B03002_007E", "B03002_008E", "B03002_009E", 
              "B03001_002E", "B03001_003E"),  
            c("mdn_hous_val", "tenure_tot", "tenure_own", "tenure_rent",
              "tot_occup", "occupied", "vacant", "tot_pop", 
              "mhh_income", "poverty", "labor_part", "unemploy",
              "high_sch", "ged",
              "tot_race", "white", "black", "am_ind",
              "asian", "islander", "other", "mixed",
              "not_hispanic", "hispanic")
            )

head(acs_2015_syr)
```

## Create variables 

Create 8 variables of interest from the US Census data

```{r eval=FALSE}

print("Don't run me")

# Create proportion white and proportion minority

# Proportion race variables (by total population per census tract)

# White non-hispanic

acs_2015_syr$prop_white <- (acs_2015_syr$white) / (acs_2015_syr$tot_race)

# All minorities (reverse of white non-hispanic) 

acs_2015_syr$prop_m <- abs((acs_2015_syr$prop_white) - 1) 

# Black

acs_2015_syr$prop_black <- acs_2015_syr$black / acs_2015_syr$tot_race

# Hispanic

acs_2015_syr$prop_hisp <- acs_2015_syr$hispanic / acs_2015_syr$tot_race

# Proportion of people whose incomes are lower than the federal poverty line

acs_2015_syr$pov <- acs_2015_syr$poverty / acs_2015_syr$tot_pop

# percentage of unemployed, NOT umemployment rate

acs_2015_syr$unemp <- acs_2015_syr$unemploy / acs_2015_syr$tot_pop

# Proportion of people with at least a high school degree, either graduated or with a GED

acs_2015_syr$educ <- (acs_2015_syr$high_sch + acs_2015_syr$ged) /                                      acs_2015_syr$tot_pop

# Proportion of occupied housing that is rented

acs_2015_syr$prop_rent <- acs_2015_syr$tenure_rent / acs_2015_syr$tot_occup


names(acs_2015_syr)

```

## Save file to GitHub

Save modified census data to GitHub

```{r eval=FALSE}

print("Don't run me")

save(acs_2015_syr, file="../ASSETS/acs_2015_syr.Rda")

```


### Step 3 - Maps and Analysis ###


# Introduction

This section details an example of how to create maps and analysis to analyze service catchment areas for the Syracuse MSA in upstate New York.

```{r setup, include=FALSE}

# global chunk options
knitr::opts_chunk$set(echo = TRUE, warning=F, message=F, fig.width=10)

library( ggmap )
library( rgdal )
library( devtools )
library( censusapi )
library( plyr )
library( gdata )
library( geojsonio )
library( acs )
library( tigris )
library( sf ) 
library( sp )  
library( rgeos )
library( spatialEco ) 
library( leaflet )
library( raster )
library( repmis )
library( TSP )
library( magrittr )
library( lattice )
library( GISTools )
library( pander )
library( xtable )
library( RCurl )
library( stargazer )
library( lmtest )
library( car )

```


# MAPS

## Load census shapefiles from GitHub

Data can be downloaded directly from GitHub into R. In this case, we are downloading geojson files created in step 2. (Note: To see how these shapefiles were created, check [Step 2](Step_2_-_Shapefiles_to_Geojson_Files.html) for more information). 

After data is downloaded, the geojson files need to be converted to sp projections.

```{r, echo=FALSE, results='hide'}

file.url <- "https://raw.githubusercontent.com/lecy/analyzing-nonprofit-service-areas/master/ASSETS/syr_merged.geojson"

syr <- geojson_read( file.url, method="local", what="sp")

spTransform(syr, CRS("+proj=longlat +datum=WGS84"))

file.url <- "https://raw.githubusercontent.com/lecy/analyzing-nonprofit-service-areas/master/ASSETS/syr_merged_cen.geojson"

centroids <- geojson_read( file.url, method="local", what="sp")

spTransform(centroids, CRS("+proj=longlat +datum=WGS84"))

file.url <- "https://raw.githubusercontent.com/lecy/analyzing-nonprofit-service-areas/master/ASSETS/pov_orgs.geojson"

pov_orgs <- geojson_read( file.url, method="local", what="sp")

spTransform(pov_orgs, CRS("+proj=longlat +datum=WGS84"))

par( mar=c(0,0,0,0) )  # drop plot margins

```
```{r}

# Print spatial projections


print("syr")
syr@proj4string
print("centroids")
centroids@proj4string
print("pov_orgs")
pov_orgs@proj4string

```
## Check shapefiles with plots

You can check how the data looks simply with the plot function. Census tracts of the Syracuse MSA are shown in black (default color) with nonprofit locations in red.

```{r}

plot(syr)
plot(pov_orgs, col="red", lwd = 3, pch = 20, add = T)

names(syr)

```
<br>
Here we notice a strong concentration of nonprofit organizations in downtown Syracuse and more spread out patterns in suburbs of Syracuse. It is somewhat difficult on the basic plot to understand the areas outside of downtown Syracuse where nonprofits concentrate in small groups. Without an underlying basemap it can be hard to disinquish where more small towns are located in the MSA. Besides adding a static baselayer map to the plot function, interactive maps found in the leaflet package offer a zoom-feature way to understand what is going on with nonprofit locations outside the city of Syracuse

## Create Leaflet maps

The leaflet package provides interactive maps that overlay spatial objects in R. This can make it easier to zoom in and out of areas of interest to notice the placement of service catchment areas in a given MSA. Leaflet includes [different styles](https://leaflet-extras.github.io/leaflet-providers/preview/) of maps that can match well with the type of the data in use or for better aesthetic appeal.

Here, we begin by creating a zoom enabled map of Syracuse, NY by providing the leaflet function with the center lat-long coordinates for Syracuse. Any coordinates may be placed into this function to center the map around a specific coordinate. 

```{r}
map <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addMarkers(lng=-76.1474244, lat=43.0481221, popup="Downtown Syracuse, NY")

map %>% addProviderTiles(providers$CartoDB)


```
<br>
Next, the view is set to the Syracuse MSA by adding a zoom out argument. Add census tract borders to the map.

```{r}


syr_map1 <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addTiles() %>% setView(-76.1474244, 43.0481221, zoom = 9) %>%
  addMarkers(lng=-76.1474244, lat=43.0481221, popup="Downtown Syracuse, NY") %>%
  addPolygons(data = syr, fillOpacity = 0.3, weight = 2)


syr_map1 %>% addProviderTiles(providers$CartoDB)  

```
<br>
This map only shows the census tract borders. We can create a choropleth map by highlighting a variable of interest which will show its distribution. The two maps below highlight two characteristics: racial diversity and income level.

```{r}

# change variables to be numeric

syr$prop_white <- as.numeric(as.character(syr$prop_white))
syr$prop_m <- as.numeric(as.character(syr$prop_m))
syr$mhh_income <- as.numeric(as.character(syr$mhh_income))

# By race

pal <- colorNumeric(
  palette = "Blues",
  domain = syr$prop_m )


syr_map2 <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addTiles() %>% setView(-76.1474244, 43.0481221, zoom = 9) %>%
  addMarkers(lng=-76.1474244, lat=43.0481221, popup="Downtown Syracuse, NY") %>%
  addPolygons(data = syr, fillOpacity = 0.3, weight = 2,
              color = ~pal(syr$prop_m)) 

syr_map2 %>% addProviderTiles(providers$CartoDB)


# By Income level

pal2 <- colorNumeric(
  palette = "Blues",
  domain = syr$mhh_income )


syr_map3 <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addTiles() %>% setView(-76.1474244, 43.0481221, zoom = 9) %>%
  addMarkers(lng=-76.1474244, lat=43.0481221, popup="Downtown Syracuse, NY") %>%
  addPolygons(data = syr, fillOpacity = 0.3, weight = 2,
              color = ~pal2(syr$mhh_income))

syr_map3 %>% addProviderTiles(providers$CartoDB)


```
<br>
The first map shows in the darker areas that, unsurpisingly, most of the minority population within the Syracuse MSA lives in the city of Syracuse instead of surrounding towns and suburbs. The darker areas on the second map note higher income census tracts. The city is rather light, suggesting lower income levels in the downtown areas of Syracuse. The light square area just south of the city of Syracuse is the Onondaga Indian Reservation, a census tract with a higher level of minority residnts but lower household income.

So where do nonprofits locate in this MSA? And where are service catchment areas in relation to where residents live?

## Create service catchment areas

Peck 2008 suggests that a nonprofit provides services to census tract if it is within 1.5 miles from the center of the respective tract. This is not an exact measurment for where actual services are provided. That would require data on all services provided by all 509 nonprofits within the Syracuse MSA. Instead, the 1.5 miles radius around the centroid of a census tract is used as approximate proxy for the extent to which citizens in need may reasonably access services that may benefit them. 

Peck's prior research (2008) suggests that the number of nonprofits serving a census tract can be measured by how many nonprofit services reach the centroid of a census tract. Yet, this can be a problematic practice once analysis extends to census tracts which are not located in inner cities. Census tracts get larger outside of cities because tract size are determined by population size not geographical size. The Census tries to create tracts of between 1,200-8,000 [residents](https://www.census.gov/geo/reference/gtc/gtc_ct.html). In rural area, it requires more geographical space to include the desired amount of residents, hence census tracts grow in geographic size the more rural it is located in an MSA.

An alternate analysis to this approach is to count the number of nonprofit service catchment areas, at a distance of 1.5 miles, that have any part within a respective census tract, or neighborhing tract. It may not cover the tract as far as the centroid, like what is done in the traditional approach, but this inclusive approach is more useful for analyzing census tracts outside highly urbanized areas and yet minimally changes analysis within city tracts. It is more useful for more rural census tracts because it allows inclusion of nonprofits providing services within rural tracts without having to account for the centroid. In addition, this approach also allows nonprofits to be accounted for in more than one census tract when their service area crosses tract boundaries. This approach resolves the concern of accounting for nonprofits located near the border of a census tract that may be realistically servicing both tracts. Since census tracts in cities are much smaller, nonprofit service areas will already capture multiple census tracts. Though this alternate approach is more inclusive of nonprofits, extending the range from past research may take into account nonprofits serving rural populations. 

The two approachs will be shown together below to understand the spatial differences between the two and why the alternative approach may be a more useful approach.

First, two different buffers, one around each centroid and another around each nonprofit organization, are created to show the different approaches on maps.

```{r, echo=FALSE, results='hide'}

# Create buffers, render in same projection
centroids_buff <- gBuffer(spgeom = centroids, width = .024, joinStyle="ROUND", byid = T)
centroids_buff@proj4string
spTransform(centroids_buff, CRS("+proj=longlat +datum=WGS84"))

org_buff <- gBuffer(spgeom = pov_orgs, width = .024, joinStyle="ROUND", byid = T)
centroids_buff@proj4string
spTransform(org_buff, CRS("+proj=longlat +datum=WGS84"))


# join data to centroid buffers
centroids_buff$ID <- c(1:185)
syr$ID <- c(1:185)
org_buff$ID <- c(1:506)
pov_orgs$ID <- c(1:506)

c_buff <- merge( centroids_buff, syr@data, "ID", "ID" )
test2 <- merge( org_buff, pov_orgs@data, "ID", "ID" )

```
<br>
Next, two different functions count the number of organizations which are located  within the buffers of the two approaches, the a centroid buffer and nonprofit buffer, in respective census tracts. The poly.counts function from the GISTools package counts the number of points within a given polygon, in this case the centroid buffer. The second function, gIntersects, creates a buffer using the rgeos package. A different function than poly.counts is required here becuase the alternative approach requires an overlay with polygon to polygon, not point to polygon.  

```{R}

# 1) Number of nonprofits within a 1.5 miles of middle of census tract.
res <- poly.counts(pov_orgs, c_buff)
syr$output1 <- setNames(res, c_buff$tract)

# 2) Number of nonprofits in each census tract
res2 <- poly.counts(pov_orgs, syr)
syr$output2 <- setNames(res2, syr$tract)

# 3) Org way:
# Number of nonprofit service catchment areas within a given census tract

inter_npo <- gIntersects(org_buff, syr, byid=TRUE)

# change logical matrix to an integer matrix

inter_npo2 <- inter_npo + 0

# Convert matrix into a data frame 

inter_npo3 <- as.data.frame(inter_npo2)

# Combine all columns into one additive vector

tot_col <- apply(inter_npo3, 1, sum)

# add number of orgs vector to census data

syr$output3 <- tot_col

# header info
print("centroid buffer = Output1")
print("number of orgs per census tract = Output2")
print("org buffer = Output3")

head(syr[,75:77])
```
<br>
Now that the two buffers are created, the two approaches to counting the number of nonprofits in a census tract can be visualized using leaflet maps.


```{r}

### Maps to See spatially what is going on

# change variables to be numeric. Required for leaflet 

syr$prop_white <- as.numeric(as.character(syr$prop_white))
syr$prop_m <- as.numeric(as.character(syr$prop_m))
syr$mhh_income <- as.numeric(as.character(syr$mhh_income))

# Centroid buffer example
syr_map4 <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addTiles() %>% setView(-76.1474244, 43.0481221, zoom = 9) %>%
  addMarkers(lng=-76.1474244, lat=43.0481221, popup="Downtown Syracuse, NY") %>%
  addPolygons(data = syr, fillOpacity = 0, weight = 2) %>%
  addCircles(data = pov_orgs, color= "red", opacity = 1.0) %>%
  addPolygons(data = centroids_buff, fillOpacity = 0.3, weight = 2)

syr_map4 %>% addProviderTiles(providers$CartoDB)


```
<br>
The first map shows the census tract centroid buffer approach. Upon investigating the first map, it can be seen that as census tracts become more rural, the 1.5 mile buffer is no longer a value tool to account for nonprofits that are located within or around the census tract. A specific example of this analytical limitation can be seen when we zoom into the small resort town of Skaneateles on the periphery of the MSA. 

```{r}

# Centroid buffer example, close up of Skaneateles
syr_map5 <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addTiles() %>% setView(-76.4291, 42.9470, zoom = 11) %>%
  addMarkers(lng=-76.4291, lat=42.9470, popup="Skaneateles, NY") %>%
  addPolygons(data = syr, fillOpacity = 0, weight = 2) %>%
  addCircles(data = pov_orgs, color= "red", opacity = 1.0) %>%
  addPolygons(data = centroids_buff, fillOpacity = 0.3, weight = 2)

syr_map5 %>% addProviderTiles(providers$CartoDB)

```
<br>
Here, the centroids buffers in these rural census tracts do not account for any nonprofits within their census tracts because the buffers are not large enough. In addition, this approach does not account for nonprofits servicing multiple census tracts. Continuing from the example, the town of Skaneateles straddles two different census tracts. Most poverty servicing nonprofits also exist on the border between two tracts and therefore in reality probably serve both census tracts. 

The alternate approach considers this limitation by accounting for nonprofit organizations serving multiple census tracts along the borders of census tracts. Here are the nonprofit buffers for the Syracuse MSA, in red:


```{r}

# org_buff example
syr_map6 <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addTiles() %>% setView(-76.1474244, 43.0481221, zoom = 9) %>%
  addMarkers(lng=-76.1474244, lat=43.0481221, popup="Downtown Syracuse, NY") %>%
  addPolygons(data = syr, fillOpacity = 0, weight = 2) %>%
  addPolygons(data = org_buff, color= "red", fillOpacity = 0.3, weight = 2) 

syr_map6 %>% addProviderTiles(providers$CartoDB)

```
<br>
It can be seen in this map that the nonprofit buffers more accurately resemble nonprofit service catchment areas by census tract. In the larger geographic census tracts, the nonprofit buffers will be counted regardless of how close they are to the center of any given tract. In addition, if the buffer crosses a border into any other census tract, that nonprofit will be considered as serving those other census tracts as well.

A main limitation to the alternate approach is that it considers nonprofits servicing another census tract regardless of how little of the buffer crosses into another tract. This means that some census tracts will artifically count nonprofits that may not in reality service that census tract. However, the alternative approach makes up for this limitation by accounting for the higher number of nonprofits that border two or more census tracts.

Let's return Skeneateles example, but with the nonprofit buffer.

```{r}

# Centroid buffer example, close up of Skaneateles
syr_map7 <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addTiles() %>% setView(-76.4291, 42.9470, zoom = 11) %>%
  addMarkers(lng=-76.4291, lat=42.9470, popup="Skaneateles, NY") %>%
  addPolygons(data = syr, fillOpacity = 0, weight = 2) %>%
  addPolygons(data = org_buff, color= "red", fillOpacity = 0.3, weight = 2) 

syr_map7 %>% addProviderTiles(providers$CartoDB)

```
<br>
Now the buffers account for nonprofits servicing both census tracts that border the town of Skaneateles, whereas in the other analytical approach none of these organizations are accounted for. The limitation of this approach is also shown in this map. In the census tract along the northern border of Skaneateles, there is a nonprofit buffer at the top right edge of the tract that barely crosses into the tract from a tract further north. Though nonprofit buffers that barely cross into another census tract may be counted, the trade-off with accounting for nonprofits that exist along tract borders is more accurate than the tradtional centroid approach when considering more rural census tracts.

# ANALYSIS

Peck (2008) found that poverty nonprofits are more likely to locate in ares with higher levels of poverty, minority residents, and renter-occupied housing in 2000 in the Phoenix MSA (p. 146). In addition, Peck observed how the number of poverty nonprofits per census tract affected poverty, controlling for other factors. Change in povery from 1990-2000 reported only education and rented housing as significantly affecting poverty change. The study concludes that even though poverty serving nonprofits located in neighborhoods that need the services, presence of nonprofits do not improve poverty in a statistically substantive way. 
Adapting work from Peck 2008, five environmental factors that may affect the location of services provided are: race, education, income, status of living (rented housing) and the proportion of poverty within a census tract. These five factors may, or may not, drive the demand for more nonprofit services to be utilized in a certain area within MSAs.

By using the nonprofit locations as buffers, more realistic service catchment areas may be utilized for analysis that account for nonprofits servicing census tracts both in urban and rural census tracts. Below, findings for the relationship between environmental factors and nonprofit locations will be presented using scatterplots, correlations, and regression analysis.

## Scatterplots

Five scatterplots showing the relationship between environmental factors and number of poverty nonprofits per census tract are created below. Using a fitted linear line in each scatterplot shows the directional relationship between variables.

```{r}
# Scatterplots


plot( syr$prop_white, syr$output3, 
      abline(lm(syr$output3~syr$prop_white)),
      main="Proportion White and # of NPOs per Census Tract",
      xlab="Proportion White", ylab="# of NPOs per Census Tract",
      pch=19, col=gray(0.5,0.5), cex=2, las = 1)

plot( syr$educ, syr$output3, 
      abline(lm(syr$output3~syr$educ)), 
      main="Proportion of population w/ high shool degree and # of NPOs per                 Census Tract",
      xlab="Proportion of population w/ high shool degree", ylab="# of NPOs per             Census Tract",
      pch=19, col=gray(0.5,0.5), cex=2, las = 1)

plot( syr$mhh_income, syr$output3, 
      abline(lm(syr$output3~syr$mhh_income)), 
      main="Median HH Income and # of NPOs per Census Tract",
      xlab="Median HH Income", ylab="# of NPOs per Census Tract",
      pch=19, col=gray(0.5,0.5), cex=2, las = 1)

plot( syr$prop_rent, syr$output3, 
      abline(lm(syr$output3~syr$prop_rent)), 
      main="Proportion of rented housing and # of NPOs per Census Tract",
      xlab="Proportion of rented housing", ylab="# of NPOs per Census Tract",
      pch=19, col=gray(0.5,0.5), cex=2, las = 1)

plot( syr$pov, syr$output3, 
      abline(lm(syr$output3~syr$pov)), 
      main="Proportion of Poverty and # of NPOs per Census Tract",
      xlab="Proportion of Poverty", ylab="# of NPOs per Census Tract",
      pch=19, col=gray(0.5,0.5), cex=2, las = 1)

```
<br>

The first three plots show a negative relationship with number of nonprofits, while the final two show a positive relationship. Census tracts in the Syracuse MSA with more white residents have less poverty nonprofits located within them. In addition, tracts with higher proportion of the population with a high school degree or a higher income also show that less poverty nonprofits locate in such areas. Higher proportions of rented housing or poverty census tracts have more poverty organizations residing within the same tracts.    

## Correlations

Going a step further, below it is shown how these six variables are correlated with each other. 

```{r}

# x is a matrix containing the data
# method : correlation method. "pearson"" or "spearman"" is supported
# removeTriangle : remove upper or lower triangle
# results :  if "html" or "latex"
# the results will be displayed in html or latex format
corstars <-function(x, method=c("pearson", "spearman"), removeTriangle=c("upper", "lower"),
                    result=c("none", "html", "latex"))
  {
  #Compute correlation matrix
  require(Hmisc)
  x <- as.matrix(x)
  correlation_matrix<-rcorr(x, type=method[1])
  R <- correlation_matrix$r # Matrix of correlation coeficients
  p <- correlation_matrix$P # Matrix of p-value 
  
  ## Define notions for significance levels; spacing is important.
  mystars <- ifelse(p < .001, "****", ifelse(p < .001, "*** ", ifelse(p < .01, "**  ", ifelse(p < .05, "*   ", "    "))))
  
  ## trunctuate the correlation matrix to two decimal
  R <- format(round(cbind(rep(-1.11, ncol(x)), R), 2))[,-1]
  
  ## build a new matrix that includes the correlations with their apropriate stars
  Rnew <- matrix(paste(R, mystars, sep=""), ncol=ncol(x))
  diag(Rnew) <- paste(diag(R), " ", sep="")
  rownames(Rnew) <- colnames(x)
  colnames(Rnew) <- paste(colnames(x), "", sep="")
  
  ## remove upper triangle of correlation matrix
  if(removeTriangle[1]=="upper"){
    Rnew <- as.matrix(Rnew)
    Rnew[upper.tri(Rnew, diag = TRUE)] <- ""
    Rnew <- as.data.frame(Rnew)
  }
  
  ## remove lower triangle of correlation matrix
  else if(removeTriangle[1]=="lower"){
    Rnew <- as.matrix(Rnew)
    Rnew[lower.tri(Rnew, diag = TRUE)] <- ""
    Rnew <- as.data.frame(Rnew)
  }
  
  ## remove last column and return the correlation matrix
  Rnew <- cbind(Rnew[1:length(Rnew)-1])
  if (result[1]=="none") return(Rnew)
  else{
    if(result[1]=="html") print(xtable(Rnew), type="html")
    else print(xtable(Rnew), type="latex") 
  }
} 


syr.sub <- syr@data[ c("output3","prop_white", "educ", "mhh_income",                                    "prop_rent","pov") ]


corstars(syr.sub)

```
<br>
Correlations between variables reveals some trends. The directional relationships between variables remain the same as found in the scatterplots. Number of poverty nonprofits per census tract (output3) is correlated the most with proportion of white residents and the least with median household income. The variables that are most correlated together are median household income, proportion of rented housing, and porportion of poverty.  

## Regressions

Bivariate relationshsips between variables in OLS regression show similar results to the scatterplots and correlations. The direction of relationship between variables, not surprisingly, continues to be the same.

```{r}
# Bivariate relationships between number of NPO service catchment areas by census tract

pander(m1 <- lm(syr$output3 ~ syr$prop_white),
       caption = "Proportion of non-white residents")   
pander(m2 <- lm(syr$output3 ~ syr$educ),
       caption = "Proportion of residents with high school degree")
pander(m3 <- lm(syr$output3 ~ syr$mhh_income),
       caption = "Median household income")
pander(m4 <- lm(syr$output3 ~ syr$prop_rent),
       caption = "Proportion of residents living in rented housing")
pander(m5 <- lm(syr$output3 ~ syr$pov),
       caption = "Proportion of residents in poverty") 

```
<br>

Additional information found in these bivariate OLS regressions not known from the scatterplots, but verified in the correlations, is that each independent variable on its own is significantly related to the number of poverty nonprofits per census tract.

Below, two different OLS regression models include all five variables of interest. The first model reports the effect of environmental factors on the number of poverty organizations per census tract. The second model switches the number of nonprofits and the poverty variable, observing the effect of other variables and number of nonprofits on poverty per census tract. These two models are adapted from Peck 2008. 

First, the models are checked to see whether any multicollinearity between predicting variables exist using a VIF test. Also, heteroscedasticity of error terms is checked with a Beusch-Pagan test.

```{R}

# OLS reg of number of NPOs by five explanatory variables

m6 <- lm(syr$output3 ~ syr$prop_white + syr$educ + syr$mhh_income + syr$prop_rent          + syr$pov)

# poverty as a dependent variable, number of NPOs as a independent variable, same other explanatory variables

m7 <- lm(syr$pov ~ syr$prop_white + syr$educ + syr$mhh_income + syr$prop_rent +           syr$output3)

# Check for multicollinearity and heteroschedasticity

  # Variance inflation factors test. Between 6-10 the variable is highly colinear     with others in the model.
vif(m6)
vif(m7)

  # Breusch-Pagan Test
bptest(m6) 
bptest(m7)
```
<br>
The VIF test reports no multicollinearity issues within the model. Median houshold income reports the highest VIF score at 5.9. This should not be a worry in the model however as scores of 10 or higher in a VIF test are more indicative of a multicollinearity problem (Hoffmann 2010). The Breusch-Pagan test rejects the null hypothesis (presence of homoscedasticity), reporting heteroscedasticity in both models. To correct for heteroscedassticity, the models are run using robust standard errors. 

Running robust standard errors in R usually requires many lines of code. But, an add-on package can be downloaded for the summary function that simplifies robust standard errors to a simple logical argument. 

```{r}
# Download robust argument for summary function
# Site for downloading robust standard error argument for summary function
# https://economictheoryblog.com/2016/08/08/robust-standard-errors-in-r/

url_robust <- "https://raw.githubusercontent.com/IsidoreBeautrelet/economictheoryblog/master/robust_summary.R"
eval(parse(text = getURL(url_robust, ssl.verifypeer = FALSE)),
     envir=.GlobalEnv)

# First adjusted regression model

set.caption("Factors affecting number of poverty nonprofits per census tract")
pander(summary(m6, robust = T))

``` 
<br>

The first OLS regression observes the factors affecting the number of poverty nonprofits per census tract. Results show that census tracts with an increase in the proportion of white non-hispanic residents and proportion of residents with a high school degree are significantly asociated with lower numbers of poverty nonprofits locating within them. Level of income, proportion of rented housing, and proportion of poverty do not affect the number of poverty nonprofits after accounting for other factors. This suggests that nonprofits are locating in the census tracts where minority or less educated populations live.

What is important to point out is that poverty is not associated with the number of poverty nonprofits. This suggests that poverty nonprofits not only are located in census tracts where the worst poverty exists, but also in locations where poverty is not very high. This suggests that poverty organizations, though located in large numbers in the central city of the MSA, also exist in substantial amount in more rural census tracts. This presumes that underserved populations who need access to nonprofits providing services have a moderate level of accessibility geographically. 

What this data cannot account for are other factors, such as the health of a client or their access to transportation, as well as the percieved factors of clients that may or may not incentvize them to contact services for nonprofits (such as perceived stigma to acess mental health services). In addtion, the observed locations of poverty nonprofits does not account for the use of satillite services or work by nonprofits in outreach programs done in the communities of need. Further research is needed to fully understand the issue of access. This project's answerable findings focus more primarily on WHERE poverty nonprofits are located and are their environmental factors that may affect the spatial location of nonprofits. 
  
Now that the location of poverty nonprofits has been investigated, does the presence of poverty nonprofits closer to higher poverty neighborhood affect poverty in any way? The second regression model observes factors affecting proportion of poverty per census tract.

```{r}

# Second adjusted regression model

set.caption("Factors affecting proportion of poverty per census tract")
pander(summary(m7, robust = T))

```
<br>

The OLS regression reports no significant relationship between the number of poverty nonprofits per census tract and poverty. Factors that are associated with lower levels of poverty are higher proportion of white non-hispanic residents, proportion of residents with a high school degree, and median household income. This suggests that socio-economic factors are predictive of census tract poverty while the number of nonprofit organizations located in tracts with more or less poverty does not. Another variable used in Peck's (2008) research which would be useful to investigate in the future is the amount of expenditures from nonprofits per census tract. Even so, Peck's research did not find a relationship to higher spending and lower poverty. Therefore, at least when it comes to improving poverty, the location of nonprofits in the Syracuse MSA does not affect the poverty status of residents in any substantial manner. 

## Discussion

What remains to be seen is whether and to what extent these relationships occur in larger, more spread out MSAs in the United States. Further research could investigate these trends using five data files located in the ASSETS folder of this repository. These data files include the poverty nonprofits in top five most populous MSAs in the United States. These datasets still require downloading their respective geo-coordinates. Following the code in this repository, analysis on these locations can be done once the files are geocoded.

## References

Hoffmann, John P. 2010. Linear Regression Analysis: Applications and Assumptions
Second Edition. Provo, UT: Adobe Acrobat Professional.

Peck, Laura R. 2008. "Do Antipoverty Nonprofits Locate Where People Need Them? Evidence from a Spatial Analysis of Phoenix." Nonprofit and Voluntary Sector Quarterly 37(1):138-151.
