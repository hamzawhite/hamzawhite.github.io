---
title: 'Future Blog Post'
date: 2199-01-01
permalink: /posts/2012/08/blog-post-4/
tags:
  - cool posts
  - category1
  - category2
---

---
title: "Anomaly detection in the transportation process"
author: "Hamza Imloul"
date: "13/01/2021"
output:
  html_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

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

```{r packages, echo=TRUE, message=FALSE}
library(tidyverse)
library(scales)
library(ggbump)
library(cowplot)
library(ggridges)
library(rdrop2)
library(readxl)
```

# Data preparation

Imported the data sets,

```{r, echo=TRUE, message=FALSE}
#'*import datasets*

# token <- readRDS("droptoken.rds")

# vehicle_zone =        drop_read_csv("vehicle_zone.csv", dtoken = token)
# vehicle_trailer =     drop_read_csv("vehicle_trailer.csv", dtoken = token)

vehicle_zone =        read.csv("vehicle_zone.csv")
vehicle_trailer =     read.csv("vehicle_trailer.csv")

events_ds =           read_xlsx("C:/Users/Hamza/Dropbox/Planning/datasets/events_ds.xlsx")

poi =                 read.csv("poi.csv")

paired.drivingtimes = read.csv("paired.drivingtimes.csv")
tkm =                 read.csv("tkm.csv", sep = ";", header = TRUE)

```

then performed basic SQL join operations on dataframes.

```{r data manip, message=FALSE}

resources = 
  vehicle_zone %>%
  left_join(vehicle_trailer, by ="vehicule") %>%
  mutate(type.trailer = ifelse(grepl("B", remorque),"Benne", ifelse(grepl("P", remorque), "Plateau", "Sans"))) %>%
  select(vehicule, remorque, zone, type.trailer)
```

## Data cleansing

An event is an arrival or departure to a site. considering every vehicle may stay over 10 minutes in a site, every two arrivals during ten minutes to the same site is considered a duplicated event, same as for departures.

```{r, echo=FALSE, message=FALSE}
events_ds %>% 
  select(heure_date, vehicule, acti, poi) %>%
  head() %>% knitr::kable()
```

after exploring the `events_ds` dataset, the observations contains arrival or departure events, and removing them will help to increase the value from our analysis.

```{r, echo=FALSE, message=FALSE}
events =
        events_ds %>%
        mutate(poi = str_replace_all(poi, c("[^[:alnum:][:space:]]"="")) %>% str_to_upper()) %>%
        mutate(odometre = NA) %>%
        left_join(select(poi, poi, type_poi)) %>%
        filter(!grepl("File|Zone indus", type_poi)) %>%
        select(-type_poi) %>%
        arrange(heure_date) %>%
        arrange(vehicule) %>%
        mutate(id_poi = cumsum(poi != lead(poi) | vehicule != lead(vehicule)) %>% lag(default = 0)) %>%
        mutate(poi = str_replace_all(poi, c("[^[:alnum:][:space:]]"="")) %>% str_to_upper())

      #min of starts
      events_start =
        events %>%
        filter(acti == "IN") %>%
        group_by(id_poi) %>%
        summarise(heure_date = heure_date[which.min(heure_date)],
                  vehicule = vehicule[which.min(heure_date)],
                  acti = acti[which.min(heure_date)],
                  poi = poi[which.min(heure_date)],
                  odometre = odometre[which.min(heure_date)]) %>%
        distinct(heure_date, vehicule, acti, poi, odometre, .keep_all = TRUE)
      
      #max of ends
      events_end =
        events %>%
        filter(acti == "OUT") %>%
        group_by(id_poi) %>%
        filter(!is.na(heure_date)) %>%
        summarise(heure_date = heure_date[which.max(heure_date)],
                  vehicule = vehicule[which.max(heure_date)],
                  acti = acti[which.max(heure_date)],
                  poi = poi[which.max(heure_date)],
                  odometre = odometre[which.max(heure_date)]) %>%
        distinct(heure_date, vehicule, acti, poi, odometre, .keep_all = TRUE)
      
      events_df =
        rbind(events_start, events_end) %>%
        arrange(heure_date) %>%
        arrange(vehicule)
      
      events_df %>% 
        select(heure_date, vehicule, acti, poi) %>%
        head() %>% knitr::kable()
```

Before data cleansing

```{r, message=FALSE}
dim(events_ds)
```

After data cleansing

```{r, message=FALSE}
dim(events_df)
```

## Data manipulation

Until this step,

-   `heure_date` : datetime of event creation
-   `acti`: event type
-   `vehicule`: vehicle
-   `poi`: site

### Business understanding

every trip is a set of departure event from the origin site, and its arrival event to the destination site. We need to form the list of trips and deduce the delay in every one. so

in this step, we mutate new columns "delay" in site, based on arrival and departure datetimes, and the statistic median of the distribution of in-site times. the same mutation on the travel times. This can be done by performing SQL join operations

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

```{r rank, echo=FALSE, message=FALSE}
trips =
        events_df %>%
        # spread(acti, heure_date) %>%
        # filter(!grepl("DEPOT", poi)) %>%
        left_join(.,select(poi, poi, type_poi), by="poi") %>%
        left_join(.,select(resources, vehicule, zone, type.trailer), by = "vehicule") %>%
        filter(!grepl("Station|zone_x|Ville|Route|Maison|File", type_poi)) %>%
        group_by(vehicule) %>% mutate(next_poi = dplyr::lead(poi, n = 1, default = NA)) %>%
        mutate(next_poi = replace_na(next_poi, "?")) %>%
        group_by(vehicule) %>% mutate(etape = ifelse(poi == next_poi, poi ,paste(poi, str_c(next_poi), sep = " - "))) %>% #convert NA to string "NA"
        group_by(vehicule) %>% mutate(end = dplyr::lead(heure_date, n=1, default=NA)) %>%
        group_by(vehicule) %>% mutate(end_odometre = dplyr::lead(odometre, n=1, default=NA)) %>%
        select(id_poi, vehicule, type.trailer, etape, heure_date, end, odometre, end_odometre) %>%
        dplyr::rename(start = heure_date) %>%
        # mutate(start = strftime(start, format = "%Y-%m-%d %H:%M:%S")) %>%
        # mutate(end = strftime(end, format = "%Y-%m-%d %H:%M:%S")) %>%
        mutate(jour = strftime(start, format = "%Y-%m-%d")) %>%
        mutate(type_etape = ifelse(grepl(" - ", etape), "trip", "poi")) %>%
        mutate(duree = difftime(as.POSIXct(as.character(end), format="%Y-%m-%d %H:%M:%S"),as.POSIXct(as.character(start), format="%Y-%m-%d %H:%M:%S"),units = "mins") %>% as.numeric() %>% round(0)) %>%
        #km
        group_by(vehicule) %>%
        mutate(km = end_odometre - odometre) %>%
        left_join(select(paired.drivingtimes, trajet, t_conduite, distance), by =c("etape"="trajet")) %>%
        dplyr::rename(tolerance=t_conduite) %>%
        group_by(etape, type.trailer, jour) %>%
        mutate(tolerance = ifelse(type_etape == "poi", replace_na(tolerance, median(duree, na.rm = TRUE)), tolerance) %>% round(0)) %>%
        mutate(retard = duree - tolerance*1.2 %>% round(0)) %>%
        mutate(retard = ifelse(retard < 0, NA, retard)) %>%
        mutate(perte_km = (km - distance) %>% round(2)) %>%
        mutate(perte_km = ifelse(perte_km < 0, NA, perte_km)) %>%
        # left_join(., select(Affectation, trajet, date, charge), by = c("jour"="date", "etape"="trajet")) %>%
        left_join(., select(tkm, trajet, tonnage), by = c("etape"="trajet")) %>%
        mutate(tonnage = replace_na(tonnage, 0)) %>%
        select(vehicule, id_poi, type.trailer, jour, etape, tonnage, distance, start, end, duree, tolerance, retard) %>%
        ungroup() %>%
        # left_join(., select(depot_time, id_poi, duree_depot), by = "id_poi") %>%
        # mutate(duree_depot = ifelse(grepl(" - ", etape),duree_depot ,NA)) %>%
        # mutate(retard = strftime(as_datetime(as.numeric(retard*60)), format="%H:%M")) %>%
        # mutate(duree_depot = strftime(as_datetime(as.numeric(duree_depot*60)), format="%H:%M")) %>%
        # mutate(duree = strftime(as_datetime(as.numeric(duree*60)), format="%H:%M")) %>%
        # mutate(tolerance = strftime(as_datetime(as.numeric(tolerance*60)), format="%H:%M")) %>%
        as_tibble()

trips %>% select(-c(vehicule)) %>% .[1000:1005,] %>% knitr::kable()
```

# Data exploration

## Exploring uncertainty in the process

let's take the example of Port of Agadir, to see the variation of the in-site time over the weeks

#### Distribution of in-site time over the weeks

```{r total_zone, echo=FALSE, message=FALSE}
# Plot: Density ridgeline plots
dt = trips %>%
  filter(etape == "PORTAGADIR") %>%
  filter(duree < quantile(duree,probs=0.90, na.rm = TRUE)) %>%
  select(vehicule, start, end, etape, duree, tolerance, retard) %>%
  mutate(sem = lubridate::week(end) %>% as.factor())

ggplot(dt, aes(x = duree, y = sem)) + 
  geom_density_ridges() +
  theme_ridges() +
  labs(title='Insite time in Port of Agadir',
       subtitle='Distribution of insite times over weeks')

```

## Time series analysis

```{r pressure, echo=FALSE, fig.height= 15, fig.width= 11, eval=FALSE}
#Time series : service time
trips %>%
  filter(etape == "PORTAGADIR"& strftime(end, format="%Y-%m-%d") > ("2020-08-1") & strftime(end, format="%Y-%m-%d") < ("2020-08-7")) %>%
  arrange(start) %>%
  filter(duree < quantile(duree,probs=0.95, na.rm = TRUE)) %>%
  mutate(COLOR = ifelse(duree > quantile(duree,probs=0.75, na.rm = TRUE) %>% round(0) | duree < quantile(duree,probs=0.05, na.rm = TRUE) %>% round(0), "red","black")) %>%
  ggplot(aes(start,duree, color = COLOR)) +
  geom_point(shape = 1, fill = "grey")+
  theme_bw() +
  geom_smooth(span = 0.1, fill = "grey", color = 'grey', alpha = 0.1)+
  scale_color_identity() +
  ggtitle("Service time : Port Agadir") + ylab("service time") + xlab("entry datetime")
```

```{r plot 10, echo=FALSE, fig.height= 15, fig.width= 11, eval=FALSE}
#condenser les donnÃ©es sur une journÃ©e
trips %>%
  filter(duree <= quantile(duree,probs=0.90, na.rm = TRUE)) %>% #remove outliers
  filter(etape == "PORTAGADIR"& strftime(end, format="%Y-%m-%d") > ("2020-08-1") & strftime(end, format="%Y-%m-%d") < ("2020-08-30")) %>%
  arrange(start) %>%
  mutate(start = lubridate::hour(start)+lubridate::minute(start)/60+lubridate::second(start)/3600) %>%
  #mutate(start = lubridate::round_date(start, "1 hours")) %>%
  ggplot(aes(start,duree, color = "grey50")) +
  geom_point(shape = 1, fill = "grey") +
  theme_bw() +
  geom_smooth(span = 0.5, fill = "grey", color = 'black', alpha = 0.1)+
  scale_color_identity() +
  ggtitle("Service time : Port Agadir") + ylab("service time") + xlab("entry datetime")
```

```{r plot 11, echo=FALSE, fig.height= 15, fig.width= 11, eval=FALSE}
#condenser les donnÃ©es sur une semaine
trips %>%
  filter(etape == "PORTAGADIR") %>%
  #filter(strftime(end, format="%Y-%m-%d") > ("2020-08-1") & strftime(end, format="%Y-%m-%d") < ("2020-08-30"))
  arrange(start) %>%
  mutate(jour = weekdays(start)) %>%
  mutate(start = lubridate::hour(start)+lubridate::minute(start)/60+lubridate::second(start)/3600) %>%
  #mutate(start = lubridate::round_date(start, "1 hours")) %>%
  mutate(COLOR = ifelse(duree > quantile(duree,probs=0.75, na.rm = TRUE) %>% round(0) | duree < quantile(duree,probs=0.05, na.rm = TRUE) %>% round(0), "red","grey20")) %>%
  ggplot(aes(start,duree, colour = COLOR)) +
  geom_point(shape = 1, fill = "grey") +
  theme_bw() +
  geom_smooth(span = 0.3, fill = "grey", color = 'black', alpha = 0.1)+
  scale_color_identity() +
  facet_wrap(vars(jour), scales = "free") + 
  ggtitle("Service time : Port Agadir") + ylab("service time") + xlab("entry datetime")
```

```{r plot 12, echo=FALSE, fig.height= 15, fig.width= 11, eval=FALSE}
trips %>%
  filter(etape == "PORTAGADIR") %>%
  filter(duree <= quantile(duree,probs=0.95, na.rm = TRUE) & duree >= quantile(duree,probs=0.05, na.rm = TRUE)) %>%
  #filter(strftime(end, format="%Y-%m-%d") > ("2020-08-1") & strftime(end, format="%Y-%m-%d") < ("2020-08-30"))
  arrange(start) %>%
  mutate(jour = weekdays(start)) %>%
  mutate(start = lubridate::wday(start)*24+lubridate::hour(start)+lubridate::minute(start)/60+lubridate::second(start)/3600) %>%
  #mutate(start = lub ridate::round_date(start, "1 hours")) %>%
  mutate(COLOR = ifelse(duree > quantile(duree,probs=0.75, na.rm = TRUE) %>% round(0) | duree < quantile(duree,probs=0.05, na.rm = TRUE) %>% round(0), "red","grey20")) %>%
  ggplot(aes(start,duree, colour = COLOR)) +
  geom_point(shape = 1, fill = "grey") +
  theme_bw() +
  geom_smooth(span = 0.1, fill = "grey", color = 'black', alpha = 0.1) +
  scale_color_identity() +
  ggtitle("Service time : Port Agadir") + ylab("service time") + xlab("entry datetime")
```

```{r plot 13, echo=FALSE, message=FALSE, fig.height= 15, fig.width= 11}
df = 
  trips %>%
  filter(etape == "PORTAGADIR") %>%
  #filter(strftime(end, format="%Y-%m-%d") > ("2020-08-1") & strftime(end, format="%Y-%m-%d") < ("2020-08-30"))
  arrange(start) %>%
  mutate(jour = as.numeric(strftime(start, format = "%u"))) %>%
  mutate(start = lubridate::hour(start)+lubridate::minute(start)/60+lubridate::second(start)/3600) %>%
  select(start, duree, jour) 

df %>%
  filter(duree <= quantile(duree,probs=0.95, na.rm = TRUE) & duree >= quantile(duree,probs=0.05, na.rm = TRUE)) %>%
  mutate(COLOR = ifelse(duree > quantile(duree,probs=0.90, na.rm = TRUE) %>% round(0) | duree < quantile(duree,probs=0.05, na.rm = TRUE) %>% round(0), "red","grey50")) %>%
  ggplot(aes(start,duree, colour = COLOR)) +
  geom_point(shape = 1, fill = "grey") +
  theme_bw() +
  geom_smooth(span = 0.3, fill = "grey", color = 'black', alpha = 0.1)+
  scale_color_identity() +
  facet_wrap(vars(jour), scales = "free") + 
  ggtitle("Service time : Port Agadir") + ylab("service time") + xlab("entry datetime")
```

```{r plot 14, echo=TRUE, fig.height= 15, fig.width= 11}

```

## Exploring the vehicle's performance

```{r total_zone_trailer, echo=TRUE, eval=FALSE, fig.height= 7, fig.width= 7}
trips %>%
  filter(type.trailer == "Benne") %>%
  select(vehicule, start, end, etape, distance, duree, tolerance, retard) %>%
  mutate(sem = lubridate::week(end) %>% as.factor()) %>%
  group_by(vehicule) %>%
  summarise(delays = sum(retard, na.rm = TRUE), distance = sum(distance, na.rm = TRUE)) %>%
  ggplot(aes(x = delays, y = distance)) +
  geom_point()
```

Thanks for reading all the paper.

