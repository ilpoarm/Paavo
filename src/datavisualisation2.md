---
title: "datavisualisation2"
output: github_document
# This part is copied from frcbs github folder
---


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


```{r}
library(tidyverse)
library(lubridate)
```

## Mean hemoglobin and low Hb deferral rates  

```{r}

lifetime_donation_data<- 
  ilposdata %>% 
    filter(donat_phleb == ("K"))  %>% 
   mutate(Year = year(dateonly))


lifetime_donation_data %>% 
  group_by(Year, gender) %>% 
  summarize(mean_Hb = mean(Hb, na.rm = TRUE),
            nb_Hb_deferrals = sum(as.numeric(Hb_deferral) - 1)) %>% 
  gather(key = key, value = value, -Year, -gender) %>% 
  ggplot(aes(x = Year, y = value)) +
  geom_point() +
  geom_line() +
  facet_grid(key ~gender, scales = "free")
```








## Distribution of donation frequency across donor

```{r}
lifetime_donation_data %>% 
  group_by(Year, gender, donor) %>% 
  summarize(nb_donations = n()) %>% 
  # gather(key = key, value = value, -Year, -sex) %>% 
  ggplot(aes(x = as.factor(Year), y = nb_donations)) +
  geom_count() +
  # geom_line() +
  facet_grid(gender ~., scales = "free")

```

```{r}
cbPalette <- c("#999999", "#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7")

lifetime_donation_data %>% 
  inner_join(nb_donations_per_year_data, by = c("donor", "Year")) %>% 
  mutate(group = case_when(gender == "Women" & age < 40 ~ "U40 Women",
                           gender == "Women" & age >= 40 ~ "40+ Women",
                           TRUE ~"Men")) %>% 
  rename(yearly_donations = nb_donations) %>% 
  group_by(group, Year, yearly_donations, donor) %>% 
  ungroup() %>% 
  group_by(group, Year, yearly_donations) %>% 
  summarise(nb_donors = n()) %>% 
  ungroup() %>% 
  mutate(group = ordered(group, levels = c("U40 Women", "40+ Women", "Men"))) %>% 
  ggplot(aes(x = Year, y = nb_donors, color = as.factor(yearly_donations),  group = as.factor(yearly_donations))) +
  geom_point() + 
  geom_line() +
  scale_y_log10(breaks =c(100, 500, 1000, 2500,  5000, 10000)) +
  scale_color_manual(values=cbPalette) +
  facet_grid(group ~., scales = "free")

```

```{r}
donations_nb_data <-
  lifetime_donation_data %>% 
    inner_join(nb_donations_per_year_data, by = c("donor", "Year")) %>% 
    mutate(group = case_when(gender == "Women" & age < 40 ~ "U40 Women",
                             gender == "Women" & age >= 40 ~ "40+ Women",
                             TRUE ~"Men")) %>% 
    rename(yearly_donations = nb_donations) %>% 
    group_by(group, Year, yearly_donations, donor) %>% 
    ungroup() %>% 
    group_by(group, Year, yearly_donations) %>% 
    summarise(nb_donors = n()) %>% 
    ungroup() %>% 
    mutate(group = ordered(group, levels = c("U40 Women", "40+ Women", "Men")))
```




```{r}
ylab = c(0,50,100)
  donations_nb_data %>% 
  mutate(total_donations = yearly_donations*nb_donors) %>% 
  ggplot(aes(x = as.character(Year), y = total_donations, fill = as.character(yearly_donations))) +
  geom_col() +
  facet_grid(group ~.) +
  # scale_y_continuous(labels = scales::unit_format("k", 1e-3)) +
   scale_y_continuous(labels = paste0(ylab, "K"),
                     breaks = 1000 * ylab)
```



```{r}
  donations_nb_data %>% 
  mutate(total_donations = yearly_donations*nb_donors) %>% 
  ggplot(aes(x = as.character(Year), y = total_donations, 
             color = as.character(yearly_donations),
             group = as.character(yearly_donations))) +
    geom_point() +
    geom_line() +
    scale_color_manual(values=cbPalette) +
    facet_grid(group ~., scales = "free") +
    theme_bw() +
    xlab("Year") +
    ylab("Total number of donations") +
    guides(color = guide_legend(title = "Nb of yearly \n donations")) +
    theme(text = element_text(size =18)) 
    # scale_color_colorblind()
```

```{r}
donations_nb_data %>% 
  mutate(total_donations = yearly_donations*nb_donors) %>% 
  group_by(Year) %>% 
  mutate(perc_donations = total_donations/sum(total_donations)) %>% 
    ggplot(aes(x = as.character(Year), y = perc_donations, fill = as.character(yearly_donations))) +
  geom_col(position = position_dodge()) +
  facet_grid(group ~.) +
  theme_bw() +
  theme(legend.position = "bottom") +
  guides(fill = guide_legend(title = "Yearly donations"),
         nrow = 1)
```
# missing donations overall
```{r}
donations_nb_data %>% 
  mutate(total_donations = yearly_donations*nb_donors,
         total_donation_sim = ifelse(group == "U40 Women" & yearly_donations > 2,
                                     nb_donors*2,
                                     nb_donors*yearly_donations)) %>% 
  group_by(Year) %>% 
  summarise(total_donations = sum(total_donations),
            total_donation_sim = sum(total_donation_sim),
            lost_donations = total_donations - total_donation_sim) %>% 
  # gather(key = key, value = nb_donations,  -Year) %>% 
  ggplot(aes(x = as.factor(Year), y = lost_donations)) +
  geom_col(alpha = 0.5) +
  theme_bw()+
  xlab("Year") +
  ylab("Missing donations") +
  theme(text = element_text(size = 18))
```
