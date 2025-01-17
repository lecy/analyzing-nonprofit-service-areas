# Step 1 - Download and Clean Data
Matt McKnight  
June 13, 2017  
# INTRODUCTION

Step 1 is presented to show the process of data collection for this project. Code from step 1 includes downloading and cleaning data from two different databases. First, information on nonprofit organizations in the Syracuse metropolitan area (MSA) is downloaded from IRS information. Coordinates for each nonprofit are then matched using a Google API. Second, U.S. Census information on demographics by census tract are downloaded from the census using an API. Data created from this code is already located in the asset folder of this repository and therefore need not be run. 




# NONPROFIT DATA

## Create NPO Data Frame

The IRS Business Master Files (BMF) includes all types of tax exempt organizations. Therefore, the first step is to subset the data to organizations whose missions focus more directly on providing services to poverty populations. The [NTEECC codes](http://nccs.urban.org/classification/national-taxonomy-exempt-entities) provide a simple way to subset the data. Here, we will subset the BMF in the Syracuse MSA using the [most recent BMF](http://nccs-data.urban.org/data.php?ds=bmf), August, 2016.

Choosing what organizations to include in the sample will follow empirical work from [Joassart-Marcelli and Woch (2003)](http://journals.sagepub.com/doi/abs/10.1177/0899764002250007) and [Peck 2008](http://journals.sagepub.com/doi/abs/10.1177/0899764006298963).Organizations from the following NTEECC 15 categories are excluded: Arts, Culture, and Humanities (A); Environment (C); Animal Related (D); Disease/Disorders (G); Medical Research (H); Public Safety, Disaster, and Relief (M); Recreation (N); International, Foreign Affairs, and National Security (Q); Philanthropy, Voluntarism, and Grant-Making Foundations (T); Science and Technology (U); Social Science (V); Public, Society Benefit (W); Religion Related (X); Mutual/Membership Benefit Organizations (Y); and Unknown (Z). 

Relevant codes from the remaining 11 NTEECC categories are included. All human services organizations are included (P). Also included are: Adult education (B60) and educational services (B90) in education; community clinics (E32) and Public Health Programs (E70) in health; substance abuse prevention and treatment (F20) and selected community mental health centers (F30) in mental health; crime prevention (I20), rehabilitation services for offenders (I40), abuse prevention (I70), and legal services (I80) from crime; employment assistance and training (J20), and vocational rehabilitation (J30) in employment; food programs (K30), nutrition (K40), and home economics (K50) from food, agriculture, and nutrition; 
housing development, construction &  management (L20), 	housing search assistance (L30), temporary housing (L40), and	housing support (L80), from housing and shelter; youth centers and clubs (O20), adult and child matching programs (O30), and youth development programs (O50) in youth development; and community and neighborhood development (S20) in community improvement and capacity building. In addtion, we add two categories from civil rights, social action, and advocacy which are neglected in past empirical research: civil rights (R20) and intergroup and race relations (R30) organizations. 


```r
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


```r
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


```r
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


```r
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


```r
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


```r
print("Don't run me")

save(acs_2015_syr, file="../ASSETS/acs_2015_syr.Rda")
```



