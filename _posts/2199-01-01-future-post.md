---
title: 'Future Blog Post'
date: 2199-01-01
permalink: /posts/2012/08/blog-post-4/
tags:
  - data analysis
  - anomaly detection
  - time series
---

---
title: "Anomaly detection in the transportation process"
author: "Hamza Imloul"
date: "13/01/2021"
output:
  html_document: default
---

# Introduction

Anomaly detection, Outlier detection aims at identifying those objects in a database that are unusual, i.e., different than the majority of the data and therefore suspicious resulting from a contamination, error, or fraud. in this article, we will try to detect anomalies in the transportation process, an anomaly is a delay in trip, or an unusual in-site time.

## Outlook

Managing a fleet of 102 vehicles on time can be stressful, especially if there is no more than one to do this. In fact, This duty needs process automation to be performed effectively. for this reason I decided to start this project using R.

### What we shall answer

-   What's the variation of the in-site time in a point of interest ?
-   The distribution of in-site time by client's location
-   Who are the vehicles with the highest delays ?

### Data Analysis steps

We have to structure our analysis, first, prepared data sets to speed up our analysis

-   Data preparation
    -   Data cleansing
-   Data Exploration
    -   Data visualization

First, these are the packages needed

# Data preparation
Imported the data sets,
then performed basic SQL join operations on dataframes.

## Data cleansing
An event is an arrival or departure to a site. considering every vehicle may stay over 10 minutes in a site, every two arrivals during ten minutes to the same site is considered a duplicated event, same as for departures.
after exploring the `events_ds` dataset, the observations contains arrival or departure events, and removing them will help to increase the value from our analysis.

## Data manipulation
Until this step,
-   `heure_date` : datetime of event creation
-   `acti`: event type
-   `vehicule`: vehicle
-   `poi`: site

### Business understanding
every trip is a set of departure event from the origin site, and its arrival event to the destination site. We need to form the list of trips and deduce the delay in every one. so in this step, we mutate new columns "delay" in site, based on arrival and departure datetimes, and the statistic median of the distribution of in-site times. the same mutation on the travel times. This can be done by performing SQL join operations

This is the dataset generated

-   `step_start` : (datetime) datetime of step start
-   `step_end`: (datetime) datetime of event end
-   `vehicule`: vehicle number
-   `step_type`: (binary) trip / in-site
-   `step`: step name, if the type is a trip, then the name is the trajectory Origin - Destination, else, it is the name of the site.
-   `tolerance_time`: the allowed time to perform the step
-   `tolerance_dist`: the allowed distance to perform the step
-   `delay`: `duration` - `tolerance_time`
-   `wasted_dist`: `odometer`- `tolerance_dist`
-   `tonnage`:

# Data exploration

## Exploring uncertainty in the process
let's take the example of Port of Agadir, to see the variation of the in-site time over the weeks

#### Distribution of in-site time over the weeks

## Time series analysis

## Exploring the vehicle's performance

Thanks for reading all the paper.

