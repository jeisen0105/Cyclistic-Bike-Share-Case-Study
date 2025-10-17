# Case Study: Cyclistic Bike-Share Analysis (Google Data Analytics Capstone)

### Table of Contents
* Introduction
* Background
* Scenario
* Ask
* Prepare
* Process
* Analyze
* Share
* Act

## Introduction

This project presents an analysis of the Cyclistic Bike-Share case study, part of the Google Data Analytics Professional Certificate capstone. The goal is to address key business questions using the six-step data analysis process: Ask, Prepare, Process, Analyze, Share, and Act. 

## Background

Cyclistic is a bike-share program based in Chicago, featuring 5,824 bicycles and 692 docking stations across the city. Unlike other bike-share companies, Cyclistic offers a range of bicycles, including those designed for people with disabilities—such as reclining bikes, hand tricycles, and cargo bikes—making the service more inclusive and accessible.

Cyclistic also provides flexible pricing options: single-ride passes, full-day passes, and annual memberships, allowing it to appeal to a broad range of users. However, the company’s marketing director believes future success lies in increasing the number of annual memberships.

To support this goal, we must analyze how casual riders (non-members) differ from annual members in their usage patterns. These insights will help inform a targeted marketing strategy aimed at increasing annual memberships.

## Scenario

In this hypothetical scenario, I am a junior data analyst on Cyclistic’s marketing analytics team. I will present my findings and recommendations to key stakeholders, including Lily Moreno (Marketing Director) and the executive team.

This report includes a clear business task, a description of the dataset, documentation of cleaning and transformation processes, a summary of analysis and visualizations, and actionable recommendations.
 
## Ask

The key business task is to analyze how annual members and casual riders use Cyclistic bikes differently, and identify ways to encourage casual riders to become annual members.

## Prepare

### Data Source

When approaching the prepare section it's critical to ensure data is reliable, original, comprehensive, current and cited. The data being used comes from Cyclistic's divvy trip data pertaining to both 2019 Q1 and 2020 Q1 and has been made available by Motivate International Inc. under their [Data License Agreement](https://divvybikes.com/data-license-agreement). Because R was used for analysis, only Q1 data was used due to size and format limitations.

The data is public and can be used to investigate how different customer types are using Cyclistic bikes. In order to comply with data privacy issues the data excludes riders personally identifiable information including names or financial details. 

In order to determine whether or not the data source is reliable, original, comprehensive, current and cited I will follow the ROCCC framework. I know the data is reliable and original because it contains accurate, complete and unbiased information on Cyclistic's historical bike trips which come from a primary source. The data also contains all information needed to understand the different ways annual and casual riders use Cyclistic bikes making it quite comprehensive. Additionally because the data sources are provided publicly by Cyclistic it can be referenced easily. Finally even though our data is 5 to 6 years old it is still young enough for our analysis. 


I first opened each .csv file and saved them to the appropriate subfolder in order to have a copy of the
original data. After saving each .csv to the subfolder I imported the data and installed the necessary packages.

```r
#Install necessary packages
install.packages("tidyverse")
install.packages("conflicted")
library(tidyverse) 
library(conflicted)

#Set dplyr::filter and dplyr::lag as the default choices
conflict_prefer("filter", "dplyr")
conflict_prefer("lag", "dplyr")
```

I then uploaded the files to R Studio to clean the data further.

```r
q1_2019 <- read_csv("Divvy_Trips_2019_Q1 - Divvy_Trips_2019_Q1.csv")
q1_2020 <- read_csv("Divvy_Trips_2020_Q1 - Divvy_Trips_2020_Q1.csv")
```


### Data Information and Organization

Datasets for Q1 of 2019 and 2020 were downloaded from the cloud and stored on my hard drive. They were then imported into Google Drive to examine in Google Sheets and then imported to RStudio for processing. After processing the data, it was finally exported to Tableau for visualization.

Each file includes the months of January, February and March but uses several different column names. The different columns were edited to match each other in order to compare the two years. 

## Process

The processing stage involves cleaning and transforming the data to ensure accuracy, consistency, and readiness for analysis. This includes removing incomplete or innaccurate entries and aligning column names/data types across datasets.


The column names in the 2019 data were renamed to match the 2020 schema, and the ride_id and rideable_type columns were explicitly converted to the character data type to allow for correct stacking  

```r
#Rename columns to make them consistent with q1_2020
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

#Convert ride_id and rideable_type to character so that they can stack correctly
q1_2019 <-  mutate(q1_2019, ride_id = as.character(ride_id)
                   ,rideable_type = as.character(rideable_type))
```

The cleaned quarterly data frames were combined into a single master data frame, all_trips.

```r
#Stack individual quarter's data frames into one big data frame
all_trips <- bind_rows(q1_2019, q1_2020)
```

The 2019 user type labels ("Subscriber" and "Customer") were standardized to match the 2020 labels ("member" and "casual") using the recode() function. Unnecessary columns (like birthyear, gender, and specific station coordinates) that were dropped from the public data in 2020 were removed for consistency.

```r
#Standardize user type labels
all_trips <-  all_trips %>% 
  mutate(member_casual = recode(member_casual,"Subscriber" = "member","Customer" = "casual"))

#Remove non essential columns
all_trips <- all_trips %>%  
  select(-c(start_lat, start_lng, end_lat, end_lng, birthyear, gender, tripduration))
```

New columns were created to extract granular time data (month, day, year, day-of-week) from the started_at timestamp, and the ride_length column was calculated by finding the difference between the end and start times. The data type for the ride_length column is also converted to numeric. 

```r
#Add columns that list the date, month, day, and year of each ride
all_trips$date <- as.Date(all_trips$started_at)
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")

#Add a "ride_length" calculation to all_trips
all_trips$ride_length <- difftime(all_trips$ended_at,all_trips$started_at)

#Convert "ride_length" from Factor to numeric 
all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
```

After noticing that the ride_length column contains negative values and the start_station_name column contains invalid entries a new dataframe is created to filter out these invalid entries. 

```r
# Create a new version of the dataframe (v2) since data is being removed
all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length<0),]
```

## Analyze

After finishing cleaning and adding more data I was finally able to conduct a descriptive analysis.

```r
#Descriptive analysis on ride_length 
mean(all_trips_v2$ride_length) #straight average (total ride length / rides)
[1] 1189.459
median(all_trips_v2$ride_length) #midpoint number in the ascending array of ride lengths
[1] 539
max(all_trips_v2$ride_length) #longest ride
[1] 10632022
min(all_trips_v2$ride_length) #shortest ride
[1] 1

#Descriptive Analysis on ride length for members and casual users
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = mean)
  all_trips_v2$member_casual all_trips_v2$ride_length
1                     casual                5372.7839
2                     member                 795.2523
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = median)
  all_trips_v2$member_casual all_trips_v2$ride_length
1                     casual                     1393
2                     member                      508
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = max)
  all_trips_v2$member_casual all_trips_v2$ride_length
1                     casual                 10632022
2                     member                  6096428
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = min)
  all_trips_v2$member_casual all_trips_v2$ride_length
1                     casual                        2
2                     member                        1

#Average ride length for memebrs and casual users depedning on weekday
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
   all_trips_v2$member_casual all_trips_v2$day_of_week
1                      casual                   Friday
2                      member                   Friday
3                      casual                   Monday
4                      member                   Monday
5                      casual                 Saturday
6                      member                 Saturday
7                      casual                   Sunday
8                      member                   Sunday
9                      casual                 Thursday
10                     member                 Thursday
11                     casual                  Tuesday
12                     member                  Tuesday
13                     casual                Wednesday
14                     member                Wednesday
all_trips_v2$ride_length
1                 6090.7373
2                  796.7338
3                 4752.0504
4                  822.3112
5                 4950.7708
6                  974.0730
7                 5061.3044
8                  972.9383
9                 8451.6669
10                 707.2093
11                4561.8039
12                 769.4416
13                4480.3724
14                 711.9838

#Order the days of the week
all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))

# Now, let's run the average ride time by each day in order for members vs casual users
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
   all_trips_v2$member_casual all_trips_v2$day_of_week all_trips_v2$ride_length
1                      casual                   Sunday                5061.3044
2                      member                   Sunday                 972.9383
3                      casual                   Monday                4752.0504
4                      member                   Monday                 822.3112
5                      casual                  Tuesday                4561.8039
6                      member                  Tuesday                 769.4416
7                      casual                Wednesday                4480.3724
8                      member                Wednesday                 711.9838
9                      casual                 Thursday                8451.6669
10                     member                 Thursday                 707.2093
11                     casual                   Friday                6090.7373
12                     member                   Friday                 796.7338
13                     casual                 Saturday                4950.7708
14                     member                 Saturday                 974.0730

# Analyze ridership data by type and weekday
all_trips_v2 %>% 
+   mutate(weekday = wday(started_at, label = TRUE)) %>%  #creates weekday field using wday()
+   group_by(member_casual, weekday) %>%  #groups by usertype and weekday
+   summarise(number_of_rides = n()							#calculates the number of rides and average duration 
+             ,average_duration = mean(ride_length)) %>% 		# calculates the average duration
+   arrange(member_casual, weekday)								# sorts
`summarise()` has grouped output by 'member_casual'. You can override using the `.groups` argument.
# A tibble: 14 × 4
# Groups:   member_casual [2]
   member_casual weekday number_of_rides average_duration
   <chr>         <ord>             <int>            <dbl>
 1 casual        Sun               18652            5061.
 2 casual        Mon                5591            4752.
 3 casual        Tue                7311            4562.
 4 casual        Wed                7690            4480.
 5 casual        Thu                7147            8452.
 6 casual        Fri                8013            6091.
 7 casual        Sat               13473            4951.
 8 member        Sun               60197             973.
 9 member        Mon              110430             822.
10 member        Tue              127974             769.
11 member        Wed              121902             712.
12 member        Thu              125228             707.
13 member        Fri              115168             797.
14 member        Sat               59413             974.
```

### Step 6:

The descriptive analysis revealed the following key metrics:

#### All riders ride length (seconds)

* Mean: 1190
* Median: 539
* Max: 10632022
* Min: 1

#### Casual vs Member ride length (seconds)

Mean
* Casual: 5373
* Member: 795

Median
* Casual: 1393
* Member: 508

Max
* Casual: 10632022
* Member: 6096428

Min
* Casual: 2
* Member: 1

### Step 7:

Finally I will visualize the data through visuals from R Studio as well as Tableau.

```r
# Visualize the number of rides by rider type
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge")

# Create a visualization for average duration
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge")
```
![Chart](Rplot01.png)

![Chart](Rplot02.png)

Visualizations in R studio are useful however Tableau has more sophisticated graphics and tools that can allow us to visualize the data in a more clear light. In order to further visualize the data in Tableau we first need to create a csv file of the processed information from R Studio.

```r
# Create a csv file that we will visualize in Google Slides, Tableau, and my presentation software
counts <- aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
write.csv(counts, file = 'avg_ride_length.csv')
write.csv(all_trips_v2, file = 'all_trips_v2.csv')
```
After downloading the csv file I was able to import the data into Tableau and create calculated fields using existing variables in order to visualize data in a different light. The new calculated fields included the average ride time in hours per customer based on the day of the week as well as the minimum and maximum average ride lengths based on rider type. Below are the visualizations I made in Tableau using the csv file I just created. 

![Chart](Dashboard1.png)

Additionally I decided to further analyze the data in Google Sheets by using the same CSV file used in Tableau. Using google sheets I decided to calculate the total number of riders per year based on their Rider Type. From there I was able to calculate the percent difference between rider types and year on the total number of rides taken.

![Chart](TotalNumberofRidesPerRide-TypePerYear.png)

## Share

A visual summary of my analysis and findings is available in this Google Slides presentation:

[Case Study Presentation](https://docs.google.com/presentation/d/1tSAN2O-nCUPZwXn4mAo8GnBrnL4hxDB_QYoiYfrqYw8/edit?usp=sharing)

## Act

### Key Takeaways

Highest Average Ride Lengths and Total Rides By Weekday
- Casual Riders: The highest total number of rides per weekday occurs on Saturday and Sunday with Sunday at the top. The highest average duration is Thursday followed by Friday then Sunday then Saturday.
- Annual Riders: The highest total number of rides per weekday occurs from Monday-Friday with Tuesdays boasting the highest number. The highest Average ride length occurs on Saturday followed by Sunday.

Total Number of Riders
- Casual Riders: Significant growth from 2019 Q1 - 2020 Q1 at roughly 93%.
- Annual Riders: Roughly 11% growth from 2019 Q1 - 2020 Q1.

Shortest to Longest Ride Length Range
- Casual Riders: Casuals ranged from roughly 0.0006 seconds to 2,953.34 seconds
- Annual Riders: Annuals ranged from roughly 0.0003 seconds to 1,693.45 seconds

Average Ride Length by Rider Type (Seconds)
- Casual Riders: 5,373 
- Annual Riders: 795

### Recommendations

There are quite a few recommendations to make in order to convert as many casual riders to annual riders as possible. I recommend focusing advertisements on Thursdays and Fridays, when casual riders exhibit the longest average ride durations. When advertising to casual riders I also recommend that Cyclistic focus on the many benefits of using an Annual Membership such as cost saving, accessibility and convenience when traveling daily to work possibly through social media. In addition to these benefits Cyclistic can also offer additional perks. These perks can include loyalty and reward programs as well as member only events in order to increase incentives. 

Another one of my recommendations includes building strategic partnerships with local businesses and organizations in order to increase the value of memberships. Local Businesses can offer discounts to Annual members providing an incentive to join. Another strategic partnership would be transit integration. Cyclistic could partner with public transit systems in order to offer a combined pass. 

Finally my last recommendation would be to use rider data to send targeted emails for casual riders that biked more than X times in a certain month. The emails could send them a personalized message letting them know how many trips they have taken and how much they could have saved with a membership. 

### Conclusion

The analysis of Cyclistic's bike share data from Q1 of 2019 and 2020 illustrate distinct usage patterns between Casual and Annual membership riders. Casual riders demonstrate significantly longer ride lengths specifically on Thursdays and Fridays and also exhibited significant growth from 2019 to 2020. To capitalize on the growth of casual riders, my team and I recommend targeted strategies to convert them into annual members. These strategies would include highlighting existing benefits of annual memberships and creating new incentives. Through using data driven insights and strategic marketing Cyclistic will be able to increase their customers with annual memberships allowing the future growth of the company. 


