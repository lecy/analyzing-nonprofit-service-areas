# Download and Clean Data
Matt McKnight  
May 30, 2017  






# NONPROFIT DATA

## Create NPO Data Frame


```r
print("Don't run me")

# Import subset from NCCS BMF Data on nonprofits in Syr MSA

source_data("https://github.com/lecy/analyzing-nonprofit-service-areas/blob/master/DATA/syr_orgs.Rda?raw=true")


# Subset - Extract poverty orgs

pov_orgs <- syr_orgs[which (  syr_orgs$NTEECC == "B61" | 
                              syr_orgs$NTEECC == "B82" |
                              syr_orgs$NTEECC == "B92" |
                            syr_orgs$nteeFinal1 == "E" | 
                            syr_orgs$nteeFinal1 == "F" |
                            syr_orgs$nteeFinal1 == "I" |
                              syr_orgs$NTEECC == "K30" |
                              syr_orgs$NTEECC == "K31" |
                              syr_orgs$NTEECC == "K34" |
                              syr_orgs$NTEECC == "K35" |
                              syr_orgs$NTEECC == "K36" |
                            syr_orgs$nteeFinal1 == "L" |  
                            syr_orgs$nteeFinal1 == "O" |
                            syr_orgs$nteeFinal1 == "P" |  
                            syr_orgs$nteeFinal1 == "R" |  
                            syr_orgs$nteeFinal1 == "S" |
                            syr_orgs$nteeFinal1 == "T"), ]
```


## Geocode Poverty Organizations


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
```


# CENSUS DATA

Download & clean Census ACS 2011-2015 5-yr data

## Census API Package


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
              "GEOID" )

acs_2015 <- getCensus( name="acs5",
                       vintage = 2015,
                       key = censuskey,
                       vars= var.list, 
                       region ="tract:*", 
                       regionin ="state:36" 
                     )


# subset to Syracuse 
# sum(acs_2015$county == "067")
# sum(acs_2015$county == "075")
# sum(acs_2015$county == "053")



acs_2015_syr <- acs_2015[which (acs_2015$county == "067" | 
                                acs_2015$county == "075" |
                                acs_2015$county == "053"), ]
```



## Rename Variables


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
         "B03001_003E" = "hispanic"
      
           ))

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
              "not_hispanic", "hispanic"))

# Create census race proportion variable

acs_2015_syr$porp_white <- (acs_2015_syr$white) / (acs_2015_syr$tot_race)
acs_2015_syr$porp_m <- abs((acs_2015_syr$porp_white) - 1) 

names( acs_2015_syr )
```
