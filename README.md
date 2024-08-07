# Bellabeat Case Study
## Rounak Saha
### 2024-06-03

# Introduction
Urška Sršen and Sando Mur founded Bellabeat, a high-tech company that manufactures health-focused smart products. Sršen used her background as an artist to develop beautifully designed technology that informs and inspires women around the world. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their own health and habits. Since it was founded in 2013, Bellabeat has grown rapidly and quickly positioned itself as a tech-driven wellness company for women.

My report will include the following deliverables:

* A clear summary of the business task
* A description of all data sources used
* Documentation of any cleaning or manipulation of data
* A summary of your analysis
* Supporting visualizations and key findings
* Your top high-level content recommendations based on your analysis

## Step 1: Ask
### Business Task
To analyze one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The insights will then help guide marketing strategy for the company.

#### 1.1 Key stakeholders:
* *Urška Sršen*: Bellabeat’s cofounder and Chief Creative Officer

* *Sando Mur*: Mathematician and Bellabeat’s cofounder; key member of the Bellabeat executive team

* *Bellabeat marketing analytics team*: A team of data analysts responsible for collecting, analyzing, and
reporting data that helps guide Bellabeat’s marketing strategy.

#### 1.2 Questions to explore for the analysis:
* What are some trends in smart device usage?
* How could these trends apply to Bellabeat customers?
* How could these trends help influence Bellabeat marketing strategy?

## Step 2: Prepare
The data being used in this case study can be found here: [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit) CC0: Public Domain, dataset made available through [Mobius](https://www.kaggle.com/arashnic)

The data is stored and uploaded in R Studio. This Kaggle data set contains personal fitness tracker from thirty fitbit users. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits.

The data set contains 18 CSV files organized in long format. Below is a breakdown of the data using the ROCCC approach:

* Reliability - **LOW**: The data comes from 30 fitbit users with unknown demographics who consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring.
* Original - **LOW**: Third party data collected using Amazon Mechanical Turk.
* Comprehensive - **MED**: The dataset contains multiple fields on daily activity intensity, calories used, daily steps taken, daily sleep time and weight record.
* Current - **LOW**: This data is from March 2016 through May 2016. The data is not current, meaning that user habits may have changed over the years.
* Cited - **LOW**: Data was collected from a third party, therefore unknown.

## Step 3: Process
We will be installing and loading all necessary packages for data wrangling 

```{r, echo = TRUE, eval = TRUE, results="hide", fig.keep = "none"}
install.packages("tidyverse", repos="https://cloud.r-project.org/")
install.packages("readr", repos="https://cloud.r-project.org/")
install.packages("dplyr", repos="https://cloud.r-project.org/")
install.packages("tidyr", repos="https://cloud.r-project.org/")
install.packages("ggplot2", repos="https://cloud.r-project.org/")
install.packages("lubridate",repos="https://cloud.r-project.org/")
install.packages("sqldf",repos="https://cloud.r-project.org/")
```


```{r, echo=TRUE, fig.keep="none", results="hide"}
library(tidyverse)
library(readr)
library(dplyr)
library(tidyr)
library(ggplot2)
library(lubridate)
library(utils)
library(sqldf)
library(knitr)
```
We will now load the dataframes using `read.csv()` function
```{r, echo = TRUE, eval = TRUE, results="hide", fig.keep = "none"}

activity <- read.csv("~/Desktop/DA prac/Tableau + SQL +EXCEL project/Bellabeat case study/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")

calories <- read.csv("~/Desktop/DA prac/Tableau + SQL +EXCEL project/Bellabeat case study/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailyCalories_merged.csv")

intensities <- read.csv("~/Desktop/DA prac/Tableau + SQL +EXCEL project/Bellabeat case study/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailyIntensities_merged.csv")

steps <- read.csv("~/Desktop/DA prac/Tableau + SQL +EXCEL project/Bellabeat case study/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailySteps_merged.csv")

heart_rate <- read.csv("~/Desktop/DA prac/Tableau + SQL +EXCEL project/Bellabeat case study/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/heartrate_seconds_merged.csv")

sleep <- read.csv("~/Desktop/DA prac/Tableau + SQL +EXCEL project/Bellabeat case study/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")

weight <- read.csv("~/Desktop/DA prac/Tableau + SQL +EXCEL project/Bellabeat case study/archive/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
```

Lets take a look at the dataframes we loaded using `glimpse()` function
```{r, echo = TRUE, eval = TRUE, fig.keep = "none"}
glimpse(activity)
glimpse(calories)
glimpse(heart_rate)
glimpse(intensities)
glimpse(sleep)
glimpse(steps)
glimpse(weight)
```

According to observation we can see that *calories*, *intensities* and *steps* are a subset of *calories* dataframe. In order to be sure, we can check using `sqldf()` function

```{r}
# check for calories

check_calories <- sqldf("select Id,ActivityDate,Calories
               from activity
               intersect
               select Id,ActivityDay,Calories
               from calories")
head(check_calories)
nrow(check_calories)

# for steps
check_steps <- sqldf("select Id,ActivityDate, totalsteps 
                     from activity
                     intersect
                     select Id,ActivityDay,Steptotal
                     from steps")
head(check_steps)
nrow(check_steps)

# for intensities
check_intensities <- sqldf("select Id, ActivityDay, SedentaryMinutes, LightlyActiveMinutes, FairlyActiveMinutes, VeryActiveMinutes,  SedentaryActiveDistance, LightActiveDistance, ModeratelyActiveDistance ,VeryActiveDistance
                           from intensities
                           intersect
                           select Id, ActivityDate, SedentaryMinutes, LightlyActiveMinutes, FairlyActiveMinutes, VeryActiveMinutes,  SedentaryActiveDistance, LightActiveDistance, ModeratelyActiveDistance ,VeryActiveDistance
                           from activity")
head(check_intensities)
nrow(check_intensities)
```
All of the 3 checks returned 940 row, so we can conclude, these are the subsets of *activity* data frame and can remove them for simplicity

```{r}
rm(check_calories,check_intensities,check_steps, calories, intensities, steps)
```

## Step 4: Analyse
To begin the analysis phase, we will first see how many participants there are in each category.

Checking the sample size of our dataframe 
```{r}
n_distinct(activity$Id)
n_distinct(heart_rate$Id)
n_distinct(sleep$Id)
n_distinct(weight$Id)
```
Checking significant changes in weight and BMI
```{r}
weight %>% group_by(Id) %>% summarise(min(WeightKg), max(WeightKg), min(BMI), max(BMI))
```
We will not include *heart_rate* and *weight* data frames in our analysis since they have very less sample size from our observation above using `n_distinct()` function and there is hardly any changes in weight for the participants. 8 and 14 participants are not enough to draw strong conclusion based on this data. Rather, sample size for *sleep* dataframe is also small but we will keep it for reference.

```{r}
# Removing heart_rate and weight data frames
rm(heart_rate, weight)
```

```{r}
# Checking for duplicates 
sum(duplicated(activity))
sum(duplicated(sleep))
```

```{r}
#Remove duplicates
sleep <- unique(sleep)
sum(duplicated(sleep))
```
```{r}
# Checking NA's
sum(is.na(activity))
sum(is.na(sleep))
```

Since R is a case-sensitive, for simplicity we can change the variable names from camel-case to lowecase.

```{r}
# Formatting variable names
activity <- rename_with(activity, tolower)

sleep <- rename_with(sleep, tolower)
```

We need to change the data type of date column since it is in character using `as.date()` function.

```{r}
# Changing date format 
activity$date <-as.Date(activity$activitydate, format = "%m/%d/%Y")

sleep$date<- as.Date(sleep$sleepday, format = "%m/%d/%Y")
```

```{r}
# Lets look at summary statistics of the activity dataset
activity %>% 
select(totalsteps,calories,veryactiveminutes,fairlyactiveminutes,lightlyactiveminutes,sedentaryminutes) %>% 
summary()

# Lets look at summary statistics of the sleep dataset
sleep %>% 
select(totalsleeprecords,totalminutesasleep) %>% 
summary()
```

##### Some imporrtant discoveries were made from the summary:

* Average Sedentary time is 991 minutes
* Most of the population is lightly active
* Average participants sleep one time a day for 7 hours
* Average person burns 96kcal in an hour 
* Daily average steps taken is 7638. The CDC recommends people to take 10000 steps a day

It is important to check the co-relation between sleep and activity because most of the times one is affected by other. In order to check that, we are going to merge the two data frames by the columns *Id* and *date*.

```{r}
# Merging the dataframes 
daily_activity_sleep <- merge(activity, sleep, by=c("id", "date"))
```

```{r}
daily_activity_sleep %>% 
group_by(id) %>% 
count(id) %>% 
  head()
```

## Step 5: Share
In this step, we will try to find trends and relationship among the participants by finding correlations between different variables.

Firstly, we will check the correlation between calories burned vs total steps taken  

```{r}
ggplot(activity) + 
  aes(x= totalsteps, y= calories) + 
  geom_smooth(color = 'gold3', fill= "skyblue")+ 
  geom_point(aes(color= calories)) +
  labs(x= 'Total Steps', y= 'Calories burned', title = "Calories burned vs Total steps",caption = "Data Collected by FitBit Fitness Tracker Data") + 
  theme(text = element_text(size = 12), legend.position = 'right')
```

We can see a positive correlation between the two, which is obvious- the more active we are, the more calorie we burn.

Next, we will check the trends among total distance vs calories burned 

```{r}
ggplot(activity)+
  aes(x= totaldistance, y= calories)+
  geom_point(aes(colour = calories))+
  geom_smooth(color = 'gold3', fill='skyblue')+
  labs(x= 'Total Distance', y= 'Calories burned', title = 'Total Distance vs Calories burned', caption = "Data Collected by FitBit Fitness Tracker Data")+
   theme(text = element_text(size = 12), legend.position = 'right')
```

The plot displays a positive trend, indicating that a person tends to burn more calories when they move longer.

Next, we will look into the sleep dataset to understand the relation between the time asleep and total time spent in bed.

```{r}
sleep %>% 
  ggplot(aes(x= totalminutesasleep, y= totaltimeinbed))+
  geom_point(aes(colour = totaltimeinbed))+
  geom_smooth(color ='gold3',fill="skyblue")+
  labs(x= 'Total minutes asleep', y= 'Total time in bed', title = 'Total Time Asleep vs Total Time Spent in Bed',caption = "Data Collected by FitBit Fitness Tracker Data")+
  theme(text = element_text(size = 12), legend.position = 'right')
```

The relationship between sleep time and total time in bed looks linear. So we can improve a feature to notify the users during their sleep time.

I intend to slice and dice the data set in order to get a deeper look into the average amount of calories, steps, distance and hours asleep recorded for each individual per weekday. This can uncover some interesting insights. First, I'll create a new data frame for this;

```{r}
daily_activity_sleep_summary <- daily_activity_sleep %>% 
  select(id, date, totalsteps, totaldistance,calories, totalminutesasleep)
  
head(daily_activity_sleep_summary)
```

Weekday Summary: 

```{r}
weekday_summary <- daily_activity_sleep_summary %>% 
  mutate(weekday= weekdays(date), sleep = totalminutesasleep/60)

weekday_summary$weekday <- ordered(weekday_summary$weekday, levels= c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))

weekday_summary <- weekday_summary%>% 
  group_by(weekday) %>% 
  summarise( steps= mean(totalsteps), distance= mean(totaldistance), sleep= mean(sleep), calories= mean(calories))
```

Lets visualize the amount of calories burnt and steps taken per day

```{r}
# Calories burned per weekday
ggplot(weekday_summary) +
  aes(x= weekday, y= calories)+
  geom_col(aes(color= weekday, fill= weekday))+
  labs(x= '', y= 'Calories burned', title = 'Calories burned per Weekday', caption = "Data Collected by FitBit Fitness Tracker Data")+
  theme(axis.text.x = element_text(angle = 45, hjust = 1))+ theme_classic()
```




```{r}
# Steps taken per weekday
ggplot(weekday_summary) +
  aes(x= weekday, y= steps)+
  geom_col(aes(color=weekday, fill=weekday))+
  labs(x= '', y= 'Total Steps',title = 'Steps taken per Weekday', caption = "Data Collected by FitBit Fitness Tracker Data")+
theme(axis.text.x = element_text(angle = 45, hjust = 1))+
  geom_hline(yintercept = 10000)+ theme_classic()
```

Now lets visualize the distance covered and sleep hours per weekday

```{r}
# Distance per weekday
ggplot(weekday_summary)+
  aes(x=weekday, y= distance)+
  geom_col(aes(fill = weekday, color= weekday))+
  labs(x= "", y="Distance", title = "Distance(km) per Weekday", caption = "Data Collected by FitBit Fitness Tracker Data")+
  theme(axis.text.x = element_text(angle = 45, hjus= 1))+ theme_classic()
```

```{r}
# Sleep hours per weekday
ggplot(weekday_summary)+
  aes(x= weekday, y=sleep)+
  geom_col(aes(colour = weekday, fill = weekday))+
  labs(x= "", y="Sleep Hours", title = "Sleep Hours per Weekday", caption = "Data Collected by FitBit Fitness Tracker Data")+
  theme(axis.text.x = element_text(angle = 45, hjus= 1))+
  geom_hline(yintercept = 8)+ theme_classic()
```

From the above summary graph we can say the following:

* Users burn more calories on Monday, Wednesday and Sunday
* Users takes more steps on Monday, Tuesday and Wednesday still which is less than the count recommended by CDC
* Distance covered correlates with calories burned and steps taken
* User are getting less than 8 hours of recommended sleep 

Some interesting inights can be drawn from how often users use smart devices. To this end, we will first group users according to the number of days they used smart devices.
```{r}
# Calculating daily usage level
daily_use <- daily_activity_sleep %>% 
  drop_na() %>% 
  group_by(id) %>% 
  summarise(day_used= n()) %>% 
  mutate(usage_level = case_when(
    day_used >=1 & day_used < 10 ~ "Low usage",
    day_used > 10 & day_used <= 20 ~ "Medium usage",
    day_used >20 & day_used <=31 ~"High usage"))

# Calculating daily use percentage
daily_use_perc <- daily_use %>% 
  group_by(usage_level) %>% 
  summarize(level_totals = n()) %>% 
  mutate(total = sum(level_totals)) %>% 
  mutate(perc= level_totals/total) %>% 
  mutate(labels = scales::percent(perc))

daily_use_perc$usage_level<- ordered(daily_use_perc$usage_level, levels= c("High usage", "Medium usage", "Low usage"))

daily_use_perc
```

Plotting the data

```{r}
daily_use_perc %>% ggplot(aes(x="", y= level_totals,fill = usage_level))+ 
  geom_bar(stat = 'identity', color="white")+
  theme_void()+
  coord_polar("y", start = 0)+
  labs(x="",y="", title = "Daily Usage of Smartwatch",caption = "Data Collected by FitBit Fitness Tracker Data")+
  theme(plot.title = element_text(hjust = 0.5, size = 12), 
        axis.line = element_blank(), 
        axis.ticks = element_blank(), 
        axis.text = element_blank())+
        scale_fill_brewer(palette = 1, name = "Usage")+
  geom_text(aes(label = labels), position = position_stack(vjust = 0.5))
```

From the above visual;

* 50% of the users use their device quite frequently - between 21 to 31 days.
* 12% use their device - 11 to 20 days.
* 38% rarely use their device - 1 to 10 days

## Step 6: Act

After analyzing the Fitbit fitness tracker data, I came up with some recommendations that will help **Bellabeat to improve its marketing strategy.**

```{r, echo= FALSE, out.width= "100%"}
knitr::include_graphics("exercise.jpg", error = FALSE)
```

**Ideas for Bellabeat app:**

1. Average total steps per day are 7638 which a little bit less for having health benefits for according to the CDC research. They found that taking 8,000 steps per day was associated with a 51% lower risk for all-cause mortality (or death from all causes). Taking 12,000 steps per day was associated with a 65% lower risk compared with taking 4,000 steps. Bellabeat should encourage people to take at 10000 per day, through a reminder of remaining step count left to achieve the target.

2. In order to reduce weight and maintain a healthy life, one should reduce calorie intake. Bellabeat can introduce a low calorie diet plan for users and provide ideas for low calorie recipes.

3. Introducing programs and challenges for fitness and health incentives. The reward will come in the form of points, which can then be utilized to get deals on Bellabeat's goods and promotions.

4. Since users are getting an average sleep time less than 8 hours a day, i recommend a notification feature to remind users to go to bed and wake up.

5. Since users have more calories burned, steps taken and distance covered on Monday, Wednesday and Sunday. Bellabeat could use this knowledge to remind users to go for a walk or run on these days. Remaining days users are less active, this can be used to notify users to continue exercise on other days as well.

6. Since 50% of the users use the device frequently. To encourage more usage days, the Bellabeat product can be made to appear more fashionable and elegant to go with a variety of attires.


 
