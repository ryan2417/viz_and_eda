ggplot\_2
================

``` r
library(tidyverse)
library(viridisLite)
library(ggridges)
library(patchwork)

knitr::opts_chunk$set(
  warning = FALSE,
  message = FALSE,
  fig.width = 6,
  fig.height = 6,
  out.width = "90%"
)
theme_set(theme_minimal() + theme(legend.position = "bottom"))

options(
  ggplot2.continuous.colour = "viridis",
  ggplot2.continuous.fill = "viridis"
)

scale_colour_discrete = scale_colour_viridis_d
scale_fill_discrete = scale_fill_viridis_d
```

``` r
weather_df <- 
  rnoaa::meteo_pull_monitors(
    c("USW00094728", "USC00519397", "USS0023B17S"),
    var = c("PRCP", "TMIN", "TMAX"), 
    date_min = "2017-01-01",
    date_max = "2017-12-31") %>%
  mutate(
    name = recode(
      id, 
      USW00094728 = "CentralPark_NY", 
      USC00519397 = "Waikiki_HA",
      USS0023B17S = "Waterhole_WA"),
    tmin = tmin / 10,
    tmax = tmax / 10) %>%
  select(name, id, everything())
```

## Start with a familiar one

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax, color = name)) +
  geom_point(alpha = .3) +
  labs(
    title = "Temperature at three stations",
    x = "Minimum daily temp(C)",
    y = "Maximum daily temp(C)",
    caption = "Data from rnoaa package with three stations"
  )
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-3-1.png" width="90%" />

## Scales

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax, color = name)) +
  geom_point(alpha = .3) +
  labs(
    title = "Temperature at three stations",
    x = "Minimum daily temp(C)",
    y = "Maximum daily temp(C)",
    caption = "Data from rnoaa package with three stations"
  ) +
  scale_x_continuous(
    breaks = c(-15, 0, 15),
    labels = c("-15C", "0C", "15C")
  ) +
  scale_y_continuous(
    trans = "sqrt",
    position = "right"
  )
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-4-1.png" width="90%" />

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax, color = name)) +
  geom_point(alpha = .3) +
  labs(
    title = "Temperature at three stations",
    x = "Minimum daily temp(C)",
    y = "Maximum daily temp(C)",
    caption = "Data from rnoaa package with three stations"
  ) +
   scale_color_hue(
    name = "Location"
  ) +
  scale_color_viridis_d()
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-5-1.png" width="90%" />
color scale `viridis`

## Themes

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax, color = name)) +
  geom_point(alpha = .3) +
  labs(
    title = "Temperature at three stations",
    x = "Minimum daily temp(C)",
    y = "Maximum daily temp(C)",
    caption = "Data from rnoaa package with three stations"
  ) +
  scale_color_hue(
    name = "Location"
  ) +
  scale_color_viridis_d() +
  theme_bw() +
  theme(legend.position = "bottom")
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-6-1.png" width="90%" />

### `theme` functions??? orders matter

``` r
weather_df %>% 
  ggplot(aes(x = tmin, y = tmax, color = name)) +
  geom_point(alpha = .3) +
  labs(
    title = "Temperature at three stations",
    x = "Minimum daily temp(C)",
    y = "Maximum daily temp(C)",
    caption = "Data from rnoaa package with three stations"
  ) +
  scale_color_hue(
    name = "Location"
  ) +
  scale_color_viridis_d() +
  theme_minimal() +
  theme(legend.position = "bottom")
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-7-1.png" width="90%" />

## `data` in geoms

``` r
central_park <-
  weather_df %>% 
  filter(name == "CentralPark_NY")

waikiki <- 
  weather_df %>% 
  filter(name == "Waikiki_HA")

waikiki %>% 
  ggplot(aes(x = date, y = tmax, color = name)) +
  geom_point() +
  geom_line(data = central_park)
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-8-1.png" width="90%" />

## `patchwork`

``` r
gg_tmax_tmin <-  
  weather_df %>% 
  ggplot(aes(x = tmin, y = tmax, color = name)) +
  geom_point(alpha = .3) +
  theme(legend.position = "none")

gg_prcp_dens <-
  weather_df %>% 
  filter(prcp > 0) %>% 
  ggplot(aes(x = prcp, fill = name)) +
  geom_density(alpha = .3) +
  theme(legend.position = "none")

gg_tmax_date <- 
  weather_df %>% 
  ggplot(aes(x = date, y = tmax, color = name)) +
  geom_point() +
  geom_smooth() +
  theme(legend.position = "bottom")

(gg_tmax_tmin + gg_prcp_dens)
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-9-1.png" width="90%" />

## data manipulation

quick example on factors

``` r
weather_df %>% 
  mutate(
    name = fct_reorder(name, tmax)
  ) %>% 
  ggplot(aes(x = name, y = tmax)) +
  geom_boxplot()
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-10-1.png" width="90%" />

What about tmax and tmin

``` r
weather_df %>% 
  pivot_longer(
    tmax:tmin,
    names_to = "obs",
    values_to = "temperature"
  ) %>% 
  ggplot(aes(x = temperature, fill = obs)) +
  geom_density(alpha = .3) +
  facet_grid(. ~ name)
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-11-1.png" width="90%" />

``` r
pulse_df <-
  haven::read_sas("data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    values_to = "bdi",
    names_prefix = "bdi_score_"
  ) %>% 
  mutate(visit = recode(visit, "bl" = "00m"))

pulse_df %>% 
  ggplot(aes(x = visit, y = bdi)) +
  geom_boxplot()
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-12-1.png" width="90%" />

``` r
pulse_df %>% 
  ggplot(aes(x = visit, y = bdi)) +
  geom_point() +
  geom_line(aes(group = id))
```

<img src="viz_part2_files/figure-gfm/unnamed-chunk-12-2.png" width="90%" />
