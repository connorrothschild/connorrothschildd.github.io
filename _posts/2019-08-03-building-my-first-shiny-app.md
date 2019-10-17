---
title: "Building my First Shiny App"
date: "2019-08-03"
category: R
tags: [r, visualization, interactive]
comments: true
---



I spent some time this weekend playing around with [Shiny](https://shiny.rstudio.com), RStudio's tool for creating interactive web apps. In a nod to my humble beginnings, I wanted to bring some interactivity to my first R project ([ever!](https://connorrothschild.github.io/r/automation/)).

I finished the project roughly a year ago, in the summer between my freshman and sophomore year. It was an exercise in plotting multiple dimensions related to something of personal interest to me: automation and its impact on jobs. I wanted to use ggplot2 to recreate a visualization I came across on Bloomberg graphics. Here's [Bloomberg's visualization](https://www.bloomberg.com/graphics/2017-job-risk/) and here's [mine](https://connorrothschild.github.io/r/automation/).

There are some obvious differences in our visualizations (our axes are inverted, they likely used D3.js while I used ggplot2), but for the most part, our visualizations depict the same lesson: lower-paying jobs and less-educated jobs are more susceptible to job displacement from automation.

A year later, there are some things about my first visualization I would definitely change (title and axis label size, unnecessary corner labels, a potentially misleading geom_smooth line), but what I really want to work on now is bringing my project closer to the Bloomberg visualization by making it interactive. (I've actually already made an [interactive version](https://public.tableau.com/profile/connor.rothschild#!/vizhome/JobAutomationRiskintheUnitedStates/Final) of the visualization using Tableau, but I wanted to do it again in R to expand my skillset!)

Enter Shiny, RStudio's tool for creating interactive visualizations. By using Shiny with [ggvis](https://ggvis.rstudio.com) (ggplot2's "successor" with interactive capabilities), I'm able to get pretty close to my initial inspiration. 

ggvis's commands are pretty similar to ggplot2, and so the learning curve wasn't that steep (with the exception of setting the default size parameter for my points, which I finally solved with [this fix](https://stackoverflow.com/questions/43466172/chang-size-of-points-depending-on-one-column-with-ggvis)). Shiny was a bit more difficult to learn, but RStudio's [online video tutorials](https://shiny.rstudio.com/tutorial/) make it a lot less daunting. All in all, the project only took one night (~3 hours) to complete. Another example of R's accessibility and ease of use!

## Clean/Prepare Data


{% highlight r %}
library(ggplot2)
library(ggthemes)
library(dplyr)
library(ggrepel)
library(tools)
library(readxl)
library(tidyverse)
library(knitr)

options(scipen=999)
theme_set(theme_minimal())

education <- read_excel("education.xlsx", skip=1)
salary <- read_excel("national_M2017_dl.xlsx")
automation <- read_excel("raw_state_automation_data.xlsx")

salary1 <- salary %>% 
group_by(OCC_TITLE) %>% 
mutate(natlwage = TOT_EMP * as.numeric(A_MEAN)) %>%
filter(!is.na(TOT_EMP)) %>%
filter(!is.na(A_MEAN)) %>%
filter(!is.na(A_MEDIAN))

salary1$A_MEDIAN = as.numeric(as.character(salary1$A_MEDIAN))
salary2 <- select(salary1, OCC_TITLE, TOT_EMP, A_MEDIAN, natlwage) %>% 
distinct()

library(plyr)
education1 <- education %>% select(-...2)

education1 <- rename(education1, c("2016 National Employment Matrix title and code" = "occupation",
                                   "Less than high school diploma" = "lessthanhs", 
                                   "High school diploma or equivalent" = "hsdiploma",
                                   "Some college, no degree" = "somecollege",
                                   "Associate's degree" = "associates",
                                   "Bachelor's degree" = "bachelors",
                                   "Master's degree" = "masters",
                                   "Doctoral or professional degree" = "professional"))

education2 <- education1 %>% 
  group_by(occupation) %>%
  mutate(hsorless = lessthanhs + hsdiploma,
         somecollegeorassociates = somecollege + associates,
         postgrad = masters + professional)

education2 <- education2 %>% drop_na()

salary2 <- rename(salary2, c("OCC_TITLE" = "occupation"))
salary2$occupation <- tolower(salary2$occupation)
education2$occupation <- tolower(education2$occupation)
edsal <- merge(as.data.frame(education2), as.data.frame(salary2), by="occupation") %>% drop_na()

  typicaleducation <- read_excel("typicaleducation.xlsx")
  typicaleducation2 <- typicaleducation %>% select(occupation,typicaled,workexp)
  typicaleducation2 <- typicaleducation2 %>% drop_na()
  typicaleducation2$occupation <- tolower(typicaleducation2$occupation)
  edsal2 <- merge(as.data.frame(edsal), as.data.frame(typicaleducation2), by="occupation")

  detach(package:plyr)
  edsal3 <- edsal2 %>% 
  group_by(typicaled) %>% 
  summarise(medianwage = mean(A_MEDIAN))
  
  automationwstates <- automation %>% select(-soc)
  automation1 <- automationwstates %>% select(occupation,probability,total)

  automation1$occupation <- str_replace_all(automation1$occupation, ";", ",")
  automation1$occupation <- tolower(automation$occupation)
  data <- merge(as.data.frame(edsal2), as.data.frame(automation1), by="occupation")

  data$occupation <- toTitleCase(data$occupation)
{% endhighlight %}

## Bring in Shiny



[You can find the Shiny app here](https://connorrothschild.shinyapps.io/ggvis/)!
