# Analyzing Nonprofit Service Areas

## Project Overview

This project sets out to show how spatial analysis can provide better understanding of nonprofit service areas in metropolitan areas of the United States. IRS data on nonprofit organizations and census tract information from the U.S. census is combined to explore the environmental effects on where nonprofit organizations locate and how the presence of nonprofits is related to poverty. We will focus on one specific context, Syracuse New York, and then expand the observation to most populated areas in the United States.

A variety of techniques were used to create this project. Here are a few valuable techniques used: Geocoding an entire dataset with one looped function, importing American Community Survey data and U.S. census shapefiles right into R without spending time going through convoluted websites, combining shapefiles into one  shapefile in the form of a geojson file, how to use interactive maps, and run basic scatterplots, correlations, and regressions in R. In this sense, this project also highlights many valuable tools available in R to create a database of useful data on nonprofits, wrangle data, and run simple analysis. All files necessary to run code are located in this project GitHub repository, allowing anyone to use the data and code for their own purposes without needing to collect data or change file locations. 

Four steps listed below outline the process of running this analysis. First, it will be shown how the main data was downloaded, geocoded, cleaned, and saved. The second step highlights how to download important shapefiles and convert them into single geojson files. Third, maps and analysis show the visual locations of nonprofits in the Syracuse, New York metropolitan area and simple scatterplots and regressions reveal the relationships between environmental factors and nonprofit locations. Finally, all prior code will be used for an extended analysis that goes beyond the Syracuse MSA to observe how location of nonprofits are affected in the five most populated areas in the U.S: New York, Los Angeles, Chicago, Dallas-Fort Worth, and Houston. 

The hope is that this project will provide nonprofit researchers with relevant data to further the understanding of how nonprofits help areas in poverty in the U.S. In addition, this project may also act as a tutorial, providing beginners to R with many techniques which may be applied in other settings.

# Step 1 - Download Syracuse Data and Census Data

This preliminary step provides the code of how data in the ASSETS folder on GitHub was downlowded and changed for the project.

[Download and clean files](Step_1_-_Download_and_Clean_Data.html)

# Step 2- Converting Shapefiles to Geojson Files

Shows how U.S. Census shapefiles can be downloaded directly from the Census to R. Additionally, it shows how to merge a data frame with shapefiles and convert shapefiles into geojson files.

[Shapefiles](Step_2_-_Shapefiles_to_Geojson_Files.html)

# Step 3 Maps and Analysis of Nonprofits

Step 3 offers both visuals of how to map spatial data and how to run basic analysis in R. A discussion of the results of nonprofits in the Syracuse MSA are included in this step.

[Maps and analysis](Step_3_-_Maps_and_Analysis.html)

# Analysis Across Top 5 Populated U.S. Metropolitan Areas

This final step combines code from all other steps to observe the effects on nonprofit location in New York, Los Angeles, Chicago, Dallas-Fort Worth, and Houston. A discussion about the code and findings is found below.

[Extended analysis](Step_4_-_Extended_Analysis.html)

# Conclusion

This project seeks to offer tools and techniques available in R programming software to improve undestanding of the spatial nature of nonprofit organizations servicing impovished populations. It provides an example of how to build a research database located online on GitHub for nonprofit research. Finally, it shows how to download open data, wrangle it, and use it in an analysis example. 

# Acknowledgements

This project was made possible by [Matthew McKnight](https://github.com/mlmcknight) under [Jesse Lecy's](https://github.com/lecy) supervision.
