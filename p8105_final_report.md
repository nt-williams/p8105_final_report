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

Motivation
----------

Organ donation is a public health crisis in New York where there are currently [9,449 people waiting](https://optn.transplant.hrsa.gov/data/view-data-reports/state-data/#) for an organ transplant. Due to the scarcity of donated organs, many people do not survive long enough to receive a donated organ, and those that do may have waited years for an organ. At Columbia University Medical center, over 2016 and 2017, the [mortality rate](https://www.srtr.org/transplant-centers/ny-presbyterian-hospitalcolumbia-univ-medical-center-nycp/?organ=kidney&recipientType=adult&donorType=) for the kidney donation waitlist was 7.2 percent.

Like every state, NY has a first person consent policy for deceased organ donation. If a person on the donor registry list dies and is a candidate for organ donation, his or her family cannot prevent an organ procurement agency from gifting the organs to a person on an organ waitlist. Although uniform enforcement of this law is questionable, first person consent is believed to be important in supplying organ donations.

New York, however, has the [lowest proportion](https://www.donatelife.net/wp-content/uploads/2018/09/DLA_AnnualReport.pdf) of eligible people enrolled on the donor registry. The plot below demonstrates how low New York falls in comparisons with nearby states. We were interested in exploring how NY donor designation share (percent of eligible people enrolled) has changed over the last decade in response to a series of policies. We attempted to explore the association between county-level demographic parameters and registration rate, trying to account for differences in registration share among counties in New York State. We were also interested in comparing country donation rates, to see if a targeted approach to increasing NY's organ donation rate may be warranted.

``` r
read_csv("data/state_donor_share.csv") %>% 
  janitor::clean_names() %>% 
  rename(state = "x1") %>% 
  ggplot(aes(x = year, y = donor_designation_share, color = state)) +
  geom_point() + 
  geom_smooth() + 
  labs(title = "State percentage enrolled comparison", 
       x = "Year", 
       y = "Eligible population enrolled (%)") + 
  viridis::scale_color_viridis(discrete = TRUE)
```

<img src="p8105_final_report_files/figure-markdown_github/state comparisons-1.png" width="65%" />

Related work
------------

[Here is a website on California's organ donor registration data.](https://www.mapbox.com/narratives/a-mission-to-heal-donor-network-west-organ-donation/?fbclid=IwAR2zqeRYa5B_j3Z88zwSIpT29kqMNzJmk49HS3Erz1e0HYLurSRwMcAnBSA) This website created an interactive map that allows users to see the donor registration proportions in different counties in California. It has another interactive map directly next to the donor registration percentage map that allows users to see the racial make up of each county. This allows the viewer to visualize how different racial compositions correspond to different registration percentages. We wanted to apply the interactive map idea to New York and use a regression approach to a look at how race and additional factors may correlate with donor registration percentage.

Initial questions
-----------------

### Trends and Policies

Over the last 6 years, the state government has passed into law several policies to increase the donor designation share. In 2012 it passed Lauren's law, which requires the DMV drivers license application to force people to decide whether or not they will join the registry. In order to complete the application, applicants must actively mark that they will skip over the question. In 2016 the governor of NY passed a law that requires health insurance applications to asks people if they want to sign up for the organ registry.

Was there a statewide change in the growth of donor designation share after any of these laws were passed? If there was, was this change seen in all counties or only a subset?

Additionally, in 2017, the size of the eligible pool increased when the minimum age for joining the registry was lowered from eighteen to sixteen. We initially wondered if 16 and 17 year olds were more likely to register, given that they sign up for permits and licenses, and therefore, wondered if there would be an increase in the percentage of eligible people enrolling on the registry. However, while studying our data, we realized that the percent eligible population enrolled variable used the same denominator (the estimated number of people at least 18), regardless of the year, so our measure was not technically measuring donor designation share after 2017, but rather the ratio between the number people enrolled and the number of people over 18. After the 2016 law, we expected this measure to increase, since the 16 and 17 year old enrollees would add to the numerator but not denominator. Did this increase actually occur?

### County Characteristics

What are some demographic or health-related factors associated with registration rate? The factors we considered include population size, race, gender, age, education, and Medicare enrollment. Our initial thought was to visualize each variable's effect by simply plotting it against registration rate. However, given the small sample size (62 counties) and confounding effect among the factors, such one-to-one associations were not easy to tell by plots. Therefore, we then tried to fit linear models with the predictors of interest, so that each demographic variable can be adjusted for each other.

Using the regression model, we tried to investigate how factors are associated with the percentage of eligible people enrolled on the donor registry for each county. Are the association positive or negative? Most of the demographic variables are standardized to the percentage scale, so we can also examine the extent to which they affect registration share by looking at the coefficients.

Data
----

New York State organ donor data was pulled from New York's official public data [API](https://health.data.ny.gov/Health/Donate-Life-Organ-and-Tissue-Donor-Registry-Enroll/sqk8-8a2h/data?fbclid=IwAR0DjHPMd3up76Cq1bMH0z12f_vr4rLz5YwtMPcT2kc_a5nP-nVJlDEinNE).

``` r
# pulling data from api and doing initial clean
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

Splines were created in correspondence with three major policy changes designed to increase donor registration. In order to include time as a variable dates were converted to the number of days since the first data point in the database (September 1, 2008). Counties Cattauragus and St. Lawrence were removed for analysis due to what we believe to be incorrect data entry (i.e. the proportion over the entire time-frame did not change at all for either of these counties).

``` r
# creating splines
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

Extra county level data came from Area Health Resources Files (AHRF). AHRF data collects healthcare related information and demographics at the county level for every county in the United States. Specific variables were chosen from AHRF to be included in this analysis. AHRF data is provided as a SAS data-file with un-informative variable names for over 3000 variables; as such, extracting specific variables presented a challenge. Variables labels in SAS are recorded as a variable attribute in R. The following code was used to extract the specific variables used in our analyses.

``` r
# pulling in ahrf data
library(haven)

ahrf <- 
  read_sas("data/ahrf.sas7bdat")

ahrf_selected <- read_csv("data/ahrf_selected_variables.csv")

# function to pull sas labels
check_label <- function(x) {
  if (attr(ahrf[[x]], "label") %in% ahrf_selected$label) {
    return(x)
  }
}

# check_label("f1529815")

keep_var <- list()

# pulling all sas labels in our list
for (i in names(ahrf)) {
 keep_var[i] <- check_label(i)
}

keep_var <- names(keep_var)

ahrf <- ahrf %>% 
  select(keep_var)

keep_names <- list()

# pulling labels to be variable names
for (i in 1:ncol(ahrf)) {
  keep_names[[i]] <- attr(ahrf[[i]], "label")
}

# converting variable names to dataframe
for (i in 1:length(keep_names)) {
  names(keep_names)[i] <- "label"
}

keep_names <- data.frame(keep_names) %>% 
  gather(keep, label, label:label.17) %>% 
  select(label)

# setting new variable names
data.table::setnames(ahrf, old = keep_var, new = keep_names$label)

ahrf <- ahrf %>% 
  janitor::clean_names() %>% 
  filter(state_name == "New York")

ahrf %>% 
  write_csv("data/ahrf_select_data.csv")
```

To explore county-level factors that may affect registration rate, we combined the organ donor data and AHRF data-set for further analysis. We used the organ registration rate in September 2018 only. Steps were followed as below to clean the data. Some original AHRF variables were divided by total population to generate percentages that are easier to interpret in a linear model.

``` r
# limiting data to september 2018
organ_2018_09 <- 
  organ %>%
  filter(date == '2018-09-01') %>% 
  rename(percent_enrolled = eligible_population_enrolled)

demo_ny <-  
  read.csv("data/ahrf_select_data.csv") %>% 
  as.tibble() %>% 
  rename(county = county_name)

# cleaning and combining ahrf and organ data
combined_df <- 
  inner_join(by = 'county', organ_2018_09, demo_ny) %>%
  select(county, opo, population_18_estimate, registry_enrollments, percent_enrolled,
         standardzd_per_capita_medcr_cost_fee_for_service_2015:percent_educ_hlth_care_soc_asst_2011_15) %>% 
  mutate(opo = fct_relevel(opo, "New York Organ Donor Network")) %>% 
  mutate(
    pop_total_2015 = (pop_total_female_2015 + pop_total_male_2015),
    percent_male_2015 = round(100 * (pop_total_male_2015 / pop_total_2015), 2),
    percent_white_2015 = (pop_white_female_2015 + pop_white_male_2015) / pop_total_2015,
    percent_white_2015 = round(100 * percent_white_2015, 2),
    percent_black_2015 = (pop_black_african_amer_female_2015 + 
                            pop_black_african_amer_male_2015) / pop_total_2015,
    percent_black_2015 = round(100 * percent_black_2015, 2),
    percent_asian_2015 = (pop_asian_female_2015 + pop_asian_male_2015) / pop_total_2015,
    percent_asian_2015 = round(100 * percent_asian_2015, 2),
    percent_medicare_enrollment_2015 = round((medicare_enrollment_aged_tot_2015 * 100 / pop_total_2015), 2)) %>% 
  select(-(pop_total_male_2015:pop_asian_female_2015), 
                -population_estimate_2016, 
                -medicare_enrollment_aged_tot_2015) %>% 
  select(county, percent_enrolled, everything())

# editing names for regression models
regression_df <- 
  combined_df %>% 
  select(-county) %>% 
  rename(pop_18 = population_18_estimate, 
         reg_enroll = registry_enrollments, 
         medcr_fee_2015 = standardzd_per_capita_medcr_cost_fee_for_service_2015, 
         percent_hs_diploma = percent_persons_25_w_hs_diploma_2011_15,
         percent_college = percent_persons_25_w_4_yrs_college_2011_15,
         percent_soc_asst = percent_educ_hlth_care_soc_asst_2011_15, 
         pop_2015 = pop_total_2015, 
         percent_medcr_2015 = percent_medicare_enrollment_2015)
```

To create the plot comparing registration rates of nearby states to registration rate in NY we manually input data from a csv file from annual reports created by an organization called [Donate Life America](https://www.donatelife.net/wp-content/uploads/2018/09/DLA_AnnualReport.pdf).

Exploratory analyses
--------------------

``` r
# finding what counties have the lowest enrollments
lowest_3_counties <- organ %>% 
  filter(date == as.Date("2018-09-01")) %>% 
  filter(county != "TOTAL NYS") %>%
  arrange(eligible_population_enrolled) %>%
  top_n(n = -3, eligible_population_enrolled)

# finding county with top enrollment
top_county <-  organ %>% 
  filter(date == as.Date("2018-09-01")) %>% 
  filter(county != "TOTAL NYS") %>%
  arrange(eligible_population_enrolled) %>%
  top_n(n = 1, eligible_population_enrolled)

# plotting population vs. percent enrolled
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

<img src="p8105_final_report_files/figure-markdown_github/population vs percent-1.png" width="65%" />

This plot shows that there is a negative correlation between the number of eligible potential donors and the percentage of eligible people enrolled. Three large counties in NYC, Bronx, Queens, and Kings have the lowest percentage. Smaller counties have percentages that align with those of neighboring states, such as New Jersey and Connecticut.

``` r
# choropleth of organ registration in ny
organ_tidy <- organ %>% 
  separate(location, c("lat", "long"), sep = ",") %>% 
  mutate(long = str_replace(long, "\\)", ""),
         long = as.numeric(long)) %>% 
  mutate(lat = str_replace(lat, "\\(", ""),
         lat = as.numeric(lat))

# getting map data
ny <- map_data("state") %>%
  filter(region == "new york")

ny_county <- map_data("county") %>% 
  filter(region == "new york") %>% 
  as.tibble() %>% 
  rename(county = subregion)

# limiting to september 2018
organ_df <-  
  organ_tidy %>% 
  filter(date == "2018-09-01") %>% 
  select(eligible_population_enrolled, county, opo) %>% 
  mutate(county = tolower(county)) %>% 
  mutate(county = recode(county, 'cattauragus' = 'cattaraugus'))

# joining organ and map data
ny_county_combined <- 
  full_join(organ_df, ny_county, by = 'county') %>% 
  filter(is.na(opo) == FALSE) %>% 
  mutate(opo = fct_recode(opo, 
                          NYODN = "New York Organ Donor Network", 
                          CDTNY = "Center for Donation and Transplant in New York", 
                          FLDRN = "Finger Lakes Donor Recovery Network"),
         opo = fct_relevel(opo, 'NYODN'))

# plotting map
ny_map <- ggplot() + 
  geom_polygon(data = ny_county_combined, 
               aes(x = long, y = lat, group = group, fill = eligible_population_enrolled)) +
  geom_path(data = ny_county_combined, 
            aes(x = long, y = lat, group = group), 
            color = "white", size = 0.1) +
  labs(title = "Proportion of eligible adults enrolled as organ donors in NY", 
       x = 'Longitude', 
       y = 'Latitude', 
       fill = 'Enrollment (%)') +
  coord_map() +
  viridis::scale_fill_viridis(option = "magma", direction = -1) + 
  theme_void()

ny_map
```

<img src="p8105_final_report_files/figure-markdown_github/choropleth-1.png" width="70%" />

The above chloropleth shows how different counties in 2018 range in the proportion of eligible adults enrolled as organ donors. Interestingly, the counties that make up and surround New York City have among the lowest enrollment rates.

``` r
# plotting number of people enrolled over time
organ %>% 
filter(county != "TOTAL NYS") %>%
ggplot(aes(x = date, y = registry_enrollments, color = county)) +
  geom_line() +
  labs(x = "Date", 
       y = "Persons enrolled", 
       title = "Registry enrollment counts across time") + 
  viridis::scale_color_viridis(discrete = TRUE) + 
  theme(legend.position = "none")
```

<img src="p8105_final_report_files/figure-markdown_github/county county enrollment-1.png" width="65%" />

The registry enrollments had a generally increasing trend across the years. However, note a sudden change in registry enrollments on 2009-06-01 and 2017-10-1. Curiously, the enrollment even declined for some counties on 2017-10-1, which indicates a flaw in the data.

``` r
opo_violin <-  
  combined_df %>% 
  mutate(opo = fct_recode(opo, 
                          NYODN = "New York Organ Donor Network", 
                          CDTNY = "Center for Donation and Transplant in New York", 
                          FLDRN = "Finger Lakes Donor Recovery Network")) %>%
  ggplot(aes(x = opo, y = percent_enrolled)) +
    geom_violin() +
    labs(x = "OPO", 
         y = "Eligible Population Enrolled (%)") +
  theme(legend.position = 'none')

opo_map <- 
  ggplot() + 
  geom_polygon(data = ny_county_combined, 
               aes(x = long, y = lat, group = group, fill = opo)) +
  viridis::scale_fill_viridis(discrete = TRUE, 
                              option = "cividis") + 
  geom_path(data = ny_county_combined, 
            aes(x = long, y = lat, group = group), 
            color = "white", size = 0.1) +
  labs(x = 'Longitude', 
       y = 'Latitude') +
  coord_map() +
  theme_void() +
  theme(legend.position = "bottom", 
        legend.title = element_blank())

wrap_elements(opo_violin + opo_map) + 
  ggtitle("OPO distribution in New York State")
```

<img src="p8105_final_report_files/figure-markdown_github/opo distribution-1.png" width="65%" />

Organ procurement organizations (OPO) are non-profit organizations that are responsible for the evaluation and procurement of deceased-donor organs for organ transplantation. There are 4 OPOs in the New York State, each with its own designated service area. From the plot above, we found that the distribution of registration rates in counties designated to the 4 OPOs are different. The New York Organ Donor Network has the lowest distribution of Percent Enrollment, and was thus used as the reference group for analyses.

Change of Slope Visualization: 2012
-----------------------------------

``` r
library(ggrepel)

diff_diff_organ_2012 <- 
  organ %>%
  filter(date == as.Date("2014-07-01") | date == as.Date("2012-07-01") | date == as.Date("2010-07-01")) %>%
  select(county, date, eligible_population_enrolled) %>%
  group_by(county) %>%
  spread(value = eligible_population_enrolled, key = date) %>%
  janitor::clean_names() %>%
  mutate(slope_2012_2010 = (x2012_07_01 - x2010_07_01)/as.integer(as.Date("2012-07-01") - as.Date("2010-07-01")),
         slope_2014_2012 = (x2014_07_01 - x2012_07_01)/as.integer(as.Date("2014-07-01") - as.Date("2012-07-01")),
         slope_diff = slope_2014_2012 - slope_2012_2010) %>%
  ungroup() %>%
  mutate(county = fct_reorder(county, slope_diff)) %>%
  mutate(is_total_nys = ifelse(county == "TOTAL NYS", 1, 0))

diff_diff_organ_ny_2012 <- filter(diff_diff_organ_2012, county %in% c("TOTAL NYS"))

diff_diff_organ_2012 %>% 
  ggplot(aes(x = county, y = slope_diff)) +
    geom_point() +
    theme_bw() +
    theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
    theme(legend.position = "none") +
    geom_text_repel(data = diff_diff_organ_ny_2012, 
                    aes(x = county, 
                        y = slope_diff, 
                        label = county), nudge_x = -2.5, nudge_y = .001) +
    labs(title = "Slope Change After 2012 Policy",
         y = "Slope Change")
```

<img src="p8105_final_report_files/figure-markdown_github/visualization 2012-1.png" width="65%" />

We calculated the average donor designation share rate of increase for each county and for the total New York State using three points: a point before Lauren's Law was passed in 2012, a point at the time of the law's passage, and a point two years after. The slope was calculated by dividing the change in donor designation share and dividing by the number of days between the two points used. Almost every county saw negative changes in slope.

Change of Slope Visualization: 2017
-----------------------------------

``` r
diff_diff_organ_2017 <- organ %>%
  filter(date == as.Date("2018-09-01") | date == as.Date("2017-02-01") | date == as.Date("2015-07-01")) %>%
  select(county, date, eligible_population_enrolled) %>%
  group_by(county) %>%
  spread(value = eligible_population_enrolled, key = date) %>%
  janitor::clean_names() %>%
  mutate(slope_2017_2015 = (x2017_02_01 - x2015_07_01)/as.integer(as.Date("2017-02-01") - as.Date("2015-07-01")),
         slope_2018_2017 = (x2018_09_01 - x2017_02_01)/as.integer(as.Date("2018-09-01") - as.Date("2017-02-01")),
         slope_diff = slope_2018_2017 - slope_2017_2015) %>%
  ungroup() %>%
  mutate(county = fct_reorder(county, slope_diff)) %>%
  mutate(is_total_nys = ifelse(county == "TOTAL NYS", 1, 0))

diff_diff_organ_ny_2017 = filter(diff_diff_organ_2017, county %in% c("TOTAL NYS"))

diff_diff_organ_2017 %>% 
  ggplot(aes(x = county, y = slope_diff)) +
    geom_point() +
    theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
    theme(legend.position = "none") +
    geom_text_repel(data = diff_diff_organ_ny_2017, 
                    aes(x = county, 
                        y = slope_diff, 
                        label = county), nudge_x = -2.5, nudge_y = .01) +
    labs(title = "Slope Change After 2017 Policy",
         y = "Slope Change")
```

<img src="p8105_final_report_files/figure-markdown_github/visualization 2017-1.png" width="65%" />

We used one and a half year increments rather than two year increments because our data does not reach two years after the passage of the law. Almost all the slopes increase after 2017. NY county, one of the larger counties in NY, is one of the 7 counties that has a decreasing average slope. The next section will test significance of these changes in slope.

Formal analyses
---------------

### Effect of major policy changes

Formal analyses of the effect of major policy changes were conducted using mixed-effects linear models with a random intercept; counties were treated as the clustering variable. The outcome was analyzed was the proportion of eligible adults registered with one of four organ donor organizations. Four successive models were created: a model only analyzing a linear effect of time; a model including a term for a spline corresponding with a 2012 policy requiring that DMV drivers license application force people to decide whether or not they will join the registry; a model including a term for a spline corresponding with a 2016 policy that added organ donor registry to the New York state health insurance marketplace application; a model including a term for a spline corresponding with a 2017 policy that lowered the age from eligible enrollees from 18 to 16 years old; and a model that included all splines. All models were adjusted for the specific organization used in each county; models were compared using likelihood ratio tests.

``` r
# plotting raw data over time
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

<img src="p8105_final_report_files/figure-markdown_github/no model-1.png" width="65%" />

The above plot shows the increasing trend of organ donor enrollment over time. The dashed lines represent the beginning of the 3 policies we analyzed in this project.

The first model built only analyzed the effect of the 2012 policy. Main effects of time and the 2012 spline were found. Overall, every one year change resulted in a 2.88 percentage point increase in registry enrollment (95% CI: 2.84, 2.93) up to the beginning of the 2012 policy. However, after the 2012 policy went into effect the rate of increase in enrollment dropped 2.56 percentage points.

``` r
# mixed model with 2012 spline
sp_12_model <- organ_sp %>% 
  lmer(eligible_population_enrolled ~ total_days + year_sp_2012 + (1 | county), data = .)

# model stats
sp_12_model %>% 
  broom::tidy() %>% 
  filter(term %in% c("total_days", "year_sp_2012")) %>% 
  mutate(estimate_year = estimate * 365, 
         se_year = std.error * 365, 
         ci_low = estimate_year - 1.96*se_year, 
         ci_upper = estimate_year + 1.96*se_year) %>% 
  mutate(term = ifelse(term == "total_days", "Time (year)", term), 
         term = ifelse(term == "year_sp_2012", "2012 spline", term)) %>% 
  select(term, estimate_year, se_year, ci_low, ci_upper) %>% 
  rename("Term" = term, 
         "Coefficient" = estimate_year, 
         "SE" = se_year, 
         "Lower bound" = ci_low, 
         "Upper bound" = ci_upper) %>% 
  knitr::kable(digits = 3) %>% 
  kable_styling(full_width = F, bootstrap_options = c("hover", "striped"))
```

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
``` r
# plotting 2012 spline model
add_predictions(organ_sp, sp_12_model) %>% 
  ggplot(aes(x = total_days, y = pred, color = opo, type = county)) + 
  geom_line() +
  geom_vline(xintercept = 1491, linetype = "dashed") +
  annotate(geom = "text", x = 1720, y = 5, label = "2012") +
  viridis::scale_color_viridis(discrete = TRUE) + 
  labs(title = "Proportion of eligible New Yorkers enrolled as organ donors", 
       subtitle = "Modeling 2012 spline", 
       x = "Time as total days", 
       y = "Eligible population enrolled (%)") + 
  theme(legend.position = "bottom", 
        legend.title = element_blank()) + 
  guides(color = guide_legend(ncol = 2))
```

<img src="p8105_final_report_files/figure-markdown_github/spline 2012-1.png" width="65%" />

Modeling the effect of the 2016 policy, we once again found main effects of time and the 2016 spline. On average, every year increase resulted in an estimated 2.56 percentage point increase in eligible adult enrollment until the 2016 policy went into effect. After the 2016 policy, the estimated rate at which enrollment was changing increased by 0.79 percentage points (95% CI: 0.69, 0.89).

``` r
# mixed model 2016 spline
sp_16_model <- organ_sp %>% 
  lmer(eligible_population_enrolled ~ total_days + year_sp_2016 + opo + (1 | county), data = .) 

# 2016 model stats
sp_16_model %>% 
  broom::tidy() %>% 
  filter(term %in% c("total_days", "year_sp_2016")) %>% 
  mutate(estimate_year = estimate * 365, 
         se_year = std.error * 365, 
         ci_low = estimate_year - 1.96*se_year, 
         ci_upper = estimate_year + 1.96*se_year) %>% 
  mutate(term = ifelse(term == "total_days", "Time (year)", term), 
         term = ifelse(term == "year_sp_2016", "2016 spline", term)) %>% 
  select(term, estimate_year, se_year, ci_low, ci_upper) %>% 
  rename("Term" = term, 
         "Coefficient" = estimate_year, 
         "SE" = se_year, 
         "Lower bound" = ci_low, 
         "Upper bound" = ci_upper) %>% 
  knitr::kable(digits = 3) %>% 
  kable_styling(full_width = F, bootstrap_options = c("hover", "striped"))
```

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
``` r
# plotting 2016 model# 2016 model stats
add_predictions(organ_sp, sp_16_model) %>% 
  ggplot(aes(x = total_days, y = pred, color = opo, type = county)) + 
  geom_line() + 
  viridis::scale_color_viridis(discrete = TRUE) + 
  labs(title = "Proportion of eligible New Yorkers enrolled as organ donors", 
       subtitle = "Modeling 2016 spline", 
       x = "Time as total days", 
       y = "Eligible population enrolled (%)") +
  geom_vline(xintercept = 2799, linetype = "dashed") +
  annotate(geom = "text", x = 2600, y = 5, label = "2016") +
  theme(legend.position = "bottom", 
        legend.title = element_blank()) + 
  guides(color = guide_legend(ncol = 2))
```

<img src="p8105_final_report_files/figure-markdown_github/spline 2016-1.png" width="65%" />

Similarly to the 2016 spline, modeling the 2017 policy we found main effects of time and the 2017 spline. On average, every year increase corresponded with a 2.56 percentage point increase in proportion of eligible individuals enrolled. However, after the 2017 spline, the rate of change in enrollment increased by 1.7 percentage points (95% CI: 1.55, 1.85) to 4.26 percentage points.

``` r
# 2017 spline mixed model
sp_17_model <- organ_sp %>% 
  lmer(eligible_population_enrolled ~ total_days + year_sp_2017 + opo + (1 | county), data = ., 
       REML = FALSE)

# 2017 model stats
sp_17_model %>% 
  broom::tidy() %>% 
  filter(term %in% c("total_days", "year_sp_2017")) %>% 
  mutate(estimate_year = estimate * 365, 
         se_year = std.error * 365, 
         ci_low = estimate_year - 1.96*se_year, 
         ci_upper = estimate_year + 1.96*se_year) %>% 
  mutate(term = ifelse(term == "total_days", "Time (year)", term), 
         term = ifelse(term == "year_sp_2017", "2017 spline", term)) %>% 
  select(term, estimate_year, se_year, ci_low, ci_upper) %>% 
  rename("Term" = term, 
         "Coefficient" = estimate_year, 
         "SE" = se_year, 
         "Lower bound" = ci_low, 
         "Upper bound" = ci_upper) %>% 
  knitr::kable(digits = 3) %>% 
  kable_styling(full_width = F, bootstrap_options = c("hover", "striped"))
```

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
``` r
# plotting model
add_predictions(organ_sp, sp_17_model) %>% 
  ggplot(aes(x = total_days, y = pred, color = opo, type = county)) + 
  geom_line() +
  viridis::scale_color_viridis(discrete = TRUE) + 
  labs(title = "Proportion of eligible New Yorkers enrolled as organ donors", 
       subtitle = "Modeling 2017 spline", 
       x = "Time as total days", 
       y = "Eligible population enrolled (%)") + 
  geom_vline(xintercept = 3088, linetype = "dashed") + 
  annotate(geom = "text", x = 3300, y = 9, label = "2017") +
  theme(legend.position = "bottom", 
        legend.title = element_blank()) + 
  guides(color = guide_legend(ncol = 2))
```

<img src="p8105_final_report_files/figure-markdown_github/spline 2017-1.png" width="65%" />

The final model analyzed accounted for all three separate policy changes. Main effects were found for all three splines. This model found that after the 2012 policy was enacted, the estimated rate of increase in population enrollment decreased by .98 percentage points (95% CI: -1.06, -0.90). However, after the 2016 policy went into effect the rate of enrollment increased again by 0.33 percentage points (95% CI: 0.039, 0.627). Lastly, after the 2017 policy went into effect the rate of enrollment increased again by 2.47 percentage points (95% CI: 2.05, 2.88). Ultimately, after all three policies came into effect the estimated increase in organ donor enrollment is 4.9 percentage points per year (an estimated net effect of 1.82 percentage points).

``` r
# all splines mixed model
all_sp_model <- organ_sp %>% 
  lmer(eligible_population_enrolled ~ total_days + year_sp_2012 + year_sp_2016 + year_sp_2017 + opo + 
         (1 | county), data = ., REML = FALSE)

# all splines model stats
all_sp_model %>% 
  broom::tidy() %>% 
  filter(term %in% c("total_days", "year_sp_2012", "year_sp_2016", "year_sp_2017")) %>% 
  mutate(estimate_year = estimate * 365, 
         se_year = std.error * 365, 
         ci_low = estimate_year - 1.96*se_year, 
         ci_upper = estimate_year + 1.96*se_year) %>% 
  mutate(term = ifelse(term == "total_days", "Time (year)", term), 
         term = ifelse(term == "year_sp_2012", "2012 spline", term), 
         term = ifelse(term == "year_sp_2016", "2016 spline", term), 
         term = ifelse(term == "year_sp_2017", "2017 spline", term)) %>% 
  select(term, estimate_year, se_year, ci_low, ci_upper) %>% 
  rename("Term" = term, 
         "Coefficient" = estimate_year, 
         "SE" = se_year, 
         "Lower bound" = ci_low, 
         "Upper bound" = ci_upper) %>% 
  knitr::kable(digits = 3) %>% 
  kable_styling(full_width = F, bootstrap_options = c("hover", "striped")) %>% 
  footnote(general = "Splines correspond with dates in which 3 major policy changes were inacted. \nEstimates are adjusted for organ donor organization")
```

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
``` r
# plotting all splines model
add_predictions(organ_sp, all_sp_model) %>% 
  ggplot(aes(x = total_days, y = pred, color = opo, type = county)) + 
  geom_line() +
  viridis::scale_color_viridis(discrete = TRUE) + 
  geom_vline(xintercept = 1491, linetype = "dashed") + 
  geom_vline(xintercept = 2799, linetype = "dashed") + 
  geom_vline(xintercept = 3088, linetype = "dashed") + 
  annotate(geom = "text", x = 1720, y = 5, label = "2012") +
  annotate(geom = "text", x = 2600, y = 5, label = "2016") +
  annotate(geom = "text", x = 3300, y = 9, label = "2017") +
  theme(legend.position = "bottom", 
        legend.title = element_blank()) + 
  guides(color = guide_legend(ncol = 2)) + 
  labs(title = "Proportion of eligible New Yorkers enrolled as organ donors", 
       subtitle = "Modeling all splines", 
       x = "Time as total days", 
       y = "Eligible population enrolled (%)")
```

<img src="p8105_final_report_files/figure-markdown_github/all splines plot-1.png" width="65%" />

### County-level factors affecting registration rate

The first model was built using criterion-based methods, searching for subsets of predictors with the optimal Cp and Adjusted R-squared.

``` r
subsets <- regsubsets(percent_enrolled ~ ., data = regression_df, force.in = NULL)
summary(subsets) 
```

As suggested by the function above (result not shown), the first model evaluated used OPO, percent of adults over 25 with a high school diploma, percent of adults over 25 with four years of college, percent of the county that is male, the percent of the county that is Asian, and the percent of the county enrolled in Medicare.

``` r
county_one <- regression_df %>% 
  lm(percent_enrolled ~ opo + percent_hs_diploma + percent_college + percent_male_2015 + percent_asian_2015 +
       percent_medcr_2015, data = .)

county_one %>% 
  broom::tidy() %>% 
  kable(digits = 3, 
        caption = "Summary of Model 1",
        col.names = c("Term", 
                      "Estimate", 
                      "Std. Error", 
                      "Statistic", 
                      "p-value"))
```

<table>
<caption>
Summary of Model 1
</caption>
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Estimate
</th>
<th style="text-align:right;">
Std. Error
</th>
<th style="text-align:right;">
Statistic
</th>
<th style="text-align:right;">
p-value
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
(Intercept)
</td>
<td style="text-align:right;">
-32.517
</td>
<td style="text-align:right;">
25.406
</td>
<td style="text-align:right;">
-1.280
</td>
<td style="text-align:right;">
0.206
</td>
</tr>
<tr>
<td style="text-align:left;">
opoCenter for Donation and Transplant in New York
</td>
<td style="text-align:right;">
7.479
</td>
<td style="text-align:right;">
2.152
</td>
<td style="text-align:right;">
3.476
</td>
<td style="text-align:right;">
0.001
</td>
</tr>
<tr>
<td style="text-align:left;">
opoFinger Lakes Donor Recovery Network
</td>
<td style="text-align:right;">
12.560
</td>
<td style="text-align:right;">
2.182
</td>
<td style="text-align:right;">
5.755
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
opoUNYTS
</td>
<td style="text-align:right;">
12.197
</td>
<td style="text-align:right;">
2.635
</td>
<td style="text-align:right;">
4.630
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_hs\_diploma
</td>
<td style="text-align:right;">
-0.475
</td>
<td style="text-align:right;">
0.222
</td>
<td style="text-align:right;">
-2.142
</td>
<td style="text-align:right;">
0.037
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_college
</td>
<td style="text-align:right;">
0.284
</td>
<td style="text-align:right;">
0.108
</td>
<td style="text-align:right;">
2.632
</td>
<td style="text-align:right;">
0.011
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_male\_2015
</td>
<td style="text-align:right;">
1.073
</td>
<td style="text-align:right;">
0.441
</td>
<td style="text-align:right;">
2.431
</td>
<td style="text-align:right;">
0.019
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_asian\_2015
</td>
<td style="text-align:right;">
-0.358
</td>
<td style="text-align:right;">
0.189
</td>
<td style="text-align:right;">
-1.890
</td>
<td style="text-align:right;">
0.064
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_medcr\_2015
</td>
<td style="text-align:right;">
0.747
</td>
<td style="text-align:right;">
0.248
</td>
<td style="text-align:right;">
3.009
</td>
<td style="text-align:right;">
0.004
</td>
</tr>
</tbody>
</table>
The second model was built based on our hypothesis that factors including OPO, Medicare enrollment, gender, ethnicity, and education status affect registration rate.

``` r
county_two <- regression_df %>% 
  lm(percent_enrolled ~ opo + percent_medcr_2015 + percent_male_2015 + percent_white_2015 + percent_hs_diploma +
       percent_college, data = .)  

county_two %>% 
  broom::tidy() %>% 
  kable(digits = 3, 
        caption = "Summary of Model 2",
        col.names = c("Term", 
                      "Estimate", 
                      "Std. Error", 
                      "Statistic", 
                      "p-value"))
```

<table>
<caption>
Summary of Model 2
</caption>
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Estimate
</th>
<th style="text-align:right;">
Std. Error
</th>
<th style="text-align:right;">
Statistic
</th>
<th style="text-align:right;">
p-value
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
(Intercept)
</td>
<td style="text-align:right;">
-41.291
</td>
<td style="text-align:right;">
26.793
</td>
<td style="text-align:right;">
-1.541
</td>
<td style="text-align:right;">
0.129
</td>
</tr>
<tr>
<td style="text-align:left;">
opoCenter for Donation and Transplant in New York
</td>
<td style="text-align:right;">
7.977
</td>
<td style="text-align:right;">
2.189
</td>
<td style="text-align:right;">
3.644
</td>
<td style="text-align:right;">
0.001
</td>
</tr>
<tr>
<td style="text-align:left;">
opoFinger Lakes Donor Recovery Network
</td>
<td style="text-align:right;">
12.960
</td>
<td style="text-align:right;">
2.234
</td>
<td style="text-align:right;">
5.802
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
opoUNYTS
</td>
<td style="text-align:right;">
12.830
</td>
<td style="text-align:right;">
2.692
</td>
<td style="text-align:right;">
4.766
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_medcr\_2015
</td>
<td style="text-align:right;">
0.725
</td>
<td style="text-align:right;">
0.276
</td>
<td style="text-align:right;">
2.631
</td>
<td style="text-align:right;">
0.011
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_male\_2015
</td>
<td style="text-align:right;">
1.060
</td>
<td style="text-align:right;">
0.464
</td>
<td style="text-align:right;">
2.283
</td>
<td style="text-align:right;">
0.027
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_white\_2015
</td>
<td style="text-align:right;">
0.095
</td>
<td style="text-align:right;">
0.109
</td>
<td style="text-align:right;">
0.876
</td>
<td style="text-align:right;">
0.385
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_hs\_diploma
</td>
<td style="text-align:right;">
-0.418
</td>
<td style="text-align:right;">
0.306
</td>
<td style="text-align:right;">
-1.365
</td>
<td style="text-align:right;">
0.178
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_college
</td>
<td style="text-align:right;">
0.258
</td>
<td style="text-align:right;">
0.123
</td>
<td style="text-align:right;">
2.102
</td>
<td style="text-align:right;">
0.041
</td>
</tr>
</tbody>
</table>
The final model was adopted from model 2, adding interaction term between ethnicity and high school education.

``` r
county_final <- regression_df %>%  
  lm(percent_enrolled ~ opo + percent_medcr_2015 + percent_male_2015 + percent_white_2015*percent_hs_diploma +
       percent_college, data = .) 

county_final %>%
  broom::tidy() %>% 
  kable(digits = 3, 
        caption = "Summary of the final model",
        col.names = c("Term", 
                      "Estimate", 
                      "Std. Error", 
                      "Statistic", 
                      "p-value"))
```

<table>
<caption>
Summary of the final model
</caption>
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Estimate
</th>
<th style="text-align:right;">
Std. Error
</th>
<th style="text-align:right;">
Statistic
</th>
<th style="text-align:right;">
p-value
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
(Intercept)
</td>
<td style="text-align:right;">
-118.033
</td>
<td style="text-align:right;">
29.318
</td>
<td style="text-align:right;">
-4.026
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
opoCenter for Donation and Transplant in New York
</td>
<td style="text-align:right;">
8.206
</td>
<td style="text-align:right;">
1.893
</td>
<td style="text-align:right;">
4.334
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
opoFinger Lakes Donor Recovery Network
</td>
<td style="text-align:right;">
13.194
</td>
<td style="text-align:right;">
1.932
</td>
<td style="text-align:right;">
6.830
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
opoUNYTS
</td>
<td style="text-align:right;">
13.706
</td>
<td style="text-align:right;">
2.336
</td>
<td style="text-align:right;">
5.867
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_medcr\_2015
</td>
<td style="text-align:right;">
0.732
</td>
<td style="text-align:right;">
0.238
</td>
<td style="text-align:right;">
3.072
</td>
<td style="text-align:right;">
0.003
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_male\_2015
</td>
<td style="text-align:right;">
1.620
</td>
<td style="text-align:right;">
0.422
</td>
<td style="text-align:right;">
3.835
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_white\_2015
</td>
<td style="text-align:right;">
0.661
</td>
<td style="text-align:right;">
0.162
</td>
<td style="text-align:right;">
4.067
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_hs\_diploma
</td>
<td style="text-align:right;">
2.156
</td>
<td style="text-align:right;">
0.659
</td>
<td style="text-align:right;">
3.274
</td>
<td style="text-align:right;">
0.002
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_college
</td>
<td style="text-align:right;">
0.381
</td>
<td style="text-align:right;">
0.110
</td>
<td style="text-align:right;">
3.463
</td>
<td style="text-align:right;">
0.001
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_white\_2015:percent\_hs\_diploma
</td>
<td style="text-align:right;">
-0.034
</td>
<td style="text-align:right;">
0.008
</td>
<td style="text-align:right;">
-4.270
</td>
<td style="text-align:right;">
0.000
</td>
</tr>
</tbody>
</table>
Model comparisons can be found in the table below.

| Model       | Adjusted r-squared | AIC    |
|-------------|--------------------|--------|
| Model 1     | 0.76               | 345.46 |
| Model 2     | 0.74               | 348.62 |
| Final model | 0.81               | 331.97 |

We found that type of OPO that conducted the registration affects registration rate. Percentage of Medicare enrollment, percentage of persons finishing high school education and finishing 4-year college education are positively associated with registration rate. This finding agrees with our hypothesis. Also, the gender `male` and race `white` are positively associated with registration rate.

#### Model validation

We used both cross validation and bootstrapping to select candidate models. Bootstrapping was used to assess the variance of the coefficients of the 3 models, and cross validation was used to evaluation the predicative ability of the three models.

``` r
set.seed(5)

bootstrap_df <-  
  regression_df %>% 
  modelr::bootstrap(n = 100) %>% 
  mutate(
    model_1 = purrr::map(strap, ~lm(percent_enrolled ~ opo + percent_hs_diploma + percent_college +
                                      percent_male_2015 + percent_asian_2015 + percent_medcr_2015, data = .x)),
    model_2 = purrr::map(strap, ~lm(percent_enrolled ~ opo + percent_medcr_2015 + percent_male_2015 +
                                      percent_white_2015 + percent_hs_diploma + percent_college, data = .x)),
    model_3 = purrr::map(strap, ~lm(percent_enrolled ~ opo + percent_medcr_2015 + percent_male_2015 +
                                      percent_white_2015*percent_hs_diploma + percent_college, data = .x)))

bootstrap_df %>% 
  mutate(
    result_1 = purrr::map(model_1, broom::tidy),
    result_2 = purrr::map(model_2, broom::tidy),
    result_3 = purrr::map(model_3, broom::tidy)) %>% 
  dplyr::select(-strap, -(model_1:model_3))  %>% 
  gather(key = model, value = result, result_1:result_3) %>% 
  unnest() %>% 
  dplyr::filter(term != "(Intercept)") %>% 
  group_by(model, term) %>% 
  summarize(boot_se = sd(estimate)) %>% 
  spread(key = model, value = boot_se)  %>% 
  kable(digits = 3, 
        caption = "Standard deviation of the coefficients for 3 models",
        col.names = c("Term", 
                      "Model 1", 
                      "Model 2", 
                      "Final Model"))
```

<table>
<caption>
Standard deviation of the coefficients for 3 models
</caption>
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Model 1
</th>
<th style="text-align:right;">
Model 2
</th>
<th style="text-align:right;">
Final Model
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
opoCenter for Donation and Transplant in New York
</td>
<td style="text-align:right;">
2.461
</td>
<td style="text-align:right;">
2.402
</td>
<td style="text-align:right;">
1.901
</td>
</tr>
<tr>
<td style="text-align:left;">
opoFinger Lakes Donor Recovery Network
</td>
<td style="text-align:right;">
2.409
</td>
<td style="text-align:right;">
2.385
</td>
<td style="text-align:right;">
2.000
</td>
</tr>
<tr>
<td style="text-align:left;">
opoUNYTS
</td>
<td style="text-align:right;">
2.602
</td>
<td style="text-align:right;">
2.634
</td>
<td style="text-align:right;">
2.378
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_asian\_2015
</td>
<td style="text-align:right;">
0.388
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_college
</td>
<td style="text-align:right;">
0.142
</td>
<td style="text-align:right;">
0.156
</td>
<td style="text-align:right;">
0.150
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_hs\_diploma
</td>
<td style="text-align:right;">
0.336
</td>
<td style="text-align:right;">
0.384
</td>
<td style="text-align:right;">
1.275
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_male\_2015
</td>
<td style="text-align:right;">
0.454
</td>
<td style="text-align:right;">
0.471
</td>
<td style="text-align:right;">
0.407
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_medcr\_2015
</td>
<td style="text-align:right;">
0.300
</td>
<td style="text-align:right;">
0.320
</td>
<td style="text-align:right;">
0.271
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_white\_2015
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.127
</td>
<td style="text-align:right;">
0.211
</td>
</tr>
<tr>
<td style="text-align:left;">
percent\_white\_2015:percent\_hs\_diploma
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
0.015
</td>
</tr>
</tbody>
</table>
The final model had the smallest SD for types of OPO, percent of male, and percent of people having Medicare. However, the coefficient for percent of people having high school diploma was the highest for the final model, so this predictor may be less reliable than others.

``` r
cv_df <- crossv_mc(regression_df, 100)  

cv_df <-  
  cv_df %>% 
  mutate(
    model_1 = purrr::map(train, ~lm(percent_enrolled ~ opo + percent_hs_diploma + percent_college +
                                      percent_male_2015 + percent_asian_2015 + percent_medcr_2015, data = .x)),
    model_2 = purrr::map(train, ~lm(percent_enrolled ~ opo + percent_medcr_2015 + percent_male_2015 +
                                      percent_white_2015 + percent_hs_diploma + percent_college, data = .x)),
    model_3 = purrr::map(train, ~lm(percent_enrolled ~ opo + percent_medcr_2015 + percent_male_2015 +
                                      percent_white_2015*percent_hs_diploma + percent_college, data = .x))) %>% 
  mutate(rmse_1 = purrr::map2_dbl(model_1, test, ~rmse(model = .x, data = .y)),
         rmse_2 = purrr::map2_dbl(model_2, test, ~rmse(model = .x, data = .y)),
         rmse_3 = purrr::map2_dbl(model_3, test, ~rmse(model = .x, data = .y))) %>% 
  select(-train, -test, -(model_1:model_3)) %>% 
  gather(key = model, value = rmse, rmse_1:rmse_3)

cv_df %>% 
  ggplot(aes(x = model, y = rmse)) + 
  geom_violin() + 
  labs(title = "Distribution of RMSE for candidate models", 
       x = "Model", 
       y = "RMSE") + 
  scale_x_discrete(labels = c("rmse_1" = "Model 1",
                              "rmse_2" = "Model 2",
                              "rmse_3" = "Final model"))
```

<img src="p8105_final_report_files/figure-markdown_github/cross validation-1.png" width="65%" />

The distribution of RMSE for the final model was the lowest compared to the other two candidate models. So our final model had the best predictive performance.

Discussion
----------

Using mixed-effects models we did find main effects of all three policies that were targeted at increasing organ donor enrollment. However, it is important to note that we cannot firmly draw any causal conclusions from the present analysis. While increases in organ donor enrollment did coincide with major policy changes in New York, we did not compare the trends of enrollment in New York with those of the greater United States or another state that did not enact any policy during the same time-frame and thus did not establish a counter-factual. We could not find such data and thus we cannot rule out that the results we observed simply reflect a general trend in the United States and not any policy taken by the state of New York. That being said, our models do show effects of policy changes.

While the policies were designed to increase enrollment, it appears the 2012 policy might have had an opposite effect. The 2012 policy mandated that when residents of New York obtained a new driver's license, they must either check a box to skip a question asking them if they want to enroll or must enroll in the registry. For future research about this policy, it would be helpful to look specifically at DMV registration rate (percentage of people applying for dmv identifications who enroll in the registry), as this 2012 policy was targeted toward the DMV. There are different avenues for enrolling on the registry, so if the rate of DMV registration increased but the rate of registration in other avenues decreased independent of the law, this legislation technically had its intended effect.

In our regression model analyzing the effect of county characteristics on registration share, we found that the most effective predictor was the type of OPO. The New York Organ Donor Network (NYODN), which serves counties with large population (e.g. Queens, Kings, Bronx), had obviously the least organ donor share, while the other three OPOs had similarly higher average registration share. Considering the large population of the counties designated to NYODN, it is reasonable to put more effort to this area.

There are also other county-level demographic and health-related characteristics affecting registration rate. We found that the percent of people enrolled in Medicare, the percent of people that finished high school and college education, the percent of male and/or white people in the population are positively associated with registration share. This finding is interesting but not conclusive, because we do not have information about people already in the registry.

Many of the smaller counties in NYS already have the enrollment registration rates seen in surrounding states. However, the larger counties in the New York City area have rates that are at least 15 percentage points lower than those of surrounding states. It is great that people in small counties are willing to donate organs, but NY needs to target larger counties with smaller rates to take advantage of the number of potential organ donors these counties represent.

Our exploration of this data-set also showed that New York needs to keep better records. Unexplained, sudden simultaneous peaks and simultaneous falls in county organ registration rates are seen in our data-set, which means there could be late updating or mistake in the data-set. This brings limitations to our trends analysis.
