# How Does a Bike-Share Navigate Speedy Success?
## Cyclistic Case Study

---

## 1. Introduction

Cyclistic is a bike-sharing company operating in Chicago. Since its launch in 2016, the company has experienced rapid growth in ridership. Despite this success, Cyclistic’s marketing team believes that long-term profitability depends on increasing the number of annual members, as annual memberships generate more stable and predictable revenue compared to casual rides.

The objective of this case study is to identify behavioural differences between casual riders and annual members, and to explore how these insights can support strategies to convert casual riders into long-term subscribers.

Using historical trip data, this analysis examines ride duration, usage frequency, and weekly riding patterns across different user types. The project demonstrates how raw data can be transformed into actionable business insights and highlights the role of data-driven decision-making in supporting sustainable business growth.

---

## 2. Data Process and Analysis

This case study is based on a hypothetical company named **Cyclistic**, using real-world public data from Chicago’s **Divvy bike-sharing program**, operated by Motivate International Inc.

The dataset consists of **Q1 trip records from 2019 and 2020**, and includes:
- Ride start and end timestamps  
- User type (casual rider or annual member)  
- Bike type  
- Station and location information  

All data usage complies with the Divvy Data License Agreement. No personally identifiable information is used, and no attempt is made to identify individual users. This analysis does not imply endorsement or affiliation with Lyft Bikes and Scooters, LLC, or the City of Chicago.

---

## 3. Data Preparation and Cleaning

### Using Microsoft Excel

To enable effective analysis, additional derived variables were created.

- A new column, **`ride_length`**, was calculated as the difference between `started_at` and `ended_at`.  
  Ride duration was formatted as **HH:MM:SS**, allowing comparison of ride lengths across users.

- A **`day_of_week`** column was generated using the `WEEKDAY()` function to identify weekly riding patterns.

These derived variables were essential for comparing behaviour between casual riders and annual members.

![Sample dataset showing ride_length and day_of_week](https://github.com/user-attachments/assets/dd0686ba-9494-45f6-b7f8-c363f34a5b97)

*Figure 1: Sample dataset showing `started_at`, `ended_at`, `ride_length`, and `day_of_week`.*

---

### Alternatively, Using SQL

Q1 2019 and Q1 2020 data were combined using Excel and uploaded to BigQuery.  
A temporary table was created to calculate ride duration in seconds and derive the day of week from the start time.

```
WITH ride_length_table AS
  (SELECT
    member_casual,
    TIMESTAMP_DIFF(ended_at, started_at,SECOND) AS ride_length,
    EXTRACT(DAYOFWEEK FROM started_at) AS day_of_week
    FROM `utility-mapper-475309-f6.cyclists_bike_share.Combined sheet` 
  )
```
## 4. Calculations and Exploratory Analysis

### Using Excel

Pivot tables were used to summarise:
Total number of rides
Average ride length
Minimum and maximum ride duration
Ride distribution by user type and day of week
These summaries enabled clear comparison of usage behaviour between casual riders and annual members.
<img width="696" height="415" alt="Screenshot 2026-01-05 at 10 44 07 am" src="https://github.com/user-attachments/assets/fc561824-6bff-4afb-ba60-ef42b3ddb1b2" />

*Figure 2: Summaries regarding overall statical summary, average of `ride_length` grouped by `member_casual`(usertype), average of `ride_length` grouped by `member_casual`(usertype) and `day_of_week`.*


### Alternatively, Using SQL
The following query aggregates ride behaviour by `member_casual`(usertype) and `day_of_week`, representing `member_casual`(usertype), `ride_length`, `day_of_week`(Sunday was indicated by 1 and so forth), `ride_count`, `avg_ride_length`, `max_ride_length` and `min_ride_length`:
```
WITH ride_length_table AS
  (SELECT
    member_casual,
    TIMESTAMP_DIFF(ended_at, started_at,SECOND) AS ride_length,
    EXTRACT(DAYOFWEEK FROM started_at) AS day_of_week
    FROM `utility-mapper-475309-f6.cyclists_bike_share.Combined sheet` 
  )

## above was copied from step 3

SELECT 
  member_casual,
  day_of_week, 
  COUNT(*) AS ride_count,
  ROUND(AVG(ride_length),0) AS avg_ride_length,
  ROUND(MAX(ride_length),0) AS max_ride_length,
  ROUND(MIN(ride_length),0) AS min_ride_length
FROM ride_length_table
GROUP BY 
  member_casual, 
  day_of_week
ORDER BY
  member_casual, 
  day_of_week  
```

<img width="897" height="501" alt="Screenshot 2026-01-05 at 11 56 17 am" src="https://github.com/user-attachments/assets/d802bf16-b673-40da-a0a8-83e9eccf4932" />

*Figure 3: Outcomes showing `member_casual`(usertype), `ride_length`, `day_of_week`(Sunday was indicated by 1 and so forth), `ride_count`, `avg_ride_length`(in second), `max_ride_length` and `min_ride_length`, grouped by usertype and `day_of_week`.*

Export the result to local drive in format of a csv file, naming it `summary_from_SQL`.

### Using R for visualization
#### To find out association between `avg_ride_length` and `day_of_week` for both member and casual, first upload the csv file from local drive to R Studio.
Then start generating visualization regarding it:

```
View(Combined_sheet_Divvy_Trips_2020_Q1)
install.packages("tidyverse")
library(tidyverse)

> ggplot(data = summary_from_SQL) +
+     geom_smooth(
+         aes(
+             x = day_of_week,
+             y = avg_ride_length,
+             colour = member_casual
+         )
+     ) +
+     labs(
+         title = "Average Ride Length by Day of Week",
+         subtitle = "Cyclistic Trips (2019 Q1 & 2020 Q1): Members vs Casual Riders"
+     )
```
<img width="865" height="612" alt="Average Ride Length by Day of Week" src="https://github.com/user-attachments/assets/0235c33c-701c-4baf-850d-8dccba01daeb" />

*Figure 4: Average ride length by day of week.*

#### To find out association between `ride_count` and `day_of_week` for both member and casual users, generate another graph:
```
ggplot(data = summary_from_SQL) +
+     geom_smooth(
+         aes(
+             x = day_of_week,
+             y = ride_count,
+             colour = member_casual
+         )
+     ) +
+     labs(
+         title = "Ride Count by Day of Week",
+         subtitle = "Cyclistic Trips (2019 Q1 & 2020 Q1): Members vs Casual Riders"
+     )
```

<img width="865" height="604" alt="Ride Count by Day of Week" src="https://github.com/user-attachments/assets/1d072413-4f53-4be6-b1c1-cde38bab9278" />

*Figure 5: Ride Count by day of week.*

