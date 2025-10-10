COVID-19 Data Analysis
================

**Author: Jacob Anderson**

``` r
set.seed(1)
```

``` r
### Import Packages: arcgisbinding, sf, ggridges, ggplot2, reshape2

library(arcgisbinding)
library(sf)
library(ggridges)
library(ggplot2)
library(reshape2)
arc.check_product()
```

    ## product: ArcGIS Pro (13.5.0.57366)
    ## license: Advanced
    ## version: 1.0.1.311

``` r
### Using the arc.open() function to open the file geodatabase and printing the contents for inspection

getwd()
```

    ## [1] "C:/Users/ja090/Desktop/Files/JHU/SU25_STATS/Testing_COVID/testing_covid_data/Scripts"

``` r
data_dir <- "C:/Users/ja090/Desktop/Files/JHU/SU25_STATS/Testing_COVID/testing_covid_data/Data"

gis_data <- arc.open(file.path(data_dir, "qa02.gdb"))

print(gis_data)
```

    ## dataset_type    : Container
    ## path            : C:/Users/ja090/Desktop/Files/JHU/SU25_STATS/Testing_COVID/testing_covid_data/Data/qa02.gdb 
    ## children        : FeatureClass[1]
    ## FeatureClass    : USCounties_Cases

``` r
### Get the metadata for the Feature Class
### Metadata:
### Dataset type: ArcGIS Feature Class
### Path: The relative path to the Feature Class
### Fields: List of available fields in the Feature Class. 86 total fields are included. Common attributes for ArcGIS data types such as OBJECTID, Shape, Shape_Length, and Shape_Area are included. Also, a diverse collection of string, double, and integer data types are provided in the Feature Class.
### Extent: Bounding box including xmin, ymin, xmax, and ymax encapsulating the total extent for this Feature Class.
### Geometry type: Polygon feature class. This is representing the county level administrative boundaries of the United States.
### Projection: WGS 1984 Web Mercator Auxiliary Sphere projection and WKID code (3857)
### Printing the metadata

us_counties <- arc.open(file.path(data_dir, "qa02.gdb", "USCounties_Cases"))

print(us_counties)
```

    ## dataset_type    : FeatureClass
    ## path            : C:/Users/ja090/Desktop/Files/JHU/SU25_STATS/Testing_COVID/testing_covid_data/Data/qa02.gdb/USCounties_Cases 
    ## fields          : OBJECTID, Shape, Countyname, ST_Name, ST_Abbr, ST_ID, FIPS, FatalityRa, Confirmedb, DeathsbyPo, PCTPOVALL_, Unemployme, 
    ## fields          : Med_HH_Inc, State_Fata, DateChecke, EM_type, EM_date, EM_notes, url, Thumbnail, Confirmed, Deaths, Age_85, Age_80_84, 
    ## fields          : Age_75_79, Age_70_74, Age_65_69, Beds_Licen, Beds_Staff, Beds_ICU, Ventilator, POP_ESTIMA, POVALL_201, Unemployed, 
    ## fields          : Median_Hou, Recovered, Active, State_Conf, State_Deat, State_Reco, State_Test, AgedPop, NewCases, NewDeaths, TotalPop, 
    ## fields          : NonHispWhP, BlackPop, AmIndop, AsianPop, PacIslPop, OtherPop, TwoMorPop, HispPop, Wh_Alone, Bk_Alone, AI_Alone, As_Alone, 
    ## fields          : NH_Alone, SO_Alone, Two_More, Not_Hisp, Age_Less15, Age_15_24, Age_25_34, Age_Over75, Agetotal, NonHisp, Age_35_64, 
    ## fields          : Age_65_74, Day_1, Day_2, Day_3, Day_4, Day_5, Day_6, Day_7, Day_8, Day_9, Day_10, Day_11, Day_12, Day_13, Day_14, 
    ## fields          : Majority_Race, Shape_Length, Shape_Area
    ## extent          : xmin=-19942590, ymin=2144547, xmax=-7452848, ymax=11536823
    ## geometry type   : Polygon
    ## WKT             : PROJCS["WGS_1984_Web_Mercator_Auxiliary_Sphere",GEOGCS["GCS_...
    ## WKID            : 3857

``` r
### Creating an initial data frame
### Converting the new data frame to an R spatial data frame with the arc.data2sf() function

us_counties_df <- arc.select(us_counties)

us_counties_sf <- arc.data2sf(us_counties_df)

## Using the head() function to inspect the first six rows of this new spatial data frame

head(us_counties_sf)
```

    ## Simple feature collection with 6 features and 83 fields
    ## Geometry type: GEOMETRY
    ## Dimension:     XY
    ## Bounding box:  xmin: -9799326 ymin: 3532299 xmax: -9467592 ymax: 4063832
    ## Projected CRS: WGS 84 / Pseudo-Mercator
    ##   OBJECTID Countyname ST_Name ST_Abbr ST_ID  FIPS FatalityRa Confirmedb DeathsbyPo PCTPOVALL_ Unemployme Med_HH_Inc State_Fata
    ## 1        1    Autauga Alabama      AL    01 01001  1.4851485    3269.73   48.56028       13.8        3.6   118.9591   1.615103
    ## 2        2    Baldwin Alabama      AL    01 01003  0.8763228    2774.03   24.30947        9.8        3.6   115.4508   1.615103
    ## 3        3    Barbour Alabama      AL    01 01005  0.7600434    3701.62   28.13392       30.9        5.2    68.9280   1.615103
    ## 4        4       Bibb Alabama      AL    01 01007  1.4749263    3026.79   44.64286       21.8        4.0    92.3478   1.615103
    ## 5        5     Blount Alabama      AL    01 01009  0.9063444    2861.34   25.93361       13.2        3.5   101.0645   1.615103
    ## 6        6    Bullock Alabama      AL    01 01011  2.4469821    6046.56  147.95818       42.5        4.7    58.6736   1.615103
    ##            DateChecke                           EM_type              EM_date                                     EM_notes
    ## 1 10/04/2020 01:04:07 Govt Ordered Community Quarantine 4/3/2020 10:40:51 PM AL Governor issued a Shelter In Place order.
    ## 2 10/04/2020 01:04:07 Govt Ordered Community Quarantine 4/3/2020 10:41:19 PM AL Governor issued a Shelter In Place order.
    ## 3 10/04/2020 01:04:07 Govt Ordered Community Quarantine 4/3/2020 10:43:21 PM AL Governor issued a Shelter In Place order.
    ## 4 10/04/2020 01:04:07 Govt Ordered Community Quarantine 4/3/2020 10:40:51 PM AL Governor issued a Shelter In Place order.
    ## 5 10/04/2020 01:04:07 Govt Ordered Community Quarantine 4/3/2020 10:40:51 PM AL Governor issued a Shelter In Place order.
    ## 6 10/04/2020 01:04:07 Govt Ordered Community Quarantine 4/3/2020 10:42:44 PM AL Governor issued a Shelter In Place order.
    ##                                                     url                                                                    Thumbnail
    ## 1 https://bao.arcgis.com/covid-19/jhu/county/01001.html https://coronavirus.jhu.edu/static/media/dashboard_infographic_thumbnail.png
    ## 2 https://bao.arcgis.com/covid-19/jhu/county/01003.html https://coronavirus.jhu.edu/static/media/dashboard_infographic_thumbnail.png
    ## 3 https://bao.arcgis.com/covid-19/jhu/county/01005.html https://coronavirus.jhu.edu/static/media/dashboard_infographic_thumbnail.png
    ## 4 https://bao.arcgis.com/covid-19/jhu/county/01007.html https://coronavirus.jhu.edu/static/media/dashboard_infographic_thumbnail.png
    ## 5 https://bao.arcgis.com/covid-19/jhu/county/01009.html https://coronavirus.jhu.edu/static/media/dashboard_infographic_thumbnail.png
    ## 6 https://bao.arcgis.com/covid-19/jhu/county/01011.html https://coronavirus.jhu.edu/static/media/dashboard_infographic_thumbnail.png
    ##   Confirmed Deaths Age_85 Age_80_84 Age_75_79 Age_70_74 Age_65_69 Beds_Licen Beds_Staff Beds_ICU Ventilator POP_ESTIMA POVALL_201 Unemployed
    ## 1      1818     27    815      1026      1498      2440      2271         85         55        6          2      55601       7587        942
    ## 2      6048     53   3949      4792      7373     11410     13141        386        362       51          8     218022      21069       3393
    ## 3       921      7    422       551       841      1305      1515         74         30        5          2      24881       6788        433
    ## 4       678     10    427       488       624       842      1280         35         25        4          1      22400       4400        344
    ## 5      1655     15    866      1459      1776      2802      3330         25         25        6          2      57840       7527        878
    ## 6       613     15    175       182       248       436       575         61         30        5          1      10138       3610        224
    ##   Median_Hou Recovered Active State_Conf State_Deat State_Reco State_Test AgedPop NewCases NewDeaths TotalPop NonHispWhP BlackPop AmIndop
    ## 1      59338         0      0     158380       2558          0    1174699    8050       13         0    55200      41412    10475     159
    ## 2      57588         0      0     158380       2558          0    1174699   40665       24         0   208107     172768    19529    1398
    ## 3      34382         0      0     158380       2558          0    1174699    4634       19         0    25782      11898    12199      63
    ## 4      46064         0      0     158380       2558          0    1174699    3661        3         0    22527      16801     4974       8
    ## 5      50412         0      0     158380       2558          0    1174699   10233       13         0    57645      50232      820     124
    ## 6      29267         0      0     158380       2558          0    1174699    1616        1         0    10352       2228     7893     122
    ##   AsianPop PacIslPop OtherPop TwoMorPop HispPop Wh_Alone Bk_Alone AI_Alone As_Alone NH_Alone SO_Alone Two_More Not_Hisp Age_Less15 Age_15_24
    ## 1      568         5       41      1012    1528    42437    10565      159      568       32      409     1030    53672      10842      7192
    ## 2     1668         9      410      2972    9353   179526    19764     1522     1680        9     2034     3572   198754      37621     23497
    ## 3       85         1       86       344    1106    12216    12266       72       96        1      778      353    24676       4517      3092
    ## 4       37         0        0       160     547    17268     5018        8       37        0        9      187    21980       3742      3005
    ## 5      198        18      174       818    5261    55054      862      141      198       18      437      935    52384      11112      6906
    ## 6       56         0        2         0      51     2276     7893      122       56        0        5        0    10301       1770      1478
    ##   Age_25_34 Age_Over75 Agetotal NonHisp Age_35_64 Age_65_74 Day_1 Day_2 Day_3 Day_4 Day_5 Day_6 Day_7 Day_8 Day_9 Day_10 Day_11 Day_12 Day_13
    ## 1      7064       3339    55200    1528     22052      4711    13     7     7     4     2    12     9     7    19     23      1     23      1
    ## 2     23326      16114   208107    9353     82998     24551    24    27   357    34    18    62    49    21   291     24     17     37     26
    ## 3      3675       1814    25782    1106      9864      2820    19     4     2    10     0     1     3     9    16      6      3     10      3
    ## 4      3075       1539    22527     547      9044      2122     3     3     8     6     1     1     2     2    10      4      3     -1      4
    ## 5      6786       4101    57645    5261     22608      6132    13     8     5     8     3     1     6     3    14     14      7     13      9
    ## 6      1120        605    10352      51      4368      1011     1     0     2     3     0     1     2     5     1      1      4      2      4
    ##   Day_14             Majority_Race                           geom
    ## 1     17        Non-Hispanic White POLYGON ((-9619464 3856528,...
    ## 2     14        Non-Hispanic White MULTIPOLYGON (((-9772603 36...
    ## 3      5 Black or African American POLYGON ((-9490665 3781395,...
    ## 4      4        Non-Hispanic White POLYGON ((-9687784 3928063,...
    ## 5      9        Non-Hispanic White POLYGON ((-9623021 4062359,...
    ## 6      2 Black or African American POLYGON ((-9559430 3798723,...

``` r
### Create a Ridge Plot
### Utilizing the '$' operator to access the data frame directly and manipulate the specific columns in the spatial data frame
### Utilizing ggplot and ggridges to compare the distribution for the main demographic groups
### Setting the 'x' as the confirmed cases per 100,000 residents, and the 'y' for each main demographic group to assess the impact of COVID-19 across the US population

us_counties_sf$cases_per_100k <- (us_counties_sf$Confirmed / us_counties_sf$TotalPop * 100000)

ggplot(us_counties_sf, aes(x = cases_per_100k,
                           y = Majority_Race,
                           fill = Majority_Race)) +
  ggridges::geom_density_ridges(alpha = 0.5, scale = 0.7) +
  theme_minimal() +
  labs(title = "COVID-19 Cases",
       x = "Cases per 100k",
       y = "Majority Group") +
  theme(legend.position = "none")
```

    ## Picking joint bandwidth of 487

![](covid_analysis_files/figure-gfm/Task%204%20-%20Ridge%20Plot%20for%20County%20Confirmed%20divided%20by%20County%20Population%20per%20100,000%20vs%20Majority%20Race-1.png)<!-- -->

``` r
### Compare whether the Confirmed Case distributions for Asian, Black, and/or Non-Hispanic White are similar
### Compare Confirmed Case for Asian and Black
### Filtering the data for confirmed cases for each demographic group
### Utilizing the '$' operator to access specific records in the data frame and the '==' operator in a query to filter to each respective demographic group

cases_asian_pop <- us_counties_sf$cases_per_100k[us_counties_sf$Majority_Race == 'Asian']

cases_black_pop <- us_counties_sf$cases_per_100k[us_counties_sf$Majority_Race == 'Black or African American']

cases_nhwhite_pop <- us_counties_sf$cases_per_100k[us_counties_sf$Majority_Race == 'Non-Hispanic White']

### Creating a variable to be stored comparing the mean values for the confirmed cases of the Asian and Black demographics
### Printing the result from t-test to evaluate the statistics

ttest_asianpop_blackpop <- t.test(cases_asian_pop, 
                                  cases_black_pop, 
                                  alternative = "two.sided",
                                  var.equal = F)
print(ttest_asianpop_blackpop)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  cases_asian_pop and cases_black_pop
    ## t = -11.231, df = 6.7002, p-value = 1.366e-05
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -4082.495 -2651.673
    ## sample estimates:
    ## mean of x mean of y 
    ##  502.4016 3869.4854

``` r
### Creating a second variable to be stored comparing the mean values for the confirmed cases of the Asian and Non-Hispanic White demographics
### Printing the result from t-test to evaluate the statistics

ttest_asianpop_nhwhitepop <- t.test(cases_asian_pop,
                                    cases_nhwhite_pop,
                                    alternative = "two.sided",
                                    var.equal = F)
print(ttest_asianpop_nhwhitepop)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  cases_asian_pop and cases_nhwhite_pop
    ## t = -5.0791, df = 4.0745, p-value = 0.006744
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -2073.0991  -614.4874
    ## sample estimates:
    ## mean of x mean of y 
    ##  502.4016 1846.1948

``` r
### Creating a variable to be stored comparing the mean values for the confirmed cases of the Black and Non-Hispanic White demographics
### Printing the result from t-test to evaluate the statistics

ttest_blackpop_nhwhitepop <- t.test(cases_black_pop,
                                    cases_nhwhite_pop,
                                    alternative = "two.sided",
                                    var.equal = F)
print(ttest_blackpop_nhwhitepop)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  cases_black_pop and cases_nhwhite_pop
    ## t = 13.906, df = 138.26, p-value < 2.2e-16
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  1735.596 2310.985
    ## sample estimates:
    ## mean of x mean of y 
    ##  3869.485  1846.195

``` r
## Based on the results from the t-test the two iterations displaying significant differences is the comparison between the Asian and Black or African American population groups, and the Black or African American and Non-Hispanic White population groups.
## The first example, there is quite a significant difference in the t-score and the p-value. This indicates that the mean values associated between these groups differ and the probability this is due to random chance is unlikely.
## The second example, there is more of a significant difference in the t-score and the p-value. This indicates that the mean values are quite different between these two groups, and the p-value indicates this observation is unlikely due to random chance.
## The most similar groups in terms of impact from COVID-19 are the Asian and Non-Hispanic White population, as these metrics are closer to 0, which indicates there is more similarity in the distribution of the mean values.
## Overall, these tests indicate that the Black or African American group were more severely impacted from COVID-19 when we compare both the ridge plots and the comparative t-tests.
```

``` r
### The demographic groups stored in a vector 'pop_groups'

pop_groups <- c("Asian", 
            "Black or African American", 
            "Non-Hispanic White", 
            "Hispanic or Latino", 
            "Native American(Including AK)")

## Setting the arguments for the matrix
## Rows and columns set using the length() function, which is designed to be 5 rows and 5 columns based on the previous list

pmatrix <- matrix(NA, 
                  nrow = length(pop_groups), 
                  ncol = length(pop_groups),
                  dimnames = list(pop_groups, pop_groups))

## For loop created for iterating over all of the combinations
## Utilizing the combn() function to generate all possible combinations
## Establishing the groups to be paired

for (pair in combn(pop_groups, 2, simplify = F)) {
  group1 <- pair[1]
  group2 <- pair[2]
  ## Creating two variables to store values for the majority demographic group in each US county
  vals1 <- us_counties_sf$cases_per_100k[us_counties_sf$Majority_Race == group1]
  vals2 <- us_counties_sf$cases_per_100k[us_counties_sf$Majority_Race == group2]
  ## Checking that the values for each group are sufficient to perform the t-test as the requirement is two
  ## Performs the t-test and stores the p-value for each demographic group
  if (length(vals1) >= 2 && length(vals2) >= 2) {
    ttest_result <- t.test(vals1, vals2, alternative = "two.sided", var.equal = F)
    pmatrix[group1, group2] <- ttest_result$p.value
    pmatrix[group2, group1] <- ttest_result$p.value
  }
}
diag(pmatrix) <- 1
print(round(pmatrix, 3))
```

    ##                               Asian Black or African American Non-Hispanic White Hispanic or Latino Native American(Including AK)
    ## Asian                         1.000                     0.000              0.007              0.000                            NA
    ## Black or African American     0.000                     1.000              0.000              0.001                            NA
    ## Non-Hispanic White            0.007                     0.000              1.000              0.000                            NA
    ## Hispanic or Latino            0.000                     0.001              0.000              1.000                            NA
    ## Native American(Including AK)    NA                        NA                 NA                 NA                             1
