---
layout: posts
title:  "Calculating Cohort Metrics"
date:   2022-02-26 13:00:00 +0100
categories: product analytics, data science,
author: Olayinka Ola
---

In this post I calculate period and cohort metrics for an online shop. I use [this dataset from Kaggle](https://www.kaggle.com/uttamp/store-data). It is an online tea retail store which sells tea of different flavours across various cities in India.

Each year represents a group of new customers defined as users who completed their first order in that year. My goal is to analyse their spend profile since acquisition through basic statistics.

Below is the code to create a distribution of each cohort.

```r
library(pacman) # Load pacman package
library(patchwork)
p_load(ggplot2, ggthemes, dplyr, readr)

# Retrieve data
data <- read.csv("storedata_total.csv")
data$firstorder <- as.POSIXct(data$firstorder, format="%d/%m/%Y")
data$firstorder <- as.Date(data$firstorder)
data$fst_order <- format(as.Date(data$firstorder, format = "%Y-%m-%d"), "%Y")
# Remove Nas
data <- data[complete.cases(data), ]
# Remove Row Based on Single Condition
data <- data[data$fst_order != 1904, ]

# Multiple conditions when adding new column to dataframe:
data <- data %>% mutate(cohort_number =
                     case_when(fst_order == 2008 ~ 1,
                               fst_order == 2009 ~ 2,
                               fst_order == 2010 ~ 3,
                               fst_order == 2011 ~ 4,
                               fst_order == 2012 ~ 5,
                               fst_order == 2013 ~ 6,
                               fst_order == 2014 ~ 7,
                               fst_order == 2015 ~ 8,
                               fst_order == 2016 ~ 9,
                               fst_order == 2017 ~ 10,
                               fst_order == 2018 ~ 11)
)

# Convert to factor
data$cohort_number <- as.factor(data$cohort_number)

# Keep Last 5 years
data_l5 <- data
# Remove Row Based on Single Condition
data_l5 <- data_l5[data_l5$fst_order != 2008, ]
data_l5 <- data_l5[data_l5$fst_order != 2009, ]
data_l5 <- data_l5[data_l5$fst_order != 2010, ]
data_l5 <- data_l5[data_l5$fst_order != 2011, ]
data_l5 <- data_l5[data_l5$fst_order != 2012, ]
data_l5 <- data_l5[data_l5$fst_order != 2013, ]

# Density plot by cohort
ggplot(data_l5, aes(x=avgorder, color=fst_order)) +
  # Density Plot
  geom_density() +
  # Grayscale
  scale_color_brewer(palette="Dark2") +
  theme_classic() +
  xlim(0, 350)
  # Titling
  labs(title = "Cohort Days in Product", x = "Product in Days", y = "Density",
    color = "Cohort", linetype = "Cohort") +
 # Title Formatting
  theme(plot.title = element_text(hjust = 0.5))
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Cohorts plotted together.png" alt="full model">

Five cohorts spend profiles plotted together

Next, is a simple calculation of retention. Below is the mean of all cohorts together over the last five years.

```r
# Average, median & standard deviation for order value over the past 5 years
mean(data_l5$avgorder)
median(data_l5$avgorder)
sd(data_l5$avgorder)
```

```
[1] 60.08528
[1] 49.61
```

To analyse each cohort I will iterate over each cohort to calculate their mean, median & standard deviation using tapply(). Using the matrix function I organise the information into a more readable format.

```r
# Average order value by cohort
means_5cohort <- tapply(data_l5$avgorder, data_l5$fst_order, mean)
medians_5cohort <- tapply(data_l5$avgorder, data_l5$fst_order, median)
mode_5cohort <- tapply(data_l5$avgorder, data_l5$fst_order, mode)
std_5cohort <- tapply(data_l5$avgorder, data_l5$fst_order, sd)

# Average months in product by cohort
mat_5cohort <- matrix(c(means_5cohort, medians_5cohort, std_5cohort), 5, 3)
rownames(mat_5cohort)<-c("2014_cohort", "2015_cohort", "2016_cohort", "2017_cohort", "2018_cohort")
colnames(mat_5cohort)<- c("mean", "median","Standard")
mat_5cohort
```

This post has been a quick look at simple cohort metric creation.

**Learn More**

- Rodrigues, J (2021). Product Analytics: Applied Data Science Techniques for Actionable Customer Insights.
