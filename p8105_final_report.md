Analyzing trends of organ donor enrollment in New York state
================

| Name          | Uni     |
|---------------|---------|
| Shuliang Deng | sd3258  |
| Nick Williams | ntw2117 |
| Sijia Yue     | sy2824  |
| Jack Yan      | xy2395  |
| Alina Levine  | al3851  |

[Click here for cleaned data used for analyses](https://www.dropbox.com/sh/grlugs9hrksvkqr/AAAwILSg5UyMrW1CD7dWmHPNa?dl=0)

Organ donation is a public health crisis in New York where there are currently 9,449 people waiting for an organ transplant. Due to the scarcity of donated organs, many people do not survive long enough receive a donated organ, and those that do may have wait years for an organ. At Columbia University Medical center, over 2016 and 2017, the mortality rate for the kidney donation waitlist was 7.2 percent. Like every state, NY has a first person consent policy for deceased organ donation. If a person on the donor registry list dies and is a candidate for organ donation, his or her family cannot prevent an organ procurement agency from gifting the organs to a person on an organ waitlist. Although uniform enforcement of this law is questionable, first person consent is believed to be important in supplying organ donations. New York, however, has the lowest proportion of eligible people enrolled on the donor registry. We were interested in exploring how NY donor designation share (percent of eligible people enrolled) has changed over the last decade and were interested in comparing country donation rates, to see if a targeted approach to increasing NY's organ donation rate may be warranted.

Initial questions
-----------------

### Trends and Policies

Over the last 6 years, the state government has passed into law several policies to increase the donor designation share. In 2012 it passed Lauren's law, which requires the DMV drivers license application to force people to decide whether or not they will join the registry. In order to complete the application, applicants must actively mark that they will skip over the question. In 2017 the govenor of NY passed a law that reuiquires health insurance applications to asks people if they want to sign up for the organ registry.

Additionally, in 2016, the size of the eligible pool increased when the minimum age for joining the registry was lowered from eighteen to sixteen.

Was there a statewide change in the growth of donor designation share after any of these laws were passed? If there was, was this change seen in all counties or only a subset?

### County Characteristics

What are some factors that predict county donor designation share?

Data
----

New York State organ donor data was pulled from New York's official public data [API](https://health.data.ny.gov/Health/Donate-Life-Organ-and-Tissue-Donor-Registry-Enroll/sqk8-8a2h/data?fbclid=IwAR0DjHPMd3up76Cq1bMH0z12f_vr4rLz5YwtMPcT2kc_a5nP-nVJlDEinNE).

``` r
organ <- GET("https://health.data.ny.gov/resource/km5a-7zrs.csv?$limit=10000") %>% 
  content("parsed") %>%
  janitor::clean_names() %>%
  filter(county != "Out of State" & county != "Unknown") %>%
  mutate(year = as.character(year)) %>%
  mutate(month = as.character(month)) %>%
  mutate(dummy_day = as.character("01")) %>%
  mutate(date = (str_c(year, month, dummy_day, sep = "-"))) %>%
  mutate(date = as.Date(date, "%Y-%m-%d")) %>% 
  rename(pop_2012 = "x2012_census_population")
```

Splines were created in correspondence with three major policy changes designed to increase donor registration. In order to include time as a variable dates were converted to the number of days since the first data point in the database (September 1, 2008).

``` r
organ_sp <- organ %>% 
  mutate(year_sp_2012 = (date > "2012-10-1") * (date - as.Date("2012-10-1")), 
         year_sp_2016 = (date > "2016-5-1") * (date - as.Date("2016-5-1")), 
         year_sp_2017 = (date > "2017-2-14") * (date - as.Date("2017-2-14")),
         total_days = date - as.Date("2008-09-01")) %>% 
  filter(!(county %in% c("TOTAL NYS", "Cattauragus", "St Lawrence"))) %>% 
  mutate(opo = as_factor(opo), 
         opo = fct_relevel(opo, 
                           "New York Organ Donor Network", 
                           "Center for Donation and Transplant in New York", 
                           "UNYTS", 
                           "Finger Lakes Donor Recovery Network"))
```

Extra county level data came from Area Health Resources Files (AHRF). AHRF data collects healthcare related information and demographics at the county level for every county in the United States. Specific variables were chosen from AHRF to be included in this analysis. AHRF data is provided as a SAS datafile with un-informative variable names for over 3000 variables; as such, extracting specific variables presented a challenge. Variables labels in SAS are recorded as a variable attribute in R. The following code was used to extract the specific variables used in our analyses.

``` r
library(haven)

ahrf <- 
  read_sas("data/ahrf.sas7bdat")

ahrf_selected <- read_csv("data/ahrf_selected_variables.csv")
  
check_label <- function(x) {
  if (attr(ahrf[[x]], "label") %in% ahrf_selected$label) {
    return(x)
  }
}

# check_label("f1529815")

keep_var <- list()

for (i in names(ahrf)) {
 keep_var[i] <- check_label(i)
}

keep_var <- names(keep_var)

ahrf <- ahrf %>% 
  select(keep_var)

keep_names <- list()

for (i in 1:ncol(ahrf)) {
  keep_names[[i]] <- attr(ahrf[[i]], "label")
}

for (i in 1:length(keep_names)) {
  names(keep_names)[i] <- "label"
}

keep_names <- data.frame(keep_names) %>% 
  gather(keep, label, label:label.17) %>% 
  select(label)

data.table::setnames(ahrf, old = keep_var, new = keep_names$label)

ahrf <- ahrf %>% 
  janitor::clean_names() %>% 
  filter(state_name == "New York")

ahrf %>% 
  write_csv("data/ahrf_select_data.csv")
```

Exploratory analyses
--------------------

``` r
lowest_3_counties = organ %>% 
  filter(date == as.Date("2018-09-01")) %>% 
  filter(county != "TOTAL NYS") %>%
  arrange(eligible_population_enrolled) %>%
  top_n(n = -3, eligible_population_enrolled)

top_county = organ %>% 
  filter(date == as.Date("2018-09-01")) %>% 
  filter(county != "TOTAL NYS") %>%
  arrange(eligible_population_enrolled) %>%
  top_n(n = 1, eligible_population_enrolled)

organ %>% 
  filter(date == as.Date("2018-09-01")) %>% 
  filter(county != "TOTAL NYS") %>%
  ggplot(aes(x = population_18_estimate, y = eligible_population_enrolled)) +
    geom_point() +
    labs(title = "Eligible Population vs Percent Enrolled by County",
         x = "Estimated Population 18 and Over",
         y = "Percent of Eligible Population Enrolled") +
    geom_text_repel(data = lowest_3_counties, 
                    aes(x = population_18_estimate, 
                        y = eligible_population_enrolled, 
                        label = county)) +
    geom_text_repel(data = top_county, 
                    aes(x = population_18_estimate, 
                        y = eligible_population_enrolled, 
                        label = county))
```

<img src="p8105_final_report_files/figure-markdown_github/population vs. percent-1.png" width="75%" />

This plot shows that there is a negative correlation between the number of eligible potential donors and the percentage of eligible people enrolled. Three large counties in NYC, Bronx, Queens, and Kings have the lowest percentage. Smaller counties have percentages that allign with those of neighboring states, such as New Jersey and Connecticut.

Formal analyses
---------------

Formal analyses of the effect of major policy changes were conducted using mixed-effects linear models with a random intercept; counties were treated as the clustering variable. The outcome was analyzed was the proportion of eligible adults registered with one of four organ donor organizations. Four successive models were created: a model only analyzing a linear effect of time; a model including a term for a spline corresponding with a 2012 policy INSERT WHAT THIS POLICY WAS; a model including a term for a spline corresponding with a 2016 policy INSERT WHAT THIS POLICY WAS; a model including a term for a spline corresponding with a 2017 policy that lowered the age from eligible enrollees from 18 to 16 years old; and a model that included all splines. All models were adjusted for the specific organization used in each county; models were compared using likelihood ratio tests.

``` r
organ_sp %>% 
  ggplot() +
  geom_line(aes(x = total_days, y = eligible_population_enrolled, color = opo, group = county), 
            alpha = 0.6) +
  viridis::scale_color_viridis(discrete = TRUE) + 
  labs(title = "Proportion of eligible New Yorkers enrolled as organ donors",
       subtitle = "Dashed lines indicate 3 policies to be anlayzed",
       x = "Time as total days",
       y = "Eligible population enrolled (%)") +
  geom_vline(xintercept = 1491, linetype = "dashed") + 
  geom_vline(xintercept = 2799, linetype = "dashed") + 
  geom_vline(xintercept = 3088, linetype = "dashed") +
  annotate(geom = "text", x = 1720, y = 53, label = "2012") +
  annotate(geom = "text", x = 2600, y = 5, label = "2016") +
  annotate(geom = "text", x = 3300, y = 5, label = "2017") +
  theme(legend.position = "bottom", 
        legend.title = element_blank()) + 
  guides(color = guide_legend(ncol = 2))
```

<img src="p8105_final_report_files/figure-markdown_github/no model-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
2.883
</td>
<td style="text-align:right;">
0.022
</td>
<td style="text-align:right;">
2.840
</td>
<td style="text-align:right;">
2.926
</td>
</tr>
<tr>
<td style="text-align:left;">
2012 spline
</td>
<td style="text-align:right;">
-0.326
</td>
<td style="text-align:right;">
0.032
</td>
<td style="text-align:right;">
-0.389
</td>
<td style="text-align:right;">
-0.262
</td>
</tr>
</tbody>
</table>
<img src="p8105_final_report_files/figure-markdown_github/spline 2012-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
2.563
</td>
<td style="text-align:right;">
0.01
</td>
<td style="text-align:right;">
2.542
</td>
<td style="text-align:right;">
2.583
</td>
</tr>
<tr>
<td style="text-align:left;">
2016 spline
</td>
<td style="text-align:right;">
0.789
</td>
<td style="text-align:right;">
0.05
</td>
<td style="text-align:right;">
0.692
</td>
<td style="text-align:right;">
0.886
</td>
</tr>
</tbody>
</table>
<img src="p8105_final_report_files/figure-markdown_github/spline 2016-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
2.561
</td>
<td style="text-align:right;">
0.009
</td>
<td style="text-align:right;">
2.543
</td>
<td style="text-align:right;">
2.579
</td>
</tr>
<tr>
<td style="text-align:left;">
2017 spline
</td>
<td style="text-align:right;">
1.700
</td>
<td style="text-align:right;">
0.078
</td>
<td style="text-align:right;">
1.548
</td>
<td style="text-align:right;">
1.852
</td>
</tr>
</tbody>
</table>
<img src="p8105_final_report_files/figure-markdown_github/spline 2017-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
3.088
</td>
<td style="text-align:right;">
0.022
</td>
<td style="text-align:right;">
3.045
</td>
<td style="text-align:right;">
3.131
</td>
</tr>
<tr>
<td style="text-align:left;">
2012 spline
</td>
<td style="text-align:right;">
-0.982
</td>
<td style="text-align:right;">
0.041
</td>
<td style="text-align:right;">
-1.063
</td>
<td style="text-align:right;">
-0.902
</td>
</tr>
<tr>
<td style="text-align:left;">
2016 spline
</td>
<td style="text-align:right;">
0.333
</td>
<td style="text-align:right;">
0.150
</td>
<td style="text-align:right;">
0.039
</td>
<td style="text-align:right;">
0.627
</td>
</tr>
<tr>
<td style="text-align:left;">
2017 spline
</td>
<td style="text-align:right;">
2.465
</td>
<td style="text-align:right;">
0.213
</td>
<td style="text-align:right;">
2.048
</td>
<td style="text-align:right;">
2.883
</td>
</tr>
</tbody>
<tfoot>
<tr>
<td style="padding: 0; border: 0;" colspan="100%">
<span style="font-style: italic;">Note: </span>
</td>
</tr>
<tr>
<td style="padding: 0; border: 0;" colspan="100%">
<sup></sup> Splines correspond with dates in which 3 major policy changes were inacted. <br>Estimates are adjusted for organ donor organization
</td>
</tr>
</tfoot>
</table>
<img src="p8105_final_report_files/figure-markdown_github/all splines-1.png" width="75%" />

Discussion
----------
