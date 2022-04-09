---
layout: posts
title:  "Exploring the individual household electric power consumption Data Set"
date:   2021-12-09 13:00:00 +0100
categories: r
author: Olayinka Ola
---
The goal of this project is to explore the Individual household electric power consumption Data Set from this [repository](http://archive.ics.uci.edu/ml/).

The dataset has 2,075,259 rows and 9 columns. As I only need dates from 2007-02-01 and 2007-02-02 I subset the data to those dates. Once the date and time data were formatted properly for the subset created I made a series of plots using the base plotting system.

The following descriptions of the 9 variables in the dataset are taken from the [UCI web site](https://archive.ics.uci.edu/ml/datasets/Individual+household+electric+power+consumption):

1. **Date**: Date in format dd/mm/yyyy
2. **Time**: time in format hh:mm:ss
3. **Global_active_power**: household global minute-averaged active power (in kilowatt)
4. **Global_reactive_power**: household global minute-averaged reactive power (in kilowatt)
5. **Voltage**: minute-averaged voltage (in volt)
6. **Global_intensity**: household global minute-averaged current intensity (in ampere)
7. **Sub_metering_1**: energy sub-metering No. 1 (in watt-hour of active energy). It corresponds to the kitchen, containing mainly a dishwasher, an oven and a microwave (hot plates are not electric but gas powered).
8. **Sub_metering_2**: energy sub-metering No. 2 (in watt-hour of active energy). It corresponds to the laundry room, containing a washing-machine, a tumble-drier, a refrigerator and a light.
9. **Sub_metering_3**: energy sub-metering No. 3 (in watt-hour of active energy). It corresponds to an electric water-heater and an air-conditioner.

[Link to the Github files](https://github.com/hightowerr/datasciencecoursera/tree/master/ExploratoryDataAnalysis/Week1Proj1)

## **Loading the data**

```r
dataset <- read.table('./Data/household_power_consumption.txt',
                      header = TRUE,
                      na="?",
                      sep = ";",
                      colClasses = c("character",
                                     "character",
                                     "numeric",
                                     "numeric",
                                     "numeric",
                                     "numeric",
                                     "numeric",
                                     "numeric",
                                     "numeric"))

## Format date to Type Date
dataset$Date <- as.Date(dataset$Date, format="%d/%m/%Y")
## Filter data set from Feb. 1, 2007 to Feb. 2, 2007
df <- dataset[dataset$Date >= "2007-02-01" & dataset$Date < "2007-02-03", ]
## Format time
df$Time <- as.POSIXct(df$Time, format="%H:%M:%S")
## Extract Time from a Column in a Dataframe
df$Time <- format(df$Time, format = "%H:%M:%S")
## Combine Date and Time column
df$datetime <- paste(df$Date,df$Time)
## Convert DateTime column
df$datetime <- ymd_hms(df$datetime)
```

### Plot 1

```r
# Make the figure
hist(df$Global_active_power,
     main="Global Active Power",
     xlab = "Global Active Power (kilowatts)",
     col="red")
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Plot 1.png" alt="Histogram">

### Plot 2

```r
# Make the figure
plot(data.frame(df$datetime, df$Global_active_power), type = "l",
     xlab = "Datetime", ylab = "Global Active Power (kilowatts)")
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Plot 2.png" alt="linegraph">

### Plot 3

```r
# Make the figure
plot(data.frame(df$datetime, df$Sub_metering_1), type = "l", xlab = "Datetime", ylab = "Energy sub metering")
lines(data.frame(df$datetime, df$Sub_metering_2), type = "l", col = "red")
lines(data.frame(df$datetime, df$Sub_metering_3), type = "l", col = "blue")

# Add a legend
legend("topright",
       legend = c("Sub metering 1", "Sub metering 2", "Sub metering 3"),
       col = c("black", "red", "blue"),
       lty = 1)
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Plot 3.png" alt="linegraph">

### Plot 4

```r
par(mfrow = c(2,2))

# Make the figure 1
plot(data.frame(df$datetime, df$Global_active_power), type = "l",
     xlab = "Datetime", ylab = "Global Active Power (kilowatts)")

# Make the figure 2
plot(data.frame(df$datetime, df$Voltage), type = "l",
     xlab = "Datetime", ylab = "Voltage")

# Make the figure 3
plot(data.frame(df$datetime, df$Sub_metering_1), type = "l", xlab = "Datetime", ylab = "Energy sub metering")
lines(data.frame(df$datetime, df$Sub_metering_2), type = "l", col = "red")
lines(data.frame(df$datetime, df$Sub_metering_3), type = "l", col = "blue")
legend("topright",
       legend = c("Sub metering 1", "Sub metering 2", "Sub metering 3"),
       col = c("black", "red", "blue"),
       lty = 1)

# Make the figure 4
plot(data.frame(df$datetime, df$Global_reactive_power), type = "l",
     xlab = "Datetime", ylab = "Global reactive power")

par(mfrow = c(1,1))
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Plot 4.png" alt="visualisation grid">

[Github Repo]: https://github.com/hightowerr/datasciencecoursera/tree/master/ExploratoryDataAnalysis/Week1Proj1
