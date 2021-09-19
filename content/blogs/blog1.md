---
categories:
- ""
- ""
date: "2017-10-31T21:28:43-05:00"
description: ""
draft: false
image: pic10.jpg
keywords: ""
slug: project1
title: Exploring IMDB Ratings Between Spielberg and Burton
---

# IMDB ratings: Differences between directors

Recall the IMBD ratings data. I would like you to explore whether the mean IMDB rating for Steven Spielberg and Tim Burton are the same or not. I have already calculated the confidence intervals for the mean ratings of these two directors and as you can see they overlap.

```{r load-movies-data, fig.width=11, warning = FALSE}
movies <- read_csv(here::here("data", "movies.csv"))
glimpse(movies)

```

```{r, burton_spielberg confidence interval, fig.width=11, warning = FALSE}
burton_spielberg <- movies %>% 
  filter(director %in% c("Steven Spielberg", "Tim Burton")) %>% 
  group_by(director) %>% 
  summarise(mean=mean(rating),
            count=n(),
            t_critical = qt(0.975, count-1),
            se = sd(rating)/sqrt(count),
            margin_of_error = t_critical*se,
            lower_ci = mean - margin_of_error,
            higher_ci = mean + margin_of_error)
burton_spielberg
  
```

```{r, replicating the plot, fig.width=11, warning = FALSE}
plot <- ggplot(burton_spielberg, aes(x=mean, y=reorder(director, mean), colour = director))+
  geom_errorbar(width = 0.05, aes(xmin = lower_ci, xmax = higher_ci), size = 2)+
  geom_point(aes(x=mean), size = 5)+
  geom_rect(aes(xmin = max(lower_ci), xmax = min(higher_ci), ymin = Inf, ymax = Inf),
            fill="gray", colour="gray",alpha = 0.3)+
  theme_bw() +
  geom_text(aes(x=mean, label = round(mean, digits=2)),
            size = 8, vjust = 2, color = "black")+
  geom_text(aes(x=lower_ci, label=round(lower_ci,digits=2)),
            size = 6, vjust = 2, color = "black")+
  geom_text(aes(x=higher_ci, label=round(higher_ci,digits=2)),
            size = 6, vjust = 2, colour = "black")+
  labs(title = "Do Spielberg ad Burton have the same IMDB ratings?", 
       subtitle = "95% confidence intervals overlap", x="Mean IMDB Rating", y="")+
  theme(plot.title=element_text(size=12,face="bold"),
        axis.text = element_text(size=10),
        legend.position = "none")
plot
  
```

*Null Hypothesis H0: Average Rating for Spielberg - Average Rating for Burton = 0* *Alternative Hypothesis H1: Difference in Means not equal to 0.*

```{r, fig.width=11, warning = FALSE}
testrating <- movies %>% 
  filter(director %in% c("Steven Spielberg", "Tim Burton")) %>% 
  select(director, rating)

t.test(testrating$rating ~ testrating$director)
```

*We test the difference between means of the two directors, and the t-stat value equals 3 while the p-value is 0.01. The confidence interval is [0.16,1.13] and does not contain the value 0. The t-statistics value is greater than 2, p-value \< 5% and 0 lies outside the confidence interval, therefore we reject H0.*

```{r, fig.width=11, warning = FALSE}
obs_diff1 <- testrating %>%
  specify(rating ~ director) %>%
  calculate(stat = "diff in means", order = c("Steven Spielberg", "Tim Burton"))
obs_diff1
```

```{r, fig.width=11, warning = FALSE}
null_dist1 <- testrating %>%
  # specify variables
  specify(rating ~ director) %>%
  
  # assume independence, i.e, there is no difference
  hypothesize(null = "independence") %>%
  
  # generate 1000 reps, of type "permute"
  generate(reps = 1000, type = "permute") %>%
  
  # calculate statistic of difference, namely "diff in means"
  calculate(stat = "diff in means", order = c("Steven Spielberg", "Tim Burton"))

null_dist1

```

```{r, fig.width=11, warning = FALSE}
ggplot(data = null_dist1, aes(x = stat)) +
  geom_histogram()+
  theme_bw()

```

```{r, fig.width=11, warning = FALSE}
obs_stat1 <- obs_diff1 %>% pull() %>% round(2)
obs_stat1
```

```{r, fig.width=11, warning = FALSE}
null_dist1 %>% visualize() +
  shade_p_value(obs_stat = obs_diff1, direction = "two-sided")+
  theme_bw()

null_dist1 %>%
  get_p_value(obs_stat = obs_diff1, direction = "two_sided")