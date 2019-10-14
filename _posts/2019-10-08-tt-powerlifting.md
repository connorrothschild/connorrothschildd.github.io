---
title: "Tidy Tuesday: Powerlifting"
date: "10/8/2019"
category: R
tags: [r, visualization, animation]
comments: true
---

My Tidy Tuesday submission for the week of October 8, 2019, focusing on international powerlifting competitions.

{% highlight r %}
library(ggplot2)
library(tidyverse)
library(cr)

set_cr_theme(font = "lato")
{% endhighlight %}

## Load and Clean Data

First, read in the data from https://openpowerlifting.org/data:


{% highlight r %}
# df <- readr::read_csv("openpowerlifting-2019-09-20.csv")
# 
# df_clean <- df %>% 
#   janitor::clean_names()
# 
# ipf_lifts <- df_clean %>% 
#   select(name:weight_class_kg, starts_with("best"), place, date, federation, meet_name)  %>% 
#   filter(!is.na(date)) %>% 
#   filter(federation == "IPF")

ipf_lifts <- readr::read_csv("ipf_lifts.csv")
{% endhighlight %}

Clean ipf_lifts, and reshape the three lifts into one column:


{% highlight r %}
ipf_lifts <- ipf_lifts %>% 
  mutate(year = lubridate::year(date))

ipf_lifts_reshape <- ipf_lifts %>% 
  tidyr::pivot_longer(cols = c("best3squat_kg", "best3bench_kg", "best3deadlift_kg"), names_to = "lift") %>% 
  select(name, sex, year, lift, value)
{% endhighlight %}

For my visualization, I'm only concerned with the *heaviest* lifts from each year:


{% highlight r %}
ipf_lifts_maxes <- ipf_lifts_reshape %>% 
  group_by(year, sex, lift) %>% 
  top_n(1, value) %>% 
  ungroup %>% 
  distinct(year, lift, value, .keep_all = TRUE)
{% endhighlight %}

In order to construct a dumbbell plot, we need both male and female observations in the same row.


{% highlight r %}
max_pivot <- ipf_lifts_maxes %>% 
  spread(sex, value)
{% endhighlight %}

Let's try to construct a dataframe for each sex:


{% highlight r %}
male_lifts <- max_pivot %>% 
  select(-name) %>% 
  filter(!is.na(M)) %>% 
  group_by(year, lift) %>% 
  summarise(male = mean(M))

female_lifts <- max_pivot %>% 
  select(-name) %>% 
  filter(!is.na(`F`)) %>% 
  group_by(year, lift) %>% 
  summarise(female = mean(`F`))
{% endhighlight %}

And join them:


{% highlight r %}
max_lifts <- merge(male_lifts, female_lifts)

max_lifts_final <- max_lifts %>% 
  group_by(year, lift) %>% 
  mutate(diff = male - female)
{% endhighlight %}

## Visualize

Finally, we can construct the visualization.

First, a static viz (thanks to hrbrmaster's `ggalt` [package](https://rud.is/b/2016/04/17/ggplot2-exercising-with-ggalt-dumbbells/)):


{% highlight r %}
library(ggtext)
max_lifts_final %>% 
  filter(year == 2019) %>% 
  ggplot() + 
  ggalt::geom_dumbbell(aes(y = lift,
                    x = female, xend = male),
                colour = "grey", size = 5,
                colour_x = "#D6604C", colour_xend = "#395B74") +
  labs(y = element_blank(),
       x = "Top Lift Recorded (kg)",
       title =  "How <span style='color:#D6604C'>Women</span> and <span style='color:#395B74'>Men</span> Differ in Top Lifts",
       subtitle = "In 2019") +
  theme(plot.title = element_markdown(lineheight = 1.1, size = 20),
        plot.subtitle = element_text(size = 15)) +
  scale_y_discrete(labels = c("Bench", "Deadlift", "Squat")) +
  drop_axis(axis = "y") +
  geom_text(aes(x = female, y = lift, label = paste(female, "kg")),
            color = "#D6604C", size = 4, vjust = -2) +
  geom_text(aes(x = male, y = lift, label = paste(male, "kg")),
            color = "#395B74", size = 4, vjust = -2) +
  geom_rect(aes(xmin=430, xmax=470, ymin=-Inf, ymax=Inf), fill="grey80") +
  geom_text(aes(label=diff, y=lift, x=450), fontface="bold", size=4) +
  geom_text(aes(x=450, y=3, label="Difference"),
                     color="grey20", size=4, vjust=-3, fontface="bold")
{% endhighlight %}

![center](/figs/2019-10-08-tt-powerlifting/unnamed-chunk-8-1.png)

Finally, we animate, using Thomas Pedersen's wonderful [gganimate package](https://github.com/thomasp85/gganimate):


{% highlight r %}
library(gganimate)
animation <- max_lifts_final %>% 
  ggplot() + 
  ggalt::geom_dumbbell(aes(y = lift,
                    x = female, xend = male),
                colour = "grey", size = 5,
                colour_x = "#D6604C", colour_xend = "#395B74") +
  labs(y = element_blank(),
       x = "Top Lift Recorded (kg)",
       title =  "How <span style='color:#D6604C'>Women</span> and <span style='color:#395B74'>Men</span> Differ in Top Lifts",
       subtitle='\nThis plot depicts the difference between the heaviest lifts for each sex at International Powerlifting Federation\nevents over time. \n \n{closest_state}') +
  theme(plot.title = element_markdown(lineheight = 1.1, size = 25, margin=margin(0,0,0,0)),
        plot.subtitle = element_text(size = 15, margin=margin(8,0,-30,0))) +
  scale_y_discrete(labels = c("Bench", "Deadlift", "Squat")) +
  drop_axis(axis = "y") +
  geom_text(aes(x = female, y = lift, label = paste(female, "kg")),
            color = "#D6604C", size = 4, vjust = -2) +
  geom_text(aes(x = male, y = lift, label = paste(male, "kg")),
            color = "#395B74", size = 4, vjust = -2) +
  transition_states(year, transition_length = 4, state_length = 1) +
  ease_aes('cubic-in-out')

a_gif <- animate(animation, 
                 fps = 10, 
                 duration = 25,
        width = 800, height = 400, 
        renderer = gifski_renderer("./animation.gif"))
{% endhighlight %}



{% highlight text %}
## 
Frame 1 (0%)
Frame 2 (0%)
Frame 3 (1%)
Frame 4 (1%)
Frame 5 (2%)
Frame 6 (2%)
Frame 7 (2%)
Frame 8 (3%)
Frame 9 (3%)
Frame 10 (4%)
Frame 11 (4%)
Frame 12 (4%)
Frame 13 (5%)
Frame 14 (5%)
Frame 15 (6%)
Frame 16 (6%)
Frame 17 (6%)
Frame 18 (7%)
Frame 19 (7%)
Frame 20 (8%)
Frame 21 (8%)
Frame 22 (8%)
Frame 23 (9%)
Frame 24 (9%)
Frame 25 (10%)
Frame 26 (10%)
Frame 27 (10%)
Frame 28 (11%)
Frame 29 (11%)
Frame 30 (12%)
Frame 31 (12%)
Frame 32 (12%)
Frame 33 (13%)
Frame 34 (13%)
Frame 35 (14%)
Frame 36 (14%)
Frame 37 (14%)
Frame 38 (15%)
Frame 39 (15%)
Frame 40 (16%)
Frame 41 (16%)
Frame 42 (16%)
Frame 43 (17%)
Frame 44 (17%)
Frame 45 (18%)
Frame 46 (18%)
Frame 47 (18%)
Frame 48 (19%)
Frame 49 (19%)
Frame 50 (20%)
Frame 51 (20%)
Frame 52 (20%)
Frame 53 (21%)
Frame 54 (21%)
Frame 55 (22%)
Frame 56 (22%)
Frame 57 (22%)
Frame 58 (23%)
Frame 59 (23%)
Frame 60 (24%)
Frame 61 (24%)
Frame 62 (24%)
Frame 63 (25%)
Frame 64 (25%)
Frame 65 (26%)
Frame 66 (26%)
Frame 67 (26%)
Frame 68 (27%)
Frame 69 (27%)
Frame 70 (28%)
Frame 71 (28%)
Frame 72 (28%)
Frame 73 (29%)
Frame 74 (29%)
Frame 75 (30%)
Frame 76 (30%)
Frame 77 (30%)
Frame 78 (31%)
Frame 79 (31%)
Frame 80 (32%)
Frame 81 (32%)
Frame 82 (32%)
Frame 83 (33%)
Frame 84 (33%)
Frame 85 (34%)
Frame 86 (34%)
Frame 87 (34%)
Frame 88 (35%)
Frame 89 (35%)
Frame 90 (36%)
Frame 91 (36%)
Frame 92 (36%)
Frame 93 (37%)
Frame 94 (37%)
Frame 95 (38%)
Frame 96 (38%)
Frame 97 (38%)
Frame 98 (39%)
Frame 99 (39%)
Frame 100 (40%)
Frame 101 (40%)
Frame 102 (40%)
Frame 103 (41%)
Frame 104 (41%)
Frame 105 (42%)
Frame 106 (42%)
Frame 107 (42%)
Frame 108 (43%)
Frame 109 (43%)
Frame 110 (44%)
Frame 111 (44%)
Frame 112 (44%)
Frame 113 (45%)
Frame 114 (45%)
Frame 115 (46%)
Frame 116 (46%)
Frame 117 (46%)
Frame 118 (47%)
Frame 119 (47%)
Frame 120 (48%)
Frame 121 (48%)
Frame 122 (48%)
Frame 123 (49%)
Frame 124 (49%)
Frame 125 (50%)
Frame 126 (50%)
Frame 127 (50%)
Frame 128 (51%)
Frame 129 (51%)
Frame 130 (52%)
Frame 131 (52%)
Frame 132 (52%)
Frame 133 (53%)
Frame 134 (53%)
Frame 135 (54%)
Frame 136 (54%)
Frame 137 (54%)
Frame 138 (55%)
Frame 139 (55%)
Frame 140 (56%)
Frame 141 (56%)
Frame 142 (56%)
Frame 143 (57%)
Frame 144 (57%)
Frame 145 (58%)
Frame 146 (58%)
Frame 147 (58%)
Frame 148 (59%)
Frame 149 (59%)
Frame 150 (60%)
Frame 151 (60%)
Frame 152 (60%)
Frame 153 (61%)
Frame 154 (61%)
Frame 155 (62%)
Frame 156 (62%)
Frame 157 (62%)
Frame 158 (63%)
Frame 159 (63%)
Frame 160 (64%)
Frame 161 (64%)
Frame 162 (64%)
Frame 163 (65%)
Frame 164 (65%)
Frame 165 (66%)
Frame 166 (66%)
Frame 167 (66%)
Frame 168 (67%)
Frame 169 (67%)
Frame 170 (68%)
Frame 171 (68%)
Frame 172 (68%)
Frame 173 (69%)
Frame 174 (69%)
Frame 175 (70%)
Frame 176 (70%)
Frame 177 (70%)
Frame 178 (71%)
Frame 179 (71%)
Frame 180 (72%)
Frame 181 (72%)
Frame 182 (72%)
Frame 183 (73%)
Frame 184 (73%)
Frame 185 (74%)
Frame 186 (74%)
Frame 187 (74%)
Frame 188 (75%)
Frame 189 (75%)
Frame 190 (76%)
Frame 191 (76%)
Frame 192 (76%)
Frame 193 (77%)
Frame 194 (77%)
Frame 195 (78%)
Frame 196 (78%)
Frame 197 (78%)
Frame 198 (79%)
Frame 199 (79%)
Frame 200 (80%)
Frame 201 (80%)
Frame 202 (80%)
Frame 203 (81%)
Frame 204 (81%)
Frame 205 (82%)
Frame 206 (82%)
Frame 207 (82%)
Frame 208 (83%)
Frame 209 (83%)
Frame 210 (84%)
Frame 211 (84%)
Frame 212 (84%)
Frame 213 (85%)
Frame 214 (85%)
Frame 215 (86%)
Frame 216 (86%)
Frame 217 (86%)
Frame 218 (87%)
Frame 219 (87%)
Frame 220 (88%)
Frame 221 (88%)
Frame 222 (88%)
Frame 223 (89%)
Frame 224 (89%)
Frame 225 (90%)
Frame 226 (90%)
Frame 227 (90%)
Frame 228 (91%)
Frame 229 (91%)
Frame 230 (92%)
Frame 231 (92%)
Frame 232 (92%)
Frame 233 (93%)
Frame 234 (93%)
Frame 235 (94%)
Frame 236 (94%)
Frame 237 (94%)
Frame 238 (95%)
Frame 239 (95%)
Frame 240 (96%)
Frame 241 (96%)
Frame 242 (96%)
Frame 243 (97%)
Frame 244 (97%)
Frame 245 (98%)
Frame 246 (98%)
Frame 247 (98%)
Frame 248 (99%)
Frame 249 (99%)
Frame 250 (100%)
## Finalizing encoding... done!
{% endhighlight %}



{% highlight r %}
a_gif
{% endhighlight %}

![center](/figs/2019-10-08-tt-powerlifting/unnamed-chunk-9-1.gif)

I'd like to include another GIF: a line chart of differences over time


{% highlight r %}
animation2 <- max_lifts_final %>% 
  ungroup %>% 
  mutate(lift = case_when(lift == "best3bench_kg" ~ "Bench",
                          lift == "best3squat_kg" ~ "Squat",
                          lift == "best3deadlift_kg" ~ "Deadlift")) %>% 
  ggplot(aes(year, diff, group = lift, color = lift)) + 
  geom_line(show.legend = FALSE) + 
  geom_segment(aes(xend = 2019.1, yend = diff), linetype = 2, colour = 'grey', show.legend = FALSE) + 
  geom_point(size = 2, show.legend = FALSE) + 
  geom_text(aes(x = 2019.1, label = lift, color = "#000000"), hjust = 0, show.legend = FALSE) + 
  drop_axis(axis = "y") +
  transition_reveal(year) +
  coord_cartesian(clip = 'off') +
  theme(plot.title = element_text(size = 20)) +
  labs(title = 'Difference over time',
       y = 'Difference (kg)',
       x = element_blank()) + 
  theme(plot.margin = margin(5.5, 40, 5.5, 5.5))

b_gif <- animate(animation2, 
                 fps = 10, 
                 duration = 25,
        width = 800, height = 200, 
        renderer = gifski_renderer("./animation2.gif"))
{% endhighlight %}



{% highlight text %}
## 
Frame 1 (0%)
Frame 2 (0%)
Frame 3 (1%)
Frame 4 (1%)
Frame 5 (2%)
Frame 6 (2%)
Frame 7 (2%)
Frame 8 (3%)
Frame 9 (3%)
Frame 10 (4%)
Frame 11 (4%)
Frame 12 (4%)
Frame 13 (5%)
Frame 14 (5%)
Frame 15 (6%)
Frame 16 (6%)
Frame 17 (6%)
Frame 18 (7%)
Frame 19 (7%)
Frame 20 (8%)
Frame 21 (8%)
Frame 22 (8%)
Frame 23 (9%)
Frame 24 (9%)
Frame 25 (10%)
Frame 26 (10%)
Frame 27 (10%)
Frame 28 (11%)
Frame 29 (11%)
Frame 30 (12%)
Frame 31 (12%)
Frame 32 (12%)
Frame 33 (13%)
Frame 34 (13%)
Frame 35 (14%)
Frame 36 (14%)
Frame 37 (14%)
Frame 38 (15%)
Frame 39 (15%)
Frame 40 (16%)
Frame 41 (16%)
Frame 42 (16%)
Frame 43 (17%)
Frame 44 (17%)
Frame 45 (18%)
Frame 46 (18%)
Frame 47 (18%)
Frame 48 (19%)
Frame 49 (19%)
Frame 50 (20%)
Frame 51 (20%)
Frame 52 (20%)
Frame 53 (21%)
Frame 54 (21%)
Frame 55 (22%)
Frame 56 (22%)
Frame 57 (22%)
Frame 58 (23%)
Frame 59 (23%)
Frame 60 (24%)
Frame 61 (24%)
Frame 62 (24%)
Frame 63 (25%)
Frame 64 (25%)
Frame 65 (26%)
Frame 66 (26%)
Frame 67 (26%)
Frame 68 (27%)
Frame 69 (27%)
Frame 70 (28%)
Frame 71 (28%)
Frame 72 (28%)
Frame 73 (29%)
Frame 74 (29%)
Frame 75 (30%)
Frame 76 (30%)
Frame 77 (30%)
Frame 78 (31%)
Frame 79 (31%)
Frame 80 (32%)
Frame 81 (32%)
Frame 82 (32%)
Frame 83 (33%)
Frame 84 (33%)
Frame 85 (34%)
Frame 86 (34%)
Frame 87 (34%)
Frame 88 (35%)
Frame 89 (35%)
Frame 90 (36%)
Frame 91 (36%)
Frame 92 (36%)
Frame 93 (37%)
Frame 94 (37%)
Frame 95 (38%)
Frame 96 (38%)
Frame 97 (38%)
Frame 98 (39%)
Frame 99 (39%)
Frame 100 (40%)
Frame 101 (40%)
Frame 102 (40%)
Frame 103 (41%)
Frame 104 (41%)
Frame 105 (42%)
Frame 106 (42%)
Frame 107 (42%)
Frame 108 (43%)
Frame 109 (43%)
Frame 110 (44%)
Frame 111 (44%)
Frame 112 (44%)
Frame 113 (45%)
Frame 114 (45%)
Frame 115 (46%)
Frame 116 (46%)
Frame 117 (46%)
Frame 118 (47%)
Frame 119 (47%)
Frame 120 (48%)
Frame 121 (48%)
Frame 122 (48%)
Frame 123 (49%)
Frame 124 (49%)
Frame 125 (50%)
Frame 126 (50%)
Frame 127 (50%)
Frame 128 (51%)
Frame 129 (51%)
Frame 130 (52%)
Frame 131 (52%)
Frame 132 (52%)
Frame 133 (53%)
Frame 134 (53%)
Frame 135 (54%)
Frame 136 (54%)
Frame 137 (54%)
Frame 138 (55%)
Frame 139 (55%)
Frame 140 (56%)
Frame 141 (56%)
Frame 142 (56%)
Frame 143 (57%)
Frame 144 (57%)
Frame 145 (58%)
Frame 146 (58%)
Frame 147 (58%)
Frame 148 (59%)
Frame 149 (59%)
Frame 150 (60%)
Frame 151 (60%)
Frame 152 (60%)
Frame 153 (61%)
Frame 154 (61%)
Frame 155 (62%)
Frame 156 (62%)
Frame 157 (62%)
Frame 158 (63%)
Frame 159 (63%)
Frame 160 (64%)
Frame 161 (64%)
Frame 162 (64%)
Frame 163 (65%)
Frame 164 (65%)
Frame 165 (66%)
Frame 166 (66%)
Frame 167 (66%)
Frame 168 (67%)
Frame 169 (67%)
Frame 170 (68%)
Frame 171 (68%)
Frame 172 (68%)
Frame 173 (69%)
Frame 174 (69%)
Frame 175 (70%)
Frame 176 (70%)
Frame 177 (70%)
Frame 178 (71%)
Frame 179 (71%)
Frame 180 (72%)
Frame 181 (72%)
Frame 182 (72%)
Frame 183 (73%)
Frame 184 (73%)
Frame 185 (74%)
Frame 186 (74%)
Frame 187 (74%)
Frame 188 (75%)
Frame 189 (75%)
Frame 190 (76%)
Frame 191 (76%)
Frame 192 (76%)
Frame 193 (77%)
Frame 194 (77%)
Frame 195 (78%)
Frame 196 (78%)
Frame 197 (78%)
Frame 198 (79%)
Frame 199 (79%)
Frame 200 (80%)
Frame 201 (80%)
Frame 202 (80%)
Frame 203 (81%)
Frame 204 (81%)
Frame 205 (82%)
Frame 206 (82%)
Frame 207 (82%)
Frame 208 (83%)
Frame 209 (83%)
Frame 210 (84%)
Frame 211 (84%)
Frame 212 (84%)
Frame 213 (85%)
Frame 214 (85%)
Frame 215 (86%)
Frame 216 (86%)
Frame 217 (86%)
Frame 218 (87%)
Frame 219 (87%)
Frame 220 (88%)
Frame 221 (88%)
Frame 222 (88%)
Frame 223 (89%)
Frame 224 (89%)
Frame 225 (90%)
Frame 226 (90%)
Frame 227 (90%)
Frame 228 (91%)
Frame 229 (91%)
Frame 230 (92%)
Frame 231 (92%)
Frame 232 (92%)
Frame 233 (93%)
Frame 234 (93%)
Frame 235 (94%)
Frame 236 (94%)
Frame 237 (94%)
Frame 238 (95%)
Frame 239 (95%)
Frame 240 (96%)
Frame 241 (96%)
Frame 242 (96%)
Frame 243 (97%)
Frame 244 (97%)
Frame 245 (98%)
Frame 246 (98%)
Frame 247 (98%)
Frame 248 (99%)
Frame 249 (99%)
Frame 250 (100%)
## Finalizing encoding... done!
{% endhighlight %}



{% highlight r %}
b_gif
{% endhighlight %}

![center](/figs/2019-10-08-tt-powerlifting/unnamed-chunk-10-1.gif)

Next, combine them using `magick` (thanks to [this
post](https://github.com/thomasp85/gganimate/wiki/Animation-Composition)):


{% highlight r %}
library(magick)
a_mgif <- image_read(a_gif)
b_mgif <- image_read(b_gif)

new_gif <- image_append(c(a_mgif[1], b_mgif[1]), stack = TRUE)
for(i in 2:250){
  combined <- image_append(c(a_mgif[i], b_mgif[i]), stack = TRUE)
  new_gif <- c(new_gif, combined)
}

new_gif
{% endhighlight %}

![center](/figs/2019-10-08-tt-powerlifting/unnamed-chunk-11-1.gif)

