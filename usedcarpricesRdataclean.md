###  When loading RStudio, this is the first script I run to load all my packages I use frequently:
```R
library(tidyverse)
library(ggplot2)
library(dplyr)
library(tidyr)
library(readr)
library(lubridate)
library(forcats)
library(readxl)
library(skimr)
library(here)
library(janitor)
library(Tmisc)
library(SimDesign)
library(writexl)

# Set global options or configurations as needed
options(stringsAsFactors = FALSE)  # Example: Disable automatic conversion of strings to factors
```
### Let's load in our csv:

```R
file_path <- "/kaggle/input/used-car-price-prediction-dataset/used_cars.csv"
car_df <- read.csv(file_path)
View(car_df)
```

### Before I clean the data I want to explore columns to see what data cleaning might be required

```R
fuel_counts <- table(car_df$fuel_type)
print(fuel_counts)
```
This code revealed that nulls are represented by "-". Empty cells are actually electric vehicles so I will be replacing actual nulls with "electric" in my cleaning.

```R
transmission_counts <- unique(car_df$transmission)
print(transmission_counts)
```

This revealed a lot of categories that only appear once in the dataset. I will assign "Other A/T" to represent outlier transmissions that operate like an Automatic or "Other M/T" to those transmissions that operate like a Manual.

```R
model_year_counts <- unique(car_df$model_year)
print(model_year_counts) 
```

All the years look normal so we won't need to clean anything in this column.

```R
brands_vector <- c("Toyota", "Ford", "Chevrolet", "Honda", "Hyundai")
fueltypeXbrandmodel <- car_df %>%
  filter(fuel_type == "-" & brand %in% brands_vector) %>%
  select(fuel_type, brand, model, model_year)
print(fueltypeXbrandmodel) #this code makes sure there are no "-" values in the fuel_type column that meet the requirement of the top 5 car brands.
```

## Now it's time to clean the dataframe car_df and assign it to a new data frame car_df_cleaned.
```R
  car_df_cleaned <- car_df %>%
    rename(mileage = milage) %>% #correct Milage header to proper spelling.
    mutate(
      model_year = as.integer(substr(model_year, 1, 4)), # change model year to integer and limit digits to 4.
      mileage = as.integer(gsub("[^0-9]", "", mileage)), #change mileage column to integer
      price = as.integer(gsub("[^0-9]", "", price)), #change price column to integer
      fuel_type = ifelse(is.na(fuel_type) | fuel_type == "", "Electric", fuel_type), # this line changes nulls to "Electric" and removes rows with "-" which are the actual null values in the column.
      transmission = case_when(
        transmission %in% c("1-Speed A/T", "1-Speed Automatic", #the following mutations turn all of these transmissions into three categories "Automatic", "Manual", and "CVT".
                            "10-Speed A/T", "10-Speed Automatic", "Automatic, 10-Spd", "10-Speed Automatic with Overdrive",
                            "2-Speed A/T", "2-Speed Automatic",
                            "4-Speed A/T", "4-Speed Automatic",
                            "5-Speed A/T", "5-Speed Automatic",
                            "8-SPEED A/T", "8-Speed A/T", "8-Speed Automatic", "8-Speed Automatic with Auto-Shift", "8-SPEED AT", "Automatic, 8-Spd Dual-Clutch",
                            "Automatic, 8-Spd", "Automatic, 8-Spd Sport w/Sport & Manual Modes", "Automatic, 8-Spd PDK Dual-Clutch",
                            "Automatic, 8-Spd M STEPTRONIC w/Drivelogic, Sport & Manual Modes",
                            "6-Speed A/T", "6 Speed At/Mt", "6-Speed", "6-Speed Automatic", "6-Speed Electronically Controlled Automatic with O ",
                            "6-Speed Automatic with Auto-Shift", "Auto, 6-Spd w/CmdShft", "6 Speed At/Mt", "6-Speed Electronically Controlled Automatic with O",
                            "7-Speed A/T", "7-Speed Automatic", "7-Speed", "7-Speed Automatic with Auto-Shift",
                            "Automatic, 7-Spd S tronic Dual-Clutch", "7-Speed DCT Automatic",
                            "9-Speed Automatic", "9-Speed A/T", "9-Speed Automatic with Auto-Shift", "Automatic, 9-Spd 9G-Tronic", 
                            "A/T", "Automatic", "F", "Transmission Overdrive Switch"
                            ) ~ "Automatic",
        transmission %in% c("Automatic CVT", "CVT Transmission", "CVT-F", "Variable"
                            ) ~ "CVT",
        transmission %in% c("1-Speed M/T",
                            "2-Speed M/T",
                            "3-Speed M/T",
                            "4-Speed M/T",
                            "5-Speed M/T",
                            "6-Speed M/T", "6-Speed Manual", "Manual, 6-Spd", "6 Speed Mt",
                            "7-Speed M/T", "7-Speed Manual",
                            "8-Speed M/T", "8-Speed Manual",
                            "9-Speed M/T",
                            "M/T", "Manual", "Transmission w/Dual Shift Mode"
                            ) ~ "Manual",
        TRUE ~ "Other" # All other transmissions that don't match Automatic or Manual
      )
    ) %>%
    filter( #here is where we are filtering out outliers that don't fit the transmission categories created in the mutations
      transmission != "Single-Speed Fixed Gear",
      transmission != "SCHEDULED FOR OR IN PRODUCTION",
      transmission != "2",
      transmission != "-",
      transmission != "Other",
      fuel_type != "not supported", 
      between(mileage, 100, 9999999) # Filter mileage between 3 and 6 digits
    ) %>% 
    filter(!(is.na(clean_title) | clean_title == "")) #there are no null's in the clean_title column, but there are empty strings so we will be removing those as well.
  View(car_df_cleaned)
```

### Now that we have cleaned our data, let's create a new data frame that only contains the top 5 car brands

```R
top5brands <- c("Toyota", "Ford", "Chevrolet", "Honda", "Hyundai")
filtered_df <- car_df_cleaned %>%
    filter(brand %in% top5brands)
View(filtered_df)
```

### At this point I am pretty happy with how the data frame is looking. I might need to do a bit more cleaning in Excel, but nothing too major. Let's export this to an xlsx.
```R
write_xlsx(filtered_df, path = "~/Professional Documents/Data Analytics Projects/topfivebrandsusedcar.xlsx", col_names = TRUE)
```
