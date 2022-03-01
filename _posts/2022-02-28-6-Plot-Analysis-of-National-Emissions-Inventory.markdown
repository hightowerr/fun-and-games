---
layout: posts
title:  "6 Plot Analysis of National Emissions Inventory in America."
date:   2022-03-01 05:00:00 +0100
categories: r, data science,
author: Olayinka Ola
---
The overall goal of this post is to explore the National Emissions Inventory database and see what it says about fine particulate matter pollution in the United states over the 10-year period 1999–2008.

## Analysis

I will address the following questions and tasks in my exploratory analysis. For each question/task I will make a single plot.

1. Have total emissions from PM2.5 decreased in the United States from 1999 to 2008? Using the base plotting system, make a plot showing the total PM2.5 emission from all sources for each of the years 1999, 2002, 2005, and 2008.

    ```r
    # Load the data:
    NEI <- readRDS("data/summarySCC_PM25.rds")
    SCC <- readRDS("data/Source_Classification_Code.rds")

    # table with emissions per year
    NEI$year <- as.factor(NEI$year)
    group_sum <- aggregate(NEI$Emissions, by=list(NEI$year), FUN=sum)
    # renamed columns
    colnames(group_sum) <- c("Year", "Emissions")

    # new column with emissions in million (for the y axis)
    group_sum$Emissions_millions <- round(group_sum[, 2] / 1000000, 0)

    # Colour Palette
    library(RColorBrewer)
    coul <- brewer.pal(5, "Set2")

    png(filename = "plot1.png")
    barplot(height=group_sum$Emissions_millions, names=group_sum$Year,
    col=coul,
    xlab = "Years",
    ylab = "Emissions (PM 2.5) in millions",
    main = "Emissions (PM 2.5) per year")
    dev.off()
    ```

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/w4p2_plot1.png" alt="full model">

2. Have total emissions from PM2.5 decreased in the Baltimore City, Maryland (fips == “24510”) from 1999 to 2008? Use the base plotting system to make a plot answering this question.

    ```r
    # Load the data:
    NEI <- readRDS("data/summarySCC_PM25.rds")
    SCC <- readRDS("data/Source_Classification_Code.rds")

    # Summarise data
    library("dplyr")
    data <- NEI %>%
    group_by(fips, year) %>%
    dplyr::summarise(gr_sum = sum(Emissions)) %>%
    as.data.frame()
    data

    ba_data <- data[data$fips == "24510", ]
    colnames(ba_data) <- c("Fips", "Year", "Emissions")
    group <- c("1999", "2002", "2005", "2008")

    library(RColorBrewer)
    coul <- brewer.pal(5, "Set2")

    # Plot
    png(filename = "plot2.png")
    barplot(height=ba_data$Emissions, names.arg=group,
    col=coul,
    xlab = "Years",
    ylab = "Emissions (PM 2.5) in millions",
    main = "Baltimore City, Maryland \n emission")
    dev.off()
    ```

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/w4p2_plot2.png" alt="full model">

3. Of the four types of sources indicated by the type (point, nonpoint, onroad, nonroad) variable, which of these four sources have seen decreases in emissions from 1999-2008 for Baltimore City? Which have seen increases in emissions from 1999-2008? Use the ggplot2 plotting system to make a plot answer this question.

    ```r
    # Load the data:
    NEI <- readRDS("data/summarySCC_PM25.rds")
    SCC <- readRDS("data/Source_Classification_Code.rds")
    NEI$year <- as.factor(NEI$year)

    # Create data subset
    library(ggplot2)
    data <- NEI %>%
    group_by(fips, type, year) %>%
    dplyr::summarise(gr_sum = sum(Emissions)) %>%
    as.data.frame()
    data
    data_group <- data[data$fips == "24510", ]
    colnames(data_group) <- c("Fips","type", "Year", "Emissions")

    # Plot
    png(filename = "plot3.png")
    g <- ggplot(data_group, aes(Year, Emissions, fill = type)) +
    ggtitle(expression("Total Baltimore " ~ PM[2.5] ~ "Emissions by Type and Year")) +
    ylab(expression("Total Baltimore " ~ PM[2.5] ~ "Emissions")) +
    xlab("Year") +
    theme(legend.title = element_text(face = "bold"))
    g + geom_bar(position="dodge", stat="identity") + facet_grid(. ~ type)
    dev.off()
    ```

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/w4p2_plot3.png" alt="full model">

4. Across the United States, how have emissions from coal combustion-related sources changed from 1999-2008?

    ```r
    # Load the data:
    NEI <- readRDS("data/summarySCC_PM25.rds")
    SCC <- readRDS("data/Source_Classification_Code.rds")

    SCCcoal <- SCC[grepl("coal", SCC$Short.Name, ignore.case = T),]
    NEIcoal <- NEI[NEI$SCC %in% SCCcoal$SCC,]
    totalCoal <- aggregate(Emissions ~ year, NEIcoal, sum)

    # new column with emissions in thousands (for the y axis)
    totalCoal$Emissions_millions <- round(totalCoal[, 2] / 1000, 0)

    # Plot
    png(filename = "plot4.png")
    library(ggplot2)
    g <- ggplot(totalCoal, aes(x = year, y = Emissions_millions))
    g + geom_line(aes(group=1)) +
    geom_point(size = 2) +
    geom_label_repel(aes(label = Emissions), box.padding = 0.35, point.padding = 0.5, segment.color = 'grey50') +
    labs(title = "Emissions from coal combustion-related sources", x="Years", y="Emissions (PM 2.5) in thousands")
    dev.off()
    ```

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/w4p2_plot4.png" alt="full model">

5. How have emissions from motor vehicle sources changed from 1999-2008 in Baltimore City?

    ```r
    # Load the data:
    NEI <- readRDS("data/summarySCC_PM25.rds")
    SCC <- readRDS("data/Source_Classification_Code.rds")

    # table with Motor Vehicle emissions per year of Baltimore
    library(magrittr)
    library(dplyr)

    motor <- arrange(
    summarize(
    group_by(
    filter(NEI, fips=="24510", type=='ON-ROAD'),
    year
    ),
    em_sum = sum(Emissions)
    ),
    desc(em_sum)
    )

    # Plot
    png(filename = "plot5.png")
    library(ggplot2)
    library(ggrepel)
    ggplot(motor, aes(x=year, y=em_sum, fill=year)) +
    geom_bar(stat="identity", colour="black") +
    geom_line(aes(group=1)) +
    geom_point(size = 2) +
    geom_label_repel(aes(label = em_sum), box.padding = 0.35, point.padding = 0.5, segment.color = 'grey50') +
    labs(title="Baltimore City: Emissions of motor vehicle", x="Years",y="Emissions (PM 2.5)")
    dev.off()
    ```

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/w4p2_plot5.png" alt="full model">

6. Compare emissions from motor vehicle sources in Baltimore City with emissions from motor vehicle sources in Los Angeles County, California (fips == “06037”). Which city has seen greater changes over time in motor vehicle emissions?

    ```r
    # Load the data:
    NEI <- readRDS("data/summarySCC_PM25.rds")
    # table with emissions per year
    NEI$year <- as.factor(NEI$year)

    # table with Motor Vehicle emissions per year of Baltimore
    library(magrittr)
    library(dplyr)

    target <- c("24510", "06037")
    ba_motor <- group_by(filter(NEI, fips %in% target, type=='ON-ROAD'), year)

    # Multiple conditions when adding new column to dataframe:
    ba_motor <- ba_motor %>% mutate(place = case_when(fips == 24510 ~ "Baltimore City", TRUE ~ "Los Angeles County"))

    ba_motorAGG <- aggregate(Emissions ~ year + fips + place, ba_motor, sum)

    # Plot
    png(filename = "plot6.png")
    g <- ggplot(ba_motorAGG, aes(year, Emissions, fill = year)) +
    ggtitle(expression("Baltimore and Los Angeles" ~ PM[2.5] ~ "Motor Vehicle Emissions by Year")) +
    ylab(expression("T~PM[2.5]~ Motor Vehicle Emissions")) +
    xlab("Year") +
    theme(legend.title = element_text(face = "bold"))
    g + geom_bar(position="dodge", stat="identity") + facet_grid(. ~ place)
    dev.off()
    ```

    <img src="{{ site.url }}{{ site.baseurl }}/assets/images/w4p2_plot6.png" alt="full model">
