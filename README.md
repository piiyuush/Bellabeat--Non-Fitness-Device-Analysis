# Bellabeat, Non-Fitness Device Analysis: How Can a Wellness Technology Company Play It Smart? 

## A Google Data Analytics Professional Certificate Capstone Project


<img align="right" width="500" height="400" src="[https://repository-images.githubusercontent.com/522069892/740a3949-ac89-4392-b167-194a026604ed](https://images.everydayhealth.com/images/healthy-living/fitness/everything-you-need-know-about-fitness-1440x810.jpg?sfvrsn=2fee0a3b_5)">

[Bellabeat]( https://bellabeat.com/) is a company specializing in crafting fitness products tailored for women. Their product lineup encompasses intelligent hydration vessels, stylish fitness timepieces and accessories, as well as yoga mats. Users can conveniently access their health metrics gathered by these gadgets through the Bellabeat app.

Bellabeat’s co-founders would like to examine data from third-party fitness devices to understand how consumers utilize these products. The company aims to leverage these insights to inform future marketing strategies.

## Ask

### Key stakeholders

1. Urška Sršen: Co-founder and Chief Creative Officer of Bellabeat.
3. Sando Mu: Co-founder of Bellabeat and mathematician.
4. The Bellabeat marketing analytics team: A group of data analysts tasked with collecting, analyzing, and presenting data to inform Bellabeat's marketing strategy.

### Bellabeat products

* Bellabeat app: The Bellabeat app offers users a comprehensive view of their health data, covering activity, sleep, stress, menstrual cycle, and mindfulness habits. This information empowers users to gain insights into their behaviors and make informed, healthy choices. Seamlessly integrating with Bellabeat's range of smart wellness products, the app serves as a hub for tracking and monitoring.

* Leaf: Bellabeat's iconic wellness tracker, the Leaf, can be worn as a bracelet, necklace, or clip. It syncs with the Bellabeat app to monitor activity, sleep, and stress levels, providing users with valuable health insights.

* Time: Combining the elegance of a traditional timepiece with smart technology, the Time wellness watch tracks user activity, sleep, and stress. Connecting to the Bellabeat app, it offers users daily wellness insights to enhance their overall health.

* Spring: The Spring water bottle utilizes smart technology to track daily water intake, ensuring users stay adequately hydrated throughout the day. By connecting to the Bellabeat app, it provides real-time updates on hydration levels, promoting better health habits.

* Bellabeat membership: Bellabeat offers a subscription-based membership program, granting users access to personalized guidance on nutrition, activity, sleep, health and beauty, and mindfulness. Tailored to individual lifestyles and goals, this membership provides 24/7 support to enhance overall well-being.

### Business task

Analyze non-Bellabeat smart device data and compare with one Bellabeat product to discover insights to help guide marketing strategies for the company.

### Key Questions

1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat marketing strategy?

## Prepare 

### Data source: 

FitBit Fitness Tracker Data on [Kaggle]( https://www.kaggle.com/datasets/arashnic/fitbit) in 18 CSV files. The data contains smart health data from personal fitness trackers for thirty fitbit users. The data was collected via a survey of personal tracker data, including minute-level output for physical activity, hear rate, and sleep monitoring, through Amazon Mechanical Turk between March 12, 2016 and May 12, 2016. It was updated two years ago as of August 2022. The data includes information about daily activity, steps, and heart rate. 

### Limitations: 

* The sample size is limited, encompassing only 30 individuals.
* The dataset is outdated, spanning six years, and may not reflect the current capabilities of FitBit devices, which may now offer more precise measurements.
* Due to the survey-based data collection method, the accuracy of the results may be compromised as participants may not always provide truthful or precise responses.
* Weight-related data is particularly sparse, with information available for only eight users. Additionally, a significant portion of entries in this category are missing, and approximately two-thirds of weight entries were manually inputted, potentially impacting the reliability of the data.


For a more throuogh look at the data see the [Bellabeat Data Dictionary and Documentation]([https://github.com/CoolBeansProgramming/Bellabeat-Case-Study/blob/main/Data%20Documentation%20and%20Data%20Dictionary.md](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/BellaBeat_Data%20Documentation%20and%20Data%20Dictionary.md)) file. 

# Process

## Choosing Data Files

As `dailyActivity_merged.csv ` provides a good summary of steps and calories burned and the `sleepDay_merged.csv` file provides sleep data these are good overall files to use to analyze patricipant usage. As fitness devices are generally used to track overall health and weight, the file `weightLogInfo_merged` containing weight data will also be used. 


## Applications
Excel will be used to load and take an initial pass for issues, R to transform and explore the data, and Tableau to interactively visuallize the data. 

## Initial Pass Through 

1) Make sure there are no blank entries in the data by using filters.
2) Convert Id field to text data type as no numerical equations are needed for this field. 
3) Convert ActivityDate from Datetime to Date types as no times are given in the data. 
4) In the `dailyActivity_merged.csv ` file, there are many instances where TotalSteps is zero and SedentaryMinutes is 1440; the number of calories burned vary between users. This is most likely due to the weight and height of the user. There are a few instances where the sedentary minutes is 1440 but the calories burned is 0. 
5) In the `weightLogInfo_merged` file, there are only two entries for the Fat field so this will not be used to draw insights. 

## Transform and Explore 
All R code can be found [here]([https://github.com/CoolBeansProgramming/Bellabeat-Case-Study/blob/main/BellaBeat_RScript.R](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/BellaBeat_RScript_data.R)).

1) Load the tidyverse package and data files 
2) Check to see if the data has been loaded correctly
3) Convert the Id field to character data type 
4) Rename ActivityDate, SleepDay, and Date to convert to date data type 

```
activity <-activity %>%
  mutate_at(vars(Id), as.character) %>%
  mutate_at(vars(ActivityDate), as.Date, format = "%m/%d/%y") %>%
  rename("Day"="ActivityDate") 
```
  
  
5) Combine data frames using left and right joins
6) Add day of the week variable 

```
combined_data <-sleep %>%
  right_join(activity, by=c("Id","Day")) %>%
  left_join(weight, by=c("Id", "Day")) %>%
  mutate(Weekday = weekdays(as.Date(Day, "m/%d/%Y")))
```

7) Filter and remove duplicate rows; count NAs and distinct entries using Id

```
combined_data <-combined_data[!duplicated(combined_data), ]
sum(is.na(combined_data))
n_distinct(combined_data$Id)
n_distinct(sleep$Id)
n_distinct(weight$Id)
```

The final data frame has 940 variables with 25 variables. There are 33 distinct Id entries total. The number of distinct users in dailyActivity, sleepDay, and weightLogInfo are 33, 24, and 8, respectively. There are 6893 NAs in the combined data. This is not surprising as there is only weight data from eight users and not all users logged sleep information. 

# Analyze 

## Select summary statistics and visualizations 

```
combined_data %>%
select(TotalMinutesAsleep, TotalSteps, TotalDistance, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryMinutes, Calories, WeightKg, Fat, BMI, IsManualReport) %>%
summary()
```


![Summary Statistics](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/Summary%20Statistics.png?raw=true)

The average user weighs 72.04 kg, has a BMI of 25.19, and spent the most time doing light activities. On average, they also slept 6.9 hours, took 7638 steps, and traveled 5.49 km per day. 

![Steps by Day](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/IMG/Total%20steps%20by%20day.jpg?raw=true)

Users took the most steps on Sundays and the least number of steps on Fridays. As all the values are fairly high, the marketing team can conclude that users value the step feature of health fitness devices. They could also assume that the feature will be very useful for Bellabeat customers. 

![Fairly Active Minutes by Day](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/IMG/Minutes%20of%20moderate%20activity%20per%20day.jpg?raw=true)

It is interesting to see that the amount of time spent being fairly active decreased on Wednesdays and then picks back up on Thursdays. This may be due to the fact that most people are going back to work on Monday and then may get discouraged or tired by Wednesday. Wednesday may also be a popular rest day, allowing them to resume their activities on Thursday. 

![Distribution of Total Sleep Time](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/IMG/Distribution%20of%20sleep%20time.jpg?raw=true)

From the above histogram, most people slept between 312 and 563 minutes (between 5.2 and 9.4 hours). Note that this does not include the total time spent in bed resting.  

![Calories vs Time Slept](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/IMG/Total%20minutes%20Asleep%20vs%20Calories.jpg?raw=true)

Besides a few outliers, calories were burned by those who slept between 5 and 7 hours. If only considering weight loss and calories burned, this aligns with the 5.2 to 9.4 hour sleep range, which may indicate that those who stay withint this range burn more calories.

![Logged Activity Distance by Day](https://github.com/piiyuush/Bellabeat--Non-Fitness-Device-Analysis/blob/main/IMG/Logged%20Activities%20Distance.jpg?raw=true)

The logged feature was not used too often as there were many blanks in the data and no records were available  for Thursday and Friday. The highest days of logged distance were on the weekend or times when many people likely have free time to do physical activities. 

## Share 

Check out the daashboard on Tableau Public: [Bellabeat Dashboard](https://public.tableau.com/views/BellabeatNon-FitnessDeviceAnalysisCaseStudy/SleepbyWeekday?:language=en-US&publish=yes&:sid=&:display_count=n&:origin=viz_share_link)

## Act

* The number of steps users took was the least on Friday which may be due to user becoming tired at the end of the week. As this is not limited to only FitBit customers, the marketing team could send notifications to users Thursday evenings and Friday and Saturday mornings encouraging users to continue being physical active throughout the day.

* Many users did not use the Logged Distance feature on the FitBit devices. This suggests that users would prefer to have their data collected automatically. The Bellabeat marketing team can decide not to a feature activity distance log function as many users seem to not use this.

* Compared to the data set size, there were very few entries for weight. Of those that were entered, about 2/3 were done manually. The individuals who did not log their weight may not have been concerned with losing weight or did not have the device needed to automatically record this data. Since many did not use the logged distance feature as well, the Bellabeat team could market weight devices like smart scales that automatically record this information.

* Other data sources, like the [Mi Band fitness tracker data (04.2016 - present)](https://www.kaggle.com/datasets/damirgadylyaev/more-than-4-years-of-steps-and-sleep-data-mi-band), could be useful for further exploration as this specific data set follows on individual over the course of six years. 


