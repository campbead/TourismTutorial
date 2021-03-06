# New Zealand Tourism Data
This is a quick tutorial to download and begin to analyse New Zealand tourism data using [R](https://www.r-project.org/). There is a lots of data availabe to analyze but in this example we are going to look at the occupancy rate of accomidations in Dunedin. 
## A note about this project and open data
This project came about from recognising a need for usage examples of open data available in New Zealand.  [data.govt.nz](https://data.govt.nz/) provides resources for accessing many open data sources.  As part of the [Dunedin Open Data Meetup](https://www.meetup.com/Dunedin-Open-Data-Meetup/), we decided to create quick usage guides to open data sources in New Zealand.  You can access the guide and contribute on the [NZdataExamples github repository](https://github.com/data-govt-nz/NZdataExamples) /salespitch

## Downloading the data
We'll first begin by downloading the data.  Stats NZ Infoshare is a great tool to begin with.  [InfoShare](http://archive.stats.govt.nz/infoshare/Default.aspx). From there we go to Tourism->Accommodation Survey->ACS->Territorial authority by variable (Monthly).  In the next page you can select what individual data you'd like.  In this example, I choose data for Dunedin City, in Territorial Authority and select all data for each of the other boxes.  Then I export as .csv file.

## Data Analysis
In this quick example I will calculate the occupancy rate for accomidations in Dunedin by month and then display this data as bar charts.  

First let's load dplyr, an incredibly useful tool for data manipulation

```r
library(dplyr)
```

The next step we load the the datafile and do a bit of data manipuation to adjust the names of the columns and to discard the some rows we don't need.

```r
datafile = 'ACS475801_20190302_015859_74.csv'
tourism <- read.csv(datafile, skip = 3, header = T)
tourism <- head(tourism,-21)
colnames(tourism)<- c("Date" , "Number_of_establishments","Capacity_daily", "Capacity_monthly", "Occupancy_monthly", "Guest_Arrivals", "Guest_Nights")
```

We can calculate the occupancy rate by the divividing the monthly occupancy by the monthy capacity.

```r
tourism$Occupancy_rate <- tourism$Occupancy_monthly / tourism$Capacity_monthly
```

Next we do a bit of manipulation to make the dates more accessible.

```r
tourism$Date_formated <- as.Date(paste(tourism$Date, "15"),format='%YM%m %d')
tourism$Date_month <- format(tourism$Date_formated, "%m")
tourism$Date_year <- format(tourism$Date_formated, "%Y")
```

Let's plot occupancy rate by month in a bar chart.

```r
monthly_tourism <- tourism %>%
  group_by(Date_month) %>%
  summarize(mean_occ = mean(Occupancy_rate)*100, std_occ = sd(Occupancy_rate)*100)

barCenters <- barplot(height = monthly_tourism$mean_occ,
                      main = "Average occupancy rate in Dunedin (2001 - 2018)", xlab = "Month", ylab = "Occupancy Rate %", ylim=c(0,100), names.arg = monthly_tourism$Date_month)
arrows(barCenters, monthly_tourism$mean_occ-monthly_tourism$std_occ,
       barCenters, monthly_tourism$mean_occ+monthly_tourism$std_occ,angle=90,code=3)
```

![](NZ_tourism_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

Now lets try that for the most recent 5 years. Here all I have changed in the filter within the dpylr block

```r
monthly_tourism_2014 <- tourism %>%
  filter(Date_year > 2013) %>% # select dates after 2013
  group_by(Date_month) %>%
  summarize(mean_occ = mean(Occupancy_rate)*100, std_occ = sd(Occupancy_rate)*100)

barCenters <- barplot(height = monthly_tourism_2014$mean_occ,
                      main = "Average occupancy rate in Dunedin (2014 - 2018)", xlab = "Month", ylab = "Occupancy Rate %", ylim=c(0,100), names.arg = monthly_tourism_2014$Date_month)
arrows(barCenters, monthly_tourism_2014$mean_occ-monthly_tourism_2014$std_occ,
       barCenters, monthly_tourism_2014$mean_occ+monthly_tourism_2014$std_occ,angle=90,code=3)
```

![](NZ_tourism_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

## Conclusion
Thanks for looking over this quick project.  The Rmd file is available here in this github repository [NZ_tourism.Rmd](https://github.com/campbead/TourismTutorial/blob/master/NZ_tourism.Rmd).

If you have anything to add please feel free to fork this repository and make pull request.
