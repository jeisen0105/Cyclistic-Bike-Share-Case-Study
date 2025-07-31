# Cyclistic

## Introduction

In this project I will be examining the Cyclistic Bike Share Case Study which is a capstone project for the Google Data Analytics Professional Certificate. I will examine this case study and address key buisness qurstions using the different steps I learned from the course including; to ask, prepare, process, analyze, share and act. 

## Background

Cycalistic is a bike share program in Chicago that features 5,824 bicycles and 692 docking stations across the city. What separates Cyclistic and other bike sharing companies is that they don't only have customary bicycles they also carry cyclistic bicycles. Cyclystic biscycles include reclining bikes, hand tricycles and cargo bikes. These options make bike sharing more inclusive to those with disabilities or those who can't use a standard two-wheeled bike. Cyclystics' vast array of different bicycles and stations allow citizens to travel conveniently throughout the city and with ease.

In addition to offering Cyclystic bikes, Cyclystic also maintains flexibility in their pricing plans. Cyclystic offers single-ride pass, full-day pass and annual memberships illustrating the company's effort to appeal to broad consumer segments. These changes are important to implement however Cyclystics marketing director believes that by maximizing the number of annual memberships the company will succeed in the future. In order to fully understand how to maximize annual memberships, we must first differentiate between casual riders (those who use the app without a membership) and our annual members. Through analyzing what separates the two groups I can help Cyclystic design a new marketing strategy to maximize conversions of casual riders into annual members.

## Scenario

In this hypothetical case study I am a junior data analyst wokring on the marketing analyst team at Cyclistic and will be presentiing my findings as well as possible solutions to key stakeholders such as Lily Moreno the director of marketting, as well as the cyclistic executive team. My report will specifically entail a clear statment on the buisness task, a descruption of data used, documentation of cleaning or manipulation of data, a summary of analysis, supporting vizualizations and my top three reccomendations based on my analysis.  

## Ask

When approaching the ask section it is vital to understand the buisness task and to consider key stakeholders. The buisness task or what we will be trying to solve is how annual members and casual riders use ciclystic bikes differentky. After we find these differences we will then be able to explore ideas on how the company can covert the casual riders into annual members. 

## Prepare

### Data Source

When approaching the prepare section its crtical to ensure data is reliable, original, comprehensive, current and cited. The data being used comes from Cyclistics divvy trip data pertaining to both 2019 Q1 and 2020 Q1 and has been made available by Motivate International Inc. under their Data License Agreement (make link clickable). Because R was used for organization and analysis only Q1 was accessable and will not containe data concerning Q2, Q3, or Q4.

The data is public and can be used to investigate how different customer types are using Cyclistic bikes. In in order to comply with data pprivacy issues the data excludes riders personally idneitfiable information including names or finnacial details. 

In order to determine wheither or not the data source is relaible, original, comprehensive, current and cited we will follow the ROCCC framwork. We know the data is reliable and original ebcause it contains accurate, complete and inbiased information on Cyclysitcs historicak bike trips which come from a prumary source. The data also contains all infmation neede to undestand the different ways annual and casual riders use Cyclistic bikes making it quite comprehensive. Adddionally because the data sources are provided publicially by Cyclistic it can be refrenced easily. Finally even though our data is 5-6 years old it is still young enough for our analysis. 

### Data Information and Organization

Datasets for Q1 of 2019 and 2020 were downloaded from the cloud and stored on my harddrive. They were then imported into Google Drive to examine in Google sheets and then imported to R studio for processing. After pprocessing the data, it was finally exported to Tableau for vizualization

Each file includes the months of January February and March but use sevieral different column names. The different columns were edtied to match eachother in order to compare the two years. 

## Process

When approaching the process stage its essential to purge the data ensuring accuracy and eliminating any incomplte entres. It is equally as vital to ensure consistnecy across all of the datas elements in order to analyze the data sucessfully.  

### Step 1: Combine and Explore Data

I first opneed each .csv ans saved them to the appropriate subfolder in order to have a copy of the
original data. After saving each .csv to the subfolder I imported the data and installed the necessary packages.

```r
install.packages("tidyverse")
library(tidyverse)  #helps wrangle data

# Use the conflicted package to manage conflicts
library(conflicted)

# Set dplyr::filter and dplyr::lag as the default choices
conflict_prefer("filter", "dplyr")
conflict_prefer("lag", "dplyr")
```

I then uploaded the files to R Studio to clean and manipulate the data further.

```r
q1_2019 <- read_csv("Divvy_Trips_2019_Q1 - Divvy_Trips_2019_Q1.csv")
q1_2020 <- read_csv("Divvy_Trips_2020_Q1 - Divvy_Trips_2020_Q1.csv")
```

Once I had both files on R studio I was able to wrangle both into a single file by renaming the collunns and converting data types to ensure they stack correctly. After stacking the data frames I then removed any inconsistencies between the two files. 

```r
# Rename columns to make them consistent with q1_2020
(q1_2019 <- rename(q1_2019
                   ,ride_id = trip_id
                   ,rideable_type = bikeid
                   ,started_at = start_time
                   ,ended_at = end_time
                   ,start_station_name = from_station_name
                   ,start_station_id = from_station_id
                   ,end_station_name = to_station_name
                   ,end_station_id = to_station_id
                   ,member_casual = usertype))

# Inspect the dataframes and look for incongruencies
str(q1_2019)
str(q1_2020)

# Convert ride_id and rideable_type to character so that they can stack correctly
q1_2019 <-  mutate(q1_2019, ride_id = as.character(ride_id)
                   ,rideable_type = as.character(rideable_type)) 

# Stack individual quarter's data frames into one big data frame
all_trips <- bind_rows(q1_2019, q1_2020)

# Remove lat, long, birthyear, and gender fields as this data was dropped beginning in 2020
all_trips <- all_trips %>%  
  select(-c(start_lat, start_lng, end_lat, end_lng, birthyear, gender,  "tripduration"))
```

I then finished cleaning and adding the data in preperation of the analysis stage. 

```r
# Reassign to the desired values (using the current 2020 labels)
all_trips <-  all_trips %>% 
  mutate(member_casual = recode(member_casual,"Subscriber" = "member","Customer" = "casual"))

# Add columns that list the date, month, day, and year of each ride: allowing you to aggregate ride data for each moth, day or year.
all_trips$date <- as.Date(all_trips$started_at)
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")

# Add a "ride_length" calculation to all_trips
all_trips$ride_length <- difftime(all_trips$ended_at,all_trips$started_at)

# Inspect the structure of the columns
str(all_trips)

# Convert "ride_length" from Factor to numeric
is.factor(all_trips$ride_length)
all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
is.numeric(all_trips$ride_length)

# Remove "bad" data, the dataframe includes a few hundred entries when bikes were taken out of docks and checked for quality by Divvy or ride_length was negative
# You will create a new version of the dataframe (v2) since data is being removed
all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length<0),]
```











