# The question I will be answering in this case study is: how do annual members and casual riders use Cyclistic bikes differently?

## Here are the deliverables I am expected to report:
### 1. A clear statement of the business task
### 2. A description of all data sources used
### 3. Documentation of any cleaning or manipulation of data
### 4. Visualizations and key findings
### 5. My top three recommendations based on my analysis

### A Clear Statement of the Business Task

## The ultimate goal for our stakeholders is to figure out if there are any trends we can identify between annual members (subscribers) to the bike rental service and casual riders(one-time users). We will utilize these insights to market towards casual riders in order to entice them to purchase an annual membership, as annual memberships show the biggest profit yields.

### A Description of All Data Sources Used

For this analysis, I have chosen to use RStudio and Tableau to showcase my insights. I will be working with a full year of data (August 2022 - July 2023) which contains over 5,000,000 rows of data, thus Excel did not feel like the proper tool to use for this analysis. 

The data comes from a repository provided by Google for this case study which contains more data outside the time scope. Further exploratory analysis into previous years' data could prove useful at a later date, as my timeline is for these deliverables is less than a week, and the stakeholders want relevant data post-pandemic in order to see what the current user trends are.

### Documentation of Any Cleaning or Manipulation of Data

Before I can perform any accurate exploratory analysis, it is important for me to join data into one table and clean the data. I am starting with RStudio to perform these actions.

#### 1. Before this code, I downloaded 12 different .csv files for 12 months of data and placed it into a folder on my Desktop called "Data_Cyclstic"

```R
library(tidyverse)
library(ggplot2)
library(forcats)
library(readxl)
library(skimr)
library(here)
library(janitor)
library(Tmisc)
library(SimDesign)

# Set the working directory to the folder containing CSV files

setwd("~/Desktop/Data_Cyclistic")
csv_files <- list.files(pattern = ".csv") # List all CSV files in the directory
combined_data <- data.frame() # Initialize an empty data frame to store the combined data

# Read and combine all CSV files into one data frame

for (file in csv_files) {
  df <- read.csv(file, header = TRUE, sep = ",")
  combined_data <- rbind(combined_data, df)
}

# Write the combined data to a new CSV file.
write.csv(combined_data, "Aug22July23_Combined.CSV", row.names = FALSE)

#assign dataframe to new combined CSV.
combined_df <- read.csv("Aug22July23_Combined.CSV")
glimpse(combined_df)
```

#### 2. After combining all 12 .csv files into one file is when I realized I would be working with over 5,000,000 rows of data. For most of this data, the majority of null's were in columns for station ID's and station names, which would interfere with the scope of the analysis. Thus I removed those rows from combined_df data frame.

```R
# Replaced 'columns_to_keep' with a vector of column names I want to keep.
columns_to_keep <- c("ride_id", "rideable_type", "started_at", "ended_at" , "start_lat", 
                     "start_lng", "end_lat", "end_lng", "member_casual") 
#remove start_station_id, start_station_id, end_station_name, end_station_id columns due to too many nulls.
combined_data <- combined_data[, columns_to_keep] #the []'s specify x and y axis variables to keep. We are keeping all x variables for integrity.
glimpse(combined_data)
```

#### 3. Now let's see if we can trim down the 5,723,606 rows a little bit. MySQL shouldn't have a problem with that many rows of data, but lets make sure we aren't working with duplicates, incorrect data types, or outliers that could interfere with our exploratory analysis. Also, I want to calculate the minutes it riders used bikes per ride_id, thus I added a couple columns to show this data.

```R
cleaned_data <- combined_data %>%
  mutate(
    started_at = as.POSIXct(started_at, format = "%Y-%m-%d %H:%M:%S"),  # Convert started_at to POSIXct
    ended_at = as.POSIXct(ended_at, format = "%Y-%m-%d %H:%M:%S"),      # Convert ended_at to POSIXct
    time_rented_seconds = as.numeric(difftime(ended_at, started_at, units = "secs")),  # Calculate time rented in seconds
    time_rented_minutes = time_rented_seconds / 60  # Calculate time rented in minutes
  ) %>% 
  filter(
    complete.cases(.),  # Filter out rows with missing values in any column
    time_rented_minutes >= quantile(time_rented_minutes, 0.10, na.rm = TRUE) &  # Filter based on 10th percentile
      time_rented_minutes <= quantile(time_rented_minutes, 0.90, na.rm = TRUE)   # Filter based on 90th percentile
  ) %>%
  distinct(
    ride_id, .keep_all = TRUE  # Keep only rows with unique ride_id values while preserving all columns
  )

glimpse(cleaned_data)  # Display the structure of the cleaned_data data frame
#Rows: 4,579,819
#Columns: 11
#$ ride_id             <chr> "8FE8F7D9C10E88C7", "34E4ED3ADF1D821B", "40759916B…
#$ started_at          <chr> "2023-04-02 08:37:28", "2023-04-19 11:29:02", "202…
#$ ended_at            <chr> "2023-04-02 08:41:37", "2023-04-19 11:52:12", "202…
#$ start_lat           <dbl> 41.80, 41.87, 41.92, 41.91, 41.91, 41.93, 42.00, 4…
#$ start_lng           <dbl> -87.60, -87.65, -87.65, -87.65, -87.63, -87.66, -8…
#$ end_lat             <dbl> 41.79, 41.93, 41.91, 41.91, 41.92, 41.91, 41.99, 4…
#$ end_lng             <dbl> -87.60, -87.68, -87.65, -87.63, -87.65, -87.65, -8…
#$ member_casual       <chr> "member", "member", "member", "member", "member", …
#$ time_rented_seconds <int> 249, 1390, 219, 290, 244, 552, 298, 659, 1204, 639…
#$ time_rented_minutes <dbl> 4.150000, 23.166667, 3.650000, 4.833333, 4.066667,…

max(cleaned_data$time_rented_seconds) #returned [1] 1770
max(cleaned_data$time_rented_minutes) #returned [1] 29.5
min(cleaned_data$time_rented_seconds) #returned [1] 193
min(cleaned_data$time_rented_minutes) #returned [1] 3.216667
```

4. Now that the data is cleaned I can now import this data into a new .csv file.
```R
write.csv(cleaned_data, "Aug22July23_Final.csv", row.names = FALSE)
```

## Visualizations and Key Findings

After I exported the data from RStudio I imported the csv to Tableau where I created this dashboard. This revealed various insights! https://public.tableau.com/views/Cyclistic_16950873000640/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link

### Here is what the Tableau dashboard reveals about causal and member behavior with using Cyclistic bikes:

1. Both members and casual users prefer to use electric bikes over classic bikes.
2. Members tend to make great use of their membership and make up the majority of rides in the data.
3. Both members and casual users use Cyclistic bikes during the colder months and use them more during the warmer months.
4. In January, there was a small increase in the number of bikes used by members.
5. Members tend to use Cyclistic bikes more during the work week, while casual users tend to use them during weekends.
6. Although, members use Cyclistic most often, casual users have longer rides on average for both bike types.

# My top three recommendations based on my analysis

1. Any marketing material created to show off Cyclistic's membership program should showcase electric bikes over their classic bikes.

2. Since members tend to use the bikes during work days, we can assume this is because they use the bikes to commute to and from work in the metropolitain area. Marketing team should generate marketing material to entice casaul users to use their bikes to commute to work. Perhaps even offer a 2 month promotion for a low fee to join the membership.

3. If we are to expect more casual users to become members, it means we will need more staff and bikes to meet the demand as it is shown that members use Cyclistic bikes more than casual users.

# Final Thoughts

After coming up with three conclusions about the data, I'd like to address some limitations of the data:

I would also suggest that if the stakeholders want deeper insight on their members, demographic data would be very helpful to see which age/gender are more likely to sign up as members.
Also, any promotional data would be helpful to to see if casual riders are more likely to sign up for recurrent years after jumping in with a promotion.
The data given also did not have complete data for the majority of data points in station_name and station_id fields.
