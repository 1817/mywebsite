---
categories:  
- ""    #the front matter should be like the one found in, e.g., blog2.md. It cannot be like the normal Rmd we used
- ""
date: "2023-06-14"
description: Machine Learning using Credit Card Fraud Data and R # the title that will show up once someone gets to this page
draft: false
image: fraud.png # save picture in \static\img\blogs. Acceptable formats= jpg, jpeg, or png . Your iPhone pics wont work

keywords: ""
slug: fraud_1 # slug is the shorthand URL address... no spaces plz
title: ML with Credit Card Fraud Data
---
![](../../../../../../../../../../../img/fraud.png)<!-- -->




# The problem: predicting credit card fraud

The goal of the project is to predict fraudulent credit card transactions.

We will be using a dataset with credit card transactions containing legitimate and fraud transactions. Fraud is typically well below 1% of all transactions, so a naive model that predicts that all transactions are legitimate and not fraudulent would have an accuracy of well over 99%-- pretty good, no? 

You can read more on credit card fraud on [Credit Card Fraud Detection Using Weighted Support Vector Machine](https://www.scirp.org/journal/paperinformation.aspx?paperid=105944)

The dataset we will use consists of credit card transactions and it includes information about each transaction including customer details, the merchant and category of purchase, and whether or not the transaction was a fraud.

## Obtain the data

The dataset is too large to be hosted on Canvas or Github, so please download it from dropbox https://www.dropbox.com/sh/q1yk8mmnbbrzavl/AAAxzRtIhag9Nc_hODafGV2ka?dl=0 and save it in your `dsb` repo, under the `data` folder.

As we will be building a classifier model using tidymodels, there's two things we need to do:

1. Define the outcome variable `is_fraud` as a factor, or categorical, variable, instead of the numerical 0-1 varaibles.
2. In tidymodels, the first level is the event of interest. If we leave our data as is, `0` is the first level, but we want to find out when we actually did (`1`) have a fraudulent transaction


```
## Rows: 671,028
## Columns: 14
## $ trans_date_trans_time <dttm> 2019-02-22 07:32:58, 2019-02-16 15:07:20, 2019-…
## $ trans_year            <dbl> 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2020, …
## $ category              <chr> "entertainment", "kids_pets", "personal_care", "…
## $ amt                   <dbl> 7.79, 3.89, 8.43, 40.00, 54.04, 95.61, 64.95, 3.…
## $ city                  <chr> "Veedersburg", "Holloway", "Arnold", "Apison", "…
## $ state                 <chr> "IN", "OH", "MO", "TN", "CO", "GA", "MN", "AL", …
## $ lat                   <dbl> 40.1186, 40.0113, 38.4305, 35.0149, 39.4584, 32.…
## $ long                  <dbl> -87.2602, -80.9701, -90.3870, -85.0164, -106.385…
## $ city_pop              <dbl> 4049, 128, 35439, 3730, 277, 1841, 136, 190178, …
## $ job                   <chr> "Development worker, community", "Child psychoth…
## $ dob                   <date> 1959-10-19, 1946-04-03, 1985-03-31, 1991-01-28,…
## $ merch_lat             <dbl> 39.41679, 39.74585, 37.73078, 34.53277, 39.95244…
## $ merch_long            <dbl> -87.52619, -81.52477, -91.36875, -84.10676, -106…
## $ is_fraud              <fct> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
```

The data dictionary is as follows

| column(variable)      | description                                 |
|-----------------------|---------------------------------------------|
| trans_date_trans_time | Transaction DateTime                        |
| trans_year            | Transaction year                            |
| category              | category of merchant                        |
| amt                   | amount of transaction                       |
| city                  | City of card holder                         |
| state                 | State of card holder                        |
| lat                   | Latitude location of purchase               |
| long                  | Longitude location of purchase              |
| city_pop              | card holder's city population               |
| job                   | job of card holder                          |
| dob                   | date of birth of card holder                |
| merch_lat             | Latitude Location of Merchant               |
| merch_long            | Longitude Location of Merchant              |
| is_fraud              | Whether Transaction is Fraud (1) or Not (0) |

We also add some of the variables we considered in our EDA for this dataset during homework 2.


```r
card_fraud <- card_fraud %>% 
  mutate( hour = hour(trans_date_trans_time),
          wday = wday(trans_date_trans_time, label = TRUE),
          month_name = month(trans_date_trans_time, label = TRUE),
          age = interval(dob, trans_date_trans_time) / years(1)
) %>% 
  rename(year = trans_year) %>% 
  
  mutate(age_group = ceiling(age/5)*5) %>% #put age into different age groups
  
  mutate(
    
    # convert latitude/longitude to radians
    lat1_radians = lat / 57.29577951,
    lat2_radians = merch_lat / 57.29577951,
    long1_radians = long / 57.29577951,
    long2_radians = merch_long / 57.29577951,
    
    # calculate distance in miles
    distance_miles = 3963.0 * acos((sin(lat1_radians) * sin(lat2_radians)) + cos(lat1_radians) * cos(lat2_radians) * cos(long2_radians - long1_radians)),

    # calculate distance in km
    distance_km = 6377.830272 * acos((sin(lat1_radians) * sin(lat2_radians)) + cos(lat1_radians) * cos(lat2_radians) * cos(long2_radians - long1_radians))

  )

card_fraud$age <- floor(card_fraud$age)
```



## Exploratory Data Analysis (EDA) (REN)

You have done some EDA and you can pool together your group's expertise in which variables to use as features.
<<<<<<< HEAD
You can reuse your EDA from earlier, but we expect at least a few visualizations and/or tables to explore the dataset and identify any useful features.
=======
You can reuse your EDA from earlier, but we expect at least a few visualisations and/or tables to explore the dataset and identify any useful features.
>>>>>>> b29b661fec234f9d3d84d74ee3b104f94f85cc5a

Group all variables by type and examine each variable class by class. The dataset has the following types of variables:

1.  Strings
2.  Geospatial Data
3.  Dates
4.  Date/Times
5.  Numerical

Strings are usually not a useful format for classification problems. The strings should be converted to factors, dropped, or otherwise transformed.


```r
#Table that summarizes the number and frequency of fraudulent transactions
fraud_cases_by_year <- card_fraud %>% 
  
  # Filter only fraud cases
  filter(is_fraud == "1") %>% 
  
  # Group by year and summarise fraud case in each year
  group_by(year) %>% 
  summarise(count_fraud_cases = n())

fraud_cases_by_year
```

```
## # A tibble: 2 × 2
##    year count_fraud_cases
##   <dbl>             <int>
## 1  2019              2721
## 2  2020              1215
```

```r
#Costs of fraud transaction

# Summarise data
summary <- card_fraud %>%
  group_by(year, is_fraud) %>%
  summarise(total_amt = sum(amt, na.rm = TRUE),
            .groups = "drop")

# Calculate the yearly total
yearly_totals <- summary %>%
  group_by(year) %>%
  summarise(yearly_total_amt = sum(total_amt),
            .groups = "drop")

# Join yearly total back to summary
summary <- summary %>%
  left_join(yearly_totals, by = "year")

# Calculate the percentage of fraudulent transactions
summary <- summary %>%
  mutate(fraud_percentage = ifelse(is_fraud == 1, (total_amt / yearly_total_amt) * 100, 0))

# Print summary
print(summary)
```

```
## # A tibble: 4 × 5
##    year is_fraud total_amt yearly_total_amt fraud_percentage
##   <dbl> <fct>        <dbl>            <dbl>            <dbl>
## 1  2019 1         1423140.        33606041.             4.23
## 2  2019 0        32182901.        33606041.             0   
## 3  2020 1          651949.        13577863.             4.80
## 4  2020 0        12925914.        13577863.             0
```


```r
#histogram that shows the distribution of amounts charged to credit card, both for legitimate and fraudulent accounts.

card_fraud %>% 
  
  # Group by fraud cases and amount and summarise total amount of each case
  group_by(is_fraud,amt) %>% 
  summarise(count = n()) %>% 
  
  # Plot histogram and splited facet by whether is fraud or not
  ggplot(aes(x=amt)) +
  geom_histogram() +
  labs(x= "Amount charged", y= "# of amount charged") +
  ggtitle("Distribution of amounts charged to credit card") +
  facet_wrap((~is_fraud)) 
```

```
## `summarise()` has grouped output by 'is_fraud'. You can override using the
## `.groups` argument.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-6-1.png" width="672" />

```r
# Filter out fraudulent transactions
fraudulent <- card_fraud %>%
  filter(is_fraud == 1)

legitimate <- card_fraud %>% 
  filter(is_fraud == 0)
# Calculate summary statistics
fraudulent_summary <- fraudulent %>%
  summarise(Mean = mean(amt, na.rm = TRUE),
            Median = median(amt, na.rm = TRUE),
            Min = min(amt, na.rm = TRUE),
            Max = max(amt, na.rm = TRUE))

legitimate_summary <- legitimate %>%
  summarise(Mean = mean(amt, na.rm = TRUE),
            Median = median(amt, na.rm = TRUE),
            Min = min(amt, na.rm = TRUE),
            Max = max(amt, na.rm = TRUE))

# Print summary statistics
print("Fraudulent Transaction Summary:")
```

```
## [1] "Fraudulent Transaction Summary:"
```

```r
print(fraudulent_summary)
```

```
## # A tibble: 1 × 4
##    Mean Median   Min   Max
##   <dbl>  <dbl> <dbl> <dbl>
## 1  527.   369.  1.06 1334.
```

```r
print("Legitimate Transaction Summary:")
```

```
## [1] "Legitimate Transaction Summary:"
```

```r
print(legitimate_summary)
```

```
## # A tibble: 1 × 4
##    Mean Median   Min    Max
##   <dbl>  <dbl> <dbl>  <dbl>
## 1  67.6   47.2     1 27120.
```

```r
#types of purchases that's likely to be fraudulent

fraudulent <- card_fraud %>%
  filter(is_fraud == 1)

# Calculate % of total fraudulent transactions by category
fraud_by_category <- fraudulent %>%
  group_by(category) %>%
  summarise(total_amt = sum(amt), .groups = "drop") %>%
  mutate(percent_of_total = (total_amt / sum(total_amt)) * 100) %>%
  arrange(desc(percent_of_total))

# Plot a bar chart
ggplot(fraud_by_category, aes(x = reorder(category, percent_of_total), y = percent_of_total)) +
  geom_bar(stat = "identity", fill = "steelblue") +
  coord_flip() +
  theme_minimal() +
  labs(title = "Fraudulent Transactions by Category", x = "Category", y = "% of Total Fraudulent Transactions")
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-7-1.png" width="672" />

```r
#time of day that most fraud happens
fraud_hour <- card_fraud %>% 
  filter(is_fraud == 1) %>% 
  group_by(hour) %>% 
  summarise(fraud_count = n(), .groups = "drop")

#plot
ggplot(fraud_hour, aes(x = hour, y = fraud_count)) +
  geom_bar(stat = "identity", fill = "steelblue") +
  labs(x = "Hour of the Day", y = "Number of Fraudulent Transactions", 
       title = "Number of Fraudulent Transactions by Hour of the Day") +
  theme_minimal()
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-8-1.png" width="672" />

```r
#age and fraud

fraud_age <- card_fraud %>% 
  filter(is_fraud == 1) %>% 
  group_by(age) %>% 
  summarise(fraud_count = n(), .groups = "drop")

#plot
ggplot(fraud_age, aes(x = age, y = fraud_count)) +
  geom_bar(stat = "identity", fill = "steelblue") +
  labs(x = "Age", y = "Number of Fraudulent Transactions", 
       title = "Number of Fraudulent Transactions by age of victim") +
  theme_minimal()
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-9-1.png" width="672" />

```r
#city pop against age

age_pop <- card_fraud %>% 
  filter(is_fraud == 1) %>% 
  group_by(age, city_pop) %>% 
  summarise(fraud_count = n(), .groups = "drop")

#plotting
ggplot(age_pop, aes(x = age, y = city_pop, size = fraud_count)) +
  geom_point(alpha = 0.6) +
  scale_size_continuous(range = c(1, 2)) +
  labs(x = "Age", y = "City Population", size = "Number of Fraudulent Transactions", 
       title = "Fraudulent transactions by Age and City Population") +
  theme_minimal()
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-9-2.png" width="672" />


***Strings to Factors*** 

-   `category`, Category of Merchant
-   `job`, Job of Credit Card Holder



***Strings to Geospatial Data*** 

We have plenty of geospatial data as lat/long pairs, so I want to convert city/state to lat/long so I can compare to the other geospatial variables. This will also make it easier to compute new variables like the distance the transaction is from the home location. 

-   `city`, City of Credit Card Holder
-   `state`, State of Credit Card Holder


```r
#Geospatial data
#intuition: filter out all frauds
#turn city locations into lat/long pairs
#map locations of cities with fraud victims to map
library(tidygeocoder)
library(sf)
```

```
## Linking to GEOS 3.11.0, GDAL 3.5.3, PROJ 9.1.0; sf_use_s2() is TRUE
```

```r
library(opencage) # for geocoding addresses
library(usethis)
library(hrbrthemes) # hrbrmstr/hrbrthemes
```

```
## NOTE: Either Arial Narrow or Roboto Condensed fonts are required to use these themes.
```

```
##       Please use hrbrthemes::import_roboto_condensed() to install Roboto Condensed and
```

```
##       if Arial Narrow is not on your system, please see https://bit.ly/arialnarrow
```

```r
library(kableExtra)
```

```
## 
## Attaching package: 'kableExtra'
```

```
## The following object is masked from 'package:dplyr':
## 
##     group_rows
```

```r
library(rnaturalearth)
```

```
## The legacy packages maptools, rgdal, and rgeos, underpinning this package
## will retire shortly. Please refer to R-spatial evolution reports on
## https://r-spatial.org/r/2023/05/15/evolution4.html for details.
## This package is now running under evolution status 0
```

```
## Support for Spatial objects (`sp`) will be deprecated in {rnaturalearth} and will be removed in a future release of the package. Please use `sf` objects with {rnaturalearth}. For example: `ne_download(returnclass = 'sf')`
```

```r
library(tmap)

map_fraud_latlong <- card_fraud %>% 
  filter(is_fraud == 1) %>% 
  mutate(state_geo = geo(state, method = "osm"))
```

```
## Passing 51 addresses to the Nominatim single address geocoder
```

```
## Query completed in: 51.5 seconds
```

```r
map_fraud_plot <- map_fraud_latlong %>% 
  count(state, sort = TRUE) %>% 
  mutate(address_geo = purrr::map(state, geo, method = "osm")) %>% 
  unnest_wider(address_geo)
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```
## Passing 1 address to the Nominatim single address geocoder
```

```
## Query completed in: 1 seconds
```

```r
usa <- ne_countries(scale = "medium", returnclass = "sf") %>% 
  filter(name != "Antarctica") %>% 
  st_geometry(usa)

ggplot(data = usa) +
  geom_sf(fill = "#fafafa") + # the first two lines just plot the world shapefile
  geom_point(data = map_fraud_plot, # then we add points
             aes(x = long, y = lat),
             size = 2,
             colour = "#001e62") +
  theme_void()
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-10-1.png" width="672" />


##  Exploring factors: how is the compactness of categories?

-   Do we have excessive number of categories? Do we want to combine some?


```r
card_fraud %>% 
  count(category, sort=TRUE)%>% 
  mutate(perc = n/sum(n))
```

```
## # A tibble: 14 × 3
##    category           n   perc
##    <chr>          <int>  <dbl>
##  1 gas_transport  68046 0.101 
##  2 grocery_pos    63791 0.0951
##  3 home           63597 0.0948
##  4 shopping_pos   60416 0.0900
##  5 kids_pets      58772 0.0876
##  6 shopping_net   50743 0.0756
##  7 entertainment  48521 0.0723
##  8 food_dining    47527 0.0708
##  9 personal_care  46843 0.0698
## 10 health_fitness 44341 0.0661
## 11 misc_pos       41244 0.0615
## 12 misc_net       32829 0.0489
## 13 grocery_net    23485 0.0350
## 14 travel         20873 0.0311
```

```r
card_fraud %>% 
  count(job, sort=TRUE) %>% 
  mutate(perc = n/sum(n))
```

```
## # A tibble: 494 × 3
##    job                            n    perc
##    <chr>                      <int>   <dbl>
##  1 Film/video editor           5106 0.00761
##  2 Exhibition designer         4728 0.00705
##  3 Naval architect             4546 0.00677
##  4 Surveyor, land/geomatics    4448 0.00663
##  5 Materials engineer          4292 0.00640
##  6 Designer, ceramics/pottery  4262 0.00635
##  7 IT trainer                  4014 0.00598
##  8 Financial adviser           3959 0.00590
##  9 Systems developer           3948 0.00588
## 10 Environmental consultant    3831 0.00571
## # ℹ 484 more rows
```


The predictors `category` and `job` are transformed into factors.


```r
card_fraud <- card_fraud %>% 
  mutate(category = factor(category),
         job = factor(job)) 
```

`category` has 14 unique values, and `job` has 494 unique values. The dataset is quite large, with over 670K records, so these variables don't have an excessive number of levels at first glance. However, it is worth seeing if we can compact the levels to a smaller number.

### Why do we care about the number of categories and whether they are "excessive"?

Consider the extreme case where a dataset had categories that only contained one record each. There is simply insufficient data to make correct predictions using category as a predictor on new data with that category label. Additionally, if your modeling uses dummy variables, having an extremely large number of categories will lead to the production of a huge number of predictors, which can slow down the fitting. This is fine if all the predictors are useful, but if they aren't useful (as in the case of having only one record for a category), trimming them will improve the speed and quality of the data fitting.

If I had subject matter expertise, I could manually combine categories. If you don't have subject matter expertise, or if performing this task would be too labor intensive, then you can use cutoffs based on the amount of data in a category. If the majority of the data exists in only a few categories, then it might be reasonable to keep those categories and lump everything else in an "other" category or perhaps even drop the data points in smaller categories. 


## Do all variables have sensible types?

Consider each variable and decide whether to keep, transform, or drop it. This is a mixture of Exploratory Data Analysis and Feature Engineering, but it's helpful to do some simple feature engineering as you explore the data. In this project, we have all data to begin with, so any transformations will be performed on the entire dataset. Ideally, do the transformations as a `recipe_step()` in the tidymodels framework. Then the transformations would be applied to any data the recipe was used on as part of the modeling workflow. There is less chance of data leakage or missing a step when you perform the feature engineering in the recipe.

## Which variables to keep in your model?

You have a number of variables and you have to decide which ones to use in your model. For instance, you have the latitude/lognitude of the customer, that of the merchant, the same data in radians, as well as the `distance_km` and `distance_miles`. Do you need them all? 


```r
 card_fraud_features <- card_fraud %>% 
  select(category, amt, is_fraud, hour, age_group, distance_km)
```

we started with 6 variables: category, amount, is_fraud, hour, age, and distance. 

Afterwards we added "city_pop" and "state" seaparately to see if it will improve the model. None of those did.

Then we manipulated "age" into different "age_groups". This improved the xgb model's accuracy. Thus we replaced age with "age_groups

We also tried grouping "hour" into different categories (morning, afternoon, night) too. This did not improve accuracy.

## Fit your workflows in smaller sample

You will be running a series of different models, along the lines of the California housing example we have seen in class. However, this dataset has 670K rows and if you try various models and run cross validation on them, your computer may slow down or crash.

Thus, we will work with a smaller sample of 10% of the values the original dataset to identify the best model, and once we have the best model we can use the full dataset to train- test our best model.



```r
# select a smaller subset
my_card_fraud <- card_fraud_features %>% 
  # select a smaller subset, 10% of the entire dataframe 
  slice_sample(prop = 0.10) 
```


## Split the data in training - testing


```r
# **Split the data**

set.seed(123)

data_split <- initial_split(my_card_fraud, # updated data
                           prop = 0.8, 
                           strata = is_fraud)

card_fraud_train <- training(data_split) 
card_fraud_test <- testing(data_split)
```


## Cross Validation

Start with 3 CV folds to quickly get an estimate for the best model and you can increase the number of folds to 5 or 10 later.


```r
set.seed(123)
cv_folds <- vfold_cv(data = card_fraud_train, 
                          v = 3, 
                          strata = is_fraud)
cv_folds 
```

```
## #  3-fold cross-validation using stratification 
## # A tibble: 3 × 2
##   splits                id   
##   <list>                <chr>
## 1 <split [35787/17894]> Fold1
## 2 <split [35787/17894]> Fold2
## 3 <split [35788/17893]> Fold3
```


## Define a tidymodels `recipe`

What steps are you going to add to your recipe? Do you need to do any log transformations?


```r
fraud_rec <- recipe(is_fraud ~ ., data = card_fraud_train) %>%
  step_log(amt) %>%
  step_novel(all_nominal(), -all_outcomes()) %>%
  step_dummy(all_nominal(), -all_outcomes()) %>% 
  step_zv(all_numeric(), -all_outcomes())  %>% 
  step_normalize(all_numeric()) 
```

Once you have your recipe, you can check the pre-processed dataframe 


```r
prepped_data <- 
  fraud_rec %>% # use the recipe object
  prep() %>% # perform the recipe on training data
  juice() # extract only the preprocessed dataframe 

glimpse(prepped_data)
```

```
## Rows: 53,681
## Columns: 18
## $ amt                     <dbl> 0.3817751, -0.1678079, 0.5180406, -0.5699504, …
## $ hour                    <dbl> -1.43901946, 0.32468057, -0.55716945, 0.471655…
## $ age_group               <dbl> -0.19156366, -0.76635635, -1.34114904, 0.67062…
## $ distance_km             <dbl> -0.2115124, -0.1644061, 0.5032902, 1.1915351, …
## $ is_fraud                <fct> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0…
## $ category_food_dining    <dbl> -0.2742835, -0.2742835, -0.2742835, -0.2742835…
## $ category_gas_transport  <dbl> -0.3339818, -0.3339818, -0.3339818, -0.3339818…
## $ category_grocery_net    <dbl> -0.1923869, -0.1923869, -0.1923869, -0.1923869…
## $ category_grocery_pos    <dbl> 3.0536996, -0.3274655, 3.0536996, -0.3274655, …
## $ category_health_fitness <dbl> -0.2661877, 3.7566782, -0.2661877, -0.2661877,…
## $ category_home           <dbl> -0.3232293, -0.3232293, -0.3232293, 3.0937210,…
## $ category_kids_pets      <dbl> -0.3085708, -0.3085708, -0.3085708, -0.3085708…
## $ category_misc_net       <dbl> -0.2279682, -0.2279682, -0.2279682, -0.2279682…
## $ category_misc_pos       <dbl> -0.255557, -0.255557, -0.255557, -0.255557, -0…
## $ category_personal_care  <dbl> -0.2694632, -0.2694632, -0.2694632, -0.2694632…
## $ category_shopping_net   <dbl> -0.2875232, -0.2875232, -0.2875232, -0.2875232…
## $ category_shopping_pos   <dbl> -0.3155083, -0.3155083, -0.3155083, -0.3155083…
## $ category_travel         <dbl> -0.1801262, -0.1801262, -0.1801262, -0.1801262…
```


## Define various models

You should define the following classification models:

1. Logistic regression, using the `glm` engine
2. Decision tree, using the `C5.0` engine
3. Random Forest, using  the `ranger` engine and setting `importance = "impurity"`)  
4. A boosted tree using Extreme Gradient Boosting, and the `xgboost` engine
5. A k-nearest neighbours,  using 4 nearest_neighbors and the `kknn` engine  


```r
## Model Building 

# 1. Pick a `model type`
# 2. set the `engine`
# 3. Set the `mode`:  classification

# Logistic regression
log_spec <-  logistic_reg() %>%  # model type
  set_engine(engine = "glm") %>%  # model engine
  set_mode("classification") # model mode

# Show your model specification
log_spec
```

```
## Logistic Regression Model Specification (classification)
## 
## Computational engine: glm
```

```r
# Decision Tree
tree_spec <- decision_tree() %>%
  set_engine(engine = "C5.0") %>%
  set_mode("classification")

tree_spec
```

```
## Decision Tree Model Specification (classification)
## 
## Computational engine: C5.0
```

```r
# Random Forest
library(ranger)

rf_spec <- 
  rand_forest() %>% 
  set_engine("ranger", importance = "impurity") %>% 
  set_mode("classification")


# Boosted tree (XGBoost)
library(xgboost)
```

```
## 
## Attaching package: 'xgboost'
```

```
## The following object is masked from 'package:dplyr':
## 
##     slice
```

```r
xgb_spec <- 
  boost_tree() %>% 
  set_engine("xgboost") %>% 
  set_mode("classification") 

# K-nearest neighbour (k-NN)
knn_spec <- 
  nearest_neighbor(neighbors = 4) %>% # we can adjust the number of neighbors 
  set_engine("kknn") %>% 
  set_mode("classification") 
```

## Bundle recipe and model with `workflows`


```r
## Bundle recipe and model with `workflows`


log_wflow <- # new workflow object
 workflow() %>% # use workflow function
 add_recipe(fraud_rec) %>%   # use the new recipe
 add_model(log_spec)   # add your model spec

tree_wflow <-
 workflow() %>%
 add_recipe(fraud_rec) %>% 
 add_model(tree_spec) 

rf_wflow <-
 workflow() %>%
 add_recipe(fraud_rec) %>% 
 add_model(rf_spec) 

xgb_wflow <-
 workflow() %>%
 add_recipe(fraud_rec) %>% 
 add_model(xgb_spec)

knn_wflow <-
 workflow() %>%
 add_recipe(fraud_rec) %>% 
 add_model(knn_spec)
```


## Fit models

You may want to compare the time it takes to fit each model. `tic()` starts a simple timer and `toc()` stops it


```r
tic()
log_res <- log_wflow %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, accuracy,
      kap, roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 
time <- toc()
```

```
## 1.299 sec elapsed
```

```r
log_time <- time[[4]]

tic()
tree_res <- tree_wflow %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, accuracy,
      kap, roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 
time <- toc()
```

```
## 5.655 sec elapsed
```

```r
tree_time <- time[[4]]

tic()
rf_res <- rf_wflow %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, accuracy,
      kap, roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 
time <- toc()
```

```
## 14.804 sec elapsed
```

```r
rf_time <- time[[4]]

tic()
xgb_res <- xgb_wflow %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, accuracy,
      kap, roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 
time <- toc()
```

```
## 1.528 sec elapsed
```

```r
xgb_time <- time[[4]]

tic()
knn_res <- knn_wflow %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, accuracy,
      kap, roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 
time <- toc()
```

```
## 94.477 sec elapsed
```

```r
knn_time <- time[[4]]
```

## Compare models


```r
## Model Comparison

log_metrics <- 
  log_res %>% 
  collect_metrics(summarise = TRUE) %>%
  # add the name of the model to every row
  mutate(model = "Logistic Regression",
         time = log_time)

# add mode models here
tree_metrics <- 
  tree_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Decision Tree",
         time = tree_time)

rf_metrics <- 
  rf_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Random Forest",
         time = rf_time)

xgb_metrics <- 
  xgb_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "XGBoost",
         time = xgb_time)

knn_metrics <- 
  knn_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Knn",
         time = knn_time)

# create dataframe with all models
model_compare <- bind_rows(log_metrics,
                            tree_metrics,
                            rf_metrics,
                           xgb_metrics,
                           knn_metrics
                      ) %>% 
  # get rid of 'sec elapsed' and turn it into a number
  mutate(time = str_sub(time, end = -13) %>% 
           as.double()
         )
```

## Which metric to use

This is a highly imbalanced data set, as roughly 99.5% of all transactions are ok, and it's only 0.5% of transactions that are fraudulent. A `naive` model, which classifies everything as ok and not-fraud, would have an accuracy of 99.5%, but what about the sensitivity, specificity, the AUC, etc?

## `last_fit()`

```r
## `last_fit()` on test set

# - `last_fit()`  fits a model to the whole training data and evaluates it on the test set. 
# - provide the workflow object of the best model as well as the data split object (not the training data). 
 
# Split the data into training and test sets
card_fraud_split <- initial_split(card_fraud_features, prop = 0.8)

# Fit the model on the training data and evaluate on the test set
last_fit_xgb <- last_fit(xgb_wflow, 
                        split = card_fraud_split,
                        metrics = metric_set(
                          accuracy, f_meas, kap, precision,
                          recall, roc_auc, sens, spec))
```



## Get variable importance using `vip` package



```r
last_fit_xgb %>% 
  pluck(".workflow", 1) %>%   
  pull_workflow_fit() %>% 
  vip(num_features = 10) +
  theme_light()
```

```
## Warning: `pull_workflow_fit()` was deprecated in workflows 0.2.3.
## ℹ Please use `extract_fit_parsnip()` instead.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
## generated.
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-19-1.png" width="672" />

## Plot Final Confusion matrix and ROC curve



```r
## Final Confusion Matrix

last_fit_xgb %>%
  collect_predictions() %>% 
  conf_mat(is_fraud, .pred_class) %>% 
  autoplot(type = "heatmap")
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-20-1.png" width="672" />

```r
## Final ROC curve
last_fit_xgb %>% 
  collect_predictions() %>% 
  roc_curve(is_fraud, .pred_1) %>% 
  autoplot()
```

<img src="/projects/fraud_1_files/figure-html/unnamed-chunk-20-2.png" width="672" />


##  Calculating the cost of fraud to the company


- How much money (in US\$ terms) are fraudulent transactions costing the company? Generate a table that summarizes the total amount of legitimate and fraudulent transactions per year and calculate the % of fraudulent transactions, in US\$ terms. Compare your model vs the naive classification that we do not have any fraudulent transactions. 


```r
# Define the best model

best_model_wflow <- xgb_wflow


best_model_preds <- 
  best_model_wflow %>% 
  fit(data = card_fraud_train) %>%  
  
  ## Use `augment()` to get predictions for entire data set
  augment(new_data = card_fraud)

best_model_preds %>% 
  conf_mat(truth = is_fraud, estimate = .pred_class)
```

```
##           Truth
## Prediction      1      0
##          1   2679    183
##          0   1257 666909
```

```r
cost <- best_model_preds %>%
  select(is_fraud, amt, pred = .pred_class) 

cost <- cost %>%
  mutate(
  

  # naive false-- we think every single transaction is ok and not fraud

    false_naive_cost = if_else(is_fraud == 1, amt, 0),

  # false negatives-- we thought they were not fraud, but they were

   false_negative_cost = if_else(is_fraud == 1 & pred == 0, amt, 0),
  
  # false positives-- we thought they were fraud, but they were not

   false_positive_cost = if_else(is_fraud ==0 & pred == 1, 0.02*amt,0),
    
  # true positives-- we thought they were fraud, and they were 

   true_positive_cost = 0,
  
  # true negatives-- we thought they were ok, and they were 
  
   true_negative_cost = 0
  
)
  
# Summarising

cost_summary <- cost %>% 
  summarise(across(starts_with(c("false","true", "amt")), 
            ~ sum(.x, na.rm = TRUE))) %>%
  mutate(perc_cost_naive = false_naive_cost/amt,
         perc_cost_classifier = (false_positive_cost + false_negative_cost + true_positive_cost + true_negative_cost)/amt)

cost_summary
```

```
## # A tibble: 1 × 8
##   false_naive_cost false_negative_cost false_positive_cost true_positive_cost
##              <dbl>               <dbl>               <dbl>              <dbl>
## 1         2075089.             429923.               3080.                  0
## # ℹ 4 more variables: true_negative_cost <dbl>, amt <dbl>,
## #   perc_cost_naive <dbl>, perc_cost_classifier <dbl>
```


- If we use a naive classifier thinking that all transactions are legitimate and not fraudulent, the cost to the company is $2,075,089.
- With our best model, the total cost of false negatives, namely transactions our classifier thinks are legitimate but which turned out to be fraud, is $429,923.

- Our classifier also has some false positives, $3,079.96, namely flagging transactions as fraudulent, but which were legitimate. Assuming the card company makes around 2% for each transaction (source: https://startups.co.uk/payment-processing/credit-card-processing-fees/), the amount of money lost due to these false positives is $61.60

- The \$ improvement over the naive policy is $1,645,105.
