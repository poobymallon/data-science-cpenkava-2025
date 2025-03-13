COVID-19
================
Cooper Penkava
2025-03-12

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Submission](#submission)
- [The Big Picture](#the-big-picture)
- [Get the Data](#get-the-data)
  - [Navigating the Census Bureau](#navigating-the-census-bureau)
    - [**q1** Load Table `B01003` into the following tibble. Make sure
      the column names are
      `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.](#q1-load-table-b01003-into-the-following-tibble-make-sure-the-column-names-are-id-geographic-area-name-estimatetotal-margin-of-errortotal)
  - [Automated Download of NYT Data](#automated-download-of-nyt-data)
    - [**q2** Visit the NYT GitHub repo and find the URL for the **raw**
      US County-level data. Assign that URL as a string to the variable
      below.](#q2-visit-the-nyt-github-repo-and-find-the-url-for-the-raw-us-county-level-data-assign-that-url-as-a-string-to-the-variable-below)
- [Join the Data](#join-the-data)
  - [**q3** Process the `id` column of `df_pop` to create a `fips`
    column.](#q3-process-the-id-column-of-df_pop-to-create-a-fips-column)
  - [**q4** Join `df_covid` with `df_q3` by the `fips` column. Use the
    proper type of join to preserve *only* the rows in
    `df_covid`.](#q4-join-df_covid-with-df_q3-by-the-fips-column-use-the-proper-type-of-join-to-preserve-only-the-rows-in-df_covid)
- [Analyze](#analyze)
  - [Normalize](#normalize)
    - [**q5** Use the `population` estimates in `df_data` to normalize
      `cases` and `deaths` to produce per 100,000 counts \[3\]. Store
      these values in the columns `cases_per100k` and
      `deaths_per100k`.](#q5-use-the-population-estimates-in-df_data-to-normalize-cases-and-deaths-to-produce-per-100000-counts-3-store-these-values-in-the-columns-cases_per100k-and-deaths_per100k)
  - [Guided EDA](#guided-eda)
    - [**q6** Compute some summaries](#q6-compute-some-summaries)
    - [**q7** Find and compare the top
      10](#q7-find-and-compare-the-top-10)
  - [Self-directed EDA](#self-directed-eda)
    - [**q8** Drive your own ship: You’ve just put together a very rich
      dataset; you now get to explore! Pick your own direction and
      generate at least one punchline figure to document an interesting
      finding. I give a couple tips & ideas
      below:](#q8-drive-your-own-ship-youve-just-put-together-a-very-rich-dataset-you-now-get-to-explore-pick-your-own-direction-and-generate-at-least-one-punchline-figure-to-document-an-interesting-finding-i-give-a-couple-tips--ideas-below)
    - [Ideas](#ideas)
    - [Aside: Some visualization
      tricks](#aside-some-visualization-tricks)
    - [Geographic exceptions](#geographic-exceptions)
- [Notes](#notes)

*Purpose*: In this challenge, you’ll learn how to navigate the U.S.
Census Bureau website, programmatically download data from the internet,
and perform a county-level population-weighted analysis of current
COVID-19 trends. This will give you the base for a very deep
investigation of COVID-19, which we’ll build upon for Project 1.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category | Needs Improvement | Satisfactory |
|----|----|----|
| Effort | Some task **q**’s left unattempted | All task **q**’s attempted |
| Observed | Did not document observations, or observations incorrect | Documented correct observations based on analysis |
| Supported | Some observations not clearly supported by analysis | All observations clearly supported by analysis (table, graph, etc.) |
| Assessed | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support |
| Specified | Uses the phrase “more data are necessary” without clarification | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability | Code sufficiently close to the [style guide](https://style.tidyverse.org/) |

## Submission

<!-- ------------------------- -->

Make sure to commit both the challenge report (`report.md` file) and
supporting files (`report_files/` folder) when you are done! Then submit
a link to Canvas. **Your Challenge submission is not complete without
all files uploaded to GitHub.**

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

*Background*:
[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) is
the disease caused by the virus SARS-CoV-2. In 2020 it became a global
pandemic, leading to huge loss of life and tremendous disruption to
society. The New York Times (as of writing) publishes up-to-date data on
the progression of the pandemic across the United States—we will study
these data in this challenge.

*Optional Readings*: I’ve found this [ProPublica
piece](https://www.propublica.org/article/how-to-understand-covid-19-numbers)
on “How to understand COVID-19 numbers” to be very informative!

# The Big Picture

<!-- -------------------------------------------------- -->

We’re about to go through *a lot* of weird steps, so let’s first fix the
big picture firmly in mind:

We want to study COVID-19 in terms of data: both case counts (number of
infections) and deaths. We’re going to do a county-level analysis in
order to get a high-resolution view of the pandemic. Since US counties
can vary widely in terms of their population, we’ll need population
estimates in order to compute infection rates (think back to the
`Titanic` challenge).

That’s the high-level view; now let’s dig into the details.

# Get the Data

<!-- -------------------------------------------------- -->

1.  County-level population estimates (Census Bureau)
2.  County-level COVID-19 counts (New York Times)

## Navigating the Census Bureau

<!-- ------------------------- -->

**Steps**: Our objective is to find the 2018 American Community
Survey\[1\] (ACS) Total Population estimates, disaggregated by counties.
To check your results, this is Table `B01003`.

1.  Go to [data.census.gov](data.census.gov).
2.  Scroll down and click `View Tables`.
3.  Apply filters to find the ACS **Total Population** estimates,
    disaggregated by counties. I used the filters:

- `Topics > Populations and People > Counts, Estimates, and Projections > Population Total`
- `Geography > County > All counties in United States`

5.  Select the **Total Population** table and click the `Download`
    button to download the data; make sure to select the 2018 5-year
    estimates.
6.  Unzip and move the data to your `challenges/data` folder.

- Note that the data will have a crazy-long filename like
  `ACSDT5Y2018.B01003_data_with_overlays_2020-07-26T094857.csv`. That’s
  because metadata is stored in the filename, such as the year of the
  estimate (`Y2018`) and my access date (`2020-07-26`). **Your filename
  will vary based on when you download the data**, so make sure to copy
  the filename that corresponds to what you downloaded!

### **q1** Load Table `B01003` into the following tibble. Make sure the column names are `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.

*Hint*: You will need to use the `skip` keyword when loading these data!

``` r
## TASK: Load the census bureau data with the following tibble name.
name = "./data/ACSDT5Y2018.B01003-Data.csv"
df_pop <- read.csv(name)
df_pop
```

    ##              GEO_ID                                       NAME     B01003_001E
    ## 1         Geography                       Geographic Area Name Estimate!!Total
    ## 2    0500000US01001                    Autauga County, Alabama           55200
    ## 3    0500000US01003                    Baldwin County, Alabama          208107
    ## 4    0500000US01005                    Barbour County, Alabama           25782
    ## 5    0500000US01007                       Bibb County, Alabama           22527
    ## 6    0500000US01009                     Blount County, Alabama           57645
    ## 7    0500000US01011                    Bullock County, Alabama           10352
    ## 8    0500000US01013                     Butler County, Alabama           20025
    ## 9    0500000US01015                    Calhoun County, Alabama          115098
    ## 10   0500000US01017                   Chambers County, Alabama           33826
    ## 11   0500000US01019                   Cherokee County, Alabama           25853
    ## 12   0500000US01021                    Chilton County, Alabama           43930
    ## 13   0500000US01023                    Choctaw County, Alabama           13075
    ## 14   0500000US01025                     Clarke County, Alabama           24387
    ## 15   0500000US01027                       Clay County, Alabama           13378
    ## 16   0500000US01029                   Cleburne County, Alabama           14938
    ## 17   0500000US01031                     Coffee County, Alabama           51288
    ## 18   0500000US01033                    Colbert County, Alabama           54495
    ## 19   0500000US01035                    Conecuh County, Alabama           12514
    ## 20   0500000US01037                      Coosa County, Alabama           10855
    ## 21   0500000US01039                  Covington County, Alabama           37351
    ## 22   0500000US01041                   Crenshaw County, Alabama           13865
    ## 23   0500000US01043                    Cullman County, Alabama           82313
    ## 24   0500000US01045                       Dale County, Alabama           49255
    ## 25   0500000US01047                     Dallas County, Alabama           40029
    ## 26   0500000US01049                     DeKalb County, Alabama           71200
    ## 27   0500000US01051                     Elmore County, Alabama           81212
    ## 28   0500000US01053                   Escambia County, Alabama           37328
    ## 29   0500000US01055                     Etowah County, Alabama          102939
    ## 30   0500000US01057                    Fayette County, Alabama           16585
    ## 31   0500000US01059                   Franklin County, Alabama           31542
    ## 32   0500000US01061                     Geneva County, Alabama           26491
    ## 33   0500000US01063                     Greene County, Alabama            8426
    ## 34   0500000US01065                       Hale County, Alabama           14887
    ## 35   0500000US01067                      Henry County, Alabama           17124
    ## 36   0500000US01069                    Houston County, Alabama          104352
    ## 37   0500000US01071                    Jackson County, Alabama           52094
    ## 38   0500000US01073                  Jefferson County, Alabama          659892
    ## 39   0500000US01075                      Lamar County, Alabama           13933
    ## 40   0500000US01077                 Lauderdale County, Alabama           92585
    ## 41   0500000US01079                   Lawrence County, Alabama           33171
    ## 42   0500000US01081                        Lee County, Alabama          159287
    ## 43   0500000US01083                  Limestone County, Alabama           93052
    ## 44   0500000US01085                    Lowndes County, Alabama           10236
    ## 45   0500000US01087                      Macon County, Alabama           19054
    ## 46   0500000US01089                    Madison County, Alabama          357560
    ## 47   0500000US01091                    Marengo County, Alabama           19538
    ## 48   0500000US01093                     Marion County, Alabama           29965
    ## 49   0500000US01095                   Marshall County, Alabama           95145
    ## 50   0500000US01097                     Mobile County, Alabama          414659
    ## 51   0500000US01099                     Monroe County, Alabama           21512
    ## 52   0500000US01101                 Montgomery County, Alabama          226941
    ## 53   0500000US01103                     Morgan County, Alabama          119122
    ## 54   0500000US01105                      Perry County, Alabama            9486
    ## 55   0500000US01107                    Pickens County, Alabama           20298
    ## 56   0500000US01109                       Pike County, Alabama           33403
    ## 57   0500000US01111                   Randolph County, Alabama           22574
    ## 58   0500000US01113                    Russell County, Alabama           58213
    ## 59   0500000US01115                  St. Clair County, Alabama           87306
    ## 60   0500000US01117                     Shelby County, Alabama          211261
    ## 61   0500000US01119                     Sumter County, Alabama           12985
    ## 62   0500000US01121                  Talladega County, Alabama           80565
    ## 63   0500000US01123                 Tallapoosa County, Alabama           40636
    ## 64   0500000US01125                 Tuscaloosa County, Alabama          206213
    ## 65   0500000US01127                     Walker County, Alabama           64493
    ## 66   0500000US01129                 Washington County, Alabama           16643
    ## 67   0500000US01131                     Wilcox County, Alabama           10809
    ## 68   0500000US01133                    Winston County, Alabama           23875
    ## 69   0500000US02013             Aleutians East Borough, Alaska            3425
    ## 70   0500000US02016         Aleutians West Census Area, Alaska            5750
    ## 71   0500000US02020             Anchorage Municipality, Alaska          296112
    ## 72   0500000US02050                 Bethel Census Area, Alaska           18040
    ## 73   0500000US02060                Bristol Bay Borough, Alaska             890
    ## 74   0500000US02068                     Denali Borough, Alaska            2232
    ## 75   0500000US02070             Dillingham Census Area, Alaska            4975
    ## 76   0500000US02090       Fairbanks North Star Borough, Alaska           99653
    ## 77   0500000US02100                     Haines Borough, Alaska            2518
    ## 78   0500000US02105          Hoonah-Angoon Census Area, Alaska            2132
    ## 79   0500000US02110            Juneau City and Borough, Alaska           32330
    ## 80   0500000US02122            Kenai Peninsula Borough, Alaska           58220
    ## 81   0500000US02130          Ketchikan Gateway Borough, Alaska           13804
    ## 82   0500000US02150              Kodiak Island Borough, Alaska           13649
    ## 83   0500000US02158               Kusilvak Census Area, Alaska            8198
    ## 84   0500000US02164         Lake and Peninsula Borough, Alaska            1375
    ## 85   0500000US02170          Matanuska-Susitna Borough, Alaska          103464
    ## 86   0500000US02180                   Nome Census Area, Alaska            9925
    ## 87   0500000US02185                North Slope Borough, Alaska            9797
    ## 88   0500000US02188           Northwest Arctic Borough, Alaska            7734
    ## 89   0500000US02195                 Petersburg Borough, Alaska            3255
    ## 90   0500000US02198  Prince of Wales-Hyder Census Area, Alaska            6474
    ## 91   0500000US02220             Sitka City and Borough, Alaska            8738
    ## 92   0500000US02230               Skagway Municipality, Alaska            1061
    ## 93   0500000US02240    Southeast Fairbanks Census Area, Alaska            6876
    ## 94   0500000US02261         Valdez-Cordova Census Area, Alaska            9301
    ## 95   0500000US02275          Wrangell City and Borough, Alaska            2484
    ## 96   0500000US02282           Yakutat City and Borough, Alaska             689
    ## 97   0500000US02290          Yukon-Koyukuk Census Area, Alaska            5415
    ## 98   0500000US04001                     Apache County, Arizona           71522
    ## 99   0500000US04003                    Cochise County, Arizona          126279
    ## 100  0500000US04005                   Coconino County, Arizona          140217
    ## 101  0500000US04007                       Gila County, Arizona           53400
    ## 102  0500000US04009                     Graham County, Arizona           37879
    ## 103  0500000US04011                   Greenlee County, Arizona            9504
    ## 104  0500000US04012                     La Paz County, Arizona           20701
    ## 105  0500000US04013                   Maricopa County, Arizona         4253913
    ## 106  0500000US04015                     Mohave County, Arizona          206064
    ## 107  0500000US04017                     Navajo County, Arizona          108705
    ## 108  0500000US04019                       Pima County, Arizona         1019722
    ## 109  0500000US04021                      Pinal County, Arizona          419721
    ## 110  0500000US04023                 Santa Cruz County, Arizona           46584
    ## 111  0500000US04025                    Yavapai County, Arizona          224645
    ## 112  0500000US04027                       Yuma County, Arizona          207829
    ## 113  0500000US05001                  Arkansas County, Arkansas           18124
    ## 114  0500000US05003                    Ashley County, Arkansas           20537
    ## 115  0500000US05005                    Baxter County, Arkansas           41219
    ## 116  0500000US05007                    Benton County, Arkansas          258980
    ## 117  0500000US05009                     Boone County, Arkansas           37288
    ## 118  0500000US05011                   Bradley County, Arkansas           10948
    ## 119  0500000US05013                   Calhoun County, Arkansas            5202
    ## 120  0500000US05015                   Carroll County, Arkansas           27887
    ## 121  0500000US05017                    Chicot County, Arkansas           10826
    ## 122  0500000US05019                     Clark County, Arkansas           22385
    ## 123  0500000US05021                      Clay County, Arkansas           15061
    ## 124  0500000US05023                  Cleburne County, Arkansas           25230
    ## 125  0500000US05025                 Cleveland County, Arkansas            8226
    ## 126  0500000US05027                  Columbia County, Arkansas           23892
    ## 127  0500000US05029                    Conway County, Arkansas           20906
    ## 128  0500000US05031                 Craighead County, Arkansas          105701
    ## 129  0500000US05033                  Crawford County, Arkansas           62472
    ## 130  0500000US05035                Crittenden County, Arkansas           49013
    ## 131  0500000US05037                     Cross County, Arkansas           16998
    ## 132  0500000US05039                    Dallas County, Arkansas            7432
    ## 133  0500000US05041                     Desha County, Arkansas           11887
    ## 134  0500000US05043                      Drew County, Arkansas           18502
    ## 135  0500000US05045                  Faulkner County, Arkansas          122416
    ## 136  0500000US05047                  Franklin County, Arkansas           17780
    ## 137  0500000US05049                    Fulton County, Arkansas           12139
    ## 138  0500000US05051                   Garland County, Arkansas           98296
    ## 139  0500000US05053                     Grant County, Arkansas           18086
    ## 140  0500000US05055                    Greene County, Arkansas           44623
    ## 141  0500000US05057                 Hempstead County, Arkansas           22018
    ## 142  0500000US05059                Hot Spring County, Arkansas           33520
    ## 143  0500000US05061                    Howard County, Arkansas           13389
    ## 144  0500000US05063              Independence County, Arkansas           37264
    ## 145  0500000US05065                     Izard County, Arkansas           13559
    ## 146  0500000US05067                   Jackson County, Arkansas           17225
    ## 147  0500000US05069                 Jefferson County, Arkansas           70424
    ## 148  0500000US05071                   Johnson County, Arkansas           26291
    ## 149  0500000US05073                 Lafayette County, Arkansas            6915
    ## 150  0500000US05075                  Lawrence County, Arkansas           16669
    ## 151  0500000US05077                       Lee County, Arkansas            9398
    ## 152  0500000US05079                   Lincoln County, Arkansas           13695
    ## 153  0500000US05081              Little River County, Arkansas           12417
    ## 154  0500000US05083                     Logan County, Arkansas           21757
    ## 155  0500000US05085                    Lonoke County, Arkansas           72206
    ## 156  0500000US05087                   Madison County, Arkansas           16076
    ## 157  0500000US05089                    Marion County, Arkansas           16438
    ## 158  0500000US05091                    Miller County, Arkansas           43759
    ## 159  0500000US05093               Mississippi County, Arkansas           42831
    ## 160  0500000US05095                    Monroe County, Arkansas            7249
    ## 161  0500000US05097                Montgomery County, Arkansas            8993
    ## 162  0500000US05099                    Nevada County, Arkansas            8440
    ## 163  0500000US05101                    Newton County, Arkansas            7848
    ## 164  0500000US05103                  Ouachita County, Arkansas           24106
    ## 165  0500000US05105                     Perry County, Arkansas           10322
    ## 166  0500000US05107                  Phillips County, Arkansas           19034
    ## 167  0500000US05109                      Pike County, Arkansas           10808
    ## 168  0500000US05111                  Poinsett County, Arkansas           24054
    ## 169  0500000US05113                      Polk County, Arkansas           20163
    ## 170  0500000US05115                      Pope County, Arkansas           63644
    ## 171  0500000US05117                   Prairie County, Arkansas            8244
    ## 172  0500000US05119                   Pulaski County, Arkansas          393463
    ## 173  0500000US05121                  Randolph County, Arkansas           17603
    ## 174  0500000US05123               St. Francis County, Arkansas           26294
    ## 175  0500000US05125                    Saline County, Arkansas          118009
    ## 176  0500000US05127                     Scott County, Arkansas           10442
    ## 177  0500000US05129                    Searcy County, Arkansas            7923
    ## 178  0500000US05131                 Sebastian County, Arkansas          127461
    ## 179  0500000US05133                    Sevier County, Arkansas           17193
    ## 180  0500000US05135                     Sharp County, Arkansas           17043
    ## 181  0500000US05137                     Stone County, Arkansas           12446
    ## 182  0500000US05139                     Union County, Arkansas           39732
    ## 183  0500000US05141                 Van Buren County, Arkansas           16684
    ## 184  0500000US05143                Washington County, Arkansas          228529
    ## 185  0500000US05145                     White County, Arkansas           78804
    ## 186  0500000US05147                  Woodruff County, Arkansas            6660
    ## 187  0500000US05149                      Yell County, Arkansas           21573
    ## 188  0500000US06001                 Alameda County, California         1643700
    ## 189  0500000US06003                  Alpine County, California            1146
    ## 190  0500000US06005                  Amador County, California           37829
    ## 191  0500000US06007                   Butte County, California          227075
    ## 192  0500000US06009               Calaveras County, California           45235
    ## 193  0500000US06011                  Colusa County, California           21464
    ## 194  0500000US06013            Contra Costa County, California         1133247
    ## 195  0500000US06015               Del Norte County, California           27424
    ## 196  0500000US06017               El Dorado County, California          186661
    ## 197  0500000US06019                  Fresno County, California          978130
    ## 198  0500000US06021                   Glenn County, California           27897
    ## 199  0500000US06023                Humboldt County, California          135768
    ## 200  0500000US06025                Imperial County, California          180216
    ## 201  0500000US06027                    Inyo County, California           18085
    ## 202  0500000US06029                    Kern County, California          883053
    ## 203  0500000US06031                   Kings County, California          150075
    ## 204  0500000US06033                    Lake County, California           64148
    ## 205  0500000US06035                  Lassen County, California           31185
    ## 206  0500000US06037             Los Angeles County, California        10098052
    ## 207  0500000US06039                  Madera County, California          155013
    ## 208  0500000US06041                   Marin County, California          260295
    ## 209  0500000US06043                Mariposa County, California           17540
    ## 210  0500000US06045               Mendocino County, California           87422
    ## 211  0500000US06047                  Merced County, California          269075
    ## 212  0500000US06049                   Modoc County, California            8938
    ## 213  0500000US06051                    Mono County, California           14174
    ## 214  0500000US06053                Monterey County, California          433212
    ## 215  0500000US06055                    Napa County, California          140530
    ## 216  0500000US06057                  Nevada County, California           99092
    ## 217  0500000US06059                  Orange County, California         3164182
    ## 218  0500000US06061                  Placer County, California          380077
    ## 219  0500000US06063                  Plumas County, California           18699
    ## 220  0500000US06065               Riverside County, California         2383286
    ## 221  0500000US06067              Sacramento County, California         1510023
    ## 222  0500000US06069              San Benito County, California           59416
    ## 223  0500000US06071          San Bernardino County, California         2135413
    ## 224  0500000US06073               San Diego County, California         3302833
    ## 225  0500000US06075           San Francisco County, California          870044
    ## 226  0500000US06077             San Joaquin County, California          732212
    ## 227  0500000US06079         San Luis Obispo County, California          281455
    ## 228  0500000US06081               San Mateo County, California          765935
    ## 229  0500000US06083           Santa Barbara County, California          443738
    ## 230  0500000US06085             Santa Clara County, California         1922200
    ## 231  0500000US06087              Santa Cruz County, California          273765
    ## 232  0500000US06089                  Shasta County, California          179085
    ## 233  0500000US06091                  Sierra County, California            2930
    ## 234  0500000US06093                Siskiyou County, California           43540
    ## 235  0500000US06095                  Solano County, California          438530
    ## 236  0500000US06097                  Sonoma County, California          501317
    ## 237  0500000US06099              Stanislaus County, California          539301
    ## 238  0500000US06101                  Sutter County, California           95872
    ## 239  0500000US06103                  Tehama County, California           63373
    ## 240  0500000US06105                 Trinity County, California           12862
    ## 241  0500000US06107                  Tulare County, California          460477
    ## 242  0500000US06109                Tuolumne County, California           53932
    ## 243  0500000US06111                 Ventura County, California          848112
    ## 244  0500000US06113                    Yolo County, California          214977
    ## 245  0500000US06115                    Yuba County, California           75493
    ## 246  0500000US08001                     Adams County, Colorado          497115
    ## 247  0500000US08003                   Alamosa County, Colorado           16444
    ## 248  0500000US08005                  Arapahoe County, Colorado          636671
    ## 249  0500000US08007                 Archuleta County, Colorado           12908
    ## 250  0500000US08009                      Baca County, Colorado            3563
    ## 251  0500000US08011                      Bent County, Colorado            5809
    ## 252  0500000US08013                   Boulder County, Colorado          321030
    ## 253  0500000US08014                Broomfield County, Colorado           66120
    ## 254  0500000US08015                   Chaffee County, Colorado           19164
    ## 255  0500000US08017                  Cheyenne County, Colorado            2039
    ## 256  0500000US08019               Clear Creek County, Colorado            9379
    ## 257  0500000US08021                   Conejos County, Colorado            8142
    ## 258  0500000US08023                  Costilla County, Colorado            3687
    ## 259  0500000US08025                   Crowley County, Colorado            5630
    ## 260  0500000US08027                    Custer County, Colorado            4640
    ## 261  0500000US08029                     Delta County, Colorado           30346
    ## 262  0500000US08031                    Denver County, Colorado          693417
    ## 263  0500000US08033                   Dolores County, Colorado            1841
    ## 264  0500000US08035                   Douglas County, Colorado          328614
    ## 265  0500000US08037                     Eagle County, Colorado           54357
    ## 266  0500000US08039                    Elbert County, Colorado           25162
    ## 267  0500000US08041                   El Paso County, Colorado          688153
    ## 268  0500000US08043                   Fremont County, Colorado           47002
    ## 269  0500000US08045                  Garfield County, Colorado           58538
    ## 270  0500000US08047                    Gilpin County, Colorado            5924
    ## 271  0500000US08049                     Grand County, Colorado           15066
    ## 272  0500000US08051                  Gunnison County, Colorado           16537
    ## 273  0500000US08053                  Hinsdale County, Colorado             878
    ## 274  0500000US08055                  Huerfano County, Colorado            6583
    ## 275  0500000US08057                   Jackson County, Colorado            1296
    ## 276  0500000US08059                 Jefferson County, Colorado          570427
    ## 277  0500000US08061                     Kiowa County, Colorado            1449
    ## 278  0500000US08063                Kit Carson County, Colorado            7635
    ## 279  0500000US08065                      Lake County, Colorado            7585
    ## 280  0500000US08067                  La Plata County, Colorado           55101
    ## 281  0500000US08069                   Larimer County, Colorado          338161
    ## 282  0500000US08071                Las Animas County, Colorado           14179
    ## 283  0500000US08073                   Lincoln County, Colorado            5548
    ## 284  0500000US08075                     Logan County, Colorado           21689
    ## 285  0500000US08077                      Mesa County, Colorado          149998
    ## 286  0500000US08079                   Mineral County, Colorado             823
    ## 287  0500000US08081                    Moffat County, Colorado           13060
    ## 288  0500000US08083                 Montezuma County, Colorado           25909
    ## 289  0500000US08085                  Montrose County, Colorado           41268
    ## 290  0500000US08087                    Morgan County, Colorado           28257
    ## 291  0500000US08089                     Otero County, Colorado           18325
    ## 292  0500000US08091                     Ouray County, Colorado            4722
    ## 293  0500000US08093                      Park County, Colorado           17392
    ## 294  0500000US08095                  Phillips County, Colorado            4318
    ## 295  0500000US08097                    Pitkin County, Colorado           17909
    ## 296  0500000US08099                   Prowers County, Colorado           12052
    ## 297  0500000US08101                    Pueblo County, Colorado          164685
    ## 298  0500000US08103                Rio Blanco County, Colorado            6465
    ## 299  0500000US08105                Rio Grande County, Colorado           11351
    ## 300  0500000US08107                     Routt County, Colorado           24874
    ## 301  0500000US08109                  Saguache County, Colorado            6468
    ## 302  0500000US08111                  San Juan County, Colorado             544
    ## 303  0500000US08113                San Miguel County, Colorado            7968
    ## 304  0500000US08115                  Sedgwick County, Colorado            2350
    ## 305  0500000US08117                    Summit County, Colorado           30429
    ## 306  0500000US08119                    Teller County, Colorado           24113
    ## 307  0500000US08121                Washington County, Colorado            4840
    ## 308  0500000US08123                      Weld County, Colorado          295123
    ## 309  0500000US08125                      Yuma County, Colorado           10069
    ## 310  0500000US09001              Fairfield County, Connecticut          944348
    ## 311  0500000US09003               Hartford County, Connecticut          894730
    ## 312  0500000US09005             Litchfield County, Connecticut          183031
    ## 313  0500000US09007              Middlesex County, Connecticut          163368
    ## 314  0500000US09009              New Haven County, Connecticut          859339
    ## 315  0500000US09011             New London County, Connecticut          268881
    ## 316  0500000US09013                Tolland County, Connecticut          151269
    ## 317  0500000US09015                Windham County, Connecticut          116538
    ## 318  0500000US10001                      Kent County, Delaware          174822
    ## 319  0500000US10003                New Castle County, Delaware          555133
    ## 320  0500000US10005                    Sussex County, Delaware          219540
    ## 321  0500000US11001 District of Columbia, District of Columbia          684498
    ## 322  0500000US12001                    Alachua County, Florida          263148
    ## 323  0500000US12003                      Baker County, Florida           27785
    ## 324  0500000US12005                        Bay County, Florida          182482
    ## 325  0500000US12007                   Bradford County, Florida           26979
    ## 326  0500000US12009                    Brevard County, Florida          576808
    ## 327  0500000US12011                    Broward County, Florida         1909151
    ## 328  0500000US12013                    Calhoun County, Florida           14444
    ## 329  0500000US12015                  Charlotte County, Florida          176954
    ## 330  0500000US12017                     Citrus County, Florida          143087
    ## 331  0500000US12019                       Clay County, Florida          207291
    ## 332  0500000US12021                    Collier County, Florida          363922
    ## 333  0500000US12023                   Columbia County, Florida           69105
    ## 334  0500000US12027                     DeSoto County, Florida           36399
    ## 335  0500000US12029                      Dixie County, Florida           16437
    ## 336  0500000US12031                      Duval County, Florida          924229
    ## 337  0500000US12033                   Escambia County, Florida          311522
    ## 338  0500000US12035                    Flagler County, Florida          107139
    ## 339  0500000US12037                   Franklin County, Florida           11736
    ## 340  0500000US12039                    Gadsden County, Florida           46017
    ## 341  0500000US12041                  Gilchrist County, Florida           17615
    ## 342  0500000US12043                     Glades County, Florida           13363
    ## 343  0500000US12045                       Gulf County, Florida           16055
    ## 344  0500000US12047                   Hamilton County, Florida           14269
    ## 345  0500000US12049                     Hardee County, Florida           27228
    ## 346  0500000US12051                     Hendry County, Florida           40127
    ## 347  0500000US12053                   Hernando County, Florida          182696
    ## 348  0500000US12055                  Highlands County, Florida          102101
    ## 349  0500000US12057               Hillsborough County, Florida         1378883
    ## 350  0500000US12059                     Holmes County, Florida           19430
    ## 351  0500000US12061               Indian River County, Florida          150984
    ## 352  0500000US12063                    Jackson County, Florida           48472
    ## 353  0500000US12065                  Jefferson County, Florida           14105
    ## 354  0500000US12067                  Lafayette County, Florida            8744
    ## 355  0500000US12069                       Lake County, Florida          335362
    ## 356  0500000US12071                        Lee County, Florida          718679
    ## 357  0500000US12073                       Leon County, Florida          288102
    ## 358  0500000US12075                       Levy County, Florida           39961
    ## 359  0500000US12077                    Liberty County, Florida            8365
    ## 360  0500000US12079                    Madison County, Florida           18474
    ## 361  0500000US12081                    Manatee County, Florida          373853
    ## 362  0500000US12083                     Marion County, Florida          348371
    ## 363  0500000US12085                     Martin County, Florida          157581
    ## 364  0500000US12086                 Miami-Dade County, Florida         2715516
    ## 365  0500000US12087                     Monroe County, Florida           76325
    ## 366  0500000US12089                     Nassau County, Florida           80578
    ## 367  0500000US12091                   Okaloosa County, Florida          200737
    ## 368  0500000US12093                 Okeechobee County, Florida           40572
    ## 369  0500000US12095                     Orange County, Florida         1321194
    ## 370  0500000US12097                    Osceola County, Florida          338619
    ## 371  0500000US12099                 Palm Beach County, Florida         1446277
    ## 372  0500000US12101                      Pasco County, Florida          510593
    ## 373  0500000US12103                   Pinellas County, Florida          957875
    ## 374  0500000US12105                       Polk County, Florida          668671
    ## 375  0500000US12107                     Putnam County, Florida           72766
    ## 376  0500000US12109                  St. Johns County, Florida          235503
    ## 377  0500000US12111                  St. Lucie County, Florida          305591
    ## 378  0500000US12113                 Santa Rosa County, Florida          170442
    ## 379  0500000US12115                   Sarasota County, Florida          412144
    ## 380  0500000US12117                   Seminole County, Florida          455086
    ## 381  0500000US12119                     Sumter County, Florida          120999
    ## 382  0500000US12121                   Suwannee County, Florida           43924
    ## 383  0500000US12123                     Taylor County, Florida           22098
    ## 384  0500000US12125                      Union County, Florida           15239
    ## 385  0500000US12127                    Volusia County, Florida          527634
    ## 386  0500000US12129                    Wakulla County, Florida           31877
    ## 387  0500000US12131                     Walton County, Florida           65858
    ## 388  0500000US12133                 Washington County, Florida           24566
    ## 389  0500000US13001                    Appling County, Georgia           18454
    ## 390  0500000US13003                   Atkinson County, Georgia            8265
    ## 391  0500000US13005                      Bacon County, Georgia           11228
    ## 392  0500000US13007                      Baker County, Georgia            3189
    ## 393  0500000US13009                    Baldwin County, Georgia           45286
    ## 394  0500000US13011                      Banks County, Georgia           18510
    ## 395  0500000US13013                     Barrow County, Georgia           76887
    ## 396  0500000US13015                     Bartow County, Georgia          103620
    ## 397  0500000US13017                   Ben Hill County, Georgia           17154
    ## 398  0500000US13019                    Berrien County, Georgia           19025
    ## 399  0500000US13021                       Bibb County, Georgia          153490
    ## 400  0500000US13023                   Bleckley County, Georgia           12775
    ## 401  0500000US13025                   Brantley County, Georgia           18561
    ## 402  0500000US13027                     Brooks County, Georgia           15622
    ## 403  0500000US13029                      Bryan County, Georgia           35885
    ## 404  0500000US13031                    Bulloch County, Georgia           74782
    ## 405  0500000US13033                      Burke County, Georgia           22550
    ## 406  0500000US13035                      Butts County, Georgia           23750
    ## 407  0500000US13037                    Calhoun County, Georgia            6428
    ## 408  0500000US13039                     Camden County, Georgia           52714
    ## 409  0500000US13043                    Candler County, Georgia           10827
    ## 410  0500000US13045                    Carroll County, Georgia          116022
    ## 411  0500000US13047                    Catoosa County, Georgia           66299
    ## 412  0500000US13049                   Charlton County, Georgia           12983
    ## 413  0500000US13051                    Chatham County, Georgia          287049
    ## 414  0500000US13053              Chattahoochee County, Georgia           10767
    ## 415  0500000US13055                  Chattooga County, Georgia           24817
    ## 416  0500000US13057                   Cherokee County, Georgia          241910
    ## 417  0500000US13059                     Clarke County, Georgia          124602
    ## 418  0500000US13061                       Clay County, Georgia            3001
    ## 419  0500000US13063                    Clayton County, Georgia          278666
    ## 420  0500000US13065                     Clinch County, Georgia            6743
    ## 421  0500000US13067                       Cobb County, Georgia          745057
    ## 422  0500000US13069                     Coffee County, Georgia           42961
    ## 423  0500000US13071                   Colquitt County, Georgia           45606
    ## 424  0500000US13073                   Columbia County, Georgia          147295
    ## 425  0500000US13075                       Cook County, Georgia           17184
    ## 426  0500000US13077                     Coweta County, Georgia          140516
    ## 427  0500000US13079                   Crawford County, Georgia           12344
    ## 428  0500000US13081                      Crisp County, Georgia           22846
    ## 429  0500000US13083                       Dade County, Georgia           16227
    ## 430  0500000US13085                     Dawson County, Georgia           23861
    ## 431  0500000US13087                    Decatur County, Georgia           26833
    ## 432  0500000US13089                     DeKalb County, Georgia          743187
    ## 433  0500000US13091                      Dodge County, Georgia           20919
    ## 434  0500000US13093                      Dooly County, Georgia           13905
    ## 435  0500000US13095                  Dougherty County, Georgia           91049
    ## 436  0500000US13097                    Douglas County, Georgia          141840
    ## 437  0500000US13099                      Early County, Georgia           10348
    ## 438  0500000US13101                     Echols County, Georgia            3994
    ## 439  0500000US13103                  Effingham County, Georgia           58689
    ## 440  0500000US13105                     Elbert County, Georgia           19212
    ## 441  0500000US13107                    Emanuel County, Georgia           22499
    ## 442  0500000US13109                      Evans County, Georgia           10727
    ## 443  0500000US13111                     Fannin County, Georgia           24925
    ## 444  0500000US13113                    Fayette County, Georgia          111369
    ## 445  0500000US13115                      Floyd County, Georgia           96824
    ## 446  0500000US13117                    Forsyth County, Georgia          219880
    ## 447  0500000US13119                   Franklin County, Georgia           22514
    ## 448  0500000US13121                     Fulton County, Georgia         1021902
    ## 449  0500000US13123                     Gilmer County, Georgia           29922
    ## 450  0500000US13125                   Glascock County, Georgia            3009
    ## 451  0500000US13127                      Glynn County, Georgia           83974
    ## 452  0500000US13129                     Gordon County, Georgia           56790
    ## 453  0500000US13131                      Grady County, Georgia           24926
    ## 454  0500000US13133                     Greene County, Georgia           16976
    ## 455  0500000US13135                   Gwinnett County, Georgia          902298
    ## 456  0500000US13137                  Habersham County, Georgia           44289
    ## 457  0500000US13139                       Hall County, Georgia          195961
    ## 458  0500000US13141                    Hancock County, Georgia            8535
    ## 459  0500000US13143                   Haralson County, Georgia           28956
    ## 460  0500000US13145                     Harris County, Georgia           33590
    ## 461  0500000US13147                       Hart County, Georgia           25631
    ## 462  0500000US13149                      Heard County, Georgia           11677
    ## 463  0500000US13151                      Henry County, Georgia          221307
    ## 464  0500000US13153                    Houston County, Georgia          151682
    ## 465  0500000US13155                      Irwin County, Georgia            9268
    ## 466  0500000US13157                    Jackson County, Georgia           65755
    ## 467  0500000US13159                     Jasper County, Georgia           13784
    ## 468  0500000US13161                 Jeff Davis County, Georgia           14991
    ## 469  0500000US13163                  Jefferson County, Georgia           15772
    ## 470  0500000US13165                    Jenkins County, Georgia            8827
    ## 471  0500000US13167                    Johnson County, Georgia            9730
    ## 472  0500000US13169                      Jones County, Georgia           28548
    ## 473  0500000US13171                      Lamar County, Georgia           18513
    ## 474  0500000US13173                     Lanier County, Georgia           10366
    ## 475  0500000US13175                    Laurens County, Georgia           47418
    ## 476  0500000US13177                        Lee County, Georgia           29348
    ## 477  0500000US13179                    Liberty County, Georgia           62108
    ## 478  0500000US13181                    Lincoln County, Georgia            7799
    ## 479  0500000US13183                       Long County, Georgia           18156
    ## 480  0500000US13185                    Lowndes County, Georgia          114582
    ## 481  0500000US13187                    Lumpkin County, Georgia           31951
    ## 482  0500000US13189                   McDuffie County, Georgia           21498
    ## 483  0500000US13191                   McIntosh County, Georgia           14120
    ## 484  0500000US13193                      Macon County, Georgia           13480
    ## 485  0500000US13195                    Madison County, Georgia           28900
    ## 486  0500000US13197                     Marion County, Georgia            8484
    ## 487  0500000US13199                 Meriwether County, Georgia           21113
    ## 488  0500000US13201                     Miller County, Georgia            5836
    ## 489  0500000US13205                   Mitchell County, Georgia           22432
    ## 490  0500000US13207                     Monroe County, Georgia           27010
    ## 491  0500000US13209                 Montgomery County, Georgia            9036
    ## 492  0500000US13211                     Morgan County, Georgia           18235
    ## 493  0500000US13213                     Murray County, Georgia           39557
    ## 494  0500000US13215                   Muscogee County, Georgia          196670
    ## 495  0500000US13217                     Newton County, Georgia          106497
    ## 496  0500000US13219                     Oconee County, Georgia           37017
    ## 497  0500000US13221                 Oglethorpe County, Georgia           14784
    ## 498  0500000US13223                   Paulding County, Georgia          155840
    ## 499  0500000US13225                      Peach County, Georgia           26966
    ## 500  0500000US13227                    Pickens County, Georgia           30832
    ## 501  0500000US13229                     Pierce County, Georgia           19164
    ## 502  0500000US13231                       Pike County, Georgia           18082
    ## 503  0500000US13233                       Polk County, Georgia           41621
    ## 504  0500000US13235                    Pulaski County, Georgia           11295
    ## 505  0500000US13237                     Putnam County, Georgia           21503
    ## 506  0500000US13239                    Quitman County, Georgia            2276
    ## 507  0500000US13241                      Rabun County, Georgia           16457
    ## 508  0500000US13243                   Randolph County, Georgia            7087
    ## 509  0500000US13245                   Richmond County, Georgia          201463
    ## 510  0500000US13247                   Rockdale County, Georgia           89011
    ## 511  0500000US13249                     Schley County, Georgia            5211
    ## 512  0500000US13251                    Screven County, Georgia           13990
    ## 513  0500000US13253                   Seminole County, Georgia            8437
    ## 514  0500000US13255                   Spalding County, Georgia           64719
    ## 515  0500000US13257                   Stephens County, Georgia           25676
    ## 516  0500000US13259                    Stewart County, Georgia            6042
    ## 517  0500000US13261                     Sumter County, Georgia           30352
    ## 518  0500000US13263                     Talbot County, Georgia            6378
    ## 519  0500000US13265                 Taliaferro County, Georgia            1665
    ## 520  0500000US13267                   Tattnall County, Georgia           25353
    ## 521  0500000US13269                     Taylor County, Georgia            8193
    ## 522  0500000US13271                    Telfair County, Georgia           16115
    ## 523  0500000US13273                    Terrell County, Georgia            8859
    ## 524  0500000US13275                     Thomas County, Georgia           44730
    ## 525  0500000US13277                       Tift County, Georgia           40510
    ## 526  0500000US13279                     Toombs County, Georgia           27048
    ## 527  0500000US13281                      Towns County, Georgia           11417
    ## 528  0500000US13283                   Treutlen County, Georgia            6777
    ## 529  0500000US13285                      Troup County, Georgia           69774
    ## 530  0500000US13287                     Turner County, Georgia            7962
    ## 531  0500000US13289                     Twiggs County, Georgia            8284
    ## 532  0500000US13291                      Union County, Georgia           22775
    ## 533  0500000US13293                      Upson County, Georgia           26216
    ## 534  0500000US13295                     Walker County, Georgia           68824
    ## 535  0500000US13297                     Walton County, Georgia           90132
    ## 536  0500000US13299                       Ware County, Georgia           35599
    ## 537  0500000US13301                     Warren County, Georgia            5346
    ## 538  0500000US13303                 Washington County, Georgia           20461
    ## 539  0500000US13305                      Wayne County, Georgia           29767
    ## 540  0500000US13307                    Webster County, Georgia            2613
    ## 541  0500000US13309                    Wheeler County, Georgia            7939
    ## 542  0500000US13311                      White County, Georgia           28928
    ## 543  0500000US13313                  Whitfield County, Georgia          103849
    ## 544  0500000US13315                     Wilcox County, Georgia            8846
    ## 545  0500000US13317                     Wilkes County, Georgia            9884
    ## 546  0500000US13319                  Wilkinson County, Georgia            9078
    ## 547  0500000US13321                      Worth County, Georgia           20656
    ## 548  0500000US15001                      Hawaii County, Hawaii          197658
    ## 549  0500000US15003                    Honolulu County, Hawaii          987638
    ## 550  0500000US15005                     Kalawao County, Hawaii              75
    ## 551  0500000US15007                       Kauai County, Hawaii           71377
    ## 552  0500000US15009                        Maui County, Hawaii          165281
    ## 553  0500000US16001                          Ada County, Idaho          446052
    ## 554  0500000US16003                        Adams County, Idaho            4019
    ## 555  0500000US16005                      Bannock County, Idaho           85065
    ## 556  0500000US16007                    Bear Lake County, Idaho            5962
    ## 557  0500000US16009                      Benewah County, Idaho            9086
    ## 558  0500000US16011                      Bingham County, Idaho           45551
    ## 559  0500000US16013                       Blaine County, Idaho           21994
    ## 560  0500000US16015                        Boise County, Idaho            7163
    ## 561  0500000US16017                       Bonner County, Idaho           42711
    ## 562  0500000US16019                   Bonneville County, Idaho          112397
    ## 563  0500000US16021                     Boundary County, Idaho           11549
    ## 564  0500000US16023                        Butte County, Idaho            2602
    ## 565  0500000US16025                        Camas County, Idaho             886
    ## 566  0500000US16027                       Canyon County, Idaho          212230
    ## 567  0500000US16029                      Caribou County, Idaho            6918
    ## 568  0500000US16031                       Cassia County, Idaho           23615
    ## 569  0500000US16033                        Clark County, Idaho            1077
    ## 570  0500000US16035                   Clearwater County, Idaho            8640
    ## 571  0500000US16037                       Custer County, Idaho            4141
    ## 572  0500000US16039                       Elmore County, Idaho           26433
    ## 573  0500000US16041                     Franklin County, Idaho           13279
    ## 574  0500000US16043                      Fremont County, Idaho           12965
    ## 575  0500000US16045                          Gem County, Idaho           17052
    ## 576  0500000US16047                      Gooding County, Idaho           15169
    ## 577  0500000US16049                        Idaho County, Idaho           16337
    ## 578  0500000US16051                    Jefferson County, Idaho           27969
    ## 579  0500000US16053                       Jerome County, Idaho           23431
    ## 580  0500000US16055                     Kootenai County, Idaho          153605
    ## 581  0500000US16057                        Latah County, Idaho           39239
    ## 582  0500000US16059                        Lemhi County, Idaho            7798
    ## 583  0500000US16061                        Lewis County, Idaho            3845
    ## 584  0500000US16063                      Lincoln County, Idaho            5321
    ## 585  0500000US16065                      Madison County, Idaho           38705
    ## 586  0500000US16067                     Minidoka County, Idaho           20615
    ## 587  0500000US16069                    Nez Perce County, Idaho           40155
    ## 588  0500000US16071                       Oneida County, Idaho            4326
    ## 589  0500000US16073                       Owyhee County, Idaho           11455
    ## 590  0500000US16075                      Payette County, Idaho           23041
    ## 591  0500000US16077                        Power County, Idaho            7713
    ## 592  0500000US16079                     Shoshone County, Idaho           12526
    ## 593  0500000US16081                        Teton County, Idaho           11080
    ## 594  0500000US16083                   Twin Falls County, Idaho           83666
    ## 595  0500000US16085                       Valley County, Idaho           10401
    ## 596  0500000US16087                   Washington County, Idaho           10025
    ## 597  0500000US17001                     Adams County, Illinois           66427
    ## 598  0500000US17003                 Alexander County, Illinois            6532
    ## 599  0500000US17005                      Bond County, Illinois           16712
    ## 600  0500000US17007                     Boone County, Illinois           53606
    ## 601  0500000US17009                     Brown County, Illinois            6675
    ## 602  0500000US17011                    Bureau County, Illinois           33381
    ## 603  0500000US17013                   Calhoun County, Illinois            4858
    ## 604  0500000US17015                   Carroll County, Illinois           14562
    ## 605  0500000US17017                      Cass County, Illinois           12665
    ## 606  0500000US17019                 Champaign County, Illinois          209448
    ## 607  0500000US17021                 Christian County, Illinois           33231
    ## 608  0500000US17023                     Clark County, Illinois           15836
    ## 609  0500000US17025                      Clay County, Illinois           13338
    ## 610  0500000US17027                   Clinton County, Illinois           37628
    ## 611  0500000US17029                     Coles County, Illinois           51736
    ## 612  0500000US17031                      Cook County, Illinois         5223719
    ## 613  0500000US17033                  Crawford County, Illinois           19088
    ## 614  0500000US17035                Cumberland County, Illinois           10865
    ## 615  0500000US17037                    DeKalb County, Illinois          104200
    ## 616  0500000US17039                   De Witt County, Illinois           16042
    ## 617  0500000US17041                   Douglas County, Illinois           19714
    ## 618  0500000US17043                    DuPage County, Illinois          931743
    ## 619  0500000US17045                     Edgar County, Illinois           17539
    ## 620  0500000US17047                   Edwards County, Illinois            6507
    ## 621  0500000US17049                 Effingham County, Illinois           34174
    ## 622  0500000US17051                   Fayette County, Illinois           21724
    ## 623  0500000US17053                      Ford County, Illinois           13398
    ## 624  0500000US17055                  Franklin County, Illinois           39127
    ## 625  0500000US17057                    Fulton County, Illinois           35418
    ## 626  0500000US17059                  Gallatin County, Illinois            5157
    ## 627  0500000US17061                    Greene County, Illinois           13218
    ## 628  0500000US17063                    Grundy County, Illinois           50509
    ## 629  0500000US17065                  Hamilton County, Illinois            8221
    ## 630  0500000US17067                   Hancock County, Illinois           18112
    ## 631  0500000US17069                    Hardin County, Illinois            4009
    ## 632  0500000US17071                 Henderson County, Illinois            6884
    ## 633  0500000US17073                     Henry County, Illinois           49464
    ## 634  0500000US17075                  Iroquois County, Illinois           28169
    ## 635  0500000US17077                   Jackson County, Illinois           58551
    ## 636  0500000US17079                    Jasper County, Illinois            9598
    ## 637  0500000US17081                 Jefferson County, Illinois           38169
    ## 638  0500000US17083                    Jersey County, Illinois           22069
    ## 639  0500000US17085                Jo Daviess County, Illinois           21834
    ## 640  0500000US17087                   Johnson County, Illinois           12602
    ## 641  0500000US17089                      Kane County, Illinois          530839
    ## 642  0500000US17091                  Kankakee County, Illinois          111061
    ## 643  0500000US17093                   Kendall County, Illinois          124626
    ## 644  0500000US17095                      Knox County, Illinois           50999
    ## 645  0500000US17097                      Lake County, Illinois          703619
    ## 646  0500000US17099                   LaSalle County, Illinois          110401
    ## 647  0500000US17101                  Lawrence County, Illinois           16189
    ## 648  0500000US17103                       Lee County, Illinois           34527
    ## 649  0500000US17105                Livingston County, Illinois           36324
    ## 650  0500000US17107                     Logan County, Illinois           29207
    ## 651  0500000US17109                 McDonough County, Illinois           30875
    ## 652  0500000US17111                   McHenry County, Illinois          307789
    ## 653  0500000US17113                    McLean County, Illinois          173219
    ## 654  0500000US17115                     Macon County, Illinois          106512
    ## 655  0500000US17117                  Macoupin County, Illinois           45719
    ## 656  0500000US17119                   Madison County, Illinois          265670
    ## 657  0500000US17121                    Marion County, Illinois           38084
    ## 658  0500000US17123                  Marshall County, Illinois           11794
    ## 659  0500000US17125                     Mason County, Illinois           13778
    ## 660  0500000US17127                    Massac County, Illinois           14430
    ## 661  0500000US17129                    Menard County, Illinois           12367
    ## 662  0500000US17131                    Mercer County, Illinois           15693
    ## 663  0500000US17133                    Monroe County, Illinois           33936
    ## 664  0500000US17135                Montgomery County, Illinois           29009
    ## 665  0500000US17137                    Morgan County, Illinois           34426
    ## 666  0500000US17139                  Moultrie County, Illinois           14703
    ## 667  0500000US17141                      Ogle County, Illinois           51328
    ## 668  0500000US17143                    Peoria County, Illinois          184463
    ## 669  0500000US17145                     Perry County, Illinois           21384
    ## 670  0500000US17147                     Piatt County, Illinois           16427
    ## 671  0500000US17149                      Pike County, Illinois           15754
    ## 672  0500000US17151                      Pope County, Illinois            4249
    ## 673  0500000US17153                   Pulaski County, Illinois            5611
    ## 674  0500000US17155                    Putnam County, Illinois            5746
    ## 675  0500000US17157                  Randolph County, Illinois           32546
    ## 676  0500000US17159                  Richland County, Illinois           15881
    ## 677  0500000US17161               Rock Island County, Illinois          145275
    ## 678  0500000US17163                 St. Clair County, Illinois          263463
    ## 679  0500000US17165                    Saline County, Illinois           24231
    ## 680  0500000US17167                  Sangamon County, Illinois          197661
    ## 681  0500000US17169                  Schuyler County, Illinois            7064
    ## 682  0500000US17171                     Scott County, Illinois            5047
    ## 683  0500000US17173                    Shelby County, Illinois           21832
    ## 684  0500000US17175                     Stark County, Illinois            5500
    ## 685  0500000US17177                Stephenson County, Illinois           45433
    ## 686  0500000US17179                  Tazewell County, Illinois          133852
    ## 687  0500000US17181                     Union County, Illinois           17127
    ## 688  0500000US17183                 Vermilion County, Illinois           78407
    ## 689  0500000US17185                    Wabash County, Illinois           11573
    ## 690  0500000US17187                    Warren County, Illinois           17338
    ## 691  0500000US17189                Washington County, Illinois           14155
    ## 692  0500000US17191                     Wayne County, Illinois           16487
    ## 693  0500000US17193                     White County, Illinois           14025
    ## 694  0500000US17195                 Whiteside County, Illinois           56396
    ## 695  0500000US17197                      Will County, Illinois          688697
    ## 696  0500000US17199                Williamson County, Illinois           67299
    ## 697  0500000US17201                 Winnebago County, Illinois          286174
    ## 698  0500000US17203                  Woodford County, Illinois           38817
    ## 699  0500000US18001                      Adams County, Indiana           35195
    ## 700  0500000US18003                      Allen County, Indiana          370016
    ## 701  0500000US18005                Bartholomew County, Indiana           81893
    ## 702  0500000US18007                     Benton County, Indiana            8667
    ## 703  0500000US18009                  Blackford County, Indiana           12129
    ## 704  0500000US18011                      Boone County, Indiana           64321
    ## 705  0500000US18013                      Brown County, Indiana           15034
    ## 706  0500000US18015                    Carroll County, Indiana           19994
    ## 707  0500000US18017                       Cass County, Indiana           38084
    ## 708  0500000US18019                      Clark County, Indiana          115702
    ## 709  0500000US18021                       Clay County, Indiana           26268
    ## 710  0500000US18023                    Clinton County, Indiana           32301
    ## 711  0500000US18025                   Crawford County, Indiana           10581
    ## 712  0500000US18027                    Daviess County, Indiana           32937
    ## 713  0500000US18029                   Dearborn County, Indiana           49501
    ## 714  0500000US18031                    Decatur County, Indiana           26552
    ## 715  0500000US18033                     DeKalb County, Indiana           42704
    ## 716  0500000US18035                   Delaware County, Indiana          115616
    ## 717  0500000US18037                     Dubois County, Indiana           42418
    ## 718  0500000US18039                    Elkhart County, Indiana          203604
    ## 719  0500000US18041                    Fayette County, Indiana           23259
    ## 720  0500000US18043                      Floyd County, Indiana           76809
    ## 721  0500000US18045                   Fountain County, Indiana           16486
    ## 722  0500000US18047                   Franklin County, Indiana           22842
    ## 723  0500000US18049                     Fulton County, Indiana           20212
    ## 724  0500000US18051                     Gibson County, Indiana           33596
    ## 725  0500000US18053                      Grant County, Indiana           66944
    ## 726  0500000US18055                     Greene County, Indiana           32295
    ## 727  0500000US18057                   Hamilton County, Indiana          316095
    ## 728  0500000US18059                    Hancock County, Indiana           73830
    ## 729  0500000US18061                   Harrison County, Indiana           39712
    ## 730  0500000US18063                  Hendricks County, Indiana          160940
    ## 731  0500000US18065                      Henry County, Indiana           48483
    ## 732  0500000US18067                     Howard County, Indiana           82387
    ## 733  0500000US18069                 Huntington County, Indiana           36378
    ## 734  0500000US18071                    Jackson County, Indiana           43938
    ## 735  0500000US18073                     Jasper County, Indiana           33449
    ## 736  0500000US18075                        Jay County, Indiana           20993
    ## 737  0500000US18077                  Jefferson County, Indiana           32237
    ## 738  0500000US18079                   Jennings County, Indiana           27727
    ## 739  0500000US18081                    Johnson County, Indiana          151564
    ## 740  0500000US18083                       Knox County, Indiana           37409
    ## 741  0500000US18085                  Kosciusko County, Indiana           78806
    ## 742  0500000US18087                   LaGrange County, Indiana           38942
    ## 743  0500000US18089                       Lake County, Indiana          486849
    ## 744  0500000US18091                    LaPorte County, Indiana          110552
    ## 745  0500000US18093                   Lawrence County, Indiana           45619
    ## 746  0500000US18095                    Madison County, Indiana          129505
    ## 747  0500000US18097                     Marion County, Indiana          944523
    ## 748  0500000US18099                   Marshall County, Indiana           46595
    ## 749  0500000US18101                     Martin County, Indiana           10210
    ## 750  0500000US18103                      Miami County, Indiana           35901
    ## 751  0500000US18105                     Monroe County, Indiana          145403
    ## 752  0500000US18107                 Montgomery County, Indiana           38276
    ## 753  0500000US18109                     Morgan County, Indiana           69727
    ## 754  0500000US18111                     Newton County, Indiana           14018
    ## 755  0500000US18113                      Noble County, Indiana           47451
    ## 756  0500000US18115                       Ohio County, Indiana            5887
    ## 757  0500000US18117                     Orange County, Indiana           19547
    ## 758  0500000US18119                       Owen County, Indiana           20878
    ## 759  0500000US18121                      Parke County, Indiana           16996
    ## 760  0500000US18123                      Perry County, Indiana           19141
    ## 761  0500000US18125                       Pike County, Indiana           12411
    ## 762  0500000US18127                     Porter County, Indiana          168041
    ## 763  0500000US18129                      Posey County, Indiana           25589
    ## 764  0500000US18131                    Pulaski County, Indiana           12660
    ## 765  0500000US18133                     Putnam County, Indiana           37559
    ## 766  0500000US18135                   Randolph County, Indiana           25076
    ## 767  0500000US18137                     Ripley County, Indiana           28425
    ## 768  0500000US18139                       Rush County, Indiana           16704
    ## 769  0500000US18141                 St. Joseph County, Indiana          269240
    ## 770  0500000US18143                      Scott County, Indiana           23743
    ## 771  0500000US18145                     Shelby County, Indiana           44399
    ## 772  0500000US18147                    Spencer County, Indiana           20526
    ## 773  0500000US18149                     Starke County, Indiana           22941
    ## 774  0500000US18151                    Steuben County, Indiana           34474
    ## 775  0500000US18153                   Sullivan County, Indiana           20792
    ## 776  0500000US18155                Switzerland County, Indiana           10628
    ## 777  0500000US18157                 Tippecanoe County, Indiana          189294
    ## 778  0500000US18159                     Tipton County, Indiana           15218
    ## 779  0500000US18161                      Union County, Indiana            7153
    ## 780  0500000US18163                Vanderburgh County, Indiana          181313
    ## 781  0500000US18165                 Vermillion County, Indiana           15560
    ## 782  0500000US18167                       Vigo County, Indiana          107693
    ## 783  0500000US18169                     Wabash County, Indiana           31631
    ## 784  0500000US18171                     Warren County, Indiana            8247
    ## 785  0500000US18173                    Warrick County, Indiana           61928
    ## 786  0500000US18175                 Washington County, Indiana           27827
    ## 787  0500000US18177                      Wayne County, Indiana           66613
    ## 788  0500000US18179                      Wells County, Indiana           27947
    ## 789  0500000US18181                      White County, Indiana           24217
    ## 790  0500000US18183                    Whitley County, Indiana           33649
    ## 791  0500000US19001                         Adair County, Iowa            7124
    ## 792  0500000US19003                         Adams County, Iowa            3726
    ## 793  0500000US19005                     Allamakee County, Iowa           13880
    ## 794  0500000US19007                     Appanoose County, Iowa           12510
    ## 795  0500000US19009                       Audubon County, Iowa            5637
    ## 796  0500000US19011                        Benton County, Iowa           25626
    ## 797  0500000US19013                    Black Hawk County, Iowa          133009
    ## 798  0500000US19015                         Boone County, Iowa           26399
    ## 799  0500000US19017                        Bremer County, Iowa           24782
    ## 800  0500000US19019                      Buchanan County, Iowa           21125
    ## 801  0500000US19021                   Buena Vista County, Iowa           20260
    ## 802  0500000US19023                        Butler County, Iowa           14735
    ## 803  0500000US19025                       Calhoun County, Iowa            9780
    ## 804  0500000US19027                       Carroll County, Iowa           20344
    ## 805  0500000US19029                          Cass County, Iowa           13191
    ## 806  0500000US19031                         Cedar County, Iowa           18445
    ## 807  0500000US19033                   Cerro Gordo County, Iowa           42984
    ## 808  0500000US19035                      Cherokee County, Iowa           11468
    ## 809  0500000US19037                     Chickasaw County, Iowa           12099
    ## 810  0500000US19039                        Clarke County, Iowa            9282
    ## 811  0500000US19041                          Clay County, Iowa           16313
    ## 812  0500000US19043                       Clayton County, Iowa           17672
    ## 813  0500000US19045                       Clinton County, Iowa           47218
    ## 814  0500000US19047                      Crawford County, Iowa           17132
    ## 815  0500000US19049                        Dallas County, Iowa           84002
    ## 816  0500000US19051                         Davis County, Iowa            8885
    ## 817  0500000US19053                       Decatur County, Iowa            8044
    ## 818  0500000US19055                      Delaware County, Iowa           17258
    ## 819  0500000US19057                    Des Moines County, Iowa           39600
    ## 820  0500000US19059                     Dickinson County, Iowa           17056
    ## 821  0500000US19061                       Dubuque County, Iowa           96802
    ## 822  0500000US19063                         Emmet County, Iowa            9551
    ## 823  0500000US19065                       Fayette County, Iowa           19929
    ## 824  0500000US19067                         Floyd County, Iowa           15858
    ## 825  0500000US19069                      Franklin County, Iowa           10245
    ## 826  0500000US19071                       Fremont County, Iowa            6968
    ## 827  0500000US19073                        Greene County, Iowa            9003
    ## 828  0500000US19075                        Grundy County, Iowa           12341
    ## 829  0500000US19077                       Guthrie County, Iowa           10674
    ## 830  0500000US19079                      Hamilton County, Iowa           15110
    ## 831  0500000US19081                       Hancock County, Iowa           10888
    ## 832  0500000US19083                        Hardin County, Iowa           17127
    ## 833  0500000US19085                      Harrison County, Iowa           14143
    ## 834  0500000US19087                         Henry County, Iowa           19926
    ## 835  0500000US19089                        Howard County, Iowa            9264
    ## 836  0500000US19091                      Humboldt County, Iowa            9566
    ## 837  0500000US19093                           Ida County, Iowa            6916
    ## 838  0500000US19095                          Iowa County, Iowa           16207
    ## 839  0500000US19097                       Jackson County, Iowa           19395
    ## 840  0500000US19099                        Jasper County, Iowa           36891
    ## 841  0500000US19101                     Jefferson County, Iowa           18077
    ## 842  0500000US19103                       Johnson County, Iowa          147001
    ## 843  0500000US19105                         Jones County, Iowa           20568
    ## 844  0500000US19107                        Keokuk County, Iowa           10200
    ## 845  0500000US19109                       Kossuth County, Iowa           15075
    ## 846  0500000US19111                           Lee County, Iowa           34541
    ## 847  0500000US19113                          Linn County, Iowa          222121
    ## 848  0500000US19115                        Louisa County, Iowa           11223
    ## 849  0500000US19117                         Lucas County, Iowa            8597
    ## 850  0500000US19119                          Lyon County, Iowa           11769
    ## 851  0500000US19121                       Madison County, Iowa           15890
    ## 852  0500000US19123                       Mahaska County, Iowa           22208
    ## 853  0500000US19125                        Marion County, Iowa           33207
    ## 854  0500000US19127                      Marshall County, Iowa           40271
    ## 855  0500000US19129                         Mills County, Iowa           14957
    ## 856  0500000US19131                      Mitchell County, Iowa           10631
    ## 857  0500000US19133                        Monona County, Iowa            8796
    ## 858  0500000US19135                        Monroe County, Iowa            7863
    ## 859  0500000US19137                    Montgomery County, Iowa           10155
    ## 860  0500000US19139                     Muscatine County, Iowa           42950
    ## 861  0500000US19141                       O'Brien County, Iowa           13911
    ## 862  0500000US19143                       Osceola County, Iowa            6115
    ## 863  0500000US19145                          Page County, Iowa           15363
    ## 864  0500000US19147                     Palo Alto County, Iowa            9055
    ## 865  0500000US19149                      Plymouth County, Iowa           25039
    ## 866  0500000US19151                    Pocahontas County, Iowa            6898
    ## 867  0500000US19153                          Polk County, Iowa          474274
    ## 868  0500000US19155                 Pottawattamie County, Iowa           93503
    ## 869  0500000US19157                     Poweshiek County, Iowa           18605
    ## 870  0500000US19159                      Ringgold County, Iowa            4984
    ## 871  0500000US19161                           Sac County, Iowa            9868
    ## 872  0500000US19163                         Scott County, Iowa          172288
    ## 873  0500000US19165                        Shelby County, Iowa           11694
    ## 874  0500000US19167                         Sioux County, Iowa           34825
    ## 875  0500000US19169                         Story County, Iowa           96922
    ## 876  0500000US19171                          Tama County, Iowa           17136
    ## 877  0500000US19173                        Taylor County, Iowa            6201
    ## 878  0500000US19175                         Union County, Iowa           12453
    ## 879  0500000US19177                     Van Buren County, Iowa            7223
    ## 880  0500000US19179                       Wapello County, Iowa           35315
    ## 881  0500000US19181                        Warren County, Iowa           49361
    ## 882  0500000US19183                    Washington County, Iowa           22143
    ## 883  0500000US19185                         Wayne County, Iowa            6413
    ## 884  0500000US19187                       Webster County, Iowa           36757
    ## 885  0500000US19189                     Winnebago County, Iowa           10571
    ## 886  0500000US19191                    Winneshiek County, Iowa           20401
    ## 887  0500000US19193                      Woodbury County, Iowa          102398
    ## 888  0500000US19195                         Worth County, Iowa            7489
    ## 889  0500000US19197                        Wright County, Iowa           12804
    ## 890  0500000US20001                       Allen County, Kansas           12630
    ## 891  0500000US20003                    Anderson County, Kansas            7852
    ## 892  0500000US20005                    Atchison County, Kansas           16363
    ## 893  0500000US20007                      Barber County, Kansas            4733
    ## 894  0500000US20009                      Barton County, Kansas           26791
    ## 895  0500000US20011                     Bourbon County, Kansas           14702
    ## 896  0500000US20013                       Brown County, Kansas            9664
    ## 897  0500000US20015                      Butler County, Kansas           66468
    ## 898  0500000US20017                       Chase County, Kansas            2645
    ## 899  0500000US20019                  Chautauqua County, Kansas            3367
    ## 900  0500000US20021                    Cherokee County, Kansas           20331
    ## 901  0500000US20023                    Cheyenne County, Kansas            2677
    ## 902  0500000US20025                       Clark County, Kansas            2053
    ## 903  0500000US20027                        Clay County, Kansas            8142
    ## 904  0500000US20029                       Cloud County, Kansas            9060
    ## 905  0500000US20031                      Coffey County, Kansas            8296
    ## 906  0500000US20033                    Comanche County, Kansas            1780
    ## 907  0500000US20035                      Cowley County, Kansas           35591
    ## 908  0500000US20037                    Crawford County, Kansas           39108
    ## 909  0500000US20039                     Decatur County, Kansas            2881
    ## 910  0500000US20041                   Dickinson County, Kansas           19004
    ## 911  0500000US20043                    Doniphan County, Kansas            7736
    ## 912  0500000US20045                     Douglas County, Kansas          119319
    ## 913  0500000US20047                     Edwards County, Kansas            2925
    ## 914  0500000US20049                         Elk County, Kansas            2562
    ## 915  0500000US20051                       Ellis County, Kansas           28878
    ## 916  0500000US20053                   Ellsworth County, Kansas            6293
    ## 917  0500000US20055                      Finney County, Kansas           36957
    ## 918  0500000US20057                        Ford County, Kansas           34484
    ## 919  0500000US20059                    Franklin County, Kansas           25563
    ## 920  0500000US20061                       Geary County, Kansas           34895
    ## 921  0500000US20063                        Gove County, Kansas            2619
    ## 922  0500000US20065                      Graham County, Kansas            2545
    ## 923  0500000US20067                       Grant County, Kansas            7616
    ## 924  0500000US20069                        Gray County, Kansas            6037
    ## 925  0500000US20071                     Greeley County, Kansas            1200
    ## 926  0500000US20073                   Greenwood County, Kansas            6156
    ## 927  0500000US20075                    Hamilton County, Kansas            2616
    ## 928  0500000US20077                      Harper County, Kansas            5673
    ## 929  0500000US20079                      Harvey County, Kansas           34555
    ## 930  0500000US20081                     Haskell County, Kansas            4047
    ## 931  0500000US20083                    Hodgeman County, Kansas            1842
    ## 932  0500000US20085                     Jackson County, Kansas           13318
    ## 933  0500000US20087                   Jefferson County, Kansas           18888
    ## 934  0500000US20089                      Jewell County, Kansas            2916
    ## 935  0500000US20091                     Johnson County, Kansas          585502
    ## 936  0500000US20093                      Kearny County, Kansas            3932
    ## 937  0500000US20095                     Kingman County, Kansas            7470
    ## 938  0500000US20097                       Kiowa County, Kansas            2526
    ## 939  0500000US20099                     Labette County, Kansas           20367
    ## 940  0500000US20101                        Lane County, Kansas            1642
    ## 941  0500000US20103                 Leavenworth County, Kansas           80042
    ## 942  0500000US20105                     Lincoln County, Kansas            3097
    ## 943  0500000US20107                        Linn County, Kansas            9635
    ## 944  0500000US20109                       Logan County, Kansas            2810
    ## 945  0500000US20111                        Lyon County, Kansas           33299
    ## 946  0500000US20113                   McPherson County, Kansas           28630
    ## 947  0500000US20115                      Marion County, Kansas           12032
    ## 948  0500000US20117                    Marshall County, Kansas            9798
    ## 949  0500000US20119                       Meade County, Kansas            4261
    ## 950  0500000US20121                       Miami County, Kansas           33127
    ## 951  0500000US20123                    Mitchell County, Kansas            6222
    ## 952  0500000US20125                  Montgomery County, Kansas           32970
    ## 953  0500000US20127                      Morris County, Kansas            5566
    ## 954  0500000US20129                      Morton County, Kansas            2838
    ## 955  0500000US20131                      Nemaha County, Kansas           10104
    ## 956  0500000US20133                      Neosho County, Kansas           16125
    ## 957  0500000US20135                        Ness County, Kansas            2955
    ## 958  0500000US20137                      Norton County, Kansas            5486
    ## 959  0500000US20139                       Osage County, Kansas           15882
    ## 960  0500000US20141                     Osborne County, Kansas            3603
    ## 961  0500000US20143                      Ottawa County, Kansas            5902
    ## 962  0500000US20145                      Pawnee County, Kansas            6709
    ## 963  0500000US20147                    Phillips County, Kansas            5408
    ## 964  0500000US20149                Pottawatomie County, Kansas           23545
    ## 965  0500000US20151                       Pratt County, Kansas            9582
    ## 966  0500000US20153                     Rawlins County, Kansas            2509
    ## 967  0500000US20155                        Reno County, Kansas           63101
    ## 968  0500000US20157                    Republic County, Kansas            4686
    ## 969  0500000US20159                        Rice County, Kansas            9762
    ## 970  0500000US20161                       Riley County, Kansas           75296
    ## 971  0500000US20163                       Rooks County, Kansas            5118
    ## 972  0500000US20165                        Rush County, Kansas            3102
    ## 973  0500000US20167                     Russell County, Kansas            6977
    ## 974  0500000US20169                      Saline County, Kansas           54977
    ## 975  0500000US20171                       Scott County, Kansas            4949
    ## 976  0500000US20173                    Sedgwick County, Kansas          512064
    ## 977  0500000US20175                      Seward County, Kansas           22692
    ## 978  0500000US20177                     Shawnee County, Kansas          178284
    ## 979  0500000US20179                    Sheridan County, Kansas            2506
    ## 980  0500000US20181                     Sherman County, Kansas            5966
    ## 981  0500000US20183                       Smith County, Kansas            3663
    ## 982  0500000US20185                    Stafford County, Kansas            4214
    ## 983  0500000US20187                     Stanton County, Kansas            2063
    ## 984  0500000US20189                     Stevens County, Kansas            5686
    ## 985  0500000US20191                      Sumner County, Kansas           23208
    ## 986  0500000US20193                      Thomas County, Kansas            7824
    ## 987  0500000US20195                       Trego County, Kansas            2858
    ## 988  0500000US20197                   Wabaunsee County, Kansas            6888
    ## 989  0500000US20199                     Wallace County, Kansas            1575
    ## 990  0500000US20201                  Washington County, Kansas            5525
    ## 991  0500000US20203                     Wichita County, Kansas            2143
    ## 992  0500000US20205                      Wilson County, Kansas            8780
    ## 993  0500000US20207                     Woodson County, Kansas            3170
    ## 994  0500000US20209                   Wyandotte County, Kansas          164345
    ## 995  0500000US21001                     Adair County, Kentucky           19241
    ## 996  0500000US21003                     Allen County, Kentucky           20794
    ## 997  0500000US21005                  Anderson County, Kentucky           22214
    ## 998  0500000US21007                   Ballard County, Kentucky            8090
    ## 999  0500000US21009                    Barren County, Kentucky           43680
    ## 1000 0500000US21011                      Bath County, Kentucky           12268
    ## 1001 0500000US21013                      Bell County, Kentucky           27188
    ## 1002 0500000US21015                     Boone County, Kentucky          129095
    ## 1003 0500000US21017                   Bourbon County, Kentucky           20144
    ## 1004 0500000US21019                      Boyd County, Kentucky           48091
    ## 1005 0500000US21021                     Boyle County, Kentucky           29913
    ## 1006 0500000US21023                   Bracken County, Kentucky            8306
    ## 1007 0500000US21025                 Breathitt County, Kentucky           13116
    ## 1008 0500000US21027              Breckinridge County, Kentucky           20080
    ## 1009 0500000US21029                   Bullitt County, Kentucky           79466
    ## 1010 0500000US21031                    Butler County, Kentucky           12745
    ## 1011 0500000US21033                  Caldwell County, Kentucky           12727
    ## 1012 0500000US21035                  Calloway County, Kentucky           38776
    ## 1013 0500000US21037                  Campbell County, Kentucky           92267
    ## 1014 0500000US21039                  Carlisle County, Kentucky            4841
    ## 1015 0500000US21041                   Carroll County, Kentucky           10711
    ## 1016 0500000US21043                    Carter County, Kentucky           27290
    ## 1017 0500000US21045                     Casey County, Kentucky           15796
    ## 1018 0500000US21047                 Christian County, Kentucky           72263
    ## 1019 0500000US21049                     Clark County, Kentucky           35872
    ## 1020 0500000US21051                      Clay County, Kentucky           20621
    ## 1021 0500000US21053                   Clinton County, Kentucky           10211
    ## 1022 0500000US21055                Crittenden County, Kentucky            9083
    ## 1023 0500000US21057                Cumberland County, Kentucky            6713
    ## 1024 0500000US21059                   Daviess County, Kentucky           99937
    ## 1025 0500000US21061                  Edmonson County, Kentucky           12122
    ## 1026 0500000US21063                   Elliott County, Kentucky            7517
    ## 1027 0500000US21065                    Estill County, Kentucky           14313
    ## 1028 0500000US21067                   Fayette County, Kentucky          318734
    ## 1029 0500000US21069                   Fleming County, Kentucky           14479
    ## 1030 0500000US21071                     Floyd County, Kentucky           36926
    ## 1031 0500000US21073                  Franklin County, Kentucky           50296
    ## 1032 0500000US21075                    Fulton County, Kentucky            6210
    ## 1033 0500000US21077                  Gallatin County, Kentucky            8703
    ## 1034 0500000US21079                   Garrard County, Kentucky           17328
    ## 1035 0500000US21081                     Grant County, Kentucky           24915
    ## 1036 0500000US21083                    Graves County, Kentucky           37294
    ## 1037 0500000US21085                   Grayson County, Kentucky           26178
    ## 1038 0500000US21087                     Green County, Kentucky           11023
    ## 1039 0500000US21089                   Greenup County, Kentucky           35765
    ## 1040 0500000US21091                   Hancock County, Kentucky            8719
    ## 1041 0500000US21093                    Hardin County, Kentucky          108095
    ## 1042 0500000US21095                    Harlan County, Kentucky           27134
    ## 1043 0500000US21097                  Harrison County, Kentucky           18668
    ## 1044 0500000US21099                      Hart County, Kentucky           18627
    ## 1045 0500000US21101                 Henderson County, Kentucky           46137
    ## 1046 0500000US21103                     Henry County, Kentucky           15814
    ## 1047 0500000US21105                   Hickman County, Kentucky            4568
    ## 1048 0500000US21107                   Hopkins County, Kentucky           45664
    ## 1049 0500000US21109                   Jackson County, Kentucky           13373
    ## 1050 0500000US21111                 Jefferson County, Kentucky          767154
    ## 1051 0500000US21113                 Jessamine County, Kentucky           52422
    ## 1052 0500000US21115                   Johnson County, Kentucky           22843
    ## 1053 0500000US21117                    Kenton County, Kentucky          164688
    ## 1054 0500000US21119                     Knott County, Kentucky           15513
    ## 1055 0500000US21121                      Knox County, Kentucky           31467
    ## 1056 0500000US21123                     Larue County, Kentucky           14156
    ## 1057 0500000US21125                    Laurel County, Kentucky           60180
    ## 1058 0500000US21127                  Lawrence County, Kentucky           15783
    ## 1059 0500000US21129                       Lee County, Kentucky            6751
    ## 1060 0500000US21131                    Leslie County, Kentucky           10472
    ## 1061 0500000US21133                   Letcher County, Kentucky           22676
    ## 1062 0500000US21135                     Lewis County, Kentucky           13490
    ## 1063 0500000US21137                   Lincoln County, Kentucky           24458
    ## 1064 0500000US21139                Livingston County, Kentucky            9263
    ## 1065 0500000US21141                     Logan County, Kentucky           26849
    ## 1066 0500000US21143                      Lyon County, Kentucky            8186
    ## 1067 0500000US21145                 McCracken County, Kentucky           65284
    ## 1068 0500000US21147                  McCreary County, Kentucky           17635
    ## 1069 0500000US21149                    McLean County, Kentucky            9331
    ## 1070 0500000US21151                   Madison County, Kentucky           89700
    ## 1071 0500000US21153                  Magoffin County, Kentucky           12666
    ## 1072 0500000US21155                    Marion County, Kentucky           19232
    ## 1073 0500000US21157                  Marshall County, Kentucky           31166
    ## 1074 0500000US21159                    Martin County, Kentucky           11919
    ## 1075 0500000US21161                     Mason County, Kentucky           17153
    ## 1076 0500000US21163                     Meade County, Kentucky           28326
    ## 1077 0500000US21165                   Menifee County, Kentucky            6405
    ## 1078 0500000US21167                    Mercer County, Kentucky           21516
    ## 1079 0500000US21169                  Metcalfe County, Kentucky           10004
    ## 1080 0500000US21171                    Monroe County, Kentucky           10634
    ## 1081 0500000US21173                Montgomery County, Kentucky           27759
    ## 1082 0500000US21175                    Morgan County, Kentucky           13285
    ## 1083 0500000US21177                Muhlenberg County, Kentucky           31081
    ## 1084 0500000US21179                    Nelson County, Kentucky           45388
    ## 1085 0500000US21181                  Nicholas County, Kentucky            7100
    ## 1086 0500000US21183                      Ohio County, Kentucky           24071
    ## 1087 0500000US21185                    Oldham County, Kentucky           65374
    ## 1088 0500000US21187                      Owen County, Kentucky           10741
    ## 1089 0500000US21189                    Owsley County, Kentucky            4463
    ## 1090 0500000US21191                 Pendleton County, Kentucky           14520
    ## 1091 0500000US21193                     Perry County, Kentucky           26917
    ## 1092 0500000US21195                      Pike County, Kentucky           60483
    ## 1093 0500000US21197                    Powell County, Kentucky           12321
    ## 1094 0500000US21199                   Pulaski County, Kentucky           64145
    ## 1095 0500000US21201                 Robertson County, Kentucky            2143
    ## 1096 0500000US21203                Rockcastle County, Kentucky           16827
    ## 1097 0500000US21205                     Rowan County, Kentucky           24499
    ## 1098 0500000US21207                   Russell County, Kentucky           17760
    ## 1099 0500000US21209                     Scott County, Kentucky           53517
    ## 1100 0500000US21211                    Shelby County, Kentucky           46786
    ## 1101 0500000US21213                   Simpson County, Kentucky           18063
    ## 1102 0500000US21215                   Spencer County, Kentucky           18246
    ## 1103 0500000US21217                    Taylor County, Kentucky           25500
    ## 1104 0500000US21219                      Todd County, Kentucky           12350
    ## 1105 0500000US21221                     Trigg County, Kentucky           14344
    ## 1106 0500000US21223                   Trimble County, Kentucky            8637
    ## 1107 0500000US21225                     Union County, Kentucky           14802
    ## 1108 0500000US21227                    Warren County, Kentucky          126427
    ## 1109 0500000US21229                Washington County, Kentucky           12019
    ## 1110 0500000US21231                     Wayne County, Kentucky           20609
    ## 1111 0500000US21233                   Webster County, Kentucky           13155
    ## 1112 0500000US21235                   Whitley County, Kentucky           36089
    ## 1113 0500000US21237                     Wolfe County, Kentucky            7223
    ## 1114 0500000US21239                  Woodford County, Kentucky           26097
    ## 1115 0500000US22001                   Acadia Parish, Louisiana           62568
    ## 1116 0500000US22003                    Allen Parish, Louisiana           25661
    ## 1117 0500000US22005                Ascension Parish, Louisiana          121176
    ## 1118 0500000US22007               Assumption Parish, Louisiana           22714
    ## 1119 0500000US22009                Avoyelles Parish, Louisiana           40882
    ## 1120 0500000US22011               Beauregard Parish, Louisiana           36769
    ## 1121 0500000US22013                Bienville Parish, Louisiana           13668
    ## 1122 0500000US22015                  Bossier Parish, Louisiana          126131
    ## 1123 0500000US22017                    Caddo Parish, Louisiana          248361
    ## 1124 0500000US22019                Calcasieu Parish, Louisiana          200182
    ## 1125 0500000US22021                 Caldwell Parish, Louisiana            9996
    ## 1126 0500000US22023                  Cameron Parish, Louisiana            6868
    ## 1127 0500000US22025                Catahoula Parish, Louisiana            9893
    ## 1128 0500000US22027                Claiborne Parish, Louisiana           16153
    ## 1129 0500000US22029                Concordia Parish, Louisiana           20021
    ## 1130 0500000US22031                  De Soto Parish, Louisiana           27216
    ## 1131 0500000US22033         East Baton Rouge Parish, Louisiana          444094
    ## 1132 0500000US22035             East Carroll Parish, Louisiana            7225
    ## 1133 0500000US22037           East Feliciana Parish, Louisiana           19499
    ## 1134 0500000US22039               Evangeline Parish, Louisiana           33636
    ## 1135 0500000US22041                 Franklin Parish, Louisiana           20322
    ## 1136 0500000US22043                    Grant Parish, Louisiana           22348
    ## 1137 0500000US22045                   Iberia Parish, Louisiana           72691
    ## 1138 0500000US22047                Iberville Parish, Louisiana           32956
    ## 1139 0500000US22049                  Jackson Parish, Louisiana           15926
    ## 1140 0500000US22051                Jefferson Parish, Louisiana          435300
    ## 1141 0500000US22053          Jefferson Davis Parish, Louisiana           31467
    ## 1142 0500000US22055                Lafayette Parish, Louisiana          240091
    ## 1143 0500000US22057                Lafourche Parish, Louisiana           98214
    ## 1144 0500000US22059                  LaSalle Parish, Louisiana           14949
    ## 1145 0500000US22061                  Lincoln Parish, Louisiana           47356
    ## 1146 0500000US22063               Livingston Parish, Louisiana          138111
    ## 1147 0500000US22065                  Madison Parish, Louisiana           11472
    ## 1148 0500000US22067                Morehouse Parish, Louisiana           25992
    ## 1149 0500000US22069             Natchitoches Parish, Louisiana           38963
    ## 1150 0500000US22071                  Orleans Parish, Louisiana          389648
    ## 1151 0500000US22073                 Ouachita Parish, Louisiana          156075
    ## 1152 0500000US22075              Plaquemines Parish, Louisiana           23373
    ## 1153 0500000US22077            Pointe Coupee Parish, Louisiana           22158
    ## 1154 0500000US22079                  Rapides Parish, Louisiana          131546
    ## 1155 0500000US22081                Red River Parish, Louisiana            8618
    ## 1156 0500000US22083                 Richland Parish, Louisiana           20474
    ## 1157 0500000US22085                   Sabine Parish, Louisiana           24088
    ## 1158 0500000US22087              St. Bernard Parish, Louisiana           45694
    ## 1159 0500000US22089              St. Charles Parish, Louisiana           52724
    ## 1160 0500000US22091               St. Helena Parish, Louisiana           10411
    ## 1161 0500000US22093                St. James Parish, Louisiana           21357
    ## 1162 0500000US22095     St. John the Baptist Parish, Louisiana           43446
    ## 1163 0500000US22097               St. Landry Parish, Louisiana           83449
    ## 1164 0500000US22099               St. Martin Parish, Louisiana           53752
    ## 1165 0500000US22101                 St. Mary Parish, Louisiana           51734
    ## 1166 0500000US22103              St. Tammany Parish, Louisiana          252093
    ## 1167 0500000US22105               Tangipahoa Parish, Louisiana          130504
    ## 1168 0500000US22107                   Tensas Parish, Louisiana            4666
    ## 1169 0500000US22109               Terrebonne Parish, Louisiana          112587
    ## 1170 0500000US22111                    Union Parish, Louisiana           22475
    ## 1171 0500000US22113                Vermilion Parish, Louisiana           59867
    ## 1172 0500000US22115                   Vernon Parish, Louisiana           51007
    ## 1173 0500000US22117               Washington Parish, Louisiana           46457
    ## 1174 0500000US22119                  Webster Parish, Louisiana           39631
    ## 1175 0500000US22121         West Baton Rouge Parish, Louisiana           25860
    ## 1176 0500000US22123             West Carroll Parish, Louisiana           11180
    ## 1177 0500000US22125           West Feliciana Parish, Louisiana           15377
    ## 1178 0500000US22127                     Winn Parish, Louisiana           14494
    ## 1179 0500000US23001                 Androscoggin County, Maine          107444
    ## 1180 0500000US23003                    Aroostook County, Maine           68269
    ## 1181 0500000US23005                   Cumberland County, Maine          290944
    ## 1182 0500000US23007                     Franklin County, Maine           30019
    ## 1183 0500000US23009                      Hancock County, Maine           54541
    ## 1184 0500000US23011                     Kennebec County, Maine          121545
    ## 1185 0500000US23013                         Knox County, Maine           39823
    ## 1186 0500000US23015                      Lincoln County, Maine           34067
    ## 1187 0500000US23017                       Oxford County, Maine           57325
    ## 1188 0500000US23019                    Penobscot County, Maine          151748
    ## 1189 0500000US23021                  Piscataquis County, Maine           16887
    ## 1190 0500000US23023                    Sagadahoc County, Maine           35277
    ## 1191 0500000US23025                     Somerset County, Maine           50710
    ## 1192 0500000US23027                        Waldo County, Maine           39418
    ## 1193 0500000US23029                   Washington County, Maine           31694
    ## 1194 0500000US23031                         York County, Maine          203102
    ## 1195 0500000US24001                  Allegany County, Maryland           71977
    ## 1196 0500000US24003              Anne Arundel County, Maryland          567696
    ## 1197 0500000US24005                 Baltimore County, Maryland          827625
    ## 1198 0500000US24009                   Calvert County, Maryland           91082
    ## 1199 0500000US24011                  Caroline County, Maryland           32875
    ## 1200 0500000US24013                   Carroll County, Maryland          167522
    ## 1201 0500000US24015                     Cecil County, Maryland          102517
    ## 1202 0500000US24017                   Charles County, Maryland          157671
    ## 1203 0500000US24019                Dorchester County, Maryland           32261
    ## 1204 0500000US24021                 Frederick County, Maryland          248472
    ## 1205 0500000US24023                   Garrett County, Maryland           29376
    ## 1206 0500000US24025                   Harford County, Maryland          251025
    ## 1207 0500000US24027                    Howard County, Maryland          315327
    ## 1208 0500000US24029                      Kent County, Maryland           19593
    ## 1209 0500000US24031                Montgomery County, Maryland         1040133
    ## 1210 0500000US24033           Prince George's County, Maryland          906202
    ## 1211 0500000US24035              Queen Anne's County, Maryland           49355
    ## 1212 0500000US24037                St. Mary's County, Maryland          111531
    ## 1213 0500000US24039                  Somerset County, Maryland           25737
    ## 1214 0500000US24041                    Talbot County, Maryland           37211
    ## 1215 0500000US24043                Washington County, Maryland          149811
    ## 1216 0500000US24045                  Wicomico County, Maryland          102172
    ## 1217 0500000US24047                 Worcester County, Maryland           51564
    ## 1218 0500000US24510                   Baltimore city, Maryland          614700
    ## 1219 0500000US25001           Barnstable County, Massachusetts          213690
    ## 1220 0500000US25003            Berkshire County, Massachusetts          127328
    ## 1221 0500000US25005              Bristol County, Massachusetts          558905
    ## 1222 0500000US25007                Dukes County, Massachusetts           17313
    ## 1223 0500000US25009                Essex County, Massachusetts          781024
    ## 1224 0500000US25011             Franklin County, Massachusetts           70935
    ## 1225 0500000US25013              Hampden County, Massachusetts          469116
    ## 1226 0500000US25015            Hampshire County, Massachusetts          161159
    ## 1227 0500000US25017            Middlesex County, Massachusetts         1595192
    ## 1228 0500000US25019            Nantucket County, Massachusetts           11101
    ## 1229 0500000US25021              Norfolk County, Massachusetts          698249
    ## 1230 0500000US25023             Plymouth County, Massachusetts          512135
    ## 1231 0500000US25025              Suffolk County, Massachusetts          791766
    ## 1232 0500000US25027            Worcester County, Massachusetts          822280
    ## 1233 0500000US26001                    Alcona County, Michigan           10364
    ## 1234 0500000US26003                     Alger County, Michigan            9194
    ## 1235 0500000US26005                   Allegan County, Michigan          115250
    ## 1236 0500000US26007                    Alpena County, Michigan           28612
    ## 1237 0500000US26009                    Antrim County, Michigan           23177
    ## 1238 0500000US26011                    Arenac County, Michigan           15165
    ## 1239 0500000US26013                    Baraga County, Michigan            8507
    ## 1240 0500000US26015                     Barry County, Michigan           60057
    ## 1241 0500000US26017                       Bay County, Michigan          104786
    ## 1242 0500000US26019                    Benzie County, Michigan           17552
    ## 1243 0500000US26021                   Berrien County, Michigan          154807
    ## 1244 0500000US26023                    Branch County, Michigan           43584
    ## 1245 0500000US26025                   Calhoun County, Michigan          134473
    ## 1246 0500000US26027                      Cass County, Michigan           51460
    ## 1247 0500000US26029                Charlevoix County, Michigan           26219
    ## 1248 0500000US26031                 Cheboygan County, Michigan           25458
    ## 1249 0500000US26033                  Chippewa County, Michigan           37834
    ## 1250 0500000US26035                     Clare County, Michigan           30616
    ## 1251 0500000US26037                   Clinton County, Michigan           77896
    ## 1252 0500000US26039                  Crawford County, Michigan           13836
    ## 1253 0500000US26041                     Delta County, Michigan           36190
    ## 1254 0500000US26043                 Dickinson County, Michigan           25570
    ## 1255 0500000US26045                     Eaton County, Michigan          109155
    ## 1256 0500000US26047                     Emmet County, Michigan           33039
    ## 1257 0500000US26049                   Genesee County, Michigan          409361
    ## 1258 0500000US26051                   Gladwin County, Michigan           25289
    ## 1259 0500000US26053                   Gogebic County, Michigan           15414
    ## 1260 0500000US26055            Grand Traverse County, Michigan           91746
    ## 1261 0500000US26057                   Gratiot County, Michigan           41067
    ## 1262 0500000US26059                 Hillsdale County, Michigan           45830
    ## 1263 0500000US26061                  Houghton County, Michigan           36360
    ## 1264 0500000US26063                     Huron County, Michigan           31543
    ## 1265 0500000US26065                    Ingham County, Michigan          289564
    ## 1266 0500000US26067                     Ionia County, Michigan           64176
    ## 1267 0500000US26069                     Iosco County, Michigan           25247
    ## 1268 0500000US26071                      Iron County, Michigan           11212
    ## 1269 0500000US26073                  Isabella County, Michigan           70775
    ## 1270 0500000US26075                   Jackson County, Michigan          158913
    ## 1271 0500000US26077                 Kalamazoo County, Michigan          261573
    ## 1272 0500000US26079                  Kalkaska County, Michigan           17463
    ## 1273 0500000US26081                      Kent County, Michigan          643140
    ## 1274 0500000US26083                  Keweenaw County, Michigan            2130
    ## 1275 0500000US26085                      Lake County, Michigan           11763
    ## 1276 0500000US26087                    Lapeer County, Michigan           88202
    ## 1277 0500000US26089                  Leelanau County, Michigan           21639
    ## 1278 0500000US26091                   Lenawee County, Michigan           98474
    ## 1279 0500000US26093                Livingston County, Michigan          188482
    ## 1280 0500000US26095                      Luce County, Michigan            6364
    ## 1281 0500000US26097                  Mackinac County, Michigan           10817
    ## 1282 0500000US26099                    Macomb County, Michigan          868704
    ## 1283 0500000US26101                  Manistee County, Michigan           24444
    ## 1284 0500000US26103                 Marquette County, Michigan           66939
    ## 1285 0500000US26105                     Mason County, Michigan           28884
    ## 1286 0500000US26107                   Mecosta County, Michigan           43264
    ## 1287 0500000US26109                 Menominee County, Michigan           23234
    ## 1288 0500000US26111                   Midland County, Michigan           83389
    ## 1289 0500000US26113                 Missaukee County, Michigan           15006
    ## 1290 0500000US26115                    Monroe County, Michigan          149699
    ## 1291 0500000US26117                  Montcalm County, Michigan           63209
    ## 1292 0500000US26119               Montmorency County, Michigan            9261
    ## 1293 0500000US26121                  Muskegon County, Michigan          173043
    ## 1294 0500000US26123                   Newaygo County, Michigan           48142
    ## 1295 0500000US26125                   Oakland County, Michigan         1250843
    ## 1296 0500000US26127                    Oceana County, Michigan           26417
    ## 1297 0500000US26129                    Ogemaw County, Michigan           20928
    ## 1298 0500000US26131                 Ontonagon County, Michigan            5968
    ## 1299 0500000US26133                   Osceola County, Michigan           23232
    ## 1300 0500000US26135                    Oscoda County, Michigan            8277
    ## 1301 0500000US26137                    Otsego County, Michigan           24397
    ## 1302 0500000US26139                    Ottawa County, Michigan          284034
    ## 1303 0500000US26141              Presque Isle County, Michigan           12797
    ## 1304 0500000US26143                 Roscommon County, Michigan           23877
    ## 1305 0500000US26145                   Saginaw County, Michigan          192778
    ## 1306 0500000US26147                 St. Clair County, Michigan          159566
    ## 1307 0500000US26149                St. Joseph County, Michigan           60897
    ## 1308 0500000US26151                   Sanilac County, Michigan           41376
    ## 1309 0500000US26153               Schoolcraft County, Michigan            8069
    ## 1310 0500000US26155                Shiawassee County, Michigan           68493
    ## 1311 0500000US26157                   Tuscola County, Michigan           53250
    ## 1312 0500000US26159                 Van Buren County, Michigan           75272
    ## 1313 0500000US26161                 Washtenaw County, Michigan          365961
    ## 1314 0500000US26163                     Wayne County, Michigan         1761382
    ## 1315 0500000US26165                   Wexford County, Michigan           33111
    ## 1316 0500000US27001                   Aitkin County, Minnesota           15834
    ## 1317 0500000US27003                    Anoka County, Minnesota          347431
    ## 1318 0500000US27005                   Becker County, Minnesota           33773
    ## 1319 0500000US27007                 Beltrami County, Minnesota           46117
    ## 1320 0500000US27009                   Benton County, Minnesota           39779
    ## 1321 0500000US27011                Big Stone County, Minnesota            5016
    ## 1322 0500000US27013               Blue Earth County, Minnesota           66322
    ## 1323 0500000US27015                    Brown County, Minnesota           25211
    ## 1324 0500000US27017                  Carlton County, Minnesota           35540
    ## 1325 0500000US27019                   Carver County, Minnesota          100416
    ## 1326 0500000US27021                     Cass County, Minnesota           29022
    ## 1327 0500000US27023                 Chippewa County, Minnesota           12010
    ## 1328 0500000US27025                  Chisago County, Minnesota           54727
    ## 1329 0500000US27027                     Clay County, Minnesota           62801
    ## 1330 0500000US27029               Clearwater County, Minnesota            8812
    ## 1331 0500000US27031                     Cook County, Minnesota            5311
    ## 1332 0500000US27033               Cottonwood County, Minnesota           11372
    ## 1333 0500000US27035                Crow Wing County, Minnesota           63855
    ## 1334 0500000US27037                   Dakota County, Minnesota          418201
    ## 1335 0500000US27039                    Dodge County, Minnesota           20582
    ## 1336 0500000US27041                  Douglas County, Minnesota           37203
    ## 1337 0500000US27043                Faribault County, Minnesota           13896
    ## 1338 0500000US27045                 Fillmore County, Minnesota           20888
    ## 1339 0500000US27047                 Freeborn County, Minnesota           30526
    ## 1340 0500000US27049                  Goodhue County, Minnesota           46217
    ## 1341 0500000US27051                    Grant County, Minnesota            5938
    ## 1342 0500000US27053                 Hennepin County, Minnesota         1235478
    ## 1343 0500000US27055                  Houston County, Minnesota           18663
    ## 1344 0500000US27057                  Hubbard County, Minnesota           20862
    ## 1345 0500000US27059                   Isanti County, Minnesota           38974
    ## 1346 0500000US27061                   Itasca County, Minnesota           45203
    ## 1347 0500000US27063                  Jackson County, Minnesota           10047
    ## 1348 0500000US27065                  Kanabec County, Minnesota           16004
    ## 1349 0500000US27067                Kandiyohi County, Minnesota           42658
    ## 1350 0500000US27069                  Kittson County, Minnesota            4337
    ## 1351 0500000US27071              Koochiching County, Minnesota           12644
    ## 1352 0500000US27073            Lac qui Parle County, Minnesota            6773
    ## 1353 0500000US27075                     Lake County, Minnesota           10569
    ## 1354 0500000US27077        Lake of the Woods County, Minnesota            3809
    ## 1355 0500000US27079                 Le Sueur County, Minnesota           27983
    ## 1356 0500000US27081                  Lincoln County, Minnesota            5707
    ## 1357 0500000US27083                     Lyon County, Minnesota           25839
    ## 1358 0500000US27085                   McLeod County, Minnesota           35825
    ## 1359 0500000US27087                 Mahnomen County, Minnesota            5506
    ## 1360 0500000US27089                 Marshall County, Minnesota            9392
    ## 1361 0500000US27091                   Martin County, Minnesota           19964
    ## 1362 0500000US27093                   Meeker County, Minnesota           23079
    ## 1363 0500000US27095               Mille Lacs County, Minnesota           25728
    ## 1364 0500000US27097                 Morrison County, Minnesota           32949
    ## 1365 0500000US27099                    Mower County, Minnesota           39602
    ## 1366 0500000US27101                   Murray County, Minnesota            8353
    ## 1367 0500000US27103                 Nicollet County, Minnesota           33783
    ## 1368 0500000US27105                   Nobles County, Minnesota           21839
    ## 1369 0500000US27107                   Norman County, Minnesota            6559
    ## 1370 0500000US27109                  Olmsted County, Minnesota          153065
    ## 1371 0500000US27111               Otter Tail County, Minnesota           57992
    ## 1372 0500000US27113               Pennington County, Minnesota           14184
    ## 1373 0500000US27115                     Pine County, Minnesota           29129
    ## 1374 0500000US27117                Pipestone County, Minnesota            9185
    ## 1375 0500000US27119                     Polk County, Minnesota           31591
    ## 1376 0500000US27121                     Pope County, Minnesota           10980
    ## 1377 0500000US27123                   Ramsey County, Minnesota          541493
    ## 1378 0500000US27125                 Red Lake County, Minnesota            4008
    ## 1379 0500000US27127                  Redwood County, Minnesota           15331
    ## 1380 0500000US27129                 Renville County, Minnesota           14721
    ## 1381 0500000US27131                     Rice County, Minnesota           65765
    ## 1382 0500000US27133                     Rock County, Minnesota            9413
    ## 1383 0500000US27135                   Roseau County, Minnesota           15462
    ## 1384 0500000US27137                St. Louis County, Minnesota          200080
    ## 1385 0500000US27139                    Scott County, Minnesota          143372
    ## 1386 0500000US27141                Sherburne County, Minnesota           93231
    ## 1387 0500000US27143                   Sibley County, Minnesota           14912
    ## 1388 0500000US27145                  Stearns County, Minnesota          156819
    ## 1389 0500000US27147                   Steele County, Minnesota           36676
    ## 1390 0500000US27149                  Stevens County, Minnesota            9784
    ## 1391 0500000US27151                    Swift County, Minnesota            9411
    ## 1392 0500000US27153                     Todd County, Minnesota           24440
    ## 1393 0500000US27155                 Traverse County, Minnesota            3337
    ## 1394 0500000US27157                  Wabasha County, Minnesota           21500
    ## 1395 0500000US27159                   Wadena County, Minnesota           13646
    ## 1396 0500000US27161                   Waseca County, Minnesota           18809
    ## 1397 0500000US27163               Washington County, Minnesota          253317
    ## 1398 0500000US27165                 Watonwan County, Minnesota           10973
    ## 1399 0500000US27167                   Wilkin County, Minnesota            6343
    ## 1400 0500000US27169                   Winona County, Minnesota           50847
    ## 1401 0500000US27171                   Wright County, Minnesota          132745
    ## 1402 0500000US27173          Yellow Medicine County, Minnesota            9868
    ## 1403 0500000US28001                  Adams County, Mississippi           31547
    ## 1404 0500000US28003                 Alcorn County, Mississippi           37180
    ## 1405 0500000US28005                  Amite County, Mississippi           12468
    ## 1406 0500000US28007                 Attala County, Mississippi           18581
    ## 1407 0500000US28009                 Benton County, Mississippi            8253
    ## 1408 0500000US28011                Bolivar County, Mississippi           32592
    ## 1409 0500000US28013                Calhoun County, Mississippi           14571
    ## 1410 0500000US28015                Carroll County, Mississippi           10129
    ## 1411 0500000US28017              Chickasaw County, Mississippi           17279
    ## 1412 0500000US28019                Choctaw County, Mississippi            8321
    ## 1413 0500000US28021              Claiborne County, Mississippi            9120
    ## 1414 0500000US28023                 Clarke County, Mississippi           15928
    ## 1415 0500000US28025                   Clay County, Mississippi           19808
    ## 1416 0500000US28027                Coahoma County, Mississippi           23802
    ## 1417 0500000US28029                 Copiah County, Mississippi           28721
    ## 1418 0500000US28031              Covington County, Mississippi           19091
    ## 1419 0500000US28033                 DeSoto County, Mississippi          176132
    ## 1420 0500000US28035                Forrest County, Mississippi           75517
    ## 1421 0500000US28037               Franklin County, Mississippi            7757
    ## 1422 0500000US28039                 George County, Mississippi           23710
    ## 1423 0500000US28041                 Greene County, Mississippi           13714
    ## 1424 0500000US28043                Grenada County, Mississippi           21278
    ## 1425 0500000US28045                Hancock County, Mississippi           46653
    ## 1426 0500000US28047               Harrison County, Mississippi          202626
    ## 1427 0500000US28049                  Hinds County, Mississippi          241774
    ## 1428 0500000US28051                 Holmes County, Mississippi           18075
    ## 1429 0500000US28053              Humphreys County, Mississippi            8539
    ## 1430 0500000US28055              Issaquena County, Mississippi            1328
    ## 1431 0500000US28057               Itawamba County, Mississippi           23480
    ## 1432 0500000US28059                Jackson County, Mississippi          142014
    ## 1433 0500000US28061                 Jasper County, Mississippi           16529
    ## 1434 0500000US28063              Jefferson County, Mississippi            7346
    ## 1435 0500000US28065        Jefferson Davis County, Mississippi           11495
    ## 1436 0500000US28067                  Jones County, Mississippi           68454
    ## 1437 0500000US28069                 Kemper County, Mississippi           10107
    ## 1438 0500000US28071              Lafayette County, Mississippi           53459
    ## 1439 0500000US28073                  Lamar County, Mississippi           61223
    ## 1440 0500000US28075             Lauderdale County, Mississippi           77323
    ## 1441 0500000US28077               Lawrence County, Mississippi           12630
    ## 1442 0500000US28079                  Leake County, Mississippi           22870
    ## 1443 0500000US28081                    Lee County, Mississippi           84915
    ## 1444 0500000US28083                Leflore County, Mississippi           29804
    ## 1445 0500000US28085                Lincoln County, Mississippi           34432
    ## 1446 0500000US28087                Lowndes County, Mississippi           59437
    ## 1447 0500000US28089                Madison County, Mississippi          103498
    ## 1448 0500000US28091                 Marion County, Mississippi           25202
    ## 1449 0500000US28093               Marshall County, Mississippi           35787
    ## 1450 0500000US28095                 Monroe County, Mississippi           35840
    ## 1451 0500000US28097             Montgomery County, Mississippi           10198
    ## 1452 0500000US28099                Neshoba County, Mississippi           29376
    ## 1453 0500000US28101                 Newton County, Mississippi           21524
    ## 1454 0500000US28103                Noxubee County, Mississippi           10828
    ## 1455 0500000US28105              Oktibbeha County, Mississippi           49481
    ## 1456 0500000US28107                 Panola County, Mississippi           34243
    ## 1457 0500000US28109            Pearl River County, Mississippi           55149
    ## 1458 0500000US28111                  Perry County, Mississippi           12028
    ## 1459 0500000US28113                   Pike County, Mississippi           39737
    ## 1460 0500000US28115               Pontotoc County, Mississippi           31315
    ## 1461 0500000US28117               Prentiss County, Mississippi           25360
    ## 1462 0500000US28119                Quitman County, Mississippi            7372
    ## 1463 0500000US28121                 Rankin County, Mississippi          151240
    ## 1464 0500000US28123                  Scott County, Mississippi           28415
    ## 1465 0500000US28125                Sharkey County, Mississippi            4511
    ## 1466 0500000US28127                Simpson County, Mississippi           27073
    ## 1467 0500000US28129                  Smith County, Mississippi           16063
    ## 1468 0500000US28131                  Stone County, Mississippi           18375
    ## 1469 0500000US28133              Sunflower County, Mississippi           26532
    ## 1470 0500000US28135           Tallahatchie County, Mississippi           14361
    ## 1471 0500000US28137                   Tate County, Mississippi           28493
    ## 1472 0500000US28139                 Tippah County, Mississippi           21990
    ## 1473 0500000US28141             Tishomingo County, Mississippi           19478
    ## 1474 0500000US28143                 Tunica County, Mississippi           10170
    ## 1475 0500000US28145                  Union County, Mississippi           28356
    ## 1476 0500000US28147               Walthall County, Mississippi           14601
    ## 1477 0500000US28149                 Warren County, Mississippi           47075
    ## 1478 0500000US28151             Washington County, Mississippi           47086
    ## 1479 0500000US28153                  Wayne County, Mississippi           20422
    ## 1480 0500000US28155                Webster County, Mississippi            9828
    ## 1481 0500000US28157              Wilkinson County, Mississippi            8990
    ## 1482 0500000US28159                Winston County, Mississippi           18358
    ## 1483 0500000US28161              Yalobusha County, Mississippi           12421
    ## 1484 0500000US28163                  Yazoo County, Mississippi           27974
    ## 1485 0500000US29001                     Adair County, Missouri           25325
    ## 1486 0500000US29003                    Andrew County, Missouri           17403
    ## 1487 0500000US29005                  Atchison County, Missouri            5270
    ## 1488 0500000US29007                   Audrain County, Missouri           25735
    ## 1489 0500000US29009                     Barry County, Missouri           35493
    ## 1490 0500000US29011                    Barton County, Missouri           11850
    ## 1491 0500000US29013                     Bates County, Missouri           16374
    ## 1492 0500000US29015                    Benton County, Missouri           18989
    ## 1493 0500000US29017                 Bollinger County, Missouri           12281
    ## 1494 0500000US29019                     Boone County, Missouri          176515
    ## 1495 0500000US29021                  Buchanan County, Missouri           89076
    ## 1496 0500000US29023                    Butler County, Missouri           42733
    ## 1497 0500000US29025                  Caldwell County, Missouri            9049
    ## 1498 0500000US29027                  Callaway County, Missouri           44840
    ## 1499 0500000US29029                    Camden County, Missouri           45096
    ## 1500 0500000US29031            Cape Girardeau County, Missouri           78324
    ## 1501 0500000US29033                   Carroll County, Missouri            8843
    ## 1502 0500000US29035                    Carter County, Missouri            6197
    ## 1503 0500000US29037                      Cass County, Missouri          102678
    ## 1504 0500000US29039                     Cedar County, Missouri           13938
    ## 1505 0500000US29041                  Chariton County, Missouri            7546
    ## 1506 0500000US29043                 Christian County, Missouri           84275
    ## 1507 0500000US29045                     Clark County, Missouri            6800
    ## 1508 0500000US29047                      Clay County, Missouri          239164
    ## 1509 0500000US29049                   Clinton County, Missouri           20475
    ## 1510 0500000US29051                      Cole County, Missouri           76740
    ## 1511 0500000US29053                    Cooper County, Missouri           17622
    ## 1512 0500000US29055                  Crawford County, Missouri           24280
    ## 1513 0500000US29057                      Dade County, Missouri            7590
    ## 1514 0500000US29059                    Dallas County, Missouri           16499
    ## 1515 0500000US29061                   Daviess County, Missouri            8302
    ## 1516 0500000US29063                    DeKalb County, Missouri           12564
    ## 1517 0500000US29065                      Dent County, Missouri           15504
    ## 1518 0500000US29067                   Douglas County, Missouri           13374
    ## 1519 0500000US29069                   Dunklin County, Missouri           30428
    ## 1520 0500000US29071                  Franklin County, Missouri          102781
    ## 1521 0500000US29073                 Gasconade County, Missouri           14746
    ## 1522 0500000US29075                    Gentry County, Missouri            6665
    ## 1523 0500000US29077                    Greene County, Missouri          288429
    ## 1524 0500000US29079                    Grundy County, Missouri           10039
    ## 1525 0500000US29081                  Harrison County, Missouri            8554
    ## 1526 0500000US29083                     Henry County, Missouri           21765
    ## 1527 0500000US29085                   Hickory County, Missouri            9368
    ## 1528 0500000US29087                      Holt County, Missouri            4456
    ## 1529 0500000US29089                    Howard County, Missouri           10113
    ## 1530 0500000US29091                    Howell County, Missouri           40102
    ## 1531 0500000US29093                      Iron County, Missouri           10221
    ## 1532 0500000US29095                   Jackson County, Missouri          692003
    ## 1533 0500000US29097                    Jasper County, Missouri          119238
    ## 1534 0500000US29099                 Jefferson County, Missouri          223302
    ## 1535 0500000US29101                   Johnson County, Missouri           53689
    ## 1536 0500000US29103                      Knox County, Missouri            3951
    ## 1537 0500000US29105                   Laclede County, Missouri           35507
    ## 1538 0500000US29107                 Lafayette County, Missouri           32589
    ## 1539 0500000US29109                  Lawrence County, Missouri           38133
    ## 1540 0500000US29111                     Lewis County, Missouri           10027
    ## 1541 0500000US29113                   Lincoln County, Missouri           55563
    ## 1542 0500000US29115                      Linn County, Missouri           12186
    ## 1543 0500000US29117                Livingston County, Missouri           15076
    ## 1544 0500000US29119                  McDonald County, Missouri           22827
    ## 1545 0500000US29121                     Macon County, Missouri           15254
    ## 1546 0500000US29123                   Madison County, Missouri           12205
    ## 1547 0500000US29125                    Maries County, Missouri            8884
    ## 1548 0500000US29127                    Marion County, Missouri           28672
    ## 1549 0500000US29129                    Mercer County, Missouri            3664
    ## 1550 0500000US29131                    Miller County, Missouri           25049
    ## 1551 0500000US29133               Mississippi County, Missouri           13748
    ## 1552 0500000US29135                  Moniteau County, Missouri           15958
    ## 1553 0500000US29137                    Monroe County, Missouri            8654
    ## 1554 0500000US29139                Montgomery County, Missouri           11545
    ## 1555 0500000US29141                    Morgan County, Missouri           20137
    ## 1556 0500000US29143                New Madrid County, Missouri           17811
    ## 1557 0500000US29145                    Newton County, Missouri           58202
    ## 1558 0500000US29147                   Nodaway County, Missouri           22547
    ## 1559 0500000US29149                    Oregon County, Missouri           10699
    ## 1560 0500000US29151                     Osage County, Missouri           13619
    ## 1561 0500000US29153                     Ozark County, Missouri            9236
    ## 1562 0500000US29155                  Pemiscot County, Missouri           17031
    ## 1563 0500000US29157                     Perry County, Missouri           19146
    ## 1564 0500000US29159                    Pettis County, Missouri           42371
    ## 1565 0500000US29161                    Phelps County, Missouri           44789
    ## 1566 0500000US29163                      Pike County, Missouri           18489
    ## 1567 0500000US29165                    Platte County, Missouri           98824
    ## 1568 0500000US29167                      Polk County, Missouri           31549
    ## 1569 0500000US29169                   Pulaski County, Missouri           52591
    ## 1570 0500000US29171                    Putnam County, Missouri            4815
    ## 1571 0500000US29173                     Ralls County, Missouri           10217
    ## 1572 0500000US29175                  Randolph County, Missouri           24945
    ## 1573 0500000US29177                       Ray County, Missouri           22825
    ## 1574 0500000US29179                  Reynolds County, Missouri            6315
    ## 1575 0500000US29181                    Ripley County, Missouri           13693
    ## 1576 0500000US29183               St. Charles County, Missouri          389985
    ## 1577 0500000US29185                 St. Clair County, Missouri            9383
    ## 1578 0500000US29186            Ste. Genevieve County, Missouri           17871
    ## 1579 0500000US29187              St. Francois County, Missouri           66342
    ## 1580 0500000US29189                 St. Louis County, Missouri          998684
    ## 1581 0500000US29195                    Saline County, Missouri           23102
    ## 1582 0500000US29197                  Schuyler County, Missouri            4502
    ## 1583 0500000US29199                  Scotland County, Missouri            4898
    ## 1584 0500000US29201                     Scott County, Missouri           38729
    ## 1585 0500000US29203                   Shannon County, Missouri            8246
    ## 1586 0500000US29205                    Shelby County, Missouri            6061
    ## 1587 0500000US29207                  Stoddard County, Missouri           29512
    ## 1588 0500000US29209                     Stone County, Missouri           31527
    ## 1589 0500000US29211                  Sullivan County, Missouri            6317
    ## 1590 0500000US29213                     Taney County, Missouri           54720
    ## 1591 0500000US29215                     Texas County, Missouri           25671
    ## 1592 0500000US29217                    Vernon County, Missouri           20691
    ## 1593 0500000US29219                    Warren County, Missouri           33908
    ## 1594 0500000US29221                Washington County, Missouri           24931
    ## 1595 0500000US29223                     Wayne County, Missouri           13308
    ## 1596 0500000US29225                   Webster County, Missouri           38082
    ## 1597 0500000US29227                     Worth County, Missouri            2040
    ## 1598 0500000US29229                    Wright County, Missouri           18293
    ## 1599 0500000US29510                   St. Louis city, Missouri          311273
    ## 1600 0500000US30001                 Beaverhead County, Montana            9393
    ## 1601 0500000US30003                   Big Horn County, Montana           13376
    ## 1602 0500000US30005                     Blaine County, Montana            6727
    ## 1603 0500000US30007                 Broadwater County, Montana            5834
    ## 1604 0500000US30009                     Carbon County, Montana           10546
    ## 1605 0500000US30011                     Carter County, Montana            1318
    ## 1606 0500000US30013                    Cascade County, Montana           81746
    ## 1607 0500000US30015                   Chouteau County, Montana            5789
    ## 1608 0500000US30017                     Custer County, Montana           11845
    ## 1609 0500000US30019                    Daniels County, Montana            1753
    ## 1610 0500000US30021                     Dawson County, Montana            9191
    ## 1611 0500000US30023                 Deer Lodge County, Montana            9100
    ## 1612 0500000US30025                     Fallon County, Montana            2838
    ## 1613 0500000US30027                     Fergus County, Montana           11273
    ## 1614 0500000US30029                   Flathead County, Montana           98082
    ## 1615 0500000US30031                   Gallatin County, Montana          104729
    ## 1616 0500000US30033                   Garfield County, Montana            1141
    ## 1617 0500000US30035                    Glacier County, Montana           13699
    ## 1618 0500000US30037              Golden Valley County, Montana             724
    ## 1619 0500000US30039                    Granite County, Montana            3269
    ## 1620 0500000US30041                       Hill County, Montana           16439
    ## 1621 0500000US30043                  Jefferson County, Montana           11778
    ## 1622 0500000US30045               Judith Basin County, Montana            1951
    ## 1623 0500000US30047                       Lake County, Montana           29774
    ## 1624 0500000US30049            Lewis and Clark County, Montana           67077
    ## 1625 0500000US30051                    Liberty County, Montana            2280
    ## 1626 0500000US30053                    Lincoln County, Montana           19358
    ## 1627 0500000US30055                     McCone County, Montana            1630
    ## 1628 0500000US30057                    Madison County, Montana            8218
    ## 1629 0500000US30059                    Meagher County, Montana            1968
    ## 1630 0500000US30061                    Mineral County, Montana            4211
    ## 1631 0500000US30063                   Missoula County, Montana          115983
    ## 1632 0500000US30065                Musselshell County, Montana            4807
    ## 1633 0500000US30067                       Park County, Montana           16246
    ## 1634 0500000US30069                  Petroleum County, Montana             432
    ## 1635 0500000US30071                   Phillips County, Montana            4124
    ## 1636 0500000US30073                    Pondera County, Montana            6044
    ## 1637 0500000US30075               Powder River County, Montana            1619
    ## 1638 0500000US30077                     Powell County, Montana            6861
    ## 1639 0500000US30079                    Prairie County, Montana            1342
    ## 1640 0500000US30081                    Ravalli County, Montana           41902
    ## 1641 0500000US30083                   Richland County, Montana           11360
    ## 1642 0500000US30085                  Roosevelt County, Montana           11228
    ## 1643 0500000US30087                    Rosebud County, Montana            9250
    ## 1644 0500000US30089                    Sanders County, Montana           11521
    ## 1645 0500000US30091                   Sheridan County, Montana            3574
    ## 1646 0500000US30093                 Silver Bow County, Montana           34814
    ## 1647 0500000US30095                 Stillwater County, Montana            9410
    ## 1648 0500000US30097                Sweet Grass County, Montana            3653
    ## 1649 0500000US30099                      Teton County, Montana            6080
    ## 1650 0500000US30101                      Toole County, Montana            4976
    ## 1651 0500000US30103                   Treasure County, Montana             777
    ## 1652 0500000US30105                     Valley County, Montana            7532
    ## 1653 0500000US30107                  Wheatland County, Montana            2149
    ## 1654 0500000US30109                     Wibaux County, Montana            1175
    ## 1655 0500000US30111                Yellowstone County, Montana          157816
    ## 1656 0500000US31001                     Adams County, Nebraska           31583
    ## 1657 0500000US31003                  Antelope County, Nebraska            6372
    ## 1658 0500000US31005                    Arthur County, Nebraska             418
    ## 1659 0500000US31007                    Banner County, Nebraska             696
    ## 1660 0500000US31009                    Blaine County, Nebraska             480
    ## 1661 0500000US31011                     Boone County, Nebraska            5313
    ## 1662 0500000US31013                 Box Butte County, Nebraska           11089
    ## 1663 0500000US31015                      Boyd County, Nebraska            2042
    ## 1664 0500000US31017                     Brown County, Nebraska            2988
    ## 1665 0500000US31019                   Buffalo County, Nebraska           49030
    ## 1666 0500000US31021                      Burt County, Nebraska            6528
    ## 1667 0500000US31023                    Butler County, Nebraska            8067
    ## 1668 0500000US31025                      Cass County, Nebraska           25702
    ## 1669 0500000US31027                     Cedar County, Nebraska            8523
    ## 1670 0500000US31029                     Chase County, Nebraska            3734
    ## 1671 0500000US31031                    Cherry County, Nebraska            5790
    ## 1672 0500000US31033                  Cheyenne County, Nebraska            9852
    ## 1673 0500000US31035                      Clay County, Nebraska            6232
    ## 1674 0500000US31037                    Colfax County, Nebraska           10760
    ## 1675 0500000US31039                    Cuming County, Nebraska            8991
    ## 1676 0500000US31041                    Custer County, Nebraska           10830
    ## 1677 0500000US31043                    Dakota County, Nebraska           20317
    ## 1678 0500000US31045                     Dawes County, Nebraska            8896
    ## 1679 0500000US31047                    Dawson County, Nebraska           23804
    ## 1680 0500000US31049                     Deuel County, Nebraska            1894
    ## 1681 0500000US31051                     Dixon County, Nebraska            5746
    ## 1682 0500000US31053                     Dodge County, Nebraska           36683
    ## 1683 0500000US31055                   Douglas County, Nebraska          554992
    ## 1684 0500000US31057                     Dundy County, Nebraska            2023
    ## 1685 0500000US31059                  Fillmore County, Nebraska            5574
    ## 1686 0500000US31061                  Franklin County, Nebraska            3006
    ## 1687 0500000US31063                  Frontier County, Nebraska            2609
    ## 1688 0500000US31065                    Furnas County, Nebraska            4786
    ## 1689 0500000US31067                      Gage County, Nebraska           21595
    ## 1690 0500000US31069                    Garden County, Nebraska            1860
    ## 1691 0500000US31071                  Garfield County, Nebraska            1975
    ## 1692 0500000US31073                    Gosper County, Nebraska            2015
    ## 1693 0500000US31075                     Grant County, Nebraska             718
    ## 1694 0500000US31077                   Greeley County, Nebraska            2410
    ## 1695 0500000US31079                      Hall County, Nebraska           61343
    ## 1696 0500000US31081                  Hamilton County, Nebraska            9178
    ## 1697 0500000US31083                    Harlan County, Nebraska            3438
    ## 1698 0500000US31085                     Hayes County, Nebraska             943
    ## 1699 0500000US31087                 Hitchcock County, Nebraska            2843
    ## 1700 0500000US31089                      Holt County, Nebraska           10245
    ## 1701 0500000US31091                    Hooker County, Nebraska             691
    ## 1702 0500000US31093                    Howard County, Nebraska            6405
    ## 1703 0500000US31095                 Jefferson County, Nebraska            7188
    ## 1704 0500000US31097                   Johnson County, Nebraska            5197
    ## 1705 0500000US31099                   Kearney County, Nebraska            6552
    ## 1706 0500000US31101                     Keith County, Nebraska            8099
    ## 1707 0500000US31103                 Keya Paha County, Nebraska             792
    ## 1708 0500000US31105                   Kimball County, Nebraska            3667
    ## 1709 0500000US31107                      Knox County, Nebraska            8460
    ## 1710 0500000US31109                 Lancaster County, Nebraska          310094
    ## 1711 0500000US31111                   Lincoln County, Nebraska           35433
    ## 1712 0500000US31113                     Logan County, Nebraska             886
    ## 1713 0500000US31115                      Loup County, Nebraska             585
    ## 1714 0500000US31117                 McPherson County, Nebraska             454
    ## 1715 0500000US31119                   Madison County, Nebraska           35164
    ## 1716 0500000US31121                   Merrick County, Nebraska            7803
    ## 1717 0500000US31123                   Morrill County, Nebraska            4841
    ## 1718 0500000US31125                     Nance County, Nebraska            3554
    ## 1719 0500000US31127                    Nemaha County, Nebraska            7004
    ## 1720 0500000US31129                  Nuckolls County, Nebraska            4275
    ## 1721 0500000US31131                      Otoe County, Nebraska           15896
    ## 1722 0500000US31133                    Pawnee County, Nebraska            2676
    ## 1723 0500000US31135                   Perkins County, Nebraska            2907
    ## 1724 0500000US31137                    Phelps County, Nebraska            9120
    ## 1725 0500000US31139                    Pierce County, Nebraska            7157
    ## 1726 0500000US31141                    Platte County, Nebraska           33063
    ## 1727 0500000US31143                      Polk County, Nebraska            5255
    ## 1728 0500000US31145                Red Willow County, Nebraska           10806
    ## 1729 0500000US31147                Richardson County, Nebraska            8009
    ## 1730 0500000US31149                      Rock County, Nebraska            1350
    ## 1731 0500000US31151                    Saline County, Nebraska           14288
    ## 1732 0500000US31153                     Sarpy County, Nebraska          178351
    ## 1733 0500000US31155                  Saunders County, Nebraska           21024
    ## 1734 0500000US31157              Scotts Bluff County, Nebraska           36255
    ## 1735 0500000US31159                    Seward County, Nebraska           17127
    ## 1736 0500000US31161                  Sheridan County, Nebraska            5234
    ## 1737 0500000US31163                   Sherman County, Nebraska            3042
    ## 1738 0500000US31165                     Sioux County, Nebraska            1266
    ## 1739 0500000US31167                   Stanton County, Nebraska            5992
    ## 1740 0500000US31169                    Thayer County, Nebraska            5098
    ## 1741 0500000US31171                    Thomas County, Nebraska             645
    ## 1742 0500000US31173                  Thurston County, Nebraska            7140
    ## 1743 0500000US31175                    Valley County, Nebraska            4224
    ## 1744 0500000US31177                Washington County, Nebraska           20219
    ## 1745 0500000US31179                     Wayne County, Nebraska            9367
    ## 1746 0500000US31181                   Webster County, Nebraska            3571
    ## 1747 0500000US31183                   Wheeler County, Nebraska             822
    ## 1748 0500000US31185                      York County, Nebraska           13799
    ## 1749 0500000US32001                   Churchill County, Nevada           24010
    ## 1750 0500000US32003                       Clark County, Nevada         2141574
    ## 1751 0500000US32005                     Douglas County, Nevada           47828
    ## 1752 0500000US32007                        Elko County, Nevada           52252
    ## 1753 0500000US32009                   Esmeralda County, Nevada             981
    ## 1754 0500000US32011                      Eureka County, Nevada            1830
    ## 1755 0500000US32013                    Humboldt County, Nevada           16904
    ## 1756 0500000US32015                      Lander County, Nevada            5746
    ## 1757 0500000US32017                     Lincoln County, Nevada            5174
    ## 1758 0500000US32019                        Lyon County, Nevada           53155
    ## 1759 0500000US32021                     Mineral County, Nevada            4448
    ## 1760 0500000US32023                         Nye County, Nevada           43705
    ## 1761 0500000US32027                    Pershing County, Nevada            6611
    ## 1762 0500000US32029                      Storey County, Nevada            3941
    ## 1763 0500000US32031                      Washoe County, Nevada          450486
    ## 1764 0500000US32033                  White Pine County, Nevada            9737
    ## 1765 0500000US32510                        Carson City, Nevada           54467
    ## 1766 0500000US33001              Belknap County, New Hampshire           60640
    ## 1767 0500000US33003              Carroll County, New Hampshire           47840
    ## 1768 0500000US33005             Cheshire County, New Hampshire           76263
    ## 1769 0500000US33007                 Coos County, New Hampshire           32038
    ## 1770 0500000US33009              Grafton County, New Hampshire           89811
    ## 1771 0500000US33011         Hillsborough County, New Hampshire          411087
    ## 1772 0500000US33013            Merrimack County, New Hampshire          149452
    ## 1773 0500000US33015           Rockingham County, New Hampshire          305129
    ## 1774 0500000US33017            Strafford County, New Hampshire          128237
    ## 1775 0500000US33019             Sullivan County, New Hampshire           43125
    ## 1776 0500000US34001                Atlantic County, New Jersey          268539
    ## 1777 0500000US34003                  Bergen County, New Jersey          929999
    ## 1778 0500000US34005              Burlington County, New Jersey          446367
    ## 1779 0500000US34007                  Camden County, New Jersey          507367
    ## 1780 0500000US34009                Cape May County, New Jersey           93705
    ## 1781 0500000US34011              Cumberland County, New Jersey          153400
    ## 1782 0500000US34013                   Essex County, New Jersey          793555
    ## 1783 0500000US34015              Gloucester County, New Jersey          290852
    ## 1784 0500000US34017                  Hudson County, New Jersey          668631
    ## 1785 0500000US34019               Hunterdon County, New Jersey          125051
    ## 1786 0500000US34021                  Mercer County, New Jersey          368762
    ## 1787 0500000US34023               Middlesex County, New Jersey          826698
    ## 1788 0500000US34025                Monmouth County, New Jersey          623387
    ## 1789 0500000US34027                  Morris County, New Jersey          494383
    ## 1790 0500000US34029                   Ocean County, New Jersey          591939
    ## 1791 0500000US34031                 Passaic County, New Jersey          504041
    ## 1792 0500000US34033                   Salem County, New Jersey           63336
    ## 1793 0500000US34035                Somerset County, New Jersey          330176
    ## 1794 0500000US34037                  Sussex County, New Jersey          142298
    ## 1795 0500000US34039                   Union County, New Jersey          553066
    ## 1796 0500000US34041                  Warren County, New Jersey          106293
    ## 1797 0500000US35001              Bernalillo County, New Mexico          677692
    ## 1798 0500000US35003                  Catron County, New Mexico            3539
    ## 1799 0500000US35005                  Chaves County, New Mexico           65459
    ## 1800 0500000US35006                  Cibola County, New Mexico           26978
    ## 1801 0500000US35007                  Colfax County, New Mexico           12353
    ## 1802 0500000US35009                   Curry County, New Mexico           50199
    ## 1803 0500000US35011                 De Baca County, New Mexico            2060
    ## 1804 0500000US35013                Doña Ana County, New Mexico          215338
    ## 1805 0500000US35015                    Eddy County, New Mexico           57437
    ## 1806 0500000US35017                   Grant County, New Mexico           28061
    ## 1807 0500000US35019               Guadalupe County, New Mexico            4382
    ## 1808 0500000US35021                 Harding County, New Mexico             459
    ## 1809 0500000US35023                 Hidalgo County, New Mexico            4371
    ## 1810 0500000US35025                     Lea County, New Mexico           70126
    ## 1811 0500000US35027                 Lincoln County, New Mexico           19482
    ## 1812 0500000US35028              Los Alamos County, New Mexico           18356
    ## 1813 0500000US35029                    Luna County, New Mexico           24264
    ## 1814 0500000US35031                McKinley County, New Mexico           72849
    ## 1815 0500000US35033                    Mora County, New Mexico            4563
    ## 1816 0500000US35035                   Otero County, New Mexico           65745
    ## 1817 0500000US35037                    Quay County, New Mexico            8373
    ## 1818 0500000US35039              Rio Arriba County, New Mexico           39307
    ## 1819 0500000US35041               Roosevelt County, New Mexico           19117
    ## 1820 0500000US35043                Sandoval County, New Mexico          140769
    ## 1821 0500000US35045                San Juan County, New Mexico          127455
    ## 1822 0500000US35047              San Miguel County, New Mexico           28034
    ## 1823 0500000US35049                Santa Fe County, New Mexico          148917
    ## 1824 0500000US35051                  Sierra County, New Mexico           11135
    ## 1825 0500000US35053                 Socorro County, New Mexico           17000
    ## 1826 0500000US35055                    Taos County, New Mexico           32888
    ## 1827 0500000US35057                Torrance County, New Mexico           15595
    ## 1828 0500000US35059                   Union County, New Mexico            4175
    ## 1829 0500000US35061                Valencia County, New Mexico           75956
    ## 1830 0500000US36001                    Albany County, New York          307426
    ## 1831 0500000US36003                  Allegany County, New York           47025
    ## 1832 0500000US36005                     Bronx County, New York         1437872
    ## 1833 0500000US36007                    Broome County, New York          194402
    ## 1834 0500000US36009               Cattaraugus County, New York           77686
    ## 1835 0500000US36011                    Cayuga County, New York           77868
    ## 1836 0500000US36013                Chautauqua County, New York          129656
    ## 1837 0500000US36015                   Chemung County, New York           85740
    ## 1838 0500000US36017                  Chenango County, New York           48348
    ## 1839 0500000US36019                   Clinton County, New York           80794
    ## 1840 0500000US36021                  Columbia County, New York           60919
    ## 1841 0500000US36023                  Cortland County, New York           48123
    ## 1842 0500000US36025                  Delaware County, New York           45502
    ## 1843 0500000US36027                  Dutchess County, New York          293894
    ## 1844 0500000US36029                      Erie County, New York          919866
    ## 1845 0500000US36031                     Essex County, New York           37751
    ## 1846 0500000US36033                  Franklin County, New York           50692
    ## 1847 0500000US36035                    Fulton County, New York           53743
    ## 1848 0500000US36037                   Genesee County, New York           58112
    ## 1849 0500000US36039                    Greene County, New York           47617
    ## 1850 0500000US36041                  Hamilton County, New York            4575
    ## 1851 0500000US36043                  Herkimer County, New York           62505
    ## 1852 0500000US36045                 Jefferson County, New York          114448
    ## 1853 0500000US36047                     Kings County, New York         2600747
    ## 1854 0500000US36049                     Lewis County, New York           26719
    ## 1855 0500000US36051                Livingston County, New York           63907
    ## 1856 0500000US36053                   Madison County, New York           71359
    ## 1857 0500000US36055                    Monroe County, New York          744248
    ## 1858 0500000US36057                Montgomery County, New York           49426
    ## 1859 0500000US36059                    Nassau County, New York         1356564
    ## 1860 0500000US36061                  New York County, New York         1632480
    ## 1861 0500000US36063                   Niagara County, New York          211704
    ## 1862 0500000US36065                    Oneida County, New York          230782
    ## 1863 0500000US36067                  Onondaga County, New York          464242
    ## 1864 0500000US36069                   Ontario County, New York          109472
    ## 1865 0500000US36071                    Orange County, New York          378227
    ## 1866 0500000US36073                   Orleans County, New York           41175
    ## 1867 0500000US36075                    Oswego County, New York          119104
    ## 1868 0500000US36077                    Otsego County, New York           60244
    ## 1869 0500000US36079                    Putnam County, New York           99070
    ## 1870 0500000US36081                    Queens County, New York         2298513
    ## 1871 0500000US36083                Rensselaer County, New York          159431
    ## 1872 0500000US36085                  Richmond County, New York          474101
    ## 1873 0500000US36087                  Rockland County, New York          323686
    ## 1874 0500000US36089              St. Lawrence County, New York          109558
    ## 1875 0500000US36091                  Saratoga County, New York          227377
    ## 1876 0500000US36093               Schenectady County, New York          154883
    ## 1877 0500000US36095                 Schoharie County, New York           31364
    ## 1878 0500000US36097                  Schuyler County, New York           17992
    ## 1879 0500000US36099                    Seneca County, New York           34612
    ## 1880 0500000US36101                   Steuben County, New York           96927
    ## 1881 0500000US36103                   Suffolk County, New York         1487901
    ## 1882 0500000US36105                  Sullivan County, New York           75211
    ## 1883 0500000US36107                     Tioga County, New York           49045
    ## 1884 0500000US36109                  Tompkins County, New York          102962
    ## 1885 0500000US36111                    Ulster County, New York          179303
    ## 1886 0500000US36113                    Warren County, New York           64480
    ## 1887 0500000US36115                Washington County, New York           61828
    ## 1888 0500000US36117                     Wayne County, New York           90856
    ## 1889 0500000US36119               Westchester County, New York          968815
    ## 1890 0500000US36121                   Wyoming County, New York           40565
    ## 1891 0500000US36123                     Yates County, New York           25009
    ## 1892 0500000US37001            Alamance County, North Carolina          160576
    ## 1893 0500000US37003           Alexander County, North Carolina           37119
    ## 1894 0500000US37005           Alleghany County, North Carolina           10973
    ## 1895 0500000US37007               Anson County, North Carolina           25306
    ## 1896 0500000US37009                Ashe County, North Carolina           26786
    ## 1897 0500000US37011               Avery County, North Carolina           17501
    ## 1898 0500000US37013            Beaufort County, North Carolina           47243
    ## 1899 0500000US37015              Bertie County, North Carolina           19644
    ## 1900 0500000US37017              Bladen County, North Carolina           33778
    ## 1901 0500000US37019           Brunswick County, North Carolina          126860
    ## 1902 0500000US37021            Buncombe County, North Carolina          254474
    ## 1903 0500000US37023               Burke County, North Carolina           89712
    ## 1904 0500000US37025            Cabarrus County, North Carolina          201448
    ## 1905 0500000US37027            Caldwell County, North Carolina           81779
    ## 1906 0500000US37029              Camden County, North Carolina           10447
    ## 1907 0500000US37031            Carteret County, North Carolina           68920
    ## 1908 0500000US37033             Caswell County, North Carolina           22746
    ## 1909 0500000US37035             Catawba County, North Carolina          156729
    ## 1910 0500000US37037             Chatham County, North Carolina           69791
    ## 1911 0500000US37039            Cherokee County, North Carolina           27668
    ## 1912 0500000US37041              Chowan County, North Carolina           14205
    ## 1913 0500000US37043                Clay County, North Carolina           10813
    ## 1914 0500000US37045           Cleveland County, North Carolina           97159
    ## 1915 0500000US37047            Columbus County, North Carolina           56293
    ## 1916 0500000US37049              Craven County, North Carolina          103082
    ## 1917 0500000US37051          Cumberland County, North Carolina          332106
    ## 1918 0500000US37053           Currituck County, North Carolina           25796
    ## 1919 0500000US37055                Dare County, North Carolina           35741
    ## 1920 0500000US37057            Davidson County, North Carolina          164664
    ## 1921 0500000US37059               Davie County, North Carolina           41991
    ## 1922 0500000US37061              Duplin County, North Carolina           59062
    ## 1923 0500000US37063              Durham County, North Carolina          306457
    ## 1924 0500000US37065           Edgecombe County, North Carolina           53332
    ## 1925 0500000US37067             Forsyth County, North Carolina          371573
    ## 1926 0500000US37069            Franklin County, North Carolina           64902
    ## 1927 0500000US37071              Gaston County, North Carolina          216585
    ## 1928 0500000US37073               Gates County, North Carolina           11563
    ## 1929 0500000US37075              Graham County, North Carolina            8557
    ## 1930 0500000US37077           Granville County, North Carolina           58874
    ## 1931 0500000US37079              Greene County, North Carolina           21008
    ## 1932 0500000US37081            Guilford County, North Carolina          523582
    ## 1933 0500000US37083             Halifax County, North Carolina           51737
    ## 1934 0500000US37085             Harnett County, North Carolina          130361
    ## 1935 0500000US37087             Haywood County, North Carolina           60433
    ## 1936 0500000US37089           Henderson County, North Carolina          113625
    ## 1937 0500000US37091            Hertford County, North Carolina           24153
    ## 1938 0500000US37093                Hoke County, North Carolina           53239
    ## 1939 0500000US37095                Hyde County, North Carolina            5393
    ## 1940 0500000US37097             Iredell County, North Carolina          172525
    ## 1941 0500000US37099             Jackson County, North Carolina           42256
    ## 1942 0500000US37101            Johnston County, North Carolina          191172
    ## 1943 0500000US37103               Jones County, North Carolina            9695
    ## 1944 0500000US37105                 Lee County, North Carolina           60125
    ## 1945 0500000US37107              Lenoir County, North Carolina           57227
    ## 1946 0500000US37109             Lincoln County, North Carolina           81441
    ## 1947 0500000US37111            McDowell County, North Carolina           45109
    ## 1948 0500000US37113               Macon County, North Carolina           34410
    ## 1949 0500000US37115             Madison County, North Carolina           21405
    ## 1950 0500000US37117              Martin County, North Carolina           23054
    ## 1951 0500000US37119         Mecklenburg County, North Carolina         1054314
    ## 1952 0500000US37121            Mitchell County, North Carolina           15040
    ## 1953 0500000US37123          Montgomery County, North Carolina           27338
    ## 1954 0500000US37125               Moore County, North Carolina           95629
    ## 1955 0500000US37127                Nash County, North Carolina           94003
    ## 1956 0500000US37129         New Hanover County, North Carolina          224231
    ## 1957 0500000US37131         Northampton County, North Carolina           20186
    ## 1958 0500000US37133              Onslow County, North Carolina          193912
    ## 1959 0500000US37135              Orange County, North Carolina          142938
    ## 1960 0500000US37137             Pamlico County, North Carolina           12742
    ## 1961 0500000US37139          Pasquotank County, North Carolina           39479
    ## 1962 0500000US37141              Pender County, North Carolina           59020
    ## 1963 0500000US37143          Perquimans County, North Carolina           13459
    ## 1964 0500000US37145              Person County, North Carolina           39305
    ## 1965 0500000US37147                Pitt County, North Carolina          177372
    ## 1966 0500000US37149                Polk County, North Carolina           20458
    ## 1967 0500000US37151            Randolph County, North Carolina          142958
    ## 1968 0500000US37153            Richmond County, North Carolina           45189
    ## 1969 0500000US37155             Robeson County, North Carolina          133442
    ## 1970 0500000US37157          Rockingham County, North Carolina           91270
    ## 1971 0500000US37159               Rowan County, North Carolina          139605
    ## 1972 0500000US37161          Rutherford County, North Carolina           66532
    ## 1973 0500000US37163             Sampson County, North Carolina           63561
    ## 1974 0500000US37165            Scotland County, North Carolina           35262
    ## 1975 0500000US37167              Stanly County, North Carolina           61114
    ## 1976 0500000US37169              Stokes County, North Carolina           45905
    ## 1977 0500000US37171               Surry County, North Carolina           72099
    ## 1978 0500000US37173               Swain County, North Carolina           14254
    ## 1979 0500000US37175        Transylvania County, North Carolina           33513
    ## 1980 0500000US37177             Tyrrell County, North Carolina            4119
    ## 1981 0500000US37179               Union County, North Carolina          226694
    ## 1982 0500000US37181               Vance County, North Carolina           44482
    ## 1983 0500000US37183                Wake County, North Carolina         1046558
    ## 1984 0500000US37185              Warren County, North Carolina           20033
    ## 1985 0500000US37187          Washington County, North Carolina           12156
    ## 1986 0500000US37189             Watauga County, North Carolina           54117
    ## 1987 0500000US37191               Wayne County, North Carolina          124002
    ## 1988 0500000US37193              Wilkes County, North Carolina           68460
    ## 1989 0500000US37195              Wilson County, North Carolina           81336
    ## 1990 0500000US37197              Yadkin County, North Carolina           37665
    ## 1991 0500000US37199              Yancey County, North Carolina           17667
    ## 1992 0500000US38001                 Adams County, North Dakota            2351
    ## 1993 0500000US38003                Barnes County, North Dakota           10836
    ## 1994 0500000US38005                Benson County, North Dakota            6886
    ## 1995 0500000US38007              Billings County, North Dakota             946
    ## 1996 0500000US38009             Bottineau County, North Dakota            6589
    ## 1997 0500000US38011                Bowman County, North Dakota            3195
    ## 1998 0500000US38013                 Burke County, North Dakota            2213
    ## 1999 0500000US38015              Burleigh County, North Dakota           93737
    ## 2000 0500000US38017                  Cass County, North Dakota          174202
    ## 2001 0500000US38019              Cavalier County, North Dakota            3824
    ## 2002 0500000US38021                Dickey County, North Dakota            4970
    ## 2003 0500000US38023                Divide County, North Dakota            2369
    ## 2004 0500000US38025                  Dunn County, North Dakota            4387
    ## 2005 0500000US38027                  Eddy County, North Dakota            2313
    ## 2006 0500000US38029                Emmons County, North Dakota            3352
    ## 2007 0500000US38031                Foster County, North Dakota            3290
    ## 2008 0500000US38033         Golden Valley County, North Dakota            1882
    ## 2009 0500000US38035           Grand Forks County, North Dakota           70400
    ## 2010 0500000US38037                 Grant County, North Dakota            2380
    ## 2011 0500000US38039                Griggs County, North Dakota            2266
    ## 2012 0500000US38041             Hettinger County, North Dakota            2576
    ## 2013 0500000US38043                Kidder County, North Dakota            2460
    ## 2014 0500000US38045               LaMoure County, North Dakota            4100
    ## 2015 0500000US38047                 Logan County, North Dakota            1927
    ## 2016 0500000US38049               McHenry County, North Dakota            5927
    ## 2017 0500000US38051              McIntosh County, North Dakota            2654
    ## 2018 0500000US38053              McKenzie County, North Dakota           12536
    ## 2019 0500000US38055                McLean County, North Dakota            9608
    ## 2020 0500000US38057                Mercer County, North Dakota            8570
    ## 2021 0500000US38059                Morton County, North Dakota           30544
    ## 2022 0500000US38061             Mountrail County, North Dakota           10152
    ## 2023 0500000US38063                Nelson County, North Dakota            2920
    ## 2024 0500000US38065                Oliver County, North Dakota            1837
    ## 2025 0500000US38067               Pembina County, North Dakota            7016
    ## 2026 0500000US38069                Pierce County, North Dakota            4210
    ## 2027 0500000US38071                Ramsey County, North Dakota           11557
    ## 2028 0500000US38073                Ransom County, North Dakota            5361
    ## 2029 0500000US38075              Renville County, North Dakota            2495
    ## 2030 0500000US38077              Richland County, North Dakota           16288
    ## 2031 0500000US38079               Rolette County, North Dakota           14603
    ## 2032 0500000US38081               Sargent County, North Dakota            3883
    ## 2033 0500000US38083              Sheridan County, North Dakota            1405
    ## 2034 0500000US38085                 Sioux County, North Dakota            4413
    ## 2035 0500000US38087                 Slope County, North Dakota             704
    ## 2036 0500000US38089                 Stark County, North Dakota           30876
    ## 2037 0500000US38091                Steele County, North Dakota            1910
    ## 2038 0500000US38093              Stutsman County, North Dakota           21064
    ## 2039 0500000US38095                Towner County, North Dakota            2246
    ## 2040 0500000US38097                Traill County, North Dakota            8019
    ## 2041 0500000US38099                 Walsh County, North Dakota           10802
    ## 2042 0500000US38101                  Ward County, North Dakota           69034
    ## 2043 0500000US38103                 Wells County, North Dakota            4055
    ## 2044 0500000US38105              Williams County, North Dakota           34061
    ## 2045 0500000US39001                         Adams County, Ohio           27878
    ## 2046 0500000US39003                         Allen County, Ohio          103642
    ## 2047 0500000US39005                       Ashland County, Ohio           53477
    ## 2048 0500000US39007                     Ashtabula County, Ohio           98136
    ## 2049 0500000US39009                        Athens County, Ohio           65936
    ## 2050 0500000US39011                      Auglaize County, Ohio           45784
    ## 2051 0500000US39013                       Belmont County, Ohio           68472
    ## 2052 0500000US39015                         Brown County, Ohio           43679
    ## 2053 0500000US39017                        Butler County, Ohio          378294
    ## 2054 0500000US39019                       Carroll County, Ohio           27578
    ## 2055 0500000US39021                     Champaign County, Ohio           38864
    ## 2056 0500000US39023                         Clark County, Ohio          135198
    ## 2057 0500000US39025                      Clermont County, Ohio          203216
    ## 2058 0500000US39027                       Clinton County, Ohio           41896
    ## 2059 0500000US39029                    Columbiana County, Ohio          104003
    ## 2060 0500000US39031                     Coshocton County, Ohio           36574
    ## 2061 0500000US39033                      Crawford County, Ohio           42021
    ## 2062 0500000US39035                      Cuyahoga County, Ohio         1253783
    ## 2063 0500000US39037                         Darke County, Ohio           51734
    ## 2064 0500000US39039                      Defiance County, Ohio           38279
    ## 2065 0500000US39041                      Delaware County, Ohio          197008
    ## 2066 0500000US39043                          Erie County, Ohio           75136
    ## 2067 0500000US39045                     Fairfield County, Ohio          152910
    ## 2068 0500000US39047                       Fayette County, Ohio           28645
    ## 2069 0500000US39049                      Franklin County, Ohio         1275333
    ## 2070 0500000US39051                        Fulton County, Ohio           42305
    ## 2071 0500000US39053                        Gallia County, Ohio           30195
    ## 2072 0500000US39055                        Geauga County, Ohio           93961
    ## 2073 0500000US39057                        Greene County, Ohio          165811
    ## 2074 0500000US39059                      Guernsey County, Ohio           39274
    ## 2075 0500000US39061                      Hamilton County, Ohio          812037
    ## 2076 0500000US39063                       Hancock County, Ohio           75690
    ## 2077 0500000US39065                        Hardin County, Ohio           31542
    ## 2078 0500000US39067                      Harrison County, Ohio           15307
    ## 2079 0500000US39069                         Henry County, Ohio           27316
    ## 2080 0500000US39071                      Highland County, Ohio           43007
    ## 2081 0500000US39073                       Hocking County, Ohio           28495
    ## 2082 0500000US39075                        Holmes County, Ohio           43859
    ## 2083 0500000US39077                         Huron County, Ohio           58457
    ## 2084 0500000US39079                       Jackson County, Ohio           32524
    ## 2085 0500000US39081                     Jefferson County, Ohio           66886
    ## 2086 0500000US39083                          Knox County, Ohio           61215
    ## 2087 0500000US39085                          Lake County, Ohio          230052
    ## 2088 0500000US39087                      Lawrence County, Ohio           60622
    ## 2089 0500000US39089                       Licking County, Ohio          172293
    ## 2090 0500000US39091                         Logan County, Ohio           45307
    ## 2091 0500000US39093                        Lorain County, Ohio          306713
    ## 2092 0500000US39095                         Lucas County, Ohio          432379
    ## 2093 0500000US39097                       Madison County, Ohio           43988
    ## 2094 0500000US39099                      Mahoning County, Ohio          231064
    ## 2095 0500000US39101                        Marion County, Ohio           65344
    ## 2096 0500000US39103                        Medina County, Ohio          177257
    ## 2097 0500000US39105                         Meigs County, Ohio           23160
    ## 2098 0500000US39107                        Mercer County, Ohio           40806
    ## 2099 0500000US39109                         Miami County, Ohio          104800
    ## 2100 0500000US39111                        Monroe County, Ohio           14090
    ## 2101 0500000US39113                    Montgomery County, Ohio          532034
    ## 2102 0500000US39115                        Morgan County, Ohio           14702
    ## 2103 0500000US39117                        Morrow County, Ohio           34976
    ## 2104 0500000US39119                     Muskingum County, Ohio           86076
    ## 2105 0500000US39121                         Noble County, Ohio           14443
    ## 2106 0500000US39123                        Ottawa County, Ohio           40709
    ## 2107 0500000US39125                      Paulding County, Ohio           18872
    ## 2108 0500000US39127                         Perry County, Ohio           35985
    ## 2109 0500000US39129                      Pickaway County, Ohio           57420
    ## 2110 0500000US39131                          Pike County, Ohio           28214
    ## 2111 0500000US39133                       Portage County, Ohio          162644
    ## 2112 0500000US39135                        Preble County, Ohio           41207
    ## 2113 0500000US39137                        Putnam County, Ohio           33969
    ## 2114 0500000US39139                      Richland County, Ohio          121324
    ## 2115 0500000US39141                          Ross County, Ohio           77051
    ## 2116 0500000US39143                      Sandusky County, Ohio           59299
    ## 2117 0500000US39145                        Scioto County, Ohio           76377
    ## 2118 0500000US39147                        Seneca County, Ohio           55475
    ## 2119 0500000US39149                        Shelby County, Ohio           48797
    ## 2120 0500000US39151                         Stark County, Ohio          373475
    ## 2121 0500000US39153                        Summit County, Ohio          541810
    ## 2122 0500000US39155                      Trumbull County, Ohio          201794
    ## 2123 0500000US39157                    Tuscarawas County, Ohio           92526
    ## 2124 0500000US39159                         Union County, Ohio           55654
    ## 2125 0500000US39161                      Van Wert County, Ohio           28281
    ## 2126 0500000US39163                        Vinton County, Ohio           13111
    ## 2127 0500000US39165                        Warren County, Ohio          226564
    ## 2128 0500000US39167                    Washington County, Ohio           60671
    ## 2129 0500000US39169                         Wayne County, Ohio          116208
    ## 2130 0500000US39171                      Williams County, Ohio           36936
    ## 2131 0500000US39173                          Wood County, Ohio          129936
    ## 2132 0500000US39175                       Wyandot County, Ohio           22107
    ## 2133 0500000US40001                     Adair County, Oklahoma           22113
    ## 2134 0500000US40003                   Alfalfa County, Oklahoma            5857
    ## 2135 0500000US40005                     Atoka County, Oklahoma           13874
    ## 2136 0500000US40007                    Beaver County, Oklahoma            5415
    ## 2137 0500000US40009                   Beckham County, Oklahoma           22621
    ## 2138 0500000US40011                    Blaine County, Oklahoma            9634
    ## 2139 0500000US40013                     Bryan County, Oklahoma           45759
    ## 2140 0500000US40015                     Caddo County, Oklahoma           29342
    ## 2141 0500000US40017                  Canadian County, Oklahoma          136710
    ## 2142 0500000US40019                    Carter County, Oklahoma           48406
    ## 2143 0500000US40021                  Cherokee County, Oklahoma           48599
    ## 2144 0500000US40023                   Choctaw County, Oklahoma           14886
    ## 2145 0500000US40025                  Cimarron County, Oklahoma            2189
    ## 2146 0500000US40027                 Cleveland County, Oklahoma          276733
    ## 2147 0500000US40029                      Coal County, Oklahoma            5618
    ## 2148 0500000US40031                  Comanche County, Oklahoma          122561
    ## 2149 0500000US40033                    Cotton County, Oklahoma            5929
    ## 2150 0500000US40035                     Craig County, Oklahoma           14493
    ## 2151 0500000US40037                     Creek County, Oklahoma           71160
    ## 2152 0500000US40039                    Custer County, Oklahoma           29209
    ## 2153 0500000US40041                  Delaware County, Oklahoma           42112
    ## 2154 0500000US40043                     Dewey County, Oklahoma            4918
    ## 2155 0500000US40045                     Ellis County, Oklahoma            4072
    ## 2156 0500000US40047                  Garfield County, Oklahoma           62190
    ## 2157 0500000US40049                    Garvin County, Oklahoma           27823
    ## 2158 0500000US40051                     Grady County, Oklahoma           54733
    ## 2159 0500000US40053                     Grant County, Oklahoma            4418
    ## 2160 0500000US40055                     Greer County, Oklahoma            5943
    ## 2161 0500000US40057                    Harmon County, Oklahoma            2721
    ## 2162 0500000US40059                    Harper County, Oklahoma            3847
    ## 2163 0500000US40061                   Haskell County, Oklahoma           12704
    ## 2164 0500000US40063                    Hughes County, Oklahoma           13460
    ## 2165 0500000US40065                   Jackson County, Oklahoma           25384
    ## 2166 0500000US40067                 Jefferson County, Oklahoma            6223
    ## 2167 0500000US40069                  Johnston County, Oklahoma           11041
    ## 2168 0500000US40071                       Kay County, Oklahoma           44880
    ## 2169 0500000US40073                Kingfisher County, Oklahoma           15618
    ## 2170 0500000US40075                     Kiowa County, Oklahoma            9001
    ## 2171 0500000US40077                   Latimer County, Oklahoma           10495
    ## 2172 0500000US40079                  Le Flore County, Oklahoma           49909
    ## 2173 0500000US40081                   Lincoln County, Oklahoma           34854
    ## 2174 0500000US40083                     Logan County, Oklahoma           46044
    ## 2175 0500000US40085                      Love County, Oklahoma            9933
    ## 2176 0500000US40087                   McClain County, Oklahoma           38634
    ## 2177 0500000US40089                 McCurtain County, Oklahoma           32966
    ## 2178 0500000US40091                  McIntosh County, Oklahoma           19819
    ## 2179 0500000US40093                     Major County, Oklahoma            7718
    ## 2180 0500000US40095                  Marshall County, Oklahoma           16376
    ## 2181 0500000US40097                     Mayes County, Oklahoma           40980
    ## 2182 0500000US40099                    Murray County, Oklahoma           13875
    ## 2183 0500000US40101                  Muskogee County, Oklahoma           69084
    ## 2184 0500000US40103                     Noble County, Oklahoma           11411
    ## 2185 0500000US40105                    Nowata County, Oklahoma           10383
    ## 2186 0500000US40107                  Okfuskee County, Oklahoma           12115
    ## 2187 0500000US40109                  Oklahoma County, Oklahoma          782051
    ## 2188 0500000US40111                  Okmulgee County, Oklahoma           38889
    ## 2189 0500000US40113                     Osage County, Oklahoma           47311
    ## 2190 0500000US40115                    Ottawa County, Oklahoma           31566
    ## 2191 0500000US40117                    Pawnee County, Oklahoma           16428
    ## 2192 0500000US40119                     Payne County, Oklahoma           81512
    ## 2193 0500000US40121                 Pittsburg County, Oklahoma           44382
    ## 2194 0500000US40123                  Pontotoc County, Oklahoma           38358
    ## 2195 0500000US40125              Pottawatomie County, Oklahoma           72000
    ## 2196 0500000US40127                Pushmataha County, Oklahoma           11119
    ## 2197 0500000US40129               Roger Mills County, Oklahoma            3708
    ## 2198 0500000US40131                    Rogers County, Oklahoma           90814
    ## 2199 0500000US40133                  Seminole County, Oklahoma           25071
    ## 2200 0500000US40135                  Sequoyah County, Oklahoma           41359
    ## 2201 0500000US40137                  Stephens County, Oklahoma           43983
    ## 2202 0500000US40139                     Texas County, Oklahoma           21121
    ## 2203 0500000US40141                   Tillman County, Oklahoma            7515
    ## 2204 0500000US40143                     Tulsa County, Oklahoma          642781
    ## 2205 0500000US40145                   Wagoner County, Oklahoma           77850
    ## 2206 0500000US40147                Washington County, Oklahoma           52001
    ## 2207 0500000US40149                   Washita County, Oklahoma           11432
    ## 2208 0500000US40151                     Woods County, Oklahoma            9127
    ## 2209 0500000US40153                  Woodward County, Oklahoma           20967
    ## 2210 0500000US41001                       Baker County, Oregon           15984
    ## 2211 0500000US41003                      Benton County, Oregon           89780
    ## 2212 0500000US41005                   Clackamas County, Oregon          405788
    ## 2213 0500000US41007                     Clatsop County, Oregon           38562
    ## 2214 0500000US41009                    Columbia County, Oregon           50851
    ## 2215 0500000US41011                        Coos County, Oregon           63308
    ## 2216 0500000US41013                       Crook County, Oregon           22337
    ## 2217 0500000US41015                       Curry County, Oregon           22507
    ## 2218 0500000US41017                   Deschutes County, Oregon          180640
    ## 2219 0500000US41019                     Douglas County, Oregon          108323
    ## 2220 0500000US41021                     Gilliam County, Oregon            1907
    ## 2221 0500000US41023                       Grant County, Oregon            7183
    ## 2222 0500000US41025                      Harney County, Oregon            7228
    ## 2223 0500000US41027                  Hood River County, Oregon           23131
    ## 2224 0500000US41029                     Jackson County, Oregon          214267
    ## 2225 0500000US41031                   Jefferson County, Oregon           23143
    ## 2226 0500000US41033                   Josephine County, Oregon           85481
    ## 2227 0500000US41035                     Klamath County, Oregon           66310
    ## 2228 0500000US41037                        Lake County, Oregon            7843
    ## 2229 0500000US41039                        Lane County, Oregon          368882
    ## 2230 0500000US41041                     Lincoln County, Oregon           47881
    ## 2231 0500000US41043                        Linn County, Oregon          122870
    ## 2232 0500000US41045                     Malheur County, Oregon           30431
    ## 2233 0500000US41047                      Marion County, Oregon          335553
    ## 2234 0500000US41049                      Morrow County, Oregon           11215
    ## 2235 0500000US41051                   Multnomah County, Oregon          798647
    ## 2236 0500000US41053                        Polk County, Oregon           81427
    ## 2237 0500000US41055                     Sherman County, Oregon            1605
    ## 2238 0500000US41057                   Tillamook County, Oregon           26076
    ## 2239 0500000US41059                    Umatilla County, Oregon           76898
    ## 2240 0500000US41061                       Union County, Oregon           26028
    ## 2241 0500000US41063                     Wallowa County, Oregon            6924
    ## 2242 0500000US41065                       Wasco County, Oregon           25866
    ## 2243 0500000US41067                  Washington County, Oregon          581821
    ## 2244 0500000US41069                     Wheeler County, Oregon            1426
    ## 2245 0500000US41071                     Yamhill County, Oregon          103820
    ## 2246 0500000US42001                 Adams County, Pennsylvania          102023
    ## 2247 0500000US42003             Allegheny County, Pennsylvania         1225561
    ## 2248 0500000US42005             Armstrong County, Pennsylvania           66331
    ## 2249 0500000US42007                Beaver County, Pennsylvania          166896
    ## 2250 0500000US42009               Bedford County, Pennsylvania           48611
    ## 2251 0500000US42011                 Berks County, Pennsylvania          416642
    ## 2252 0500000US42013                 Blair County, Pennsylvania          123842
    ## 2253 0500000US42015              Bradford County, Pennsylvania           61304
    ## 2254 0500000US42017                 Bucks County, Pennsylvania          626370
    ## 2255 0500000US42019                Butler County, Pennsylvania          186566
    ## 2256 0500000US42021               Cambria County, Pennsylvania          134550
    ## 2257 0500000US42023               Cameron County, Pennsylvania            4686
    ## 2258 0500000US42025                Carbon County, Pennsylvania           63931
    ## 2259 0500000US42027                Centre County, Pennsylvania          161443
    ## 2260 0500000US42029               Chester County, Pennsylvania          517156
    ## 2261 0500000US42031               Clarion County, Pennsylvania           38827
    ## 2262 0500000US42033            Clearfield County, Pennsylvania           80216
    ## 2263 0500000US42035               Clinton County, Pennsylvania           39074
    ## 2264 0500000US42037              Columbia County, Pennsylvania           66220
    ## 2265 0500000US42039              Crawford County, Pennsylvania           86164
    ## 2266 0500000US42041            Cumberland County, Pennsylvania          247433
    ## 2267 0500000US42043               Dauphin County, Pennsylvania          274515
    ## 2268 0500000US42045              Delaware County, Pennsylvania          563527
    ## 2269 0500000US42047                   Elk County, Pennsylvania           30608
    ## 2270 0500000US42049                  Erie County, Pennsylvania          275972
    ## 2271 0500000US42051               Fayette County, Pennsylvania          132289
    ## 2272 0500000US42053                Forest County, Pennsylvania            7351
    ## 2273 0500000US42055              Franklin County, Pennsylvania          153751
    ## 2274 0500000US42057                Fulton County, Pennsylvania           14506
    ## 2275 0500000US42059                Greene County, Pennsylvania           37144
    ## 2276 0500000US42061            Huntingdon County, Pennsylvania           45421
    ## 2277 0500000US42063               Indiana County, Pennsylvania           85755
    ## 2278 0500000US42065             Jefferson County, Pennsylvania           44084
    ## 2279 0500000US42067               Juniata County, Pennsylvania           24562
    ## 2280 0500000US42069            Lackawanna County, Pennsylvania          211454
    ## 2281 0500000US42071             Lancaster County, Pennsylvania          538347
    ## 2282 0500000US42073              Lawrence County, Pennsylvania           87382
    ## 2283 0500000US42075               Lebanon County, Pennsylvania          138674
    ## 2284 0500000US42077                Lehigh County, Pennsylvania          362613
    ## 2285 0500000US42079               Luzerne County, Pennsylvania          317884
    ## 2286 0500000US42081              Lycoming County, Pennsylvania          114859
    ## 2287 0500000US42083                McKean County, Pennsylvania           41806
    ## 2288 0500000US42085                Mercer County, Pennsylvania          112630
    ## 2289 0500000US42087               Mifflin County, Pennsylvania           46362
    ## 2290 0500000US42089                Monroe County, Pennsylvania          167586
    ## 2291 0500000US42091            Montgomery County, Pennsylvania          821301
    ## 2292 0500000US42093               Montour County, Pennsylvania           18294
    ## 2293 0500000US42095           Northampton County, Pennsylvania          301778
    ## 2294 0500000US42097        Northumberland County, Pennsylvania           92325
    ## 2295 0500000US42099                 Perry County, Pennsylvania           45924
    ## 2296 0500000US42101          Philadelphia County, Pennsylvania         1575522
    ## 2297 0500000US42103                  Pike County, Pennsylvania           55498
    ## 2298 0500000US42105                Potter County, Pennsylvania           16937
    ## 2299 0500000US42107            Schuylkill County, Pennsylvania          143555
    ## 2300 0500000US42109                Snyder County, Pennsylvania           40466
    ## 2301 0500000US42111              Somerset County, Pennsylvania           74949
    ## 2302 0500000US42113              Sullivan County, Pennsylvania            6177
    ## 2303 0500000US42115           Susquehanna County, Pennsylvania           41340
    ## 2304 0500000US42117                 Tioga County, Pennsylvania           41226
    ## 2305 0500000US42119                 Union County, Pennsylvania           45114
    ## 2306 0500000US42121               Venango County, Pennsylvania           52376
    ## 2307 0500000US42123                Warren County, Pennsylvania           40035
    ## 2308 0500000US42125            Washington County, Pennsylvania          207547
    ## 2309 0500000US42127                 Wayne County, Pennsylvania           51536
    ## 2310 0500000US42129          Westmoreland County, Pennsylvania          354751
    ## 2311 0500000US42131               Wyoming County, Pennsylvania           27588
    ## 2312 0500000US42133                  York County, Pennsylvania          444014
    ## 2313 0500000US44001               Bristol County, Rhode Island           48900
    ## 2314 0500000US44003                  Kent County, Rhode Island          163861
    ## 2315 0500000US44005               Newport County, Rhode Island           83075
    ## 2316 0500000US44007            Providence County, Rhode Island          634533
    ## 2317 0500000US44009            Washington County, Rhode Island          126242
    ## 2318 0500000US45001           Abbeville County, South Carolina           24657
    ## 2319 0500000US45003               Aiken County, South Carolina          166926
    ## 2320 0500000US45005           Allendale County, South Carolina            9214
    ## 2321 0500000US45007            Anderson County, South Carolina          195995
    ## 2322 0500000US45009             Bamberg County, South Carolina           14600
    ## 2323 0500000US45011            Barnwell County, South Carolina           21577
    ## 2324 0500000US45013            Beaufort County, South Carolina          182658
    ## 2325 0500000US45015            Berkeley County, South Carolina          209065
    ## 2326 0500000US45017             Calhoun County, South Carolina           14713
    ## 2327 0500000US45019          Charleston County, South Carolina          394708
    ## 2328 0500000US45021            Cherokee County, South Carolina           56711
    ## 2329 0500000US45023             Chester County, South Carolina           32326
    ## 2330 0500000US45025        Chesterfield County, South Carolina           46024
    ## 2331 0500000US45027           Clarendon County, South Carolina           34017
    ## 2332 0500000US45029            Colleton County, South Carolina           37568
    ## 2333 0500000US45031          Darlington County, South Carolina           67253
    ## 2334 0500000US45033              Dillon County, South Carolina           30871
    ## 2335 0500000US45035          Dorchester County, South Carolina          155474
    ## 2336 0500000US45037           Edgefield County, South Carolina           26769
    ## 2337 0500000US45039           Fairfield County, South Carolina           22712
    ## 2338 0500000US45041            Florence County, South Carolina          138561
    ## 2339 0500000US45043          Georgetown County, South Carolina           61605
    ## 2340 0500000US45045          Greenville County, South Carolina          498402
    ## 2341 0500000US45047           Greenwood County, South Carolina           70264
    ## 2342 0500000US45049             Hampton County, South Carolina           19807
    ## 2343 0500000US45051               Horry County, South Carolina          320915
    ## 2344 0500000US45053              Jasper County, South Carolina           27900
    ## 2345 0500000US45055             Kershaw County, South Carolina           64361
    ## 2346 0500000US45057           Lancaster County, South Carolina           89546
    ## 2347 0500000US45059             Laurens County, South Carolina           66710
    ## 2348 0500000US45061                 Lee County, South Carolina           17606
    ## 2349 0500000US45063           Lexington County, South Carolina          286316
    ## 2350 0500000US45065           McCormick County, South Carolina            9606
    ## 2351 0500000US45067              Marion County, South Carolina           31562
    ## 2352 0500000US45069            Marlboro County, South Carolina           27131
    ## 2353 0500000US45071            Newberry County, South Carolina           38068
    ## 2354 0500000US45073              Oconee County, South Carolina           76696
    ## 2355 0500000US45075          Orangeburg County, South Carolina           88454
    ## 2356 0500000US45077             Pickens County, South Carolina          122746
    ## 2357 0500000US45079            Richland County, South Carolina          408263
    ## 2358 0500000US45081              Saluda County, South Carolina           20299
    ## 2359 0500000US45083         Spartanburg County, South Carolina          302195
    ## 2360 0500000US45085              Sumter County, South Carolina          106995
    ## 2361 0500000US45087               Union County, South Carolina           27644
    ## 2362 0500000US45089        Williamsburg County, South Carolina           31794
    ## 2363 0500000US45091                York County, South Carolina          258641
    ## 2364 0500000US46003                Aurora County, South Dakota            2759
    ## 2365 0500000US46005                Beadle County, South Dakota           18374
    ## 2366 0500000US46007               Bennett County, South Dakota            3437
    ## 2367 0500000US46009             Bon Homme County, South Dakota            6969
    ## 2368 0500000US46011             Brookings County, South Dakota           34239
    ## 2369 0500000US46013                 Brown County, South Dakota           38840
    ## 2370 0500000US46015                 Brule County, South Dakota            5256
    ## 2371 0500000US46017               Buffalo County, South Dakota            2053
    ## 2372 0500000US46019                 Butte County, South Dakota           10177
    ## 2373 0500000US46021              Campbell County, South Dakota            1435
    ## 2374 0500000US46023           Charles Mix County, South Dakota            9344
    ## 2375 0500000US46025                 Clark County, South Dakota            3673
    ## 2376 0500000US46027                  Clay County, South Dakota           13925
    ## 2377 0500000US46029             Codington County, South Dakota           27993
    ## 2378 0500000US46031                Corson County, South Dakota            4168
    ## 2379 0500000US46033                Custer County, South Dakota            8573
    ## 2380 0500000US46035               Davison County, South Dakota           19901
    ## 2381 0500000US46037                   Day County, South Dakota            5506
    ## 2382 0500000US46039                 Deuel County, South Dakota            4306
    ## 2383 0500000US46041                 Dewey County, South Dakota            5779
    ## 2384 0500000US46043               Douglas County, South Dakota            2930
    ## 2385 0500000US46045               Edmunds County, South Dakota            3940
    ## 2386 0500000US46047            Fall River County, South Dakota            6774
    ## 2387 0500000US46049                 Faulk County, South Dakota            2322
    ## 2388 0500000US46051                 Grant County, South Dakota            7217
    ## 2389 0500000US46053               Gregory County, South Dakota            4201
    ## 2390 0500000US46055                Haakon County, South Dakota            2082
    ## 2391 0500000US46057                Hamlin County, South Dakota            6000
    ## 2392 0500000US46059                  Hand County, South Dakota            3301
    ## 2393 0500000US46061                Hanson County, South Dakota            3397
    ## 2394 0500000US46063               Harding County, South Dakota            1311
    ## 2395 0500000US46065                Hughes County, South Dakota           17617
    ## 2396 0500000US46067            Hutchinson County, South Dakota            7315
    ## 2397 0500000US46069                  Hyde County, South Dakota            1331
    ## 2398 0500000US46071               Jackson County, South Dakota            3287
    ## 2399 0500000US46073               Jerauld County, South Dakota            2029
    ## 2400 0500000US46075                 Jones County, South Dakota             735
    ## 2401 0500000US46077             Kingsbury County, South Dakota            4967
    ## 2402 0500000US46079                  Lake County, South Dakota           12574
    ## 2403 0500000US46081              Lawrence County, South Dakota           25234
    ## 2404 0500000US46083               Lincoln County, South Dakota           54914
    ## 2405 0500000US46085                 Lyman County, South Dakota            3869
    ## 2406 0500000US46087                McCook County, South Dakota            5511
    ## 2407 0500000US46089             McPherson County, South Dakota            2364
    ## 2408 0500000US46091              Marshall County, South Dakota            4895
    ## 2409 0500000US46093                 Meade County, South Dakota           27424
    ## 2410 0500000US46095              Mellette County, South Dakota            2055
    ## 2411 0500000US46097                 Miner County, South Dakota            2229
    ## 2412 0500000US46099             Minnehaha County, South Dakota          186749
    ## 2413 0500000US46101                 Moody County, South Dakota            6506
    ## 2414 0500000US46102         Oglala Lakota County, South Dakota           14335
    ## 2415 0500000US46103            Pennington County, South Dakota          109294
    ## 2416 0500000US46105               Perkins County, South Dakota            2907
    ## 2417 0500000US46107                Potter County, South Dakota            2326
    ## 2418 0500000US46109               Roberts County, South Dakota           10285
    ## 2419 0500000US46111               Sanborn County, South Dakota            2388
    ## 2420 0500000US46115                 Spink County, South Dakota            6543
    ## 2421 0500000US46117               Stanley County, South Dakota            2997
    ## 2422 0500000US46119                 Sully County, South Dakota            1331
    ## 2423 0500000US46121                  Todd County, South Dakota           10146
    ## 2424 0500000US46123                 Tripp County, South Dakota            5468
    ## 2425 0500000US46125                Turner County, South Dakota            8264
    ## 2426 0500000US46127                 Union County, South Dakota           15177
    ## 2427 0500000US46129              Walworth County, South Dakota            5510
    ## 2428 0500000US46135               Yankton County, South Dakota           22717
    ## 2429 0500000US46137               Ziebach County, South Dakota            2814
    ## 2430 0500000US47001                 Anderson County, Tennessee           75775
    ## 2431 0500000US47003                  Bedford County, Tennessee           47558
    ## 2432 0500000US47005                   Benton County, Tennessee           16112
    ## 2433 0500000US47007                  Bledsoe County, Tennessee           14602
    ## 2434 0500000US47009                   Blount County, Tennessee          128443
    ## 2435 0500000US47011                  Bradley County, Tennessee          104557
    ## 2436 0500000US47013                 Campbell County, Tennessee           39687
    ## 2437 0500000US47015                   Cannon County, Tennessee           13976
    ## 2438 0500000US47017                  Carroll County, Tennessee           28018
    ## 2439 0500000US47019                   Carter County, Tennessee           56391
    ## 2440 0500000US47021                 Cheatham County, Tennessee           39929
    ## 2441 0500000US47023                  Chester County, Tennessee           17150
    ## 2442 0500000US47025                Claiborne County, Tennessee           31613
    ## 2443 0500000US47027                     Clay County, Tennessee            7686
    ## 2444 0500000US47029                    Cocke County, Tennessee           35336
    ## 2445 0500000US47031                   Coffee County, Tennessee           54531
    ## 2446 0500000US47033                 Crockett County, Tennessee           14499
    ## 2447 0500000US47035               Cumberland County, Tennessee           58634
    ## 2448 0500000US47037                 Davidson County, Tennessee          684017
    ## 2449 0500000US47039                  Decatur County, Tennessee           11683
    ## 2450 0500000US47041                   DeKalb County, Tennessee           19601
    ## 2451 0500000US47043                  Dickson County, Tennessee           51988
    ## 2452 0500000US47045                     Dyer County, Tennessee           37576
    ## 2453 0500000US47047                  Fayette County, Tennessee           39692
    ## 2454 0500000US47049                 Fentress County, Tennessee           17994
    ## 2455 0500000US47051                 Franklin County, Tennessee           41512
    ## 2456 0500000US47053                   Gibson County, Tennessee           49175
    ## 2457 0500000US47055                    Giles County, Tennessee           29167
    ## 2458 0500000US47057                 Grainger County, Tennessee           23013
    ## 2459 0500000US47059                   Greene County, Tennessee           68669
    ## 2460 0500000US47061                   Grundy County, Tennessee           13331
    ## 2461 0500000US47063                  Hamblen County, Tennessee           63740
    ## 2462 0500000US47065                 Hamilton County, Tennessee          357546
    ## 2463 0500000US47067                  Hancock County, Tennessee            6585
    ## 2464 0500000US47069                 Hardeman County, Tennessee           25562
    ## 2465 0500000US47071                   Hardin County, Tennessee           25771
    ## 2466 0500000US47073                  Hawkins County, Tennessee           56402
    ## 2467 0500000US47075                  Haywood County, Tennessee           17779
    ## 2468 0500000US47077                Henderson County, Tennessee           27859
    ## 2469 0500000US47079                    Henry County, Tennessee           32279
    ## 2470 0500000US47081                  Hickman County, Tennessee           24678
    ## 2471 0500000US47083                  Houston County, Tennessee            8176
    ## 2472 0500000US47085                Humphreys County, Tennessee           18318
    ## 2473 0500000US47087                  Jackson County, Tennessee           11615
    ## 2474 0500000US47089                Jefferson County, Tennessee           53247
    ## 2475 0500000US47091                  Johnson County, Tennessee           17789
    ## 2476 0500000US47093                     Knox County, Tennessee          456185
    ## 2477 0500000US47095                     Lake County, Tennessee            7526
    ## 2478 0500000US47097               Lauderdale County, Tennessee           26297
    ## 2479 0500000US47099                 Lawrence County, Tennessee           42937
    ## 2480 0500000US47101                    Lewis County, Tennessee           11956
    ## 2481 0500000US47103                  Lincoln County, Tennessee           33711
    ## 2482 0500000US47105                   Loudon County, Tennessee           51610
    ## 2483 0500000US47107                   McMinn County, Tennessee           52773
    ## 2484 0500000US47109                  McNairy County, Tennessee           25903
    ## 2485 0500000US47111                    Macon County, Tennessee           23487
    ## 2486 0500000US47113                  Madison County, Tennessee           97682
    ## 2487 0500000US47115                   Marion County, Tennessee           28417
    ## 2488 0500000US47117                 Marshall County, Tennessee           32269
    ## 2489 0500000US47119                    Maury County, Tennessee           89776
    ## 2490 0500000US47121                    Meigs County, Tennessee           11962
    ## 2491 0500000US47123                   Monroe County, Tennessee           45876
    ## 2492 0500000US47125               Montgomery County, Tennessee          196387
    ## 2493 0500000US47127                    Moore County, Tennessee            6322
    ## 2494 0500000US47129                   Morgan County, Tennessee           21596
    ## 2495 0500000US47131                    Obion County, Tennessee           30520
    ## 2496 0500000US47133                  Overton County, Tennessee           22004
    ## 2497 0500000US47135                    Perry County, Tennessee            7912
    ## 2498 0500000US47137                  Pickett County, Tennessee            5088
    ## 2499 0500000US47139                     Polk County, Tennessee           16782
    ## 2500 0500000US47141                   Putnam County, Tennessee           76440
    ## 2501 0500000US47143                     Rhea County, Tennessee           32628
    ## 2502 0500000US47145                    Roane County, Tennessee           52897
    ## 2503 0500000US47147                Robertson County, Tennessee           69344
    ## 2504 0500000US47149               Rutherford County, Tennessee          307128
    ## 2505 0500000US47151                    Scott County, Tennessee           21954
    ## 2506 0500000US47153               Sequatchie County, Tennessee           14730
    ## 2507 0500000US47155                   Sevier County, Tennessee           96287
    ## 2508 0500000US47157                   Shelby County, Tennessee          937005
    ## 2509 0500000US47159                    Smith County, Tennessee           19458
    ## 2510 0500000US47161                  Stewart County, Tennessee           13301
    ## 2511 0500000US47163                 Sullivan County, Tennessee          156734
    ## 2512 0500000US47165                   Sumner County, Tennessee          179473
    ## 2513 0500000US47167                   Tipton County, Tennessee           61446
    ## 2514 0500000US47169                Trousdale County, Tennessee            9573
    ## 2515 0500000US47171                   Unicoi County, Tennessee           17780
    ## 2516 0500000US47173                    Union County, Tennessee           19293
    ## 2517 0500000US47175                Van Buren County, Tennessee            5704
    ## 2518 0500000US47177                   Warren County, Tennessee           40454
    ## 2519 0500000US47179               Washington County, Tennessee          127055
    ## 2520 0500000US47181                    Wayne County, Tennessee           16649
    ## 2521 0500000US47183                  Weakley County, Tennessee           33626
    ## 2522 0500000US47185                    White County, Tennessee           26580
    ## 2523 0500000US47187               Williamson County, Tennessee          218648
    ## 2524 0500000US47189                   Wilson County, Tennessee          132663
    ## 2525 0500000US48001                     Anderson County, Texas           57863
    ## 2526 0500000US48003                      Andrews County, Texas           17818
    ## 2527 0500000US48005                     Angelina County, Texas           87607
    ## 2528 0500000US48007                      Aransas County, Texas           24763
    ## 2529 0500000US48009                       Archer County, Texas            8789
    ## 2530 0500000US48011                    Armstrong County, Texas            1916
    ## 2531 0500000US48013                     Atascosa County, Texas           48828
    ## 2532 0500000US48015                       Austin County, Texas           29565
    ## 2533 0500000US48017                       Bailey County, Texas            7092
    ## 2534 0500000US48019                      Bandera County, Texas           21763
    ## 2535 0500000US48021                      Bastrop County, Texas           82577
    ## 2536 0500000US48023                       Baylor County, Texas            3591
    ## 2537 0500000US48025                          Bee County, Texas           32691
    ## 2538 0500000US48027                         Bell County, Texas          342236
    ## 2539 0500000US48029                        Bexar County, Texas         1925865
    ## 2540 0500000US48031                       Blanco County, Texas           11279
    ## 2541 0500000US48033                       Borden County, Texas             665
    ## 2542 0500000US48035                       Bosque County, Texas           18122
    ## 2543 0500000US48037                        Bowie County, Texas           93858
    ## 2544 0500000US48039                     Brazoria County, Texas          353999
    ## 2545 0500000US48041                       Brazos County, Texas          219193
    ## 2546 0500000US48043                     Brewster County, Texas            9216
    ## 2547 0500000US48045                      Briscoe County, Texas            1546
    ## 2548 0500000US48047                       Brooks County, Texas            7180
    ## 2549 0500000US48049                        Brown County, Texas           37834
    ## 2550 0500000US48051                     Burleson County, Texas           17863
    ## 2551 0500000US48053                       Burnet County, Texas           45750
    ## 2552 0500000US48055                     Caldwell County, Texas           41401
    ## 2553 0500000US48057                      Calhoun County, Texas           21807
    ## 2554 0500000US48059                     Callahan County, Texas           13770
    ## 2555 0500000US48061                      Cameron County, Texas          421750
    ## 2556 0500000US48063                         Camp County, Texas           12813
    ## 2557 0500000US48065                       Carson County, Texas            6032
    ## 2558 0500000US48067                         Cass County, Texas           30087
    ## 2559 0500000US48069                       Castro County, Texas            7787
    ## 2560 0500000US48071                     Chambers County, Texas           40292
    ## 2561 0500000US48073                     Cherokee County, Texas           51903
    ## 2562 0500000US48075                    Childress County, Texas            7226
    ## 2563 0500000US48077                         Clay County, Texas           10387
    ## 2564 0500000US48079                      Cochran County, Texas            2904
    ## 2565 0500000US48081                         Coke County, Texas            3275
    ## 2566 0500000US48083                      Coleman County, Texas            8391
    ## 2567 0500000US48085                       Collin County, Texas          944350
    ## 2568 0500000US48087                Collingsworth County, Texas            2996
    ## 2569 0500000US48089                     Colorado County, Texas           21022
    ## 2570 0500000US48091                        Comal County, Texas          135097
    ## 2571 0500000US48093                     Comanche County, Texas           13495
    ## 2572 0500000US48095                       Concho County, Texas            4233
    ## 2573 0500000US48097                        Cooke County, Texas           39571
    ## 2574 0500000US48099                      Coryell County, Texas           75389
    ## 2575 0500000US48101                       Cottle County, Texas            1623
    ## 2576 0500000US48103                        Crane County, Texas            4839
    ## 2577 0500000US48105                     Crockett County, Texas            3633
    ## 2578 0500000US48107                       Crosby County, Texas            5861
    ## 2579 0500000US48109                    Culberson County, Texas            2241
    ## 2580 0500000US48111                       Dallam County, Texas            7243
    ## 2581 0500000US48113                       Dallas County, Texas         2586552
    ## 2582 0500000US48115                       Dawson County, Texas           12964
    ## 2583 0500000US48117                   Deaf Smith County, Texas           18899
    ## 2584 0500000US48119                        Delta County, Texas            5215
    ## 2585 0500000US48121                       Denton County, Texas          807047
    ## 2586 0500000US48123                       DeWitt County, Texas           20435
    ## 2587 0500000US48125                      Dickens County, Texas            2216
    ## 2588 0500000US48127                       Dimmit County, Texas           10663
    ## 2589 0500000US48129                       Donley County, Texas            3387
    ## 2590 0500000US48131                        Duval County, Texas           11355
    ## 2591 0500000US48133                     Eastland County, Texas           18270
    ## 2592 0500000US48135                        Ector County, Texas          158342
    ## 2593 0500000US48137                      Edwards County, Texas            2055
    ## 2594 0500000US48139                        Ellis County, Texas          168838
    ## 2595 0500000US48141                      El Paso County, Texas          837654
    ## 2596 0500000US48143                        Erath County, Texas           41482
    ## 2597 0500000US48145                        Falls County, Texas           17299
    ## 2598 0500000US48147                       Fannin County, Texas           34175
    ## 2599 0500000US48149                      Fayette County, Texas           25066
    ## 2600 0500000US48151                       Fisher County, Texas            3883
    ## 2601 0500000US48153                        Floyd County, Texas            5872
    ## 2602 0500000US48155                        Foard County, Texas            1408
    ## 2603 0500000US48157                    Fort Bend County, Texas          739342
    ## 2604 0500000US48159                     Franklin County, Texas           10679
    ## 2605 0500000US48161                    Freestone County, Texas           19709
    ## 2606 0500000US48163                         Frio County, Texas           19394
    ## 2607 0500000US48165                       Gaines County, Texas           20321
    ## 2608 0500000US48167                    Galveston County, Texas          327089
    ## 2609 0500000US48169                        Garza County, Texas            6288
    ## 2610 0500000US48171                    Gillespie County, Texas           26208
    ## 2611 0500000US48173                    Glasscock County, Texas            1430
    ## 2612 0500000US48175                       Goliad County, Texas            7531
    ## 2613 0500000US48177                     Gonzales County, Texas           20667
    ## 2614 0500000US48179                         Gray County, Texas           22685
    ## 2615 0500000US48181                      Grayson County, Texas          128560
    ## 2616 0500000US48183                        Gregg County, Texas          123494
    ## 2617 0500000US48185                       Grimes County, Texas           27630
    ## 2618 0500000US48187                    Guadalupe County, Texas          155137
    ## 2619 0500000US48189                         Hale County, Texas           34113
    ## 2620 0500000US48191                         Hall County, Texas            3074
    ## 2621 0500000US48193                     Hamilton County, Texas            8269
    ## 2622 0500000US48195                     Hansford County, Texas            5547
    ## 2623 0500000US48197                     Hardeman County, Texas            3952
    ## 2624 0500000US48199                       Hardin County, Texas           56379
    ## 2625 0500000US48201                       Harris County, Texas         4602523
    ## 2626 0500000US48203                     Harrison County, Texas           66645
    ## 2627 0500000US48205                      Hartley County, Texas            5767
    ## 2628 0500000US48207                      Haskell County, Texas            5809
    ## 2629 0500000US48209                         Hays County, Texas          204150
    ## 2630 0500000US48211                     Hemphill County, Texas            4061
    ## 2631 0500000US48213                    Henderson County, Texas           80460
    ## 2632 0500000US48215                      Hidalgo County, Texas          849389
    ## 2633 0500000US48217                         Hill County, Texas           35399
    ## 2634 0500000US48219                      Hockley County, Texas           23162
    ## 2635 0500000US48221                         Hood County, Texas           56901
    ## 2636 0500000US48223                      Hopkins County, Texas           36240
    ## 2637 0500000US48225                      Houston County, Texas           22955
    ## 2638 0500000US48227                       Howard County, Texas           36667
    ## 2639 0500000US48229                     Hudspeth County, Texas            4098
    ## 2640 0500000US48231                         Hunt County, Texas           92152
    ## 2641 0500000US48233                   Hutchinson County, Texas           21571
    ## 2642 0500000US48235                        Irion County, Texas            1524
    ## 2643 0500000US48237                         Jack County, Texas            8842
    ## 2644 0500000US48239                      Jackson County, Texas           14820
    ## 2645 0500000US48241                       Jasper County, Texas           35504
    ## 2646 0500000US48243                   Jeff Davis County, Texas            2234
    ## 2647 0500000US48245                    Jefferson County, Texas          255210
    ## 2648 0500000US48247                     Jim Hogg County, Texas            5282
    ## 2649 0500000US48249                    Jim Wells County, Texas           41192
    ## 2650 0500000US48251                      Johnson County, Texas          163475
    ## 2651 0500000US48253                        Jones County, Texas           19891
    ## 2652 0500000US48255                       Karnes County, Texas           15387
    ## 2653 0500000US48257                      Kaufman County, Texas          118910
    ## 2654 0500000US48259                      Kendall County, Texas           41982
    ## 2655 0500000US48261                       Kenedy County, Texas             595
    ## 2656 0500000US48263                         Kent County, Texas             749
    ## 2657 0500000US48265                         Kerr County, Texas           51365
    ## 2658 0500000US48267                       Kimble County, Texas            4408
    ## 2659 0500000US48269                         King County, Texas             228
    ## 2660 0500000US48271                       Kinney County, Texas            3675
    ## 2661 0500000US48273                      Kleberg County, Texas           31425
    ## 2662 0500000US48275                         Knox County, Texas            3733
    ## 2663 0500000US48277                        Lamar County, Texas           49532
    ## 2664 0500000US48279                         Lamb County, Texas           13262
    ## 2665 0500000US48281                     Lampasas County, Texas           20640
    ## 2666 0500000US48283                     La Salle County, Texas            7409
    ## 2667 0500000US48285                       Lavaca County, Texas           19941
    ## 2668 0500000US48287                          Lee County, Texas           16952
    ## 2669 0500000US48289                         Leon County, Texas           17098
    ## 2670 0500000US48291                      Liberty County, Texas           81862
    ## 2671 0500000US48293                    Limestone County, Texas           23515
    ## 2672 0500000US48295                     Lipscomb County, Texas            3469
    ## 2673 0500000US48297                     Live Oak County, Texas           12123
    ## 2674 0500000US48299                        Llano County, Texas           20640
    ## 2675 0500000US48301                       Loving County, Texas             102
    ## 2676 0500000US48303                      Lubbock County, Texas          301454
    ## 2677 0500000US48305                         Lynn County, Texas            5808
    ## 2678 0500000US48307                    McCulloch County, Texas            8098
    ## 2679 0500000US48309                     McLennan County, Texas          248429
    ## 2680 0500000US48311                     McMullen County, Texas             662
    ## 2681 0500000US48313                      Madison County, Texas           14128
    ## 2682 0500000US48315                       Marion County, Texas           10083
    ## 2683 0500000US48317                       Martin County, Texas            5614
    ## 2684 0500000US48319                        Mason County, Texas            4161
    ## 2685 0500000US48321                    Matagorda County, Texas           36743
    ## 2686 0500000US48323                     Maverick County, Texas           57970
    ## 2687 0500000US48325                       Medina County, Texas           49334
    ## 2688 0500000US48327                       Menard County, Texas            2123
    ## 2689 0500000US48329                      Midland County, Texas          164194
    ## 2690 0500000US48331                        Milam County, Texas           24664
    ## 2691 0500000US48333                        Mills County, Texas            4902
    ## 2692 0500000US48335                     Mitchell County, Texas            8558
    ## 2693 0500000US48337                     Montague County, Texas           19409
    ## 2694 0500000US48339                   Montgomery County, Texas          554445
    ## 2695 0500000US48341                        Moore County, Texas           21801
    ## 2696 0500000US48343                       Morris County, Texas           12424
    ## 2697 0500000US48345                       Motley County, Texas            1156
    ## 2698 0500000US48347                  Nacogdoches County, Texas           65558
    ## 2699 0500000US48349                      Navarro County, Texas           48583
    ## 2700 0500000US48351                       Newton County, Texas           14057
    ## 2701 0500000US48353                        Nolan County, Texas           14966
    ## 2702 0500000US48355                       Nueces County, Texas          360486
    ## 2703 0500000US48357                    Ochiltree County, Texas           10348
    ## 2704 0500000US48359                       Oldham County, Texas            2090
    ## 2705 0500000US48361                       Orange County, Texas           84047
    ## 2706 0500000US48363                   Palo Pinto County, Texas           28317
    ## 2707 0500000US48365                       Panola County, Texas           23440
    ## 2708 0500000US48367                       Parker County, Texas          129802
    ## 2709 0500000US48369                       Parmer County, Texas            9852
    ## 2710 0500000US48371                        Pecos County, Texas           15797
    ## 2711 0500000US48373                         Polk County, Texas           47837
    ## 2712 0500000US48375                       Potter County, Texas          120899
    ## 2713 0500000US48377                     Presidio County, Texas            7123
    ## 2714 0500000US48379                        Rains County, Texas           11473
    ## 2715 0500000US48381                      Randall County, Texas          132475
    ## 2716 0500000US48383                       Reagan County, Texas            3752
    ## 2717 0500000US48385                         Real County, Texas            3389
    ## 2718 0500000US48387                    Red River County, Texas           12275
    ## 2719 0500000US48389                       Reeves County, Texas           15125
    ## 2720 0500000US48391                      Refugio County, Texas            7236
    ## 2721 0500000US48393                      Roberts County, Texas             885
    ## 2722 0500000US48395                    Robertson County, Texas           16890
    ## 2723 0500000US48397                     Rockwall County, Texas           93642
    ## 2724 0500000US48399                      Runnels County, Texas           10310
    ## 2725 0500000US48401                         Rusk County, Texas           53595
    ## 2726 0500000US48403                       Sabine County, Texas           10458
    ## 2727 0500000US48405                San Augustine County, Texas            8327
    ## 2728 0500000US48407                  San Jacinto County, Texas           27819
    ## 2729 0500000US48409                 San Patricio County, Texas           67046
    ## 2730 0500000US48411                     San Saba County, Texas            5962
    ## 2731 0500000US48413                   Schleicher County, Texas            3061
    ## 2732 0500000US48415                       Scurry County, Texas           17239
    ## 2733 0500000US48417                  Shackelford County, Texas            3311
    ## 2734 0500000US48419                       Shelby County, Texas           25478
    ## 2735 0500000US48421                      Sherman County, Texas            3058
    ## 2736 0500000US48423                        Smith County, Texas          225015
    ## 2737 0500000US48425                    Somervell County, Texas            8743
    ## 2738 0500000US48427                        Starr County, Texas           63894
    ## 2739 0500000US48429                     Stephens County, Texas            9372
    ## 2740 0500000US48431                     Sterling County, Texas            1141
    ## 2741 0500000US48433                    Stonewall County, Texas            1385
    ## 2742 0500000US48435                       Sutton County, Texas            3865
    ## 2743 0500000US48437                      Swisher County, Texas            7484
    ## 2744 0500000US48439                      Tarrant County, Texas         2019977
    ## 2745 0500000US48441                       Taylor County, Texas          136348
    ## 2746 0500000US48443                      Terrell County, Texas             862
    ## 2747 0500000US48445                        Terry County, Texas           12615
    ## 2748 0500000US48447                 Throckmorton County, Texas            1567
    ## 2749 0500000US48449                        Titus County, Texas           32730
    ## 2750 0500000US48451                    Tom Green County, Texas          117466
    ## 2751 0500000US48453                       Travis County, Texas         1203166
    ## 2752 0500000US48455                      Trinity County, Texas           14569
    ## 2753 0500000US48457                        Tyler County, Texas           21496
    ## 2754 0500000US48459                       Upshur County, Texas           40769
    ## 2755 0500000US48461                        Upton County, Texas            3634
    ## 2756 0500000US48463                       Uvalde County, Texas           27009
    ## 2757 0500000US48465                    Val Verde County, Texas           49027
    ## 2758 0500000US48467                    Van Zandt County, Texas           54368
    ## 2759 0500000US48469                     Victoria County, Texas           91970
    ## 2760 0500000US48471                       Walker County, Texas           71539
    ## 2761 0500000US48473                       Waller County, Texas           49987
    ## 2762 0500000US48475                         Ward County, Texas           11586
    ## 2763 0500000US48477                   Washington County, Texas           34796
    ## 2764 0500000US48479                         Webb County, Texas          272053
    ## 2765 0500000US48481                      Wharton County, Texas           41551
    ## 2766 0500000US48483                      Wheeler County, Texas            5482
    ## 2767 0500000US48485                      Wichita County, Texas          131818
    ## 2768 0500000US48487                    Wilbarger County, Texas           12906
    ## 2769 0500000US48489                      Willacy County, Texas           21754
    ## 2770 0500000US48491                   Williamson County, Texas          527057
    ## 2771 0500000US48493                       Wilson County, Texas           48198
    ## 2772 0500000US48495                      Winkler County, Texas            7802
    ## 2773 0500000US48497                         Wise County, Texas           64639
    ## 2774 0500000US48499                         Wood County, Texas           43815
    ## 2775 0500000US48501                       Yoakum County, Texas            8571
    ## 2776 0500000US48503                        Young County, Texas           18114
    ## 2777 0500000US48505                       Zapata County, Texas           14369
    ## 2778 0500000US48507                       Zavala County, Texas           12131
    ## 2779 0500000US49001                        Beaver County, Utah            6443
    ## 2780 0500000US49003                     Box Elder County, Utah           53001
    ## 2781 0500000US49005                         Cache County, Utah          122336
    ## 2782 0500000US49007                        Carbon County, Utah           20356
    ## 2783 0500000US49009                       Daggett County, Utah             612
    ## 2784 0500000US49011                         Davis County, Utah          340621
    ## 2785 0500000US49013                      Duchesne County, Utah           20219
    ## 2786 0500000US49015                         Emery County, Utah           10248
    ## 2787 0500000US49017                      Garfield County, Utah            5017
    ## 2788 0500000US49019                         Grand County, Utah            9616
    ## 2789 0500000US49021                          Iron County, Utah           49691
    ## 2790 0500000US49023                          Juab County, Utah           10948
    ## 2791 0500000US49025                          Kane County, Utah            7350
    ## 2792 0500000US49027                       Millard County, Utah           12733
    ## 2793 0500000US49029                        Morgan County, Utah           11391
    ## 2794 0500000US49031                         Piute County, Utah            1904
    ## 2795 0500000US49033                          Rich County, Utah            2350
    ## 2796 0500000US49035                     Salt Lake County, Utah         1120805
    ## 2797 0500000US49037                      San Juan County, Utah           15281
    ## 2798 0500000US49039                       Sanpete County, Utah           29366
    ## 2799 0500000US49041                        Sevier County, Utah           21118
    ## 2800 0500000US49043                        Summit County, Utah           40511
    ## 2801 0500000US49045                        Tooele County, Utah           65185
    ## 2802 0500000US49047                        Uintah County, Utah           36323
    ## 2803 0500000US49049                          Utah County, Utah          590440
    ## 2804 0500000US49051                       Wasatch County, Utah           30523
    ## 2805 0500000US49053                    Washington County, Utah          160537
    ## 2806 0500000US49055                         Wayne County, Utah            2694
    ## 2807 0500000US49057                         Weber County, Utah          247731
    ## 2808 0500000US50001                    Addison County, Vermont           36939
    ## 2809 0500000US50003                 Bennington County, Vermont           35920
    ## 2810 0500000US50005                  Caledonia County, Vermont           30425
    ## 2811 0500000US50007                 Chittenden County, Vermont          162052
    ## 2812 0500000US50009                      Essex County, Vermont            6208
    ## 2813 0500000US50011                   Franklin County, Vermont           49025
    ## 2814 0500000US50013                 Grand Isle County, Vermont            6965
    ## 2815 0500000US50015                   Lamoille County, Vermont           25268
    ## 2816 0500000US50017                     Orange County, Vermont           28937
    ## 2817 0500000US50019                    Orleans County, Vermont           26911
    ## 2818 0500000US50021                    Rutland County, Vermont           59273
    ## 2819 0500000US50023                 Washington County, Vermont           58477
    ## 2820 0500000US50025                    Windham County, Vermont           43150
    ## 2821 0500000US50027                    Windsor County, Vermont           55427
    ## 2822 0500000US51001                  Accomack County, Virginia           32742
    ## 2823 0500000US51003                 Albemarle County, Virginia          106355
    ## 2824 0500000US51005                 Alleghany County, Virginia           15286
    ## 2825 0500000US51007                    Amelia County, Virginia           12854
    ## 2826 0500000US51009                   Amherst County, Virginia           31882
    ## 2827 0500000US51011                Appomattox County, Virginia           15577
    ## 2828 0500000US51013                 Arlington County, Virginia          231803
    ## 2829 0500000US51015                   Augusta County, Virginia           74701
    ## 2830 0500000US51017                      Bath County, Virginia            4393
    ## 2831 0500000US51019                   Bedford County, Virginia           77908
    ## 2832 0500000US51021                     Bland County, Virginia            6447
    ## 2833 0500000US51023                 Botetourt County, Virginia           33222
    ## 2834 0500000US51025                 Brunswick County, Virginia           16665
    ## 2835 0500000US51027                  Buchanan County, Virginia           22138
    ## 2836 0500000US51029                Buckingham County, Virginia           17004
    ## 2837 0500000US51031                  Campbell County, Virginia           55170
    ## 2838 0500000US51033                  Caroline County, Virginia           30184
    ## 2839 0500000US51035                   Carroll County, Virginia           29738
    ## 2840 0500000US51036              Charles City County, Virginia            6995
    ## 2841 0500000US51037                 Charlotte County, Virginia           12095
    ## 2842 0500000US51041              Chesterfield County, Virginia          339447
    ## 2843 0500000US51043                    Clarke County, Virginia           14365
    ## 2844 0500000US51045                     Craig County, Virginia            5113
    ## 2845 0500000US51047                  Culpeper County, Virginia           50450
    ## 2846 0500000US51049                Cumberland County, Virginia            9786
    ## 2847 0500000US51051                 Dickenson County, Virginia           14960
    ## 2848 0500000US51053                 Dinwiddie County, Virginia           28308
    ## 2849 0500000US51057                     Essex County, Virginia           11036
    ## 2850 0500000US51059                   Fairfax County, Virginia         1143529
    ## 2851 0500000US51061                  Fauquier County, Virginia           69115
    ## 2852 0500000US51063                     Floyd County, Virginia           15666
    ## 2853 0500000US51065                  Fluvanna County, Virginia           26282
    ## 2854 0500000US51067                  Franklin County, Virginia           56233
    ## 2855 0500000US51069                 Frederick County, Virginia           85153
    ## 2856 0500000US51071                     Giles County, Virginia           16814
    ## 2857 0500000US51073                Gloucester County, Virginia           37161
    ## 2858 0500000US51075                 Goochland County, Virginia           22482
    ## 2859 0500000US51077                   Grayson County, Virginia           15811
    ## 2860 0500000US51079                    Greene County, Virginia           19410
    ## 2861 0500000US51081               Greensville County, Virginia           11659
    ## 2862 0500000US51083                   Halifax County, Virginia           34779
    ## 2863 0500000US51085                   Hanover County, Virginia          104449
    ## 2864 0500000US51087                   Henrico County, Virginia          325642
    ## 2865 0500000US51089                     Henry County, Virginia           51588
    ## 2866 0500000US51091                  Highland County, Virginia            2214
    ## 2867 0500000US51093             Isle of Wight County, Virginia           36372
    ## 2868 0500000US51095                James City County, Virginia           74153
    ## 2869 0500000US51097            King and Queen County, Virginia            7052
    ## 2870 0500000US51099               King George County, Virginia           25890
    ## 2871 0500000US51101              King William County, Virginia           16497
    ## 2872 0500000US51103                 Lancaster County, Virginia           10804
    ## 2873 0500000US51105                       Lee County, Virginia           24134
    ## 2874 0500000US51107                   Loudoun County, Virginia          385143
    ## 2875 0500000US51109                    Louisa County, Virginia           35380
    ## 2876 0500000US51111                 Lunenburg County, Virginia           12278
    ## 2877 0500000US51113                   Madison County, Virginia           13139
    ## 2878 0500000US51115                   Mathews County, Virginia            8796
    ## 2879 0500000US51117               Mecklenburg County, Virginia           30847
    ## 2880 0500000US51119                 Middlesex County, Virginia           10700
    ## 2881 0500000US51121                Montgomery County, Virginia           97997
    ## 2882 0500000US51125                    Nelson County, Virginia           14812
    ## 2883 0500000US51127                  New Kent County, Virginia           21103
    ## 2884 0500000US51131               Northampton County, Virginia           11957
    ## 2885 0500000US51133            Northumberland County, Virginia           12223
    ## 2886 0500000US51135                  Nottoway County, Virginia           15500
    ## 2887 0500000US51137                    Orange County, Virginia           35612
    ## 2888 0500000US51139                      Page County, Virginia           23749
    ## 2889 0500000US51141                   Patrick County, Virginia           17859
    ## 2890 0500000US51143              Pittsylvania County, Virginia           61676
    ## 2891 0500000US51145                  Powhatan County, Virginia           28574
    ## 2892 0500000US51147             Prince Edward County, Virginia           22956
    ## 2893 0500000US51149             Prince George County, Virginia           37894
    ## 2894 0500000US51153            Prince William County, Virginia          456749
    ## 2895 0500000US51155                   Pulaski County, Virginia           34234
    ## 2896 0500000US51157              Rappahannock County, Virginia            7332
    ## 2897 0500000US51159                  Richmond County, Virginia            8878
    ## 2898 0500000US51161                   Roanoke County, Virginia           93583
    ## 2899 0500000US51163                Rockbridge County, Virginia           22509
    ## 2900 0500000US51165                Rockingham County, Virginia           79444
    ## 2901 0500000US51167                   Russell County, Virginia           27408
    ## 2902 0500000US51169                     Scott County, Virginia           22009
    ## 2903 0500000US51171                Shenandoah County, Virginia           43045
    ## 2904 0500000US51173                     Smyth County, Virginia           31059
    ## 2905 0500000US51175               Southampton County, Virginia           17939
    ## 2906 0500000US51177              Spotsylvania County, Virginia          131412
    ## 2907 0500000US51179                  Stafford County, Virginia          144012
    ## 2908 0500000US51181                     Surry County, Virginia            6600
    ## 2909 0500000US51183                    Sussex County, Virginia           11486
    ## 2910 0500000US51185                  Tazewell County, Virginia           42080
    ## 2911 0500000US51187                    Warren County, Virginia           39449
    ## 2912 0500000US51191                Washington County, Virginia           54406
    ## 2913 0500000US51193              Westmoreland County, Virginia           17638
    ## 2914 0500000US51195                      Wise County, Virginia           39025
    ## 2915 0500000US51197                     Wythe County, Virginia           28940
    ## 2916 0500000US51199                      York County, Virginia           67587
    ## 2917 0500000US51510                  Alexandria city, Virginia          156505
    ## 2918 0500000US51520                     Bristol city, Virginia           16843
    ## 2919 0500000US51530                 Buena Vista city, Virginia            6399
    ## 2920 0500000US51540             Charlottesville city, Virginia           47042
    ## 2921 0500000US51550                  Chesapeake city, Virginia          237820
    ## 2922 0500000US51570            Colonial Heights city, Virginia           17593
    ## 2923 0500000US51580                   Covington city, Virginia            5582
    ## 2924 0500000US51590                    Danville city, Virginia           41512
    ## 2925 0500000US51595                     Emporia city, Virginia            5381
    ## 2926 0500000US51600                     Fairfax city, Virginia           23865
    ## 2927 0500000US51610                Falls Church city, Virginia           14067
    ## 2928 0500000US51620                    Franklin city, Virginia            8211
    ## 2929 0500000US51630              Fredericksburg city, Virginia           28469
    ## 2930 0500000US51640                       Galax city, Virginia            6638
    ## 2931 0500000US51650                     Hampton city, Virginia          135583
    ## 2932 0500000US51660                Harrisonburg city, Virginia           53391
    ## 2933 0500000US51670                    Hopewell city, Virginia           22408
    ## 2934 0500000US51678                   Lexington city, Virginia            7110
    ## 2935 0500000US51680                   Lynchburg city, Virginia           80131
    ## 2936 0500000US51683                    Manassas city, Virginia           41457
    ## 2937 0500000US51685               Manassas Park city, Virginia           16423
    ## 2938 0500000US51690                Martinsville city, Virginia           13101
    ## 2939 0500000US51700                Newport News city, Virginia          180145
    ## 2940 0500000US51710                     Norfolk city, Virginia          245592
    ## 2941 0500000US51720                      Norton city, Virginia            3990
    ## 2942 0500000US51730                  Petersburg city, Virginia           31827
    ## 2943 0500000US51735                    Poquoson city, Virginia           12039
    ## 2944 0500000US51740                  Portsmouth city, Virginia           95311
    ## 2945 0500000US51750                     Radford city, Virginia           17630
    ## 2946 0500000US51760                    Richmond city, Virginia          223787
    ## 2947 0500000US51770                     Roanoke city, Virginia           99621
    ## 2948 0500000US51775                       Salem city, Virginia           25519
    ## 2949 0500000US51790                    Staunton city, Virginia           24452
    ## 2950 0500000US51800                     Suffolk city, Virginia           89160
    ## 2951 0500000US51810              Virginia Beach city, Virginia          450135
    ## 2952 0500000US51820                  Waynesboro city, Virginia           21926
    ## 2953 0500000US51830                Williamsburg city, Virginia           14788
    ## 2954 0500000US51840                  Winchester city, Virginia           27789
    ## 2955 0500000US53001                   Adams County, Washington           19452
    ## 2956 0500000US53003                  Asotin County, Washington           22337
    ## 2957 0500000US53005                  Benton County, Washington          194168
    ## 2958 0500000US53007                  Chelan County, Washington           75757
    ## 2959 0500000US53009                 Clallam County, Washington           74487
    ## 2960 0500000US53011                   Clark County, Washington          465384
    ## 2961 0500000US53013                Columbia County, Washington            4001
    ## 2962 0500000US53015                 Cowlitz County, Washington          105112
    ## 2963 0500000US53017                 Douglas County, Washington           41371
    ## 2964 0500000US53019                   Ferry County, Washington            7576
    ## 2965 0500000US53021                Franklin County, Washington           90660
    ## 2966 0500000US53023                Garfield County, Washington            2224
    ## 2967 0500000US53025                   Grant County, Washington           94860
    ## 2968 0500000US53027            Grays Harbor County, Washington           71967
    ## 2969 0500000US53029                  Island County, Washington           81636
    ## 2970 0500000US53031               Jefferson County, Washington           30856
    ## 2971 0500000US53033                    King County, Washington         2163257
    ## 2972 0500000US53035                  Kitsap County, Washington          262475
    ## 2973 0500000US53037                Kittitas County, Washington           44825
    ## 2974 0500000US53039               Klickitat County, Washington           21396
    ## 2975 0500000US53041                   Lewis County, Washington           76947
    ## 2976 0500000US53043                 Lincoln County, Washington           10435
    ## 2977 0500000US53045                   Mason County, Washington           62627
    ## 2978 0500000US53047                Okanogan County, Washington           41638
    ## 2979 0500000US53049                 Pacific County, Washington           21281
    ## 2980 0500000US53051            Pend Oreille County, Washington           13219
    ## 2981 0500000US53053                  Pierce County, Washington          859840
    ## 2982 0500000US53055                San Juan County, Washington           16473
    ## 2983 0500000US53057                  Skagit County, Washington          123907
    ## 2984 0500000US53059                Skamania County, Washington           11620
    ## 2985 0500000US53061               Snohomish County, Washington          786620
    ## 2986 0500000US53063                 Spokane County, Washington          497875
    ## 2987 0500000US53065                 Stevens County, Washington           44214
    ## 2988 0500000US53067                Thurston County, Washington          274684
    ## 2989 0500000US53069               Wahkiakum County, Washington            4189
    ## 2990 0500000US53071             Walla Walla County, Washington           60236
    ## 2991 0500000US53073                 Whatcom County, Washington          216812
    ## 2992 0500000US53075                 Whitman County, Washington           48593
    ## 2993 0500000US53077                  Yakima County, Washington          249325
    ## 2994 0500000US54001              Barbour County, West Virginia           16730
    ## 2995 0500000US54003             Berkeley County, West Virginia          113495
    ## 2996 0500000US54005                Boone County, West Virginia           22817
    ## 2997 0500000US54007              Braxton County, West Virginia           14282
    ## 2998 0500000US54009               Brooke County, West Virginia           22772
    ## 2999 0500000US54011               Cabell County, West Virginia           95318
    ## 3000 0500000US54013              Calhoun County, West Virginia            7396
    ## 3001 0500000US54015                 Clay County, West Virginia            8785
    ## 3002 0500000US54017            Doddridge County, West Virginia            8536
    ## 3003 0500000US54019              Fayette County, West Virginia           44126
    ## 3004 0500000US54021               Gilmer County, West Virginia            8205
    ## 3005 0500000US54023                Grant County, West Virginia           11641
    ## 3006 0500000US54025           Greenbrier County, West Virginia           35347
    ## 3007 0500000US54027            Hampshire County, West Virginia           23363
    ## 3008 0500000US54029              Hancock County, West Virginia           29680
    ## 3009 0500000US54031                Hardy County, West Virginia           13842
    ## 3010 0500000US54033             Harrison County, West Virginia           68209
    ## 3011 0500000US54035              Jackson County, West Virginia           29018
    ## 3012 0500000US54037            Jefferson County, West Virginia           56179
    ## 3013 0500000US54039              Kanawha County, West Virginia          185710
    ## 3014 0500000US54041                Lewis County, West Virginia           16276
    ## 3015 0500000US54043              Lincoln County, West Virginia           21078
    ## 3016 0500000US54045                Logan County, West Virginia           33801
    ## 3017 0500000US54047             McDowell County, West Virginia           19217
    ## 3018 0500000US54049               Marion County, West Virginia           56497
    ## 3019 0500000US54051             Marshall County, West Virginia           31645
    ## 3020 0500000US54053                Mason County, West Virginia           26939
    ## 3021 0500000US54055               Mercer County, West Virginia           60486
    ## 3022 0500000US54057              Mineral County, West Virginia           27278
    ## 3023 0500000US54059                Mingo County, West Virginia           24741
    ## 3024 0500000US54061           Monongalia County, West Virginia          105252
    ## 3025 0500000US54063               Monroe County, West Virginia           13467
    ## 3026 0500000US54065               Morgan County, West Virginia           17624
    ## 3027 0500000US54067             Nicholas County, West Virginia           25324
    ## 3028 0500000US54069                 Ohio County, West Virginia           42547
    ## 3029 0500000US54071            Pendleton County, West Virginia            7056
    ## 3030 0500000US54073            Pleasants County, West Virginia            7507
    ## 3031 0500000US54075           Pocahontas County, West Virginia            8531
    ## 3032 0500000US54077              Preston County, West Virginia           33837
    ## 3033 0500000US54079               Putnam County, West Virginia           56652
    ## 3034 0500000US54081              Raleigh County, West Virginia           76232
    ## 3035 0500000US54083             Randolph County, West Virginia           29065
    ## 3036 0500000US54085              Ritchie County, West Virginia            9932
    ## 3037 0500000US54087                Roane County, West Virginia           14205
    ## 3038 0500000US54089              Summers County, West Virginia           13018
    ## 3039 0500000US54091               Taylor County, West Virginia           16951
    ## 3040 0500000US54093               Tucker County, West Virginia            7027
    ## 3041 0500000US54095                Tyler County, West Virginia            8909
    ## 3042 0500000US54097               Upshur County, West Virginia           24605
    ## 3043 0500000US54099                Wayne County, West Virginia           40708
    ## 3044 0500000US54101              Webster County, West Virginia            8518
    ## 3045 0500000US54103               Wetzel County, West Virginia           15614
    ## 3046 0500000US54105                 Wirt County, West Virginia            5797
    ## 3047 0500000US54107                 Wood County, West Virginia           85556
    ## 3048 0500000US54109              Wyoming County, West Virginia           21711
    ## 3049 0500000US55001                    Adams County, Wisconsin           20073
    ## 3050 0500000US55003                  Ashland County, Wisconsin           15712
    ## 3051 0500000US55005                   Barron County, Wisconsin           45252
    ## 3052 0500000US55007                 Bayfield County, Wisconsin           14992
    ## 3053 0500000US55009                    Brown County, Wisconsin          259786
    ## 3054 0500000US55011                  Buffalo County, Wisconsin           13167
    ## 3055 0500000US55013                  Burnett County, Wisconsin           15258
    ## 3056 0500000US55015                  Calumet County, Wisconsin           49807
    ## 3057 0500000US55017                 Chippewa County, Wisconsin           63635
    ## 3058 0500000US55019                    Clark County, Wisconsin           34491
    ## 3059 0500000US55021                 Columbia County, Wisconsin           56954
    ## 3060 0500000US55023                 Crawford County, Wisconsin           16288
    ## 3061 0500000US55025                     Dane County, Wisconsin          529843
    ## 3062 0500000US55027                    Dodge County, Wisconsin           87776
    ## 3063 0500000US55029                     Door County, Wisconsin           27439
    ## 3064 0500000US55031                  Douglas County, Wisconsin           43402
    ## 3065 0500000US55033                     Dunn County, Wisconsin           44498
    ## 3066 0500000US55035               Eau Claire County, Wisconsin          102991
    ## 3067 0500000US55037                 Florence County, Wisconsin            4337
    ## 3068 0500000US55039              Fond du Lac County, Wisconsin          102315
    ## 3069 0500000US55041                   Forest County, Wisconsin            9018
    ## 3070 0500000US55043                    Grant County, Wisconsin           51828
    ## 3071 0500000US55045                    Green County, Wisconsin           36864
    ## 3072 0500000US55047               Green Lake County, Wisconsin           18757
    ## 3073 0500000US55049                     Iowa County, Wisconsin           23620
    ## 3074 0500000US55051                     Iron County, Wisconsin            5715
    ## 3075 0500000US55053                  Jackson County, Wisconsin           20506
    ## 3076 0500000US55055                Jefferson County, Wisconsin           84652
    ## 3077 0500000US55057                   Juneau County, Wisconsin           26419
    ## 3078 0500000US55059                  Kenosha County, Wisconsin          168330
    ## 3079 0500000US55061                 Kewaunee County, Wisconsin           20360
    ## 3080 0500000US55063                La Crosse County, Wisconsin          117850
    ## 3081 0500000US55065                Lafayette County, Wisconsin           16735
    ## 3082 0500000US55067                 Langlade County, Wisconsin           19164
    ## 3083 0500000US55069                  Lincoln County, Wisconsin           27848
    ## 3084 0500000US55071                Manitowoc County, Wisconsin           79407
    ## 3085 0500000US55073                 Marathon County, Wisconsin          135264
    ## 3086 0500000US55075                Marinette County, Wisconsin           40537
    ## 3087 0500000US55077                Marquette County, Wisconsin           15207
    ## 3088 0500000US55078                Menominee County, Wisconsin            4579
    ## 3089 0500000US55079                Milwaukee County, Wisconsin          954209
    ## 3090 0500000US55081                   Monroe County, Wisconsin           45502
    ## 3091 0500000US55083                   Oconto County, Wisconsin           37556
    ## 3092 0500000US55085                   Oneida County, Wisconsin           35345
    ## 3093 0500000US55087                Outagamie County, Wisconsin          184754
    ## 3094 0500000US55089                  Ozaukee County, Wisconsin           88284
    ## 3095 0500000US55091                    Pepin County, Wisconsin            7262
    ## 3096 0500000US55093                   Pierce County, Wisconsin           41603
    ## 3097 0500000US55095                     Polk County, Wisconsin           43349
    ## 3098 0500000US55097                  Portage County, Wisconsin           70599
    ## 3099 0500000US55099                    Price County, Wisconsin           13490
    ## 3100 0500000US55101                   Racine County, Wisconsin          195398
    ## 3101 0500000US55103                 Richland County, Wisconsin           17539
    ## 3102 0500000US55105                     Rock County, Wisconsin          161769
    ## 3103 0500000US55107                     Rusk County, Wisconsin           14183
    ## 3104 0500000US55109                St. Croix County, Wisconsin           87917
    ## 3105 0500000US55111                     Sauk County, Wisconsin           63596
    ## 3106 0500000US55113                   Sawyer County, Wisconsin           16370
    ## 3107 0500000US55115                  Shawano County, Wisconsin           41009
    ## 3108 0500000US55117                Sheboygan County, Wisconsin          115205
    ## 3109 0500000US55119                   Taylor County, Wisconsin           20356
    ## 3110 0500000US55121              Trempealeau County, Wisconsin           29438
    ## 3111 0500000US55123                   Vernon County, Wisconsin           30516
    ## 3112 0500000US55125                    Vilas County, Wisconsin           21593
    ## 3113 0500000US55127                 Walworth County, Wisconsin          103013
    ## 3114 0500000US55129                 Washburn County, Wisconsin           15689
    ## 3115 0500000US55131               Washington County, Wisconsin          134535
    ## 3116 0500000US55133                 Waukesha County, Wisconsin          398879
    ## 3117 0500000US55135                  Waupaca County, Wisconsin           51444
    ## 3118 0500000US55137                 Waushara County, Wisconsin           24116
    ## 3119 0500000US55139                Winnebago County, Wisconsin          169926
    ## 3120 0500000US55141                     Wood County, Wisconsin           73274
    ## 3121 0500000US56001                     Albany County, Wyoming           38102
    ## 3122 0500000US56003                   Big Horn County, Wyoming           11901
    ## 3123 0500000US56005                   Campbell County, Wyoming           47708
    ## 3124 0500000US56007                     Carbon County, Wyoming           15477
    ## 3125 0500000US56009                   Converse County, Wyoming           13997
    ## 3126 0500000US56011                      Crook County, Wyoming            7410
    ## 3127 0500000US56013                    Fremont County, Wyoming           40076
    ## 3128 0500000US56015                     Goshen County, Wyoming           13438
    ## 3129 0500000US56017                Hot Springs County, Wyoming            4680
    ## 3130 0500000US56019                    Johnson County, Wyoming            8515
    ## 3131 0500000US56021                    Laramie County, Wyoming           97692
    ## 3132 0500000US56023                    Lincoln County, Wyoming           19011
    ## 3133 0500000US56025                    Natrona County, Wyoming           80610
    ## 3134 0500000US56027                   Niobrara County, Wyoming            2448
    ## 3135 0500000US56029                       Park County, Wyoming           29121
    ## 3136 0500000US56031                     Platte County, Wyoming            8673
    ## 3137 0500000US56033                   Sheridan County, Wyoming           30012
    ## 3138 0500000US56035                   Sublette County, Wyoming            9951
    ## 3139 0500000US56037                 Sweetwater County, Wyoming           44117
    ## 3140 0500000US56039                      Teton County, Wyoming           23059
    ## 3141 0500000US56041                      Uinta County, Wyoming           20609
    ## 3142 0500000US56043                   Washakie County, Wyoming            8129
    ## 3143 0500000US56045                     Weston County, Wyoming            7100
    ## 3144 0500000US72001            Adjuntas Municipio, Puerto Rico           18181
    ## 3145 0500000US72003              Aguada Municipio, Puerto Rico           38643
    ## 3146 0500000US72005           Aguadilla Municipio, Puerto Rico           54166
    ## 3147 0500000US72007        Aguas Buenas Municipio, Puerto Rico           26275
    ## 3148 0500000US72009            Aibonito Municipio, Puerto Rico           23457
    ## 3149 0500000US72011              Añasco Municipio, Puerto Rico           27368
    ## 3150 0500000US72013             Arecibo Municipio, Puerto Rico           87242
    ## 3151 0500000US72015              Arroyo Municipio, Puerto Rico           18111
    ## 3152 0500000US72017         Barceloneta Municipio, Puerto Rico           24299
    ## 3153 0500000US72019        Barranquitas Municipio, Puerto Rico           28755
    ## 3154 0500000US72021             Bayamón Municipio, Puerto Rico          182955
    ## 3155 0500000US72023           Cabo Rojo Municipio, Puerto Rico           49005
    ## 3156 0500000US72025              Caguas Municipio, Puerto Rico          131363
    ## 3157 0500000US72027               Camuy Municipio, Puerto Rico           32222
    ## 3158 0500000US72029           Canóvanas Municipio, Puerto Rico           46108
    ## 3159 0500000US72031            Carolina Municipio, Puerto Rico          157453
    ## 3160 0500000US72033              Cataño Municipio, Puerto Rico           24888
    ## 3161 0500000US72035               Cayey Municipio, Puerto Rico           44530
    ## 3162 0500000US72037               Ceiba Municipio, Puerto Rico           11853
    ## 3163 0500000US72039              Ciales Municipio, Puerto Rico           16912
    ## 3164 0500000US72041               Cidra Municipio, Puerto Rico           40343
    ## 3165 0500000US72043               Coamo Municipio, Puerto Rico           39265
    ## 3166 0500000US72045             Comerío Municipio, Puerto Rico           19539
    ## 3167 0500000US72047             Corozal Municipio, Puerto Rico           34165
    ## 3168 0500000US72049             Culebra Municipio, Puerto Rico            1314
    ## 3169 0500000US72051              Dorado Municipio, Puerto Rico           37208
    ## 3170 0500000US72053             Fajardo Municipio, Puerto Rico           32001
    ## 3171 0500000US72054             Florida Municipio, Puerto Rico           11910
    ## 3172 0500000US72055             Guánica Municipio, Puerto Rico           16783
    ## 3173 0500000US72057             Guayama Municipio, Puerto Rico           41706
    ## 3174 0500000US72059          Guayanilla Municipio, Puerto Rico           19008
    ## 3175 0500000US72061            Guaynabo Municipio, Puerto Rico           88663
    ## 3176 0500000US72063              Gurabo Municipio, Puerto Rico           46894
    ## 3177 0500000US72065             Hatillo Municipio, Puerto Rico           40390
    ## 3178 0500000US72067         Hormigueros Municipio, Puerto Rico           16180
    ## 3179 0500000US72069             Humacao Municipio, Puerto Rico           53466
    ## 3180 0500000US72071             Isabela Municipio, Puerto Rico           42420
    ## 3181 0500000US72073              Jayuya Municipio, Puerto Rico           14906
    ## 3182 0500000US72075          Juana Díaz Municipio, Puerto Rico           46960
    ## 3183 0500000US72077              Juncos Municipio, Puerto Rico           39128
    ## 3184 0500000US72079               Lajas Municipio, Puerto Rico           23315
    ## 3185 0500000US72081               Lares Municipio, Puerto Rico           26451
    ## 3186 0500000US72083          Las Marías Municipio, Puerto Rico            8599
    ## 3187 0500000US72085         Las Piedras Municipio, Puerto Rico           37768
    ## 3188 0500000US72087               Loíza Municipio, Puerto Rico           26463
    ## 3189 0500000US72089            Luquillo Municipio, Puerto Rico           18547
    ## 3190 0500000US72091              Manatí Municipio, Puerto Rico           39692
    ## 3191 0500000US72093             Maricao Municipio, Puerto Rico            6202
    ## 3192 0500000US72095             Maunabo Municipio, Puerto Rico           11023
    ## 3193 0500000US72097            Mayagüez Municipio, Puerto Rico           77255
    ## 3194 0500000US72099                Moca Municipio, Puerto Rico           36872
    ## 3195 0500000US72101             Morovis Municipio, Puerto Rico           31320
    ## 3196 0500000US72103             Naguabo Municipio, Puerto Rico           26266
    ## 3197 0500000US72105           Naranjito Municipio, Puerto Rico           28557
    ## 3198 0500000US72107            Orocovis Municipio, Puerto Rico           21407
    ## 3199 0500000US72109            Patillas Municipio, Puerto Rico           17334
    ## 3200 0500000US72111            Peñuelas Municipio, Puerto Rico           20984
    ## 3201 0500000US72113               Ponce Municipio, Puerto Rico          143926
    ## 3202 0500000US72115        Quebradillas Municipio, Puerto Rico           24036
    ## 3203 0500000US72117              Rincón Municipio, Puerto Rico           14269
    ## 3204 0500000US72119          Río Grande Municipio, Puerto Rico           50550
    ## 3205 0500000US72121       Sabana Grande Municipio, Puerto Rico           23054
    ## 3206 0500000US72123             Salinas Municipio, Puerto Rico           28633
    ## 3207 0500000US72125          San Germán Municipio, Puerto Rico           32114
    ## 3208 0500000US72127            San Juan Municipio, Puerto Rico          344606
    ## 3209 0500000US72129         San Lorenzo Municipio, Puerto Rico           37873
    ## 3210 0500000US72131       San Sebastián Municipio, Puerto Rico           37964
    ## 3211 0500000US72133        Santa Isabel Municipio, Puerto Rico           22066
    ## 3212 0500000US72135            Toa Alta Municipio, Puerto Rico           73405
    ## 3213 0500000US72137            Toa Baja Municipio, Puerto Rico           79726
    ## 3214 0500000US72139       Trujillo Alto Municipio, Puerto Rico           67780
    ## 3215 0500000US72141              Utuado Municipio, Puerto Rico           29402
    ## 3216 0500000US72143           Vega Alta Municipio, Puerto Rico           37724
    ## 3217 0500000US72145           Vega Baja Municipio, Puerto Rico           53371
    ## 3218 0500000US72147             Vieques Municipio, Puerto Rico            8771
    ## 3219 0500000US72149            Villalba Municipio, Puerto Rico           22993
    ## 3220 0500000US72151             Yabucoa Municipio, Puerto Rico           34149
    ## 3221 0500000US72153               Yauco Municipio, Puerto Rico           36439
    ##                 B01003_001M  X
    ## 1    Margin of Error!!Total NA
    ## 2                     ***** NA
    ## 3                     ***** NA
    ## 4                     ***** NA
    ## 5                     ***** NA
    ## 6                     ***** NA
    ## 7                     ***** NA
    ## 8                     ***** NA
    ## 9                     ***** NA
    ## 10                    ***** NA
    ## 11                    ***** NA
    ## 12                    ***** NA
    ## 13                    ***** NA
    ## 14                    ***** NA
    ## 15                    ***** NA
    ## 16                    ***** NA
    ## 17                    ***** NA
    ## 18                    ***** NA
    ## 19                    ***** NA
    ## 20                    ***** NA
    ## 21                    ***** NA
    ## 22                    ***** NA
    ## 23                    ***** NA
    ## 24                    ***** NA
    ## 25                    ***** NA
    ## 26                    ***** NA
    ## 27                    ***** NA
    ## 28                    ***** NA
    ## 29                    ***** NA
    ## 30                    ***** NA
    ## 31                    ***** NA
    ## 32                    ***** NA
    ## 33                    ***** NA
    ## 34                    ***** NA
    ## 35                    ***** NA
    ## 36                    ***** NA
    ## 37                    ***** NA
    ## 38                    ***** NA
    ## 39                    ***** NA
    ## 40                    ***** NA
    ## 41                    ***** NA
    ## 42                    ***** NA
    ## 43                    ***** NA
    ## 44                    ***** NA
    ## 45                    ***** NA
    ## 46                    ***** NA
    ## 47                    ***** NA
    ## 48                    ***** NA
    ## 49                    ***** NA
    ## 50                    ***** NA
    ## 51                    ***** NA
    ## 52                    ***** NA
    ## 53                    ***** NA
    ## 54                    ***** NA
    ## 55                    ***** NA
    ## 56                    ***** NA
    ## 57                    ***** NA
    ## 58                    ***** NA
    ## 59                    ***** NA
    ## 60                    ***** NA
    ## 61                    ***** NA
    ## 62                    ***** NA
    ## 63                    ***** NA
    ## 64                    ***** NA
    ## 65                    ***** NA
    ## 66                    ***** NA
    ## 67                    ***** NA
    ## 68                    ***** NA
    ## 69                    ***** NA
    ## 70                    ***** NA
    ## 71                    ***** NA
    ## 72                    ***** NA
    ## 73                       89 NA
    ## 74                      380 NA
    ## 75                    ***** NA
    ## 76                    ***** NA
    ## 77                    ***** NA
    ## 78                    ***** NA
    ## 79                    ***** NA
    ## 80                    ***** NA
    ## 81                    ***** NA
    ## 82                    ***** NA
    ## 83                    ***** NA
    ## 84                      380 NA
    ## 85                    ***** NA
    ## 86                    ***** NA
    ## 87                    ***** NA
    ## 88                    ***** NA
    ## 89                    ***** NA
    ## 90                    ***** NA
    ## 91                    ***** NA
    ## 92                      123 NA
    ## 93                    ***** NA
    ## 94                    ***** NA
    ## 95                    ***** NA
    ## 96                       79 NA
    ## 97                    ***** NA
    ## 98                    ***** NA
    ## 99                    ***** NA
    ## 100                   ***** NA
    ## 101                   ***** NA
    ## 102                   ***** NA
    ## 103                   ***** NA
    ## 104                   ***** NA
    ## 105                   ***** NA
    ## 106                   ***** NA
    ## 107                   ***** NA
    ## 108                   ***** NA
    ## 109                   ***** NA
    ## 110                   ***** NA
    ## 111                   ***** NA
    ## 112                   ***** NA
    ## 113                   ***** NA
    ## 114                   ***** NA
    ## 115                   ***** NA
    ## 116                   ***** NA
    ## 117                   ***** NA
    ## 118                   ***** NA
    ## 119                   ***** NA
    ## 120                   ***** NA
    ## 121                   ***** NA
    ## 122                   ***** NA
    ## 123                   ***** NA
    ## 124                   ***** NA
    ## 125                   ***** NA
    ## 126                   ***** NA
    ## 127                   ***** NA
    ## 128                   ***** NA
    ## 129                   ***** NA
    ## 130                   ***** NA
    ## 131                   ***** NA
    ## 132                   ***** NA
    ## 133                   ***** NA
    ## 134                   ***** NA
    ## 135                   ***** NA
    ## 136                   ***** NA
    ## 137                   ***** NA
    ## 138                   ***** NA
    ## 139                   ***** NA
    ## 140                   ***** NA
    ## 141                   ***** NA
    ## 142                   ***** NA
    ## 143                   ***** NA
    ## 144                   ***** NA
    ## 145                   ***** NA
    ## 146                   ***** NA
    ## 147                   ***** NA
    ## 148                   ***** NA
    ## 149                   ***** NA
    ## 150                   ***** NA
    ## 151                   ***** NA
    ## 152                   ***** NA
    ## 153                   ***** NA
    ## 154                   ***** NA
    ## 155                   ***** NA
    ## 156                   ***** NA
    ## 157                   ***** NA
    ## 158                   ***** NA
    ## 159                   ***** NA
    ## 160                   ***** NA
    ## 161                   ***** NA
    ## 162                   ***** NA
    ## 163                   ***** NA
    ## 164                   ***** NA
    ## 165                   ***** NA
    ## 166                   ***** NA
    ## 167                   ***** NA
    ## 168                   ***** NA
    ## 169                   ***** NA
    ## 170                   ***** NA
    ## 171                   ***** NA
    ## 172                   ***** NA
    ## 173                   ***** NA
    ## 174                   ***** NA
    ## 175                   ***** NA
    ## 176                   ***** NA
    ## 177                   ***** NA
    ## 178                   ***** NA
    ## 179                   ***** NA
    ## 180                   ***** NA
    ## 181                   ***** NA
    ## 182                   ***** NA
    ## 183                   ***** NA
    ## 184                   ***** NA
    ## 185                   ***** NA
    ## 186                   ***** NA
    ## 187                   ***** NA
    ## 188                   ***** NA
    ## 189                     161 NA
    ## 190                   ***** NA
    ## 191                   ***** NA
    ## 192                   ***** NA
    ## 193                   ***** NA
    ## 194                   ***** NA
    ## 195                   ***** NA
    ## 196                   ***** NA
    ## 197                   ***** NA
    ## 198                   ***** NA
    ## 199                   ***** NA
    ## 200                   ***** NA
    ## 201                   ***** NA
    ## 202                   ***** NA
    ## 203                   ***** NA
    ## 204                   ***** NA
    ## 205                   ***** NA
    ## 206                   ***** NA
    ## 207                   ***** NA
    ## 208                   ***** NA
    ## 209                   ***** NA
    ## 210                   ***** NA
    ## 211                   ***** NA
    ## 212                   ***** NA
    ## 213                   ***** NA
    ## 214                   ***** NA
    ## 215                   ***** NA
    ## 216                   ***** NA
    ## 217                   ***** NA
    ## 218                   ***** NA
    ## 219                   ***** NA
    ## 220                   ***** NA
    ## 221                   ***** NA
    ## 222                   ***** NA
    ## 223                   ***** NA
    ## 224                   ***** NA
    ## 225                   ***** NA
    ## 226                   ***** NA
    ## 227                   ***** NA
    ## 228                   ***** NA
    ## 229                   ***** NA
    ## 230                   ***** NA
    ## 231                   ***** NA
    ## 232                   ***** NA
    ## 233                     161 NA
    ## 234                   ***** NA
    ## 235                   ***** NA
    ## 236                   ***** NA
    ## 237                   ***** NA
    ## 238                   ***** NA
    ## 239                   ***** NA
    ## 240                   ***** NA
    ## 241                   ***** NA
    ## 242                   ***** NA
    ## 243                   ***** NA
    ## 244                   ***** NA
    ## 245                   ***** NA
    ## 246                   ***** NA
    ## 247                   ***** NA
    ## 248                   ***** NA
    ## 249                   ***** NA
    ## 250                   ***** NA
    ## 251                   ***** NA
    ## 252                   ***** NA
    ## 253                   ***** NA
    ## 254                   ***** NA
    ## 255                     170 NA
    ## 256                   ***** NA
    ## 257                   ***** NA
    ## 258                   ***** NA
    ## 259                   ***** NA
    ## 260                   ***** NA
    ## 261                   ***** NA
    ## 262                   ***** NA
    ## 263                     170 NA
    ## 264                   ***** NA
    ## 265                   ***** NA
    ## 266                   ***** NA
    ## 267                   ***** NA
    ## 268                   ***** NA
    ## 269                   ***** NA
    ## 270                   ***** NA
    ## 271                   ***** NA
    ## 272                   ***** NA
    ## 273                     108 NA
    ## 274                   ***** NA
    ## 275                     118 NA
    ## 276                   ***** NA
    ## 277                     118 NA
    ## 278                   ***** NA
    ## 279                   ***** NA
    ## 280                   ***** NA
    ## 281                   ***** NA
    ## 282                   ***** NA
    ## 283                   ***** NA
    ## 284                   ***** NA
    ## 285                   ***** NA
    ## 286                     113 NA
    ## 287                   ***** NA
    ## 288                   ***** NA
    ## 289                   ***** NA
    ## 290                   ***** NA
    ## 291                   ***** NA
    ## 292                   ***** NA
    ## 293                   ***** NA
    ## 294                   ***** NA
    ## 295                   ***** NA
    ## 296                   ***** NA
    ## 297                   ***** NA
    ## 298                   ***** NA
    ## 299                   ***** NA
    ## 300                   ***** NA
    ## 301                   ***** NA
    ## 302                      97 NA
    ## 303                   ***** NA
    ## 304                   ***** NA
    ## 305                   ***** NA
    ## 306                   ***** NA
    ## 307                   ***** NA
    ## 308                   ***** NA
    ## 309                   ***** NA
    ## 310                   ***** NA
    ## 311                   ***** NA
    ## 312                   ***** NA
    ## 313                   ***** NA
    ## 314                   ***** NA
    ## 315                   ***** NA
    ## 316                   ***** NA
    ## 317                   ***** NA
    ## 318                   ***** NA
    ## 319                   ***** NA
    ## 320                   ***** NA
    ## 321                   ***** NA
    ## 322                   ***** NA
    ## 323                   ***** NA
    ## 324                   ***** NA
    ## 325                   ***** NA
    ## 326                   ***** NA
    ## 327                   ***** NA
    ## 328                   ***** NA
    ## 329                   ***** NA
    ## 330                   ***** NA
    ## 331                   ***** NA
    ## 332                   ***** NA
    ## 333                   ***** NA
    ## 334                   ***** NA
    ## 335                   ***** NA
    ## 336                   ***** NA
    ## 337                   ***** NA
    ## 338                   ***** NA
    ## 339                   ***** NA
    ## 340                   ***** NA
    ## 341                   ***** NA
    ## 342                   ***** NA
    ## 343                   ***** NA
    ## 344                   ***** NA
    ## 345                   ***** NA
    ## 346                   ***** NA
    ## 347                   ***** NA
    ## 348                   ***** NA
    ## 349                   ***** NA
    ## 350                   ***** NA
    ## 351                   ***** NA
    ## 352                   ***** NA
    ## 353                   ***** NA
    ## 354                   ***** NA
    ## 355                   ***** NA
    ## 356                   ***** NA
    ## 357                   ***** NA
    ## 358                   ***** NA
    ## 359                   ***** NA
    ## 360                   ***** NA
    ## 361                   ***** NA
    ## 362                   ***** NA
    ## 363                   ***** NA
    ## 364                   ***** NA
    ## 365                   ***** NA
    ## 366                   ***** NA
    ## 367                   ***** NA
    ## 368                   ***** NA
    ## 369                   ***** NA
    ## 370                   ***** NA
    ## 371                   ***** NA
    ## 372                   ***** NA
    ## 373                   ***** NA
    ## 374                   ***** NA
    ## 375                   ***** NA
    ## 376                   ***** NA
    ## 377                   ***** NA
    ## 378                   ***** NA
    ## 379                   ***** NA
    ## 380                   ***** NA
    ## 381                   ***** NA
    ## 382                   ***** NA
    ## 383                   ***** NA
    ## 384                   ***** NA
    ## 385                   ***** NA
    ## 386                   ***** NA
    ## 387                   ***** NA
    ## 388                   ***** NA
    ## 389                   ***** NA
    ## 390                   ***** NA
    ## 391                   ***** NA
    ## 392                   ***** NA
    ## 393                   ***** NA
    ## 394                   ***** NA
    ## 395                   ***** NA
    ## 396                   ***** NA
    ## 397                   ***** NA
    ## 398                   ***** NA
    ## 399                   ***** NA
    ## 400                   ***** NA
    ## 401                   ***** NA
    ## 402                   ***** NA
    ## 403                   ***** NA
    ## 404                   ***** NA
    ## 405                   ***** NA
    ## 406                   ***** NA
    ## 407                   ***** NA
    ## 408                   ***** NA
    ## 409                   ***** NA
    ## 410                   ***** NA
    ## 411                   ***** NA
    ## 412                   ***** NA
    ## 413                   ***** NA
    ## 414                   ***** NA
    ## 415                   ***** NA
    ## 416                   ***** NA
    ## 417                   ***** NA
    ## 418                   ***** NA
    ## 419                   ***** NA
    ## 420                   ***** NA
    ## 421                   ***** NA
    ## 422                   ***** NA
    ## 423                   ***** NA
    ## 424                   ***** NA
    ## 425                   ***** NA
    ## 426                   ***** NA
    ## 427                   ***** NA
    ## 428                   ***** NA
    ## 429                   ***** NA
    ## 430                   ***** NA
    ## 431                   ***** NA
    ## 432                   ***** NA
    ## 433                   ***** NA
    ## 434                   ***** NA
    ## 435                   ***** NA
    ## 436                   ***** NA
    ## 437                   ***** NA
    ## 438                   ***** NA
    ## 439                   ***** NA
    ## 440                   ***** NA
    ## 441                   ***** NA
    ## 442                   ***** NA
    ## 443                   ***** NA
    ## 444                   ***** NA
    ## 445                   ***** NA
    ## 446                   ***** NA
    ## 447                   ***** NA
    ## 448                   ***** NA
    ## 449                   ***** NA
    ## 450                   ***** NA
    ## 451                   ***** NA
    ## 452                   ***** NA
    ## 453                   ***** NA
    ## 454                   ***** NA
    ## 455                   ***** NA
    ## 456                   ***** NA
    ## 457                   ***** NA
    ## 458                   ***** NA
    ## 459                   ***** NA
    ## 460                   ***** NA
    ## 461                   ***** NA
    ## 462                   ***** NA
    ## 463                   ***** NA
    ## 464                   ***** NA
    ## 465                   ***** NA
    ## 466                   ***** NA
    ## 467                   ***** NA
    ## 468                   ***** NA
    ## 469                   ***** NA
    ## 470                   ***** NA
    ## 471                   ***** NA
    ## 472                   ***** NA
    ## 473                   ***** NA
    ## 474                   ***** NA
    ## 475                   ***** NA
    ## 476                   ***** NA
    ## 477                   ***** NA
    ## 478                   ***** NA
    ## 479                   ***** NA
    ## 480                   ***** NA
    ## 481                   ***** NA
    ## 482                   ***** NA
    ## 483                   ***** NA
    ## 484                   ***** NA
    ## 485                   ***** NA
    ## 486                   ***** NA
    ## 487                   ***** NA
    ## 488                   ***** NA
    ## 489                   ***** NA
    ## 490                   ***** NA
    ## 491                   ***** NA
    ## 492                   ***** NA
    ## 493                   ***** NA
    ## 494                   ***** NA
    ## 495                   ***** NA
    ## 496                   ***** NA
    ## 497                   ***** NA
    ## 498                   ***** NA
    ## 499                   ***** NA
    ## 500                   ***** NA
    ## 501                   ***** NA
    ## 502                   ***** NA
    ## 503                   ***** NA
    ## 504                   ***** NA
    ## 505                   ***** NA
    ## 506                     152 NA
    ## 507                   ***** NA
    ## 508                   ***** NA
    ## 509                   ***** NA
    ## 510                   ***** NA
    ## 511                   ***** NA
    ## 512                   ***** NA
    ## 513                   ***** NA
    ## 514                   ***** NA
    ## 515                   ***** NA
    ## 516                   ***** NA
    ## 517                   ***** NA
    ## 518                   ***** NA
    ## 519                     152 NA
    ## 520                   ***** NA
    ## 521                   ***** NA
    ## 522                   ***** NA
    ## 523                   ***** NA
    ## 524                   ***** NA
    ## 525                   ***** NA
    ## 526                   ***** NA
    ## 527                   ***** NA
    ## 528                   ***** NA
    ## 529                   ***** NA
    ## 530                   ***** NA
    ## 531                   ***** NA
    ## 532                   ***** NA
    ## 533                   ***** NA
    ## 534                   ***** NA
    ## 535                   ***** NA
    ## 536                   ***** NA
    ## 537                   ***** NA
    ## 538                   ***** NA
    ## 539                   ***** NA
    ## 540                   ***** NA
    ## 541                   ***** NA
    ## 542                   ***** NA
    ## 543                   ***** NA
    ## 544                   ***** NA
    ## 545                   ***** NA
    ## 546                   ***** NA
    ## 547                   ***** NA
    ## 548                   ***** NA
    ## 549                   ***** NA
    ## 550                      16 NA
    ## 551                   ***** NA
    ## 552                      16 NA
    ## 553                   ***** NA
    ## 554                   ***** NA
    ## 555                   ***** NA
    ## 556                   ***** NA
    ## 557                   ***** NA
    ## 558                   ***** NA
    ## 559                   ***** NA
    ## 560                   ***** NA
    ## 561                   ***** NA
    ## 562                   ***** NA
    ## 563                   ***** NA
    ## 564                   ***** NA
    ## 565                     105 NA
    ## 566                   ***** NA
    ## 567                   ***** NA
    ## 568                   ***** NA
    ## 569                     105 NA
    ## 570                   ***** NA
    ## 571                   ***** NA
    ## 572                   ***** NA
    ## 573                   ***** NA
    ## 574                   ***** NA
    ## 575                   ***** NA
    ## 576                   ***** NA
    ## 577                   ***** NA
    ## 578                   ***** NA
    ## 579                   ***** NA
    ## 580                   ***** NA
    ## 581                   ***** NA
    ## 582                   ***** NA
    ## 583                   ***** NA
    ## 584                   ***** NA
    ## 585                   ***** NA
    ## 586                   ***** NA
    ## 587                   ***** NA
    ## 588                   ***** NA
    ## 589                   ***** NA
    ## 590                   ***** NA
    ## 591                   ***** NA
    ## 592                   ***** NA
    ## 593                   ***** NA
    ## 594                   ***** NA
    ## 595                   ***** NA
    ## 596                   ***** NA
    ## 597                   ***** NA
    ## 598                   ***** NA
    ## 599                   ***** NA
    ## 600                   ***** NA
    ## 601                   ***** NA
    ## 602                   ***** NA
    ## 603                   ***** NA
    ## 604                   ***** NA
    ## 605                   ***** NA
    ## 606                   ***** NA
    ## 607                   ***** NA
    ## 608                   ***** NA
    ## 609                   ***** NA
    ## 610                   ***** NA
    ## 611                   ***** NA
    ## 612                   ***** NA
    ## 613                   ***** NA
    ## 614                   ***** NA
    ## 615                   ***** NA
    ## 616                   ***** NA
    ## 617                   ***** NA
    ## 618                   ***** NA
    ## 619                   ***** NA
    ## 620                   ***** NA
    ## 621                   ***** NA
    ## 622                   ***** NA
    ## 623                   ***** NA
    ## 624                   ***** NA
    ## 625                   ***** NA
    ## 626                   ***** NA
    ## 627                   ***** NA
    ## 628                   ***** NA
    ## 629                   ***** NA
    ## 630                   ***** NA
    ## 631                   ***** NA
    ## 632                   ***** NA
    ## 633                   ***** NA
    ## 634                   ***** NA
    ## 635                   ***** NA
    ## 636                   ***** NA
    ## 637                   ***** NA
    ## 638                   ***** NA
    ## 639                   ***** NA
    ## 640                   ***** NA
    ## 641                   ***** NA
    ## 642                   ***** NA
    ## 643                   ***** NA
    ## 644                   ***** NA
    ## 645                   ***** NA
    ## 646                   ***** NA
    ## 647                   ***** NA
    ## 648                   ***** NA
    ## 649                   ***** NA
    ## 650                   ***** NA
    ## 651                   ***** NA
    ## 652                   ***** NA
    ## 653                   ***** NA
    ## 654                   ***** NA
    ## 655                   ***** NA
    ## 656                   ***** NA
    ## 657                   ***** NA
    ## 658                   ***** NA
    ## 659                   ***** NA
    ## 660                   ***** NA
    ## 661                   ***** NA
    ## 662                   ***** NA
    ## 663                   ***** NA
    ## 664                   ***** NA
    ## 665                   ***** NA
    ## 666                   ***** NA
    ## 667                   ***** NA
    ## 668                   ***** NA
    ## 669                   ***** NA
    ## 670                   ***** NA
    ## 671                   ***** NA
    ## 672                   ***** NA
    ## 673                   ***** NA
    ## 674                   ***** NA
    ## 675                   ***** NA
    ## 676                   ***** NA
    ## 677                   ***** NA
    ## 678                   ***** NA
    ## 679                   ***** NA
    ## 680                   ***** NA
    ## 681                   ***** NA
    ## 682                   ***** NA
    ## 683                   ***** NA
    ## 684                   ***** NA
    ## 685                   ***** NA
    ## 686                   ***** NA
    ## 687                   ***** NA
    ## 688                   ***** NA
    ## 689                   ***** NA
    ## 690                   ***** NA
    ## 691                   ***** NA
    ## 692                   ***** NA
    ## 693                   ***** NA
    ## 694                   ***** NA
    ## 695                   ***** NA
    ## 696                   ***** NA
    ## 697                   ***** NA
    ## 698                   ***** NA
    ## 699                   ***** NA
    ## 700                   ***** NA
    ## 701                   ***** NA
    ## 702                   ***** NA
    ## 703                   ***** NA
    ## 704                   ***** NA
    ## 705                   ***** NA
    ## 706                   ***** NA
    ## 707                   ***** NA
    ## 708                   ***** NA
    ## 709                   ***** NA
    ## 710                   ***** NA
    ## 711                   ***** NA
    ## 712                   ***** NA
    ## 713                   ***** NA
    ## 714                   ***** NA
    ## 715                   ***** NA
    ## 716                   ***** NA
    ## 717                   ***** NA
    ## 718                   ***** NA
    ## 719                   ***** NA
    ## 720                   ***** NA
    ## 721                   ***** NA
    ## 722                   ***** NA
    ## 723                   ***** NA
    ## 724                   ***** NA
    ## 725                   ***** NA
    ## 726                   ***** NA
    ## 727                   ***** NA
    ## 728                   ***** NA
    ## 729                   ***** NA
    ## 730                   ***** NA
    ## 731                   ***** NA
    ## 732                   ***** NA
    ## 733                   ***** NA
    ## 734                   ***** NA
    ## 735                   ***** NA
    ## 736                   ***** NA
    ## 737                   ***** NA
    ## 738                   ***** NA
    ## 739                   ***** NA
    ## 740                   ***** NA
    ## 741                   ***** NA
    ## 742                   ***** NA
    ## 743                   ***** NA
    ## 744                   ***** NA
    ## 745                   ***** NA
    ## 746                   ***** NA
    ## 747                   ***** NA
    ## 748                   ***** NA
    ## 749                   ***** NA
    ## 750                   ***** NA
    ## 751                   ***** NA
    ## 752                   ***** NA
    ## 753                   ***** NA
    ## 754                   ***** NA
    ## 755                   ***** NA
    ## 756                   ***** NA
    ## 757                   ***** NA
    ## 758                   ***** NA
    ## 759                   ***** NA
    ## 760                   ***** NA
    ## 761                   ***** NA
    ## 762                   ***** NA
    ## 763                   ***** NA
    ## 764                   ***** NA
    ## 765                   ***** NA
    ## 766                   ***** NA
    ## 767                   ***** NA
    ## 768                   ***** NA
    ## 769                   ***** NA
    ## 770                   ***** NA
    ## 771                   ***** NA
    ## 772                   ***** NA
    ## 773                   ***** NA
    ## 774                   ***** NA
    ## 775                   ***** NA
    ## 776                   ***** NA
    ## 777                   ***** NA
    ## 778                   ***** NA
    ## 779                   ***** NA
    ## 780                   ***** NA
    ## 781                   ***** NA
    ## 782                   ***** NA
    ## 783                   ***** NA
    ## 784                   ***** NA
    ## 785                   ***** NA
    ## 786                   ***** NA
    ## 787                   ***** NA
    ## 788                   ***** NA
    ## 789                   ***** NA
    ## 790                   ***** NA
    ## 791                   ***** NA
    ## 792                   ***** NA
    ## 793                   ***** NA
    ## 794                   ***** NA
    ## 795                   ***** NA
    ## 796                   ***** NA
    ## 797                   ***** NA
    ## 798                   ***** NA
    ## 799                   ***** NA
    ## 800                   ***** NA
    ## 801                   ***** NA
    ## 802                   ***** NA
    ## 803                   ***** NA
    ## 804                   ***** NA
    ## 805                   ***** NA
    ## 806                   ***** NA
    ## 807                   ***** NA
    ## 808                   ***** NA
    ## 809                   ***** NA
    ## 810                   ***** NA
    ## 811                   ***** NA
    ## 812                   ***** NA
    ## 813                   ***** NA
    ## 814                   ***** NA
    ## 815                   ***** NA
    ## 816                   ***** NA
    ## 817                   ***** NA
    ## 818                   ***** NA
    ## 819                   ***** NA
    ## 820                   ***** NA
    ## 821                   ***** NA
    ## 822                   ***** NA
    ## 823                   ***** NA
    ## 824                   ***** NA
    ## 825                   ***** NA
    ## 826                   ***** NA
    ## 827                   ***** NA
    ## 828                   ***** NA
    ## 829                   ***** NA
    ## 830                   ***** NA
    ## 831                   ***** NA
    ## 832                   ***** NA
    ## 833                   ***** NA
    ## 834                   ***** NA
    ## 835                   ***** NA
    ## 836                   ***** NA
    ## 837                   ***** NA
    ## 838                   ***** NA
    ## 839                   ***** NA
    ## 840                   ***** NA
    ## 841                   ***** NA
    ## 842                   ***** NA
    ## 843                   ***** NA
    ## 844                   ***** NA
    ## 845                   ***** NA
    ## 846                   ***** NA
    ## 847                   ***** NA
    ## 848                   ***** NA
    ## 849                   ***** NA
    ## 850                   ***** NA
    ## 851                   ***** NA
    ## 852                   ***** NA
    ## 853                   ***** NA
    ## 854                   ***** NA
    ## 855                   ***** NA
    ## 856                   ***** NA
    ## 857                   ***** NA
    ## 858                   ***** NA
    ## 859                   ***** NA
    ## 860                   ***** NA
    ## 861                   ***** NA
    ## 862                   ***** NA
    ## 863                   ***** NA
    ## 864                   ***** NA
    ## 865                   ***** NA
    ## 866                   ***** NA
    ## 867                   ***** NA
    ## 868                   ***** NA
    ## 869                   ***** NA
    ## 870                   ***** NA
    ## 871                   ***** NA
    ## 872                   ***** NA
    ## 873                   ***** NA
    ## 874                   ***** NA
    ## 875                   ***** NA
    ## 876                   ***** NA
    ## 877                   ***** NA
    ## 878                   ***** NA
    ## 879                   ***** NA
    ## 880                   ***** NA
    ## 881                   ***** NA
    ## 882                   ***** NA
    ## 883                   ***** NA
    ## 884                   ***** NA
    ## 885                   ***** NA
    ## 886                   ***** NA
    ## 887                   ***** NA
    ## 888                   ***** NA
    ## 889                   ***** NA
    ## 890                   ***** NA
    ## 891                   ***** NA
    ## 892                   ***** NA
    ## 893                     149 NA
    ## 894                   ***** NA
    ## 895                   ***** NA
    ## 896                   ***** NA
    ## 897                   ***** NA
    ## 898                   ***** NA
    ## 899                   ***** NA
    ## 900                   ***** NA
    ## 901                   ***** NA
    ## 902                   ***** NA
    ## 903                   ***** NA
    ## 904                   ***** NA
    ## 905                   ***** NA
    ## 906                     149 NA
    ## 907                   ***** NA
    ## 908                   ***** NA
    ## 909                   ***** NA
    ## 910                   ***** NA
    ## 911                   ***** NA
    ## 912                   ***** NA
    ## 913                   ***** NA
    ## 914                   ***** NA
    ## 915                   ***** NA
    ## 916                   ***** NA
    ## 917                   ***** NA
    ## 918                   ***** NA
    ## 919                   ***** NA
    ## 920                   ***** NA
    ## 921                     114 NA
    ## 922                   ***** NA
    ## 923                   ***** NA
    ## 924                   ***** NA
    ## 925                     112 NA
    ## 926                   ***** NA
    ## 927                   ***** NA
    ## 928                   ***** NA
    ## 929                   ***** NA
    ## 930                   ***** NA
    ## 931                     132 NA
    ## 932                   ***** NA
    ## 933                   ***** NA
    ## 934                   ***** NA
    ## 935                   ***** NA
    ## 936                   ***** NA
    ## 937                   ***** NA
    ## 938                   ***** NA
    ## 939                   ***** NA
    ## 940                     114 NA
    ## 941                   ***** NA
    ## 942                   ***** NA
    ## 943                   ***** NA
    ## 944                   ***** NA
    ## 945                   ***** NA
    ## 946                   ***** NA
    ## 947                   ***** NA
    ## 948                   ***** NA
    ## 949                   ***** NA
    ## 950                   ***** NA
    ## 951                   ***** NA
    ## 952                   ***** NA
    ## 953                   ***** NA
    ## 954                   ***** NA
    ## 955                   ***** NA
    ## 956                   ***** NA
    ## 957                   ***** NA
    ## 958                   ***** NA
    ## 959                   ***** NA
    ## 960                   ***** NA
    ## 961                   ***** NA
    ## 962                   ***** NA
    ## 963                   ***** NA
    ## 964                   ***** NA
    ## 965                   ***** NA
    ## 966                   ***** NA
    ## 967                   ***** NA
    ## 968                   ***** NA
    ## 969                   ***** NA
    ## 970                   ***** NA
    ## 971                   ***** NA
    ## 972                     132 NA
    ## 973                   ***** NA
    ## 974                   ***** NA
    ## 975                   ***** NA
    ## 976                   ***** NA
    ## 977                   ***** NA
    ## 978                   ***** NA
    ## 979                   ***** NA
    ## 980                   ***** NA
    ## 981                   ***** NA
    ## 982                   ***** NA
    ## 983                   ***** NA
    ## 984                   ***** NA
    ## 985                   ***** NA
    ## 986                   ***** NA
    ## 987                   ***** NA
    ## 988                   ***** NA
    ## 989                     112 NA
    ## 990                   ***** NA
    ## 991                   ***** NA
    ## 992                   ***** NA
    ## 993                   ***** NA
    ## 994                   ***** NA
    ## 995                   ***** NA
    ## 996                   ***** NA
    ## 997                   ***** NA
    ## 998                   ***** NA
    ## 999                   ***** NA
    ## 1000                  ***** NA
    ## 1001                  ***** NA
    ## 1002                  ***** NA
    ## 1003                  ***** NA
    ## 1004                  ***** NA
    ## 1005                  ***** NA
    ## 1006                  ***** NA
    ## 1007                  ***** NA
    ## 1008                  ***** NA
    ## 1009                  ***** NA
    ## 1010                  ***** NA
    ## 1011                  ***** NA
    ## 1012                  ***** NA
    ## 1013                  ***** NA
    ## 1014                  ***** NA
    ## 1015                  ***** NA
    ## 1016                  ***** NA
    ## 1017                  ***** NA
    ## 1018                  ***** NA
    ## 1019                  ***** NA
    ## 1020                  ***** NA
    ## 1021                  ***** NA
    ## 1022                  ***** NA
    ## 1023                  ***** NA
    ## 1024                  ***** NA
    ## 1025                  ***** NA
    ## 1026                  ***** NA
    ## 1027                  ***** NA
    ## 1028                  ***** NA
    ## 1029                  ***** NA
    ## 1030                  ***** NA
    ## 1031                  ***** NA
    ## 1032                  ***** NA
    ## 1033                  ***** NA
    ## 1034                  ***** NA
    ## 1035                  ***** NA
    ## 1036                  ***** NA
    ## 1037                  ***** NA
    ## 1038                  ***** NA
    ## 1039                  ***** NA
    ## 1040                  ***** NA
    ## 1041                  ***** NA
    ## 1042                  ***** NA
    ## 1043                  ***** NA
    ## 1044                  ***** NA
    ## 1045                  ***** NA
    ## 1046                  ***** NA
    ## 1047                  ***** NA
    ## 1048                  ***** NA
    ## 1049                  ***** NA
    ## 1050                  ***** NA
    ## 1051                  ***** NA
    ## 1052                  ***** NA
    ## 1053                  ***** NA
    ## 1054                  ***** NA
    ## 1055                  ***** NA
    ## 1056                  ***** NA
    ## 1057                  ***** NA
    ## 1058                  ***** NA
    ## 1059                  ***** NA
    ## 1060                  ***** NA
    ## 1061                  ***** NA
    ## 1062                  ***** NA
    ## 1063                  ***** NA
    ## 1064                  ***** NA
    ## 1065                  ***** NA
    ## 1066                  ***** NA
    ## 1067                  ***** NA
    ## 1068                  ***** NA
    ## 1069                  ***** NA
    ## 1070                  ***** NA
    ## 1071                  ***** NA
    ## 1072                  ***** NA
    ## 1073                  ***** NA
    ## 1074                  ***** NA
    ## 1075                  ***** NA
    ## 1076                  ***** NA
    ## 1077                  ***** NA
    ## 1078                  ***** NA
    ## 1079                  ***** NA
    ## 1080                  ***** NA
    ## 1081                  ***** NA
    ## 1082                  ***** NA
    ## 1083                  ***** NA
    ## 1084                  ***** NA
    ## 1085                  ***** NA
    ## 1086                  ***** NA
    ## 1087                  ***** NA
    ## 1088                  ***** NA
    ## 1089                  ***** NA
    ## 1090                  ***** NA
    ## 1091                  ***** NA
    ## 1092                  ***** NA
    ## 1093                  ***** NA
    ## 1094                  ***** NA
    ## 1095                  ***** NA
    ## 1096                  ***** NA
    ## 1097                  ***** NA
    ## 1098                  ***** NA
    ## 1099                  ***** NA
    ## 1100                  ***** NA
    ## 1101                  ***** NA
    ## 1102                  ***** NA
    ## 1103                  ***** NA
    ## 1104                  ***** NA
    ## 1105                  ***** NA
    ## 1106                  ***** NA
    ## 1107                  ***** NA
    ## 1108                  ***** NA
    ## 1109                  ***** NA
    ## 1110                  ***** NA
    ## 1111                  ***** NA
    ## 1112                  ***** NA
    ## 1113                  ***** NA
    ## 1114                  ***** NA
    ## 1115                  ***** NA
    ## 1116                  ***** NA
    ## 1117                  ***** NA
    ## 1118                  ***** NA
    ## 1119                  ***** NA
    ## 1120                  ***** NA
    ## 1121                  ***** NA
    ## 1122                  ***** NA
    ## 1123                  ***** NA
    ## 1124                  ***** NA
    ## 1125                  ***** NA
    ## 1126                  ***** NA
    ## 1127                  ***** NA
    ## 1128                  ***** NA
    ## 1129                  ***** NA
    ## 1130                  ***** NA
    ## 1131                  ***** NA
    ## 1132                  ***** NA
    ## 1133                  ***** NA
    ## 1134                  ***** NA
    ## 1135                  ***** NA
    ## 1136                  ***** NA
    ## 1137                  ***** NA
    ## 1138                  ***** NA
    ## 1139                  ***** NA
    ## 1140                  ***** NA
    ## 1141                  ***** NA
    ## 1142                  ***** NA
    ## 1143                  ***** NA
    ## 1144                  ***** NA
    ## 1145                  ***** NA
    ## 1146                  ***** NA
    ## 1147                  ***** NA
    ## 1148                  ***** NA
    ## 1149                  ***** NA
    ## 1150                  ***** NA
    ## 1151                  ***** NA
    ## 1152                  ***** NA
    ## 1153                  ***** NA
    ## 1154                  ***** NA
    ## 1155                  ***** NA
    ## 1156                  ***** NA
    ## 1157                  ***** NA
    ## 1158                  ***** NA
    ## 1159                  ***** NA
    ## 1160                  ***** NA
    ## 1161                  ***** NA
    ## 1162                  ***** NA
    ## 1163                  ***** NA
    ## 1164                  ***** NA
    ## 1165                  ***** NA
    ## 1166                  ***** NA
    ## 1167                  ***** NA
    ## 1168                  ***** NA
    ## 1169                  ***** NA
    ## 1170                  ***** NA
    ## 1171                  ***** NA
    ## 1172                  ***** NA
    ## 1173                  ***** NA
    ## 1174                  ***** NA
    ## 1175                  ***** NA
    ## 1176                  ***** NA
    ## 1177                  ***** NA
    ## 1178                  ***** NA
    ## 1179                  ***** NA
    ## 1180                  ***** NA
    ## 1181                  ***** NA
    ## 1182                  ***** NA
    ## 1183                  ***** NA
    ## 1184                  ***** NA
    ## 1185                  ***** NA
    ## 1186                  ***** NA
    ## 1187                  ***** NA
    ## 1188                  ***** NA
    ## 1189                  ***** NA
    ## 1190                  ***** NA
    ## 1191                  ***** NA
    ## 1192                  ***** NA
    ## 1193                  ***** NA
    ## 1194                  ***** NA
    ## 1195                  ***** NA
    ## 1196                  ***** NA
    ## 1197                  ***** NA
    ## 1198                  ***** NA
    ## 1199                  ***** NA
    ## 1200                  ***** NA
    ## 1201                  ***** NA
    ## 1202                  ***** NA
    ## 1203                  ***** NA
    ## 1204                  ***** NA
    ## 1205                  ***** NA
    ## 1206                  ***** NA
    ## 1207                  ***** NA
    ## 1208                  ***** NA
    ## 1209                  ***** NA
    ## 1210                  ***** NA
    ## 1211                  ***** NA
    ## 1212                  ***** NA
    ## 1213                  ***** NA
    ## 1214                  ***** NA
    ## 1215                  ***** NA
    ## 1216                  ***** NA
    ## 1217                  ***** NA
    ## 1218                  ***** NA
    ## 1219                  ***** NA
    ## 1220                  ***** NA
    ## 1221                  ***** NA
    ## 1222                  ***** NA
    ## 1223                  ***** NA
    ## 1224                  ***** NA
    ## 1225                  ***** NA
    ## 1226                  ***** NA
    ## 1227                  ***** NA
    ## 1228                  ***** NA
    ## 1229                  ***** NA
    ## 1230                  ***** NA
    ## 1231                  ***** NA
    ## 1232                  ***** NA
    ## 1233                  ***** NA
    ## 1234                  ***** NA
    ## 1235                  ***** NA
    ## 1236                  ***** NA
    ## 1237                  ***** NA
    ## 1238                  ***** NA
    ## 1239                  ***** NA
    ## 1240                  ***** NA
    ## 1241                  ***** NA
    ## 1242                  ***** NA
    ## 1243                  ***** NA
    ## 1244                  ***** NA
    ## 1245                  ***** NA
    ## 1246                  ***** NA
    ## 1247                  ***** NA
    ## 1248                  ***** NA
    ## 1249                  ***** NA
    ## 1250                  ***** NA
    ## 1251                  ***** NA
    ## 1252                  ***** NA
    ## 1253                  ***** NA
    ## 1254                  ***** NA
    ## 1255                  ***** NA
    ## 1256                  ***** NA
    ## 1257                  ***** NA
    ## 1258                  ***** NA
    ## 1259                  ***** NA
    ## 1260                  ***** NA
    ## 1261                  ***** NA
    ## 1262                  ***** NA
    ## 1263                  ***** NA
    ## 1264                  ***** NA
    ## 1265                  ***** NA
    ## 1266                  ***** NA
    ## 1267                  ***** NA
    ## 1268                  ***** NA
    ## 1269                  ***** NA
    ## 1270                  ***** NA
    ## 1271                  ***** NA
    ## 1272                  ***** NA
    ## 1273                  ***** NA
    ## 1274                  ***** NA
    ## 1275                  ***** NA
    ## 1276                  ***** NA
    ## 1277                  ***** NA
    ## 1278                  ***** NA
    ## 1279                  ***** NA
    ## 1280                  ***** NA
    ## 1281                  ***** NA
    ## 1282                  ***** NA
    ## 1283                  ***** NA
    ## 1284                  ***** NA
    ## 1285                  ***** NA
    ## 1286                  ***** NA
    ## 1287                  ***** NA
    ## 1288                  ***** NA
    ## 1289                  ***** NA
    ## 1290                  ***** NA
    ## 1291                  ***** NA
    ## 1292                  ***** NA
    ## 1293                  ***** NA
    ## 1294                  ***** NA
    ## 1295                  ***** NA
    ## 1296                  ***** NA
    ## 1297                  ***** NA
    ## 1298                  ***** NA
    ## 1299                  ***** NA
    ## 1300                  ***** NA
    ## 1301                  ***** NA
    ## 1302                  ***** NA
    ## 1303                  ***** NA
    ## 1304                  ***** NA
    ## 1305                  ***** NA
    ## 1306                  ***** NA
    ## 1307                  ***** NA
    ## 1308                  ***** NA
    ## 1309                  ***** NA
    ## 1310                  ***** NA
    ## 1311                  ***** NA
    ## 1312                  ***** NA
    ## 1313                  ***** NA
    ## 1314                  ***** NA
    ## 1315                  ***** NA
    ## 1316                  ***** NA
    ## 1317                  ***** NA
    ## 1318                  ***** NA
    ## 1319                  ***** NA
    ## 1320                  ***** NA
    ## 1321                  ***** NA
    ## 1322                  ***** NA
    ## 1323                  ***** NA
    ## 1324                  ***** NA
    ## 1325                  ***** NA
    ## 1326                  ***** NA
    ## 1327                  ***** NA
    ## 1328                  ***** NA
    ## 1329                  ***** NA
    ## 1330                  ***** NA
    ## 1331                  ***** NA
    ## 1332                  ***** NA
    ## 1333                  ***** NA
    ## 1334                  ***** NA
    ## 1335                  ***** NA
    ## 1336                  ***** NA
    ## 1337                  ***** NA
    ## 1338                  ***** NA
    ## 1339                  ***** NA
    ## 1340                  ***** NA
    ## 1341                  ***** NA
    ## 1342                  ***** NA
    ## 1343                  ***** NA
    ## 1344                  ***** NA
    ## 1345                  ***** NA
    ## 1346                  ***** NA
    ## 1347                  ***** NA
    ## 1348                  ***** NA
    ## 1349                  ***** NA
    ## 1350                  ***** NA
    ## 1351                  ***** NA
    ## 1352                  ***** NA
    ## 1353                  ***** NA
    ## 1354                  ***** NA
    ## 1355                  ***** NA
    ## 1356                  ***** NA
    ## 1357                  ***** NA
    ## 1358                  ***** NA
    ## 1359                  ***** NA
    ## 1360                  ***** NA
    ## 1361                  ***** NA
    ## 1362                  ***** NA
    ## 1363                  ***** NA
    ## 1364                  ***** NA
    ## 1365                  ***** NA
    ## 1366                  ***** NA
    ## 1367                  ***** NA
    ## 1368                  ***** NA
    ## 1369                  ***** NA
    ## 1370                  ***** NA
    ## 1371                  ***** NA
    ## 1372                  ***** NA
    ## 1373                  ***** NA
    ## 1374                  ***** NA
    ## 1375                  ***** NA
    ## 1376                  ***** NA
    ## 1377                  ***** NA
    ## 1378                  ***** NA
    ## 1379                  ***** NA
    ## 1380                  ***** NA
    ## 1381                  ***** NA
    ## 1382                  ***** NA
    ## 1383                  ***** NA
    ## 1384                  ***** NA
    ## 1385                  ***** NA
    ## 1386                  ***** NA
    ## 1387                  ***** NA
    ## 1388                  ***** NA
    ## 1389                  ***** NA
    ## 1390                  ***** NA
    ## 1391                  ***** NA
    ## 1392                  ***** NA
    ## 1393                  ***** NA
    ## 1394                  ***** NA
    ## 1395                  ***** NA
    ## 1396                  ***** NA
    ## 1397                  ***** NA
    ## 1398                  ***** NA
    ## 1399                  ***** NA
    ## 1400                  ***** NA
    ## 1401                  ***** NA
    ## 1402                  ***** NA
    ## 1403                  ***** NA
    ## 1404                  ***** NA
    ## 1405                  ***** NA
    ## 1406                  ***** NA
    ## 1407                  ***** NA
    ## 1408                  ***** NA
    ## 1409                  ***** NA
    ## 1410                  ***** NA
    ## 1411                  ***** NA
    ## 1412                  ***** NA
    ## 1413                  ***** NA
    ## 1414                  ***** NA
    ## 1415                  ***** NA
    ## 1416                  ***** NA
    ## 1417                  ***** NA
    ## 1418                  ***** NA
    ## 1419                  ***** NA
    ## 1420                  ***** NA
    ## 1421                  ***** NA
    ## 1422                  ***** NA
    ## 1423                  ***** NA
    ## 1424                  ***** NA
    ## 1425                  ***** NA
    ## 1426                  ***** NA
    ## 1427                  ***** NA
    ## 1428                  ***** NA
    ## 1429                  ***** NA
    ## 1430                    195 NA
    ## 1431                  ***** NA
    ## 1432                  ***** NA
    ## 1433                  ***** NA
    ## 1434                  ***** NA
    ## 1435                  ***** NA
    ## 1436                  ***** NA
    ## 1437                  ***** NA
    ## 1438                  ***** NA
    ## 1439                  ***** NA
    ## 1440                  ***** NA
    ## 1441                  ***** NA
    ## 1442                  ***** NA
    ## 1443                  ***** NA
    ## 1444                  ***** NA
    ## 1445                  ***** NA
    ## 1446                  ***** NA
    ## 1447                  ***** NA
    ## 1448                  ***** NA
    ## 1449                  ***** NA
    ## 1450                  ***** NA
    ## 1451                  ***** NA
    ## 1452                  ***** NA
    ## 1453                  ***** NA
    ## 1454                  ***** NA
    ## 1455                  ***** NA
    ## 1456                  ***** NA
    ## 1457                  ***** NA
    ## 1458                  ***** NA
    ## 1459                  ***** NA
    ## 1460                  ***** NA
    ## 1461                  ***** NA
    ## 1462                  ***** NA
    ## 1463                  ***** NA
    ## 1464                  ***** NA
    ## 1465                    195 NA
    ## 1466                  ***** NA
    ## 1467                  ***** NA
    ## 1468                  ***** NA
    ## 1469                  ***** NA
    ## 1470                  ***** NA
    ## 1471                  ***** NA
    ## 1472                  ***** NA
    ## 1473                  ***** NA
    ## 1474                  ***** NA
    ## 1475                  ***** NA
    ## 1476                  ***** NA
    ## 1477                  ***** NA
    ## 1478                  ***** NA
    ## 1479                  ***** NA
    ## 1480                  ***** NA
    ## 1481                  ***** NA
    ## 1482                  ***** NA
    ## 1483                  ***** NA
    ## 1484                  ***** NA
    ## 1485                  ***** NA
    ## 1486                  ***** NA
    ## 1487                  ***** NA
    ## 1488                  ***** NA
    ## 1489                  ***** NA
    ## 1490                  ***** NA
    ## 1491                  ***** NA
    ## 1492                  ***** NA
    ## 1493                  ***** NA
    ## 1494                  ***** NA
    ## 1495                  ***** NA
    ## 1496                  ***** NA
    ## 1497                  ***** NA
    ## 1498                  ***** NA
    ## 1499                  ***** NA
    ## 1500                  ***** NA
    ## 1501                  ***** NA
    ## 1502                  ***** NA
    ## 1503                  ***** NA
    ## 1504                  ***** NA
    ## 1505                  ***** NA
    ## 1506                  ***** NA
    ## 1507                  ***** NA
    ## 1508                  ***** NA
    ## 1509                  ***** NA
    ## 1510                  ***** NA
    ## 1511                  ***** NA
    ## 1512                  ***** NA
    ## 1513                  ***** NA
    ## 1514                  ***** NA
    ## 1515                  ***** NA
    ## 1516                  ***** NA
    ## 1517                  ***** NA
    ## 1518                  ***** NA
    ## 1519                  ***** NA
    ## 1520                  ***** NA
    ## 1521                  ***** NA
    ## 1522                  ***** NA
    ## 1523                  ***** NA
    ## 1524                  ***** NA
    ## 1525                  ***** NA
    ## 1526                  ***** NA
    ## 1527                  ***** NA
    ## 1528                  ***** NA
    ## 1529                  ***** NA
    ## 1530                  ***** NA
    ## 1531                  ***** NA
    ## 1532                  ***** NA
    ## 1533                  ***** NA
    ## 1534                  ***** NA
    ## 1535                  ***** NA
    ## 1536                  ***** NA
    ## 1537                  ***** NA
    ## 1538                  ***** NA
    ## 1539                  ***** NA
    ## 1540                  ***** NA
    ## 1541                  ***** NA
    ## 1542                  ***** NA
    ## 1543                  ***** NA
    ## 1544                  ***** NA
    ## 1545                  ***** NA
    ## 1546                  ***** NA
    ## 1547                  ***** NA
    ## 1548                  ***** NA
    ## 1549                  ***** NA
    ## 1550                  ***** NA
    ## 1551                  ***** NA
    ## 1552                  ***** NA
    ## 1553                  ***** NA
    ## 1554                  ***** NA
    ## 1555                  ***** NA
    ## 1556                  ***** NA
    ## 1557                  ***** NA
    ## 1558                  ***** NA
    ## 1559                  ***** NA
    ## 1560                  ***** NA
    ## 1561                  ***** NA
    ## 1562                  ***** NA
    ## 1563                  ***** NA
    ## 1564                  ***** NA
    ## 1565                  ***** NA
    ## 1566                  ***** NA
    ## 1567                  ***** NA
    ## 1568                  ***** NA
    ## 1569                  ***** NA
    ## 1570                  ***** NA
    ## 1571                  ***** NA
    ## 1572                  ***** NA
    ## 1573                  ***** NA
    ## 1574                  ***** NA
    ## 1575                  ***** NA
    ## 1576                  ***** NA
    ## 1577                  ***** NA
    ## 1578                  ***** NA
    ## 1579                  ***** NA
    ## 1580                  ***** NA
    ## 1581                  ***** NA
    ## 1582                  ***** NA
    ## 1583                  ***** NA
    ## 1584                  ***** NA
    ## 1585                  ***** NA
    ## 1586                  ***** NA
    ## 1587                  ***** NA
    ## 1588                  ***** NA
    ## 1589                  ***** NA
    ## 1590                  ***** NA
    ## 1591                  ***** NA
    ## 1592                  ***** NA
    ## 1593                  ***** NA
    ## 1594                  ***** NA
    ## 1595                  ***** NA
    ## 1596                  ***** NA
    ## 1597                  ***** NA
    ## 1598                  ***** NA
    ## 1599                  ***** NA
    ## 1600                  ***** NA
    ## 1601                  ***** NA
    ## 1602                  ***** NA
    ## 1603                  ***** NA
    ## 1604                  ***** NA
    ## 1605                    110 NA
    ## 1606                  ***** NA
    ## 1607                  ***** NA
    ## 1608                  ***** NA
    ## 1609                    157 NA
    ## 1610                  ***** NA
    ## 1611                  ***** NA
    ## 1612                    157 NA
    ## 1613                  ***** NA
    ## 1614                  ***** NA
    ## 1615                  ***** NA
    ## 1616                    114 NA
    ## 1617                  ***** NA
    ## 1618                    100 NA
    ## 1619                  ***** NA
    ## 1620                  ***** NA
    ## 1621                  ***** NA
    ## 1622                  ***** NA
    ## 1623                  ***** NA
    ## 1624                  ***** NA
    ## 1625                    201 NA
    ## 1626                  ***** NA
    ## 1627                    118 NA
    ## 1628                  ***** NA
    ## 1629                    201 NA
    ## 1630                  ***** NA
    ## 1631                  ***** NA
    ## 1632                    108 NA
    ## 1633                  ***** NA
    ## 1634                     57 NA
    ## 1635                  ***** NA
    ## 1636                  ***** NA
    ## 1637                    110 NA
    ## 1638                  ***** NA
    ## 1639                    116 NA
    ## 1640                  ***** NA
    ## 1641                  ***** NA
    ## 1642                  ***** NA
    ## 1643                  ***** NA
    ## 1644                  ***** NA
    ## 1645                    157 NA
    ## 1646                  ***** NA
    ## 1647                  ***** NA
    ## 1648                  ***** NA
    ## 1649                  ***** NA
    ## 1650                  ***** NA
    ## 1651                     88 NA
    ## 1652                  ***** NA
    ## 1653                  ***** NA
    ## 1654                    145 NA
    ## 1655                  ***** NA
    ## 1656                  ***** NA
    ## 1657                  ***** NA
    ## 1658                     46 NA
    ## 1659                     72 NA
    ## 1660                     55 NA
    ## 1661                  ***** NA
    ## 1662                  ***** NA
    ## 1663                     90 NA
    ## 1664                    104 NA
    ## 1665                  ***** NA
    ## 1666                  ***** NA
    ## 1667                  ***** NA
    ## 1668                  ***** NA
    ## 1669                  ***** NA
    ## 1670                    214 NA
    ## 1671                  ***** NA
    ## 1672                  ***** NA
    ## 1673                  ***** NA
    ## 1674                  ***** NA
    ## 1675                  ***** NA
    ## 1676                  ***** NA
    ## 1677                  ***** NA
    ## 1678                  ***** NA
    ## 1679                  ***** NA
    ## 1680                    120 NA
    ## 1681                  ***** NA
    ## 1682                  ***** NA
    ## 1683                  ***** NA
    ## 1684                    214 NA
    ## 1685                  ***** NA
    ## 1686                  ***** NA
    ## 1687                     74 NA
    ## 1688                  ***** NA
    ## 1689                  ***** NA
    ## 1690                    117 NA
    ## 1691                    102 NA
    ## 1692                  ***** NA
    ## 1693                     63 NA
    ## 1694                  ***** NA
    ## 1695                  ***** NA
    ## 1696                  ***** NA
    ## 1697                  ***** NA
    ## 1698                     74 NA
    ## 1699                  ***** NA
    ## 1700                  ***** NA
    ## 1701                     82 NA
    ## 1702                  ***** NA
    ## 1703                  ***** NA
    ## 1704                  ***** NA
    ## 1705                  ***** NA
    ## 1706                  ***** NA
    ## 1707                    104 NA
    ## 1708                  ***** NA
    ## 1709                  ***** NA
    ## 1710                  ***** NA
    ## 1711                  ***** NA
    ## 1712                     75 NA
    ## 1713                     61 NA
    ## 1714                     89 NA
    ## 1715                  ***** NA
    ## 1716                  ***** NA
    ## 1717                  ***** NA
    ## 1718                  ***** NA
    ## 1719                  ***** NA
    ## 1720                  ***** NA
    ## 1721                  ***** NA
    ## 1722                  ***** NA
    ## 1723                  ***** NA
    ## 1724                  ***** NA
    ## 1725                  ***** NA
    ## 1726                  ***** NA
    ## 1727                  ***** NA
    ## 1728                  ***** NA
    ## 1729                  ***** NA
    ## 1730                     90 NA
    ## 1731                  ***** NA
    ## 1732                  ***** NA
    ## 1733                  ***** NA
    ## 1734                  ***** NA
    ## 1735                  ***** NA
    ## 1736                  ***** NA
    ## 1737                  ***** NA
    ## 1738                     72 NA
    ## 1739                  ***** NA
    ## 1740                  ***** NA
    ## 1741                     72 NA
    ## 1742                  ***** NA
    ## 1743                     61 NA
    ## 1744                  ***** NA
    ## 1745                  ***** NA
    ## 1746                  ***** NA
    ## 1747                    102 NA
    ## 1748                  ***** NA
    ## 1749                  ***** NA
    ## 1750                  ***** NA
    ## 1751                  ***** NA
    ## 1752                  ***** NA
    ## 1753                    167 NA
    ## 1754                    167 NA
    ## 1755                  ***** NA
    ## 1756                  ***** NA
    ## 1757                  ***** NA
    ## 1758                  ***** NA
    ## 1759                  ***** NA
    ## 1760                  ***** NA
    ## 1761                  ***** NA
    ## 1762                  ***** NA
    ## 1763                  ***** NA
    ## 1764                  ***** NA
    ## 1765                  ***** NA
    ## 1766                  ***** NA
    ## 1767                  ***** NA
    ## 1768                  ***** NA
    ## 1769                  ***** NA
    ## 1770                  ***** NA
    ## 1771                  ***** NA
    ## 1772                  ***** NA
    ## 1773                  ***** NA
    ## 1774                  ***** NA
    ## 1775                  ***** NA
    ## 1776                  ***** NA
    ## 1777                  ***** NA
    ## 1778                  ***** NA
    ## 1779                  ***** NA
    ## 1780                  ***** NA
    ## 1781                  ***** NA
    ## 1782                  ***** NA
    ## 1783                  ***** NA
    ## 1784                  ***** NA
    ## 1785                  ***** NA
    ## 1786                  ***** NA
    ## 1787                  ***** NA
    ## 1788                  ***** NA
    ## 1789                  ***** NA
    ## 1790                  ***** NA
    ## 1791                  ***** NA
    ## 1792                  ***** NA
    ## 1793                  ***** NA
    ## 1794                  ***** NA
    ## 1795                  ***** NA
    ## 1796                  ***** NA
    ## 1797                  ***** NA
    ## 1798                  ***** NA
    ## 1799                  ***** NA
    ## 1800                  ***** NA
    ## 1801                  ***** NA
    ## 1802                  ***** NA
    ## 1803                     69 NA
    ## 1804                  ***** NA
    ## 1805                  ***** NA
    ## 1806                  ***** NA
    ## 1807                  ***** NA
    ## 1808                     69 NA
    ## 1809                  ***** NA
    ## 1810                  ***** NA
    ## 1811                  ***** NA
    ## 1812                  ***** NA
    ## 1813                  ***** NA
    ## 1814                  ***** NA
    ## 1815                  ***** NA
    ## 1816                  ***** NA
    ## 1817                  ***** NA
    ## 1818                  ***** NA
    ## 1819                  ***** NA
    ## 1820                  ***** NA
    ## 1821                  ***** NA
    ## 1822                  ***** NA
    ## 1823                  ***** NA
    ## 1824                  ***** NA
    ## 1825                  ***** NA
    ## 1826                  ***** NA
    ## 1827                  ***** NA
    ## 1828                  ***** NA
    ## 1829                  ***** NA
    ## 1830                  ***** NA
    ## 1831                  ***** NA
    ## 1832                  ***** NA
    ## 1833                  ***** NA
    ## 1834                  ***** NA
    ## 1835                  ***** NA
    ## 1836                  ***** NA
    ## 1837                  ***** NA
    ## 1838                  ***** NA
    ## 1839                  ***** NA
    ## 1840                  ***** NA
    ## 1841                  ***** NA
    ## 1842                  ***** NA
    ## 1843                  ***** NA
    ## 1844                  ***** NA
    ## 1845                  ***** NA
    ## 1846                  ***** NA
    ## 1847                  ***** NA
    ## 1848                  ***** NA
    ## 1849                  ***** NA
    ## 1850                  ***** NA
    ## 1851                  ***** NA
    ## 1852                  ***** NA
    ## 1853                  ***** NA
    ## 1854                  ***** NA
    ## 1855                  ***** NA
    ## 1856                  ***** NA
    ## 1857                  ***** NA
    ## 1858                  ***** NA
    ## 1859                  ***** NA
    ## 1860                  ***** NA
    ## 1861                  ***** NA
    ## 1862                  ***** NA
    ## 1863                  ***** NA
    ## 1864                  ***** NA
    ## 1865                  ***** NA
    ## 1866                  ***** NA
    ## 1867                  ***** NA
    ## 1868                  ***** NA
    ## 1869                  ***** NA
    ## 1870                  ***** NA
    ## 1871                  ***** NA
    ## 1872                  ***** NA
    ## 1873                  ***** NA
    ## 1874                  ***** NA
    ## 1875                  ***** NA
    ## 1876                  ***** NA
    ## 1877                  ***** NA
    ## 1878                  ***** NA
    ## 1879                  ***** NA
    ## 1880                  ***** NA
    ## 1881                  ***** NA
    ## 1882                  ***** NA
    ## 1883                  ***** NA
    ## 1884                  ***** NA
    ## 1885                  ***** NA
    ## 1886                  ***** NA
    ## 1887                  ***** NA
    ## 1888                  ***** NA
    ## 1889                  ***** NA
    ## 1890                  ***** NA
    ## 1891                  ***** NA
    ## 1892                  ***** NA
    ## 1893                  ***** NA
    ## 1894                  ***** NA
    ## 1895                  ***** NA
    ## 1896                  ***** NA
    ## 1897                  ***** NA
    ## 1898                  ***** NA
    ## 1899                  ***** NA
    ## 1900                  ***** NA
    ## 1901                  ***** NA
    ## 1902                  ***** NA
    ## 1903                  ***** NA
    ## 1904                  ***** NA
    ## 1905                  ***** NA
    ## 1906                  ***** NA
    ## 1907                  ***** NA
    ## 1908                  ***** NA
    ## 1909                  ***** NA
    ## 1910                  ***** NA
    ## 1911                  ***** NA
    ## 1912                  ***** NA
    ## 1913                  ***** NA
    ## 1914                  ***** NA
    ## 1915                  ***** NA
    ## 1916                  ***** NA
    ## 1917                  ***** NA
    ## 1918                  ***** NA
    ## 1919                  ***** NA
    ## 1920                  ***** NA
    ## 1921                  ***** NA
    ## 1922                  ***** NA
    ## 1923                  ***** NA
    ## 1924                  ***** NA
    ## 1925                  ***** NA
    ## 1926                  ***** NA
    ## 1927                  ***** NA
    ## 1928                  ***** NA
    ## 1929                  ***** NA
    ## 1930                  ***** NA
    ## 1931                  ***** NA
    ## 1932                  ***** NA
    ## 1933                  ***** NA
    ## 1934                  ***** NA
    ## 1935                  ***** NA
    ## 1936                  ***** NA
    ## 1937                  ***** NA
    ## 1938                  ***** NA
    ## 1939                  ***** NA
    ## 1940                  ***** NA
    ## 1941                  ***** NA
    ## 1942                  ***** NA
    ## 1943                  ***** NA
    ## 1944                  ***** NA
    ## 1945                  ***** NA
    ## 1946                  ***** NA
    ## 1947                  ***** NA
    ## 1948                  ***** NA
    ## 1949                  ***** NA
    ## 1950                  ***** NA
    ## 1951                  ***** NA
    ## 1952                  ***** NA
    ## 1953                  ***** NA
    ## 1954                  ***** NA
    ## 1955                  ***** NA
    ## 1956                  ***** NA
    ## 1957                  ***** NA
    ## 1958                  ***** NA
    ## 1959                  ***** NA
    ## 1960                  ***** NA
    ## 1961                  ***** NA
    ## 1962                  ***** NA
    ## 1963                  ***** NA
    ## 1964                  ***** NA
    ## 1965                  ***** NA
    ## 1966                  ***** NA
    ## 1967                  ***** NA
    ## 1968                  ***** NA
    ## 1969                  ***** NA
    ## 1970                  ***** NA
    ## 1971                  ***** NA
    ## 1972                  ***** NA
    ## 1973                  ***** NA
    ## 1974                  ***** NA
    ## 1975                  ***** NA
    ## 1976                  ***** NA
    ## 1977                  ***** NA
    ## 1978                  ***** NA
    ## 1979                  ***** NA
    ## 1980                  ***** NA
    ## 1981                  ***** NA
    ## 1982                  ***** NA
    ## 1983                  ***** NA
    ## 1984                  ***** NA
    ## 1985                  ***** NA
    ## 1986                  ***** NA
    ## 1987                  ***** NA
    ## 1988                  ***** NA
    ## 1989                  ***** NA
    ## 1990                  ***** NA
    ## 1991                  ***** NA
    ## 1992                  ***** NA
    ## 1993                  ***** NA
    ## 1994                  ***** NA
    ## 1995                     90 NA
    ## 1996                  ***** NA
    ## 1997                  ***** NA
    ## 1998                  ***** NA
    ## 1999                  ***** NA
    ## 2000                  ***** NA
    ## 2001                  ***** NA
    ## 2002                  ***** NA
    ## 2003                  ***** NA
    ## 2004                  ***** NA
    ## 2005                  ***** NA
    ## 2006                  ***** NA
    ## 2007                  ***** NA
    ## 2008                    100 NA
    ## 2009                  ***** NA
    ## 2010                  ***** NA
    ## 2011                  ***** NA
    ## 2012                  ***** NA
    ## 2013                  ***** NA
    ## 2014                  ***** NA
    ## 2015                  ***** NA
    ## 2016                  ***** NA
    ## 2017                  ***** NA
    ## 2018                  ***** NA
    ## 2019                  ***** NA
    ## 2020                  ***** NA
    ## 2021                  ***** NA
    ## 2022                  ***** NA
    ## 2023                  ***** NA
    ## 2024                     92 NA
    ## 2025                  ***** NA
    ## 2026                  ***** NA
    ## 2027                  ***** NA
    ## 2028                  ***** NA
    ## 2029                  ***** NA
    ## 2030                  ***** NA
    ## 2031                  ***** NA
    ## 2032                  ***** NA
    ## 2033                     92 NA
    ## 2034                  ***** NA
    ## 2035                     70 NA
    ## 2036                  ***** NA
    ## 2037                  ***** NA
    ## 2038                  ***** NA
    ## 2039                  ***** NA
    ## 2040                  ***** NA
    ## 2041                  ***** NA
    ## 2042                  ***** NA
    ## 2043                  ***** NA
    ## 2044                  ***** NA
    ## 2045                  ***** NA
    ## 2046                  ***** NA
    ## 2047                  ***** NA
    ## 2048                  ***** NA
    ## 2049                  ***** NA
    ## 2050                  ***** NA
    ## 2051                  ***** NA
    ## 2052                  ***** NA
    ## 2053                  ***** NA
    ## 2054                  ***** NA
    ## 2055                  ***** NA
    ## 2056                  ***** NA
    ## 2057                  ***** NA
    ## 2058                  ***** NA
    ## 2059                  ***** NA
    ## 2060                  ***** NA
    ## 2061                  ***** NA
    ## 2062                  ***** NA
    ## 2063                  ***** NA
    ## 2064                  ***** NA
    ## 2065                  ***** NA
    ## 2066                  ***** NA
    ## 2067                  ***** NA
    ## 2068                  ***** NA
    ## 2069                  ***** NA
    ## 2070                  ***** NA
    ## 2071                  ***** NA
    ## 2072                  ***** NA
    ## 2073                  ***** NA
    ## 2074                  ***** NA
    ## 2075                  ***** NA
    ## 2076                  ***** NA
    ## 2077                  ***** NA
    ## 2078                  ***** NA
    ## 2079                  ***** NA
    ## 2080                  ***** NA
    ## 2081                  ***** NA
    ## 2082                  ***** NA
    ## 2083                  ***** NA
    ## 2084                  ***** NA
    ## 2085                  ***** NA
    ## 2086                  ***** NA
    ## 2087                  ***** NA
    ## 2088                  ***** NA
    ## 2089                  ***** NA
    ## 2090                  ***** NA
    ## 2091                  ***** NA
    ## 2092                  ***** NA
    ## 2093                  ***** NA
    ## 2094                  ***** NA
    ## 2095                  ***** NA
    ## 2096                  ***** NA
    ## 2097                  ***** NA
    ## 2098                  ***** NA
    ## 2099                  ***** NA
    ## 2100                  ***** NA
    ## 2101                  ***** NA
    ## 2102                  ***** NA
    ## 2103                  ***** NA
    ## 2104                  ***** NA
    ## 2105                  ***** NA
    ## 2106                  ***** NA
    ## 2107                  ***** NA
    ## 2108                  ***** NA
    ## 2109                  ***** NA
    ## 2110                  ***** NA
    ## 2111                  ***** NA
    ## 2112                  ***** NA
    ## 2113                  ***** NA
    ## 2114                  ***** NA
    ## 2115                  ***** NA
    ## 2116                  ***** NA
    ## 2117                  ***** NA
    ## 2118                  ***** NA
    ## 2119                  ***** NA
    ## 2120                  ***** NA
    ## 2121                  ***** NA
    ## 2122                  ***** NA
    ## 2123                  ***** NA
    ## 2124                  ***** NA
    ## 2125                  ***** NA
    ## 2126                  ***** NA
    ## 2127                  ***** NA
    ## 2128                  ***** NA
    ## 2129                  ***** NA
    ## 2130                  ***** NA
    ## 2131                  ***** NA
    ## 2132                  ***** NA
    ## 2133                  ***** NA
    ## 2134                  ***** NA
    ## 2135                  ***** NA
    ## 2136                  ***** NA
    ## 2137                  ***** NA
    ## 2138                  ***** NA
    ## 2139                  ***** NA
    ## 2140                  ***** NA
    ## 2141                  ***** NA
    ## 2142                  ***** NA
    ## 2143                  ***** NA
    ## 2144                  ***** NA
    ## 2145                  ***** NA
    ## 2146                  ***** NA
    ## 2147                  ***** NA
    ## 2148                  ***** NA
    ## 2149                  ***** NA
    ## 2150                  ***** NA
    ## 2151                  ***** NA
    ## 2152                  ***** NA
    ## 2153                  ***** NA
    ## 2154                  ***** NA
    ## 2155                  ***** NA
    ## 2156                  ***** NA
    ## 2157                  ***** NA
    ## 2158                  ***** NA
    ## 2159                  ***** NA
    ## 2160                  ***** NA
    ## 2161                  ***** NA
    ## 2162                  ***** NA
    ## 2163                  ***** NA
    ## 2164                  ***** NA
    ## 2165                  ***** NA
    ## 2166                  ***** NA
    ## 2167                  ***** NA
    ## 2168                  ***** NA
    ## 2169                  ***** NA
    ## 2170                  ***** NA
    ## 2171                  ***** NA
    ## 2172                  ***** NA
    ## 2173                  ***** NA
    ## 2174                  ***** NA
    ## 2175                  ***** NA
    ## 2176                  ***** NA
    ## 2177                  ***** NA
    ## 2178                  ***** NA
    ## 2179                  ***** NA
    ## 2180                  ***** NA
    ## 2181                  ***** NA
    ## 2182                  ***** NA
    ## 2183                  ***** NA
    ## 2184                  ***** NA
    ## 2185                  ***** NA
    ## 2186                  ***** NA
    ## 2187                  ***** NA
    ## 2188                  ***** NA
    ## 2189                  ***** NA
    ## 2190                  ***** NA
    ## 2191                  ***** NA
    ## 2192                  ***** NA
    ## 2193                  ***** NA
    ## 2194                  ***** NA
    ## 2195                  ***** NA
    ## 2196                  ***** NA
    ## 2197                  ***** NA
    ## 2198                  ***** NA
    ## 2199                  ***** NA
    ## 2200                  ***** NA
    ## 2201                  ***** NA
    ## 2202                  ***** NA
    ## 2203                  ***** NA
    ## 2204                  ***** NA
    ## 2205                  ***** NA
    ## 2206                  ***** NA
    ## 2207                  ***** NA
    ## 2208                  ***** NA
    ## 2209                  ***** NA
    ## 2210                  ***** NA
    ## 2211                  ***** NA
    ## 2212                  ***** NA
    ## 2213                  ***** NA
    ## 2214                  ***** NA
    ## 2215                  ***** NA
    ## 2216                  ***** NA
    ## 2217                  ***** NA
    ## 2218                  ***** NA
    ## 2219                  ***** NA
    ## 2220                    144 NA
    ## 2221                  ***** NA
    ## 2222                  ***** NA
    ## 2223                  ***** NA
    ## 2224                  ***** NA
    ## 2225                  ***** NA
    ## 2226                  ***** NA
    ## 2227                  ***** NA
    ## 2228                  ***** NA
    ## 2229                  ***** NA
    ## 2230                  ***** NA
    ## 2231                  ***** NA
    ## 2232                  ***** NA
    ## 2233                  ***** NA
    ## 2234                  ***** NA
    ## 2235                  ***** NA
    ## 2236                  ***** NA
    ## 2237                    106 NA
    ## 2238                  ***** NA
    ## 2239                  ***** NA
    ## 2240                  ***** NA
    ## 2241                  ***** NA
    ## 2242                  ***** NA
    ## 2243                  ***** NA
    ## 2244                    124 NA
    ## 2245                  ***** NA
    ## 2246                  ***** NA
    ## 2247                  ***** NA
    ## 2248                  ***** NA
    ## 2249                  ***** NA
    ## 2250                  ***** NA
    ## 2251                  ***** NA
    ## 2252                  ***** NA
    ## 2253                  ***** NA
    ## 2254                  ***** NA
    ## 2255                  ***** NA
    ## 2256                  ***** NA
    ## 2257                  ***** NA
    ## 2258                  ***** NA
    ## 2259                  ***** NA
    ## 2260                  ***** NA
    ## 2261                  ***** NA
    ## 2262                  ***** NA
    ## 2263                  ***** NA
    ## 2264                  ***** NA
    ## 2265                  ***** NA
    ## 2266                  ***** NA
    ## 2267                  ***** NA
    ## 2268                  ***** NA
    ## 2269                  ***** NA
    ## 2270                  ***** NA
    ## 2271                  ***** NA
    ## 2272                  ***** NA
    ## 2273                  ***** NA
    ## 2274                  ***** NA
    ## 2275                  ***** NA
    ## 2276                  ***** NA
    ## 2277                  ***** NA
    ## 2278                  ***** NA
    ## 2279                  ***** NA
    ## 2280                  ***** NA
    ## 2281                  ***** NA
    ## 2282                  ***** NA
    ## 2283                  ***** NA
    ## 2284                  ***** NA
    ## 2285                  ***** NA
    ## 2286                  ***** NA
    ## 2287                  ***** NA
    ## 2288                  ***** NA
    ## 2289                  ***** NA
    ## 2290                  ***** NA
    ## 2291                  ***** NA
    ## 2292                  ***** NA
    ## 2293                  ***** NA
    ## 2294                  ***** NA
    ## 2295                  ***** NA
    ## 2296                  ***** NA
    ## 2297                  ***** NA
    ## 2298                  ***** NA
    ## 2299                  ***** NA
    ## 2300                  ***** NA
    ## 2301                  ***** NA
    ## 2302                  ***** NA
    ## 2303                  ***** NA
    ## 2304                  ***** NA
    ## 2305                  ***** NA
    ## 2306                  ***** NA
    ## 2307                  ***** NA
    ## 2308                  ***** NA
    ## 2309                  ***** NA
    ## 2310                  ***** NA
    ## 2311                  ***** NA
    ## 2312                  ***** NA
    ## 2313                  ***** NA
    ## 2314                  ***** NA
    ## 2315                  ***** NA
    ## 2316                  ***** NA
    ## 2317                  ***** NA
    ## 2318                  ***** NA
    ## 2319                  ***** NA
    ## 2320                  ***** NA
    ## 2321                  ***** NA
    ## 2322                  ***** NA
    ## 2323                  ***** NA
    ## 2324                  ***** NA
    ## 2325                  ***** NA
    ## 2326                  ***** NA
    ## 2327                  ***** NA
    ## 2328                  ***** NA
    ## 2329                  ***** NA
    ## 2330                  ***** NA
    ## 2331                  ***** NA
    ## 2332                  ***** NA
    ## 2333                  ***** NA
    ## 2334                  ***** NA
    ## 2335                  ***** NA
    ## 2336                  ***** NA
    ## 2337                  ***** NA
    ## 2338                  ***** NA
    ## 2339                  ***** NA
    ## 2340                  ***** NA
    ## 2341                  ***** NA
    ## 2342                  ***** NA
    ## 2343                  ***** NA
    ## 2344                  ***** NA
    ## 2345                  ***** NA
    ## 2346                  ***** NA
    ## 2347                  ***** NA
    ## 2348                  ***** NA
    ## 2349                  ***** NA
    ## 2350                  ***** NA
    ## 2351                  ***** NA
    ## 2352                  ***** NA
    ## 2353                  ***** NA
    ## 2354                  ***** NA
    ## 2355                  ***** NA
    ## 2356                  ***** NA
    ## 2357                  ***** NA
    ## 2358                  ***** NA
    ## 2359                  ***** NA
    ## 2360                  ***** NA
    ## 2361                  ***** NA
    ## 2362                  ***** NA
    ## 2363                  ***** NA
    ## 2364                  ***** NA
    ## 2365                  ***** NA
    ## 2366                  ***** NA
    ## 2367                  ***** NA
    ## 2368                  ***** NA
    ## 2369                  ***** NA
    ## 2370                  ***** NA
    ## 2371                  ***** NA
    ## 2372                  ***** NA
    ## 2373                    168 NA
    ## 2374                  ***** NA
    ## 2375                  ***** NA
    ## 2376                  ***** NA
    ## 2377                  ***** NA
    ## 2378                  ***** NA
    ## 2379                  ***** NA
    ## 2380                  ***** NA
    ## 2381                  ***** NA
    ## 2382                  ***** NA
    ## 2383                  ***** NA
    ## 2384                  ***** NA
    ## 2385                  ***** NA
    ## 2386                  ***** NA
    ## 2387                  ***** NA
    ## 2388                  ***** NA
    ## 2389                  ***** NA
    ## 2390                     96 NA
    ## 2391                  ***** NA
    ## 2392                    112 NA
    ## 2393                  ***** NA
    ## 2394                     95 NA
    ## 2395                  ***** NA
    ## 2396                  ***** NA
    ## 2397                    112 NA
    ## 2398                  ***** NA
    ## 2399                  ***** NA
    ## 2400                     96 NA
    ## 2401                  ***** NA
    ## 2402                  ***** NA
    ## 2403                  ***** NA
    ## 2404                  ***** NA
    ## 2405                  ***** NA
    ## 2406                  ***** NA
    ## 2407                    168 NA
    ## 2408                  ***** NA
    ## 2409                  ***** NA
    ## 2410                  ***** NA
    ## 2411                  ***** NA
    ## 2412                  ***** NA
    ## 2413                  ***** NA
    ## 2414                  ***** NA
    ## 2415                  ***** NA
    ## 2416                     95 NA
    ## 2417                    104 NA
    ## 2418                  ***** NA
    ## 2419                  ***** NA
    ## 2420                  ***** NA
    ## 2421                  ***** NA
    ## 2422                    104 NA
    ## 2423                  ***** NA
    ## 2424                  ***** NA
    ## 2425                  ***** NA
    ## 2426                  ***** NA
    ## 2427                  ***** NA
    ## 2428                  ***** NA
    ## 2429                  ***** NA
    ## 2430                  ***** NA
    ## 2431                  ***** NA
    ## 2432                  ***** NA
    ## 2433                  ***** NA
    ## 2434                  ***** NA
    ## 2435                  ***** NA
    ## 2436                  ***** NA
    ## 2437                  ***** NA
    ## 2438                  ***** NA
    ## 2439                  ***** NA
    ## 2440                  ***** NA
    ## 2441                  ***** NA
    ## 2442                  ***** NA
    ## 2443                  ***** NA
    ## 2444                  ***** NA
    ## 2445                  ***** NA
    ## 2446                  ***** NA
    ## 2447                  ***** NA
    ## 2448                  ***** NA
    ## 2449                  ***** NA
    ## 2450                  ***** NA
    ## 2451                  ***** NA
    ## 2452                  ***** NA
    ## 2453                  ***** NA
    ## 2454                  ***** NA
    ## 2455                  ***** NA
    ## 2456                  ***** NA
    ## 2457                  ***** NA
    ## 2458                  ***** NA
    ## 2459                  ***** NA
    ## 2460                  ***** NA
    ## 2461                  ***** NA
    ## 2462                  ***** NA
    ## 2463                  ***** NA
    ## 2464                  ***** NA
    ## 2465                  ***** NA
    ## 2466                  ***** NA
    ## 2467                  ***** NA
    ## 2468                  ***** NA
    ## 2469                  ***** NA
    ## 2470                  ***** NA
    ## 2471                  ***** NA
    ## 2472                  ***** NA
    ## 2473                  ***** NA
    ## 2474                  ***** NA
    ## 2475                  ***** NA
    ## 2476                  ***** NA
    ## 2477                  ***** NA
    ## 2478                  ***** NA
    ## 2479                  ***** NA
    ## 2480                  ***** NA
    ## 2481                  ***** NA
    ## 2482                  ***** NA
    ## 2483                  ***** NA
    ## 2484                  ***** NA
    ## 2485                  ***** NA
    ## 2486                  ***** NA
    ## 2487                  ***** NA
    ## 2488                  ***** NA
    ## 2489                  ***** NA
    ## 2490                  ***** NA
    ## 2491                  ***** NA
    ## 2492                  ***** NA
    ## 2493                  ***** NA
    ## 2494                  ***** NA
    ## 2495                  ***** NA
    ## 2496                  ***** NA
    ## 2497                  ***** NA
    ## 2498                  ***** NA
    ## 2499                  ***** NA
    ## 2500                  ***** NA
    ## 2501                  ***** NA
    ## 2502                  ***** NA
    ## 2503                  ***** NA
    ## 2504                  ***** NA
    ## 2505                  ***** NA
    ## 2506                  ***** NA
    ## 2507                  ***** NA
    ## 2508                  ***** NA
    ## 2509                  ***** NA
    ## 2510                  ***** NA
    ## 2511                  ***** NA
    ## 2512                  ***** NA
    ## 2513                  ***** NA
    ## 2514                  ***** NA
    ## 2515                  ***** NA
    ## 2516                  ***** NA
    ## 2517                  ***** NA
    ## 2518                  ***** NA
    ## 2519                  ***** NA
    ## 2520                  ***** NA
    ## 2521                  ***** NA
    ## 2522                  ***** NA
    ## 2523                  ***** NA
    ## 2524                  ***** NA
    ## 2525                  ***** NA
    ## 2526                  ***** NA
    ## 2527                  ***** NA
    ## 2528                  ***** NA
    ## 2529                  ***** NA
    ## 2530                    125 NA
    ## 2531                  ***** NA
    ## 2532                  ***** NA
    ## 2533                  ***** NA
    ## 2534                  ***** NA
    ## 2535                  ***** NA
    ## 2536                  ***** NA
    ## 2537                  ***** NA
    ## 2538                  ***** NA
    ## 2539                  ***** NA
    ## 2540                  ***** NA
    ## 2541                    119 NA
    ## 2542                  ***** NA
    ## 2543                  ***** NA
    ## 2544                  ***** NA
    ## 2545                  ***** NA
    ## 2546                  ***** NA
    ## 2547                    128 NA
    ## 2548                  ***** NA
    ## 2549                  ***** NA
    ## 2550                  ***** NA
    ## 2551                  ***** NA
    ## 2552                  ***** NA
    ## 2553                  ***** NA
    ## 2554                  ***** NA
    ## 2555                  ***** NA
    ## 2556                  ***** NA
    ## 2557                  ***** NA
    ## 2558                  ***** NA
    ## 2559                  ***** NA
    ## 2560                  ***** NA
    ## 2561                  ***** NA
    ## 2562                  ***** NA
    ## 2563                  ***** NA
    ## 2564                  ***** NA
    ## 2565                  ***** NA
    ## 2566                  ***** NA
    ## 2567                  ***** NA
    ## 2568                  ***** NA
    ## 2569                  ***** NA
    ## 2570                  ***** NA
    ## 2571                  ***** NA
    ## 2572                  ***** NA
    ## 2573                  ***** NA
    ## 2574                  ***** NA
    ## 2575                    188 NA
    ## 2576                  ***** NA
    ## 2577                    186 NA
    ## 2578                  ***** NA
    ## 2579                  ***** NA
    ## 2580                  ***** NA
    ## 2581                  ***** NA
    ## 2582                  ***** NA
    ## 2583                  ***** NA
    ## 2584                  ***** NA
    ## 2585                  ***** NA
    ## 2586                  ***** NA
    ## 2587                  ***** NA
    ## 2588                  ***** NA
    ## 2589                  ***** NA
    ## 2590                  ***** NA
    ## 2591                  ***** NA
    ## 2592                  ***** NA
    ## 2593                    193 NA
    ## 2594                  ***** NA
    ## 2595                  ***** NA
    ## 2596                  ***** NA
    ## 2597                  ***** NA
    ## 2598                  ***** NA
    ## 2599                  ***** NA
    ## 2600                  ***** NA
    ## 2601                  ***** NA
    ## 2602                    137 NA
    ## 2603                  ***** NA
    ## 2604                  ***** NA
    ## 2605                  ***** NA
    ## 2606                  ***** NA
    ## 2607                  ***** NA
    ## 2608                  ***** NA
    ## 2609                    188 NA
    ## 2610                  ***** NA
    ## 2611                    146 NA
    ## 2612                  ***** NA
    ## 2613                  ***** NA
    ## 2614                  ***** NA
    ## 2615                  ***** NA
    ## 2616                  ***** NA
    ## 2617                  ***** NA
    ## 2618                  ***** NA
    ## 2619                  ***** NA
    ## 2620                  ***** NA
    ## 2621                  ***** NA
    ## 2622                  ***** NA
    ## 2623                  ***** NA
    ## 2624                  ***** NA
    ## 2625                  ***** NA
    ## 2626                  ***** NA
    ## 2627                  ***** NA
    ## 2628                  ***** NA
    ## 2629                  ***** NA
    ## 2630                  ***** NA
    ## 2631                  ***** NA
    ## 2632                  ***** NA
    ## 2633                  ***** NA
    ## 2634                  ***** NA
    ## 2635                  ***** NA
    ## 2636                  ***** NA
    ## 2637                  ***** NA
    ## 2638                  ***** NA
    ## 2639                  ***** NA
    ## 2640                  ***** NA
    ## 2641                  ***** NA
    ## 2642                    153 NA
    ## 2643                  ***** NA
    ## 2644                  ***** NA
    ## 2645                  ***** NA
    ## 2646                  ***** NA
    ## 2647                  ***** NA
    ## 2648                  ***** NA
    ## 2649                  ***** NA
    ## 2650                  ***** NA
    ## 2651                  ***** NA
    ## 2652                  ***** NA
    ## 2653                  ***** NA
    ## 2654                  ***** NA
    ## 2655                    181 NA
    ## 2656                    118 NA
    ## 2657                  ***** NA
    ## 2658                  ***** NA
    ## 2659                     69 NA
    ## 2660                  ***** NA
    ## 2661                  ***** NA
    ## 2662                  ***** NA
    ## 2663                  ***** NA
    ## 2664                  ***** NA
    ## 2665                  ***** NA
    ## 2666                    181 NA
    ## 2667                  ***** NA
    ## 2668                  ***** NA
    ## 2669                  ***** NA
    ## 2670                  ***** NA
    ## 2671                  ***** NA
    ## 2672                  ***** NA
    ## 2673                  ***** NA
    ## 2674                  ***** NA
    ## 2675                     47 NA
    ## 2676                  ***** NA
    ## 2677                  ***** NA
    ## 2678                  ***** NA
    ## 2679                  ***** NA
    ## 2680                    193 NA
    ## 2681                  ***** NA
    ## 2682                  ***** NA
    ## 2683                  ***** NA
    ## 2684                  ***** NA
    ## 2685                  ***** NA
    ## 2686                  ***** NA
    ## 2687                  ***** NA
    ## 2688                  ***** NA
    ## 2689                  ***** NA
    ## 2690                  ***** NA
    ## 2691                  ***** NA
    ## 2692                  ***** NA
    ## 2693                  ***** NA
    ## 2694                  ***** NA
    ## 2695                  ***** NA
    ## 2696                  ***** NA
    ## 2697                    128 NA
    ## 2698                  ***** NA
    ## 2699                  ***** NA
    ## 2700                  ***** NA
    ## 2701                  ***** NA
    ## 2702                  ***** NA
    ## 2703                  ***** NA
    ## 2704                  ***** NA
    ## 2705                  ***** NA
    ## 2706                  ***** NA
    ## 2707                  ***** NA
    ## 2708                  ***** NA
    ## 2709                  ***** NA
    ## 2710                  ***** NA
    ## 2711                  ***** NA
    ## 2712                  ***** NA
    ## 2713                  ***** NA
    ## 2714                  ***** NA
    ## 2715                  ***** NA
    ## 2716                  ***** NA
    ## 2717                  ***** NA
    ## 2718                  ***** NA
    ## 2719                  ***** NA
    ## 2720                  ***** NA
    ## 2721                    125 NA
    ## 2722                  ***** NA
    ## 2723                  ***** NA
    ## 2724                  ***** NA
    ## 2725                  ***** NA
    ## 2726                  ***** NA
    ## 2727                  ***** NA
    ## 2728                  ***** NA
    ## 2729                  ***** NA
    ## 2730                  ***** NA
    ## 2731                  ***** NA
    ## 2732                  ***** NA
    ## 2733                  ***** NA
    ## 2734                  ***** NA
    ## 2735                  ***** NA
    ## 2736                  ***** NA
    ## 2737                  ***** NA
    ## 2738                  ***** NA
    ## 2739                  ***** NA
    ## 2740                    137 NA
    ## 2741                    118 NA
    ## 2742                  ***** NA
    ## 2743                  ***** NA
    ## 2744                  ***** NA
    ## 2745                  ***** NA
    ## 2746                    187 NA
    ## 2747                  ***** NA
    ## 2748                    153 NA
    ## 2749                  ***** NA
    ## 2750                  ***** NA
    ## 2751                  ***** NA
    ## 2752                  ***** NA
    ## 2753                  ***** NA
    ## 2754                  ***** NA
    ## 2755                  ***** NA
    ## 2756                  ***** NA
    ## 2757                  ***** NA
    ## 2758                  ***** NA
    ## 2759                  ***** NA
    ## 2760                  ***** NA
    ## 2761                  ***** NA
    ## 2762                  ***** NA
    ## 2763                  ***** NA
    ## 2764                  ***** NA
    ## 2765                  ***** NA
    ## 2766                  ***** NA
    ## 2767                  ***** NA
    ## 2768                  ***** NA
    ## 2769                  ***** NA
    ## 2770                  ***** NA
    ## 2771                  ***** NA
    ## 2772                  ***** NA
    ## 2773                  ***** NA
    ## 2774                  ***** NA
    ## 2775                  ***** NA
    ## 2776                  ***** NA
    ## 2777                  ***** NA
    ## 2778                  ***** NA
    ## 2779                  ***** NA
    ## 2780                  ***** NA
    ## 2781                  ***** NA
    ## 2782                  ***** NA
    ## 2783                    168 NA
    ## 2784                  ***** NA
    ## 2785                  ***** NA
    ## 2786                  ***** NA
    ## 2787                  ***** NA
    ## 2788                  ***** NA
    ## 2789                  ***** NA
    ## 2790                  ***** NA
    ## 2791                  ***** NA
    ## 2792                  ***** NA
    ## 2793                  ***** NA
    ## 2794                    168 NA
    ## 2795                  ***** NA
    ## 2796                  ***** NA
    ## 2797                  ***** NA
    ## 2798                  ***** NA
    ## 2799                  ***** NA
    ## 2800                  ***** NA
    ## 2801                  ***** NA
    ## 2802                  ***** NA
    ## 2803                  ***** NA
    ## 2804                  ***** NA
    ## 2805                  ***** NA
    ## 2806                  ***** NA
    ## 2807                  ***** NA
    ## 2808                  ***** NA
    ## 2809                  ***** NA
    ## 2810                  ***** NA
    ## 2811                  ***** NA
    ## 2812                  ***** NA
    ## 2813                  ***** NA
    ## 2814                  ***** NA
    ## 2815                  ***** NA
    ## 2816                  ***** NA
    ## 2817                  ***** NA
    ## 2818                  ***** NA
    ## 2819                  ***** NA
    ## 2820                  ***** NA
    ## 2821                  ***** NA
    ## 2822                  ***** NA
    ## 2823                  ***** NA
    ## 2824                  ***** NA
    ## 2825                  ***** NA
    ## 2826                  ***** NA
    ## 2827                  ***** NA
    ## 2828                  ***** NA
    ## 2829                  ***** NA
    ## 2830                  ***** NA
    ## 2831                  ***** NA
    ## 2832                  ***** NA
    ## 2833                  ***** NA
    ## 2834                  ***** NA
    ## 2835                  ***** NA
    ## 2836                  ***** NA
    ## 2837                  ***** NA
    ## 2838                  ***** NA
    ## 2839                  ***** NA
    ## 2840                  ***** NA
    ## 2841                  ***** NA
    ## 2842                  ***** NA
    ## 2843                  ***** NA
    ## 2844                  ***** NA
    ## 2845                  ***** NA
    ## 2846                  ***** NA
    ## 2847                  ***** NA
    ## 2848                  ***** NA
    ## 2849                  ***** NA
    ## 2850                  ***** NA
    ## 2851                  ***** NA
    ## 2852                  ***** NA
    ## 2853                  ***** NA
    ## 2854                  ***** NA
    ## 2855                  ***** NA
    ## 2856                  ***** NA
    ## 2857                  ***** NA
    ## 2858                  ***** NA
    ## 2859                  ***** NA
    ## 2860                  ***** NA
    ## 2861                  ***** NA
    ## 2862                  ***** NA
    ## 2863                  ***** NA
    ## 2864                  ***** NA
    ## 2865                  ***** NA
    ## 2866                  ***** NA
    ## 2867                  ***** NA
    ## 2868                  ***** NA
    ## 2869                  ***** NA
    ## 2870                  ***** NA
    ## 2871                  ***** NA
    ## 2872                  ***** NA
    ## 2873                  ***** NA
    ## 2874                  ***** NA
    ## 2875                  ***** NA
    ## 2876                  ***** NA
    ## 2877                  ***** NA
    ## 2878                  ***** NA
    ## 2879                  ***** NA
    ## 2880                  ***** NA
    ## 2881                  ***** NA
    ## 2882                  ***** NA
    ## 2883                  ***** NA
    ## 2884                  ***** NA
    ## 2885                  ***** NA
    ## 2886                  ***** NA
    ## 2887                  ***** NA
    ## 2888                  ***** NA
    ## 2889                  ***** NA
    ## 2890                  ***** NA
    ## 2891                  ***** NA
    ## 2892                  ***** NA
    ## 2893                  ***** NA
    ## 2894                  ***** NA
    ## 2895                  ***** NA
    ## 2896                  ***** NA
    ## 2897                  ***** NA
    ## 2898                  ***** NA
    ## 2899                  ***** NA
    ## 2900                  ***** NA
    ## 2901                  ***** NA
    ## 2902                  ***** NA
    ## 2903                  ***** NA
    ## 2904                  ***** NA
    ## 2905                  ***** NA
    ## 2906                  ***** NA
    ## 2907                  ***** NA
    ## 2908                  ***** NA
    ## 2909                  ***** NA
    ## 2910                  ***** NA
    ## 2911                  ***** NA
    ## 2912                  ***** NA
    ## 2913                  ***** NA
    ## 2914                  ***** NA
    ## 2915                  ***** NA
    ## 2916                  ***** NA
    ## 2917                  ***** NA
    ## 2918                  ***** NA
    ## 2919                  ***** NA
    ## 2920                  ***** NA
    ## 2921                  ***** NA
    ## 2922                  ***** NA
    ## 2923                  ***** NA
    ## 2924                  ***** NA
    ## 2925                  ***** NA
    ## 2926                  ***** NA
    ## 2927                  ***** NA
    ## 2928                  ***** NA
    ## 2929                  ***** NA
    ## 2930                  ***** NA
    ## 2931                  ***** NA
    ## 2932                  ***** NA
    ## 2933                  ***** NA
    ## 2934                  ***** NA
    ## 2935                  ***** NA
    ## 2936                  ***** NA
    ## 2937                  ***** NA
    ## 2938                  ***** NA
    ## 2939                  ***** NA
    ## 2940                  ***** NA
    ## 2941                  ***** NA
    ## 2942                  ***** NA
    ## 2943                  ***** NA
    ## 2944                  ***** NA
    ## 2945                  ***** NA
    ## 2946                  ***** NA
    ## 2947                  ***** NA
    ## 2948                  ***** NA
    ## 2949                  ***** NA
    ## 2950                  ***** NA
    ## 2951                  ***** NA
    ## 2952                  ***** NA
    ## 2953                  ***** NA
    ## 2954                  ***** NA
    ## 2955                  ***** NA
    ## 2956                  ***** NA
    ## 2957                  ***** NA
    ## 2958                  ***** NA
    ## 2959                  ***** NA
    ## 2960                  ***** NA
    ## 2961                  ***** NA
    ## 2962                  ***** NA
    ## 2963                  ***** NA
    ## 2964                  ***** NA
    ## 2965                  ***** NA
    ## 2966                  ***** NA
    ## 2967                  ***** NA
    ## 2968                  ***** NA
    ## 2969                  ***** NA
    ## 2970                  ***** NA
    ## 2971                  ***** NA
    ## 2972                  ***** NA
    ## 2973                  ***** NA
    ## 2974                  ***** NA
    ## 2975                  ***** NA
    ## 2976                  ***** NA
    ## 2977                  ***** NA
    ## 2978                  ***** NA
    ## 2979                  ***** NA
    ## 2980                  ***** NA
    ## 2981                  ***** NA
    ## 2982                  ***** NA
    ## 2983                  ***** NA
    ## 2984                  ***** NA
    ## 2985                  ***** NA
    ## 2986                  ***** NA
    ## 2987                  ***** NA
    ## 2988                  ***** NA
    ## 2989                  ***** NA
    ## 2990                  ***** NA
    ## 2991                  ***** NA
    ## 2992                  ***** NA
    ## 2993                  ***** NA
    ## 2994                  ***** NA
    ## 2995                  ***** NA
    ## 2996                  ***** NA
    ## 2997                  ***** NA
    ## 2998                  ***** NA
    ## 2999                  ***** NA
    ## 3000                  ***** NA
    ## 3001                  ***** NA
    ## 3002                  ***** NA
    ## 3003                  ***** NA
    ## 3004                  ***** NA
    ## 3005                  ***** NA
    ## 3006                  ***** NA
    ## 3007                  ***** NA
    ## 3008                  ***** NA
    ## 3009                  ***** NA
    ## 3010                  ***** NA
    ## 3011                  ***** NA
    ## 3012                  ***** NA
    ## 3013                  ***** NA
    ## 3014                  ***** NA
    ## 3015                  ***** NA
    ## 3016                  ***** NA
    ## 3017                  ***** NA
    ## 3018                  ***** NA
    ## 3019                  ***** NA
    ## 3020                  ***** NA
    ## 3021                  ***** NA
    ## 3022                  ***** NA
    ## 3023                  ***** NA
    ## 3024                  ***** NA
    ## 3025                  ***** NA
    ## 3026                  ***** NA
    ## 3027                  ***** NA
    ## 3028                  ***** NA
    ## 3029                  ***** NA
    ## 3030                  ***** NA
    ## 3031                  ***** NA
    ## 3032                  ***** NA
    ## 3033                  ***** NA
    ## 3034                  ***** NA
    ## 3035                  ***** NA
    ## 3036                  ***** NA
    ## 3037                  ***** NA
    ## 3038                  ***** NA
    ## 3039                  ***** NA
    ## 3040                  ***** NA
    ## 3041                  ***** NA
    ## 3042                  ***** NA
    ## 3043                  ***** NA
    ## 3044                  ***** NA
    ## 3045                  ***** NA
    ## 3046                  ***** NA
    ## 3047                  ***** NA
    ## 3048                  ***** NA
    ## 3049                  ***** NA
    ## 3050                  ***** NA
    ## 3051                  ***** NA
    ## 3052                  ***** NA
    ## 3053                  ***** NA
    ## 3054                  ***** NA
    ## 3055                  ***** NA
    ## 3056                  ***** NA
    ## 3057                  ***** NA
    ## 3058                  ***** NA
    ## 3059                  ***** NA
    ## 3060                  ***** NA
    ## 3061                  ***** NA
    ## 3062                  ***** NA
    ## 3063                  ***** NA
    ## 3064                  ***** NA
    ## 3065                  ***** NA
    ## 3066                  ***** NA
    ## 3067                  ***** NA
    ## 3068                  ***** NA
    ## 3069                  ***** NA
    ## 3070                  ***** NA
    ## 3071                  ***** NA
    ## 3072                  ***** NA
    ## 3073                  ***** NA
    ## 3074                  ***** NA
    ## 3075                  ***** NA
    ## 3076                  ***** NA
    ## 3077                  ***** NA
    ## 3078                  ***** NA
    ## 3079                  ***** NA
    ## 3080                  ***** NA
    ## 3081                  ***** NA
    ## 3082                  ***** NA
    ## 3083                  ***** NA
    ## 3084                  ***** NA
    ## 3085                  ***** NA
    ## 3086                  ***** NA
    ## 3087                  ***** NA
    ## 3088                  ***** NA
    ## 3089                  ***** NA
    ## 3090                  ***** NA
    ## 3091                  ***** NA
    ## 3092                  ***** NA
    ## 3093                  ***** NA
    ## 3094                  ***** NA
    ## 3095                  ***** NA
    ## 3096                  ***** NA
    ## 3097                  ***** NA
    ## 3098                  ***** NA
    ## 3099                  ***** NA
    ## 3100                  ***** NA
    ## 3101                  ***** NA
    ## 3102                  ***** NA
    ## 3103                  ***** NA
    ## 3104                  ***** NA
    ## 3105                  ***** NA
    ## 3106                  ***** NA
    ## 3107                  ***** NA
    ## 3108                  ***** NA
    ## 3109                  ***** NA
    ## 3110                  ***** NA
    ## 3111                  ***** NA
    ## 3112                  ***** NA
    ## 3113                  ***** NA
    ## 3114                  ***** NA
    ## 3115                  ***** NA
    ## 3116                  ***** NA
    ## 3117                  ***** NA
    ## 3118                  ***** NA
    ## 3119                  ***** NA
    ## 3120                  ***** NA
    ## 3121                  ***** NA
    ## 3122                  ***** NA
    ## 3123                  ***** NA
    ## 3124                  ***** NA
    ## 3125                  ***** NA
    ## 3126                  ***** NA
    ## 3127                  ***** NA
    ## 3128                  ***** NA
    ## 3129                  ***** NA
    ## 3130                  ***** NA
    ## 3131                  ***** NA
    ## 3132                  ***** NA
    ## 3133                  ***** NA
    ## 3134                  ***** NA
    ## 3135                  ***** NA
    ## 3136                  ***** NA
    ## 3137                  ***** NA
    ## 3138                  ***** NA
    ## 3139                  ***** NA
    ## 3140                  ***** NA
    ## 3141                  ***** NA
    ## 3142                  ***** NA
    ## 3143                  ***** NA
    ## 3144                  ***** NA
    ## 3145                  ***** NA
    ## 3146                  ***** NA
    ## 3147                  ***** NA
    ## 3148                  ***** NA
    ## 3149                  ***** NA
    ## 3150                  ***** NA
    ## 3151                  ***** NA
    ## 3152                  ***** NA
    ## 3153                  ***** NA
    ## 3154                  ***** NA
    ## 3155                  ***** NA
    ## 3156                  ***** NA
    ## 3157                  ***** NA
    ## 3158                  ***** NA
    ## 3159                  ***** NA
    ## 3160                  ***** NA
    ## 3161                  ***** NA
    ## 3162                  ***** NA
    ## 3163                  ***** NA
    ## 3164                  ***** NA
    ## 3165                  ***** NA
    ## 3166                  ***** NA
    ## 3167                  ***** NA
    ## 3168                    218 NA
    ## 3169                  ***** NA
    ## 3170                  ***** NA
    ## 3171                  ***** NA
    ## 3172                  ***** NA
    ## 3173                  ***** NA
    ## 3174                  ***** NA
    ## 3175                  ***** NA
    ## 3176                  ***** NA
    ## 3177                  ***** NA
    ## 3178                  ***** NA
    ## 3179                  ***** NA
    ## 3180                  ***** NA
    ## 3181                  ***** NA
    ## 3182                  ***** NA
    ## 3183                  ***** NA
    ## 3184                  ***** NA
    ## 3185                  ***** NA
    ## 3186                  ***** NA
    ## 3187                  ***** NA
    ## 3188                  ***** NA
    ## 3189                  ***** NA
    ## 3190                  ***** NA
    ## 3191                    218 NA
    ## 3192                  ***** NA
    ## 3193                  ***** NA
    ## 3194                  ***** NA
    ## 3195                  ***** NA
    ## 3196                  ***** NA
    ## 3197                  ***** NA
    ## 3198                  ***** NA
    ## 3199                  ***** NA
    ## 3200                  ***** NA
    ## 3201                  ***** NA
    ## 3202                  ***** NA
    ## 3203                  ***** NA
    ## 3204                  ***** NA
    ## 3205                  ***** NA
    ## 3206                  ***** NA
    ## 3207                  ***** NA
    ## 3208                  ***** NA
    ## 3209                  ***** NA
    ## 3210                  ***** NA
    ## 3211                  ***** NA
    ## 3212                  ***** NA
    ## 3213                  ***** NA
    ## 3214                  ***** NA
    ## 3215                  ***** NA
    ## 3216                  ***** NA
    ## 3217                  ***** NA
    ## 3218                  ***** NA
    ## 3219                  ***** NA
    ## 3220                  ***** NA
    ## 3221                  ***** NA

*Note*: You can find information on 1-year, 3-year, and 5-year estimates
[here](https://www.census.gov/programs-surveys/acs/guidance/estimates.html).
The punchline is that 5-year estimates are more reliable but less
current.

## Automated Download of NYT Data

<!-- ------------------------- -->

ACS 5-year estimates don’t change all that often, but the COVID-19 data
are changing rapidly. To that end, it would be nice to be able to
*programmatically* download the most recent data for analysis; that way
we can update our analysis whenever we want simply by re-running our
notebook. This next problem will have you set up such a pipeline.

The New York Times is publishing up-to-date data on COVID-19 on
[GitHub](https://github.com/nytimes/covid-19-data).

### **q2** Visit the NYT [GitHub](https://github.com/nytimes/covid-19-data) repo and find the URL for the **raw** US County-level data. Assign that URL as a string to the variable below.

``` r
## TASK: Find the URL for the NYT covid-19 county-level data
url_counties <- "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties.csv"
```

Once you have the url, the following code will download a local copy of
the data, then load the data into R.

``` r
## NOTE: No need to change this; just execute
## Set the filename of the data to download
filename_nyt <- "./data/nyt_counties.csv"

## Download the data locally
curl::curl_download(
        url_counties,
        destfile = filename_nyt
      )

## Loads the downloaded csv
df_covid <- read_csv(filename_nyt)
```

    ## Rows: 2502832 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): county, state, fips
    ## dbl  (2): cases, deaths
    ## date (1): date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

You can now re-run the chunk above (or the entire notebook) to pull the
most recent version of the data. Thus you can periodically re-run this
notebook to check in on the pandemic as it evolves.

*Note*: You should feel free to copy-paste the code above for your own
future projects!

# Join the Data

<!-- -------------------------------------------------- -->

To get a sense of our task, let’s take a glimpse at our two data
sources.

``` r
## NOTE: No need to change this; just execute
df_pop 
```

    ##              GEO_ID                                       NAME     B01003_001E
    ## 1         Geography                       Geographic Area Name Estimate!!Total
    ## 2    0500000US01001                    Autauga County, Alabama           55200
    ## 3    0500000US01003                    Baldwin County, Alabama          208107
    ## 4    0500000US01005                    Barbour County, Alabama           25782
    ## 5    0500000US01007                       Bibb County, Alabama           22527
    ## 6    0500000US01009                     Blount County, Alabama           57645
    ## 7    0500000US01011                    Bullock County, Alabama           10352
    ## 8    0500000US01013                     Butler County, Alabama           20025
    ## 9    0500000US01015                    Calhoun County, Alabama          115098
    ## 10   0500000US01017                   Chambers County, Alabama           33826
    ## 11   0500000US01019                   Cherokee County, Alabama           25853
    ## 12   0500000US01021                    Chilton County, Alabama           43930
    ## 13   0500000US01023                    Choctaw County, Alabama           13075
    ## 14   0500000US01025                     Clarke County, Alabama           24387
    ## 15   0500000US01027                       Clay County, Alabama           13378
    ## 16   0500000US01029                   Cleburne County, Alabama           14938
    ## 17   0500000US01031                     Coffee County, Alabama           51288
    ## 18   0500000US01033                    Colbert County, Alabama           54495
    ## 19   0500000US01035                    Conecuh County, Alabama           12514
    ## 20   0500000US01037                      Coosa County, Alabama           10855
    ## 21   0500000US01039                  Covington County, Alabama           37351
    ## 22   0500000US01041                   Crenshaw County, Alabama           13865
    ## 23   0500000US01043                    Cullman County, Alabama           82313
    ## 24   0500000US01045                       Dale County, Alabama           49255
    ## 25   0500000US01047                     Dallas County, Alabama           40029
    ## 26   0500000US01049                     DeKalb County, Alabama           71200
    ## 27   0500000US01051                     Elmore County, Alabama           81212
    ## 28   0500000US01053                   Escambia County, Alabama           37328
    ## 29   0500000US01055                     Etowah County, Alabama          102939
    ## 30   0500000US01057                    Fayette County, Alabama           16585
    ## 31   0500000US01059                   Franklin County, Alabama           31542
    ## 32   0500000US01061                     Geneva County, Alabama           26491
    ## 33   0500000US01063                     Greene County, Alabama            8426
    ## 34   0500000US01065                       Hale County, Alabama           14887
    ## 35   0500000US01067                      Henry County, Alabama           17124
    ## 36   0500000US01069                    Houston County, Alabama          104352
    ## 37   0500000US01071                    Jackson County, Alabama           52094
    ## 38   0500000US01073                  Jefferson County, Alabama          659892
    ## 39   0500000US01075                      Lamar County, Alabama           13933
    ## 40   0500000US01077                 Lauderdale County, Alabama           92585
    ## 41   0500000US01079                   Lawrence County, Alabama           33171
    ## 42   0500000US01081                        Lee County, Alabama          159287
    ## 43   0500000US01083                  Limestone County, Alabama           93052
    ## 44   0500000US01085                    Lowndes County, Alabama           10236
    ## 45   0500000US01087                      Macon County, Alabama           19054
    ## 46   0500000US01089                    Madison County, Alabama          357560
    ## 47   0500000US01091                    Marengo County, Alabama           19538
    ## 48   0500000US01093                     Marion County, Alabama           29965
    ## 49   0500000US01095                   Marshall County, Alabama           95145
    ## 50   0500000US01097                     Mobile County, Alabama          414659
    ## 51   0500000US01099                     Monroe County, Alabama           21512
    ## 52   0500000US01101                 Montgomery County, Alabama          226941
    ## 53   0500000US01103                     Morgan County, Alabama          119122
    ## 54   0500000US01105                      Perry County, Alabama            9486
    ## 55   0500000US01107                    Pickens County, Alabama           20298
    ## 56   0500000US01109                       Pike County, Alabama           33403
    ## 57   0500000US01111                   Randolph County, Alabama           22574
    ## 58   0500000US01113                    Russell County, Alabama           58213
    ## 59   0500000US01115                  St. Clair County, Alabama           87306
    ## 60   0500000US01117                     Shelby County, Alabama          211261
    ## 61   0500000US01119                     Sumter County, Alabama           12985
    ## 62   0500000US01121                  Talladega County, Alabama           80565
    ## 63   0500000US01123                 Tallapoosa County, Alabama           40636
    ## 64   0500000US01125                 Tuscaloosa County, Alabama          206213
    ## 65   0500000US01127                     Walker County, Alabama           64493
    ## 66   0500000US01129                 Washington County, Alabama           16643
    ## 67   0500000US01131                     Wilcox County, Alabama           10809
    ## 68   0500000US01133                    Winston County, Alabama           23875
    ## 69   0500000US02013             Aleutians East Borough, Alaska            3425
    ## 70   0500000US02016         Aleutians West Census Area, Alaska            5750
    ## 71   0500000US02020             Anchorage Municipality, Alaska          296112
    ## 72   0500000US02050                 Bethel Census Area, Alaska           18040
    ## 73   0500000US02060                Bristol Bay Borough, Alaska             890
    ## 74   0500000US02068                     Denali Borough, Alaska            2232
    ## 75   0500000US02070             Dillingham Census Area, Alaska            4975
    ## 76   0500000US02090       Fairbanks North Star Borough, Alaska           99653
    ## 77   0500000US02100                     Haines Borough, Alaska            2518
    ## 78   0500000US02105          Hoonah-Angoon Census Area, Alaska            2132
    ## 79   0500000US02110            Juneau City and Borough, Alaska           32330
    ## 80   0500000US02122            Kenai Peninsula Borough, Alaska           58220
    ## 81   0500000US02130          Ketchikan Gateway Borough, Alaska           13804
    ## 82   0500000US02150              Kodiak Island Borough, Alaska           13649
    ## 83   0500000US02158               Kusilvak Census Area, Alaska            8198
    ## 84   0500000US02164         Lake and Peninsula Borough, Alaska            1375
    ## 85   0500000US02170          Matanuska-Susitna Borough, Alaska          103464
    ## 86   0500000US02180                   Nome Census Area, Alaska            9925
    ## 87   0500000US02185                North Slope Borough, Alaska            9797
    ## 88   0500000US02188           Northwest Arctic Borough, Alaska            7734
    ## 89   0500000US02195                 Petersburg Borough, Alaska            3255
    ## 90   0500000US02198  Prince of Wales-Hyder Census Area, Alaska            6474
    ## 91   0500000US02220             Sitka City and Borough, Alaska            8738
    ## 92   0500000US02230               Skagway Municipality, Alaska            1061
    ## 93   0500000US02240    Southeast Fairbanks Census Area, Alaska            6876
    ## 94   0500000US02261         Valdez-Cordova Census Area, Alaska            9301
    ## 95   0500000US02275          Wrangell City and Borough, Alaska            2484
    ## 96   0500000US02282           Yakutat City and Borough, Alaska             689
    ## 97   0500000US02290          Yukon-Koyukuk Census Area, Alaska            5415
    ## 98   0500000US04001                     Apache County, Arizona           71522
    ## 99   0500000US04003                    Cochise County, Arizona          126279
    ## 100  0500000US04005                   Coconino County, Arizona          140217
    ## 101  0500000US04007                       Gila County, Arizona           53400
    ## 102  0500000US04009                     Graham County, Arizona           37879
    ## 103  0500000US04011                   Greenlee County, Arizona            9504
    ## 104  0500000US04012                     La Paz County, Arizona           20701
    ## 105  0500000US04013                   Maricopa County, Arizona         4253913
    ## 106  0500000US04015                     Mohave County, Arizona          206064
    ## 107  0500000US04017                     Navajo County, Arizona          108705
    ## 108  0500000US04019                       Pima County, Arizona         1019722
    ## 109  0500000US04021                      Pinal County, Arizona          419721
    ## 110  0500000US04023                 Santa Cruz County, Arizona           46584
    ## 111  0500000US04025                    Yavapai County, Arizona          224645
    ## 112  0500000US04027                       Yuma County, Arizona          207829
    ## 113  0500000US05001                  Arkansas County, Arkansas           18124
    ## 114  0500000US05003                    Ashley County, Arkansas           20537
    ## 115  0500000US05005                    Baxter County, Arkansas           41219
    ## 116  0500000US05007                    Benton County, Arkansas          258980
    ## 117  0500000US05009                     Boone County, Arkansas           37288
    ## 118  0500000US05011                   Bradley County, Arkansas           10948
    ## 119  0500000US05013                   Calhoun County, Arkansas            5202
    ## 120  0500000US05015                   Carroll County, Arkansas           27887
    ## 121  0500000US05017                    Chicot County, Arkansas           10826
    ## 122  0500000US05019                     Clark County, Arkansas           22385
    ## 123  0500000US05021                      Clay County, Arkansas           15061
    ## 124  0500000US05023                  Cleburne County, Arkansas           25230
    ## 125  0500000US05025                 Cleveland County, Arkansas            8226
    ## 126  0500000US05027                  Columbia County, Arkansas           23892
    ## 127  0500000US05029                    Conway County, Arkansas           20906
    ## 128  0500000US05031                 Craighead County, Arkansas          105701
    ## 129  0500000US05033                  Crawford County, Arkansas           62472
    ## 130  0500000US05035                Crittenden County, Arkansas           49013
    ## 131  0500000US05037                     Cross County, Arkansas           16998
    ## 132  0500000US05039                    Dallas County, Arkansas            7432
    ## 133  0500000US05041                     Desha County, Arkansas           11887
    ## 134  0500000US05043                      Drew County, Arkansas           18502
    ## 135  0500000US05045                  Faulkner County, Arkansas          122416
    ## 136  0500000US05047                  Franklin County, Arkansas           17780
    ## 137  0500000US05049                    Fulton County, Arkansas           12139
    ## 138  0500000US05051                   Garland County, Arkansas           98296
    ## 139  0500000US05053                     Grant County, Arkansas           18086
    ## 140  0500000US05055                    Greene County, Arkansas           44623
    ## 141  0500000US05057                 Hempstead County, Arkansas           22018
    ## 142  0500000US05059                Hot Spring County, Arkansas           33520
    ## 143  0500000US05061                    Howard County, Arkansas           13389
    ## 144  0500000US05063              Independence County, Arkansas           37264
    ## 145  0500000US05065                     Izard County, Arkansas           13559
    ## 146  0500000US05067                   Jackson County, Arkansas           17225
    ## 147  0500000US05069                 Jefferson County, Arkansas           70424
    ## 148  0500000US05071                   Johnson County, Arkansas           26291
    ## 149  0500000US05073                 Lafayette County, Arkansas            6915
    ## 150  0500000US05075                  Lawrence County, Arkansas           16669
    ## 151  0500000US05077                       Lee County, Arkansas            9398
    ## 152  0500000US05079                   Lincoln County, Arkansas           13695
    ## 153  0500000US05081              Little River County, Arkansas           12417
    ## 154  0500000US05083                     Logan County, Arkansas           21757
    ## 155  0500000US05085                    Lonoke County, Arkansas           72206
    ## 156  0500000US05087                   Madison County, Arkansas           16076
    ## 157  0500000US05089                    Marion County, Arkansas           16438
    ## 158  0500000US05091                    Miller County, Arkansas           43759
    ## 159  0500000US05093               Mississippi County, Arkansas           42831
    ## 160  0500000US05095                    Monroe County, Arkansas            7249
    ## 161  0500000US05097                Montgomery County, Arkansas            8993
    ## 162  0500000US05099                    Nevada County, Arkansas            8440
    ## 163  0500000US05101                    Newton County, Arkansas            7848
    ## 164  0500000US05103                  Ouachita County, Arkansas           24106
    ## 165  0500000US05105                     Perry County, Arkansas           10322
    ## 166  0500000US05107                  Phillips County, Arkansas           19034
    ## 167  0500000US05109                      Pike County, Arkansas           10808
    ## 168  0500000US05111                  Poinsett County, Arkansas           24054
    ## 169  0500000US05113                      Polk County, Arkansas           20163
    ## 170  0500000US05115                      Pope County, Arkansas           63644
    ## 171  0500000US05117                   Prairie County, Arkansas            8244
    ## 172  0500000US05119                   Pulaski County, Arkansas          393463
    ## 173  0500000US05121                  Randolph County, Arkansas           17603
    ## 174  0500000US05123               St. Francis County, Arkansas           26294
    ## 175  0500000US05125                    Saline County, Arkansas          118009
    ## 176  0500000US05127                     Scott County, Arkansas           10442
    ## 177  0500000US05129                    Searcy County, Arkansas            7923
    ## 178  0500000US05131                 Sebastian County, Arkansas          127461
    ## 179  0500000US05133                    Sevier County, Arkansas           17193
    ## 180  0500000US05135                     Sharp County, Arkansas           17043
    ## 181  0500000US05137                     Stone County, Arkansas           12446
    ## 182  0500000US05139                     Union County, Arkansas           39732
    ## 183  0500000US05141                 Van Buren County, Arkansas           16684
    ## 184  0500000US05143                Washington County, Arkansas          228529
    ## 185  0500000US05145                     White County, Arkansas           78804
    ## 186  0500000US05147                  Woodruff County, Arkansas            6660
    ## 187  0500000US05149                      Yell County, Arkansas           21573
    ## 188  0500000US06001                 Alameda County, California         1643700
    ## 189  0500000US06003                  Alpine County, California            1146
    ## 190  0500000US06005                  Amador County, California           37829
    ## 191  0500000US06007                   Butte County, California          227075
    ## 192  0500000US06009               Calaveras County, California           45235
    ## 193  0500000US06011                  Colusa County, California           21464
    ## 194  0500000US06013            Contra Costa County, California         1133247
    ## 195  0500000US06015               Del Norte County, California           27424
    ## 196  0500000US06017               El Dorado County, California          186661
    ## 197  0500000US06019                  Fresno County, California          978130
    ## 198  0500000US06021                   Glenn County, California           27897
    ## 199  0500000US06023                Humboldt County, California          135768
    ## 200  0500000US06025                Imperial County, California          180216
    ## 201  0500000US06027                    Inyo County, California           18085
    ## 202  0500000US06029                    Kern County, California          883053
    ## 203  0500000US06031                   Kings County, California          150075
    ## 204  0500000US06033                    Lake County, California           64148
    ## 205  0500000US06035                  Lassen County, California           31185
    ## 206  0500000US06037             Los Angeles County, California        10098052
    ## 207  0500000US06039                  Madera County, California          155013
    ## 208  0500000US06041                   Marin County, California          260295
    ## 209  0500000US06043                Mariposa County, California           17540
    ## 210  0500000US06045               Mendocino County, California           87422
    ## 211  0500000US06047                  Merced County, California          269075
    ## 212  0500000US06049                   Modoc County, California            8938
    ## 213  0500000US06051                    Mono County, California           14174
    ## 214  0500000US06053                Monterey County, California          433212
    ## 215  0500000US06055                    Napa County, California          140530
    ## 216  0500000US06057                  Nevada County, California           99092
    ## 217  0500000US06059                  Orange County, California         3164182
    ## 218  0500000US06061                  Placer County, California          380077
    ## 219  0500000US06063                  Plumas County, California           18699
    ## 220  0500000US06065               Riverside County, California         2383286
    ## 221  0500000US06067              Sacramento County, California         1510023
    ## 222  0500000US06069              San Benito County, California           59416
    ## 223  0500000US06071          San Bernardino County, California         2135413
    ## 224  0500000US06073               San Diego County, California         3302833
    ## 225  0500000US06075           San Francisco County, California          870044
    ## 226  0500000US06077             San Joaquin County, California          732212
    ## 227  0500000US06079         San Luis Obispo County, California          281455
    ## 228  0500000US06081               San Mateo County, California          765935
    ## 229  0500000US06083           Santa Barbara County, California          443738
    ## 230  0500000US06085             Santa Clara County, California         1922200
    ## 231  0500000US06087              Santa Cruz County, California          273765
    ## 232  0500000US06089                  Shasta County, California          179085
    ## 233  0500000US06091                  Sierra County, California            2930
    ## 234  0500000US06093                Siskiyou County, California           43540
    ## 235  0500000US06095                  Solano County, California          438530
    ## 236  0500000US06097                  Sonoma County, California          501317
    ## 237  0500000US06099              Stanislaus County, California          539301
    ## 238  0500000US06101                  Sutter County, California           95872
    ## 239  0500000US06103                  Tehama County, California           63373
    ## 240  0500000US06105                 Trinity County, California           12862
    ## 241  0500000US06107                  Tulare County, California          460477
    ## 242  0500000US06109                Tuolumne County, California           53932
    ## 243  0500000US06111                 Ventura County, California          848112
    ## 244  0500000US06113                    Yolo County, California          214977
    ## 245  0500000US06115                    Yuba County, California           75493
    ## 246  0500000US08001                     Adams County, Colorado          497115
    ## 247  0500000US08003                   Alamosa County, Colorado           16444
    ## 248  0500000US08005                  Arapahoe County, Colorado          636671
    ## 249  0500000US08007                 Archuleta County, Colorado           12908
    ## 250  0500000US08009                      Baca County, Colorado            3563
    ## 251  0500000US08011                      Bent County, Colorado            5809
    ## 252  0500000US08013                   Boulder County, Colorado          321030
    ## 253  0500000US08014                Broomfield County, Colorado           66120
    ## 254  0500000US08015                   Chaffee County, Colorado           19164
    ## 255  0500000US08017                  Cheyenne County, Colorado            2039
    ## 256  0500000US08019               Clear Creek County, Colorado            9379
    ## 257  0500000US08021                   Conejos County, Colorado            8142
    ## 258  0500000US08023                  Costilla County, Colorado            3687
    ## 259  0500000US08025                   Crowley County, Colorado            5630
    ## 260  0500000US08027                    Custer County, Colorado            4640
    ## 261  0500000US08029                     Delta County, Colorado           30346
    ## 262  0500000US08031                    Denver County, Colorado          693417
    ## 263  0500000US08033                   Dolores County, Colorado            1841
    ## 264  0500000US08035                   Douglas County, Colorado          328614
    ## 265  0500000US08037                     Eagle County, Colorado           54357
    ## 266  0500000US08039                    Elbert County, Colorado           25162
    ## 267  0500000US08041                   El Paso County, Colorado          688153
    ## 268  0500000US08043                   Fremont County, Colorado           47002
    ## 269  0500000US08045                  Garfield County, Colorado           58538
    ## 270  0500000US08047                    Gilpin County, Colorado            5924
    ## 271  0500000US08049                     Grand County, Colorado           15066
    ## 272  0500000US08051                  Gunnison County, Colorado           16537
    ## 273  0500000US08053                  Hinsdale County, Colorado             878
    ## 274  0500000US08055                  Huerfano County, Colorado            6583
    ## 275  0500000US08057                   Jackson County, Colorado            1296
    ## 276  0500000US08059                 Jefferson County, Colorado          570427
    ## 277  0500000US08061                     Kiowa County, Colorado            1449
    ## 278  0500000US08063                Kit Carson County, Colorado            7635
    ## 279  0500000US08065                      Lake County, Colorado            7585
    ## 280  0500000US08067                  La Plata County, Colorado           55101
    ## 281  0500000US08069                   Larimer County, Colorado          338161
    ## 282  0500000US08071                Las Animas County, Colorado           14179
    ## 283  0500000US08073                   Lincoln County, Colorado            5548
    ## 284  0500000US08075                     Logan County, Colorado           21689
    ## 285  0500000US08077                      Mesa County, Colorado          149998
    ## 286  0500000US08079                   Mineral County, Colorado             823
    ## 287  0500000US08081                    Moffat County, Colorado           13060
    ## 288  0500000US08083                 Montezuma County, Colorado           25909
    ## 289  0500000US08085                  Montrose County, Colorado           41268
    ## 290  0500000US08087                    Morgan County, Colorado           28257
    ## 291  0500000US08089                     Otero County, Colorado           18325
    ## 292  0500000US08091                     Ouray County, Colorado            4722
    ## 293  0500000US08093                      Park County, Colorado           17392
    ## 294  0500000US08095                  Phillips County, Colorado            4318
    ## 295  0500000US08097                    Pitkin County, Colorado           17909
    ## 296  0500000US08099                   Prowers County, Colorado           12052
    ## 297  0500000US08101                    Pueblo County, Colorado          164685
    ## 298  0500000US08103                Rio Blanco County, Colorado            6465
    ## 299  0500000US08105                Rio Grande County, Colorado           11351
    ## 300  0500000US08107                     Routt County, Colorado           24874
    ## 301  0500000US08109                  Saguache County, Colorado            6468
    ## 302  0500000US08111                  San Juan County, Colorado             544
    ## 303  0500000US08113                San Miguel County, Colorado            7968
    ## 304  0500000US08115                  Sedgwick County, Colorado            2350
    ## 305  0500000US08117                    Summit County, Colorado           30429
    ## 306  0500000US08119                    Teller County, Colorado           24113
    ## 307  0500000US08121                Washington County, Colorado            4840
    ## 308  0500000US08123                      Weld County, Colorado          295123
    ## 309  0500000US08125                      Yuma County, Colorado           10069
    ## 310  0500000US09001              Fairfield County, Connecticut          944348
    ## 311  0500000US09003               Hartford County, Connecticut          894730
    ## 312  0500000US09005             Litchfield County, Connecticut          183031
    ## 313  0500000US09007              Middlesex County, Connecticut          163368
    ## 314  0500000US09009              New Haven County, Connecticut          859339
    ## 315  0500000US09011             New London County, Connecticut          268881
    ## 316  0500000US09013                Tolland County, Connecticut          151269
    ## 317  0500000US09015                Windham County, Connecticut          116538
    ## 318  0500000US10001                      Kent County, Delaware          174822
    ## 319  0500000US10003                New Castle County, Delaware          555133
    ## 320  0500000US10005                    Sussex County, Delaware          219540
    ## 321  0500000US11001 District of Columbia, District of Columbia          684498
    ## 322  0500000US12001                    Alachua County, Florida          263148
    ## 323  0500000US12003                      Baker County, Florida           27785
    ## 324  0500000US12005                        Bay County, Florida          182482
    ## 325  0500000US12007                   Bradford County, Florida           26979
    ## 326  0500000US12009                    Brevard County, Florida          576808
    ## 327  0500000US12011                    Broward County, Florida         1909151
    ## 328  0500000US12013                    Calhoun County, Florida           14444
    ## 329  0500000US12015                  Charlotte County, Florida          176954
    ## 330  0500000US12017                     Citrus County, Florida          143087
    ## 331  0500000US12019                       Clay County, Florida          207291
    ## 332  0500000US12021                    Collier County, Florida          363922
    ## 333  0500000US12023                   Columbia County, Florida           69105
    ## 334  0500000US12027                     DeSoto County, Florida           36399
    ## 335  0500000US12029                      Dixie County, Florida           16437
    ## 336  0500000US12031                      Duval County, Florida          924229
    ## 337  0500000US12033                   Escambia County, Florida          311522
    ## 338  0500000US12035                    Flagler County, Florida          107139
    ## 339  0500000US12037                   Franklin County, Florida           11736
    ## 340  0500000US12039                    Gadsden County, Florida           46017
    ## 341  0500000US12041                  Gilchrist County, Florida           17615
    ## 342  0500000US12043                     Glades County, Florida           13363
    ## 343  0500000US12045                       Gulf County, Florida           16055
    ## 344  0500000US12047                   Hamilton County, Florida           14269
    ## 345  0500000US12049                     Hardee County, Florida           27228
    ## 346  0500000US12051                     Hendry County, Florida           40127
    ## 347  0500000US12053                   Hernando County, Florida          182696
    ## 348  0500000US12055                  Highlands County, Florida          102101
    ## 349  0500000US12057               Hillsborough County, Florida         1378883
    ## 350  0500000US12059                     Holmes County, Florida           19430
    ## 351  0500000US12061               Indian River County, Florida          150984
    ## 352  0500000US12063                    Jackson County, Florida           48472
    ## 353  0500000US12065                  Jefferson County, Florida           14105
    ## 354  0500000US12067                  Lafayette County, Florida            8744
    ## 355  0500000US12069                       Lake County, Florida          335362
    ## 356  0500000US12071                        Lee County, Florida          718679
    ## 357  0500000US12073                       Leon County, Florida          288102
    ## 358  0500000US12075                       Levy County, Florida           39961
    ## 359  0500000US12077                    Liberty County, Florida            8365
    ## 360  0500000US12079                    Madison County, Florida           18474
    ## 361  0500000US12081                    Manatee County, Florida          373853
    ## 362  0500000US12083                     Marion County, Florida          348371
    ## 363  0500000US12085                     Martin County, Florida          157581
    ## 364  0500000US12086                 Miami-Dade County, Florida         2715516
    ## 365  0500000US12087                     Monroe County, Florida           76325
    ## 366  0500000US12089                     Nassau County, Florida           80578
    ## 367  0500000US12091                   Okaloosa County, Florida          200737
    ## 368  0500000US12093                 Okeechobee County, Florida           40572
    ## 369  0500000US12095                     Orange County, Florida         1321194
    ## 370  0500000US12097                    Osceola County, Florida          338619
    ## 371  0500000US12099                 Palm Beach County, Florida         1446277
    ## 372  0500000US12101                      Pasco County, Florida          510593
    ## 373  0500000US12103                   Pinellas County, Florida          957875
    ## 374  0500000US12105                       Polk County, Florida          668671
    ## 375  0500000US12107                     Putnam County, Florida           72766
    ## 376  0500000US12109                  St. Johns County, Florida          235503
    ## 377  0500000US12111                  St. Lucie County, Florida          305591
    ## 378  0500000US12113                 Santa Rosa County, Florida          170442
    ## 379  0500000US12115                   Sarasota County, Florida          412144
    ## 380  0500000US12117                   Seminole County, Florida          455086
    ## 381  0500000US12119                     Sumter County, Florida          120999
    ## 382  0500000US12121                   Suwannee County, Florida           43924
    ## 383  0500000US12123                     Taylor County, Florida           22098
    ## 384  0500000US12125                      Union County, Florida           15239
    ## 385  0500000US12127                    Volusia County, Florida          527634
    ## 386  0500000US12129                    Wakulla County, Florida           31877
    ## 387  0500000US12131                     Walton County, Florida           65858
    ## 388  0500000US12133                 Washington County, Florida           24566
    ## 389  0500000US13001                    Appling County, Georgia           18454
    ## 390  0500000US13003                   Atkinson County, Georgia            8265
    ## 391  0500000US13005                      Bacon County, Georgia           11228
    ## 392  0500000US13007                      Baker County, Georgia            3189
    ## 393  0500000US13009                    Baldwin County, Georgia           45286
    ## 394  0500000US13011                      Banks County, Georgia           18510
    ## 395  0500000US13013                     Barrow County, Georgia           76887
    ## 396  0500000US13015                     Bartow County, Georgia          103620
    ## 397  0500000US13017                   Ben Hill County, Georgia           17154
    ## 398  0500000US13019                    Berrien County, Georgia           19025
    ## 399  0500000US13021                       Bibb County, Georgia          153490
    ## 400  0500000US13023                   Bleckley County, Georgia           12775
    ## 401  0500000US13025                   Brantley County, Georgia           18561
    ## 402  0500000US13027                     Brooks County, Georgia           15622
    ## 403  0500000US13029                      Bryan County, Georgia           35885
    ## 404  0500000US13031                    Bulloch County, Georgia           74782
    ## 405  0500000US13033                      Burke County, Georgia           22550
    ## 406  0500000US13035                      Butts County, Georgia           23750
    ## 407  0500000US13037                    Calhoun County, Georgia            6428
    ## 408  0500000US13039                     Camden County, Georgia           52714
    ## 409  0500000US13043                    Candler County, Georgia           10827
    ## 410  0500000US13045                    Carroll County, Georgia          116022
    ## 411  0500000US13047                    Catoosa County, Georgia           66299
    ## 412  0500000US13049                   Charlton County, Georgia           12983
    ## 413  0500000US13051                    Chatham County, Georgia          287049
    ## 414  0500000US13053              Chattahoochee County, Georgia           10767
    ## 415  0500000US13055                  Chattooga County, Georgia           24817
    ## 416  0500000US13057                   Cherokee County, Georgia          241910
    ## 417  0500000US13059                     Clarke County, Georgia          124602
    ## 418  0500000US13061                       Clay County, Georgia            3001
    ## 419  0500000US13063                    Clayton County, Georgia          278666
    ## 420  0500000US13065                     Clinch County, Georgia            6743
    ## 421  0500000US13067                       Cobb County, Georgia          745057
    ## 422  0500000US13069                     Coffee County, Georgia           42961
    ## 423  0500000US13071                   Colquitt County, Georgia           45606
    ## 424  0500000US13073                   Columbia County, Georgia          147295
    ## 425  0500000US13075                       Cook County, Georgia           17184
    ## 426  0500000US13077                     Coweta County, Georgia          140516
    ## 427  0500000US13079                   Crawford County, Georgia           12344
    ## 428  0500000US13081                      Crisp County, Georgia           22846
    ## 429  0500000US13083                       Dade County, Georgia           16227
    ## 430  0500000US13085                     Dawson County, Georgia           23861
    ## 431  0500000US13087                    Decatur County, Georgia           26833
    ## 432  0500000US13089                     DeKalb County, Georgia          743187
    ## 433  0500000US13091                      Dodge County, Georgia           20919
    ## 434  0500000US13093                      Dooly County, Georgia           13905
    ## 435  0500000US13095                  Dougherty County, Georgia           91049
    ## 436  0500000US13097                    Douglas County, Georgia          141840
    ## 437  0500000US13099                      Early County, Georgia           10348
    ## 438  0500000US13101                     Echols County, Georgia            3994
    ## 439  0500000US13103                  Effingham County, Georgia           58689
    ## 440  0500000US13105                     Elbert County, Georgia           19212
    ## 441  0500000US13107                    Emanuel County, Georgia           22499
    ## 442  0500000US13109                      Evans County, Georgia           10727
    ## 443  0500000US13111                     Fannin County, Georgia           24925
    ## 444  0500000US13113                    Fayette County, Georgia          111369
    ## 445  0500000US13115                      Floyd County, Georgia           96824
    ## 446  0500000US13117                    Forsyth County, Georgia          219880
    ## 447  0500000US13119                   Franklin County, Georgia           22514
    ## 448  0500000US13121                     Fulton County, Georgia         1021902
    ## 449  0500000US13123                     Gilmer County, Georgia           29922
    ## 450  0500000US13125                   Glascock County, Georgia            3009
    ## 451  0500000US13127                      Glynn County, Georgia           83974
    ## 452  0500000US13129                     Gordon County, Georgia           56790
    ## 453  0500000US13131                      Grady County, Georgia           24926
    ## 454  0500000US13133                     Greene County, Georgia           16976
    ## 455  0500000US13135                   Gwinnett County, Georgia          902298
    ## 456  0500000US13137                  Habersham County, Georgia           44289
    ## 457  0500000US13139                       Hall County, Georgia          195961
    ## 458  0500000US13141                    Hancock County, Georgia            8535
    ## 459  0500000US13143                   Haralson County, Georgia           28956
    ## 460  0500000US13145                     Harris County, Georgia           33590
    ## 461  0500000US13147                       Hart County, Georgia           25631
    ## 462  0500000US13149                      Heard County, Georgia           11677
    ## 463  0500000US13151                      Henry County, Georgia          221307
    ## 464  0500000US13153                    Houston County, Georgia          151682
    ## 465  0500000US13155                      Irwin County, Georgia            9268
    ## 466  0500000US13157                    Jackson County, Georgia           65755
    ## 467  0500000US13159                     Jasper County, Georgia           13784
    ## 468  0500000US13161                 Jeff Davis County, Georgia           14991
    ## 469  0500000US13163                  Jefferson County, Georgia           15772
    ## 470  0500000US13165                    Jenkins County, Georgia            8827
    ## 471  0500000US13167                    Johnson County, Georgia            9730
    ## 472  0500000US13169                      Jones County, Georgia           28548
    ## 473  0500000US13171                      Lamar County, Georgia           18513
    ## 474  0500000US13173                     Lanier County, Georgia           10366
    ## 475  0500000US13175                    Laurens County, Georgia           47418
    ## 476  0500000US13177                        Lee County, Georgia           29348
    ## 477  0500000US13179                    Liberty County, Georgia           62108
    ## 478  0500000US13181                    Lincoln County, Georgia            7799
    ## 479  0500000US13183                       Long County, Georgia           18156
    ## 480  0500000US13185                    Lowndes County, Georgia          114582
    ## 481  0500000US13187                    Lumpkin County, Georgia           31951
    ## 482  0500000US13189                   McDuffie County, Georgia           21498
    ## 483  0500000US13191                   McIntosh County, Georgia           14120
    ## 484  0500000US13193                      Macon County, Georgia           13480
    ## 485  0500000US13195                    Madison County, Georgia           28900
    ## 486  0500000US13197                     Marion County, Georgia            8484
    ## 487  0500000US13199                 Meriwether County, Georgia           21113
    ## 488  0500000US13201                     Miller County, Georgia            5836
    ## 489  0500000US13205                   Mitchell County, Georgia           22432
    ## 490  0500000US13207                     Monroe County, Georgia           27010
    ## 491  0500000US13209                 Montgomery County, Georgia            9036
    ## 492  0500000US13211                     Morgan County, Georgia           18235
    ## 493  0500000US13213                     Murray County, Georgia           39557
    ## 494  0500000US13215                   Muscogee County, Georgia          196670
    ## 495  0500000US13217                     Newton County, Georgia          106497
    ## 496  0500000US13219                     Oconee County, Georgia           37017
    ## 497  0500000US13221                 Oglethorpe County, Georgia           14784
    ## 498  0500000US13223                   Paulding County, Georgia          155840
    ## 499  0500000US13225                      Peach County, Georgia           26966
    ## 500  0500000US13227                    Pickens County, Georgia           30832
    ## 501  0500000US13229                     Pierce County, Georgia           19164
    ## 502  0500000US13231                       Pike County, Georgia           18082
    ## 503  0500000US13233                       Polk County, Georgia           41621
    ## 504  0500000US13235                    Pulaski County, Georgia           11295
    ## 505  0500000US13237                     Putnam County, Georgia           21503
    ## 506  0500000US13239                    Quitman County, Georgia            2276
    ## 507  0500000US13241                      Rabun County, Georgia           16457
    ## 508  0500000US13243                   Randolph County, Georgia            7087
    ## 509  0500000US13245                   Richmond County, Georgia          201463
    ## 510  0500000US13247                   Rockdale County, Georgia           89011
    ## 511  0500000US13249                     Schley County, Georgia            5211
    ## 512  0500000US13251                    Screven County, Georgia           13990
    ## 513  0500000US13253                   Seminole County, Georgia            8437
    ## 514  0500000US13255                   Spalding County, Georgia           64719
    ## 515  0500000US13257                   Stephens County, Georgia           25676
    ## 516  0500000US13259                    Stewart County, Georgia            6042
    ## 517  0500000US13261                     Sumter County, Georgia           30352
    ## 518  0500000US13263                     Talbot County, Georgia            6378
    ## 519  0500000US13265                 Taliaferro County, Georgia            1665
    ## 520  0500000US13267                   Tattnall County, Georgia           25353
    ## 521  0500000US13269                     Taylor County, Georgia            8193
    ## 522  0500000US13271                    Telfair County, Georgia           16115
    ## 523  0500000US13273                    Terrell County, Georgia            8859
    ## 524  0500000US13275                     Thomas County, Georgia           44730
    ## 525  0500000US13277                       Tift County, Georgia           40510
    ## 526  0500000US13279                     Toombs County, Georgia           27048
    ## 527  0500000US13281                      Towns County, Georgia           11417
    ## 528  0500000US13283                   Treutlen County, Georgia            6777
    ## 529  0500000US13285                      Troup County, Georgia           69774
    ## 530  0500000US13287                     Turner County, Georgia            7962
    ## 531  0500000US13289                     Twiggs County, Georgia            8284
    ## 532  0500000US13291                      Union County, Georgia           22775
    ## 533  0500000US13293                      Upson County, Georgia           26216
    ## 534  0500000US13295                     Walker County, Georgia           68824
    ## 535  0500000US13297                     Walton County, Georgia           90132
    ## 536  0500000US13299                       Ware County, Georgia           35599
    ## 537  0500000US13301                     Warren County, Georgia            5346
    ## 538  0500000US13303                 Washington County, Georgia           20461
    ## 539  0500000US13305                      Wayne County, Georgia           29767
    ## 540  0500000US13307                    Webster County, Georgia            2613
    ## 541  0500000US13309                    Wheeler County, Georgia            7939
    ## 542  0500000US13311                      White County, Georgia           28928
    ## 543  0500000US13313                  Whitfield County, Georgia          103849
    ## 544  0500000US13315                     Wilcox County, Georgia            8846
    ## 545  0500000US13317                     Wilkes County, Georgia            9884
    ## 546  0500000US13319                  Wilkinson County, Georgia            9078
    ## 547  0500000US13321                      Worth County, Georgia           20656
    ## 548  0500000US15001                      Hawaii County, Hawaii          197658
    ## 549  0500000US15003                    Honolulu County, Hawaii          987638
    ## 550  0500000US15005                     Kalawao County, Hawaii              75
    ## 551  0500000US15007                       Kauai County, Hawaii           71377
    ## 552  0500000US15009                        Maui County, Hawaii          165281
    ## 553  0500000US16001                          Ada County, Idaho          446052
    ## 554  0500000US16003                        Adams County, Idaho            4019
    ## 555  0500000US16005                      Bannock County, Idaho           85065
    ## 556  0500000US16007                    Bear Lake County, Idaho            5962
    ## 557  0500000US16009                      Benewah County, Idaho            9086
    ## 558  0500000US16011                      Bingham County, Idaho           45551
    ## 559  0500000US16013                       Blaine County, Idaho           21994
    ## 560  0500000US16015                        Boise County, Idaho            7163
    ## 561  0500000US16017                       Bonner County, Idaho           42711
    ## 562  0500000US16019                   Bonneville County, Idaho          112397
    ## 563  0500000US16021                     Boundary County, Idaho           11549
    ## 564  0500000US16023                        Butte County, Idaho            2602
    ## 565  0500000US16025                        Camas County, Idaho             886
    ## 566  0500000US16027                       Canyon County, Idaho          212230
    ## 567  0500000US16029                      Caribou County, Idaho            6918
    ## 568  0500000US16031                       Cassia County, Idaho           23615
    ## 569  0500000US16033                        Clark County, Idaho            1077
    ## 570  0500000US16035                   Clearwater County, Idaho            8640
    ## 571  0500000US16037                       Custer County, Idaho            4141
    ## 572  0500000US16039                       Elmore County, Idaho           26433
    ## 573  0500000US16041                     Franklin County, Idaho           13279
    ## 574  0500000US16043                      Fremont County, Idaho           12965
    ## 575  0500000US16045                          Gem County, Idaho           17052
    ## 576  0500000US16047                      Gooding County, Idaho           15169
    ## 577  0500000US16049                        Idaho County, Idaho           16337
    ## 578  0500000US16051                    Jefferson County, Idaho           27969
    ## 579  0500000US16053                       Jerome County, Idaho           23431
    ## 580  0500000US16055                     Kootenai County, Idaho          153605
    ## 581  0500000US16057                        Latah County, Idaho           39239
    ## 582  0500000US16059                        Lemhi County, Idaho            7798
    ## 583  0500000US16061                        Lewis County, Idaho            3845
    ## 584  0500000US16063                      Lincoln County, Idaho            5321
    ## 585  0500000US16065                      Madison County, Idaho           38705
    ## 586  0500000US16067                     Minidoka County, Idaho           20615
    ## 587  0500000US16069                    Nez Perce County, Idaho           40155
    ## 588  0500000US16071                       Oneida County, Idaho            4326
    ## 589  0500000US16073                       Owyhee County, Idaho           11455
    ## 590  0500000US16075                      Payette County, Idaho           23041
    ## 591  0500000US16077                        Power County, Idaho            7713
    ## 592  0500000US16079                     Shoshone County, Idaho           12526
    ## 593  0500000US16081                        Teton County, Idaho           11080
    ## 594  0500000US16083                   Twin Falls County, Idaho           83666
    ## 595  0500000US16085                       Valley County, Idaho           10401
    ## 596  0500000US16087                   Washington County, Idaho           10025
    ## 597  0500000US17001                     Adams County, Illinois           66427
    ## 598  0500000US17003                 Alexander County, Illinois            6532
    ## 599  0500000US17005                      Bond County, Illinois           16712
    ## 600  0500000US17007                     Boone County, Illinois           53606
    ## 601  0500000US17009                     Brown County, Illinois            6675
    ## 602  0500000US17011                    Bureau County, Illinois           33381
    ## 603  0500000US17013                   Calhoun County, Illinois            4858
    ## 604  0500000US17015                   Carroll County, Illinois           14562
    ## 605  0500000US17017                      Cass County, Illinois           12665
    ## 606  0500000US17019                 Champaign County, Illinois          209448
    ## 607  0500000US17021                 Christian County, Illinois           33231
    ## 608  0500000US17023                     Clark County, Illinois           15836
    ## 609  0500000US17025                      Clay County, Illinois           13338
    ## 610  0500000US17027                   Clinton County, Illinois           37628
    ## 611  0500000US17029                     Coles County, Illinois           51736
    ## 612  0500000US17031                      Cook County, Illinois         5223719
    ## 613  0500000US17033                  Crawford County, Illinois           19088
    ## 614  0500000US17035                Cumberland County, Illinois           10865
    ## 615  0500000US17037                    DeKalb County, Illinois          104200
    ## 616  0500000US17039                   De Witt County, Illinois           16042
    ## 617  0500000US17041                   Douglas County, Illinois           19714
    ## 618  0500000US17043                    DuPage County, Illinois          931743
    ## 619  0500000US17045                     Edgar County, Illinois           17539
    ## 620  0500000US17047                   Edwards County, Illinois            6507
    ## 621  0500000US17049                 Effingham County, Illinois           34174
    ## 622  0500000US17051                   Fayette County, Illinois           21724
    ## 623  0500000US17053                      Ford County, Illinois           13398
    ## 624  0500000US17055                  Franklin County, Illinois           39127
    ## 625  0500000US17057                    Fulton County, Illinois           35418
    ## 626  0500000US17059                  Gallatin County, Illinois            5157
    ## 627  0500000US17061                    Greene County, Illinois           13218
    ## 628  0500000US17063                    Grundy County, Illinois           50509
    ## 629  0500000US17065                  Hamilton County, Illinois            8221
    ## 630  0500000US17067                   Hancock County, Illinois           18112
    ## 631  0500000US17069                    Hardin County, Illinois            4009
    ## 632  0500000US17071                 Henderson County, Illinois            6884
    ## 633  0500000US17073                     Henry County, Illinois           49464
    ## 634  0500000US17075                  Iroquois County, Illinois           28169
    ## 635  0500000US17077                   Jackson County, Illinois           58551
    ## 636  0500000US17079                    Jasper County, Illinois            9598
    ## 637  0500000US17081                 Jefferson County, Illinois           38169
    ## 638  0500000US17083                    Jersey County, Illinois           22069
    ## 639  0500000US17085                Jo Daviess County, Illinois           21834
    ## 640  0500000US17087                   Johnson County, Illinois           12602
    ## 641  0500000US17089                      Kane County, Illinois          530839
    ## 642  0500000US17091                  Kankakee County, Illinois          111061
    ## 643  0500000US17093                   Kendall County, Illinois          124626
    ## 644  0500000US17095                      Knox County, Illinois           50999
    ## 645  0500000US17097                      Lake County, Illinois          703619
    ## 646  0500000US17099                   LaSalle County, Illinois          110401
    ## 647  0500000US17101                  Lawrence County, Illinois           16189
    ## 648  0500000US17103                       Lee County, Illinois           34527
    ## 649  0500000US17105                Livingston County, Illinois           36324
    ## 650  0500000US17107                     Logan County, Illinois           29207
    ## 651  0500000US17109                 McDonough County, Illinois           30875
    ## 652  0500000US17111                   McHenry County, Illinois          307789
    ## 653  0500000US17113                    McLean County, Illinois          173219
    ## 654  0500000US17115                     Macon County, Illinois          106512
    ## 655  0500000US17117                  Macoupin County, Illinois           45719
    ## 656  0500000US17119                   Madison County, Illinois          265670
    ## 657  0500000US17121                    Marion County, Illinois           38084
    ## 658  0500000US17123                  Marshall County, Illinois           11794
    ## 659  0500000US17125                     Mason County, Illinois           13778
    ## 660  0500000US17127                    Massac County, Illinois           14430
    ## 661  0500000US17129                    Menard County, Illinois           12367
    ## 662  0500000US17131                    Mercer County, Illinois           15693
    ## 663  0500000US17133                    Monroe County, Illinois           33936
    ## 664  0500000US17135                Montgomery County, Illinois           29009
    ## 665  0500000US17137                    Morgan County, Illinois           34426
    ## 666  0500000US17139                  Moultrie County, Illinois           14703
    ## 667  0500000US17141                      Ogle County, Illinois           51328
    ## 668  0500000US17143                    Peoria County, Illinois          184463
    ## 669  0500000US17145                     Perry County, Illinois           21384
    ## 670  0500000US17147                     Piatt County, Illinois           16427
    ## 671  0500000US17149                      Pike County, Illinois           15754
    ## 672  0500000US17151                      Pope County, Illinois            4249
    ## 673  0500000US17153                   Pulaski County, Illinois            5611
    ## 674  0500000US17155                    Putnam County, Illinois            5746
    ## 675  0500000US17157                  Randolph County, Illinois           32546
    ## 676  0500000US17159                  Richland County, Illinois           15881
    ## 677  0500000US17161               Rock Island County, Illinois          145275
    ## 678  0500000US17163                 St. Clair County, Illinois          263463
    ## 679  0500000US17165                    Saline County, Illinois           24231
    ## 680  0500000US17167                  Sangamon County, Illinois          197661
    ## 681  0500000US17169                  Schuyler County, Illinois            7064
    ## 682  0500000US17171                     Scott County, Illinois            5047
    ## 683  0500000US17173                    Shelby County, Illinois           21832
    ## 684  0500000US17175                     Stark County, Illinois            5500
    ## 685  0500000US17177                Stephenson County, Illinois           45433
    ## 686  0500000US17179                  Tazewell County, Illinois          133852
    ## 687  0500000US17181                     Union County, Illinois           17127
    ## 688  0500000US17183                 Vermilion County, Illinois           78407
    ## 689  0500000US17185                    Wabash County, Illinois           11573
    ## 690  0500000US17187                    Warren County, Illinois           17338
    ## 691  0500000US17189                Washington County, Illinois           14155
    ## 692  0500000US17191                     Wayne County, Illinois           16487
    ## 693  0500000US17193                     White County, Illinois           14025
    ## 694  0500000US17195                 Whiteside County, Illinois           56396
    ## 695  0500000US17197                      Will County, Illinois          688697
    ## 696  0500000US17199                Williamson County, Illinois           67299
    ## 697  0500000US17201                 Winnebago County, Illinois          286174
    ## 698  0500000US17203                  Woodford County, Illinois           38817
    ## 699  0500000US18001                      Adams County, Indiana           35195
    ## 700  0500000US18003                      Allen County, Indiana          370016
    ## 701  0500000US18005                Bartholomew County, Indiana           81893
    ## 702  0500000US18007                     Benton County, Indiana            8667
    ## 703  0500000US18009                  Blackford County, Indiana           12129
    ## 704  0500000US18011                      Boone County, Indiana           64321
    ## 705  0500000US18013                      Brown County, Indiana           15034
    ## 706  0500000US18015                    Carroll County, Indiana           19994
    ## 707  0500000US18017                       Cass County, Indiana           38084
    ## 708  0500000US18019                      Clark County, Indiana          115702
    ## 709  0500000US18021                       Clay County, Indiana           26268
    ## 710  0500000US18023                    Clinton County, Indiana           32301
    ## 711  0500000US18025                   Crawford County, Indiana           10581
    ## 712  0500000US18027                    Daviess County, Indiana           32937
    ## 713  0500000US18029                   Dearborn County, Indiana           49501
    ## 714  0500000US18031                    Decatur County, Indiana           26552
    ## 715  0500000US18033                     DeKalb County, Indiana           42704
    ## 716  0500000US18035                   Delaware County, Indiana          115616
    ## 717  0500000US18037                     Dubois County, Indiana           42418
    ## 718  0500000US18039                    Elkhart County, Indiana          203604
    ## 719  0500000US18041                    Fayette County, Indiana           23259
    ## 720  0500000US18043                      Floyd County, Indiana           76809
    ## 721  0500000US18045                   Fountain County, Indiana           16486
    ## 722  0500000US18047                   Franklin County, Indiana           22842
    ## 723  0500000US18049                     Fulton County, Indiana           20212
    ## 724  0500000US18051                     Gibson County, Indiana           33596
    ## 725  0500000US18053                      Grant County, Indiana           66944
    ## 726  0500000US18055                     Greene County, Indiana           32295
    ## 727  0500000US18057                   Hamilton County, Indiana          316095
    ## 728  0500000US18059                    Hancock County, Indiana           73830
    ## 729  0500000US18061                   Harrison County, Indiana           39712
    ## 730  0500000US18063                  Hendricks County, Indiana          160940
    ## 731  0500000US18065                      Henry County, Indiana           48483
    ## 732  0500000US18067                     Howard County, Indiana           82387
    ## 733  0500000US18069                 Huntington County, Indiana           36378
    ## 734  0500000US18071                    Jackson County, Indiana           43938
    ## 735  0500000US18073                     Jasper County, Indiana           33449
    ## 736  0500000US18075                        Jay County, Indiana           20993
    ## 737  0500000US18077                  Jefferson County, Indiana           32237
    ## 738  0500000US18079                   Jennings County, Indiana           27727
    ## 739  0500000US18081                    Johnson County, Indiana          151564
    ## 740  0500000US18083                       Knox County, Indiana           37409
    ## 741  0500000US18085                  Kosciusko County, Indiana           78806
    ## 742  0500000US18087                   LaGrange County, Indiana           38942
    ## 743  0500000US18089                       Lake County, Indiana          486849
    ## 744  0500000US18091                    LaPorte County, Indiana          110552
    ## 745  0500000US18093                   Lawrence County, Indiana           45619
    ## 746  0500000US18095                    Madison County, Indiana          129505
    ## 747  0500000US18097                     Marion County, Indiana          944523
    ## 748  0500000US18099                   Marshall County, Indiana           46595
    ## 749  0500000US18101                     Martin County, Indiana           10210
    ## 750  0500000US18103                      Miami County, Indiana           35901
    ## 751  0500000US18105                     Monroe County, Indiana          145403
    ## 752  0500000US18107                 Montgomery County, Indiana           38276
    ## 753  0500000US18109                     Morgan County, Indiana           69727
    ## 754  0500000US18111                     Newton County, Indiana           14018
    ## 755  0500000US18113                      Noble County, Indiana           47451
    ## 756  0500000US18115                       Ohio County, Indiana            5887
    ## 757  0500000US18117                     Orange County, Indiana           19547
    ## 758  0500000US18119                       Owen County, Indiana           20878
    ## 759  0500000US18121                      Parke County, Indiana           16996
    ## 760  0500000US18123                      Perry County, Indiana           19141
    ## 761  0500000US18125                       Pike County, Indiana           12411
    ## 762  0500000US18127                     Porter County, Indiana          168041
    ## 763  0500000US18129                      Posey County, Indiana           25589
    ## 764  0500000US18131                    Pulaski County, Indiana           12660
    ## 765  0500000US18133                     Putnam County, Indiana           37559
    ## 766  0500000US18135                   Randolph County, Indiana           25076
    ## 767  0500000US18137                     Ripley County, Indiana           28425
    ## 768  0500000US18139                       Rush County, Indiana           16704
    ## 769  0500000US18141                 St. Joseph County, Indiana          269240
    ## 770  0500000US18143                      Scott County, Indiana           23743
    ## 771  0500000US18145                     Shelby County, Indiana           44399
    ## 772  0500000US18147                    Spencer County, Indiana           20526
    ## 773  0500000US18149                     Starke County, Indiana           22941
    ## 774  0500000US18151                    Steuben County, Indiana           34474
    ## 775  0500000US18153                   Sullivan County, Indiana           20792
    ## 776  0500000US18155                Switzerland County, Indiana           10628
    ## 777  0500000US18157                 Tippecanoe County, Indiana          189294
    ## 778  0500000US18159                     Tipton County, Indiana           15218
    ## 779  0500000US18161                      Union County, Indiana            7153
    ## 780  0500000US18163                Vanderburgh County, Indiana          181313
    ## 781  0500000US18165                 Vermillion County, Indiana           15560
    ## 782  0500000US18167                       Vigo County, Indiana          107693
    ## 783  0500000US18169                     Wabash County, Indiana           31631
    ## 784  0500000US18171                     Warren County, Indiana            8247
    ## 785  0500000US18173                    Warrick County, Indiana           61928
    ## 786  0500000US18175                 Washington County, Indiana           27827
    ## 787  0500000US18177                      Wayne County, Indiana           66613
    ## 788  0500000US18179                      Wells County, Indiana           27947
    ## 789  0500000US18181                      White County, Indiana           24217
    ## 790  0500000US18183                    Whitley County, Indiana           33649
    ## 791  0500000US19001                         Adair County, Iowa            7124
    ## 792  0500000US19003                         Adams County, Iowa            3726
    ## 793  0500000US19005                     Allamakee County, Iowa           13880
    ## 794  0500000US19007                     Appanoose County, Iowa           12510
    ## 795  0500000US19009                       Audubon County, Iowa            5637
    ## 796  0500000US19011                        Benton County, Iowa           25626
    ## 797  0500000US19013                    Black Hawk County, Iowa          133009
    ## 798  0500000US19015                         Boone County, Iowa           26399
    ## 799  0500000US19017                        Bremer County, Iowa           24782
    ## 800  0500000US19019                      Buchanan County, Iowa           21125
    ## 801  0500000US19021                   Buena Vista County, Iowa           20260
    ## 802  0500000US19023                        Butler County, Iowa           14735
    ## 803  0500000US19025                       Calhoun County, Iowa            9780
    ## 804  0500000US19027                       Carroll County, Iowa           20344
    ## 805  0500000US19029                          Cass County, Iowa           13191
    ## 806  0500000US19031                         Cedar County, Iowa           18445
    ## 807  0500000US19033                   Cerro Gordo County, Iowa           42984
    ## 808  0500000US19035                      Cherokee County, Iowa           11468
    ## 809  0500000US19037                     Chickasaw County, Iowa           12099
    ## 810  0500000US19039                        Clarke County, Iowa            9282
    ## 811  0500000US19041                          Clay County, Iowa           16313
    ## 812  0500000US19043                       Clayton County, Iowa           17672
    ## 813  0500000US19045                       Clinton County, Iowa           47218
    ## 814  0500000US19047                      Crawford County, Iowa           17132
    ## 815  0500000US19049                        Dallas County, Iowa           84002
    ## 816  0500000US19051                         Davis County, Iowa            8885
    ## 817  0500000US19053                       Decatur County, Iowa            8044
    ## 818  0500000US19055                      Delaware County, Iowa           17258
    ## 819  0500000US19057                    Des Moines County, Iowa           39600
    ## 820  0500000US19059                     Dickinson County, Iowa           17056
    ## 821  0500000US19061                       Dubuque County, Iowa           96802
    ## 822  0500000US19063                         Emmet County, Iowa            9551
    ## 823  0500000US19065                       Fayette County, Iowa           19929
    ## 824  0500000US19067                         Floyd County, Iowa           15858
    ## 825  0500000US19069                      Franklin County, Iowa           10245
    ## 826  0500000US19071                       Fremont County, Iowa            6968
    ## 827  0500000US19073                        Greene County, Iowa            9003
    ## 828  0500000US19075                        Grundy County, Iowa           12341
    ## 829  0500000US19077                       Guthrie County, Iowa           10674
    ## 830  0500000US19079                      Hamilton County, Iowa           15110
    ## 831  0500000US19081                       Hancock County, Iowa           10888
    ## 832  0500000US19083                        Hardin County, Iowa           17127
    ## 833  0500000US19085                      Harrison County, Iowa           14143
    ## 834  0500000US19087                         Henry County, Iowa           19926
    ## 835  0500000US19089                        Howard County, Iowa            9264
    ## 836  0500000US19091                      Humboldt County, Iowa            9566
    ## 837  0500000US19093                           Ida County, Iowa            6916
    ## 838  0500000US19095                          Iowa County, Iowa           16207
    ## 839  0500000US19097                       Jackson County, Iowa           19395
    ## 840  0500000US19099                        Jasper County, Iowa           36891
    ## 841  0500000US19101                     Jefferson County, Iowa           18077
    ## 842  0500000US19103                       Johnson County, Iowa          147001
    ## 843  0500000US19105                         Jones County, Iowa           20568
    ## 844  0500000US19107                        Keokuk County, Iowa           10200
    ## 845  0500000US19109                       Kossuth County, Iowa           15075
    ## 846  0500000US19111                           Lee County, Iowa           34541
    ## 847  0500000US19113                          Linn County, Iowa          222121
    ## 848  0500000US19115                        Louisa County, Iowa           11223
    ## 849  0500000US19117                         Lucas County, Iowa            8597
    ## 850  0500000US19119                          Lyon County, Iowa           11769
    ## 851  0500000US19121                       Madison County, Iowa           15890
    ## 852  0500000US19123                       Mahaska County, Iowa           22208
    ## 853  0500000US19125                        Marion County, Iowa           33207
    ## 854  0500000US19127                      Marshall County, Iowa           40271
    ## 855  0500000US19129                         Mills County, Iowa           14957
    ## 856  0500000US19131                      Mitchell County, Iowa           10631
    ## 857  0500000US19133                        Monona County, Iowa            8796
    ## 858  0500000US19135                        Monroe County, Iowa            7863
    ## 859  0500000US19137                    Montgomery County, Iowa           10155
    ## 860  0500000US19139                     Muscatine County, Iowa           42950
    ## 861  0500000US19141                       O'Brien County, Iowa           13911
    ## 862  0500000US19143                       Osceola County, Iowa            6115
    ## 863  0500000US19145                          Page County, Iowa           15363
    ## 864  0500000US19147                     Palo Alto County, Iowa            9055
    ## 865  0500000US19149                      Plymouth County, Iowa           25039
    ## 866  0500000US19151                    Pocahontas County, Iowa            6898
    ## 867  0500000US19153                          Polk County, Iowa          474274
    ## 868  0500000US19155                 Pottawattamie County, Iowa           93503
    ## 869  0500000US19157                     Poweshiek County, Iowa           18605
    ## 870  0500000US19159                      Ringgold County, Iowa            4984
    ## 871  0500000US19161                           Sac County, Iowa            9868
    ## 872  0500000US19163                         Scott County, Iowa          172288
    ## 873  0500000US19165                        Shelby County, Iowa           11694
    ## 874  0500000US19167                         Sioux County, Iowa           34825
    ## 875  0500000US19169                         Story County, Iowa           96922
    ## 876  0500000US19171                          Tama County, Iowa           17136
    ## 877  0500000US19173                        Taylor County, Iowa            6201
    ## 878  0500000US19175                         Union County, Iowa           12453
    ## 879  0500000US19177                     Van Buren County, Iowa            7223
    ## 880  0500000US19179                       Wapello County, Iowa           35315
    ## 881  0500000US19181                        Warren County, Iowa           49361
    ## 882  0500000US19183                    Washington County, Iowa           22143
    ## 883  0500000US19185                         Wayne County, Iowa            6413
    ## 884  0500000US19187                       Webster County, Iowa           36757
    ## 885  0500000US19189                     Winnebago County, Iowa           10571
    ## 886  0500000US19191                    Winneshiek County, Iowa           20401
    ## 887  0500000US19193                      Woodbury County, Iowa          102398
    ## 888  0500000US19195                         Worth County, Iowa            7489
    ## 889  0500000US19197                        Wright County, Iowa           12804
    ## 890  0500000US20001                       Allen County, Kansas           12630
    ## 891  0500000US20003                    Anderson County, Kansas            7852
    ## 892  0500000US20005                    Atchison County, Kansas           16363
    ## 893  0500000US20007                      Barber County, Kansas            4733
    ## 894  0500000US20009                      Barton County, Kansas           26791
    ## 895  0500000US20011                     Bourbon County, Kansas           14702
    ## 896  0500000US20013                       Brown County, Kansas            9664
    ## 897  0500000US20015                      Butler County, Kansas           66468
    ## 898  0500000US20017                       Chase County, Kansas            2645
    ## 899  0500000US20019                  Chautauqua County, Kansas            3367
    ## 900  0500000US20021                    Cherokee County, Kansas           20331
    ## 901  0500000US20023                    Cheyenne County, Kansas            2677
    ## 902  0500000US20025                       Clark County, Kansas            2053
    ## 903  0500000US20027                        Clay County, Kansas            8142
    ## 904  0500000US20029                       Cloud County, Kansas            9060
    ## 905  0500000US20031                      Coffey County, Kansas            8296
    ## 906  0500000US20033                    Comanche County, Kansas            1780
    ## 907  0500000US20035                      Cowley County, Kansas           35591
    ## 908  0500000US20037                    Crawford County, Kansas           39108
    ## 909  0500000US20039                     Decatur County, Kansas            2881
    ## 910  0500000US20041                   Dickinson County, Kansas           19004
    ## 911  0500000US20043                    Doniphan County, Kansas            7736
    ## 912  0500000US20045                     Douglas County, Kansas          119319
    ## 913  0500000US20047                     Edwards County, Kansas            2925
    ## 914  0500000US20049                         Elk County, Kansas            2562
    ## 915  0500000US20051                       Ellis County, Kansas           28878
    ## 916  0500000US20053                   Ellsworth County, Kansas            6293
    ## 917  0500000US20055                      Finney County, Kansas           36957
    ## 918  0500000US20057                        Ford County, Kansas           34484
    ## 919  0500000US20059                    Franklin County, Kansas           25563
    ## 920  0500000US20061                       Geary County, Kansas           34895
    ## 921  0500000US20063                        Gove County, Kansas            2619
    ## 922  0500000US20065                      Graham County, Kansas            2545
    ## 923  0500000US20067                       Grant County, Kansas            7616
    ## 924  0500000US20069                        Gray County, Kansas            6037
    ## 925  0500000US20071                     Greeley County, Kansas            1200
    ## 926  0500000US20073                   Greenwood County, Kansas            6156
    ## 927  0500000US20075                    Hamilton County, Kansas            2616
    ## 928  0500000US20077                      Harper County, Kansas            5673
    ## 929  0500000US20079                      Harvey County, Kansas           34555
    ## 930  0500000US20081                     Haskell County, Kansas            4047
    ## 931  0500000US20083                    Hodgeman County, Kansas            1842
    ## 932  0500000US20085                     Jackson County, Kansas           13318
    ## 933  0500000US20087                   Jefferson County, Kansas           18888
    ## 934  0500000US20089                      Jewell County, Kansas            2916
    ## 935  0500000US20091                     Johnson County, Kansas          585502
    ## 936  0500000US20093                      Kearny County, Kansas            3932
    ## 937  0500000US20095                     Kingman County, Kansas            7470
    ## 938  0500000US20097                       Kiowa County, Kansas            2526
    ## 939  0500000US20099                     Labette County, Kansas           20367
    ## 940  0500000US20101                        Lane County, Kansas            1642
    ## 941  0500000US20103                 Leavenworth County, Kansas           80042
    ## 942  0500000US20105                     Lincoln County, Kansas            3097
    ## 943  0500000US20107                        Linn County, Kansas            9635
    ## 944  0500000US20109                       Logan County, Kansas            2810
    ## 945  0500000US20111                        Lyon County, Kansas           33299
    ## 946  0500000US20113                   McPherson County, Kansas           28630
    ## 947  0500000US20115                      Marion County, Kansas           12032
    ## 948  0500000US20117                    Marshall County, Kansas            9798
    ## 949  0500000US20119                       Meade County, Kansas            4261
    ## 950  0500000US20121                       Miami County, Kansas           33127
    ## 951  0500000US20123                    Mitchell County, Kansas            6222
    ## 952  0500000US20125                  Montgomery County, Kansas           32970
    ## 953  0500000US20127                      Morris County, Kansas            5566
    ## 954  0500000US20129                      Morton County, Kansas            2838
    ## 955  0500000US20131                      Nemaha County, Kansas           10104
    ## 956  0500000US20133                      Neosho County, Kansas           16125
    ## 957  0500000US20135                        Ness County, Kansas            2955
    ## 958  0500000US20137                      Norton County, Kansas            5486
    ## 959  0500000US20139                       Osage County, Kansas           15882
    ## 960  0500000US20141                     Osborne County, Kansas            3603
    ## 961  0500000US20143                      Ottawa County, Kansas            5902
    ## 962  0500000US20145                      Pawnee County, Kansas            6709
    ## 963  0500000US20147                    Phillips County, Kansas            5408
    ## 964  0500000US20149                Pottawatomie County, Kansas           23545
    ## 965  0500000US20151                       Pratt County, Kansas            9582
    ## 966  0500000US20153                     Rawlins County, Kansas            2509
    ## 967  0500000US20155                        Reno County, Kansas           63101
    ## 968  0500000US20157                    Republic County, Kansas            4686
    ## 969  0500000US20159                        Rice County, Kansas            9762
    ## 970  0500000US20161                       Riley County, Kansas           75296
    ## 971  0500000US20163                       Rooks County, Kansas            5118
    ## 972  0500000US20165                        Rush County, Kansas            3102
    ## 973  0500000US20167                     Russell County, Kansas            6977
    ## 974  0500000US20169                      Saline County, Kansas           54977
    ## 975  0500000US20171                       Scott County, Kansas            4949
    ## 976  0500000US20173                    Sedgwick County, Kansas          512064
    ## 977  0500000US20175                      Seward County, Kansas           22692
    ## 978  0500000US20177                     Shawnee County, Kansas          178284
    ## 979  0500000US20179                    Sheridan County, Kansas            2506
    ## 980  0500000US20181                     Sherman County, Kansas            5966
    ## 981  0500000US20183                       Smith County, Kansas            3663
    ## 982  0500000US20185                    Stafford County, Kansas            4214
    ## 983  0500000US20187                     Stanton County, Kansas            2063
    ## 984  0500000US20189                     Stevens County, Kansas            5686
    ## 985  0500000US20191                      Sumner County, Kansas           23208
    ## 986  0500000US20193                      Thomas County, Kansas            7824
    ## 987  0500000US20195                       Trego County, Kansas            2858
    ## 988  0500000US20197                   Wabaunsee County, Kansas            6888
    ## 989  0500000US20199                     Wallace County, Kansas            1575
    ## 990  0500000US20201                  Washington County, Kansas            5525
    ## 991  0500000US20203                     Wichita County, Kansas            2143
    ## 992  0500000US20205                      Wilson County, Kansas            8780
    ## 993  0500000US20207                     Woodson County, Kansas            3170
    ## 994  0500000US20209                   Wyandotte County, Kansas          164345
    ## 995  0500000US21001                     Adair County, Kentucky           19241
    ## 996  0500000US21003                     Allen County, Kentucky           20794
    ## 997  0500000US21005                  Anderson County, Kentucky           22214
    ## 998  0500000US21007                   Ballard County, Kentucky            8090
    ## 999  0500000US21009                    Barren County, Kentucky           43680
    ## 1000 0500000US21011                      Bath County, Kentucky           12268
    ## 1001 0500000US21013                      Bell County, Kentucky           27188
    ## 1002 0500000US21015                     Boone County, Kentucky          129095
    ## 1003 0500000US21017                   Bourbon County, Kentucky           20144
    ## 1004 0500000US21019                      Boyd County, Kentucky           48091
    ## 1005 0500000US21021                     Boyle County, Kentucky           29913
    ## 1006 0500000US21023                   Bracken County, Kentucky            8306
    ## 1007 0500000US21025                 Breathitt County, Kentucky           13116
    ## 1008 0500000US21027              Breckinridge County, Kentucky           20080
    ## 1009 0500000US21029                   Bullitt County, Kentucky           79466
    ## 1010 0500000US21031                    Butler County, Kentucky           12745
    ## 1011 0500000US21033                  Caldwell County, Kentucky           12727
    ## 1012 0500000US21035                  Calloway County, Kentucky           38776
    ## 1013 0500000US21037                  Campbell County, Kentucky           92267
    ## 1014 0500000US21039                  Carlisle County, Kentucky            4841
    ## 1015 0500000US21041                   Carroll County, Kentucky           10711
    ## 1016 0500000US21043                    Carter County, Kentucky           27290
    ## 1017 0500000US21045                     Casey County, Kentucky           15796
    ## 1018 0500000US21047                 Christian County, Kentucky           72263
    ## 1019 0500000US21049                     Clark County, Kentucky           35872
    ## 1020 0500000US21051                      Clay County, Kentucky           20621
    ## 1021 0500000US21053                   Clinton County, Kentucky           10211
    ## 1022 0500000US21055                Crittenden County, Kentucky            9083
    ## 1023 0500000US21057                Cumberland County, Kentucky            6713
    ## 1024 0500000US21059                   Daviess County, Kentucky           99937
    ## 1025 0500000US21061                  Edmonson County, Kentucky           12122
    ## 1026 0500000US21063                   Elliott County, Kentucky            7517
    ## 1027 0500000US21065                    Estill County, Kentucky           14313
    ## 1028 0500000US21067                   Fayette County, Kentucky          318734
    ## 1029 0500000US21069                   Fleming County, Kentucky           14479
    ## 1030 0500000US21071                     Floyd County, Kentucky           36926
    ## 1031 0500000US21073                  Franklin County, Kentucky           50296
    ## 1032 0500000US21075                    Fulton County, Kentucky            6210
    ## 1033 0500000US21077                  Gallatin County, Kentucky            8703
    ## 1034 0500000US21079                   Garrard County, Kentucky           17328
    ## 1035 0500000US21081                     Grant County, Kentucky           24915
    ## 1036 0500000US21083                    Graves County, Kentucky           37294
    ## 1037 0500000US21085                   Grayson County, Kentucky           26178
    ## 1038 0500000US21087                     Green County, Kentucky           11023
    ## 1039 0500000US21089                   Greenup County, Kentucky           35765
    ## 1040 0500000US21091                   Hancock County, Kentucky            8719
    ## 1041 0500000US21093                    Hardin County, Kentucky          108095
    ## 1042 0500000US21095                    Harlan County, Kentucky           27134
    ## 1043 0500000US21097                  Harrison County, Kentucky           18668
    ## 1044 0500000US21099                      Hart County, Kentucky           18627
    ## 1045 0500000US21101                 Henderson County, Kentucky           46137
    ## 1046 0500000US21103                     Henry County, Kentucky           15814
    ## 1047 0500000US21105                   Hickman County, Kentucky            4568
    ## 1048 0500000US21107                   Hopkins County, Kentucky           45664
    ## 1049 0500000US21109                   Jackson County, Kentucky           13373
    ## 1050 0500000US21111                 Jefferson County, Kentucky          767154
    ## 1051 0500000US21113                 Jessamine County, Kentucky           52422
    ## 1052 0500000US21115                   Johnson County, Kentucky           22843
    ## 1053 0500000US21117                    Kenton County, Kentucky          164688
    ## 1054 0500000US21119                     Knott County, Kentucky           15513
    ## 1055 0500000US21121                      Knox County, Kentucky           31467
    ## 1056 0500000US21123                     Larue County, Kentucky           14156
    ## 1057 0500000US21125                    Laurel County, Kentucky           60180
    ## 1058 0500000US21127                  Lawrence County, Kentucky           15783
    ## 1059 0500000US21129                       Lee County, Kentucky            6751
    ## 1060 0500000US21131                    Leslie County, Kentucky           10472
    ## 1061 0500000US21133                   Letcher County, Kentucky           22676
    ## 1062 0500000US21135                     Lewis County, Kentucky           13490
    ## 1063 0500000US21137                   Lincoln County, Kentucky           24458
    ## 1064 0500000US21139                Livingston County, Kentucky            9263
    ## 1065 0500000US21141                     Logan County, Kentucky           26849
    ## 1066 0500000US21143                      Lyon County, Kentucky            8186
    ## 1067 0500000US21145                 McCracken County, Kentucky           65284
    ## 1068 0500000US21147                  McCreary County, Kentucky           17635
    ## 1069 0500000US21149                    McLean County, Kentucky            9331
    ## 1070 0500000US21151                   Madison County, Kentucky           89700
    ## 1071 0500000US21153                  Magoffin County, Kentucky           12666
    ## 1072 0500000US21155                    Marion County, Kentucky           19232
    ## 1073 0500000US21157                  Marshall County, Kentucky           31166
    ## 1074 0500000US21159                    Martin County, Kentucky           11919
    ## 1075 0500000US21161                     Mason County, Kentucky           17153
    ## 1076 0500000US21163                     Meade County, Kentucky           28326
    ## 1077 0500000US21165                   Menifee County, Kentucky            6405
    ## 1078 0500000US21167                    Mercer County, Kentucky           21516
    ## 1079 0500000US21169                  Metcalfe County, Kentucky           10004
    ## 1080 0500000US21171                    Monroe County, Kentucky           10634
    ## 1081 0500000US21173                Montgomery County, Kentucky           27759
    ## 1082 0500000US21175                    Morgan County, Kentucky           13285
    ## 1083 0500000US21177                Muhlenberg County, Kentucky           31081
    ## 1084 0500000US21179                    Nelson County, Kentucky           45388
    ## 1085 0500000US21181                  Nicholas County, Kentucky            7100
    ## 1086 0500000US21183                      Ohio County, Kentucky           24071
    ## 1087 0500000US21185                    Oldham County, Kentucky           65374
    ## 1088 0500000US21187                      Owen County, Kentucky           10741
    ## 1089 0500000US21189                    Owsley County, Kentucky            4463
    ## 1090 0500000US21191                 Pendleton County, Kentucky           14520
    ## 1091 0500000US21193                     Perry County, Kentucky           26917
    ## 1092 0500000US21195                      Pike County, Kentucky           60483
    ## 1093 0500000US21197                    Powell County, Kentucky           12321
    ## 1094 0500000US21199                   Pulaski County, Kentucky           64145
    ## 1095 0500000US21201                 Robertson County, Kentucky            2143
    ## 1096 0500000US21203                Rockcastle County, Kentucky           16827
    ## 1097 0500000US21205                     Rowan County, Kentucky           24499
    ## 1098 0500000US21207                   Russell County, Kentucky           17760
    ## 1099 0500000US21209                     Scott County, Kentucky           53517
    ## 1100 0500000US21211                    Shelby County, Kentucky           46786
    ## 1101 0500000US21213                   Simpson County, Kentucky           18063
    ## 1102 0500000US21215                   Spencer County, Kentucky           18246
    ## 1103 0500000US21217                    Taylor County, Kentucky           25500
    ## 1104 0500000US21219                      Todd County, Kentucky           12350
    ## 1105 0500000US21221                     Trigg County, Kentucky           14344
    ## 1106 0500000US21223                   Trimble County, Kentucky            8637
    ## 1107 0500000US21225                     Union County, Kentucky           14802
    ## 1108 0500000US21227                    Warren County, Kentucky          126427
    ## 1109 0500000US21229                Washington County, Kentucky           12019
    ## 1110 0500000US21231                     Wayne County, Kentucky           20609
    ## 1111 0500000US21233                   Webster County, Kentucky           13155
    ## 1112 0500000US21235                   Whitley County, Kentucky           36089
    ## 1113 0500000US21237                     Wolfe County, Kentucky            7223
    ## 1114 0500000US21239                  Woodford County, Kentucky           26097
    ## 1115 0500000US22001                   Acadia Parish, Louisiana           62568
    ## 1116 0500000US22003                    Allen Parish, Louisiana           25661
    ## 1117 0500000US22005                Ascension Parish, Louisiana          121176
    ## 1118 0500000US22007               Assumption Parish, Louisiana           22714
    ## 1119 0500000US22009                Avoyelles Parish, Louisiana           40882
    ## 1120 0500000US22011               Beauregard Parish, Louisiana           36769
    ## 1121 0500000US22013                Bienville Parish, Louisiana           13668
    ## 1122 0500000US22015                  Bossier Parish, Louisiana          126131
    ## 1123 0500000US22017                    Caddo Parish, Louisiana          248361
    ## 1124 0500000US22019                Calcasieu Parish, Louisiana          200182
    ## 1125 0500000US22021                 Caldwell Parish, Louisiana            9996
    ## 1126 0500000US22023                  Cameron Parish, Louisiana            6868
    ## 1127 0500000US22025                Catahoula Parish, Louisiana            9893
    ## 1128 0500000US22027                Claiborne Parish, Louisiana           16153
    ## 1129 0500000US22029                Concordia Parish, Louisiana           20021
    ## 1130 0500000US22031                  De Soto Parish, Louisiana           27216
    ## 1131 0500000US22033         East Baton Rouge Parish, Louisiana          444094
    ## 1132 0500000US22035             East Carroll Parish, Louisiana            7225
    ## 1133 0500000US22037           East Feliciana Parish, Louisiana           19499
    ## 1134 0500000US22039               Evangeline Parish, Louisiana           33636
    ## 1135 0500000US22041                 Franklin Parish, Louisiana           20322
    ## 1136 0500000US22043                    Grant Parish, Louisiana           22348
    ## 1137 0500000US22045                   Iberia Parish, Louisiana           72691
    ## 1138 0500000US22047                Iberville Parish, Louisiana           32956
    ## 1139 0500000US22049                  Jackson Parish, Louisiana           15926
    ## 1140 0500000US22051                Jefferson Parish, Louisiana          435300
    ## 1141 0500000US22053          Jefferson Davis Parish, Louisiana           31467
    ## 1142 0500000US22055                Lafayette Parish, Louisiana          240091
    ## 1143 0500000US22057                Lafourche Parish, Louisiana           98214
    ## 1144 0500000US22059                  LaSalle Parish, Louisiana           14949
    ## 1145 0500000US22061                  Lincoln Parish, Louisiana           47356
    ## 1146 0500000US22063               Livingston Parish, Louisiana          138111
    ## 1147 0500000US22065                  Madison Parish, Louisiana           11472
    ## 1148 0500000US22067                Morehouse Parish, Louisiana           25992
    ## 1149 0500000US22069             Natchitoches Parish, Louisiana           38963
    ## 1150 0500000US22071                  Orleans Parish, Louisiana          389648
    ## 1151 0500000US22073                 Ouachita Parish, Louisiana          156075
    ## 1152 0500000US22075              Plaquemines Parish, Louisiana           23373
    ## 1153 0500000US22077            Pointe Coupee Parish, Louisiana           22158
    ## 1154 0500000US22079                  Rapides Parish, Louisiana          131546
    ## 1155 0500000US22081                Red River Parish, Louisiana            8618
    ## 1156 0500000US22083                 Richland Parish, Louisiana           20474
    ## 1157 0500000US22085                   Sabine Parish, Louisiana           24088
    ## 1158 0500000US22087              St. Bernard Parish, Louisiana           45694
    ## 1159 0500000US22089              St. Charles Parish, Louisiana           52724
    ## 1160 0500000US22091               St. Helena Parish, Louisiana           10411
    ## 1161 0500000US22093                St. James Parish, Louisiana           21357
    ## 1162 0500000US22095     St. John the Baptist Parish, Louisiana           43446
    ## 1163 0500000US22097               St. Landry Parish, Louisiana           83449
    ## 1164 0500000US22099               St. Martin Parish, Louisiana           53752
    ## 1165 0500000US22101                 St. Mary Parish, Louisiana           51734
    ## 1166 0500000US22103              St. Tammany Parish, Louisiana          252093
    ## 1167 0500000US22105               Tangipahoa Parish, Louisiana          130504
    ## 1168 0500000US22107                   Tensas Parish, Louisiana            4666
    ## 1169 0500000US22109               Terrebonne Parish, Louisiana          112587
    ## 1170 0500000US22111                    Union Parish, Louisiana           22475
    ## 1171 0500000US22113                Vermilion Parish, Louisiana           59867
    ## 1172 0500000US22115                   Vernon Parish, Louisiana           51007
    ## 1173 0500000US22117               Washington Parish, Louisiana           46457
    ## 1174 0500000US22119                  Webster Parish, Louisiana           39631
    ## 1175 0500000US22121         West Baton Rouge Parish, Louisiana           25860
    ## 1176 0500000US22123             West Carroll Parish, Louisiana           11180
    ## 1177 0500000US22125           West Feliciana Parish, Louisiana           15377
    ## 1178 0500000US22127                     Winn Parish, Louisiana           14494
    ## 1179 0500000US23001                 Androscoggin County, Maine          107444
    ## 1180 0500000US23003                    Aroostook County, Maine           68269
    ## 1181 0500000US23005                   Cumberland County, Maine          290944
    ## 1182 0500000US23007                     Franklin County, Maine           30019
    ## 1183 0500000US23009                      Hancock County, Maine           54541
    ## 1184 0500000US23011                     Kennebec County, Maine          121545
    ## 1185 0500000US23013                         Knox County, Maine           39823
    ## 1186 0500000US23015                      Lincoln County, Maine           34067
    ## 1187 0500000US23017                       Oxford County, Maine           57325
    ## 1188 0500000US23019                    Penobscot County, Maine          151748
    ## 1189 0500000US23021                  Piscataquis County, Maine           16887
    ## 1190 0500000US23023                    Sagadahoc County, Maine           35277
    ## 1191 0500000US23025                     Somerset County, Maine           50710
    ## 1192 0500000US23027                        Waldo County, Maine           39418
    ## 1193 0500000US23029                   Washington County, Maine           31694
    ## 1194 0500000US23031                         York County, Maine          203102
    ## 1195 0500000US24001                  Allegany County, Maryland           71977
    ## 1196 0500000US24003              Anne Arundel County, Maryland          567696
    ## 1197 0500000US24005                 Baltimore County, Maryland          827625
    ## 1198 0500000US24009                   Calvert County, Maryland           91082
    ## 1199 0500000US24011                  Caroline County, Maryland           32875
    ## 1200 0500000US24013                   Carroll County, Maryland          167522
    ## 1201 0500000US24015                     Cecil County, Maryland          102517
    ## 1202 0500000US24017                   Charles County, Maryland          157671
    ## 1203 0500000US24019                Dorchester County, Maryland           32261
    ## 1204 0500000US24021                 Frederick County, Maryland          248472
    ## 1205 0500000US24023                   Garrett County, Maryland           29376
    ## 1206 0500000US24025                   Harford County, Maryland          251025
    ## 1207 0500000US24027                    Howard County, Maryland          315327
    ## 1208 0500000US24029                      Kent County, Maryland           19593
    ## 1209 0500000US24031                Montgomery County, Maryland         1040133
    ## 1210 0500000US24033           Prince George's County, Maryland          906202
    ## 1211 0500000US24035              Queen Anne's County, Maryland           49355
    ## 1212 0500000US24037                St. Mary's County, Maryland          111531
    ## 1213 0500000US24039                  Somerset County, Maryland           25737
    ## 1214 0500000US24041                    Talbot County, Maryland           37211
    ## 1215 0500000US24043                Washington County, Maryland          149811
    ## 1216 0500000US24045                  Wicomico County, Maryland          102172
    ## 1217 0500000US24047                 Worcester County, Maryland           51564
    ## 1218 0500000US24510                   Baltimore city, Maryland          614700
    ## 1219 0500000US25001           Barnstable County, Massachusetts          213690
    ## 1220 0500000US25003            Berkshire County, Massachusetts          127328
    ## 1221 0500000US25005              Bristol County, Massachusetts          558905
    ## 1222 0500000US25007                Dukes County, Massachusetts           17313
    ## 1223 0500000US25009                Essex County, Massachusetts          781024
    ## 1224 0500000US25011             Franklin County, Massachusetts           70935
    ## 1225 0500000US25013              Hampden County, Massachusetts          469116
    ## 1226 0500000US25015            Hampshire County, Massachusetts          161159
    ## 1227 0500000US25017            Middlesex County, Massachusetts         1595192
    ## 1228 0500000US25019            Nantucket County, Massachusetts           11101
    ## 1229 0500000US25021              Norfolk County, Massachusetts          698249
    ## 1230 0500000US25023             Plymouth County, Massachusetts          512135
    ## 1231 0500000US25025              Suffolk County, Massachusetts          791766
    ## 1232 0500000US25027            Worcester County, Massachusetts          822280
    ## 1233 0500000US26001                    Alcona County, Michigan           10364
    ## 1234 0500000US26003                     Alger County, Michigan            9194
    ## 1235 0500000US26005                   Allegan County, Michigan          115250
    ## 1236 0500000US26007                    Alpena County, Michigan           28612
    ## 1237 0500000US26009                    Antrim County, Michigan           23177
    ## 1238 0500000US26011                    Arenac County, Michigan           15165
    ## 1239 0500000US26013                    Baraga County, Michigan            8507
    ## 1240 0500000US26015                     Barry County, Michigan           60057
    ## 1241 0500000US26017                       Bay County, Michigan          104786
    ## 1242 0500000US26019                    Benzie County, Michigan           17552
    ## 1243 0500000US26021                   Berrien County, Michigan          154807
    ## 1244 0500000US26023                    Branch County, Michigan           43584
    ## 1245 0500000US26025                   Calhoun County, Michigan          134473
    ## 1246 0500000US26027                      Cass County, Michigan           51460
    ## 1247 0500000US26029                Charlevoix County, Michigan           26219
    ## 1248 0500000US26031                 Cheboygan County, Michigan           25458
    ## 1249 0500000US26033                  Chippewa County, Michigan           37834
    ## 1250 0500000US26035                     Clare County, Michigan           30616
    ## 1251 0500000US26037                   Clinton County, Michigan           77896
    ## 1252 0500000US26039                  Crawford County, Michigan           13836
    ## 1253 0500000US26041                     Delta County, Michigan           36190
    ## 1254 0500000US26043                 Dickinson County, Michigan           25570
    ## 1255 0500000US26045                     Eaton County, Michigan          109155
    ## 1256 0500000US26047                     Emmet County, Michigan           33039
    ## 1257 0500000US26049                   Genesee County, Michigan          409361
    ## 1258 0500000US26051                   Gladwin County, Michigan           25289
    ## 1259 0500000US26053                   Gogebic County, Michigan           15414
    ## 1260 0500000US26055            Grand Traverse County, Michigan           91746
    ## 1261 0500000US26057                   Gratiot County, Michigan           41067
    ## 1262 0500000US26059                 Hillsdale County, Michigan           45830
    ## 1263 0500000US26061                  Houghton County, Michigan           36360
    ## 1264 0500000US26063                     Huron County, Michigan           31543
    ## 1265 0500000US26065                    Ingham County, Michigan          289564
    ## 1266 0500000US26067                     Ionia County, Michigan           64176
    ## 1267 0500000US26069                     Iosco County, Michigan           25247
    ## 1268 0500000US26071                      Iron County, Michigan           11212
    ## 1269 0500000US26073                  Isabella County, Michigan           70775
    ## 1270 0500000US26075                   Jackson County, Michigan          158913
    ## 1271 0500000US26077                 Kalamazoo County, Michigan          261573
    ## 1272 0500000US26079                  Kalkaska County, Michigan           17463
    ## 1273 0500000US26081                      Kent County, Michigan          643140
    ## 1274 0500000US26083                  Keweenaw County, Michigan            2130
    ## 1275 0500000US26085                      Lake County, Michigan           11763
    ## 1276 0500000US26087                    Lapeer County, Michigan           88202
    ## 1277 0500000US26089                  Leelanau County, Michigan           21639
    ## 1278 0500000US26091                   Lenawee County, Michigan           98474
    ## 1279 0500000US26093                Livingston County, Michigan          188482
    ## 1280 0500000US26095                      Luce County, Michigan            6364
    ## 1281 0500000US26097                  Mackinac County, Michigan           10817
    ## 1282 0500000US26099                    Macomb County, Michigan          868704
    ## 1283 0500000US26101                  Manistee County, Michigan           24444
    ## 1284 0500000US26103                 Marquette County, Michigan           66939
    ## 1285 0500000US26105                     Mason County, Michigan           28884
    ## 1286 0500000US26107                   Mecosta County, Michigan           43264
    ## 1287 0500000US26109                 Menominee County, Michigan           23234
    ## 1288 0500000US26111                   Midland County, Michigan           83389
    ## 1289 0500000US26113                 Missaukee County, Michigan           15006
    ## 1290 0500000US26115                    Monroe County, Michigan          149699
    ## 1291 0500000US26117                  Montcalm County, Michigan           63209
    ## 1292 0500000US26119               Montmorency County, Michigan            9261
    ## 1293 0500000US26121                  Muskegon County, Michigan          173043
    ## 1294 0500000US26123                   Newaygo County, Michigan           48142
    ## 1295 0500000US26125                   Oakland County, Michigan         1250843
    ## 1296 0500000US26127                    Oceana County, Michigan           26417
    ## 1297 0500000US26129                    Ogemaw County, Michigan           20928
    ## 1298 0500000US26131                 Ontonagon County, Michigan            5968
    ## 1299 0500000US26133                   Osceola County, Michigan           23232
    ## 1300 0500000US26135                    Oscoda County, Michigan            8277
    ## 1301 0500000US26137                    Otsego County, Michigan           24397
    ## 1302 0500000US26139                    Ottawa County, Michigan          284034
    ## 1303 0500000US26141              Presque Isle County, Michigan           12797
    ## 1304 0500000US26143                 Roscommon County, Michigan           23877
    ## 1305 0500000US26145                   Saginaw County, Michigan          192778
    ## 1306 0500000US26147                 St. Clair County, Michigan          159566
    ## 1307 0500000US26149                St. Joseph County, Michigan           60897
    ## 1308 0500000US26151                   Sanilac County, Michigan           41376
    ## 1309 0500000US26153               Schoolcraft County, Michigan            8069
    ## 1310 0500000US26155                Shiawassee County, Michigan           68493
    ## 1311 0500000US26157                   Tuscola County, Michigan           53250
    ## 1312 0500000US26159                 Van Buren County, Michigan           75272
    ## 1313 0500000US26161                 Washtenaw County, Michigan          365961
    ## 1314 0500000US26163                     Wayne County, Michigan         1761382
    ## 1315 0500000US26165                   Wexford County, Michigan           33111
    ## 1316 0500000US27001                   Aitkin County, Minnesota           15834
    ## 1317 0500000US27003                    Anoka County, Minnesota          347431
    ## 1318 0500000US27005                   Becker County, Minnesota           33773
    ## 1319 0500000US27007                 Beltrami County, Minnesota           46117
    ## 1320 0500000US27009                   Benton County, Minnesota           39779
    ## 1321 0500000US27011                Big Stone County, Minnesota            5016
    ## 1322 0500000US27013               Blue Earth County, Minnesota           66322
    ## 1323 0500000US27015                    Brown County, Minnesota           25211
    ## 1324 0500000US27017                  Carlton County, Minnesota           35540
    ## 1325 0500000US27019                   Carver County, Minnesota          100416
    ## 1326 0500000US27021                     Cass County, Minnesota           29022
    ## 1327 0500000US27023                 Chippewa County, Minnesota           12010
    ## 1328 0500000US27025                  Chisago County, Minnesota           54727
    ## 1329 0500000US27027                     Clay County, Minnesota           62801
    ## 1330 0500000US27029               Clearwater County, Minnesota            8812
    ## 1331 0500000US27031                     Cook County, Minnesota            5311
    ## 1332 0500000US27033               Cottonwood County, Minnesota           11372
    ## 1333 0500000US27035                Crow Wing County, Minnesota           63855
    ## 1334 0500000US27037                   Dakota County, Minnesota          418201
    ## 1335 0500000US27039                    Dodge County, Minnesota           20582
    ## 1336 0500000US27041                  Douglas County, Minnesota           37203
    ## 1337 0500000US27043                Faribault County, Minnesota           13896
    ## 1338 0500000US27045                 Fillmore County, Minnesota           20888
    ## 1339 0500000US27047                 Freeborn County, Minnesota           30526
    ## 1340 0500000US27049                  Goodhue County, Minnesota           46217
    ## 1341 0500000US27051                    Grant County, Minnesota            5938
    ## 1342 0500000US27053                 Hennepin County, Minnesota         1235478
    ## 1343 0500000US27055                  Houston County, Minnesota           18663
    ## 1344 0500000US27057                  Hubbard County, Minnesota           20862
    ## 1345 0500000US27059                   Isanti County, Minnesota           38974
    ## 1346 0500000US27061                   Itasca County, Minnesota           45203
    ## 1347 0500000US27063                  Jackson County, Minnesota           10047
    ## 1348 0500000US27065                  Kanabec County, Minnesota           16004
    ## 1349 0500000US27067                Kandiyohi County, Minnesota           42658
    ## 1350 0500000US27069                  Kittson County, Minnesota            4337
    ## 1351 0500000US27071              Koochiching County, Minnesota           12644
    ## 1352 0500000US27073            Lac qui Parle County, Minnesota            6773
    ## 1353 0500000US27075                     Lake County, Minnesota           10569
    ## 1354 0500000US27077        Lake of the Woods County, Minnesota            3809
    ## 1355 0500000US27079                 Le Sueur County, Minnesota           27983
    ## 1356 0500000US27081                  Lincoln County, Minnesota            5707
    ## 1357 0500000US27083                     Lyon County, Minnesota           25839
    ## 1358 0500000US27085                   McLeod County, Minnesota           35825
    ## 1359 0500000US27087                 Mahnomen County, Minnesota            5506
    ## 1360 0500000US27089                 Marshall County, Minnesota            9392
    ## 1361 0500000US27091                   Martin County, Minnesota           19964
    ## 1362 0500000US27093                   Meeker County, Minnesota           23079
    ## 1363 0500000US27095               Mille Lacs County, Minnesota           25728
    ## 1364 0500000US27097                 Morrison County, Minnesota           32949
    ## 1365 0500000US27099                    Mower County, Minnesota           39602
    ## 1366 0500000US27101                   Murray County, Minnesota            8353
    ## 1367 0500000US27103                 Nicollet County, Minnesota           33783
    ## 1368 0500000US27105                   Nobles County, Minnesota           21839
    ## 1369 0500000US27107                   Norman County, Minnesota            6559
    ## 1370 0500000US27109                  Olmsted County, Minnesota          153065
    ## 1371 0500000US27111               Otter Tail County, Minnesota           57992
    ## 1372 0500000US27113               Pennington County, Minnesota           14184
    ## 1373 0500000US27115                     Pine County, Minnesota           29129
    ## 1374 0500000US27117                Pipestone County, Minnesota            9185
    ## 1375 0500000US27119                     Polk County, Minnesota           31591
    ## 1376 0500000US27121                     Pope County, Minnesota           10980
    ## 1377 0500000US27123                   Ramsey County, Minnesota          541493
    ## 1378 0500000US27125                 Red Lake County, Minnesota            4008
    ## 1379 0500000US27127                  Redwood County, Minnesota           15331
    ## 1380 0500000US27129                 Renville County, Minnesota           14721
    ## 1381 0500000US27131                     Rice County, Minnesota           65765
    ## 1382 0500000US27133                     Rock County, Minnesota            9413
    ## 1383 0500000US27135                   Roseau County, Minnesota           15462
    ## 1384 0500000US27137                St. Louis County, Minnesota          200080
    ## 1385 0500000US27139                    Scott County, Minnesota          143372
    ## 1386 0500000US27141                Sherburne County, Minnesota           93231
    ## 1387 0500000US27143                   Sibley County, Minnesota           14912
    ## 1388 0500000US27145                  Stearns County, Minnesota          156819
    ## 1389 0500000US27147                   Steele County, Minnesota           36676
    ## 1390 0500000US27149                  Stevens County, Minnesota            9784
    ## 1391 0500000US27151                    Swift County, Minnesota            9411
    ## 1392 0500000US27153                     Todd County, Minnesota           24440
    ## 1393 0500000US27155                 Traverse County, Minnesota            3337
    ## 1394 0500000US27157                  Wabasha County, Minnesota           21500
    ## 1395 0500000US27159                   Wadena County, Minnesota           13646
    ## 1396 0500000US27161                   Waseca County, Minnesota           18809
    ## 1397 0500000US27163               Washington County, Minnesota          253317
    ## 1398 0500000US27165                 Watonwan County, Minnesota           10973
    ## 1399 0500000US27167                   Wilkin County, Minnesota            6343
    ## 1400 0500000US27169                   Winona County, Minnesota           50847
    ## 1401 0500000US27171                   Wright County, Minnesota          132745
    ## 1402 0500000US27173          Yellow Medicine County, Minnesota            9868
    ## 1403 0500000US28001                  Adams County, Mississippi           31547
    ## 1404 0500000US28003                 Alcorn County, Mississippi           37180
    ## 1405 0500000US28005                  Amite County, Mississippi           12468
    ## 1406 0500000US28007                 Attala County, Mississippi           18581
    ## 1407 0500000US28009                 Benton County, Mississippi            8253
    ## 1408 0500000US28011                Bolivar County, Mississippi           32592
    ## 1409 0500000US28013                Calhoun County, Mississippi           14571
    ## 1410 0500000US28015                Carroll County, Mississippi           10129
    ## 1411 0500000US28017              Chickasaw County, Mississippi           17279
    ## 1412 0500000US28019                Choctaw County, Mississippi            8321
    ## 1413 0500000US28021              Claiborne County, Mississippi            9120
    ## 1414 0500000US28023                 Clarke County, Mississippi           15928
    ## 1415 0500000US28025                   Clay County, Mississippi           19808
    ## 1416 0500000US28027                Coahoma County, Mississippi           23802
    ## 1417 0500000US28029                 Copiah County, Mississippi           28721
    ## 1418 0500000US28031              Covington County, Mississippi           19091
    ## 1419 0500000US28033                 DeSoto County, Mississippi          176132
    ## 1420 0500000US28035                Forrest County, Mississippi           75517
    ## 1421 0500000US28037               Franklin County, Mississippi            7757
    ## 1422 0500000US28039                 George County, Mississippi           23710
    ## 1423 0500000US28041                 Greene County, Mississippi           13714
    ## 1424 0500000US28043                Grenada County, Mississippi           21278
    ## 1425 0500000US28045                Hancock County, Mississippi           46653
    ## 1426 0500000US28047               Harrison County, Mississippi          202626
    ## 1427 0500000US28049                  Hinds County, Mississippi          241774
    ## 1428 0500000US28051                 Holmes County, Mississippi           18075
    ## 1429 0500000US28053              Humphreys County, Mississippi            8539
    ## 1430 0500000US28055              Issaquena County, Mississippi            1328
    ## 1431 0500000US28057               Itawamba County, Mississippi           23480
    ## 1432 0500000US28059                Jackson County, Mississippi          142014
    ## 1433 0500000US28061                 Jasper County, Mississippi           16529
    ## 1434 0500000US28063              Jefferson County, Mississippi            7346
    ## 1435 0500000US28065        Jefferson Davis County, Mississippi           11495
    ## 1436 0500000US28067                  Jones County, Mississippi           68454
    ## 1437 0500000US28069                 Kemper County, Mississippi           10107
    ## 1438 0500000US28071              Lafayette County, Mississippi           53459
    ## 1439 0500000US28073                  Lamar County, Mississippi           61223
    ## 1440 0500000US28075             Lauderdale County, Mississippi           77323
    ## 1441 0500000US28077               Lawrence County, Mississippi           12630
    ## 1442 0500000US28079                  Leake County, Mississippi           22870
    ## 1443 0500000US28081                    Lee County, Mississippi           84915
    ## 1444 0500000US28083                Leflore County, Mississippi           29804
    ## 1445 0500000US28085                Lincoln County, Mississippi           34432
    ## 1446 0500000US28087                Lowndes County, Mississippi           59437
    ## 1447 0500000US28089                Madison County, Mississippi          103498
    ## 1448 0500000US28091                 Marion County, Mississippi           25202
    ## 1449 0500000US28093               Marshall County, Mississippi           35787
    ## 1450 0500000US28095                 Monroe County, Mississippi           35840
    ## 1451 0500000US28097             Montgomery County, Mississippi           10198
    ## 1452 0500000US28099                Neshoba County, Mississippi           29376
    ## 1453 0500000US28101                 Newton County, Mississippi           21524
    ## 1454 0500000US28103                Noxubee County, Mississippi           10828
    ## 1455 0500000US28105              Oktibbeha County, Mississippi           49481
    ## 1456 0500000US28107                 Panola County, Mississippi           34243
    ## 1457 0500000US28109            Pearl River County, Mississippi           55149
    ## 1458 0500000US28111                  Perry County, Mississippi           12028
    ## 1459 0500000US28113                   Pike County, Mississippi           39737
    ## 1460 0500000US28115               Pontotoc County, Mississippi           31315
    ## 1461 0500000US28117               Prentiss County, Mississippi           25360
    ## 1462 0500000US28119                Quitman County, Mississippi            7372
    ## 1463 0500000US28121                 Rankin County, Mississippi          151240
    ## 1464 0500000US28123                  Scott County, Mississippi           28415
    ## 1465 0500000US28125                Sharkey County, Mississippi            4511
    ## 1466 0500000US28127                Simpson County, Mississippi           27073
    ## 1467 0500000US28129                  Smith County, Mississippi           16063
    ## 1468 0500000US28131                  Stone County, Mississippi           18375
    ## 1469 0500000US28133              Sunflower County, Mississippi           26532
    ## 1470 0500000US28135           Tallahatchie County, Mississippi           14361
    ## 1471 0500000US28137                   Tate County, Mississippi           28493
    ## 1472 0500000US28139                 Tippah County, Mississippi           21990
    ## 1473 0500000US28141             Tishomingo County, Mississippi           19478
    ## 1474 0500000US28143                 Tunica County, Mississippi           10170
    ## 1475 0500000US28145                  Union County, Mississippi           28356
    ## 1476 0500000US28147               Walthall County, Mississippi           14601
    ## 1477 0500000US28149                 Warren County, Mississippi           47075
    ## 1478 0500000US28151             Washington County, Mississippi           47086
    ## 1479 0500000US28153                  Wayne County, Mississippi           20422
    ## 1480 0500000US28155                Webster County, Mississippi            9828
    ## 1481 0500000US28157              Wilkinson County, Mississippi            8990
    ## 1482 0500000US28159                Winston County, Mississippi           18358
    ## 1483 0500000US28161              Yalobusha County, Mississippi           12421
    ## 1484 0500000US28163                  Yazoo County, Mississippi           27974
    ## 1485 0500000US29001                     Adair County, Missouri           25325
    ## 1486 0500000US29003                    Andrew County, Missouri           17403
    ## 1487 0500000US29005                  Atchison County, Missouri            5270
    ## 1488 0500000US29007                   Audrain County, Missouri           25735
    ## 1489 0500000US29009                     Barry County, Missouri           35493
    ## 1490 0500000US29011                    Barton County, Missouri           11850
    ## 1491 0500000US29013                     Bates County, Missouri           16374
    ## 1492 0500000US29015                    Benton County, Missouri           18989
    ## 1493 0500000US29017                 Bollinger County, Missouri           12281
    ## 1494 0500000US29019                     Boone County, Missouri          176515
    ## 1495 0500000US29021                  Buchanan County, Missouri           89076
    ## 1496 0500000US29023                    Butler County, Missouri           42733
    ## 1497 0500000US29025                  Caldwell County, Missouri            9049
    ## 1498 0500000US29027                  Callaway County, Missouri           44840
    ## 1499 0500000US29029                    Camden County, Missouri           45096
    ## 1500 0500000US29031            Cape Girardeau County, Missouri           78324
    ## 1501 0500000US29033                   Carroll County, Missouri            8843
    ## 1502 0500000US29035                    Carter County, Missouri            6197
    ## 1503 0500000US29037                      Cass County, Missouri          102678
    ## 1504 0500000US29039                     Cedar County, Missouri           13938
    ## 1505 0500000US29041                  Chariton County, Missouri            7546
    ## 1506 0500000US29043                 Christian County, Missouri           84275
    ## 1507 0500000US29045                     Clark County, Missouri            6800
    ## 1508 0500000US29047                      Clay County, Missouri          239164
    ## 1509 0500000US29049                   Clinton County, Missouri           20475
    ## 1510 0500000US29051                      Cole County, Missouri           76740
    ## 1511 0500000US29053                    Cooper County, Missouri           17622
    ## 1512 0500000US29055                  Crawford County, Missouri           24280
    ## 1513 0500000US29057                      Dade County, Missouri            7590
    ## 1514 0500000US29059                    Dallas County, Missouri           16499
    ## 1515 0500000US29061                   Daviess County, Missouri            8302
    ## 1516 0500000US29063                    DeKalb County, Missouri           12564
    ## 1517 0500000US29065                      Dent County, Missouri           15504
    ## 1518 0500000US29067                   Douglas County, Missouri           13374
    ## 1519 0500000US29069                   Dunklin County, Missouri           30428
    ## 1520 0500000US29071                  Franklin County, Missouri          102781
    ## 1521 0500000US29073                 Gasconade County, Missouri           14746
    ## 1522 0500000US29075                    Gentry County, Missouri            6665
    ## 1523 0500000US29077                    Greene County, Missouri          288429
    ## 1524 0500000US29079                    Grundy County, Missouri           10039
    ## 1525 0500000US29081                  Harrison County, Missouri            8554
    ## 1526 0500000US29083                     Henry County, Missouri           21765
    ## 1527 0500000US29085                   Hickory County, Missouri            9368
    ## 1528 0500000US29087                      Holt County, Missouri            4456
    ## 1529 0500000US29089                    Howard County, Missouri           10113
    ## 1530 0500000US29091                    Howell County, Missouri           40102
    ## 1531 0500000US29093                      Iron County, Missouri           10221
    ## 1532 0500000US29095                   Jackson County, Missouri          692003
    ## 1533 0500000US29097                    Jasper County, Missouri          119238
    ## 1534 0500000US29099                 Jefferson County, Missouri          223302
    ## 1535 0500000US29101                   Johnson County, Missouri           53689
    ## 1536 0500000US29103                      Knox County, Missouri            3951
    ## 1537 0500000US29105                   Laclede County, Missouri           35507
    ## 1538 0500000US29107                 Lafayette County, Missouri           32589
    ## 1539 0500000US29109                  Lawrence County, Missouri           38133
    ## 1540 0500000US29111                     Lewis County, Missouri           10027
    ## 1541 0500000US29113                   Lincoln County, Missouri           55563
    ## 1542 0500000US29115                      Linn County, Missouri           12186
    ## 1543 0500000US29117                Livingston County, Missouri           15076
    ## 1544 0500000US29119                  McDonald County, Missouri           22827
    ## 1545 0500000US29121                     Macon County, Missouri           15254
    ## 1546 0500000US29123                   Madison County, Missouri           12205
    ## 1547 0500000US29125                    Maries County, Missouri            8884
    ## 1548 0500000US29127                    Marion County, Missouri           28672
    ## 1549 0500000US29129                    Mercer County, Missouri            3664
    ## 1550 0500000US29131                    Miller County, Missouri           25049
    ## 1551 0500000US29133               Mississippi County, Missouri           13748
    ## 1552 0500000US29135                  Moniteau County, Missouri           15958
    ## 1553 0500000US29137                    Monroe County, Missouri            8654
    ## 1554 0500000US29139                Montgomery County, Missouri           11545
    ## 1555 0500000US29141                    Morgan County, Missouri           20137
    ## 1556 0500000US29143                New Madrid County, Missouri           17811
    ## 1557 0500000US29145                    Newton County, Missouri           58202
    ## 1558 0500000US29147                   Nodaway County, Missouri           22547
    ## 1559 0500000US29149                    Oregon County, Missouri           10699
    ## 1560 0500000US29151                     Osage County, Missouri           13619
    ## 1561 0500000US29153                     Ozark County, Missouri            9236
    ## 1562 0500000US29155                  Pemiscot County, Missouri           17031
    ## 1563 0500000US29157                     Perry County, Missouri           19146
    ## 1564 0500000US29159                    Pettis County, Missouri           42371
    ## 1565 0500000US29161                    Phelps County, Missouri           44789
    ## 1566 0500000US29163                      Pike County, Missouri           18489
    ## 1567 0500000US29165                    Platte County, Missouri           98824
    ## 1568 0500000US29167                      Polk County, Missouri           31549
    ## 1569 0500000US29169                   Pulaski County, Missouri           52591
    ## 1570 0500000US29171                    Putnam County, Missouri            4815
    ## 1571 0500000US29173                     Ralls County, Missouri           10217
    ## 1572 0500000US29175                  Randolph County, Missouri           24945
    ## 1573 0500000US29177                       Ray County, Missouri           22825
    ## 1574 0500000US29179                  Reynolds County, Missouri            6315
    ## 1575 0500000US29181                    Ripley County, Missouri           13693
    ## 1576 0500000US29183               St. Charles County, Missouri          389985
    ## 1577 0500000US29185                 St. Clair County, Missouri            9383
    ## 1578 0500000US29186            Ste. Genevieve County, Missouri           17871
    ## 1579 0500000US29187              St. Francois County, Missouri           66342
    ## 1580 0500000US29189                 St. Louis County, Missouri          998684
    ## 1581 0500000US29195                    Saline County, Missouri           23102
    ## 1582 0500000US29197                  Schuyler County, Missouri            4502
    ## 1583 0500000US29199                  Scotland County, Missouri            4898
    ## 1584 0500000US29201                     Scott County, Missouri           38729
    ## 1585 0500000US29203                   Shannon County, Missouri            8246
    ## 1586 0500000US29205                    Shelby County, Missouri            6061
    ## 1587 0500000US29207                  Stoddard County, Missouri           29512
    ## 1588 0500000US29209                     Stone County, Missouri           31527
    ## 1589 0500000US29211                  Sullivan County, Missouri            6317
    ## 1590 0500000US29213                     Taney County, Missouri           54720
    ## 1591 0500000US29215                     Texas County, Missouri           25671
    ## 1592 0500000US29217                    Vernon County, Missouri           20691
    ## 1593 0500000US29219                    Warren County, Missouri           33908
    ## 1594 0500000US29221                Washington County, Missouri           24931
    ## 1595 0500000US29223                     Wayne County, Missouri           13308
    ## 1596 0500000US29225                   Webster County, Missouri           38082
    ## 1597 0500000US29227                     Worth County, Missouri            2040
    ## 1598 0500000US29229                    Wright County, Missouri           18293
    ## 1599 0500000US29510                   St. Louis city, Missouri          311273
    ## 1600 0500000US30001                 Beaverhead County, Montana            9393
    ## 1601 0500000US30003                   Big Horn County, Montana           13376
    ## 1602 0500000US30005                     Blaine County, Montana            6727
    ## 1603 0500000US30007                 Broadwater County, Montana            5834
    ## 1604 0500000US30009                     Carbon County, Montana           10546
    ## 1605 0500000US30011                     Carter County, Montana            1318
    ## 1606 0500000US30013                    Cascade County, Montana           81746
    ## 1607 0500000US30015                   Chouteau County, Montana            5789
    ## 1608 0500000US30017                     Custer County, Montana           11845
    ## 1609 0500000US30019                    Daniels County, Montana            1753
    ## 1610 0500000US30021                     Dawson County, Montana            9191
    ## 1611 0500000US30023                 Deer Lodge County, Montana            9100
    ## 1612 0500000US30025                     Fallon County, Montana            2838
    ## 1613 0500000US30027                     Fergus County, Montana           11273
    ## 1614 0500000US30029                   Flathead County, Montana           98082
    ## 1615 0500000US30031                   Gallatin County, Montana          104729
    ## 1616 0500000US30033                   Garfield County, Montana            1141
    ## 1617 0500000US30035                    Glacier County, Montana           13699
    ## 1618 0500000US30037              Golden Valley County, Montana             724
    ## 1619 0500000US30039                    Granite County, Montana            3269
    ## 1620 0500000US30041                       Hill County, Montana           16439
    ## 1621 0500000US30043                  Jefferson County, Montana           11778
    ## 1622 0500000US30045               Judith Basin County, Montana            1951
    ## 1623 0500000US30047                       Lake County, Montana           29774
    ## 1624 0500000US30049            Lewis and Clark County, Montana           67077
    ## 1625 0500000US30051                    Liberty County, Montana            2280
    ## 1626 0500000US30053                    Lincoln County, Montana           19358
    ## 1627 0500000US30055                     McCone County, Montana            1630
    ## 1628 0500000US30057                    Madison County, Montana            8218
    ## 1629 0500000US30059                    Meagher County, Montana            1968
    ## 1630 0500000US30061                    Mineral County, Montana            4211
    ## 1631 0500000US30063                   Missoula County, Montana          115983
    ## 1632 0500000US30065                Musselshell County, Montana            4807
    ## 1633 0500000US30067                       Park County, Montana           16246
    ## 1634 0500000US30069                  Petroleum County, Montana             432
    ## 1635 0500000US30071                   Phillips County, Montana            4124
    ## 1636 0500000US30073                    Pondera County, Montana            6044
    ## 1637 0500000US30075               Powder River County, Montana            1619
    ## 1638 0500000US30077                     Powell County, Montana            6861
    ## 1639 0500000US30079                    Prairie County, Montana            1342
    ## 1640 0500000US30081                    Ravalli County, Montana           41902
    ## 1641 0500000US30083                   Richland County, Montana           11360
    ## 1642 0500000US30085                  Roosevelt County, Montana           11228
    ## 1643 0500000US30087                    Rosebud County, Montana            9250
    ## 1644 0500000US30089                    Sanders County, Montana           11521
    ## 1645 0500000US30091                   Sheridan County, Montana            3574
    ## 1646 0500000US30093                 Silver Bow County, Montana           34814
    ## 1647 0500000US30095                 Stillwater County, Montana            9410
    ## 1648 0500000US30097                Sweet Grass County, Montana            3653
    ## 1649 0500000US30099                      Teton County, Montana            6080
    ## 1650 0500000US30101                      Toole County, Montana            4976
    ## 1651 0500000US30103                   Treasure County, Montana             777
    ## 1652 0500000US30105                     Valley County, Montana            7532
    ## 1653 0500000US30107                  Wheatland County, Montana            2149
    ## 1654 0500000US30109                     Wibaux County, Montana            1175
    ## 1655 0500000US30111                Yellowstone County, Montana          157816
    ## 1656 0500000US31001                     Adams County, Nebraska           31583
    ## 1657 0500000US31003                  Antelope County, Nebraska            6372
    ## 1658 0500000US31005                    Arthur County, Nebraska             418
    ## 1659 0500000US31007                    Banner County, Nebraska             696
    ## 1660 0500000US31009                    Blaine County, Nebraska             480
    ## 1661 0500000US31011                     Boone County, Nebraska            5313
    ## 1662 0500000US31013                 Box Butte County, Nebraska           11089
    ## 1663 0500000US31015                      Boyd County, Nebraska            2042
    ## 1664 0500000US31017                     Brown County, Nebraska            2988
    ## 1665 0500000US31019                   Buffalo County, Nebraska           49030
    ## 1666 0500000US31021                      Burt County, Nebraska            6528
    ## 1667 0500000US31023                    Butler County, Nebraska            8067
    ## 1668 0500000US31025                      Cass County, Nebraska           25702
    ## 1669 0500000US31027                     Cedar County, Nebraska            8523
    ## 1670 0500000US31029                     Chase County, Nebraska            3734
    ## 1671 0500000US31031                    Cherry County, Nebraska            5790
    ## 1672 0500000US31033                  Cheyenne County, Nebraska            9852
    ## 1673 0500000US31035                      Clay County, Nebraska            6232
    ## 1674 0500000US31037                    Colfax County, Nebraska           10760
    ## 1675 0500000US31039                    Cuming County, Nebraska            8991
    ## 1676 0500000US31041                    Custer County, Nebraska           10830
    ## 1677 0500000US31043                    Dakota County, Nebraska           20317
    ## 1678 0500000US31045                     Dawes County, Nebraska            8896
    ## 1679 0500000US31047                    Dawson County, Nebraska           23804
    ## 1680 0500000US31049                     Deuel County, Nebraska            1894
    ## 1681 0500000US31051                     Dixon County, Nebraska            5746
    ## 1682 0500000US31053                     Dodge County, Nebraska           36683
    ## 1683 0500000US31055                   Douglas County, Nebraska          554992
    ## 1684 0500000US31057                     Dundy County, Nebraska            2023
    ## 1685 0500000US31059                  Fillmore County, Nebraska            5574
    ## 1686 0500000US31061                  Franklin County, Nebraska            3006
    ## 1687 0500000US31063                  Frontier County, Nebraska            2609
    ## 1688 0500000US31065                    Furnas County, Nebraska            4786
    ## 1689 0500000US31067                      Gage County, Nebraska           21595
    ## 1690 0500000US31069                    Garden County, Nebraska            1860
    ## 1691 0500000US31071                  Garfield County, Nebraska            1975
    ## 1692 0500000US31073                    Gosper County, Nebraska            2015
    ## 1693 0500000US31075                     Grant County, Nebraska             718
    ## 1694 0500000US31077                   Greeley County, Nebraska            2410
    ## 1695 0500000US31079                      Hall County, Nebraska           61343
    ## 1696 0500000US31081                  Hamilton County, Nebraska            9178
    ## 1697 0500000US31083                    Harlan County, Nebraska            3438
    ## 1698 0500000US31085                     Hayes County, Nebraska             943
    ## 1699 0500000US31087                 Hitchcock County, Nebraska            2843
    ## 1700 0500000US31089                      Holt County, Nebraska           10245
    ## 1701 0500000US31091                    Hooker County, Nebraska             691
    ## 1702 0500000US31093                    Howard County, Nebraska            6405
    ## 1703 0500000US31095                 Jefferson County, Nebraska            7188
    ## 1704 0500000US31097                   Johnson County, Nebraska            5197
    ## 1705 0500000US31099                   Kearney County, Nebraska            6552
    ## 1706 0500000US31101                     Keith County, Nebraska            8099
    ## 1707 0500000US31103                 Keya Paha County, Nebraska             792
    ## 1708 0500000US31105                   Kimball County, Nebraska            3667
    ## 1709 0500000US31107                      Knox County, Nebraska            8460
    ## 1710 0500000US31109                 Lancaster County, Nebraska          310094
    ## 1711 0500000US31111                   Lincoln County, Nebraska           35433
    ## 1712 0500000US31113                     Logan County, Nebraska             886
    ## 1713 0500000US31115                      Loup County, Nebraska             585
    ## 1714 0500000US31117                 McPherson County, Nebraska             454
    ## 1715 0500000US31119                   Madison County, Nebraska           35164
    ## 1716 0500000US31121                   Merrick County, Nebraska            7803
    ## 1717 0500000US31123                   Morrill County, Nebraska            4841
    ## 1718 0500000US31125                     Nance County, Nebraska            3554
    ## 1719 0500000US31127                    Nemaha County, Nebraska            7004
    ## 1720 0500000US31129                  Nuckolls County, Nebraska            4275
    ## 1721 0500000US31131                      Otoe County, Nebraska           15896
    ## 1722 0500000US31133                    Pawnee County, Nebraska            2676
    ## 1723 0500000US31135                   Perkins County, Nebraska            2907
    ## 1724 0500000US31137                    Phelps County, Nebraska            9120
    ## 1725 0500000US31139                    Pierce County, Nebraska            7157
    ## 1726 0500000US31141                    Platte County, Nebraska           33063
    ## 1727 0500000US31143                      Polk County, Nebraska            5255
    ## 1728 0500000US31145                Red Willow County, Nebraska           10806
    ## 1729 0500000US31147                Richardson County, Nebraska            8009
    ## 1730 0500000US31149                      Rock County, Nebraska            1350
    ## 1731 0500000US31151                    Saline County, Nebraska           14288
    ## 1732 0500000US31153                     Sarpy County, Nebraska          178351
    ## 1733 0500000US31155                  Saunders County, Nebraska           21024
    ## 1734 0500000US31157              Scotts Bluff County, Nebraska           36255
    ## 1735 0500000US31159                    Seward County, Nebraska           17127
    ## 1736 0500000US31161                  Sheridan County, Nebraska            5234
    ## 1737 0500000US31163                   Sherman County, Nebraska            3042
    ## 1738 0500000US31165                     Sioux County, Nebraska            1266
    ## 1739 0500000US31167                   Stanton County, Nebraska            5992
    ## 1740 0500000US31169                    Thayer County, Nebraska            5098
    ## 1741 0500000US31171                    Thomas County, Nebraska             645
    ## 1742 0500000US31173                  Thurston County, Nebraska            7140
    ## 1743 0500000US31175                    Valley County, Nebraska            4224
    ## 1744 0500000US31177                Washington County, Nebraska           20219
    ## 1745 0500000US31179                     Wayne County, Nebraska            9367
    ## 1746 0500000US31181                   Webster County, Nebraska            3571
    ## 1747 0500000US31183                   Wheeler County, Nebraska             822
    ## 1748 0500000US31185                      York County, Nebraska           13799
    ## 1749 0500000US32001                   Churchill County, Nevada           24010
    ## 1750 0500000US32003                       Clark County, Nevada         2141574
    ## 1751 0500000US32005                     Douglas County, Nevada           47828
    ## 1752 0500000US32007                        Elko County, Nevada           52252
    ## 1753 0500000US32009                   Esmeralda County, Nevada             981
    ## 1754 0500000US32011                      Eureka County, Nevada            1830
    ## 1755 0500000US32013                    Humboldt County, Nevada           16904
    ## 1756 0500000US32015                      Lander County, Nevada            5746
    ## 1757 0500000US32017                     Lincoln County, Nevada            5174
    ## 1758 0500000US32019                        Lyon County, Nevada           53155
    ## 1759 0500000US32021                     Mineral County, Nevada            4448
    ## 1760 0500000US32023                         Nye County, Nevada           43705
    ## 1761 0500000US32027                    Pershing County, Nevada            6611
    ## 1762 0500000US32029                      Storey County, Nevada            3941
    ## 1763 0500000US32031                      Washoe County, Nevada          450486
    ## 1764 0500000US32033                  White Pine County, Nevada            9737
    ## 1765 0500000US32510                        Carson City, Nevada           54467
    ## 1766 0500000US33001              Belknap County, New Hampshire           60640
    ## 1767 0500000US33003              Carroll County, New Hampshire           47840
    ## 1768 0500000US33005             Cheshire County, New Hampshire           76263
    ## 1769 0500000US33007                 Coos County, New Hampshire           32038
    ## 1770 0500000US33009              Grafton County, New Hampshire           89811
    ## 1771 0500000US33011         Hillsborough County, New Hampshire          411087
    ## 1772 0500000US33013            Merrimack County, New Hampshire          149452
    ## 1773 0500000US33015           Rockingham County, New Hampshire          305129
    ## 1774 0500000US33017            Strafford County, New Hampshire          128237
    ## 1775 0500000US33019             Sullivan County, New Hampshire           43125
    ## 1776 0500000US34001                Atlantic County, New Jersey          268539
    ## 1777 0500000US34003                  Bergen County, New Jersey          929999
    ## 1778 0500000US34005              Burlington County, New Jersey          446367
    ## 1779 0500000US34007                  Camden County, New Jersey          507367
    ## 1780 0500000US34009                Cape May County, New Jersey           93705
    ## 1781 0500000US34011              Cumberland County, New Jersey          153400
    ## 1782 0500000US34013                   Essex County, New Jersey          793555
    ## 1783 0500000US34015              Gloucester County, New Jersey          290852
    ## 1784 0500000US34017                  Hudson County, New Jersey          668631
    ## 1785 0500000US34019               Hunterdon County, New Jersey          125051
    ## 1786 0500000US34021                  Mercer County, New Jersey          368762
    ## 1787 0500000US34023               Middlesex County, New Jersey          826698
    ## 1788 0500000US34025                Monmouth County, New Jersey          623387
    ## 1789 0500000US34027                  Morris County, New Jersey          494383
    ## 1790 0500000US34029                   Ocean County, New Jersey          591939
    ## 1791 0500000US34031                 Passaic County, New Jersey          504041
    ## 1792 0500000US34033                   Salem County, New Jersey           63336
    ## 1793 0500000US34035                Somerset County, New Jersey          330176
    ## 1794 0500000US34037                  Sussex County, New Jersey          142298
    ## 1795 0500000US34039                   Union County, New Jersey          553066
    ## 1796 0500000US34041                  Warren County, New Jersey          106293
    ## 1797 0500000US35001              Bernalillo County, New Mexico          677692
    ## 1798 0500000US35003                  Catron County, New Mexico            3539
    ## 1799 0500000US35005                  Chaves County, New Mexico           65459
    ## 1800 0500000US35006                  Cibola County, New Mexico           26978
    ## 1801 0500000US35007                  Colfax County, New Mexico           12353
    ## 1802 0500000US35009                   Curry County, New Mexico           50199
    ## 1803 0500000US35011                 De Baca County, New Mexico            2060
    ## 1804 0500000US35013                Doña Ana County, New Mexico          215338
    ## 1805 0500000US35015                    Eddy County, New Mexico           57437
    ## 1806 0500000US35017                   Grant County, New Mexico           28061
    ## 1807 0500000US35019               Guadalupe County, New Mexico            4382
    ## 1808 0500000US35021                 Harding County, New Mexico             459
    ## 1809 0500000US35023                 Hidalgo County, New Mexico            4371
    ## 1810 0500000US35025                     Lea County, New Mexico           70126
    ## 1811 0500000US35027                 Lincoln County, New Mexico           19482
    ## 1812 0500000US35028              Los Alamos County, New Mexico           18356
    ## 1813 0500000US35029                    Luna County, New Mexico           24264
    ## 1814 0500000US35031                McKinley County, New Mexico           72849
    ## 1815 0500000US35033                    Mora County, New Mexico            4563
    ## 1816 0500000US35035                   Otero County, New Mexico           65745
    ## 1817 0500000US35037                    Quay County, New Mexico            8373
    ## 1818 0500000US35039              Rio Arriba County, New Mexico           39307
    ## 1819 0500000US35041               Roosevelt County, New Mexico           19117
    ## 1820 0500000US35043                Sandoval County, New Mexico          140769
    ## 1821 0500000US35045                San Juan County, New Mexico          127455
    ## 1822 0500000US35047              San Miguel County, New Mexico           28034
    ## 1823 0500000US35049                Santa Fe County, New Mexico          148917
    ## 1824 0500000US35051                  Sierra County, New Mexico           11135
    ## 1825 0500000US35053                 Socorro County, New Mexico           17000
    ## 1826 0500000US35055                    Taos County, New Mexico           32888
    ## 1827 0500000US35057                Torrance County, New Mexico           15595
    ## 1828 0500000US35059                   Union County, New Mexico            4175
    ## 1829 0500000US35061                Valencia County, New Mexico           75956
    ## 1830 0500000US36001                    Albany County, New York          307426
    ## 1831 0500000US36003                  Allegany County, New York           47025
    ## 1832 0500000US36005                     Bronx County, New York         1437872
    ## 1833 0500000US36007                    Broome County, New York          194402
    ## 1834 0500000US36009               Cattaraugus County, New York           77686
    ## 1835 0500000US36011                    Cayuga County, New York           77868
    ## 1836 0500000US36013                Chautauqua County, New York          129656
    ## 1837 0500000US36015                   Chemung County, New York           85740
    ## 1838 0500000US36017                  Chenango County, New York           48348
    ## 1839 0500000US36019                   Clinton County, New York           80794
    ## 1840 0500000US36021                  Columbia County, New York           60919
    ## 1841 0500000US36023                  Cortland County, New York           48123
    ## 1842 0500000US36025                  Delaware County, New York           45502
    ## 1843 0500000US36027                  Dutchess County, New York          293894
    ## 1844 0500000US36029                      Erie County, New York          919866
    ## 1845 0500000US36031                     Essex County, New York           37751
    ## 1846 0500000US36033                  Franklin County, New York           50692
    ## 1847 0500000US36035                    Fulton County, New York           53743
    ## 1848 0500000US36037                   Genesee County, New York           58112
    ## 1849 0500000US36039                    Greene County, New York           47617
    ## 1850 0500000US36041                  Hamilton County, New York            4575
    ## 1851 0500000US36043                  Herkimer County, New York           62505
    ## 1852 0500000US36045                 Jefferson County, New York          114448
    ## 1853 0500000US36047                     Kings County, New York         2600747
    ## 1854 0500000US36049                     Lewis County, New York           26719
    ## 1855 0500000US36051                Livingston County, New York           63907
    ## 1856 0500000US36053                   Madison County, New York           71359
    ## 1857 0500000US36055                    Monroe County, New York          744248
    ## 1858 0500000US36057                Montgomery County, New York           49426
    ## 1859 0500000US36059                    Nassau County, New York         1356564
    ## 1860 0500000US36061                  New York County, New York         1632480
    ## 1861 0500000US36063                   Niagara County, New York          211704
    ## 1862 0500000US36065                    Oneida County, New York          230782
    ## 1863 0500000US36067                  Onondaga County, New York          464242
    ## 1864 0500000US36069                   Ontario County, New York          109472
    ## 1865 0500000US36071                    Orange County, New York          378227
    ## 1866 0500000US36073                   Orleans County, New York           41175
    ## 1867 0500000US36075                    Oswego County, New York          119104
    ## 1868 0500000US36077                    Otsego County, New York           60244
    ## 1869 0500000US36079                    Putnam County, New York           99070
    ## 1870 0500000US36081                    Queens County, New York         2298513
    ## 1871 0500000US36083                Rensselaer County, New York          159431
    ## 1872 0500000US36085                  Richmond County, New York          474101
    ## 1873 0500000US36087                  Rockland County, New York          323686
    ## 1874 0500000US36089              St. Lawrence County, New York          109558
    ## 1875 0500000US36091                  Saratoga County, New York          227377
    ## 1876 0500000US36093               Schenectady County, New York          154883
    ## 1877 0500000US36095                 Schoharie County, New York           31364
    ## 1878 0500000US36097                  Schuyler County, New York           17992
    ## 1879 0500000US36099                    Seneca County, New York           34612
    ## 1880 0500000US36101                   Steuben County, New York           96927
    ## 1881 0500000US36103                   Suffolk County, New York         1487901
    ## 1882 0500000US36105                  Sullivan County, New York           75211
    ## 1883 0500000US36107                     Tioga County, New York           49045
    ## 1884 0500000US36109                  Tompkins County, New York          102962
    ## 1885 0500000US36111                    Ulster County, New York          179303
    ## 1886 0500000US36113                    Warren County, New York           64480
    ## 1887 0500000US36115                Washington County, New York           61828
    ## 1888 0500000US36117                     Wayne County, New York           90856
    ## 1889 0500000US36119               Westchester County, New York          968815
    ## 1890 0500000US36121                   Wyoming County, New York           40565
    ## 1891 0500000US36123                     Yates County, New York           25009
    ## 1892 0500000US37001            Alamance County, North Carolina          160576
    ## 1893 0500000US37003           Alexander County, North Carolina           37119
    ## 1894 0500000US37005           Alleghany County, North Carolina           10973
    ## 1895 0500000US37007               Anson County, North Carolina           25306
    ## 1896 0500000US37009                Ashe County, North Carolina           26786
    ## 1897 0500000US37011               Avery County, North Carolina           17501
    ## 1898 0500000US37013            Beaufort County, North Carolina           47243
    ## 1899 0500000US37015              Bertie County, North Carolina           19644
    ## 1900 0500000US37017              Bladen County, North Carolina           33778
    ## 1901 0500000US37019           Brunswick County, North Carolina          126860
    ## 1902 0500000US37021            Buncombe County, North Carolina          254474
    ## 1903 0500000US37023               Burke County, North Carolina           89712
    ## 1904 0500000US37025            Cabarrus County, North Carolina          201448
    ## 1905 0500000US37027            Caldwell County, North Carolina           81779
    ## 1906 0500000US37029              Camden County, North Carolina           10447
    ## 1907 0500000US37031            Carteret County, North Carolina           68920
    ## 1908 0500000US37033             Caswell County, North Carolina           22746
    ## 1909 0500000US37035             Catawba County, North Carolina          156729
    ## 1910 0500000US37037             Chatham County, North Carolina           69791
    ## 1911 0500000US37039            Cherokee County, North Carolina           27668
    ## 1912 0500000US37041              Chowan County, North Carolina           14205
    ## 1913 0500000US37043                Clay County, North Carolina           10813
    ## 1914 0500000US37045           Cleveland County, North Carolina           97159
    ## 1915 0500000US37047            Columbus County, North Carolina           56293
    ## 1916 0500000US37049              Craven County, North Carolina          103082
    ## 1917 0500000US37051          Cumberland County, North Carolina          332106
    ## 1918 0500000US37053           Currituck County, North Carolina           25796
    ## 1919 0500000US37055                Dare County, North Carolina           35741
    ## 1920 0500000US37057            Davidson County, North Carolina          164664
    ## 1921 0500000US37059               Davie County, North Carolina           41991
    ## 1922 0500000US37061              Duplin County, North Carolina           59062
    ## 1923 0500000US37063              Durham County, North Carolina          306457
    ## 1924 0500000US37065           Edgecombe County, North Carolina           53332
    ## 1925 0500000US37067             Forsyth County, North Carolina          371573
    ## 1926 0500000US37069            Franklin County, North Carolina           64902
    ## 1927 0500000US37071              Gaston County, North Carolina          216585
    ## 1928 0500000US37073               Gates County, North Carolina           11563
    ## 1929 0500000US37075              Graham County, North Carolina            8557
    ## 1930 0500000US37077           Granville County, North Carolina           58874
    ## 1931 0500000US37079              Greene County, North Carolina           21008
    ## 1932 0500000US37081            Guilford County, North Carolina          523582
    ## 1933 0500000US37083             Halifax County, North Carolina           51737
    ## 1934 0500000US37085             Harnett County, North Carolina          130361
    ## 1935 0500000US37087             Haywood County, North Carolina           60433
    ## 1936 0500000US37089           Henderson County, North Carolina          113625
    ## 1937 0500000US37091            Hertford County, North Carolina           24153
    ## 1938 0500000US37093                Hoke County, North Carolina           53239
    ## 1939 0500000US37095                Hyde County, North Carolina            5393
    ## 1940 0500000US37097             Iredell County, North Carolina          172525
    ## 1941 0500000US37099             Jackson County, North Carolina           42256
    ## 1942 0500000US37101            Johnston County, North Carolina          191172
    ## 1943 0500000US37103               Jones County, North Carolina            9695
    ## 1944 0500000US37105                 Lee County, North Carolina           60125
    ## 1945 0500000US37107              Lenoir County, North Carolina           57227
    ## 1946 0500000US37109             Lincoln County, North Carolina           81441
    ## 1947 0500000US37111            McDowell County, North Carolina           45109
    ## 1948 0500000US37113               Macon County, North Carolina           34410
    ## 1949 0500000US37115             Madison County, North Carolina           21405
    ## 1950 0500000US37117              Martin County, North Carolina           23054
    ## 1951 0500000US37119         Mecklenburg County, North Carolina         1054314
    ## 1952 0500000US37121            Mitchell County, North Carolina           15040
    ## 1953 0500000US37123          Montgomery County, North Carolina           27338
    ## 1954 0500000US37125               Moore County, North Carolina           95629
    ## 1955 0500000US37127                Nash County, North Carolina           94003
    ## 1956 0500000US37129         New Hanover County, North Carolina          224231
    ## 1957 0500000US37131         Northampton County, North Carolina           20186
    ## 1958 0500000US37133              Onslow County, North Carolina          193912
    ## 1959 0500000US37135              Orange County, North Carolina          142938
    ## 1960 0500000US37137             Pamlico County, North Carolina           12742
    ## 1961 0500000US37139          Pasquotank County, North Carolina           39479
    ## 1962 0500000US37141              Pender County, North Carolina           59020
    ## 1963 0500000US37143          Perquimans County, North Carolina           13459
    ## 1964 0500000US37145              Person County, North Carolina           39305
    ## 1965 0500000US37147                Pitt County, North Carolina          177372
    ## 1966 0500000US37149                Polk County, North Carolina           20458
    ## 1967 0500000US37151            Randolph County, North Carolina          142958
    ## 1968 0500000US37153            Richmond County, North Carolina           45189
    ## 1969 0500000US37155             Robeson County, North Carolina          133442
    ## 1970 0500000US37157          Rockingham County, North Carolina           91270
    ## 1971 0500000US37159               Rowan County, North Carolina          139605
    ## 1972 0500000US37161          Rutherford County, North Carolina           66532
    ## 1973 0500000US37163             Sampson County, North Carolina           63561
    ## 1974 0500000US37165            Scotland County, North Carolina           35262
    ## 1975 0500000US37167              Stanly County, North Carolina           61114
    ## 1976 0500000US37169              Stokes County, North Carolina           45905
    ## 1977 0500000US37171               Surry County, North Carolina           72099
    ## 1978 0500000US37173               Swain County, North Carolina           14254
    ## 1979 0500000US37175        Transylvania County, North Carolina           33513
    ## 1980 0500000US37177             Tyrrell County, North Carolina            4119
    ## 1981 0500000US37179               Union County, North Carolina          226694
    ## 1982 0500000US37181               Vance County, North Carolina           44482
    ## 1983 0500000US37183                Wake County, North Carolina         1046558
    ## 1984 0500000US37185              Warren County, North Carolina           20033
    ## 1985 0500000US37187          Washington County, North Carolina           12156
    ## 1986 0500000US37189             Watauga County, North Carolina           54117
    ## 1987 0500000US37191               Wayne County, North Carolina          124002
    ## 1988 0500000US37193              Wilkes County, North Carolina           68460
    ## 1989 0500000US37195              Wilson County, North Carolina           81336
    ## 1990 0500000US37197              Yadkin County, North Carolina           37665
    ## 1991 0500000US37199              Yancey County, North Carolina           17667
    ## 1992 0500000US38001                 Adams County, North Dakota            2351
    ## 1993 0500000US38003                Barnes County, North Dakota           10836
    ## 1994 0500000US38005                Benson County, North Dakota            6886
    ## 1995 0500000US38007              Billings County, North Dakota             946
    ## 1996 0500000US38009             Bottineau County, North Dakota            6589
    ## 1997 0500000US38011                Bowman County, North Dakota            3195
    ## 1998 0500000US38013                 Burke County, North Dakota            2213
    ## 1999 0500000US38015              Burleigh County, North Dakota           93737
    ## 2000 0500000US38017                  Cass County, North Dakota          174202
    ## 2001 0500000US38019              Cavalier County, North Dakota            3824
    ## 2002 0500000US38021                Dickey County, North Dakota            4970
    ## 2003 0500000US38023                Divide County, North Dakota            2369
    ## 2004 0500000US38025                  Dunn County, North Dakota            4387
    ## 2005 0500000US38027                  Eddy County, North Dakota            2313
    ## 2006 0500000US38029                Emmons County, North Dakota            3352
    ## 2007 0500000US38031                Foster County, North Dakota            3290
    ## 2008 0500000US38033         Golden Valley County, North Dakota            1882
    ## 2009 0500000US38035           Grand Forks County, North Dakota           70400
    ## 2010 0500000US38037                 Grant County, North Dakota            2380
    ## 2011 0500000US38039                Griggs County, North Dakota            2266
    ## 2012 0500000US38041             Hettinger County, North Dakota            2576
    ## 2013 0500000US38043                Kidder County, North Dakota            2460
    ## 2014 0500000US38045               LaMoure County, North Dakota            4100
    ## 2015 0500000US38047                 Logan County, North Dakota            1927
    ## 2016 0500000US38049               McHenry County, North Dakota            5927
    ## 2017 0500000US38051              McIntosh County, North Dakota            2654
    ## 2018 0500000US38053              McKenzie County, North Dakota           12536
    ## 2019 0500000US38055                McLean County, North Dakota            9608
    ## 2020 0500000US38057                Mercer County, North Dakota            8570
    ## 2021 0500000US38059                Morton County, North Dakota           30544
    ## 2022 0500000US38061             Mountrail County, North Dakota           10152
    ## 2023 0500000US38063                Nelson County, North Dakota            2920
    ## 2024 0500000US38065                Oliver County, North Dakota            1837
    ## 2025 0500000US38067               Pembina County, North Dakota            7016
    ## 2026 0500000US38069                Pierce County, North Dakota            4210
    ## 2027 0500000US38071                Ramsey County, North Dakota           11557
    ## 2028 0500000US38073                Ransom County, North Dakota            5361
    ## 2029 0500000US38075              Renville County, North Dakota            2495
    ## 2030 0500000US38077              Richland County, North Dakota           16288
    ## 2031 0500000US38079               Rolette County, North Dakota           14603
    ## 2032 0500000US38081               Sargent County, North Dakota            3883
    ## 2033 0500000US38083              Sheridan County, North Dakota            1405
    ## 2034 0500000US38085                 Sioux County, North Dakota            4413
    ## 2035 0500000US38087                 Slope County, North Dakota             704
    ## 2036 0500000US38089                 Stark County, North Dakota           30876
    ## 2037 0500000US38091                Steele County, North Dakota            1910
    ## 2038 0500000US38093              Stutsman County, North Dakota           21064
    ## 2039 0500000US38095                Towner County, North Dakota            2246
    ## 2040 0500000US38097                Traill County, North Dakota            8019
    ## 2041 0500000US38099                 Walsh County, North Dakota           10802
    ## 2042 0500000US38101                  Ward County, North Dakota           69034
    ## 2043 0500000US38103                 Wells County, North Dakota            4055
    ## 2044 0500000US38105              Williams County, North Dakota           34061
    ## 2045 0500000US39001                         Adams County, Ohio           27878
    ## 2046 0500000US39003                         Allen County, Ohio          103642
    ## 2047 0500000US39005                       Ashland County, Ohio           53477
    ## 2048 0500000US39007                     Ashtabula County, Ohio           98136
    ## 2049 0500000US39009                        Athens County, Ohio           65936
    ## 2050 0500000US39011                      Auglaize County, Ohio           45784
    ## 2051 0500000US39013                       Belmont County, Ohio           68472
    ## 2052 0500000US39015                         Brown County, Ohio           43679
    ## 2053 0500000US39017                        Butler County, Ohio          378294
    ## 2054 0500000US39019                       Carroll County, Ohio           27578
    ## 2055 0500000US39021                     Champaign County, Ohio           38864
    ## 2056 0500000US39023                         Clark County, Ohio          135198
    ## 2057 0500000US39025                      Clermont County, Ohio          203216
    ## 2058 0500000US39027                       Clinton County, Ohio           41896
    ## 2059 0500000US39029                    Columbiana County, Ohio          104003
    ## 2060 0500000US39031                     Coshocton County, Ohio           36574
    ## 2061 0500000US39033                      Crawford County, Ohio           42021
    ## 2062 0500000US39035                      Cuyahoga County, Ohio         1253783
    ## 2063 0500000US39037                         Darke County, Ohio           51734
    ## 2064 0500000US39039                      Defiance County, Ohio           38279
    ## 2065 0500000US39041                      Delaware County, Ohio          197008
    ## 2066 0500000US39043                          Erie County, Ohio           75136
    ## 2067 0500000US39045                     Fairfield County, Ohio          152910
    ## 2068 0500000US39047                       Fayette County, Ohio           28645
    ## 2069 0500000US39049                      Franklin County, Ohio         1275333
    ## 2070 0500000US39051                        Fulton County, Ohio           42305
    ## 2071 0500000US39053                        Gallia County, Ohio           30195
    ## 2072 0500000US39055                        Geauga County, Ohio           93961
    ## 2073 0500000US39057                        Greene County, Ohio          165811
    ## 2074 0500000US39059                      Guernsey County, Ohio           39274
    ## 2075 0500000US39061                      Hamilton County, Ohio          812037
    ## 2076 0500000US39063                       Hancock County, Ohio           75690
    ## 2077 0500000US39065                        Hardin County, Ohio           31542
    ## 2078 0500000US39067                      Harrison County, Ohio           15307
    ## 2079 0500000US39069                         Henry County, Ohio           27316
    ## 2080 0500000US39071                      Highland County, Ohio           43007
    ## 2081 0500000US39073                       Hocking County, Ohio           28495
    ## 2082 0500000US39075                        Holmes County, Ohio           43859
    ## 2083 0500000US39077                         Huron County, Ohio           58457
    ## 2084 0500000US39079                       Jackson County, Ohio           32524
    ## 2085 0500000US39081                     Jefferson County, Ohio           66886
    ## 2086 0500000US39083                          Knox County, Ohio           61215
    ## 2087 0500000US39085                          Lake County, Ohio          230052
    ## 2088 0500000US39087                      Lawrence County, Ohio           60622
    ## 2089 0500000US39089                       Licking County, Ohio          172293
    ## 2090 0500000US39091                         Logan County, Ohio           45307
    ## 2091 0500000US39093                        Lorain County, Ohio          306713
    ## 2092 0500000US39095                         Lucas County, Ohio          432379
    ## 2093 0500000US39097                       Madison County, Ohio           43988
    ## 2094 0500000US39099                      Mahoning County, Ohio          231064
    ## 2095 0500000US39101                        Marion County, Ohio           65344
    ## 2096 0500000US39103                        Medina County, Ohio          177257
    ## 2097 0500000US39105                         Meigs County, Ohio           23160
    ## 2098 0500000US39107                        Mercer County, Ohio           40806
    ## 2099 0500000US39109                         Miami County, Ohio          104800
    ## 2100 0500000US39111                        Monroe County, Ohio           14090
    ## 2101 0500000US39113                    Montgomery County, Ohio          532034
    ## 2102 0500000US39115                        Morgan County, Ohio           14702
    ## 2103 0500000US39117                        Morrow County, Ohio           34976
    ## 2104 0500000US39119                     Muskingum County, Ohio           86076
    ## 2105 0500000US39121                         Noble County, Ohio           14443
    ## 2106 0500000US39123                        Ottawa County, Ohio           40709
    ## 2107 0500000US39125                      Paulding County, Ohio           18872
    ## 2108 0500000US39127                         Perry County, Ohio           35985
    ## 2109 0500000US39129                      Pickaway County, Ohio           57420
    ## 2110 0500000US39131                          Pike County, Ohio           28214
    ## 2111 0500000US39133                       Portage County, Ohio          162644
    ## 2112 0500000US39135                        Preble County, Ohio           41207
    ## 2113 0500000US39137                        Putnam County, Ohio           33969
    ## 2114 0500000US39139                      Richland County, Ohio          121324
    ## 2115 0500000US39141                          Ross County, Ohio           77051
    ## 2116 0500000US39143                      Sandusky County, Ohio           59299
    ## 2117 0500000US39145                        Scioto County, Ohio           76377
    ## 2118 0500000US39147                        Seneca County, Ohio           55475
    ## 2119 0500000US39149                        Shelby County, Ohio           48797
    ## 2120 0500000US39151                         Stark County, Ohio          373475
    ## 2121 0500000US39153                        Summit County, Ohio          541810
    ## 2122 0500000US39155                      Trumbull County, Ohio          201794
    ## 2123 0500000US39157                    Tuscarawas County, Ohio           92526
    ## 2124 0500000US39159                         Union County, Ohio           55654
    ## 2125 0500000US39161                      Van Wert County, Ohio           28281
    ## 2126 0500000US39163                        Vinton County, Ohio           13111
    ## 2127 0500000US39165                        Warren County, Ohio          226564
    ## 2128 0500000US39167                    Washington County, Ohio           60671
    ## 2129 0500000US39169                         Wayne County, Ohio          116208
    ## 2130 0500000US39171                      Williams County, Ohio           36936
    ## 2131 0500000US39173                          Wood County, Ohio          129936
    ## 2132 0500000US39175                       Wyandot County, Ohio           22107
    ## 2133 0500000US40001                     Adair County, Oklahoma           22113
    ## 2134 0500000US40003                   Alfalfa County, Oklahoma            5857
    ## 2135 0500000US40005                     Atoka County, Oklahoma           13874
    ## 2136 0500000US40007                    Beaver County, Oklahoma            5415
    ## 2137 0500000US40009                   Beckham County, Oklahoma           22621
    ## 2138 0500000US40011                    Blaine County, Oklahoma            9634
    ## 2139 0500000US40013                     Bryan County, Oklahoma           45759
    ## 2140 0500000US40015                     Caddo County, Oklahoma           29342
    ## 2141 0500000US40017                  Canadian County, Oklahoma          136710
    ## 2142 0500000US40019                    Carter County, Oklahoma           48406
    ## 2143 0500000US40021                  Cherokee County, Oklahoma           48599
    ## 2144 0500000US40023                   Choctaw County, Oklahoma           14886
    ## 2145 0500000US40025                  Cimarron County, Oklahoma            2189
    ## 2146 0500000US40027                 Cleveland County, Oklahoma          276733
    ## 2147 0500000US40029                      Coal County, Oklahoma            5618
    ## 2148 0500000US40031                  Comanche County, Oklahoma          122561
    ## 2149 0500000US40033                    Cotton County, Oklahoma            5929
    ## 2150 0500000US40035                     Craig County, Oklahoma           14493
    ## 2151 0500000US40037                     Creek County, Oklahoma           71160
    ## 2152 0500000US40039                    Custer County, Oklahoma           29209
    ## 2153 0500000US40041                  Delaware County, Oklahoma           42112
    ## 2154 0500000US40043                     Dewey County, Oklahoma            4918
    ## 2155 0500000US40045                     Ellis County, Oklahoma            4072
    ## 2156 0500000US40047                  Garfield County, Oklahoma           62190
    ## 2157 0500000US40049                    Garvin County, Oklahoma           27823
    ## 2158 0500000US40051                     Grady County, Oklahoma           54733
    ## 2159 0500000US40053                     Grant County, Oklahoma            4418
    ## 2160 0500000US40055                     Greer County, Oklahoma            5943
    ## 2161 0500000US40057                    Harmon County, Oklahoma            2721
    ## 2162 0500000US40059                    Harper County, Oklahoma            3847
    ## 2163 0500000US40061                   Haskell County, Oklahoma           12704
    ## 2164 0500000US40063                    Hughes County, Oklahoma           13460
    ## 2165 0500000US40065                   Jackson County, Oklahoma           25384
    ## 2166 0500000US40067                 Jefferson County, Oklahoma            6223
    ## 2167 0500000US40069                  Johnston County, Oklahoma           11041
    ## 2168 0500000US40071                       Kay County, Oklahoma           44880
    ## 2169 0500000US40073                Kingfisher County, Oklahoma           15618
    ## 2170 0500000US40075                     Kiowa County, Oklahoma            9001
    ## 2171 0500000US40077                   Latimer County, Oklahoma           10495
    ## 2172 0500000US40079                  Le Flore County, Oklahoma           49909
    ## 2173 0500000US40081                   Lincoln County, Oklahoma           34854
    ## 2174 0500000US40083                     Logan County, Oklahoma           46044
    ## 2175 0500000US40085                      Love County, Oklahoma            9933
    ## 2176 0500000US40087                   McClain County, Oklahoma           38634
    ## 2177 0500000US40089                 McCurtain County, Oklahoma           32966
    ## 2178 0500000US40091                  McIntosh County, Oklahoma           19819
    ## 2179 0500000US40093                     Major County, Oklahoma            7718
    ## 2180 0500000US40095                  Marshall County, Oklahoma           16376
    ## 2181 0500000US40097                     Mayes County, Oklahoma           40980
    ## 2182 0500000US40099                    Murray County, Oklahoma           13875
    ## 2183 0500000US40101                  Muskogee County, Oklahoma           69084
    ## 2184 0500000US40103                     Noble County, Oklahoma           11411
    ## 2185 0500000US40105                    Nowata County, Oklahoma           10383
    ## 2186 0500000US40107                  Okfuskee County, Oklahoma           12115
    ## 2187 0500000US40109                  Oklahoma County, Oklahoma          782051
    ## 2188 0500000US40111                  Okmulgee County, Oklahoma           38889
    ## 2189 0500000US40113                     Osage County, Oklahoma           47311
    ## 2190 0500000US40115                    Ottawa County, Oklahoma           31566
    ## 2191 0500000US40117                    Pawnee County, Oklahoma           16428
    ## 2192 0500000US40119                     Payne County, Oklahoma           81512
    ## 2193 0500000US40121                 Pittsburg County, Oklahoma           44382
    ## 2194 0500000US40123                  Pontotoc County, Oklahoma           38358
    ## 2195 0500000US40125              Pottawatomie County, Oklahoma           72000
    ## 2196 0500000US40127                Pushmataha County, Oklahoma           11119
    ## 2197 0500000US40129               Roger Mills County, Oklahoma            3708
    ## 2198 0500000US40131                    Rogers County, Oklahoma           90814
    ## 2199 0500000US40133                  Seminole County, Oklahoma           25071
    ## 2200 0500000US40135                  Sequoyah County, Oklahoma           41359
    ## 2201 0500000US40137                  Stephens County, Oklahoma           43983
    ## 2202 0500000US40139                     Texas County, Oklahoma           21121
    ## 2203 0500000US40141                   Tillman County, Oklahoma            7515
    ## 2204 0500000US40143                     Tulsa County, Oklahoma          642781
    ## 2205 0500000US40145                   Wagoner County, Oklahoma           77850
    ## 2206 0500000US40147                Washington County, Oklahoma           52001
    ## 2207 0500000US40149                   Washita County, Oklahoma           11432
    ## 2208 0500000US40151                     Woods County, Oklahoma            9127
    ## 2209 0500000US40153                  Woodward County, Oklahoma           20967
    ## 2210 0500000US41001                       Baker County, Oregon           15984
    ## 2211 0500000US41003                      Benton County, Oregon           89780
    ## 2212 0500000US41005                   Clackamas County, Oregon          405788
    ## 2213 0500000US41007                     Clatsop County, Oregon           38562
    ## 2214 0500000US41009                    Columbia County, Oregon           50851
    ## 2215 0500000US41011                        Coos County, Oregon           63308
    ## 2216 0500000US41013                       Crook County, Oregon           22337
    ## 2217 0500000US41015                       Curry County, Oregon           22507
    ## 2218 0500000US41017                   Deschutes County, Oregon          180640
    ## 2219 0500000US41019                     Douglas County, Oregon          108323
    ## 2220 0500000US41021                     Gilliam County, Oregon            1907
    ## 2221 0500000US41023                       Grant County, Oregon            7183
    ## 2222 0500000US41025                      Harney County, Oregon            7228
    ## 2223 0500000US41027                  Hood River County, Oregon           23131
    ## 2224 0500000US41029                     Jackson County, Oregon          214267
    ## 2225 0500000US41031                   Jefferson County, Oregon           23143
    ## 2226 0500000US41033                   Josephine County, Oregon           85481
    ## 2227 0500000US41035                     Klamath County, Oregon           66310
    ## 2228 0500000US41037                        Lake County, Oregon            7843
    ## 2229 0500000US41039                        Lane County, Oregon          368882
    ## 2230 0500000US41041                     Lincoln County, Oregon           47881
    ## 2231 0500000US41043                        Linn County, Oregon          122870
    ## 2232 0500000US41045                     Malheur County, Oregon           30431
    ## 2233 0500000US41047                      Marion County, Oregon          335553
    ## 2234 0500000US41049                      Morrow County, Oregon           11215
    ## 2235 0500000US41051                   Multnomah County, Oregon          798647
    ## 2236 0500000US41053                        Polk County, Oregon           81427
    ## 2237 0500000US41055                     Sherman County, Oregon            1605
    ## 2238 0500000US41057                   Tillamook County, Oregon           26076
    ## 2239 0500000US41059                    Umatilla County, Oregon           76898
    ## 2240 0500000US41061                       Union County, Oregon           26028
    ## 2241 0500000US41063                     Wallowa County, Oregon            6924
    ## 2242 0500000US41065                       Wasco County, Oregon           25866
    ## 2243 0500000US41067                  Washington County, Oregon          581821
    ## 2244 0500000US41069                     Wheeler County, Oregon            1426
    ## 2245 0500000US41071                     Yamhill County, Oregon          103820
    ## 2246 0500000US42001                 Adams County, Pennsylvania          102023
    ## 2247 0500000US42003             Allegheny County, Pennsylvania         1225561
    ## 2248 0500000US42005             Armstrong County, Pennsylvania           66331
    ## 2249 0500000US42007                Beaver County, Pennsylvania          166896
    ## 2250 0500000US42009               Bedford County, Pennsylvania           48611
    ## 2251 0500000US42011                 Berks County, Pennsylvania          416642
    ## 2252 0500000US42013                 Blair County, Pennsylvania          123842
    ## 2253 0500000US42015              Bradford County, Pennsylvania           61304
    ## 2254 0500000US42017                 Bucks County, Pennsylvania          626370
    ## 2255 0500000US42019                Butler County, Pennsylvania          186566
    ## 2256 0500000US42021               Cambria County, Pennsylvania          134550
    ## 2257 0500000US42023               Cameron County, Pennsylvania            4686
    ## 2258 0500000US42025                Carbon County, Pennsylvania           63931
    ## 2259 0500000US42027                Centre County, Pennsylvania          161443
    ## 2260 0500000US42029               Chester County, Pennsylvania          517156
    ## 2261 0500000US42031               Clarion County, Pennsylvania           38827
    ## 2262 0500000US42033            Clearfield County, Pennsylvania           80216
    ## 2263 0500000US42035               Clinton County, Pennsylvania           39074
    ## 2264 0500000US42037              Columbia County, Pennsylvania           66220
    ## 2265 0500000US42039              Crawford County, Pennsylvania           86164
    ## 2266 0500000US42041            Cumberland County, Pennsylvania          247433
    ## 2267 0500000US42043               Dauphin County, Pennsylvania          274515
    ## 2268 0500000US42045              Delaware County, Pennsylvania          563527
    ## 2269 0500000US42047                   Elk County, Pennsylvania           30608
    ## 2270 0500000US42049                  Erie County, Pennsylvania          275972
    ## 2271 0500000US42051               Fayette County, Pennsylvania          132289
    ## 2272 0500000US42053                Forest County, Pennsylvania            7351
    ## 2273 0500000US42055              Franklin County, Pennsylvania          153751
    ## 2274 0500000US42057                Fulton County, Pennsylvania           14506
    ## 2275 0500000US42059                Greene County, Pennsylvania           37144
    ## 2276 0500000US42061            Huntingdon County, Pennsylvania           45421
    ## 2277 0500000US42063               Indiana County, Pennsylvania           85755
    ## 2278 0500000US42065             Jefferson County, Pennsylvania           44084
    ## 2279 0500000US42067               Juniata County, Pennsylvania           24562
    ## 2280 0500000US42069            Lackawanna County, Pennsylvania          211454
    ## 2281 0500000US42071             Lancaster County, Pennsylvania          538347
    ## 2282 0500000US42073              Lawrence County, Pennsylvania           87382
    ## 2283 0500000US42075               Lebanon County, Pennsylvania          138674
    ## 2284 0500000US42077                Lehigh County, Pennsylvania          362613
    ## 2285 0500000US42079               Luzerne County, Pennsylvania          317884
    ## 2286 0500000US42081              Lycoming County, Pennsylvania          114859
    ## 2287 0500000US42083                McKean County, Pennsylvania           41806
    ## 2288 0500000US42085                Mercer County, Pennsylvania          112630
    ## 2289 0500000US42087               Mifflin County, Pennsylvania           46362
    ## 2290 0500000US42089                Monroe County, Pennsylvania          167586
    ## 2291 0500000US42091            Montgomery County, Pennsylvania          821301
    ## 2292 0500000US42093               Montour County, Pennsylvania           18294
    ## 2293 0500000US42095           Northampton County, Pennsylvania          301778
    ## 2294 0500000US42097        Northumberland County, Pennsylvania           92325
    ## 2295 0500000US42099                 Perry County, Pennsylvania           45924
    ## 2296 0500000US42101          Philadelphia County, Pennsylvania         1575522
    ## 2297 0500000US42103                  Pike County, Pennsylvania           55498
    ## 2298 0500000US42105                Potter County, Pennsylvania           16937
    ## 2299 0500000US42107            Schuylkill County, Pennsylvania          143555
    ## 2300 0500000US42109                Snyder County, Pennsylvania           40466
    ## 2301 0500000US42111              Somerset County, Pennsylvania           74949
    ## 2302 0500000US42113              Sullivan County, Pennsylvania            6177
    ## 2303 0500000US42115           Susquehanna County, Pennsylvania           41340
    ## 2304 0500000US42117                 Tioga County, Pennsylvania           41226
    ## 2305 0500000US42119                 Union County, Pennsylvania           45114
    ## 2306 0500000US42121               Venango County, Pennsylvania           52376
    ## 2307 0500000US42123                Warren County, Pennsylvania           40035
    ## 2308 0500000US42125            Washington County, Pennsylvania          207547
    ## 2309 0500000US42127                 Wayne County, Pennsylvania           51536
    ## 2310 0500000US42129          Westmoreland County, Pennsylvania          354751
    ## 2311 0500000US42131               Wyoming County, Pennsylvania           27588
    ## 2312 0500000US42133                  York County, Pennsylvania          444014
    ## 2313 0500000US44001               Bristol County, Rhode Island           48900
    ## 2314 0500000US44003                  Kent County, Rhode Island          163861
    ## 2315 0500000US44005               Newport County, Rhode Island           83075
    ## 2316 0500000US44007            Providence County, Rhode Island          634533
    ## 2317 0500000US44009            Washington County, Rhode Island          126242
    ## 2318 0500000US45001           Abbeville County, South Carolina           24657
    ## 2319 0500000US45003               Aiken County, South Carolina          166926
    ## 2320 0500000US45005           Allendale County, South Carolina            9214
    ## 2321 0500000US45007            Anderson County, South Carolina          195995
    ## 2322 0500000US45009             Bamberg County, South Carolina           14600
    ## 2323 0500000US45011            Barnwell County, South Carolina           21577
    ## 2324 0500000US45013            Beaufort County, South Carolina          182658
    ## 2325 0500000US45015            Berkeley County, South Carolina          209065
    ## 2326 0500000US45017             Calhoun County, South Carolina           14713
    ## 2327 0500000US45019          Charleston County, South Carolina          394708
    ## 2328 0500000US45021            Cherokee County, South Carolina           56711
    ## 2329 0500000US45023             Chester County, South Carolina           32326
    ## 2330 0500000US45025        Chesterfield County, South Carolina           46024
    ## 2331 0500000US45027           Clarendon County, South Carolina           34017
    ## 2332 0500000US45029            Colleton County, South Carolina           37568
    ## 2333 0500000US45031          Darlington County, South Carolina           67253
    ## 2334 0500000US45033              Dillon County, South Carolina           30871
    ## 2335 0500000US45035          Dorchester County, South Carolina          155474
    ## 2336 0500000US45037           Edgefield County, South Carolina           26769
    ## 2337 0500000US45039           Fairfield County, South Carolina           22712
    ## 2338 0500000US45041            Florence County, South Carolina          138561
    ## 2339 0500000US45043          Georgetown County, South Carolina           61605
    ## 2340 0500000US45045          Greenville County, South Carolina          498402
    ## 2341 0500000US45047           Greenwood County, South Carolina           70264
    ## 2342 0500000US45049             Hampton County, South Carolina           19807
    ## 2343 0500000US45051               Horry County, South Carolina          320915
    ## 2344 0500000US45053              Jasper County, South Carolina           27900
    ## 2345 0500000US45055             Kershaw County, South Carolina           64361
    ## 2346 0500000US45057           Lancaster County, South Carolina           89546
    ## 2347 0500000US45059             Laurens County, South Carolina           66710
    ## 2348 0500000US45061                 Lee County, South Carolina           17606
    ## 2349 0500000US45063           Lexington County, South Carolina          286316
    ## 2350 0500000US45065           McCormick County, South Carolina            9606
    ## 2351 0500000US45067              Marion County, South Carolina           31562
    ## 2352 0500000US45069            Marlboro County, South Carolina           27131
    ## 2353 0500000US45071            Newberry County, South Carolina           38068
    ## 2354 0500000US45073              Oconee County, South Carolina           76696
    ## 2355 0500000US45075          Orangeburg County, South Carolina           88454
    ## 2356 0500000US45077             Pickens County, South Carolina          122746
    ## 2357 0500000US45079            Richland County, South Carolina          408263
    ## 2358 0500000US45081              Saluda County, South Carolina           20299
    ## 2359 0500000US45083         Spartanburg County, South Carolina          302195
    ## 2360 0500000US45085              Sumter County, South Carolina          106995
    ## 2361 0500000US45087               Union County, South Carolina           27644
    ## 2362 0500000US45089        Williamsburg County, South Carolina           31794
    ## 2363 0500000US45091                York County, South Carolina          258641
    ## 2364 0500000US46003                Aurora County, South Dakota            2759
    ## 2365 0500000US46005                Beadle County, South Dakota           18374
    ## 2366 0500000US46007               Bennett County, South Dakota            3437
    ## 2367 0500000US46009             Bon Homme County, South Dakota            6969
    ## 2368 0500000US46011             Brookings County, South Dakota           34239
    ## 2369 0500000US46013                 Brown County, South Dakota           38840
    ## 2370 0500000US46015                 Brule County, South Dakota            5256
    ## 2371 0500000US46017               Buffalo County, South Dakota            2053
    ## 2372 0500000US46019                 Butte County, South Dakota           10177
    ## 2373 0500000US46021              Campbell County, South Dakota            1435
    ## 2374 0500000US46023           Charles Mix County, South Dakota            9344
    ## 2375 0500000US46025                 Clark County, South Dakota            3673
    ## 2376 0500000US46027                  Clay County, South Dakota           13925
    ## 2377 0500000US46029             Codington County, South Dakota           27993
    ## 2378 0500000US46031                Corson County, South Dakota            4168
    ## 2379 0500000US46033                Custer County, South Dakota            8573
    ## 2380 0500000US46035               Davison County, South Dakota           19901
    ## 2381 0500000US46037                   Day County, South Dakota            5506
    ## 2382 0500000US46039                 Deuel County, South Dakota            4306
    ## 2383 0500000US46041                 Dewey County, South Dakota            5779
    ## 2384 0500000US46043               Douglas County, South Dakota            2930
    ## 2385 0500000US46045               Edmunds County, South Dakota            3940
    ## 2386 0500000US46047            Fall River County, South Dakota            6774
    ## 2387 0500000US46049                 Faulk County, South Dakota            2322
    ## 2388 0500000US46051                 Grant County, South Dakota            7217
    ## 2389 0500000US46053               Gregory County, South Dakota            4201
    ## 2390 0500000US46055                Haakon County, South Dakota            2082
    ## 2391 0500000US46057                Hamlin County, South Dakota            6000
    ## 2392 0500000US46059                  Hand County, South Dakota            3301
    ## 2393 0500000US46061                Hanson County, South Dakota            3397
    ## 2394 0500000US46063               Harding County, South Dakota            1311
    ## 2395 0500000US46065                Hughes County, South Dakota           17617
    ## 2396 0500000US46067            Hutchinson County, South Dakota            7315
    ## 2397 0500000US46069                  Hyde County, South Dakota            1331
    ## 2398 0500000US46071               Jackson County, South Dakota            3287
    ## 2399 0500000US46073               Jerauld County, South Dakota            2029
    ## 2400 0500000US46075                 Jones County, South Dakota             735
    ## 2401 0500000US46077             Kingsbury County, South Dakota            4967
    ## 2402 0500000US46079                  Lake County, South Dakota           12574
    ## 2403 0500000US46081              Lawrence County, South Dakota           25234
    ## 2404 0500000US46083               Lincoln County, South Dakota           54914
    ## 2405 0500000US46085                 Lyman County, South Dakota            3869
    ## 2406 0500000US46087                McCook County, South Dakota            5511
    ## 2407 0500000US46089             McPherson County, South Dakota            2364
    ## 2408 0500000US46091              Marshall County, South Dakota            4895
    ## 2409 0500000US46093                 Meade County, South Dakota           27424
    ## 2410 0500000US46095              Mellette County, South Dakota            2055
    ## 2411 0500000US46097                 Miner County, South Dakota            2229
    ## 2412 0500000US46099             Minnehaha County, South Dakota          186749
    ## 2413 0500000US46101                 Moody County, South Dakota            6506
    ## 2414 0500000US46102         Oglala Lakota County, South Dakota           14335
    ## 2415 0500000US46103            Pennington County, South Dakota          109294
    ## 2416 0500000US46105               Perkins County, South Dakota            2907
    ## 2417 0500000US46107                Potter County, South Dakota            2326
    ## 2418 0500000US46109               Roberts County, South Dakota           10285
    ## 2419 0500000US46111               Sanborn County, South Dakota            2388
    ## 2420 0500000US46115                 Spink County, South Dakota            6543
    ## 2421 0500000US46117               Stanley County, South Dakota            2997
    ## 2422 0500000US46119                 Sully County, South Dakota            1331
    ## 2423 0500000US46121                  Todd County, South Dakota           10146
    ## 2424 0500000US46123                 Tripp County, South Dakota            5468
    ## 2425 0500000US46125                Turner County, South Dakota            8264
    ## 2426 0500000US46127                 Union County, South Dakota           15177
    ## 2427 0500000US46129              Walworth County, South Dakota            5510
    ## 2428 0500000US46135               Yankton County, South Dakota           22717
    ## 2429 0500000US46137               Ziebach County, South Dakota            2814
    ## 2430 0500000US47001                 Anderson County, Tennessee           75775
    ## 2431 0500000US47003                  Bedford County, Tennessee           47558
    ## 2432 0500000US47005                   Benton County, Tennessee           16112
    ## 2433 0500000US47007                  Bledsoe County, Tennessee           14602
    ## 2434 0500000US47009                   Blount County, Tennessee          128443
    ## 2435 0500000US47011                  Bradley County, Tennessee          104557
    ## 2436 0500000US47013                 Campbell County, Tennessee           39687
    ## 2437 0500000US47015                   Cannon County, Tennessee           13976
    ## 2438 0500000US47017                  Carroll County, Tennessee           28018
    ## 2439 0500000US47019                   Carter County, Tennessee           56391
    ## 2440 0500000US47021                 Cheatham County, Tennessee           39929
    ## 2441 0500000US47023                  Chester County, Tennessee           17150
    ## 2442 0500000US47025                Claiborne County, Tennessee           31613
    ## 2443 0500000US47027                     Clay County, Tennessee            7686
    ## 2444 0500000US47029                    Cocke County, Tennessee           35336
    ## 2445 0500000US47031                   Coffee County, Tennessee           54531
    ## 2446 0500000US47033                 Crockett County, Tennessee           14499
    ## 2447 0500000US47035               Cumberland County, Tennessee           58634
    ## 2448 0500000US47037                 Davidson County, Tennessee          684017
    ## 2449 0500000US47039                  Decatur County, Tennessee           11683
    ## 2450 0500000US47041                   DeKalb County, Tennessee           19601
    ## 2451 0500000US47043                  Dickson County, Tennessee           51988
    ## 2452 0500000US47045                     Dyer County, Tennessee           37576
    ## 2453 0500000US47047                  Fayette County, Tennessee           39692
    ## 2454 0500000US47049                 Fentress County, Tennessee           17994
    ## 2455 0500000US47051                 Franklin County, Tennessee           41512
    ## 2456 0500000US47053                   Gibson County, Tennessee           49175
    ## 2457 0500000US47055                    Giles County, Tennessee           29167
    ## 2458 0500000US47057                 Grainger County, Tennessee           23013
    ## 2459 0500000US47059                   Greene County, Tennessee           68669
    ## 2460 0500000US47061                   Grundy County, Tennessee           13331
    ## 2461 0500000US47063                  Hamblen County, Tennessee           63740
    ## 2462 0500000US47065                 Hamilton County, Tennessee          357546
    ## 2463 0500000US47067                  Hancock County, Tennessee            6585
    ## 2464 0500000US47069                 Hardeman County, Tennessee           25562
    ## 2465 0500000US47071                   Hardin County, Tennessee           25771
    ## 2466 0500000US47073                  Hawkins County, Tennessee           56402
    ## 2467 0500000US47075                  Haywood County, Tennessee           17779
    ## 2468 0500000US47077                Henderson County, Tennessee           27859
    ## 2469 0500000US47079                    Henry County, Tennessee           32279
    ## 2470 0500000US47081                  Hickman County, Tennessee           24678
    ## 2471 0500000US47083                  Houston County, Tennessee            8176
    ## 2472 0500000US47085                Humphreys County, Tennessee           18318
    ## 2473 0500000US47087                  Jackson County, Tennessee           11615
    ## 2474 0500000US47089                Jefferson County, Tennessee           53247
    ## 2475 0500000US47091                  Johnson County, Tennessee           17789
    ## 2476 0500000US47093                     Knox County, Tennessee          456185
    ## 2477 0500000US47095                     Lake County, Tennessee            7526
    ## 2478 0500000US47097               Lauderdale County, Tennessee           26297
    ## 2479 0500000US47099                 Lawrence County, Tennessee           42937
    ## 2480 0500000US47101                    Lewis County, Tennessee           11956
    ## 2481 0500000US47103                  Lincoln County, Tennessee           33711
    ## 2482 0500000US47105                   Loudon County, Tennessee           51610
    ## 2483 0500000US47107                   McMinn County, Tennessee           52773
    ## 2484 0500000US47109                  McNairy County, Tennessee           25903
    ## 2485 0500000US47111                    Macon County, Tennessee           23487
    ## 2486 0500000US47113                  Madison County, Tennessee           97682
    ## 2487 0500000US47115                   Marion County, Tennessee           28417
    ## 2488 0500000US47117                 Marshall County, Tennessee           32269
    ## 2489 0500000US47119                    Maury County, Tennessee           89776
    ## 2490 0500000US47121                    Meigs County, Tennessee           11962
    ## 2491 0500000US47123                   Monroe County, Tennessee           45876
    ## 2492 0500000US47125               Montgomery County, Tennessee          196387
    ## 2493 0500000US47127                    Moore County, Tennessee            6322
    ## 2494 0500000US47129                   Morgan County, Tennessee           21596
    ## 2495 0500000US47131                    Obion County, Tennessee           30520
    ## 2496 0500000US47133                  Overton County, Tennessee           22004
    ## 2497 0500000US47135                    Perry County, Tennessee            7912
    ## 2498 0500000US47137                  Pickett County, Tennessee            5088
    ## 2499 0500000US47139                     Polk County, Tennessee           16782
    ## 2500 0500000US47141                   Putnam County, Tennessee           76440
    ## 2501 0500000US47143                     Rhea County, Tennessee           32628
    ## 2502 0500000US47145                    Roane County, Tennessee           52897
    ## 2503 0500000US47147                Robertson County, Tennessee           69344
    ## 2504 0500000US47149               Rutherford County, Tennessee          307128
    ## 2505 0500000US47151                    Scott County, Tennessee           21954
    ## 2506 0500000US47153               Sequatchie County, Tennessee           14730
    ## 2507 0500000US47155                   Sevier County, Tennessee           96287
    ## 2508 0500000US47157                   Shelby County, Tennessee          937005
    ## 2509 0500000US47159                    Smith County, Tennessee           19458
    ## 2510 0500000US47161                  Stewart County, Tennessee           13301
    ## 2511 0500000US47163                 Sullivan County, Tennessee          156734
    ## 2512 0500000US47165                   Sumner County, Tennessee          179473
    ## 2513 0500000US47167                   Tipton County, Tennessee           61446
    ## 2514 0500000US47169                Trousdale County, Tennessee            9573
    ## 2515 0500000US47171                   Unicoi County, Tennessee           17780
    ## 2516 0500000US47173                    Union County, Tennessee           19293
    ## 2517 0500000US47175                Van Buren County, Tennessee            5704
    ## 2518 0500000US47177                   Warren County, Tennessee           40454
    ## 2519 0500000US47179               Washington County, Tennessee          127055
    ## 2520 0500000US47181                    Wayne County, Tennessee           16649
    ## 2521 0500000US47183                  Weakley County, Tennessee           33626
    ## 2522 0500000US47185                    White County, Tennessee           26580
    ## 2523 0500000US47187               Williamson County, Tennessee          218648
    ## 2524 0500000US47189                   Wilson County, Tennessee          132663
    ## 2525 0500000US48001                     Anderson County, Texas           57863
    ## 2526 0500000US48003                      Andrews County, Texas           17818
    ## 2527 0500000US48005                     Angelina County, Texas           87607
    ## 2528 0500000US48007                      Aransas County, Texas           24763
    ## 2529 0500000US48009                       Archer County, Texas            8789
    ## 2530 0500000US48011                    Armstrong County, Texas            1916
    ## 2531 0500000US48013                     Atascosa County, Texas           48828
    ## 2532 0500000US48015                       Austin County, Texas           29565
    ## 2533 0500000US48017                       Bailey County, Texas            7092
    ## 2534 0500000US48019                      Bandera County, Texas           21763
    ## 2535 0500000US48021                      Bastrop County, Texas           82577
    ## 2536 0500000US48023                       Baylor County, Texas            3591
    ## 2537 0500000US48025                          Bee County, Texas           32691
    ## 2538 0500000US48027                         Bell County, Texas          342236
    ## 2539 0500000US48029                        Bexar County, Texas         1925865
    ## 2540 0500000US48031                       Blanco County, Texas           11279
    ## 2541 0500000US48033                       Borden County, Texas             665
    ## 2542 0500000US48035                       Bosque County, Texas           18122
    ## 2543 0500000US48037                        Bowie County, Texas           93858
    ## 2544 0500000US48039                     Brazoria County, Texas          353999
    ## 2545 0500000US48041                       Brazos County, Texas          219193
    ## 2546 0500000US48043                     Brewster County, Texas            9216
    ## 2547 0500000US48045                      Briscoe County, Texas            1546
    ## 2548 0500000US48047                       Brooks County, Texas            7180
    ## 2549 0500000US48049                        Brown County, Texas           37834
    ## 2550 0500000US48051                     Burleson County, Texas           17863
    ## 2551 0500000US48053                       Burnet County, Texas           45750
    ## 2552 0500000US48055                     Caldwell County, Texas           41401
    ## 2553 0500000US48057                      Calhoun County, Texas           21807
    ## 2554 0500000US48059                     Callahan County, Texas           13770
    ## 2555 0500000US48061                      Cameron County, Texas          421750
    ## 2556 0500000US48063                         Camp County, Texas           12813
    ## 2557 0500000US48065                       Carson County, Texas            6032
    ## 2558 0500000US48067                         Cass County, Texas           30087
    ## 2559 0500000US48069                       Castro County, Texas            7787
    ## 2560 0500000US48071                     Chambers County, Texas           40292
    ## 2561 0500000US48073                     Cherokee County, Texas           51903
    ## 2562 0500000US48075                    Childress County, Texas            7226
    ## 2563 0500000US48077                         Clay County, Texas           10387
    ## 2564 0500000US48079                      Cochran County, Texas            2904
    ## 2565 0500000US48081                         Coke County, Texas            3275
    ## 2566 0500000US48083                      Coleman County, Texas            8391
    ## 2567 0500000US48085                       Collin County, Texas          944350
    ## 2568 0500000US48087                Collingsworth County, Texas            2996
    ## 2569 0500000US48089                     Colorado County, Texas           21022
    ## 2570 0500000US48091                        Comal County, Texas          135097
    ## 2571 0500000US48093                     Comanche County, Texas           13495
    ## 2572 0500000US48095                       Concho County, Texas            4233
    ## 2573 0500000US48097                        Cooke County, Texas           39571
    ## 2574 0500000US48099                      Coryell County, Texas           75389
    ## 2575 0500000US48101                       Cottle County, Texas            1623
    ## 2576 0500000US48103                        Crane County, Texas            4839
    ## 2577 0500000US48105                     Crockett County, Texas            3633
    ## 2578 0500000US48107                       Crosby County, Texas            5861
    ## 2579 0500000US48109                    Culberson County, Texas            2241
    ## 2580 0500000US48111                       Dallam County, Texas            7243
    ## 2581 0500000US48113                       Dallas County, Texas         2586552
    ## 2582 0500000US48115                       Dawson County, Texas           12964
    ## 2583 0500000US48117                   Deaf Smith County, Texas           18899
    ## 2584 0500000US48119                        Delta County, Texas            5215
    ## 2585 0500000US48121                       Denton County, Texas          807047
    ## 2586 0500000US48123                       DeWitt County, Texas           20435
    ## 2587 0500000US48125                      Dickens County, Texas            2216
    ## 2588 0500000US48127                       Dimmit County, Texas           10663
    ## 2589 0500000US48129                       Donley County, Texas            3387
    ## 2590 0500000US48131                        Duval County, Texas           11355
    ## 2591 0500000US48133                     Eastland County, Texas           18270
    ## 2592 0500000US48135                        Ector County, Texas          158342
    ## 2593 0500000US48137                      Edwards County, Texas            2055
    ## 2594 0500000US48139                        Ellis County, Texas          168838
    ## 2595 0500000US48141                      El Paso County, Texas          837654
    ## 2596 0500000US48143                        Erath County, Texas           41482
    ## 2597 0500000US48145                        Falls County, Texas           17299
    ## 2598 0500000US48147                       Fannin County, Texas           34175
    ## 2599 0500000US48149                      Fayette County, Texas           25066
    ## 2600 0500000US48151                       Fisher County, Texas            3883
    ## 2601 0500000US48153                        Floyd County, Texas            5872
    ## 2602 0500000US48155                        Foard County, Texas            1408
    ## 2603 0500000US48157                    Fort Bend County, Texas          739342
    ## 2604 0500000US48159                     Franklin County, Texas           10679
    ## 2605 0500000US48161                    Freestone County, Texas           19709
    ## 2606 0500000US48163                         Frio County, Texas           19394
    ## 2607 0500000US48165                       Gaines County, Texas           20321
    ## 2608 0500000US48167                    Galveston County, Texas          327089
    ## 2609 0500000US48169                        Garza County, Texas            6288
    ## 2610 0500000US48171                    Gillespie County, Texas           26208
    ## 2611 0500000US48173                    Glasscock County, Texas            1430
    ## 2612 0500000US48175                       Goliad County, Texas            7531
    ## 2613 0500000US48177                     Gonzales County, Texas           20667
    ## 2614 0500000US48179                         Gray County, Texas           22685
    ## 2615 0500000US48181                      Grayson County, Texas          128560
    ## 2616 0500000US48183                        Gregg County, Texas          123494
    ## 2617 0500000US48185                       Grimes County, Texas           27630
    ## 2618 0500000US48187                    Guadalupe County, Texas          155137
    ## 2619 0500000US48189                         Hale County, Texas           34113
    ## 2620 0500000US48191                         Hall County, Texas            3074
    ## 2621 0500000US48193                     Hamilton County, Texas            8269
    ## 2622 0500000US48195                     Hansford County, Texas            5547
    ## 2623 0500000US48197                     Hardeman County, Texas            3952
    ## 2624 0500000US48199                       Hardin County, Texas           56379
    ## 2625 0500000US48201                       Harris County, Texas         4602523
    ## 2626 0500000US48203                     Harrison County, Texas           66645
    ## 2627 0500000US48205                      Hartley County, Texas            5767
    ## 2628 0500000US48207                      Haskell County, Texas            5809
    ## 2629 0500000US48209                         Hays County, Texas          204150
    ## 2630 0500000US48211                     Hemphill County, Texas            4061
    ## 2631 0500000US48213                    Henderson County, Texas           80460
    ## 2632 0500000US48215                      Hidalgo County, Texas          849389
    ## 2633 0500000US48217                         Hill County, Texas           35399
    ## 2634 0500000US48219                      Hockley County, Texas           23162
    ## 2635 0500000US48221                         Hood County, Texas           56901
    ## 2636 0500000US48223                      Hopkins County, Texas           36240
    ## 2637 0500000US48225                      Houston County, Texas           22955
    ## 2638 0500000US48227                       Howard County, Texas           36667
    ## 2639 0500000US48229                     Hudspeth County, Texas            4098
    ## 2640 0500000US48231                         Hunt County, Texas           92152
    ## 2641 0500000US48233                   Hutchinson County, Texas           21571
    ## 2642 0500000US48235                        Irion County, Texas            1524
    ## 2643 0500000US48237                         Jack County, Texas            8842
    ## 2644 0500000US48239                      Jackson County, Texas           14820
    ## 2645 0500000US48241                       Jasper County, Texas           35504
    ## 2646 0500000US48243                   Jeff Davis County, Texas            2234
    ## 2647 0500000US48245                    Jefferson County, Texas          255210
    ## 2648 0500000US48247                     Jim Hogg County, Texas            5282
    ## 2649 0500000US48249                    Jim Wells County, Texas           41192
    ## 2650 0500000US48251                      Johnson County, Texas          163475
    ## 2651 0500000US48253                        Jones County, Texas           19891
    ## 2652 0500000US48255                       Karnes County, Texas           15387
    ## 2653 0500000US48257                      Kaufman County, Texas          118910
    ## 2654 0500000US48259                      Kendall County, Texas           41982
    ## 2655 0500000US48261                       Kenedy County, Texas             595
    ## 2656 0500000US48263                         Kent County, Texas             749
    ## 2657 0500000US48265                         Kerr County, Texas           51365
    ## 2658 0500000US48267                       Kimble County, Texas            4408
    ## 2659 0500000US48269                         King County, Texas             228
    ## 2660 0500000US48271                       Kinney County, Texas            3675
    ## 2661 0500000US48273                      Kleberg County, Texas           31425
    ## 2662 0500000US48275                         Knox County, Texas            3733
    ## 2663 0500000US48277                        Lamar County, Texas           49532
    ## 2664 0500000US48279                         Lamb County, Texas           13262
    ## 2665 0500000US48281                     Lampasas County, Texas           20640
    ## 2666 0500000US48283                     La Salle County, Texas            7409
    ## 2667 0500000US48285                       Lavaca County, Texas           19941
    ## 2668 0500000US48287                          Lee County, Texas           16952
    ## 2669 0500000US48289                         Leon County, Texas           17098
    ## 2670 0500000US48291                      Liberty County, Texas           81862
    ## 2671 0500000US48293                    Limestone County, Texas           23515
    ## 2672 0500000US48295                     Lipscomb County, Texas            3469
    ## 2673 0500000US48297                     Live Oak County, Texas           12123
    ## 2674 0500000US48299                        Llano County, Texas           20640
    ## 2675 0500000US48301                       Loving County, Texas             102
    ## 2676 0500000US48303                      Lubbock County, Texas          301454
    ## 2677 0500000US48305                         Lynn County, Texas            5808
    ## 2678 0500000US48307                    McCulloch County, Texas            8098
    ## 2679 0500000US48309                     McLennan County, Texas          248429
    ## 2680 0500000US48311                     McMullen County, Texas             662
    ## 2681 0500000US48313                      Madison County, Texas           14128
    ## 2682 0500000US48315                       Marion County, Texas           10083
    ## 2683 0500000US48317                       Martin County, Texas            5614
    ## 2684 0500000US48319                        Mason County, Texas            4161
    ## 2685 0500000US48321                    Matagorda County, Texas           36743
    ## 2686 0500000US48323                     Maverick County, Texas           57970
    ## 2687 0500000US48325                       Medina County, Texas           49334
    ## 2688 0500000US48327                       Menard County, Texas            2123
    ## 2689 0500000US48329                      Midland County, Texas          164194
    ## 2690 0500000US48331                        Milam County, Texas           24664
    ## 2691 0500000US48333                        Mills County, Texas            4902
    ## 2692 0500000US48335                     Mitchell County, Texas            8558
    ## 2693 0500000US48337                     Montague County, Texas           19409
    ## 2694 0500000US48339                   Montgomery County, Texas          554445
    ## 2695 0500000US48341                        Moore County, Texas           21801
    ## 2696 0500000US48343                       Morris County, Texas           12424
    ## 2697 0500000US48345                       Motley County, Texas            1156
    ## 2698 0500000US48347                  Nacogdoches County, Texas           65558
    ## 2699 0500000US48349                      Navarro County, Texas           48583
    ## 2700 0500000US48351                       Newton County, Texas           14057
    ## 2701 0500000US48353                        Nolan County, Texas           14966
    ## 2702 0500000US48355                       Nueces County, Texas          360486
    ## 2703 0500000US48357                    Ochiltree County, Texas           10348
    ## 2704 0500000US48359                       Oldham County, Texas            2090
    ## 2705 0500000US48361                       Orange County, Texas           84047
    ## 2706 0500000US48363                   Palo Pinto County, Texas           28317
    ## 2707 0500000US48365                       Panola County, Texas           23440
    ## 2708 0500000US48367                       Parker County, Texas          129802
    ## 2709 0500000US48369                       Parmer County, Texas            9852
    ## 2710 0500000US48371                        Pecos County, Texas           15797
    ## 2711 0500000US48373                         Polk County, Texas           47837
    ## 2712 0500000US48375                       Potter County, Texas          120899
    ## 2713 0500000US48377                     Presidio County, Texas            7123
    ## 2714 0500000US48379                        Rains County, Texas           11473
    ## 2715 0500000US48381                      Randall County, Texas          132475
    ## 2716 0500000US48383                       Reagan County, Texas            3752
    ## 2717 0500000US48385                         Real County, Texas            3389
    ## 2718 0500000US48387                    Red River County, Texas           12275
    ## 2719 0500000US48389                       Reeves County, Texas           15125
    ## 2720 0500000US48391                      Refugio County, Texas            7236
    ## 2721 0500000US48393                      Roberts County, Texas             885
    ## 2722 0500000US48395                    Robertson County, Texas           16890
    ## 2723 0500000US48397                     Rockwall County, Texas           93642
    ## 2724 0500000US48399                      Runnels County, Texas           10310
    ## 2725 0500000US48401                         Rusk County, Texas           53595
    ## 2726 0500000US48403                       Sabine County, Texas           10458
    ## 2727 0500000US48405                San Augustine County, Texas            8327
    ## 2728 0500000US48407                  San Jacinto County, Texas           27819
    ## 2729 0500000US48409                 San Patricio County, Texas           67046
    ## 2730 0500000US48411                     San Saba County, Texas            5962
    ## 2731 0500000US48413                   Schleicher County, Texas            3061
    ## 2732 0500000US48415                       Scurry County, Texas           17239
    ## 2733 0500000US48417                  Shackelford County, Texas            3311
    ## 2734 0500000US48419                       Shelby County, Texas           25478
    ## 2735 0500000US48421                      Sherman County, Texas            3058
    ## 2736 0500000US48423                        Smith County, Texas          225015
    ## 2737 0500000US48425                    Somervell County, Texas            8743
    ## 2738 0500000US48427                        Starr County, Texas           63894
    ## 2739 0500000US48429                     Stephens County, Texas            9372
    ## 2740 0500000US48431                     Sterling County, Texas            1141
    ## 2741 0500000US48433                    Stonewall County, Texas            1385
    ## 2742 0500000US48435                       Sutton County, Texas            3865
    ## 2743 0500000US48437                      Swisher County, Texas            7484
    ## 2744 0500000US48439                      Tarrant County, Texas         2019977
    ## 2745 0500000US48441                       Taylor County, Texas          136348
    ## 2746 0500000US48443                      Terrell County, Texas             862
    ## 2747 0500000US48445                        Terry County, Texas           12615
    ## 2748 0500000US48447                 Throckmorton County, Texas            1567
    ## 2749 0500000US48449                        Titus County, Texas           32730
    ## 2750 0500000US48451                    Tom Green County, Texas          117466
    ## 2751 0500000US48453                       Travis County, Texas         1203166
    ## 2752 0500000US48455                      Trinity County, Texas           14569
    ## 2753 0500000US48457                        Tyler County, Texas           21496
    ## 2754 0500000US48459                       Upshur County, Texas           40769
    ## 2755 0500000US48461                        Upton County, Texas            3634
    ## 2756 0500000US48463                       Uvalde County, Texas           27009
    ## 2757 0500000US48465                    Val Verde County, Texas           49027
    ## 2758 0500000US48467                    Van Zandt County, Texas           54368
    ## 2759 0500000US48469                     Victoria County, Texas           91970
    ## 2760 0500000US48471                       Walker County, Texas           71539
    ## 2761 0500000US48473                       Waller County, Texas           49987
    ## 2762 0500000US48475                         Ward County, Texas           11586
    ## 2763 0500000US48477                   Washington County, Texas           34796
    ## 2764 0500000US48479                         Webb County, Texas          272053
    ## 2765 0500000US48481                      Wharton County, Texas           41551
    ## 2766 0500000US48483                      Wheeler County, Texas            5482
    ## 2767 0500000US48485                      Wichita County, Texas          131818
    ## 2768 0500000US48487                    Wilbarger County, Texas           12906
    ## 2769 0500000US48489                      Willacy County, Texas           21754
    ## 2770 0500000US48491                   Williamson County, Texas          527057
    ## 2771 0500000US48493                       Wilson County, Texas           48198
    ## 2772 0500000US48495                      Winkler County, Texas            7802
    ## 2773 0500000US48497                         Wise County, Texas           64639
    ## 2774 0500000US48499                         Wood County, Texas           43815
    ## 2775 0500000US48501                       Yoakum County, Texas            8571
    ## 2776 0500000US48503                        Young County, Texas           18114
    ## 2777 0500000US48505                       Zapata County, Texas           14369
    ## 2778 0500000US48507                       Zavala County, Texas           12131
    ## 2779 0500000US49001                        Beaver County, Utah            6443
    ## 2780 0500000US49003                     Box Elder County, Utah           53001
    ## 2781 0500000US49005                         Cache County, Utah          122336
    ## 2782 0500000US49007                        Carbon County, Utah           20356
    ## 2783 0500000US49009                       Daggett County, Utah             612
    ## 2784 0500000US49011                         Davis County, Utah          340621
    ## 2785 0500000US49013                      Duchesne County, Utah           20219
    ## 2786 0500000US49015                         Emery County, Utah           10248
    ## 2787 0500000US49017                      Garfield County, Utah            5017
    ## 2788 0500000US49019                         Grand County, Utah            9616
    ## 2789 0500000US49021                          Iron County, Utah           49691
    ## 2790 0500000US49023                          Juab County, Utah           10948
    ## 2791 0500000US49025                          Kane County, Utah            7350
    ## 2792 0500000US49027                       Millard County, Utah           12733
    ## 2793 0500000US49029                        Morgan County, Utah           11391
    ## 2794 0500000US49031                         Piute County, Utah            1904
    ## 2795 0500000US49033                          Rich County, Utah            2350
    ## 2796 0500000US49035                     Salt Lake County, Utah         1120805
    ## 2797 0500000US49037                      San Juan County, Utah           15281
    ## 2798 0500000US49039                       Sanpete County, Utah           29366
    ## 2799 0500000US49041                        Sevier County, Utah           21118
    ## 2800 0500000US49043                        Summit County, Utah           40511
    ## 2801 0500000US49045                        Tooele County, Utah           65185
    ## 2802 0500000US49047                        Uintah County, Utah           36323
    ## 2803 0500000US49049                          Utah County, Utah          590440
    ## 2804 0500000US49051                       Wasatch County, Utah           30523
    ## 2805 0500000US49053                    Washington County, Utah          160537
    ## 2806 0500000US49055                         Wayne County, Utah            2694
    ## 2807 0500000US49057                         Weber County, Utah          247731
    ## 2808 0500000US50001                    Addison County, Vermont           36939
    ## 2809 0500000US50003                 Bennington County, Vermont           35920
    ## 2810 0500000US50005                  Caledonia County, Vermont           30425
    ## 2811 0500000US50007                 Chittenden County, Vermont          162052
    ## 2812 0500000US50009                      Essex County, Vermont            6208
    ## 2813 0500000US50011                   Franklin County, Vermont           49025
    ## 2814 0500000US50013                 Grand Isle County, Vermont            6965
    ## 2815 0500000US50015                   Lamoille County, Vermont           25268
    ## 2816 0500000US50017                     Orange County, Vermont           28937
    ## 2817 0500000US50019                    Orleans County, Vermont           26911
    ## 2818 0500000US50021                    Rutland County, Vermont           59273
    ## 2819 0500000US50023                 Washington County, Vermont           58477
    ## 2820 0500000US50025                    Windham County, Vermont           43150
    ## 2821 0500000US50027                    Windsor County, Vermont           55427
    ## 2822 0500000US51001                  Accomack County, Virginia           32742
    ## 2823 0500000US51003                 Albemarle County, Virginia          106355
    ## 2824 0500000US51005                 Alleghany County, Virginia           15286
    ## 2825 0500000US51007                    Amelia County, Virginia           12854
    ## 2826 0500000US51009                   Amherst County, Virginia           31882
    ## 2827 0500000US51011                Appomattox County, Virginia           15577
    ## 2828 0500000US51013                 Arlington County, Virginia          231803
    ## 2829 0500000US51015                   Augusta County, Virginia           74701
    ## 2830 0500000US51017                      Bath County, Virginia            4393
    ## 2831 0500000US51019                   Bedford County, Virginia           77908
    ## 2832 0500000US51021                     Bland County, Virginia            6447
    ## 2833 0500000US51023                 Botetourt County, Virginia           33222
    ## 2834 0500000US51025                 Brunswick County, Virginia           16665
    ## 2835 0500000US51027                  Buchanan County, Virginia           22138
    ## 2836 0500000US51029                Buckingham County, Virginia           17004
    ## 2837 0500000US51031                  Campbell County, Virginia           55170
    ## 2838 0500000US51033                  Caroline County, Virginia           30184
    ## 2839 0500000US51035                   Carroll County, Virginia           29738
    ## 2840 0500000US51036              Charles City County, Virginia            6995
    ## 2841 0500000US51037                 Charlotte County, Virginia           12095
    ## 2842 0500000US51041              Chesterfield County, Virginia          339447
    ## 2843 0500000US51043                    Clarke County, Virginia           14365
    ## 2844 0500000US51045                     Craig County, Virginia            5113
    ## 2845 0500000US51047                  Culpeper County, Virginia           50450
    ## 2846 0500000US51049                Cumberland County, Virginia            9786
    ## 2847 0500000US51051                 Dickenson County, Virginia           14960
    ## 2848 0500000US51053                 Dinwiddie County, Virginia           28308
    ## 2849 0500000US51057                     Essex County, Virginia           11036
    ## 2850 0500000US51059                   Fairfax County, Virginia         1143529
    ## 2851 0500000US51061                  Fauquier County, Virginia           69115
    ## 2852 0500000US51063                     Floyd County, Virginia           15666
    ## 2853 0500000US51065                  Fluvanna County, Virginia           26282
    ## 2854 0500000US51067                  Franklin County, Virginia           56233
    ## 2855 0500000US51069                 Frederick County, Virginia           85153
    ## 2856 0500000US51071                     Giles County, Virginia           16814
    ## 2857 0500000US51073                Gloucester County, Virginia           37161
    ## 2858 0500000US51075                 Goochland County, Virginia           22482
    ## 2859 0500000US51077                   Grayson County, Virginia           15811
    ## 2860 0500000US51079                    Greene County, Virginia           19410
    ## 2861 0500000US51081               Greensville County, Virginia           11659
    ## 2862 0500000US51083                   Halifax County, Virginia           34779
    ## 2863 0500000US51085                   Hanover County, Virginia          104449
    ## 2864 0500000US51087                   Henrico County, Virginia          325642
    ## 2865 0500000US51089                     Henry County, Virginia           51588
    ## 2866 0500000US51091                  Highland County, Virginia            2214
    ## 2867 0500000US51093             Isle of Wight County, Virginia           36372
    ## 2868 0500000US51095                James City County, Virginia           74153
    ## 2869 0500000US51097            King and Queen County, Virginia            7052
    ## 2870 0500000US51099               King George County, Virginia           25890
    ## 2871 0500000US51101              King William County, Virginia           16497
    ## 2872 0500000US51103                 Lancaster County, Virginia           10804
    ## 2873 0500000US51105                       Lee County, Virginia           24134
    ## 2874 0500000US51107                   Loudoun County, Virginia          385143
    ## 2875 0500000US51109                    Louisa County, Virginia           35380
    ## 2876 0500000US51111                 Lunenburg County, Virginia           12278
    ## 2877 0500000US51113                   Madison County, Virginia           13139
    ## 2878 0500000US51115                   Mathews County, Virginia            8796
    ## 2879 0500000US51117               Mecklenburg County, Virginia           30847
    ## 2880 0500000US51119                 Middlesex County, Virginia           10700
    ## 2881 0500000US51121                Montgomery County, Virginia           97997
    ## 2882 0500000US51125                    Nelson County, Virginia           14812
    ## 2883 0500000US51127                  New Kent County, Virginia           21103
    ## 2884 0500000US51131               Northampton County, Virginia           11957
    ## 2885 0500000US51133            Northumberland County, Virginia           12223
    ## 2886 0500000US51135                  Nottoway County, Virginia           15500
    ## 2887 0500000US51137                    Orange County, Virginia           35612
    ## 2888 0500000US51139                      Page County, Virginia           23749
    ## 2889 0500000US51141                   Patrick County, Virginia           17859
    ## 2890 0500000US51143              Pittsylvania County, Virginia           61676
    ## 2891 0500000US51145                  Powhatan County, Virginia           28574
    ## 2892 0500000US51147             Prince Edward County, Virginia           22956
    ## 2893 0500000US51149             Prince George County, Virginia           37894
    ## 2894 0500000US51153            Prince William County, Virginia          456749
    ## 2895 0500000US51155                   Pulaski County, Virginia           34234
    ## 2896 0500000US51157              Rappahannock County, Virginia            7332
    ## 2897 0500000US51159                  Richmond County, Virginia            8878
    ## 2898 0500000US51161                   Roanoke County, Virginia           93583
    ## 2899 0500000US51163                Rockbridge County, Virginia           22509
    ## 2900 0500000US51165                Rockingham County, Virginia           79444
    ## 2901 0500000US51167                   Russell County, Virginia           27408
    ## 2902 0500000US51169                     Scott County, Virginia           22009
    ## 2903 0500000US51171                Shenandoah County, Virginia           43045
    ## 2904 0500000US51173                     Smyth County, Virginia           31059
    ## 2905 0500000US51175               Southampton County, Virginia           17939
    ## 2906 0500000US51177              Spotsylvania County, Virginia          131412
    ## 2907 0500000US51179                  Stafford County, Virginia          144012
    ## 2908 0500000US51181                     Surry County, Virginia            6600
    ## 2909 0500000US51183                    Sussex County, Virginia           11486
    ## 2910 0500000US51185                  Tazewell County, Virginia           42080
    ## 2911 0500000US51187                    Warren County, Virginia           39449
    ## 2912 0500000US51191                Washington County, Virginia           54406
    ## 2913 0500000US51193              Westmoreland County, Virginia           17638
    ## 2914 0500000US51195                      Wise County, Virginia           39025
    ## 2915 0500000US51197                     Wythe County, Virginia           28940
    ## 2916 0500000US51199                      York County, Virginia           67587
    ## 2917 0500000US51510                  Alexandria city, Virginia          156505
    ## 2918 0500000US51520                     Bristol city, Virginia           16843
    ## 2919 0500000US51530                 Buena Vista city, Virginia            6399
    ## 2920 0500000US51540             Charlottesville city, Virginia           47042
    ## 2921 0500000US51550                  Chesapeake city, Virginia          237820
    ## 2922 0500000US51570            Colonial Heights city, Virginia           17593
    ## 2923 0500000US51580                   Covington city, Virginia            5582
    ## 2924 0500000US51590                    Danville city, Virginia           41512
    ## 2925 0500000US51595                     Emporia city, Virginia            5381
    ## 2926 0500000US51600                     Fairfax city, Virginia           23865
    ## 2927 0500000US51610                Falls Church city, Virginia           14067
    ## 2928 0500000US51620                    Franklin city, Virginia            8211
    ## 2929 0500000US51630              Fredericksburg city, Virginia           28469
    ## 2930 0500000US51640                       Galax city, Virginia            6638
    ## 2931 0500000US51650                     Hampton city, Virginia          135583
    ## 2932 0500000US51660                Harrisonburg city, Virginia           53391
    ## 2933 0500000US51670                    Hopewell city, Virginia           22408
    ## 2934 0500000US51678                   Lexington city, Virginia            7110
    ## 2935 0500000US51680                   Lynchburg city, Virginia           80131
    ## 2936 0500000US51683                    Manassas city, Virginia           41457
    ## 2937 0500000US51685               Manassas Park city, Virginia           16423
    ## 2938 0500000US51690                Martinsville city, Virginia           13101
    ## 2939 0500000US51700                Newport News city, Virginia          180145
    ## 2940 0500000US51710                     Norfolk city, Virginia          245592
    ## 2941 0500000US51720                      Norton city, Virginia            3990
    ## 2942 0500000US51730                  Petersburg city, Virginia           31827
    ## 2943 0500000US51735                    Poquoson city, Virginia           12039
    ## 2944 0500000US51740                  Portsmouth city, Virginia           95311
    ## 2945 0500000US51750                     Radford city, Virginia           17630
    ## 2946 0500000US51760                    Richmond city, Virginia          223787
    ## 2947 0500000US51770                     Roanoke city, Virginia           99621
    ## 2948 0500000US51775                       Salem city, Virginia           25519
    ## 2949 0500000US51790                    Staunton city, Virginia           24452
    ## 2950 0500000US51800                     Suffolk city, Virginia           89160
    ## 2951 0500000US51810              Virginia Beach city, Virginia          450135
    ## 2952 0500000US51820                  Waynesboro city, Virginia           21926
    ## 2953 0500000US51830                Williamsburg city, Virginia           14788
    ## 2954 0500000US51840                  Winchester city, Virginia           27789
    ## 2955 0500000US53001                   Adams County, Washington           19452
    ## 2956 0500000US53003                  Asotin County, Washington           22337
    ## 2957 0500000US53005                  Benton County, Washington          194168
    ## 2958 0500000US53007                  Chelan County, Washington           75757
    ## 2959 0500000US53009                 Clallam County, Washington           74487
    ## 2960 0500000US53011                   Clark County, Washington          465384
    ## 2961 0500000US53013                Columbia County, Washington            4001
    ## 2962 0500000US53015                 Cowlitz County, Washington          105112
    ## 2963 0500000US53017                 Douglas County, Washington           41371
    ## 2964 0500000US53019                   Ferry County, Washington            7576
    ## 2965 0500000US53021                Franklin County, Washington           90660
    ## 2966 0500000US53023                Garfield County, Washington            2224
    ## 2967 0500000US53025                   Grant County, Washington           94860
    ## 2968 0500000US53027            Grays Harbor County, Washington           71967
    ## 2969 0500000US53029                  Island County, Washington           81636
    ## 2970 0500000US53031               Jefferson County, Washington           30856
    ## 2971 0500000US53033                    King County, Washington         2163257
    ## 2972 0500000US53035                  Kitsap County, Washington          262475
    ## 2973 0500000US53037                Kittitas County, Washington           44825
    ## 2974 0500000US53039               Klickitat County, Washington           21396
    ## 2975 0500000US53041                   Lewis County, Washington           76947
    ## 2976 0500000US53043                 Lincoln County, Washington           10435
    ## 2977 0500000US53045                   Mason County, Washington           62627
    ## 2978 0500000US53047                Okanogan County, Washington           41638
    ## 2979 0500000US53049                 Pacific County, Washington           21281
    ## 2980 0500000US53051            Pend Oreille County, Washington           13219
    ## 2981 0500000US53053                  Pierce County, Washington          859840
    ## 2982 0500000US53055                San Juan County, Washington           16473
    ## 2983 0500000US53057                  Skagit County, Washington          123907
    ## 2984 0500000US53059                Skamania County, Washington           11620
    ## 2985 0500000US53061               Snohomish County, Washington          786620
    ## 2986 0500000US53063                 Spokane County, Washington          497875
    ## 2987 0500000US53065                 Stevens County, Washington           44214
    ## 2988 0500000US53067                Thurston County, Washington          274684
    ## 2989 0500000US53069               Wahkiakum County, Washington            4189
    ## 2990 0500000US53071             Walla Walla County, Washington           60236
    ## 2991 0500000US53073                 Whatcom County, Washington          216812
    ## 2992 0500000US53075                 Whitman County, Washington           48593
    ## 2993 0500000US53077                  Yakima County, Washington          249325
    ## 2994 0500000US54001              Barbour County, West Virginia           16730
    ## 2995 0500000US54003             Berkeley County, West Virginia          113495
    ## 2996 0500000US54005                Boone County, West Virginia           22817
    ## 2997 0500000US54007              Braxton County, West Virginia           14282
    ## 2998 0500000US54009               Brooke County, West Virginia           22772
    ## 2999 0500000US54011               Cabell County, West Virginia           95318
    ## 3000 0500000US54013              Calhoun County, West Virginia            7396
    ## 3001 0500000US54015                 Clay County, West Virginia            8785
    ## 3002 0500000US54017            Doddridge County, West Virginia            8536
    ## 3003 0500000US54019              Fayette County, West Virginia           44126
    ## 3004 0500000US54021               Gilmer County, West Virginia            8205
    ## 3005 0500000US54023                Grant County, West Virginia           11641
    ## 3006 0500000US54025           Greenbrier County, West Virginia           35347
    ## 3007 0500000US54027            Hampshire County, West Virginia           23363
    ## 3008 0500000US54029              Hancock County, West Virginia           29680
    ## 3009 0500000US54031                Hardy County, West Virginia           13842
    ## 3010 0500000US54033             Harrison County, West Virginia           68209
    ## 3011 0500000US54035              Jackson County, West Virginia           29018
    ## 3012 0500000US54037            Jefferson County, West Virginia           56179
    ## 3013 0500000US54039              Kanawha County, West Virginia          185710
    ## 3014 0500000US54041                Lewis County, West Virginia           16276
    ## 3015 0500000US54043              Lincoln County, West Virginia           21078
    ## 3016 0500000US54045                Logan County, West Virginia           33801
    ## 3017 0500000US54047             McDowell County, West Virginia           19217
    ## 3018 0500000US54049               Marion County, West Virginia           56497
    ## 3019 0500000US54051             Marshall County, West Virginia           31645
    ## 3020 0500000US54053                Mason County, West Virginia           26939
    ## 3021 0500000US54055               Mercer County, West Virginia           60486
    ## 3022 0500000US54057              Mineral County, West Virginia           27278
    ## 3023 0500000US54059                Mingo County, West Virginia           24741
    ## 3024 0500000US54061           Monongalia County, West Virginia          105252
    ## 3025 0500000US54063               Monroe County, West Virginia           13467
    ## 3026 0500000US54065               Morgan County, West Virginia           17624
    ## 3027 0500000US54067             Nicholas County, West Virginia           25324
    ## 3028 0500000US54069                 Ohio County, West Virginia           42547
    ## 3029 0500000US54071            Pendleton County, West Virginia            7056
    ## 3030 0500000US54073            Pleasants County, West Virginia            7507
    ## 3031 0500000US54075           Pocahontas County, West Virginia            8531
    ## 3032 0500000US54077              Preston County, West Virginia           33837
    ## 3033 0500000US54079               Putnam County, West Virginia           56652
    ## 3034 0500000US54081              Raleigh County, West Virginia           76232
    ## 3035 0500000US54083             Randolph County, West Virginia           29065
    ## 3036 0500000US54085              Ritchie County, West Virginia            9932
    ## 3037 0500000US54087                Roane County, West Virginia           14205
    ## 3038 0500000US54089              Summers County, West Virginia           13018
    ## 3039 0500000US54091               Taylor County, West Virginia           16951
    ## 3040 0500000US54093               Tucker County, West Virginia            7027
    ## 3041 0500000US54095                Tyler County, West Virginia            8909
    ## 3042 0500000US54097               Upshur County, West Virginia           24605
    ## 3043 0500000US54099                Wayne County, West Virginia           40708
    ## 3044 0500000US54101              Webster County, West Virginia            8518
    ## 3045 0500000US54103               Wetzel County, West Virginia           15614
    ## 3046 0500000US54105                 Wirt County, West Virginia            5797
    ## 3047 0500000US54107                 Wood County, West Virginia           85556
    ## 3048 0500000US54109              Wyoming County, West Virginia           21711
    ## 3049 0500000US55001                    Adams County, Wisconsin           20073
    ## 3050 0500000US55003                  Ashland County, Wisconsin           15712
    ## 3051 0500000US55005                   Barron County, Wisconsin           45252
    ## 3052 0500000US55007                 Bayfield County, Wisconsin           14992
    ## 3053 0500000US55009                    Brown County, Wisconsin          259786
    ## 3054 0500000US55011                  Buffalo County, Wisconsin           13167
    ## 3055 0500000US55013                  Burnett County, Wisconsin           15258
    ## 3056 0500000US55015                  Calumet County, Wisconsin           49807
    ## 3057 0500000US55017                 Chippewa County, Wisconsin           63635
    ## 3058 0500000US55019                    Clark County, Wisconsin           34491
    ## 3059 0500000US55021                 Columbia County, Wisconsin           56954
    ## 3060 0500000US55023                 Crawford County, Wisconsin           16288
    ## 3061 0500000US55025                     Dane County, Wisconsin          529843
    ## 3062 0500000US55027                    Dodge County, Wisconsin           87776
    ## 3063 0500000US55029                     Door County, Wisconsin           27439
    ## 3064 0500000US55031                  Douglas County, Wisconsin           43402
    ## 3065 0500000US55033                     Dunn County, Wisconsin           44498
    ## 3066 0500000US55035               Eau Claire County, Wisconsin          102991
    ## 3067 0500000US55037                 Florence County, Wisconsin            4337
    ## 3068 0500000US55039              Fond du Lac County, Wisconsin          102315
    ## 3069 0500000US55041                   Forest County, Wisconsin            9018
    ## 3070 0500000US55043                    Grant County, Wisconsin           51828
    ## 3071 0500000US55045                    Green County, Wisconsin           36864
    ## 3072 0500000US55047               Green Lake County, Wisconsin           18757
    ## 3073 0500000US55049                     Iowa County, Wisconsin           23620
    ## 3074 0500000US55051                     Iron County, Wisconsin            5715
    ## 3075 0500000US55053                  Jackson County, Wisconsin           20506
    ## 3076 0500000US55055                Jefferson County, Wisconsin           84652
    ## 3077 0500000US55057                   Juneau County, Wisconsin           26419
    ## 3078 0500000US55059                  Kenosha County, Wisconsin          168330
    ## 3079 0500000US55061                 Kewaunee County, Wisconsin           20360
    ## 3080 0500000US55063                La Crosse County, Wisconsin          117850
    ## 3081 0500000US55065                Lafayette County, Wisconsin           16735
    ## 3082 0500000US55067                 Langlade County, Wisconsin           19164
    ## 3083 0500000US55069                  Lincoln County, Wisconsin           27848
    ## 3084 0500000US55071                Manitowoc County, Wisconsin           79407
    ## 3085 0500000US55073                 Marathon County, Wisconsin          135264
    ## 3086 0500000US55075                Marinette County, Wisconsin           40537
    ## 3087 0500000US55077                Marquette County, Wisconsin           15207
    ## 3088 0500000US55078                Menominee County, Wisconsin            4579
    ## 3089 0500000US55079                Milwaukee County, Wisconsin          954209
    ## 3090 0500000US55081                   Monroe County, Wisconsin           45502
    ## 3091 0500000US55083                   Oconto County, Wisconsin           37556
    ## 3092 0500000US55085                   Oneida County, Wisconsin           35345
    ## 3093 0500000US55087                Outagamie County, Wisconsin          184754
    ## 3094 0500000US55089                  Ozaukee County, Wisconsin           88284
    ## 3095 0500000US55091                    Pepin County, Wisconsin            7262
    ## 3096 0500000US55093                   Pierce County, Wisconsin           41603
    ## 3097 0500000US55095                     Polk County, Wisconsin           43349
    ## 3098 0500000US55097                  Portage County, Wisconsin           70599
    ## 3099 0500000US55099                    Price County, Wisconsin           13490
    ## 3100 0500000US55101                   Racine County, Wisconsin          195398
    ## 3101 0500000US55103                 Richland County, Wisconsin           17539
    ## 3102 0500000US55105                     Rock County, Wisconsin          161769
    ## 3103 0500000US55107                     Rusk County, Wisconsin           14183
    ## 3104 0500000US55109                St. Croix County, Wisconsin           87917
    ## 3105 0500000US55111                     Sauk County, Wisconsin           63596
    ## 3106 0500000US55113                   Sawyer County, Wisconsin           16370
    ## 3107 0500000US55115                  Shawano County, Wisconsin           41009
    ## 3108 0500000US55117                Sheboygan County, Wisconsin          115205
    ## 3109 0500000US55119                   Taylor County, Wisconsin           20356
    ## 3110 0500000US55121              Trempealeau County, Wisconsin           29438
    ## 3111 0500000US55123                   Vernon County, Wisconsin           30516
    ## 3112 0500000US55125                    Vilas County, Wisconsin           21593
    ## 3113 0500000US55127                 Walworth County, Wisconsin          103013
    ## 3114 0500000US55129                 Washburn County, Wisconsin           15689
    ## 3115 0500000US55131               Washington County, Wisconsin          134535
    ## 3116 0500000US55133                 Waukesha County, Wisconsin          398879
    ## 3117 0500000US55135                  Waupaca County, Wisconsin           51444
    ## 3118 0500000US55137                 Waushara County, Wisconsin           24116
    ## 3119 0500000US55139                Winnebago County, Wisconsin          169926
    ## 3120 0500000US55141                     Wood County, Wisconsin           73274
    ## 3121 0500000US56001                     Albany County, Wyoming           38102
    ## 3122 0500000US56003                   Big Horn County, Wyoming           11901
    ## 3123 0500000US56005                   Campbell County, Wyoming           47708
    ## 3124 0500000US56007                     Carbon County, Wyoming           15477
    ## 3125 0500000US56009                   Converse County, Wyoming           13997
    ## 3126 0500000US56011                      Crook County, Wyoming            7410
    ## 3127 0500000US56013                    Fremont County, Wyoming           40076
    ## 3128 0500000US56015                     Goshen County, Wyoming           13438
    ## 3129 0500000US56017                Hot Springs County, Wyoming            4680
    ## 3130 0500000US56019                    Johnson County, Wyoming            8515
    ## 3131 0500000US56021                    Laramie County, Wyoming           97692
    ## 3132 0500000US56023                    Lincoln County, Wyoming           19011
    ## 3133 0500000US56025                    Natrona County, Wyoming           80610
    ## 3134 0500000US56027                   Niobrara County, Wyoming            2448
    ## 3135 0500000US56029                       Park County, Wyoming           29121
    ## 3136 0500000US56031                     Platte County, Wyoming            8673
    ## 3137 0500000US56033                   Sheridan County, Wyoming           30012
    ## 3138 0500000US56035                   Sublette County, Wyoming            9951
    ## 3139 0500000US56037                 Sweetwater County, Wyoming           44117
    ## 3140 0500000US56039                      Teton County, Wyoming           23059
    ## 3141 0500000US56041                      Uinta County, Wyoming           20609
    ## 3142 0500000US56043                   Washakie County, Wyoming            8129
    ## 3143 0500000US56045                     Weston County, Wyoming            7100
    ## 3144 0500000US72001            Adjuntas Municipio, Puerto Rico           18181
    ## 3145 0500000US72003              Aguada Municipio, Puerto Rico           38643
    ## 3146 0500000US72005           Aguadilla Municipio, Puerto Rico           54166
    ## 3147 0500000US72007        Aguas Buenas Municipio, Puerto Rico           26275
    ## 3148 0500000US72009            Aibonito Municipio, Puerto Rico           23457
    ## 3149 0500000US72011              Añasco Municipio, Puerto Rico           27368
    ## 3150 0500000US72013             Arecibo Municipio, Puerto Rico           87242
    ## 3151 0500000US72015              Arroyo Municipio, Puerto Rico           18111
    ## 3152 0500000US72017         Barceloneta Municipio, Puerto Rico           24299
    ## 3153 0500000US72019        Barranquitas Municipio, Puerto Rico           28755
    ## 3154 0500000US72021             Bayamón Municipio, Puerto Rico          182955
    ## 3155 0500000US72023           Cabo Rojo Municipio, Puerto Rico           49005
    ## 3156 0500000US72025              Caguas Municipio, Puerto Rico          131363
    ## 3157 0500000US72027               Camuy Municipio, Puerto Rico           32222
    ## 3158 0500000US72029           Canóvanas Municipio, Puerto Rico           46108
    ## 3159 0500000US72031            Carolina Municipio, Puerto Rico          157453
    ## 3160 0500000US72033              Cataño Municipio, Puerto Rico           24888
    ## 3161 0500000US72035               Cayey Municipio, Puerto Rico           44530
    ## 3162 0500000US72037               Ceiba Municipio, Puerto Rico           11853
    ## 3163 0500000US72039              Ciales Municipio, Puerto Rico           16912
    ## 3164 0500000US72041               Cidra Municipio, Puerto Rico           40343
    ## 3165 0500000US72043               Coamo Municipio, Puerto Rico           39265
    ## 3166 0500000US72045             Comerío Municipio, Puerto Rico           19539
    ## 3167 0500000US72047             Corozal Municipio, Puerto Rico           34165
    ## 3168 0500000US72049             Culebra Municipio, Puerto Rico            1314
    ## 3169 0500000US72051              Dorado Municipio, Puerto Rico           37208
    ## 3170 0500000US72053             Fajardo Municipio, Puerto Rico           32001
    ## 3171 0500000US72054             Florida Municipio, Puerto Rico           11910
    ## 3172 0500000US72055             Guánica Municipio, Puerto Rico           16783
    ## 3173 0500000US72057             Guayama Municipio, Puerto Rico           41706
    ## 3174 0500000US72059          Guayanilla Municipio, Puerto Rico           19008
    ## 3175 0500000US72061            Guaynabo Municipio, Puerto Rico           88663
    ## 3176 0500000US72063              Gurabo Municipio, Puerto Rico           46894
    ## 3177 0500000US72065             Hatillo Municipio, Puerto Rico           40390
    ## 3178 0500000US72067         Hormigueros Municipio, Puerto Rico           16180
    ## 3179 0500000US72069             Humacao Municipio, Puerto Rico           53466
    ## 3180 0500000US72071             Isabela Municipio, Puerto Rico           42420
    ## 3181 0500000US72073              Jayuya Municipio, Puerto Rico           14906
    ## 3182 0500000US72075          Juana Díaz Municipio, Puerto Rico           46960
    ## 3183 0500000US72077              Juncos Municipio, Puerto Rico           39128
    ## 3184 0500000US72079               Lajas Municipio, Puerto Rico           23315
    ## 3185 0500000US72081               Lares Municipio, Puerto Rico           26451
    ## 3186 0500000US72083          Las Marías Municipio, Puerto Rico            8599
    ## 3187 0500000US72085         Las Piedras Municipio, Puerto Rico           37768
    ## 3188 0500000US72087               Loíza Municipio, Puerto Rico           26463
    ## 3189 0500000US72089            Luquillo Municipio, Puerto Rico           18547
    ## 3190 0500000US72091              Manatí Municipio, Puerto Rico           39692
    ## 3191 0500000US72093             Maricao Municipio, Puerto Rico            6202
    ## 3192 0500000US72095             Maunabo Municipio, Puerto Rico           11023
    ## 3193 0500000US72097            Mayagüez Municipio, Puerto Rico           77255
    ## 3194 0500000US72099                Moca Municipio, Puerto Rico           36872
    ## 3195 0500000US72101             Morovis Municipio, Puerto Rico           31320
    ## 3196 0500000US72103             Naguabo Municipio, Puerto Rico           26266
    ## 3197 0500000US72105           Naranjito Municipio, Puerto Rico           28557
    ## 3198 0500000US72107            Orocovis Municipio, Puerto Rico           21407
    ## 3199 0500000US72109            Patillas Municipio, Puerto Rico           17334
    ## 3200 0500000US72111            Peñuelas Municipio, Puerto Rico           20984
    ## 3201 0500000US72113               Ponce Municipio, Puerto Rico          143926
    ## 3202 0500000US72115        Quebradillas Municipio, Puerto Rico           24036
    ## 3203 0500000US72117              Rincón Municipio, Puerto Rico           14269
    ## 3204 0500000US72119          Río Grande Municipio, Puerto Rico           50550
    ## 3205 0500000US72121       Sabana Grande Municipio, Puerto Rico           23054
    ## 3206 0500000US72123             Salinas Municipio, Puerto Rico           28633
    ## 3207 0500000US72125          San Germán Municipio, Puerto Rico           32114
    ## 3208 0500000US72127            San Juan Municipio, Puerto Rico          344606
    ## 3209 0500000US72129         San Lorenzo Municipio, Puerto Rico           37873
    ## 3210 0500000US72131       San Sebastián Municipio, Puerto Rico           37964
    ## 3211 0500000US72133        Santa Isabel Municipio, Puerto Rico           22066
    ## 3212 0500000US72135            Toa Alta Municipio, Puerto Rico           73405
    ## 3213 0500000US72137            Toa Baja Municipio, Puerto Rico           79726
    ## 3214 0500000US72139       Trujillo Alto Municipio, Puerto Rico           67780
    ## 3215 0500000US72141              Utuado Municipio, Puerto Rico           29402
    ## 3216 0500000US72143           Vega Alta Municipio, Puerto Rico           37724
    ## 3217 0500000US72145           Vega Baja Municipio, Puerto Rico           53371
    ## 3218 0500000US72147             Vieques Municipio, Puerto Rico            8771
    ## 3219 0500000US72149            Villalba Municipio, Puerto Rico           22993
    ## 3220 0500000US72151             Yabucoa Municipio, Puerto Rico           34149
    ## 3221 0500000US72153               Yauco Municipio, Puerto Rico           36439
    ##                 B01003_001M  X
    ## 1    Margin of Error!!Total NA
    ## 2                     ***** NA
    ## 3                     ***** NA
    ## 4                     ***** NA
    ## 5                     ***** NA
    ## 6                     ***** NA
    ## 7                     ***** NA
    ## 8                     ***** NA
    ## 9                     ***** NA
    ## 10                    ***** NA
    ## 11                    ***** NA
    ## 12                    ***** NA
    ## 13                    ***** NA
    ## 14                    ***** NA
    ## 15                    ***** NA
    ## 16                    ***** NA
    ## 17                    ***** NA
    ## 18                    ***** NA
    ## 19                    ***** NA
    ## 20                    ***** NA
    ## 21                    ***** NA
    ## 22                    ***** NA
    ## 23                    ***** NA
    ## 24                    ***** NA
    ## 25                    ***** NA
    ## 26                    ***** NA
    ## 27                    ***** NA
    ## 28                    ***** NA
    ## 29                    ***** NA
    ## 30                    ***** NA
    ## 31                    ***** NA
    ## 32                    ***** NA
    ## 33                    ***** NA
    ## 34                    ***** NA
    ## 35                    ***** NA
    ## 36                    ***** NA
    ## 37                    ***** NA
    ## 38                    ***** NA
    ## 39                    ***** NA
    ## 40                    ***** NA
    ## 41                    ***** NA
    ## 42                    ***** NA
    ## 43                    ***** NA
    ## 44                    ***** NA
    ## 45                    ***** NA
    ## 46                    ***** NA
    ## 47                    ***** NA
    ## 48                    ***** NA
    ## 49                    ***** NA
    ## 50                    ***** NA
    ## 51                    ***** NA
    ## 52                    ***** NA
    ## 53                    ***** NA
    ## 54                    ***** NA
    ## 55                    ***** NA
    ## 56                    ***** NA
    ## 57                    ***** NA
    ## 58                    ***** NA
    ## 59                    ***** NA
    ## 60                    ***** NA
    ## 61                    ***** NA
    ## 62                    ***** NA
    ## 63                    ***** NA
    ## 64                    ***** NA
    ## 65                    ***** NA
    ## 66                    ***** NA
    ## 67                    ***** NA
    ## 68                    ***** NA
    ## 69                    ***** NA
    ## 70                    ***** NA
    ## 71                    ***** NA
    ## 72                    ***** NA
    ## 73                       89 NA
    ## 74                      380 NA
    ## 75                    ***** NA
    ## 76                    ***** NA
    ## 77                    ***** NA
    ## 78                    ***** NA
    ## 79                    ***** NA
    ## 80                    ***** NA
    ## 81                    ***** NA
    ## 82                    ***** NA
    ## 83                    ***** NA
    ## 84                      380 NA
    ## 85                    ***** NA
    ## 86                    ***** NA
    ## 87                    ***** NA
    ## 88                    ***** NA
    ## 89                    ***** NA
    ## 90                    ***** NA
    ## 91                    ***** NA
    ## 92                      123 NA
    ## 93                    ***** NA
    ## 94                    ***** NA
    ## 95                    ***** NA
    ## 96                       79 NA
    ## 97                    ***** NA
    ## 98                    ***** NA
    ## 99                    ***** NA
    ## 100                   ***** NA
    ## 101                   ***** NA
    ## 102                   ***** NA
    ## 103                   ***** NA
    ## 104                   ***** NA
    ## 105                   ***** NA
    ## 106                   ***** NA
    ## 107                   ***** NA
    ## 108                   ***** NA
    ## 109                   ***** NA
    ## 110                   ***** NA
    ## 111                   ***** NA
    ## 112                   ***** NA
    ## 113                   ***** NA
    ## 114                   ***** NA
    ## 115                   ***** NA
    ## 116                   ***** NA
    ## 117                   ***** NA
    ## 118                   ***** NA
    ## 119                   ***** NA
    ## 120                   ***** NA
    ## 121                   ***** NA
    ## 122                   ***** NA
    ## 123                   ***** NA
    ## 124                   ***** NA
    ## 125                   ***** NA
    ## 126                   ***** NA
    ## 127                   ***** NA
    ## 128                   ***** NA
    ## 129                   ***** NA
    ## 130                   ***** NA
    ## 131                   ***** NA
    ## 132                   ***** NA
    ## 133                   ***** NA
    ## 134                   ***** NA
    ## 135                   ***** NA
    ## 136                   ***** NA
    ## 137                   ***** NA
    ## 138                   ***** NA
    ## 139                   ***** NA
    ## 140                   ***** NA
    ## 141                   ***** NA
    ## 142                   ***** NA
    ## 143                   ***** NA
    ## 144                   ***** NA
    ## 145                   ***** NA
    ## 146                   ***** NA
    ## 147                   ***** NA
    ## 148                   ***** NA
    ## 149                   ***** NA
    ## 150                   ***** NA
    ## 151                   ***** NA
    ## 152                   ***** NA
    ## 153                   ***** NA
    ## 154                   ***** NA
    ## 155                   ***** NA
    ## 156                   ***** NA
    ## 157                   ***** NA
    ## 158                   ***** NA
    ## 159                   ***** NA
    ## 160                   ***** NA
    ## 161                   ***** NA
    ## 162                   ***** NA
    ## 163                   ***** NA
    ## 164                   ***** NA
    ## 165                   ***** NA
    ## 166                   ***** NA
    ## 167                   ***** NA
    ## 168                   ***** NA
    ## 169                   ***** NA
    ## 170                   ***** NA
    ## 171                   ***** NA
    ## 172                   ***** NA
    ## 173                   ***** NA
    ## 174                   ***** NA
    ## 175                   ***** NA
    ## 176                   ***** NA
    ## 177                   ***** NA
    ## 178                   ***** NA
    ## 179                   ***** NA
    ## 180                   ***** NA
    ## 181                   ***** NA
    ## 182                   ***** NA
    ## 183                   ***** NA
    ## 184                   ***** NA
    ## 185                   ***** NA
    ## 186                   ***** NA
    ## 187                   ***** NA
    ## 188                   ***** NA
    ## 189                     161 NA
    ## 190                   ***** NA
    ## 191                   ***** NA
    ## 192                   ***** NA
    ## 193                   ***** NA
    ## 194                   ***** NA
    ## 195                   ***** NA
    ## 196                   ***** NA
    ## 197                   ***** NA
    ## 198                   ***** NA
    ## 199                   ***** NA
    ## 200                   ***** NA
    ## 201                   ***** NA
    ## 202                   ***** NA
    ## 203                   ***** NA
    ## 204                   ***** NA
    ## 205                   ***** NA
    ## 206                   ***** NA
    ## 207                   ***** NA
    ## 208                   ***** NA
    ## 209                   ***** NA
    ## 210                   ***** NA
    ## 211                   ***** NA
    ## 212                   ***** NA
    ## 213                   ***** NA
    ## 214                   ***** NA
    ## 215                   ***** NA
    ## 216                   ***** NA
    ## 217                   ***** NA
    ## 218                   ***** NA
    ## 219                   ***** NA
    ## 220                   ***** NA
    ## 221                   ***** NA
    ## 222                   ***** NA
    ## 223                   ***** NA
    ## 224                   ***** NA
    ## 225                   ***** NA
    ## 226                   ***** NA
    ## 227                   ***** NA
    ## 228                   ***** NA
    ## 229                   ***** NA
    ## 230                   ***** NA
    ## 231                   ***** NA
    ## 232                   ***** NA
    ## 233                     161 NA
    ## 234                   ***** NA
    ## 235                   ***** NA
    ## 236                   ***** NA
    ## 237                   ***** NA
    ## 238                   ***** NA
    ## 239                   ***** NA
    ## 240                   ***** NA
    ## 241                   ***** NA
    ## 242                   ***** NA
    ## 243                   ***** NA
    ## 244                   ***** NA
    ## 245                   ***** NA
    ## 246                   ***** NA
    ## 247                   ***** NA
    ## 248                   ***** NA
    ## 249                   ***** NA
    ## 250                   ***** NA
    ## 251                   ***** NA
    ## 252                   ***** NA
    ## 253                   ***** NA
    ## 254                   ***** NA
    ## 255                     170 NA
    ## 256                   ***** NA
    ## 257                   ***** NA
    ## 258                   ***** NA
    ## 259                   ***** NA
    ## 260                   ***** NA
    ## 261                   ***** NA
    ## 262                   ***** NA
    ## 263                     170 NA
    ## 264                   ***** NA
    ## 265                   ***** NA
    ## 266                   ***** NA
    ## 267                   ***** NA
    ## 268                   ***** NA
    ## 269                   ***** NA
    ## 270                   ***** NA
    ## 271                   ***** NA
    ## 272                   ***** NA
    ## 273                     108 NA
    ## 274                   ***** NA
    ## 275                     118 NA
    ## 276                   ***** NA
    ## 277                     118 NA
    ## 278                   ***** NA
    ## 279                   ***** NA
    ## 280                   ***** NA
    ## 281                   ***** NA
    ## 282                   ***** NA
    ## 283                   ***** NA
    ## 284                   ***** NA
    ## 285                   ***** NA
    ## 286                     113 NA
    ## 287                   ***** NA
    ## 288                   ***** NA
    ## 289                   ***** NA
    ## 290                   ***** NA
    ## 291                   ***** NA
    ## 292                   ***** NA
    ## 293                   ***** NA
    ## 294                   ***** NA
    ## 295                   ***** NA
    ## 296                   ***** NA
    ## 297                   ***** NA
    ## 298                   ***** NA
    ## 299                   ***** NA
    ## 300                   ***** NA
    ## 301                   ***** NA
    ## 302                      97 NA
    ## 303                   ***** NA
    ## 304                   ***** NA
    ## 305                   ***** NA
    ## 306                   ***** NA
    ## 307                   ***** NA
    ## 308                   ***** NA
    ## 309                   ***** NA
    ## 310                   ***** NA
    ## 311                   ***** NA
    ## 312                   ***** NA
    ## 313                   ***** NA
    ## 314                   ***** NA
    ## 315                   ***** NA
    ## 316                   ***** NA
    ## 317                   ***** NA
    ## 318                   ***** NA
    ## 319                   ***** NA
    ## 320                   ***** NA
    ## 321                   ***** NA
    ## 322                   ***** NA
    ## 323                   ***** NA
    ## 324                   ***** NA
    ## 325                   ***** NA
    ## 326                   ***** NA
    ## 327                   ***** NA
    ## 328                   ***** NA
    ## 329                   ***** NA
    ## 330                   ***** NA
    ## 331                   ***** NA
    ## 332                   ***** NA
    ## 333                   ***** NA
    ## 334                   ***** NA
    ## 335                   ***** NA
    ## 336                   ***** NA
    ## 337                   ***** NA
    ## 338                   ***** NA
    ## 339                   ***** NA
    ## 340                   ***** NA
    ## 341                   ***** NA
    ## 342                   ***** NA
    ## 343                   ***** NA
    ## 344                   ***** NA
    ## 345                   ***** NA
    ## 346                   ***** NA
    ## 347                   ***** NA
    ## 348                   ***** NA
    ## 349                   ***** NA
    ## 350                   ***** NA
    ## 351                   ***** NA
    ## 352                   ***** NA
    ## 353                   ***** NA
    ## 354                   ***** NA
    ## 355                   ***** NA
    ## 356                   ***** NA
    ## 357                   ***** NA
    ## 358                   ***** NA
    ## 359                   ***** NA
    ## 360                   ***** NA
    ## 361                   ***** NA
    ## 362                   ***** NA
    ## 363                   ***** NA
    ## 364                   ***** NA
    ## 365                   ***** NA
    ## 366                   ***** NA
    ## 367                   ***** NA
    ## 368                   ***** NA
    ## 369                   ***** NA
    ## 370                   ***** NA
    ## 371                   ***** NA
    ## 372                   ***** NA
    ## 373                   ***** NA
    ## 374                   ***** NA
    ## 375                   ***** NA
    ## 376                   ***** NA
    ## 377                   ***** NA
    ## 378                   ***** NA
    ## 379                   ***** NA
    ## 380                   ***** NA
    ## 381                   ***** NA
    ## 382                   ***** NA
    ## 383                   ***** NA
    ## 384                   ***** NA
    ## 385                   ***** NA
    ## 386                   ***** NA
    ## 387                   ***** NA
    ## 388                   ***** NA
    ## 389                   ***** NA
    ## 390                   ***** NA
    ## 391                   ***** NA
    ## 392                   ***** NA
    ## 393                   ***** NA
    ## 394                   ***** NA
    ## 395                   ***** NA
    ## 396                   ***** NA
    ## 397                   ***** NA
    ## 398                   ***** NA
    ## 399                   ***** NA
    ## 400                   ***** NA
    ## 401                   ***** NA
    ## 402                   ***** NA
    ## 403                   ***** NA
    ## 404                   ***** NA
    ## 405                   ***** NA
    ## 406                   ***** NA
    ## 407                   ***** NA
    ## 408                   ***** NA
    ## 409                   ***** NA
    ## 410                   ***** NA
    ## 411                   ***** NA
    ## 412                   ***** NA
    ## 413                   ***** NA
    ## 414                   ***** NA
    ## 415                   ***** NA
    ## 416                   ***** NA
    ## 417                   ***** NA
    ## 418                   ***** NA
    ## 419                   ***** NA
    ## 420                   ***** NA
    ## 421                   ***** NA
    ## 422                   ***** NA
    ## 423                   ***** NA
    ## 424                   ***** NA
    ## 425                   ***** NA
    ## 426                   ***** NA
    ## 427                   ***** NA
    ## 428                   ***** NA
    ## 429                   ***** NA
    ## 430                   ***** NA
    ## 431                   ***** NA
    ## 432                   ***** NA
    ## 433                   ***** NA
    ## 434                   ***** NA
    ## 435                   ***** NA
    ## 436                   ***** NA
    ## 437                   ***** NA
    ## 438                   ***** NA
    ## 439                   ***** NA
    ## 440                   ***** NA
    ## 441                   ***** NA
    ## 442                   ***** NA
    ## 443                   ***** NA
    ## 444                   ***** NA
    ## 445                   ***** NA
    ## 446                   ***** NA
    ## 447                   ***** NA
    ## 448                   ***** NA
    ## 449                   ***** NA
    ## 450                   ***** NA
    ## 451                   ***** NA
    ## 452                   ***** NA
    ## 453                   ***** NA
    ## 454                   ***** NA
    ## 455                   ***** NA
    ## 456                   ***** NA
    ## 457                   ***** NA
    ## 458                   ***** NA
    ## 459                   ***** NA
    ## 460                   ***** NA
    ## 461                   ***** NA
    ## 462                   ***** NA
    ## 463                   ***** NA
    ## 464                   ***** NA
    ## 465                   ***** NA
    ## 466                   ***** NA
    ## 467                   ***** NA
    ## 468                   ***** NA
    ## 469                   ***** NA
    ## 470                   ***** NA
    ## 471                   ***** NA
    ## 472                   ***** NA
    ## 473                   ***** NA
    ## 474                   ***** NA
    ## 475                   ***** NA
    ## 476                   ***** NA
    ## 477                   ***** NA
    ## 478                   ***** NA
    ## 479                   ***** NA
    ## 480                   ***** NA
    ## 481                   ***** NA
    ## 482                   ***** NA
    ## 483                   ***** NA
    ## 484                   ***** NA
    ## 485                   ***** NA
    ## 486                   ***** NA
    ## 487                   ***** NA
    ## 488                   ***** NA
    ## 489                   ***** NA
    ## 490                   ***** NA
    ## 491                   ***** NA
    ## 492                   ***** NA
    ## 493                   ***** NA
    ## 494                   ***** NA
    ## 495                   ***** NA
    ## 496                   ***** NA
    ## 497                   ***** NA
    ## 498                   ***** NA
    ## 499                   ***** NA
    ## 500                   ***** NA
    ## 501                   ***** NA
    ## 502                   ***** NA
    ## 503                   ***** NA
    ## 504                   ***** NA
    ## 505                   ***** NA
    ## 506                     152 NA
    ## 507                   ***** NA
    ## 508                   ***** NA
    ## 509                   ***** NA
    ## 510                   ***** NA
    ## 511                   ***** NA
    ## 512                   ***** NA
    ## 513                   ***** NA
    ## 514                   ***** NA
    ## 515                   ***** NA
    ## 516                   ***** NA
    ## 517                   ***** NA
    ## 518                   ***** NA
    ## 519                     152 NA
    ## 520                   ***** NA
    ## 521                   ***** NA
    ## 522                   ***** NA
    ## 523                   ***** NA
    ## 524                   ***** NA
    ## 525                   ***** NA
    ## 526                   ***** NA
    ## 527                   ***** NA
    ## 528                   ***** NA
    ## 529                   ***** NA
    ## 530                   ***** NA
    ## 531                   ***** NA
    ## 532                   ***** NA
    ## 533                   ***** NA
    ## 534                   ***** NA
    ## 535                   ***** NA
    ## 536                   ***** NA
    ## 537                   ***** NA
    ## 538                   ***** NA
    ## 539                   ***** NA
    ## 540                   ***** NA
    ## 541                   ***** NA
    ## 542                   ***** NA
    ## 543                   ***** NA
    ## 544                   ***** NA
    ## 545                   ***** NA
    ## 546                   ***** NA
    ## 547                   ***** NA
    ## 548                   ***** NA
    ## 549                   ***** NA
    ## 550                      16 NA
    ## 551                   ***** NA
    ## 552                      16 NA
    ## 553                   ***** NA
    ## 554                   ***** NA
    ## 555                   ***** NA
    ## 556                   ***** NA
    ## 557                   ***** NA
    ## 558                   ***** NA
    ## 559                   ***** NA
    ## 560                   ***** NA
    ## 561                   ***** NA
    ## 562                   ***** NA
    ## 563                   ***** NA
    ## 564                   ***** NA
    ## 565                     105 NA
    ## 566                   ***** NA
    ## 567                   ***** NA
    ## 568                   ***** NA
    ## 569                     105 NA
    ## 570                   ***** NA
    ## 571                   ***** NA
    ## 572                   ***** NA
    ## 573                   ***** NA
    ## 574                   ***** NA
    ## 575                   ***** NA
    ## 576                   ***** NA
    ## 577                   ***** NA
    ## 578                   ***** NA
    ## 579                   ***** NA
    ## 580                   ***** NA
    ## 581                   ***** NA
    ## 582                   ***** NA
    ## 583                   ***** NA
    ## 584                   ***** NA
    ## 585                   ***** NA
    ## 586                   ***** NA
    ## 587                   ***** NA
    ## 588                   ***** NA
    ## 589                   ***** NA
    ## 590                   ***** NA
    ## 591                   ***** NA
    ## 592                   ***** NA
    ## 593                   ***** NA
    ## 594                   ***** NA
    ## 595                   ***** NA
    ## 596                   ***** NA
    ## 597                   ***** NA
    ## 598                   ***** NA
    ## 599                   ***** NA
    ## 600                   ***** NA
    ## 601                   ***** NA
    ## 602                   ***** NA
    ## 603                   ***** NA
    ## 604                   ***** NA
    ## 605                   ***** NA
    ## 606                   ***** NA
    ## 607                   ***** NA
    ## 608                   ***** NA
    ## 609                   ***** NA
    ## 610                   ***** NA
    ## 611                   ***** NA
    ## 612                   ***** NA
    ## 613                   ***** NA
    ## 614                   ***** NA
    ## 615                   ***** NA
    ## 616                   ***** NA
    ## 617                   ***** NA
    ## 618                   ***** NA
    ## 619                   ***** NA
    ## 620                   ***** NA
    ## 621                   ***** NA
    ## 622                   ***** NA
    ## 623                   ***** NA
    ## 624                   ***** NA
    ## 625                   ***** NA
    ## 626                   ***** NA
    ## 627                   ***** NA
    ## 628                   ***** NA
    ## 629                   ***** NA
    ## 630                   ***** NA
    ## 631                   ***** NA
    ## 632                   ***** NA
    ## 633                   ***** NA
    ## 634                   ***** NA
    ## 635                   ***** NA
    ## 636                   ***** NA
    ## 637                   ***** NA
    ## 638                   ***** NA
    ## 639                   ***** NA
    ## 640                   ***** NA
    ## 641                   ***** NA
    ## 642                   ***** NA
    ## 643                   ***** NA
    ## 644                   ***** NA
    ## 645                   ***** NA
    ## 646                   ***** NA
    ## 647                   ***** NA
    ## 648                   ***** NA
    ## 649                   ***** NA
    ## 650                   ***** NA
    ## 651                   ***** NA
    ## 652                   ***** NA
    ## 653                   ***** NA
    ## 654                   ***** NA
    ## 655                   ***** NA
    ## 656                   ***** NA
    ## 657                   ***** NA
    ## 658                   ***** NA
    ## 659                   ***** NA
    ## 660                   ***** NA
    ## 661                   ***** NA
    ## 662                   ***** NA
    ## 663                   ***** NA
    ## 664                   ***** NA
    ## 665                   ***** NA
    ## 666                   ***** NA
    ## 667                   ***** NA
    ## 668                   ***** NA
    ## 669                   ***** NA
    ## 670                   ***** NA
    ## 671                   ***** NA
    ## 672                   ***** NA
    ## 673                   ***** NA
    ## 674                   ***** NA
    ## 675                   ***** NA
    ## 676                   ***** NA
    ## 677                   ***** NA
    ## 678                   ***** NA
    ## 679                   ***** NA
    ## 680                   ***** NA
    ## 681                   ***** NA
    ## 682                   ***** NA
    ## 683                   ***** NA
    ## 684                   ***** NA
    ## 685                   ***** NA
    ## 686                   ***** NA
    ## 687                   ***** NA
    ## 688                   ***** NA
    ## 689                   ***** NA
    ## 690                   ***** NA
    ## 691                   ***** NA
    ## 692                   ***** NA
    ## 693                   ***** NA
    ## 694                   ***** NA
    ## 695                   ***** NA
    ## 696                   ***** NA
    ## 697                   ***** NA
    ## 698                   ***** NA
    ## 699                   ***** NA
    ## 700                   ***** NA
    ## 701                   ***** NA
    ## 702                   ***** NA
    ## 703                   ***** NA
    ## 704                   ***** NA
    ## 705                   ***** NA
    ## 706                   ***** NA
    ## 707                   ***** NA
    ## 708                   ***** NA
    ## 709                   ***** NA
    ## 710                   ***** NA
    ## 711                   ***** NA
    ## 712                   ***** NA
    ## 713                   ***** NA
    ## 714                   ***** NA
    ## 715                   ***** NA
    ## 716                   ***** NA
    ## 717                   ***** NA
    ## 718                   ***** NA
    ## 719                   ***** NA
    ## 720                   ***** NA
    ## 721                   ***** NA
    ## 722                   ***** NA
    ## 723                   ***** NA
    ## 724                   ***** NA
    ## 725                   ***** NA
    ## 726                   ***** NA
    ## 727                   ***** NA
    ## 728                   ***** NA
    ## 729                   ***** NA
    ## 730                   ***** NA
    ## 731                   ***** NA
    ## 732                   ***** NA
    ## 733                   ***** NA
    ## 734                   ***** NA
    ## 735                   ***** NA
    ## 736                   ***** NA
    ## 737                   ***** NA
    ## 738                   ***** NA
    ## 739                   ***** NA
    ## 740                   ***** NA
    ## 741                   ***** NA
    ## 742                   ***** NA
    ## 743                   ***** NA
    ## 744                   ***** NA
    ## 745                   ***** NA
    ## 746                   ***** NA
    ## 747                   ***** NA
    ## 748                   ***** NA
    ## 749                   ***** NA
    ## 750                   ***** NA
    ## 751                   ***** NA
    ## 752                   ***** NA
    ## 753                   ***** NA
    ## 754                   ***** NA
    ## 755                   ***** NA
    ## 756                   ***** NA
    ## 757                   ***** NA
    ## 758                   ***** NA
    ## 759                   ***** NA
    ## 760                   ***** NA
    ## 761                   ***** NA
    ## 762                   ***** NA
    ## 763                   ***** NA
    ## 764                   ***** NA
    ## 765                   ***** NA
    ## 766                   ***** NA
    ## 767                   ***** NA
    ## 768                   ***** NA
    ## 769                   ***** NA
    ## 770                   ***** NA
    ## 771                   ***** NA
    ## 772                   ***** NA
    ## 773                   ***** NA
    ## 774                   ***** NA
    ## 775                   ***** NA
    ## 776                   ***** NA
    ## 777                   ***** NA
    ## 778                   ***** NA
    ## 779                   ***** NA
    ## 780                   ***** NA
    ## 781                   ***** NA
    ## 782                   ***** NA
    ## 783                   ***** NA
    ## 784                   ***** NA
    ## 785                   ***** NA
    ## 786                   ***** NA
    ## 787                   ***** NA
    ## 788                   ***** NA
    ## 789                   ***** NA
    ## 790                   ***** NA
    ## 791                   ***** NA
    ## 792                   ***** NA
    ## 793                   ***** NA
    ## 794                   ***** NA
    ## 795                   ***** NA
    ## 796                   ***** NA
    ## 797                   ***** NA
    ## 798                   ***** NA
    ## 799                   ***** NA
    ## 800                   ***** NA
    ## 801                   ***** NA
    ## 802                   ***** NA
    ## 803                   ***** NA
    ## 804                   ***** NA
    ## 805                   ***** NA
    ## 806                   ***** NA
    ## 807                   ***** NA
    ## 808                   ***** NA
    ## 809                   ***** NA
    ## 810                   ***** NA
    ## 811                   ***** NA
    ## 812                   ***** NA
    ## 813                   ***** NA
    ## 814                   ***** NA
    ## 815                   ***** NA
    ## 816                   ***** NA
    ## 817                   ***** NA
    ## 818                   ***** NA
    ## 819                   ***** NA
    ## 820                   ***** NA
    ## 821                   ***** NA
    ## 822                   ***** NA
    ## 823                   ***** NA
    ## 824                   ***** NA
    ## 825                   ***** NA
    ## 826                   ***** NA
    ## 827                   ***** NA
    ## 828                   ***** NA
    ## 829                   ***** NA
    ## 830                   ***** NA
    ## 831                   ***** NA
    ## 832                   ***** NA
    ## 833                   ***** NA
    ## 834                   ***** NA
    ## 835                   ***** NA
    ## 836                   ***** NA
    ## 837                   ***** NA
    ## 838                   ***** NA
    ## 839                   ***** NA
    ## 840                   ***** NA
    ## 841                   ***** NA
    ## 842                   ***** NA
    ## 843                   ***** NA
    ## 844                   ***** NA
    ## 845                   ***** NA
    ## 846                   ***** NA
    ## 847                   ***** NA
    ## 848                   ***** NA
    ## 849                   ***** NA
    ## 850                   ***** NA
    ## 851                   ***** NA
    ## 852                   ***** NA
    ## 853                   ***** NA
    ## 854                   ***** NA
    ## 855                   ***** NA
    ## 856                   ***** NA
    ## 857                   ***** NA
    ## 858                   ***** NA
    ## 859                   ***** NA
    ## 860                   ***** NA
    ## 861                   ***** NA
    ## 862                   ***** NA
    ## 863                   ***** NA
    ## 864                   ***** NA
    ## 865                   ***** NA
    ## 866                   ***** NA
    ## 867                   ***** NA
    ## 868                   ***** NA
    ## 869                   ***** NA
    ## 870                   ***** NA
    ## 871                   ***** NA
    ## 872                   ***** NA
    ## 873                   ***** NA
    ## 874                   ***** NA
    ## 875                   ***** NA
    ## 876                   ***** NA
    ## 877                   ***** NA
    ## 878                   ***** NA
    ## 879                   ***** NA
    ## 880                   ***** NA
    ## 881                   ***** NA
    ## 882                   ***** NA
    ## 883                   ***** NA
    ## 884                   ***** NA
    ## 885                   ***** NA
    ## 886                   ***** NA
    ## 887                   ***** NA
    ## 888                   ***** NA
    ## 889                   ***** NA
    ## 890                   ***** NA
    ## 891                   ***** NA
    ## 892                   ***** NA
    ## 893                     149 NA
    ## 894                   ***** NA
    ## 895                   ***** NA
    ## 896                   ***** NA
    ## 897                   ***** NA
    ## 898                   ***** NA
    ## 899                   ***** NA
    ## 900                   ***** NA
    ## 901                   ***** NA
    ## 902                   ***** NA
    ## 903                   ***** NA
    ## 904                   ***** NA
    ## 905                   ***** NA
    ## 906                     149 NA
    ## 907                   ***** NA
    ## 908                   ***** NA
    ## 909                   ***** NA
    ## 910                   ***** NA
    ## 911                   ***** NA
    ## 912                   ***** NA
    ## 913                   ***** NA
    ## 914                   ***** NA
    ## 915                   ***** NA
    ## 916                   ***** NA
    ## 917                   ***** NA
    ## 918                   ***** NA
    ## 919                   ***** NA
    ## 920                   ***** NA
    ## 921                     114 NA
    ## 922                   ***** NA
    ## 923                   ***** NA
    ## 924                   ***** NA
    ## 925                     112 NA
    ## 926                   ***** NA
    ## 927                   ***** NA
    ## 928                   ***** NA
    ## 929                   ***** NA
    ## 930                   ***** NA
    ## 931                     132 NA
    ## 932                   ***** NA
    ## 933                   ***** NA
    ## 934                   ***** NA
    ## 935                   ***** NA
    ## 936                   ***** NA
    ## 937                   ***** NA
    ## 938                   ***** NA
    ## 939                   ***** NA
    ## 940                     114 NA
    ## 941                   ***** NA
    ## 942                   ***** NA
    ## 943                   ***** NA
    ## 944                   ***** NA
    ## 945                   ***** NA
    ## 946                   ***** NA
    ## 947                   ***** NA
    ## 948                   ***** NA
    ## 949                   ***** NA
    ## 950                   ***** NA
    ## 951                   ***** NA
    ## 952                   ***** NA
    ## 953                   ***** NA
    ## 954                   ***** NA
    ## 955                   ***** NA
    ## 956                   ***** NA
    ## 957                   ***** NA
    ## 958                   ***** NA
    ## 959                   ***** NA
    ## 960                   ***** NA
    ## 961                   ***** NA
    ## 962                   ***** NA
    ## 963                   ***** NA
    ## 964                   ***** NA
    ## 965                   ***** NA
    ## 966                   ***** NA
    ## 967                   ***** NA
    ## 968                   ***** NA
    ## 969                   ***** NA
    ## 970                   ***** NA
    ## 971                   ***** NA
    ## 972                     132 NA
    ## 973                   ***** NA
    ## 974                   ***** NA
    ## 975                   ***** NA
    ## 976                   ***** NA
    ## 977                   ***** NA
    ## 978                   ***** NA
    ## 979                   ***** NA
    ## 980                   ***** NA
    ## 981                   ***** NA
    ## 982                   ***** NA
    ## 983                   ***** NA
    ## 984                   ***** NA
    ## 985                   ***** NA
    ## 986                   ***** NA
    ## 987                   ***** NA
    ## 988                   ***** NA
    ## 989                     112 NA
    ## 990                   ***** NA
    ## 991                   ***** NA
    ## 992                   ***** NA
    ## 993                   ***** NA
    ## 994                   ***** NA
    ## 995                   ***** NA
    ## 996                   ***** NA
    ## 997                   ***** NA
    ## 998                   ***** NA
    ## 999                   ***** NA
    ## 1000                  ***** NA
    ## 1001                  ***** NA
    ## 1002                  ***** NA
    ## 1003                  ***** NA
    ## 1004                  ***** NA
    ## 1005                  ***** NA
    ## 1006                  ***** NA
    ## 1007                  ***** NA
    ## 1008                  ***** NA
    ## 1009                  ***** NA
    ## 1010                  ***** NA
    ## 1011                  ***** NA
    ## 1012                  ***** NA
    ## 1013                  ***** NA
    ## 1014                  ***** NA
    ## 1015                  ***** NA
    ## 1016                  ***** NA
    ## 1017                  ***** NA
    ## 1018                  ***** NA
    ## 1019                  ***** NA
    ## 1020                  ***** NA
    ## 1021                  ***** NA
    ## 1022                  ***** NA
    ## 1023                  ***** NA
    ## 1024                  ***** NA
    ## 1025                  ***** NA
    ## 1026                  ***** NA
    ## 1027                  ***** NA
    ## 1028                  ***** NA
    ## 1029                  ***** NA
    ## 1030                  ***** NA
    ## 1031                  ***** NA
    ## 1032                  ***** NA
    ## 1033                  ***** NA
    ## 1034                  ***** NA
    ## 1035                  ***** NA
    ## 1036                  ***** NA
    ## 1037                  ***** NA
    ## 1038                  ***** NA
    ## 1039                  ***** NA
    ## 1040                  ***** NA
    ## 1041                  ***** NA
    ## 1042                  ***** NA
    ## 1043                  ***** NA
    ## 1044                  ***** NA
    ## 1045                  ***** NA
    ## 1046                  ***** NA
    ## 1047                  ***** NA
    ## 1048                  ***** NA
    ## 1049                  ***** NA
    ## 1050                  ***** NA
    ## 1051                  ***** NA
    ## 1052                  ***** NA
    ## 1053                  ***** NA
    ## 1054                  ***** NA
    ## 1055                  ***** NA
    ## 1056                  ***** NA
    ## 1057                  ***** NA
    ## 1058                  ***** NA
    ## 1059                  ***** NA
    ## 1060                  ***** NA
    ## 1061                  ***** NA
    ## 1062                  ***** NA
    ## 1063                  ***** NA
    ## 1064                  ***** NA
    ## 1065                  ***** NA
    ## 1066                  ***** NA
    ## 1067                  ***** NA
    ## 1068                  ***** NA
    ## 1069                  ***** NA
    ## 1070                  ***** NA
    ## 1071                  ***** NA
    ## 1072                  ***** NA
    ## 1073                  ***** NA
    ## 1074                  ***** NA
    ## 1075                  ***** NA
    ## 1076                  ***** NA
    ## 1077                  ***** NA
    ## 1078                  ***** NA
    ## 1079                  ***** NA
    ## 1080                  ***** NA
    ## 1081                  ***** NA
    ## 1082                  ***** NA
    ## 1083                  ***** NA
    ## 1084                  ***** NA
    ## 1085                  ***** NA
    ## 1086                  ***** NA
    ## 1087                  ***** NA
    ## 1088                  ***** NA
    ## 1089                  ***** NA
    ## 1090                  ***** NA
    ## 1091                  ***** NA
    ## 1092                  ***** NA
    ## 1093                  ***** NA
    ## 1094                  ***** NA
    ## 1095                  ***** NA
    ## 1096                  ***** NA
    ## 1097                  ***** NA
    ## 1098                  ***** NA
    ## 1099                  ***** NA
    ## 1100                  ***** NA
    ## 1101                  ***** NA
    ## 1102                  ***** NA
    ## 1103                  ***** NA
    ## 1104                  ***** NA
    ## 1105                  ***** NA
    ## 1106                  ***** NA
    ## 1107                  ***** NA
    ## 1108                  ***** NA
    ## 1109                  ***** NA
    ## 1110                  ***** NA
    ## 1111                  ***** NA
    ## 1112                  ***** NA
    ## 1113                  ***** NA
    ## 1114                  ***** NA
    ## 1115                  ***** NA
    ## 1116                  ***** NA
    ## 1117                  ***** NA
    ## 1118                  ***** NA
    ## 1119                  ***** NA
    ## 1120                  ***** NA
    ## 1121                  ***** NA
    ## 1122                  ***** NA
    ## 1123                  ***** NA
    ## 1124                  ***** NA
    ## 1125                  ***** NA
    ## 1126                  ***** NA
    ## 1127                  ***** NA
    ## 1128                  ***** NA
    ## 1129                  ***** NA
    ## 1130                  ***** NA
    ## 1131                  ***** NA
    ## 1132                  ***** NA
    ## 1133                  ***** NA
    ## 1134                  ***** NA
    ## 1135                  ***** NA
    ## 1136                  ***** NA
    ## 1137                  ***** NA
    ## 1138                  ***** NA
    ## 1139                  ***** NA
    ## 1140                  ***** NA
    ## 1141                  ***** NA
    ## 1142                  ***** NA
    ## 1143                  ***** NA
    ## 1144                  ***** NA
    ## 1145                  ***** NA
    ## 1146                  ***** NA
    ## 1147                  ***** NA
    ## 1148                  ***** NA
    ## 1149                  ***** NA
    ## 1150                  ***** NA
    ## 1151                  ***** NA
    ## 1152                  ***** NA
    ## 1153                  ***** NA
    ## 1154                  ***** NA
    ## 1155                  ***** NA
    ## 1156                  ***** NA
    ## 1157                  ***** NA
    ## 1158                  ***** NA
    ## 1159                  ***** NA
    ## 1160                  ***** NA
    ## 1161                  ***** NA
    ## 1162                  ***** NA
    ## 1163                  ***** NA
    ## 1164                  ***** NA
    ## 1165                  ***** NA
    ## 1166                  ***** NA
    ## 1167                  ***** NA
    ## 1168                  ***** NA
    ## 1169                  ***** NA
    ## 1170                  ***** NA
    ## 1171                  ***** NA
    ## 1172                  ***** NA
    ## 1173                  ***** NA
    ## 1174                  ***** NA
    ## 1175                  ***** NA
    ## 1176                  ***** NA
    ## 1177                  ***** NA
    ## 1178                  ***** NA
    ## 1179                  ***** NA
    ## 1180                  ***** NA
    ## 1181                  ***** NA
    ## 1182                  ***** NA
    ## 1183                  ***** NA
    ## 1184                  ***** NA
    ## 1185                  ***** NA
    ## 1186                  ***** NA
    ## 1187                  ***** NA
    ## 1188                  ***** NA
    ## 1189                  ***** NA
    ## 1190                  ***** NA
    ## 1191                  ***** NA
    ## 1192                  ***** NA
    ## 1193                  ***** NA
    ## 1194                  ***** NA
    ## 1195                  ***** NA
    ## 1196                  ***** NA
    ## 1197                  ***** NA
    ## 1198                  ***** NA
    ## 1199                  ***** NA
    ## 1200                  ***** NA
    ## 1201                  ***** NA
    ## 1202                  ***** NA
    ## 1203                  ***** NA
    ## 1204                  ***** NA
    ## 1205                  ***** NA
    ## 1206                  ***** NA
    ## 1207                  ***** NA
    ## 1208                  ***** NA
    ## 1209                  ***** NA
    ## 1210                  ***** NA
    ## 1211                  ***** NA
    ## 1212                  ***** NA
    ## 1213                  ***** NA
    ## 1214                  ***** NA
    ## 1215                  ***** NA
    ## 1216                  ***** NA
    ## 1217                  ***** NA
    ## 1218                  ***** NA
    ## 1219                  ***** NA
    ## 1220                  ***** NA
    ## 1221                  ***** NA
    ## 1222                  ***** NA
    ## 1223                  ***** NA
    ## 1224                  ***** NA
    ## 1225                  ***** NA
    ## 1226                  ***** NA
    ## 1227                  ***** NA
    ## 1228                  ***** NA
    ## 1229                  ***** NA
    ## 1230                  ***** NA
    ## 1231                  ***** NA
    ## 1232                  ***** NA
    ## 1233                  ***** NA
    ## 1234                  ***** NA
    ## 1235                  ***** NA
    ## 1236                  ***** NA
    ## 1237                  ***** NA
    ## 1238                  ***** NA
    ## 1239                  ***** NA
    ## 1240                  ***** NA
    ## 1241                  ***** NA
    ## 1242                  ***** NA
    ## 1243                  ***** NA
    ## 1244                  ***** NA
    ## 1245                  ***** NA
    ## 1246                  ***** NA
    ## 1247                  ***** NA
    ## 1248                  ***** NA
    ## 1249                  ***** NA
    ## 1250                  ***** NA
    ## 1251                  ***** NA
    ## 1252                  ***** NA
    ## 1253                  ***** NA
    ## 1254                  ***** NA
    ## 1255                  ***** NA
    ## 1256                  ***** NA
    ## 1257                  ***** NA
    ## 1258                  ***** NA
    ## 1259                  ***** NA
    ## 1260                  ***** NA
    ## 1261                  ***** NA
    ## 1262                  ***** NA
    ## 1263                  ***** NA
    ## 1264                  ***** NA
    ## 1265                  ***** NA
    ## 1266                  ***** NA
    ## 1267                  ***** NA
    ## 1268                  ***** NA
    ## 1269                  ***** NA
    ## 1270                  ***** NA
    ## 1271                  ***** NA
    ## 1272                  ***** NA
    ## 1273                  ***** NA
    ## 1274                  ***** NA
    ## 1275                  ***** NA
    ## 1276                  ***** NA
    ## 1277                  ***** NA
    ## 1278                  ***** NA
    ## 1279                  ***** NA
    ## 1280                  ***** NA
    ## 1281                  ***** NA
    ## 1282                  ***** NA
    ## 1283                  ***** NA
    ## 1284                  ***** NA
    ## 1285                  ***** NA
    ## 1286                  ***** NA
    ## 1287                  ***** NA
    ## 1288                  ***** NA
    ## 1289                  ***** NA
    ## 1290                  ***** NA
    ## 1291                  ***** NA
    ## 1292                  ***** NA
    ## 1293                  ***** NA
    ## 1294                  ***** NA
    ## 1295                  ***** NA
    ## 1296                  ***** NA
    ## 1297                  ***** NA
    ## 1298                  ***** NA
    ## 1299                  ***** NA
    ## 1300                  ***** NA
    ## 1301                  ***** NA
    ## 1302                  ***** NA
    ## 1303                  ***** NA
    ## 1304                  ***** NA
    ## 1305                  ***** NA
    ## 1306                  ***** NA
    ## 1307                  ***** NA
    ## 1308                  ***** NA
    ## 1309                  ***** NA
    ## 1310                  ***** NA
    ## 1311                  ***** NA
    ## 1312                  ***** NA
    ## 1313                  ***** NA
    ## 1314                  ***** NA
    ## 1315                  ***** NA
    ## 1316                  ***** NA
    ## 1317                  ***** NA
    ## 1318                  ***** NA
    ## 1319                  ***** NA
    ## 1320                  ***** NA
    ## 1321                  ***** NA
    ## 1322                  ***** NA
    ## 1323                  ***** NA
    ## 1324                  ***** NA
    ## 1325                  ***** NA
    ## 1326                  ***** NA
    ## 1327                  ***** NA
    ## 1328                  ***** NA
    ## 1329                  ***** NA
    ## 1330                  ***** NA
    ## 1331                  ***** NA
    ## 1332                  ***** NA
    ## 1333                  ***** NA
    ## 1334                  ***** NA
    ## 1335                  ***** NA
    ## 1336                  ***** NA
    ## 1337                  ***** NA
    ## 1338                  ***** NA
    ## 1339                  ***** NA
    ## 1340                  ***** NA
    ## 1341                  ***** NA
    ## 1342                  ***** NA
    ## 1343                  ***** NA
    ## 1344                  ***** NA
    ## 1345                  ***** NA
    ## 1346                  ***** NA
    ## 1347                  ***** NA
    ## 1348                  ***** NA
    ## 1349                  ***** NA
    ## 1350                  ***** NA
    ## 1351                  ***** NA
    ## 1352                  ***** NA
    ## 1353                  ***** NA
    ## 1354                  ***** NA
    ## 1355                  ***** NA
    ## 1356                  ***** NA
    ## 1357                  ***** NA
    ## 1358                  ***** NA
    ## 1359                  ***** NA
    ## 1360                  ***** NA
    ## 1361                  ***** NA
    ## 1362                  ***** NA
    ## 1363                  ***** NA
    ## 1364                  ***** NA
    ## 1365                  ***** NA
    ## 1366                  ***** NA
    ## 1367                  ***** NA
    ## 1368                  ***** NA
    ## 1369                  ***** NA
    ## 1370                  ***** NA
    ## 1371                  ***** NA
    ## 1372                  ***** NA
    ## 1373                  ***** NA
    ## 1374                  ***** NA
    ## 1375                  ***** NA
    ## 1376                  ***** NA
    ## 1377                  ***** NA
    ## 1378                  ***** NA
    ## 1379                  ***** NA
    ## 1380                  ***** NA
    ## 1381                  ***** NA
    ## 1382                  ***** NA
    ## 1383                  ***** NA
    ## 1384                  ***** NA
    ## 1385                  ***** NA
    ## 1386                  ***** NA
    ## 1387                  ***** NA
    ## 1388                  ***** NA
    ## 1389                  ***** NA
    ## 1390                  ***** NA
    ## 1391                  ***** NA
    ## 1392                  ***** NA
    ## 1393                  ***** NA
    ## 1394                  ***** NA
    ## 1395                  ***** NA
    ## 1396                  ***** NA
    ## 1397                  ***** NA
    ## 1398                  ***** NA
    ## 1399                  ***** NA
    ## 1400                  ***** NA
    ## 1401                  ***** NA
    ## 1402                  ***** NA
    ## 1403                  ***** NA
    ## 1404                  ***** NA
    ## 1405                  ***** NA
    ## 1406                  ***** NA
    ## 1407                  ***** NA
    ## 1408                  ***** NA
    ## 1409                  ***** NA
    ## 1410                  ***** NA
    ## 1411                  ***** NA
    ## 1412                  ***** NA
    ## 1413                  ***** NA
    ## 1414                  ***** NA
    ## 1415                  ***** NA
    ## 1416                  ***** NA
    ## 1417                  ***** NA
    ## 1418                  ***** NA
    ## 1419                  ***** NA
    ## 1420                  ***** NA
    ## 1421                  ***** NA
    ## 1422                  ***** NA
    ## 1423                  ***** NA
    ## 1424                  ***** NA
    ## 1425                  ***** NA
    ## 1426                  ***** NA
    ## 1427                  ***** NA
    ## 1428                  ***** NA
    ## 1429                  ***** NA
    ## 1430                    195 NA
    ## 1431                  ***** NA
    ## 1432                  ***** NA
    ## 1433                  ***** NA
    ## 1434                  ***** NA
    ## 1435                  ***** NA
    ## 1436                  ***** NA
    ## 1437                  ***** NA
    ## 1438                  ***** NA
    ## 1439                  ***** NA
    ## 1440                  ***** NA
    ## 1441                  ***** NA
    ## 1442                  ***** NA
    ## 1443                  ***** NA
    ## 1444                  ***** NA
    ## 1445                  ***** NA
    ## 1446                  ***** NA
    ## 1447                  ***** NA
    ## 1448                  ***** NA
    ## 1449                  ***** NA
    ## 1450                  ***** NA
    ## 1451                  ***** NA
    ## 1452                  ***** NA
    ## 1453                  ***** NA
    ## 1454                  ***** NA
    ## 1455                  ***** NA
    ## 1456                  ***** NA
    ## 1457                  ***** NA
    ## 1458                  ***** NA
    ## 1459                  ***** NA
    ## 1460                  ***** NA
    ## 1461                  ***** NA
    ## 1462                  ***** NA
    ## 1463                  ***** NA
    ## 1464                  ***** NA
    ## 1465                    195 NA
    ## 1466                  ***** NA
    ## 1467                  ***** NA
    ## 1468                  ***** NA
    ## 1469                  ***** NA
    ## 1470                  ***** NA
    ## 1471                  ***** NA
    ## 1472                  ***** NA
    ## 1473                  ***** NA
    ## 1474                  ***** NA
    ## 1475                  ***** NA
    ## 1476                  ***** NA
    ## 1477                  ***** NA
    ## 1478                  ***** NA
    ## 1479                  ***** NA
    ## 1480                  ***** NA
    ## 1481                  ***** NA
    ## 1482                  ***** NA
    ## 1483                  ***** NA
    ## 1484                  ***** NA
    ## 1485                  ***** NA
    ## 1486                  ***** NA
    ## 1487                  ***** NA
    ## 1488                  ***** NA
    ## 1489                  ***** NA
    ## 1490                  ***** NA
    ## 1491                  ***** NA
    ## 1492                  ***** NA
    ## 1493                  ***** NA
    ## 1494                  ***** NA
    ## 1495                  ***** NA
    ## 1496                  ***** NA
    ## 1497                  ***** NA
    ## 1498                  ***** NA
    ## 1499                  ***** NA
    ## 1500                  ***** NA
    ## 1501                  ***** NA
    ## 1502                  ***** NA
    ## 1503                  ***** NA
    ## 1504                  ***** NA
    ## 1505                  ***** NA
    ## 1506                  ***** NA
    ## 1507                  ***** NA
    ## 1508                  ***** NA
    ## 1509                  ***** NA
    ## 1510                  ***** NA
    ## 1511                  ***** NA
    ## 1512                  ***** NA
    ## 1513                  ***** NA
    ## 1514                  ***** NA
    ## 1515                  ***** NA
    ## 1516                  ***** NA
    ## 1517                  ***** NA
    ## 1518                  ***** NA
    ## 1519                  ***** NA
    ## 1520                  ***** NA
    ## 1521                  ***** NA
    ## 1522                  ***** NA
    ## 1523                  ***** NA
    ## 1524                  ***** NA
    ## 1525                  ***** NA
    ## 1526                  ***** NA
    ## 1527                  ***** NA
    ## 1528                  ***** NA
    ## 1529                  ***** NA
    ## 1530                  ***** NA
    ## 1531                  ***** NA
    ## 1532                  ***** NA
    ## 1533                  ***** NA
    ## 1534                  ***** NA
    ## 1535                  ***** NA
    ## 1536                  ***** NA
    ## 1537                  ***** NA
    ## 1538                  ***** NA
    ## 1539                  ***** NA
    ## 1540                  ***** NA
    ## 1541                  ***** NA
    ## 1542                  ***** NA
    ## 1543                  ***** NA
    ## 1544                  ***** NA
    ## 1545                  ***** NA
    ## 1546                  ***** NA
    ## 1547                  ***** NA
    ## 1548                  ***** NA
    ## 1549                  ***** NA
    ## 1550                  ***** NA
    ## 1551                  ***** NA
    ## 1552                  ***** NA
    ## 1553                  ***** NA
    ## 1554                  ***** NA
    ## 1555                  ***** NA
    ## 1556                  ***** NA
    ## 1557                  ***** NA
    ## 1558                  ***** NA
    ## 1559                  ***** NA
    ## 1560                  ***** NA
    ## 1561                  ***** NA
    ## 1562                  ***** NA
    ## 1563                  ***** NA
    ## 1564                  ***** NA
    ## 1565                  ***** NA
    ## 1566                  ***** NA
    ## 1567                  ***** NA
    ## 1568                  ***** NA
    ## 1569                  ***** NA
    ## 1570                  ***** NA
    ## 1571                  ***** NA
    ## 1572                  ***** NA
    ## 1573                  ***** NA
    ## 1574                  ***** NA
    ## 1575                  ***** NA
    ## 1576                  ***** NA
    ## 1577                  ***** NA
    ## 1578                  ***** NA
    ## 1579                  ***** NA
    ## 1580                  ***** NA
    ## 1581                  ***** NA
    ## 1582                  ***** NA
    ## 1583                  ***** NA
    ## 1584                  ***** NA
    ## 1585                  ***** NA
    ## 1586                  ***** NA
    ## 1587                  ***** NA
    ## 1588                  ***** NA
    ## 1589                  ***** NA
    ## 1590                  ***** NA
    ## 1591                  ***** NA
    ## 1592                  ***** NA
    ## 1593                  ***** NA
    ## 1594                  ***** NA
    ## 1595                  ***** NA
    ## 1596                  ***** NA
    ## 1597                  ***** NA
    ## 1598                  ***** NA
    ## 1599                  ***** NA
    ## 1600                  ***** NA
    ## 1601                  ***** NA
    ## 1602                  ***** NA
    ## 1603                  ***** NA
    ## 1604                  ***** NA
    ## 1605                    110 NA
    ## 1606                  ***** NA
    ## 1607                  ***** NA
    ## 1608                  ***** NA
    ## 1609                    157 NA
    ## 1610                  ***** NA
    ## 1611                  ***** NA
    ## 1612                    157 NA
    ## 1613                  ***** NA
    ## 1614                  ***** NA
    ## 1615                  ***** NA
    ## 1616                    114 NA
    ## 1617                  ***** NA
    ## 1618                    100 NA
    ## 1619                  ***** NA
    ## 1620                  ***** NA
    ## 1621                  ***** NA
    ## 1622                  ***** NA
    ## 1623                  ***** NA
    ## 1624                  ***** NA
    ## 1625                    201 NA
    ## 1626                  ***** NA
    ## 1627                    118 NA
    ## 1628                  ***** NA
    ## 1629                    201 NA
    ## 1630                  ***** NA
    ## 1631                  ***** NA
    ## 1632                    108 NA
    ## 1633                  ***** NA
    ## 1634                     57 NA
    ## 1635                  ***** NA
    ## 1636                  ***** NA
    ## 1637                    110 NA
    ## 1638                  ***** NA
    ## 1639                    116 NA
    ## 1640                  ***** NA
    ## 1641                  ***** NA
    ## 1642                  ***** NA
    ## 1643                  ***** NA
    ## 1644                  ***** NA
    ## 1645                    157 NA
    ## 1646                  ***** NA
    ## 1647                  ***** NA
    ## 1648                  ***** NA
    ## 1649                  ***** NA
    ## 1650                  ***** NA
    ## 1651                     88 NA
    ## 1652                  ***** NA
    ## 1653                  ***** NA
    ## 1654                    145 NA
    ## 1655                  ***** NA
    ## 1656                  ***** NA
    ## 1657                  ***** NA
    ## 1658                     46 NA
    ## 1659                     72 NA
    ## 1660                     55 NA
    ## 1661                  ***** NA
    ## 1662                  ***** NA
    ## 1663                     90 NA
    ## 1664                    104 NA
    ## 1665                  ***** NA
    ## 1666                  ***** NA
    ## 1667                  ***** NA
    ## 1668                  ***** NA
    ## 1669                  ***** NA
    ## 1670                    214 NA
    ## 1671                  ***** NA
    ## 1672                  ***** NA
    ## 1673                  ***** NA
    ## 1674                  ***** NA
    ## 1675                  ***** NA
    ## 1676                  ***** NA
    ## 1677                  ***** NA
    ## 1678                  ***** NA
    ## 1679                  ***** NA
    ## 1680                    120 NA
    ## 1681                  ***** NA
    ## 1682                  ***** NA
    ## 1683                  ***** NA
    ## 1684                    214 NA
    ## 1685                  ***** NA
    ## 1686                  ***** NA
    ## 1687                     74 NA
    ## 1688                  ***** NA
    ## 1689                  ***** NA
    ## 1690                    117 NA
    ## 1691                    102 NA
    ## 1692                  ***** NA
    ## 1693                     63 NA
    ## 1694                  ***** NA
    ## 1695                  ***** NA
    ## 1696                  ***** NA
    ## 1697                  ***** NA
    ## 1698                     74 NA
    ## 1699                  ***** NA
    ## 1700                  ***** NA
    ## 1701                     82 NA
    ## 1702                  ***** NA
    ## 1703                  ***** NA
    ## 1704                  ***** NA
    ## 1705                  ***** NA
    ## 1706                  ***** NA
    ## 1707                    104 NA
    ## 1708                  ***** NA
    ## 1709                  ***** NA
    ## 1710                  ***** NA
    ## 1711                  ***** NA
    ## 1712                     75 NA
    ## 1713                     61 NA
    ## 1714                     89 NA
    ## 1715                  ***** NA
    ## 1716                  ***** NA
    ## 1717                  ***** NA
    ## 1718                  ***** NA
    ## 1719                  ***** NA
    ## 1720                  ***** NA
    ## 1721                  ***** NA
    ## 1722                  ***** NA
    ## 1723                  ***** NA
    ## 1724                  ***** NA
    ## 1725                  ***** NA
    ## 1726                  ***** NA
    ## 1727                  ***** NA
    ## 1728                  ***** NA
    ## 1729                  ***** NA
    ## 1730                     90 NA
    ## 1731                  ***** NA
    ## 1732                  ***** NA
    ## 1733                  ***** NA
    ## 1734                  ***** NA
    ## 1735                  ***** NA
    ## 1736                  ***** NA
    ## 1737                  ***** NA
    ## 1738                     72 NA
    ## 1739                  ***** NA
    ## 1740                  ***** NA
    ## 1741                     72 NA
    ## 1742                  ***** NA
    ## 1743                     61 NA
    ## 1744                  ***** NA
    ## 1745                  ***** NA
    ## 1746                  ***** NA
    ## 1747                    102 NA
    ## 1748                  ***** NA
    ## 1749                  ***** NA
    ## 1750                  ***** NA
    ## 1751                  ***** NA
    ## 1752                  ***** NA
    ## 1753                    167 NA
    ## 1754                    167 NA
    ## 1755                  ***** NA
    ## 1756                  ***** NA
    ## 1757                  ***** NA
    ## 1758                  ***** NA
    ## 1759                  ***** NA
    ## 1760                  ***** NA
    ## 1761                  ***** NA
    ## 1762                  ***** NA
    ## 1763                  ***** NA
    ## 1764                  ***** NA
    ## 1765                  ***** NA
    ## 1766                  ***** NA
    ## 1767                  ***** NA
    ## 1768                  ***** NA
    ## 1769                  ***** NA
    ## 1770                  ***** NA
    ## 1771                  ***** NA
    ## 1772                  ***** NA
    ## 1773                  ***** NA
    ## 1774                  ***** NA
    ## 1775                  ***** NA
    ## 1776                  ***** NA
    ## 1777                  ***** NA
    ## 1778                  ***** NA
    ## 1779                  ***** NA
    ## 1780                  ***** NA
    ## 1781                  ***** NA
    ## 1782                  ***** NA
    ## 1783                  ***** NA
    ## 1784                  ***** NA
    ## 1785                  ***** NA
    ## 1786                  ***** NA
    ## 1787                  ***** NA
    ## 1788                  ***** NA
    ## 1789                  ***** NA
    ## 1790                  ***** NA
    ## 1791                  ***** NA
    ## 1792                  ***** NA
    ## 1793                  ***** NA
    ## 1794                  ***** NA
    ## 1795                  ***** NA
    ## 1796                  ***** NA
    ## 1797                  ***** NA
    ## 1798                  ***** NA
    ## 1799                  ***** NA
    ## 1800                  ***** NA
    ## 1801                  ***** NA
    ## 1802                  ***** NA
    ## 1803                     69 NA
    ## 1804                  ***** NA
    ## 1805                  ***** NA
    ## 1806                  ***** NA
    ## 1807                  ***** NA
    ## 1808                     69 NA
    ## 1809                  ***** NA
    ## 1810                  ***** NA
    ## 1811                  ***** NA
    ## 1812                  ***** NA
    ## 1813                  ***** NA
    ## 1814                  ***** NA
    ## 1815                  ***** NA
    ## 1816                  ***** NA
    ## 1817                  ***** NA
    ## 1818                  ***** NA
    ## 1819                  ***** NA
    ## 1820                  ***** NA
    ## 1821                  ***** NA
    ## 1822                  ***** NA
    ## 1823                  ***** NA
    ## 1824                  ***** NA
    ## 1825                  ***** NA
    ## 1826                  ***** NA
    ## 1827                  ***** NA
    ## 1828                  ***** NA
    ## 1829                  ***** NA
    ## 1830                  ***** NA
    ## 1831                  ***** NA
    ## 1832                  ***** NA
    ## 1833                  ***** NA
    ## 1834                  ***** NA
    ## 1835                  ***** NA
    ## 1836                  ***** NA
    ## 1837                  ***** NA
    ## 1838                  ***** NA
    ## 1839                  ***** NA
    ## 1840                  ***** NA
    ## 1841                  ***** NA
    ## 1842                  ***** NA
    ## 1843                  ***** NA
    ## 1844                  ***** NA
    ## 1845                  ***** NA
    ## 1846                  ***** NA
    ## 1847                  ***** NA
    ## 1848                  ***** NA
    ## 1849                  ***** NA
    ## 1850                  ***** NA
    ## 1851                  ***** NA
    ## 1852                  ***** NA
    ## 1853                  ***** NA
    ## 1854                  ***** NA
    ## 1855                  ***** NA
    ## 1856                  ***** NA
    ## 1857                  ***** NA
    ## 1858                  ***** NA
    ## 1859                  ***** NA
    ## 1860                  ***** NA
    ## 1861                  ***** NA
    ## 1862                  ***** NA
    ## 1863                  ***** NA
    ## 1864                  ***** NA
    ## 1865                  ***** NA
    ## 1866                  ***** NA
    ## 1867                  ***** NA
    ## 1868                  ***** NA
    ## 1869                  ***** NA
    ## 1870                  ***** NA
    ## 1871                  ***** NA
    ## 1872                  ***** NA
    ## 1873                  ***** NA
    ## 1874                  ***** NA
    ## 1875                  ***** NA
    ## 1876                  ***** NA
    ## 1877                  ***** NA
    ## 1878                  ***** NA
    ## 1879                  ***** NA
    ## 1880                  ***** NA
    ## 1881                  ***** NA
    ## 1882                  ***** NA
    ## 1883                  ***** NA
    ## 1884                  ***** NA
    ## 1885                  ***** NA
    ## 1886                  ***** NA
    ## 1887                  ***** NA
    ## 1888                  ***** NA
    ## 1889                  ***** NA
    ## 1890                  ***** NA
    ## 1891                  ***** NA
    ## 1892                  ***** NA
    ## 1893                  ***** NA
    ## 1894                  ***** NA
    ## 1895                  ***** NA
    ## 1896                  ***** NA
    ## 1897                  ***** NA
    ## 1898                  ***** NA
    ## 1899                  ***** NA
    ## 1900                  ***** NA
    ## 1901                  ***** NA
    ## 1902                  ***** NA
    ## 1903                  ***** NA
    ## 1904                  ***** NA
    ## 1905                  ***** NA
    ## 1906                  ***** NA
    ## 1907                  ***** NA
    ## 1908                  ***** NA
    ## 1909                  ***** NA
    ## 1910                  ***** NA
    ## 1911                  ***** NA
    ## 1912                  ***** NA
    ## 1913                  ***** NA
    ## 1914                  ***** NA
    ## 1915                  ***** NA
    ## 1916                  ***** NA
    ## 1917                  ***** NA
    ## 1918                  ***** NA
    ## 1919                  ***** NA
    ## 1920                  ***** NA
    ## 1921                  ***** NA
    ## 1922                  ***** NA
    ## 1923                  ***** NA
    ## 1924                  ***** NA
    ## 1925                  ***** NA
    ## 1926                  ***** NA
    ## 1927                  ***** NA
    ## 1928                  ***** NA
    ## 1929                  ***** NA
    ## 1930                  ***** NA
    ## 1931                  ***** NA
    ## 1932                  ***** NA
    ## 1933                  ***** NA
    ## 1934                  ***** NA
    ## 1935                  ***** NA
    ## 1936                  ***** NA
    ## 1937                  ***** NA
    ## 1938                  ***** NA
    ## 1939                  ***** NA
    ## 1940                  ***** NA
    ## 1941                  ***** NA
    ## 1942                  ***** NA
    ## 1943                  ***** NA
    ## 1944                  ***** NA
    ## 1945                  ***** NA
    ## 1946                  ***** NA
    ## 1947                  ***** NA
    ## 1948                  ***** NA
    ## 1949                  ***** NA
    ## 1950                  ***** NA
    ## 1951                  ***** NA
    ## 1952                  ***** NA
    ## 1953                  ***** NA
    ## 1954                  ***** NA
    ## 1955                  ***** NA
    ## 1956                  ***** NA
    ## 1957                  ***** NA
    ## 1958                  ***** NA
    ## 1959                  ***** NA
    ## 1960                  ***** NA
    ## 1961                  ***** NA
    ## 1962                  ***** NA
    ## 1963                  ***** NA
    ## 1964                  ***** NA
    ## 1965                  ***** NA
    ## 1966                  ***** NA
    ## 1967                  ***** NA
    ## 1968                  ***** NA
    ## 1969                  ***** NA
    ## 1970                  ***** NA
    ## 1971                  ***** NA
    ## 1972                  ***** NA
    ## 1973                  ***** NA
    ## 1974                  ***** NA
    ## 1975                  ***** NA
    ## 1976                  ***** NA
    ## 1977                  ***** NA
    ## 1978                  ***** NA
    ## 1979                  ***** NA
    ## 1980                  ***** NA
    ## 1981                  ***** NA
    ## 1982                  ***** NA
    ## 1983                  ***** NA
    ## 1984                  ***** NA
    ## 1985                  ***** NA
    ## 1986                  ***** NA
    ## 1987                  ***** NA
    ## 1988                  ***** NA
    ## 1989                  ***** NA
    ## 1990                  ***** NA
    ## 1991                  ***** NA
    ## 1992                  ***** NA
    ## 1993                  ***** NA
    ## 1994                  ***** NA
    ## 1995                     90 NA
    ## 1996                  ***** NA
    ## 1997                  ***** NA
    ## 1998                  ***** NA
    ## 1999                  ***** NA
    ## 2000                  ***** NA
    ## 2001                  ***** NA
    ## 2002                  ***** NA
    ## 2003                  ***** NA
    ## 2004                  ***** NA
    ## 2005                  ***** NA
    ## 2006                  ***** NA
    ## 2007                  ***** NA
    ## 2008                    100 NA
    ## 2009                  ***** NA
    ## 2010                  ***** NA
    ## 2011                  ***** NA
    ## 2012                  ***** NA
    ## 2013                  ***** NA
    ## 2014                  ***** NA
    ## 2015                  ***** NA
    ## 2016                  ***** NA
    ## 2017                  ***** NA
    ## 2018                  ***** NA
    ## 2019                  ***** NA
    ## 2020                  ***** NA
    ## 2021                  ***** NA
    ## 2022                  ***** NA
    ## 2023                  ***** NA
    ## 2024                     92 NA
    ## 2025                  ***** NA
    ## 2026                  ***** NA
    ## 2027                  ***** NA
    ## 2028                  ***** NA
    ## 2029                  ***** NA
    ## 2030                  ***** NA
    ## 2031                  ***** NA
    ## 2032                  ***** NA
    ## 2033                     92 NA
    ## 2034                  ***** NA
    ## 2035                     70 NA
    ## 2036                  ***** NA
    ## 2037                  ***** NA
    ## 2038                  ***** NA
    ## 2039                  ***** NA
    ## 2040                  ***** NA
    ## 2041                  ***** NA
    ## 2042                  ***** NA
    ## 2043                  ***** NA
    ## 2044                  ***** NA
    ## 2045                  ***** NA
    ## 2046                  ***** NA
    ## 2047                  ***** NA
    ## 2048                  ***** NA
    ## 2049                  ***** NA
    ## 2050                  ***** NA
    ## 2051                  ***** NA
    ## 2052                  ***** NA
    ## 2053                  ***** NA
    ## 2054                  ***** NA
    ## 2055                  ***** NA
    ## 2056                  ***** NA
    ## 2057                  ***** NA
    ## 2058                  ***** NA
    ## 2059                  ***** NA
    ## 2060                  ***** NA
    ## 2061                  ***** NA
    ## 2062                  ***** NA
    ## 2063                  ***** NA
    ## 2064                  ***** NA
    ## 2065                  ***** NA
    ## 2066                  ***** NA
    ## 2067                  ***** NA
    ## 2068                  ***** NA
    ## 2069                  ***** NA
    ## 2070                  ***** NA
    ## 2071                  ***** NA
    ## 2072                  ***** NA
    ## 2073                  ***** NA
    ## 2074                  ***** NA
    ## 2075                  ***** NA
    ## 2076                  ***** NA
    ## 2077                  ***** NA
    ## 2078                  ***** NA
    ## 2079                  ***** NA
    ## 2080                  ***** NA
    ## 2081                  ***** NA
    ## 2082                  ***** NA
    ## 2083                  ***** NA
    ## 2084                  ***** NA
    ## 2085                  ***** NA
    ## 2086                  ***** NA
    ## 2087                  ***** NA
    ## 2088                  ***** NA
    ## 2089                  ***** NA
    ## 2090                  ***** NA
    ## 2091                  ***** NA
    ## 2092                  ***** NA
    ## 2093                  ***** NA
    ## 2094                  ***** NA
    ## 2095                  ***** NA
    ## 2096                  ***** NA
    ## 2097                  ***** NA
    ## 2098                  ***** NA
    ## 2099                  ***** NA
    ## 2100                  ***** NA
    ## 2101                  ***** NA
    ## 2102                  ***** NA
    ## 2103                  ***** NA
    ## 2104                  ***** NA
    ## 2105                  ***** NA
    ## 2106                  ***** NA
    ## 2107                  ***** NA
    ## 2108                  ***** NA
    ## 2109                  ***** NA
    ## 2110                  ***** NA
    ## 2111                  ***** NA
    ## 2112                  ***** NA
    ## 2113                  ***** NA
    ## 2114                  ***** NA
    ## 2115                  ***** NA
    ## 2116                  ***** NA
    ## 2117                  ***** NA
    ## 2118                  ***** NA
    ## 2119                  ***** NA
    ## 2120                  ***** NA
    ## 2121                  ***** NA
    ## 2122                  ***** NA
    ## 2123                  ***** NA
    ## 2124                  ***** NA
    ## 2125                  ***** NA
    ## 2126                  ***** NA
    ## 2127                  ***** NA
    ## 2128                  ***** NA
    ## 2129                  ***** NA
    ## 2130                  ***** NA
    ## 2131                  ***** NA
    ## 2132                  ***** NA
    ## 2133                  ***** NA
    ## 2134                  ***** NA
    ## 2135                  ***** NA
    ## 2136                  ***** NA
    ## 2137                  ***** NA
    ## 2138                  ***** NA
    ## 2139                  ***** NA
    ## 2140                  ***** NA
    ## 2141                  ***** NA
    ## 2142                  ***** NA
    ## 2143                  ***** NA
    ## 2144                  ***** NA
    ## 2145                  ***** NA
    ## 2146                  ***** NA
    ## 2147                  ***** NA
    ## 2148                  ***** NA
    ## 2149                  ***** NA
    ## 2150                  ***** NA
    ## 2151                  ***** NA
    ## 2152                  ***** NA
    ## 2153                  ***** NA
    ## 2154                  ***** NA
    ## 2155                  ***** NA
    ## 2156                  ***** NA
    ## 2157                  ***** NA
    ## 2158                  ***** NA
    ## 2159                  ***** NA
    ## 2160                  ***** NA
    ## 2161                  ***** NA
    ## 2162                  ***** NA
    ## 2163                  ***** NA
    ## 2164                  ***** NA
    ## 2165                  ***** NA
    ## 2166                  ***** NA
    ## 2167                  ***** NA
    ## 2168                  ***** NA
    ## 2169                  ***** NA
    ## 2170                  ***** NA
    ## 2171                  ***** NA
    ## 2172                  ***** NA
    ## 2173                  ***** NA
    ## 2174                  ***** NA
    ## 2175                  ***** NA
    ## 2176                  ***** NA
    ## 2177                  ***** NA
    ## 2178                  ***** NA
    ## 2179                  ***** NA
    ## 2180                  ***** NA
    ## 2181                  ***** NA
    ## 2182                  ***** NA
    ## 2183                  ***** NA
    ## 2184                  ***** NA
    ## 2185                  ***** NA
    ## 2186                  ***** NA
    ## 2187                  ***** NA
    ## 2188                  ***** NA
    ## 2189                  ***** NA
    ## 2190                  ***** NA
    ## 2191                  ***** NA
    ## 2192                  ***** NA
    ## 2193                  ***** NA
    ## 2194                  ***** NA
    ## 2195                  ***** NA
    ## 2196                  ***** NA
    ## 2197                  ***** NA
    ## 2198                  ***** NA
    ## 2199                  ***** NA
    ## 2200                  ***** NA
    ## 2201                  ***** NA
    ## 2202                  ***** NA
    ## 2203                  ***** NA
    ## 2204                  ***** NA
    ## 2205                  ***** NA
    ## 2206                  ***** NA
    ## 2207                  ***** NA
    ## 2208                  ***** NA
    ## 2209                  ***** NA
    ## 2210                  ***** NA
    ## 2211                  ***** NA
    ## 2212                  ***** NA
    ## 2213                  ***** NA
    ## 2214                  ***** NA
    ## 2215                  ***** NA
    ## 2216                  ***** NA
    ## 2217                  ***** NA
    ## 2218                  ***** NA
    ## 2219                  ***** NA
    ## 2220                    144 NA
    ## 2221                  ***** NA
    ## 2222                  ***** NA
    ## 2223                  ***** NA
    ## 2224                  ***** NA
    ## 2225                  ***** NA
    ## 2226                  ***** NA
    ## 2227                  ***** NA
    ## 2228                  ***** NA
    ## 2229                  ***** NA
    ## 2230                  ***** NA
    ## 2231                  ***** NA
    ## 2232                  ***** NA
    ## 2233                  ***** NA
    ## 2234                  ***** NA
    ## 2235                  ***** NA
    ## 2236                  ***** NA
    ## 2237                    106 NA
    ## 2238                  ***** NA
    ## 2239                  ***** NA
    ## 2240                  ***** NA
    ## 2241                  ***** NA
    ## 2242                  ***** NA
    ## 2243                  ***** NA
    ## 2244                    124 NA
    ## 2245                  ***** NA
    ## 2246                  ***** NA
    ## 2247                  ***** NA
    ## 2248                  ***** NA
    ## 2249                  ***** NA
    ## 2250                  ***** NA
    ## 2251                  ***** NA
    ## 2252                  ***** NA
    ## 2253                  ***** NA
    ## 2254                  ***** NA
    ## 2255                  ***** NA
    ## 2256                  ***** NA
    ## 2257                  ***** NA
    ## 2258                  ***** NA
    ## 2259                  ***** NA
    ## 2260                  ***** NA
    ## 2261                  ***** NA
    ## 2262                  ***** NA
    ## 2263                  ***** NA
    ## 2264                  ***** NA
    ## 2265                  ***** NA
    ## 2266                  ***** NA
    ## 2267                  ***** NA
    ## 2268                  ***** NA
    ## 2269                  ***** NA
    ## 2270                  ***** NA
    ## 2271                  ***** NA
    ## 2272                  ***** NA
    ## 2273                  ***** NA
    ## 2274                  ***** NA
    ## 2275                  ***** NA
    ## 2276                  ***** NA
    ## 2277                  ***** NA
    ## 2278                  ***** NA
    ## 2279                  ***** NA
    ## 2280                  ***** NA
    ## 2281                  ***** NA
    ## 2282                  ***** NA
    ## 2283                  ***** NA
    ## 2284                  ***** NA
    ## 2285                  ***** NA
    ## 2286                  ***** NA
    ## 2287                  ***** NA
    ## 2288                  ***** NA
    ## 2289                  ***** NA
    ## 2290                  ***** NA
    ## 2291                  ***** NA
    ## 2292                  ***** NA
    ## 2293                  ***** NA
    ## 2294                  ***** NA
    ## 2295                  ***** NA
    ## 2296                  ***** NA
    ## 2297                  ***** NA
    ## 2298                  ***** NA
    ## 2299                  ***** NA
    ## 2300                  ***** NA
    ## 2301                  ***** NA
    ## 2302                  ***** NA
    ## 2303                  ***** NA
    ## 2304                  ***** NA
    ## 2305                  ***** NA
    ## 2306                  ***** NA
    ## 2307                  ***** NA
    ## 2308                  ***** NA
    ## 2309                  ***** NA
    ## 2310                  ***** NA
    ## 2311                  ***** NA
    ## 2312                  ***** NA
    ## 2313                  ***** NA
    ## 2314                  ***** NA
    ## 2315                  ***** NA
    ## 2316                  ***** NA
    ## 2317                  ***** NA
    ## 2318                  ***** NA
    ## 2319                  ***** NA
    ## 2320                  ***** NA
    ## 2321                  ***** NA
    ## 2322                  ***** NA
    ## 2323                  ***** NA
    ## 2324                  ***** NA
    ## 2325                  ***** NA
    ## 2326                  ***** NA
    ## 2327                  ***** NA
    ## 2328                  ***** NA
    ## 2329                  ***** NA
    ## 2330                  ***** NA
    ## 2331                  ***** NA
    ## 2332                  ***** NA
    ## 2333                  ***** NA
    ## 2334                  ***** NA
    ## 2335                  ***** NA
    ## 2336                  ***** NA
    ## 2337                  ***** NA
    ## 2338                  ***** NA
    ## 2339                  ***** NA
    ## 2340                  ***** NA
    ## 2341                  ***** NA
    ## 2342                  ***** NA
    ## 2343                  ***** NA
    ## 2344                  ***** NA
    ## 2345                  ***** NA
    ## 2346                  ***** NA
    ## 2347                  ***** NA
    ## 2348                  ***** NA
    ## 2349                  ***** NA
    ## 2350                  ***** NA
    ## 2351                  ***** NA
    ## 2352                  ***** NA
    ## 2353                  ***** NA
    ## 2354                  ***** NA
    ## 2355                  ***** NA
    ## 2356                  ***** NA
    ## 2357                  ***** NA
    ## 2358                  ***** NA
    ## 2359                  ***** NA
    ## 2360                  ***** NA
    ## 2361                  ***** NA
    ## 2362                  ***** NA
    ## 2363                  ***** NA
    ## 2364                  ***** NA
    ## 2365                  ***** NA
    ## 2366                  ***** NA
    ## 2367                  ***** NA
    ## 2368                  ***** NA
    ## 2369                  ***** NA
    ## 2370                  ***** NA
    ## 2371                  ***** NA
    ## 2372                  ***** NA
    ## 2373                    168 NA
    ## 2374                  ***** NA
    ## 2375                  ***** NA
    ## 2376                  ***** NA
    ## 2377                  ***** NA
    ## 2378                  ***** NA
    ## 2379                  ***** NA
    ## 2380                  ***** NA
    ## 2381                  ***** NA
    ## 2382                  ***** NA
    ## 2383                  ***** NA
    ## 2384                  ***** NA
    ## 2385                  ***** NA
    ## 2386                  ***** NA
    ## 2387                  ***** NA
    ## 2388                  ***** NA
    ## 2389                  ***** NA
    ## 2390                     96 NA
    ## 2391                  ***** NA
    ## 2392                    112 NA
    ## 2393                  ***** NA
    ## 2394                     95 NA
    ## 2395                  ***** NA
    ## 2396                  ***** NA
    ## 2397                    112 NA
    ## 2398                  ***** NA
    ## 2399                  ***** NA
    ## 2400                     96 NA
    ## 2401                  ***** NA
    ## 2402                  ***** NA
    ## 2403                  ***** NA
    ## 2404                  ***** NA
    ## 2405                  ***** NA
    ## 2406                  ***** NA
    ## 2407                    168 NA
    ## 2408                  ***** NA
    ## 2409                  ***** NA
    ## 2410                  ***** NA
    ## 2411                  ***** NA
    ## 2412                  ***** NA
    ## 2413                  ***** NA
    ## 2414                  ***** NA
    ## 2415                  ***** NA
    ## 2416                     95 NA
    ## 2417                    104 NA
    ## 2418                  ***** NA
    ## 2419                  ***** NA
    ## 2420                  ***** NA
    ## 2421                  ***** NA
    ## 2422                    104 NA
    ## 2423                  ***** NA
    ## 2424                  ***** NA
    ## 2425                  ***** NA
    ## 2426                  ***** NA
    ## 2427                  ***** NA
    ## 2428                  ***** NA
    ## 2429                  ***** NA
    ## 2430                  ***** NA
    ## 2431                  ***** NA
    ## 2432                  ***** NA
    ## 2433                  ***** NA
    ## 2434                  ***** NA
    ## 2435                  ***** NA
    ## 2436                  ***** NA
    ## 2437                  ***** NA
    ## 2438                  ***** NA
    ## 2439                  ***** NA
    ## 2440                  ***** NA
    ## 2441                  ***** NA
    ## 2442                  ***** NA
    ## 2443                  ***** NA
    ## 2444                  ***** NA
    ## 2445                  ***** NA
    ## 2446                  ***** NA
    ## 2447                  ***** NA
    ## 2448                  ***** NA
    ## 2449                  ***** NA
    ## 2450                  ***** NA
    ## 2451                  ***** NA
    ## 2452                  ***** NA
    ## 2453                  ***** NA
    ## 2454                  ***** NA
    ## 2455                  ***** NA
    ## 2456                  ***** NA
    ## 2457                  ***** NA
    ## 2458                  ***** NA
    ## 2459                  ***** NA
    ## 2460                  ***** NA
    ## 2461                  ***** NA
    ## 2462                  ***** NA
    ## 2463                  ***** NA
    ## 2464                  ***** NA
    ## 2465                  ***** NA
    ## 2466                  ***** NA
    ## 2467                  ***** NA
    ## 2468                  ***** NA
    ## 2469                  ***** NA
    ## 2470                  ***** NA
    ## 2471                  ***** NA
    ## 2472                  ***** NA
    ## 2473                  ***** NA
    ## 2474                  ***** NA
    ## 2475                  ***** NA
    ## 2476                  ***** NA
    ## 2477                  ***** NA
    ## 2478                  ***** NA
    ## 2479                  ***** NA
    ## 2480                  ***** NA
    ## 2481                  ***** NA
    ## 2482                  ***** NA
    ## 2483                  ***** NA
    ## 2484                  ***** NA
    ## 2485                  ***** NA
    ## 2486                  ***** NA
    ## 2487                  ***** NA
    ## 2488                  ***** NA
    ## 2489                  ***** NA
    ## 2490                  ***** NA
    ## 2491                  ***** NA
    ## 2492                  ***** NA
    ## 2493                  ***** NA
    ## 2494                  ***** NA
    ## 2495                  ***** NA
    ## 2496                  ***** NA
    ## 2497                  ***** NA
    ## 2498                  ***** NA
    ## 2499                  ***** NA
    ## 2500                  ***** NA
    ## 2501                  ***** NA
    ## 2502                  ***** NA
    ## 2503                  ***** NA
    ## 2504                  ***** NA
    ## 2505                  ***** NA
    ## 2506                  ***** NA
    ## 2507                  ***** NA
    ## 2508                  ***** NA
    ## 2509                  ***** NA
    ## 2510                  ***** NA
    ## 2511                  ***** NA
    ## 2512                  ***** NA
    ## 2513                  ***** NA
    ## 2514                  ***** NA
    ## 2515                  ***** NA
    ## 2516                  ***** NA
    ## 2517                  ***** NA
    ## 2518                  ***** NA
    ## 2519                  ***** NA
    ## 2520                  ***** NA
    ## 2521                  ***** NA
    ## 2522                  ***** NA
    ## 2523                  ***** NA
    ## 2524                  ***** NA
    ## 2525                  ***** NA
    ## 2526                  ***** NA
    ## 2527                  ***** NA
    ## 2528                  ***** NA
    ## 2529                  ***** NA
    ## 2530                    125 NA
    ## 2531                  ***** NA
    ## 2532                  ***** NA
    ## 2533                  ***** NA
    ## 2534                  ***** NA
    ## 2535                  ***** NA
    ## 2536                  ***** NA
    ## 2537                  ***** NA
    ## 2538                  ***** NA
    ## 2539                  ***** NA
    ## 2540                  ***** NA
    ## 2541                    119 NA
    ## 2542                  ***** NA
    ## 2543                  ***** NA
    ## 2544                  ***** NA
    ## 2545                  ***** NA
    ## 2546                  ***** NA
    ## 2547                    128 NA
    ## 2548                  ***** NA
    ## 2549                  ***** NA
    ## 2550                  ***** NA
    ## 2551                  ***** NA
    ## 2552                  ***** NA
    ## 2553                  ***** NA
    ## 2554                  ***** NA
    ## 2555                  ***** NA
    ## 2556                  ***** NA
    ## 2557                  ***** NA
    ## 2558                  ***** NA
    ## 2559                  ***** NA
    ## 2560                  ***** NA
    ## 2561                  ***** NA
    ## 2562                  ***** NA
    ## 2563                  ***** NA
    ## 2564                  ***** NA
    ## 2565                  ***** NA
    ## 2566                  ***** NA
    ## 2567                  ***** NA
    ## 2568                  ***** NA
    ## 2569                  ***** NA
    ## 2570                  ***** NA
    ## 2571                  ***** NA
    ## 2572                  ***** NA
    ## 2573                  ***** NA
    ## 2574                  ***** NA
    ## 2575                    188 NA
    ## 2576                  ***** NA
    ## 2577                    186 NA
    ## 2578                  ***** NA
    ## 2579                  ***** NA
    ## 2580                  ***** NA
    ## 2581                  ***** NA
    ## 2582                  ***** NA
    ## 2583                  ***** NA
    ## 2584                  ***** NA
    ## 2585                  ***** NA
    ## 2586                  ***** NA
    ## 2587                  ***** NA
    ## 2588                  ***** NA
    ## 2589                  ***** NA
    ## 2590                  ***** NA
    ## 2591                  ***** NA
    ## 2592                  ***** NA
    ## 2593                    193 NA
    ## 2594                  ***** NA
    ## 2595                  ***** NA
    ## 2596                  ***** NA
    ## 2597                  ***** NA
    ## 2598                  ***** NA
    ## 2599                  ***** NA
    ## 2600                  ***** NA
    ## 2601                  ***** NA
    ## 2602                    137 NA
    ## 2603                  ***** NA
    ## 2604                  ***** NA
    ## 2605                  ***** NA
    ## 2606                  ***** NA
    ## 2607                  ***** NA
    ## 2608                  ***** NA
    ## 2609                    188 NA
    ## 2610                  ***** NA
    ## 2611                    146 NA
    ## 2612                  ***** NA
    ## 2613                  ***** NA
    ## 2614                  ***** NA
    ## 2615                  ***** NA
    ## 2616                  ***** NA
    ## 2617                  ***** NA
    ## 2618                  ***** NA
    ## 2619                  ***** NA
    ## 2620                  ***** NA
    ## 2621                  ***** NA
    ## 2622                  ***** NA
    ## 2623                  ***** NA
    ## 2624                  ***** NA
    ## 2625                  ***** NA
    ## 2626                  ***** NA
    ## 2627                  ***** NA
    ## 2628                  ***** NA
    ## 2629                  ***** NA
    ## 2630                  ***** NA
    ## 2631                  ***** NA
    ## 2632                  ***** NA
    ## 2633                  ***** NA
    ## 2634                  ***** NA
    ## 2635                  ***** NA
    ## 2636                  ***** NA
    ## 2637                  ***** NA
    ## 2638                  ***** NA
    ## 2639                  ***** NA
    ## 2640                  ***** NA
    ## 2641                  ***** NA
    ## 2642                    153 NA
    ## 2643                  ***** NA
    ## 2644                  ***** NA
    ## 2645                  ***** NA
    ## 2646                  ***** NA
    ## 2647                  ***** NA
    ## 2648                  ***** NA
    ## 2649                  ***** NA
    ## 2650                  ***** NA
    ## 2651                  ***** NA
    ## 2652                  ***** NA
    ## 2653                  ***** NA
    ## 2654                  ***** NA
    ## 2655                    181 NA
    ## 2656                    118 NA
    ## 2657                  ***** NA
    ## 2658                  ***** NA
    ## 2659                     69 NA
    ## 2660                  ***** NA
    ## 2661                  ***** NA
    ## 2662                  ***** NA
    ## 2663                  ***** NA
    ## 2664                  ***** NA
    ## 2665                  ***** NA
    ## 2666                    181 NA
    ## 2667                  ***** NA
    ## 2668                  ***** NA
    ## 2669                  ***** NA
    ## 2670                  ***** NA
    ## 2671                  ***** NA
    ## 2672                  ***** NA
    ## 2673                  ***** NA
    ## 2674                  ***** NA
    ## 2675                     47 NA
    ## 2676                  ***** NA
    ## 2677                  ***** NA
    ## 2678                  ***** NA
    ## 2679                  ***** NA
    ## 2680                    193 NA
    ## 2681                  ***** NA
    ## 2682                  ***** NA
    ## 2683                  ***** NA
    ## 2684                  ***** NA
    ## 2685                  ***** NA
    ## 2686                  ***** NA
    ## 2687                  ***** NA
    ## 2688                  ***** NA
    ## 2689                  ***** NA
    ## 2690                  ***** NA
    ## 2691                  ***** NA
    ## 2692                  ***** NA
    ## 2693                  ***** NA
    ## 2694                  ***** NA
    ## 2695                  ***** NA
    ## 2696                  ***** NA
    ## 2697                    128 NA
    ## 2698                  ***** NA
    ## 2699                  ***** NA
    ## 2700                  ***** NA
    ## 2701                  ***** NA
    ## 2702                  ***** NA
    ## 2703                  ***** NA
    ## 2704                  ***** NA
    ## 2705                  ***** NA
    ## 2706                  ***** NA
    ## 2707                  ***** NA
    ## 2708                  ***** NA
    ## 2709                  ***** NA
    ## 2710                  ***** NA
    ## 2711                  ***** NA
    ## 2712                  ***** NA
    ## 2713                  ***** NA
    ## 2714                  ***** NA
    ## 2715                  ***** NA
    ## 2716                  ***** NA
    ## 2717                  ***** NA
    ## 2718                  ***** NA
    ## 2719                  ***** NA
    ## 2720                  ***** NA
    ## 2721                    125 NA
    ## 2722                  ***** NA
    ## 2723                  ***** NA
    ## 2724                  ***** NA
    ## 2725                  ***** NA
    ## 2726                  ***** NA
    ## 2727                  ***** NA
    ## 2728                  ***** NA
    ## 2729                  ***** NA
    ## 2730                  ***** NA
    ## 2731                  ***** NA
    ## 2732                  ***** NA
    ## 2733                  ***** NA
    ## 2734                  ***** NA
    ## 2735                  ***** NA
    ## 2736                  ***** NA
    ## 2737                  ***** NA
    ## 2738                  ***** NA
    ## 2739                  ***** NA
    ## 2740                    137 NA
    ## 2741                    118 NA
    ## 2742                  ***** NA
    ## 2743                  ***** NA
    ## 2744                  ***** NA
    ## 2745                  ***** NA
    ## 2746                    187 NA
    ## 2747                  ***** NA
    ## 2748                    153 NA
    ## 2749                  ***** NA
    ## 2750                  ***** NA
    ## 2751                  ***** NA
    ## 2752                  ***** NA
    ## 2753                  ***** NA
    ## 2754                  ***** NA
    ## 2755                  ***** NA
    ## 2756                  ***** NA
    ## 2757                  ***** NA
    ## 2758                  ***** NA
    ## 2759                  ***** NA
    ## 2760                  ***** NA
    ## 2761                  ***** NA
    ## 2762                  ***** NA
    ## 2763                  ***** NA
    ## 2764                  ***** NA
    ## 2765                  ***** NA
    ## 2766                  ***** NA
    ## 2767                  ***** NA
    ## 2768                  ***** NA
    ## 2769                  ***** NA
    ## 2770                  ***** NA
    ## 2771                  ***** NA
    ## 2772                  ***** NA
    ## 2773                  ***** NA
    ## 2774                  ***** NA
    ## 2775                  ***** NA
    ## 2776                  ***** NA
    ## 2777                  ***** NA
    ## 2778                  ***** NA
    ## 2779                  ***** NA
    ## 2780                  ***** NA
    ## 2781                  ***** NA
    ## 2782                  ***** NA
    ## 2783                    168 NA
    ## 2784                  ***** NA
    ## 2785                  ***** NA
    ## 2786                  ***** NA
    ## 2787                  ***** NA
    ## 2788                  ***** NA
    ## 2789                  ***** NA
    ## 2790                  ***** NA
    ## 2791                  ***** NA
    ## 2792                  ***** NA
    ## 2793                  ***** NA
    ## 2794                    168 NA
    ## 2795                  ***** NA
    ## 2796                  ***** NA
    ## 2797                  ***** NA
    ## 2798                  ***** NA
    ## 2799                  ***** NA
    ## 2800                  ***** NA
    ## 2801                  ***** NA
    ## 2802                  ***** NA
    ## 2803                  ***** NA
    ## 2804                  ***** NA
    ## 2805                  ***** NA
    ## 2806                  ***** NA
    ## 2807                  ***** NA
    ## 2808                  ***** NA
    ## 2809                  ***** NA
    ## 2810                  ***** NA
    ## 2811                  ***** NA
    ## 2812                  ***** NA
    ## 2813                  ***** NA
    ## 2814                  ***** NA
    ## 2815                  ***** NA
    ## 2816                  ***** NA
    ## 2817                  ***** NA
    ## 2818                  ***** NA
    ## 2819                  ***** NA
    ## 2820                  ***** NA
    ## 2821                  ***** NA
    ## 2822                  ***** NA
    ## 2823                  ***** NA
    ## 2824                  ***** NA
    ## 2825                  ***** NA
    ## 2826                  ***** NA
    ## 2827                  ***** NA
    ## 2828                  ***** NA
    ## 2829                  ***** NA
    ## 2830                  ***** NA
    ## 2831                  ***** NA
    ## 2832                  ***** NA
    ## 2833                  ***** NA
    ## 2834                  ***** NA
    ## 2835                  ***** NA
    ## 2836                  ***** NA
    ## 2837                  ***** NA
    ## 2838                  ***** NA
    ## 2839                  ***** NA
    ## 2840                  ***** NA
    ## 2841                  ***** NA
    ## 2842                  ***** NA
    ## 2843                  ***** NA
    ## 2844                  ***** NA
    ## 2845                  ***** NA
    ## 2846                  ***** NA
    ## 2847                  ***** NA
    ## 2848                  ***** NA
    ## 2849                  ***** NA
    ## 2850                  ***** NA
    ## 2851                  ***** NA
    ## 2852                  ***** NA
    ## 2853                  ***** NA
    ## 2854                  ***** NA
    ## 2855                  ***** NA
    ## 2856                  ***** NA
    ## 2857                  ***** NA
    ## 2858                  ***** NA
    ## 2859                  ***** NA
    ## 2860                  ***** NA
    ## 2861                  ***** NA
    ## 2862                  ***** NA
    ## 2863                  ***** NA
    ## 2864                  ***** NA
    ## 2865                  ***** NA
    ## 2866                  ***** NA
    ## 2867                  ***** NA
    ## 2868                  ***** NA
    ## 2869                  ***** NA
    ## 2870                  ***** NA
    ## 2871                  ***** NA
    ## 2872                  ***** NA
    ## 2873                  ***** NA
    ## 2874                  ***** NA
    ## 2875                  ***** NA
    ## 2876                  ***** NA
    ## 2877                  ***** NA
    ## 2878                  ***** NA
    ## 2879                  ***** NA
    ## 2880                  ***** NA
    ## 2881                  ***** NA
    ## 2882                  ***** NA
    ## 2883                  ***** NA
    ## 2884                  ***** NA
    ## 2885                  ***** NA
    ## 2886                  ***** NA
    ## 2887                  ***** NA
    ## 2888                  ***** NA
    ## 2889                  ***** NA
    ## 2890                  ***** NA
    ## 2891                  ***** NA
    ## 2892                  ***** NA
    ## 2893                  ***** NA
    ## 2894                  ***** NA
    ## 2895                  ***** NA
    ## 2896                  ***** NA
    ## 2897                  ***** NA
    ## 2898                  ***** NA
    ## 2899                  ***** NA
    ## 2900                  ***** NA
    ## 2901                  ***** NA
    ## 2902                  ***** NA
    ## 2903                  ***** NA
    ## 2904                  ***** NA
    ## 2905                  ***** NA
    ## 2906                  ***** NA
    ## 2907                  ***** NA
    ## 2908                  ***** NA
    ## 2909                  ***** NA
    ## 2910                  ***** NA
    ## 2911                  ***** NA
    ## 2912                  ***** NA
    ## 2913                  ***** NA
    ## 2914                  ***** NA
    ## 2915                  ***** NA
    ## 2916                  ***** NA
    ## 2917                  ***** NA
    ## 2918                  ***** NA
    ## 2919                  ***** NA
    ## 2920                  ***** NA
    ## 2921                  ***** NA
    ## 2922                  ***** NA
    ## 2923                  ***** NA
    ## 2924                  ***** NA
    ## 2925                  ***** NA
    ## 2926                  ***** NA
    ## 2927                  ***** NA
    ## 2928                  ***** NA
    ## 2929                  ***** NA
    ## 2930                  ***** NA
    ## 2931                  ***** NA
    ## 2932                  ***** NA
    ## 2933                  ***** NA
    ## 2934                  ***** NA
    ## 2935                  ***** NA
    ## 2936                  ***** NA
    ## 2937                  ***** NA
    ## 2938                  ***** NA
    ## 2939                  ***** NA
    ## 2940                  ***** NA
    ## 2941                  ***** NA
    ## 2942                  ***** NA
    ## 2943                  ***** NA
    ## 2944                  ***** NA
    ## 2945                  ***** NA
    ## 2946                  ***** NA
    ## 2947                  ***** NA
    ## 2948                  ***** NA
    ## 2949                  ***** NA
    ## 2950                  ***** NA
    ## 2951                  ***** NA
    ## 2952                  ***** NA
    ## 2953                  ***** NA
    ## 2954                  ***** NA
    ## 2955                  ***** NA
    ## 2956                  ***** NA
    ## 2957                  ***** NA
    ## 2958                  ***** NA
    ## 2959                  ***** NA
    ## 2960                  ***** NA
    ## 2961                  ***** NA
    ## 2962                  ***** NA
    ## 2963                  ***** NA
    ## 2964                  ***** NA
    ## 2965                  ***** NA
    ## 2966                  ***** NA
    ## 2967                  ***** NA
    ## 2968                  ***** NA
    ## 2969                  ***** NA
    ## 2970                  ***** NA
    ## 2971                  ***** NA
    ## 2972                  ***** NA
    ## 2973                  ***** NA
    ## 2974                  ***** NA
    ## 2975                  ***** NA
    ## 2976                  ***** NA
    ## 2977                  ***** NA
    ## 2978                  ***** NA
    ## 2979                  ***** NA
    ## 2980                  ***** NA
    ## 2981                  ***** NA
    ## 2982                  ***** NA
    ## 2983                  ***** NA
    ## 2984                  ***** NA
    ## 2985                  ***** NA
    ## 2986                  ***** NA
    ## 2987                  ***** NA
    ## 2988                  ***** NA
    ## 2989                  ***** NA
    ## 2990                  ***** NA
    ## 2991                  ***** NA
    ## 2992                  ***** NA
    ## 2993                  ***** NA
    ## 2994                  ***** NA
    ## 2995                  ***** NA
    ## 2996                  ***** NA
    ## 2997                  ***** NA
    ## 2998                  ***** NA
    ## 2999                  ***** NA
    ## 3000                  ***** NA
    ## 3001                  ***** NA
    ## 3002                  ***** NA
    ## 3003                  ***** NA
    ## 3004                  ***** NA
    ## 3005                  ***** NA
    ## 3006                  ***** NA
    ## 3007                  ***** NA
    ## 3008                  ***** NA
    ## 3009                  ***** NA
    ## 3010                  ***** NA
    ## 3011                  ***** NA
    ## 3012                  ***** NA
    ## 3013                  ***** NA
    ## 3014                  ***** NA
    ## 3015                  ***** NA
    ## 3016                  ***** NA
    ## 3017                  ***** NA
    ## 3018                  ***** NA
    ## 3019                  ***** NA
    ## 3020                  ***** NA
    ## 3021                  ***** NA
    ## 3022                  ***** NA
    ## 3023                  ***** NA
    ## 3024                  ***** NA
    ## 3025                  ***** NA
    ## 3026                  ***** NA
    ## 3027                  ***** NA
    ## 3028                  ***** NA
    ## 3029                  ***** NA
    ## 3030                  ***** NA
    ## 3031                  ***** NA
    ## 3032                  ***** NA
    ## 3033                  ***** NA
    ## 3034                  ***** NA
    ## 3035                  ***** NA
    ## 3036                  ***** NA
    ## 3037                  ***** NA
    ## 3038                  ***** NA
    ## 3039                  ***** NA
    ## 3040                  ***** NA
    ## 3041                  ***** NA
    ## 3042                  ***** NA
    ## 3043                  ***** NA
    ## 3044                  ***** NA
    ## 3045                  ***** NA
    ## 3046                  ***** NA
    ## 3047                  ***** NA
    ## 3048                  ***** NA
    ## 3049                  ***** NA
    ## 3050                  ***** NA
    ## 3051                  ***** NA
    ## 3052                  ***** NA
    ## 3053                  ***** NA
    ## 3054                  ***** NA
    ## 3055                  ***** NA
    ## 3056                  ***** NA
    ## 3057                  ***** NA
    ## 3058                  ***** NA
    ## 3059                  ***** NA
    ## 3060                  ***** NA
    ## 3061                  ***** NA
    ## 3062                  ***** NA
    ## 3063                  ***** NA
    ## 3064                  ***** NA
    ## 3065                  ***** NA
    ## 3066                  ***** NA
    ## 3067                  ***** NA
    ## 3068                  ***** NA
    ## 3069                  ***** NA
    ## 3070                  ***** NA
    ## 3071                  ***** NA
    ## 3072                  ***** NA
    ## 3073                  ***** NA
    ## 3074                  ***** NA
    ## 3075                  ***** NA
    ## 3076                  ***** NA
    ## 3077                  ***** NA
    ## 3078                  ***** NA
    ## 3079                  ***** NA
    ## 3080                  ***** NA
    ## 3081                  ***** NA
    ## 3082                  ***** NA
    ## 3083                  ***** NA
    ## 3084                  ***** NA
    ## 3085                  ***** NA
    ## 3086                  ***** NA
    ## 3087                  ***** NA
    ## 3088                  ***** NA
    ## 3089                  ***** NA
    ## 3090                  ***** NA
    ## 3091                  ***** NA
    ## 3092                  ***** NA
    ## 3093                  ***** NA
    ## 3094                  ***** NA
    ## 3095                  ***** NA
    ## 3096                  ***** NA
    ## 3097                  ***** NA
    ## 3098                  ***** NA
    ## 3099                  ***** NA
    ## 3100                  ***** NA
    ## 3101                  ***** NA
    ## 3102                  ***** NA
    ## 3103                  ***** NA
    ## 3104                  ***** NA
    ## 3105                  ***** NA
    ## 3106                  ***** NA
    ## 3107                  ***** NA
    ## 3108                  ***** NA
    ## 3109                  ***** NA
    ## 3110                  ***** NA
    ## 3111                  ***** NA
    ## 3112                  ***** NA
    ## 3113                  ***** NA
    ## 3114                  ***** NA
    ## 3115                  ***** NA
    ## 3116                  ***** NA
    ## 3117                  ***** NA
    ## 3118                  ***** NA
    ## 3119                  ***** NA
    ## 3120                  ***** NA
    ## 3121                  ***** NA
    ## 3122                  ***** NA
    ## 3123                  ***** NA
    ## 3124                  ***** NA
    ## 3125                  ***** NA
    ## 3126                  ***** NA
    ## 3127                  ***** NA
    ## 3128                  ***** NA
    ## 3129                  ***** NA
    ## 3130                  ***** NA
    ## 3131                  ***** NA
    ## 3132                  ***** NA
    ## 3133                  ***** NA
    ## 3134                  ***** NA
    ## 3135                  ***** NA
    ## 3136                  ***** NA
    ## 3137                  ***** NA
    ## 3138                  ***** NA
    ## 3139                  ***** NA
    ## 3140                  ***** NA
    ## 3141                  ***** NA
    ## 3142                  ***** NA
    ## 3143                  ***** NA
    ## 3144                  ***** NA
    ## 3145                  ***** NA
    ## 3146                  ***** NA
    ## 3147                  ***** NA
    ## 3148                  ***** NA
    ## 3149                  ***** NA
    ## 3150                  ***** NA
    ## 3151                  ***** NA
    ## 3152                  ***** NA
    ## 3153                  ***** NA
    ## 3154                  ***** NA
    ## 3155                  ***** NA
    ## 3156                  ***** NA
    ## 3157                  ***** NA
    ## 3158                  ***** NA
    ## 3159                  ***** NA
    ## 3160                  ***** NA
    ## 3161                  ***** NA
    ## 3162                  ***** NA
    ## 3163                  ***** NA
    ## 3164                  ***** NA
    ## 3165                  ***** NA
    ## 3166                  ***** NA
    ## 3167                  ***** NA
    ## 3168                    218 NA
    ## 3169                  ***** NA
    ## 3170                  ***** NA
    ## 3171                  ***** NA
    ## 3172                  ***** NA
    ## 3173                  ***** NA
    ## 3174                  ***** NA
    ## 3175                  ***** NA
    ## 3176                  ***** NA
    ## 3177                  ***** NA
    ## 3178                  ***** NA
    ## 3179                  ***** NA
    ## 3180                  ***** NA
    ## 3181                  ***** NA
    ## 3182                  ***** NA
    ## 3183                  ***** NA
    ## 3184                  ***** NA
    ## 3185                  ***** NA
    ## 3186                  ***** NA
    ## 3187                  ***** NA
    ## 3188                  ***** NA
    ## 3189                  ***** NA
    ## 3190                  ***** NA
    ## 3191                    218 NA
    ## 3192                  ***** NA
    ## 3193                  ***** NA
    ## 3194                  ***** NA
    ## 3195                  ***** NA
    ## 3196                  ***** NA
    ## 3197                  ***** NA
    ## 3198                  ***** NA
    ## 3199                  ***** NA
    ## 3200                  ***** NA
    ## 3201                  ***** NA
    ## 3202                  ***** NA
    ## 3203                  ***** NA
    ## 3204                  ***** NA
    ## 3205                  ***** NA
    ## 3206                  ***** NA
    ## 3207                  ***** NA
    ## 3208                  ***** NA
    ## 3209                  ***** NA
    ## 3210                  ***** NA
    ## 3211                  ***** NA
    ## 3212                  ***** NA
    ## 3213                  ***** NA
    ## 3214                  ***** NA
    ## 3215                  ***** NA
    ## 3216                  ***** NA
    ## 3217                  ***** NA
    ## 3218                  ***** NA
    ## 3219                  ***** NA
    ## 3220                  ***** NA
    ## 3221                  ***** NA

``` r
df_covid %>% glimpse
```

    ## Rows: 2,502,832
    ## Columns: 6
    ## $ date   <date> 2020-01-21, 2020-01-22, 2020-01-23, 2020-01-24, 2020-01-24, 20…
    ## $ county <chr> "Snohomish", "Snohomish", "Snohomish", "Cook", "Snohomish", "Or…
    ## $ state  <chr> "Washington", "Washington", "Washington", "Illinois", "Washingt…
    ## $ fips   <chr> "53061", "53061", "53061", "17031", "53061", "06059", "17031", …
    ## $ cases  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ deaths <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …

To join these datasets, we’ll need to use [FIPS county
codes](https://en.wikipedia.org/wiki/FIPS_county_code).\[2\] The last
`5` digits of the `id` column in `df_pop` is the FIPS county code, while
the NYT data `df_covid` already contains the `fips`.

### **q3** Process the `id` column of `df_pop` to create a `fips` column.

``` r
## TASK: Create a `fips` column by extracting the county code
colnames(df_pop) <- df_pop[1, ]

df_rename <- df_pop[-1, ]

df_q3 <- df_rename %>%
  mutate(fips = substr(`Geography`, nchar(`Geography`) - 4, nchar(`Geography`)))
```

Use the following test to check your answer.

``` r
## NOTE: No need to change this
## Check known county
assertthat::assert_that(
              (df_q3 %>%
              filter(str_detect(`Geographic Area Name`, "Autauga County")) %>%
              pull(fips)) == "01001"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

### **q4** Join `df_covid` with `df_q3` by the `fips` column. Use the proper type of join to preserve *only* the rows in `df_covid`.

``` r
## TASK: Join df_covid and df_q3 by fips.
df_q4 <- left_join(df_covid, df_q3, by = "fips")
```

Use the following test to check your answer.

``` r
## NOTE: No need to change this
if (!any(df_q4 %>% pull(fips) %>% str_detect(., "02105"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 contains a row for the Hoonah-Angoon Census Area (AK),",
    "which is not in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_q4 %>% pull(fips) %>% str_detect(., "78010"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 does not include St. Croix, US Virgin Islands,",
    "which is in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

For convenience, I down-select some columns and produce more convenient
column names.

``` r
## NOTE: No need to change; run this to produce a more convenient tibble
df_data <-
  df_q4 %>%
  select(
    date,
    county,
    state,
    fips,
    cases,
    deaths,
    population = `Estimate!!Total`
  )
df_data
```

    ## # A tibble: 2,502,832 × 7
    ##    date       county      state      fips  cases deaths population
    ##    <date>     <chr>       <chr>      <chr> <dbl>  <dbl> <chr>     
    ##  1 2020-01-21 Snohomish   Washington 53061     1      0 786620    
    ##  2 2020-01-22 Snohomish   Washington 53061     1      0 786620    
    ##  3 2020-01-23 Snohomish   Washington 53061     1      0 786620    
    ##  4 2020-01-24 Cook        Illinois   17031     1      0 5223719   
    ##  5 2020-01-24 Snohomish   Washington 53061     1      0 786620    
    ##  6 2020-01-25 Orange      California 06059     1      0 3164182   
    ##  7 2020-01-25 Cook        Illinois   17031     1      0 5223719   
    ##  8 2020-01-25 Snohomish   Washington 53061     1      0 786620    
    ##  9 2020-01-26 Maricopa    Arizona    04013     1      0 4253913   
    ## 10 2020-01-26 Los Angeles California 06037     1      0 10098052  
    ## # ℹ 2,502,822 more rows

# Analyze

<!-- -------------------------------------------------- -->

Now that we’ve done the hard work of loading and wrangling the data, we
can finally start our analysis. Our first step will be to produce county
population-normalized cases and death counts. Then we will explore the
data.

## Normalize

<!-- ------------------------- -->

### **q5** Use the `population` estimates in `df_data` to normalize `cases` and `deaths` to produce per 100,000 counts \[3\]. Store these values in the columns `cases_per100k` and `deaths_per100k`.

``` r
## TASK: Normalize cases and deaths
df_normalized <- df_data %>%
  filter(!is.na(population)) %>%  
  mutate(
    population = as.numeric(population),
    cases_per100k = cases / population * 100000,
    deaths_per100k = deaths / population * 100000
  )
df_normalized
```

    ## # A tibble: 2,474,010 × 9
    ##    date       county      state      fips  cases deaths population cases_per100k
    ##    <date>     <chr>       <chr>      <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2020-01-21 Snohomish   Washington 53061     1      0     786620       0.127  
    ##  2 2020-01-22 Snohomish   Washington 53061     1      0     786620       0.127  
    ##  3 2020-01-23 Snohomish   Washington 53061     1      0     786620       0.127  
    ##  4 2020-01-24 Cook        Illinois   17031     1      0    5223719       0.0191 
    ##  5 2020-01-24 Snohomish   Washington 53061     1      0     786620       0.127  
    ##  6 2020-01-25 Orange      California 06059     1      0    3164182       0.0316 
    ##  7 2020-01-25 Cook        Illinois   17031     1      0    5223719       0.0191 
    ##  8 2020-01-25 Snohomish   Washington 53061     1      0     786620       0.127  
    ##  9 2020-01-26 Maricopa    Arizona    04013     1      0    4253913       0.0235 
    ## 10 2020-01-26 Los Angeles California 06037     1      0   10098052       0.00990
    ## # ℹ 2,474,000 more rows
    ## # ℹ 1 more variable: deaths_per100k <dbl>

You may use the following test to check your work.

``` r
## NOTE: No need to change this
## Check known county data
if (any(df_normalized %>% pull(date) %>% str_detect(., "2020-01-21"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2020-01-21 not found; did you download the historical data (correct),",
    "or just the most recent data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_normalized %>% pull(date) %>% str_detect(., "2022-05-13"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2022-05-13 not found; did you download the historical data (correct),",
    "or a single year's data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
## Check datatypes
assertthat::assert_that(is.numeric(df_normalized$cases))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$population))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$cases_per100k))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths_per100k))
```

    ## [1] TRUE

``` r
## Check that normalization is correct
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(cases_per100k) - 0.127) < 1e-3
            )
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(deaths_per100k) - 0) < 1e-3
            )
```

    ## [1] TRUE

``` r
print("Excellent!")
```

    ## [1] "Excellent!"

## Guided EDA

<!-- ------------------------- -->

Before turning you loose, let’s complete a couple guided EDA tasks.

### **q6** Compute some summaries

Compute the mean and standard deviation for `cases_per100k` and
`deaths_per100k`. *Make sure to carefully choose **which rows** to
include in your summaries,* and justify why!

``` r
## TASK: Compute mean and sd for cases_per100k and deaths_per100k
cases_vals <- df_normalized %>%
  filter(!is.na(cases_per100k) & cases_per100k < 200) %>%
  summarize(
    mean_cases = mean(cases_per100k),
    sd_cases = sd(cases_per100k)        
            )

death_vals <- df_normalized %>%
  filter(!is.na(deaths_per100k) & cases_per100k < 200) %>%
  summarize(
    mean_deaths = mean(deaths_per100k),
    sd_deaths = sd(deaths_per100k)        
            )

cases_vals
```

    ## # A tibble: 1 × 2
    ##   mean_cases sd_cases
    ##        <dbl>    <dbl>
    ## 1       71.9     55.8

``` r
death_vals
```

    ## # A tibble: 1 × 2
    ##   mean_deaths sd_deaths
    ##         <dbl>     <dbl>
    ## 1        1.95      4.32

- Which rows did you pick?
  - when i did this at first, i just omitted the ones with na values, if
    there were any, but i was getting insanely high values for my mean
    and sd, so I decided to remove all values above 200
- Why?
  - I removed especially high values because if a really small county
    has even 1 case and then you multiply that by 100000, then it
    becomes a super outlier and totally screws up your mean and sd

### **q7** Find and compare the top 10

Find the top 10 counties in terms of `cases_per100k`, and the top 10 in
terms of `deaths_per100k`. Report the population of each county along
with the per-100,000 counts. Compare the counts against the mean values
you found in q6. Note any observations.

``` r
## TASK: Find the top 10 max cases_per100k counties; report populations as well

## TASK: Find the top 10 deaths_per100k counties; report populations as well

top_cases <- df_normalized %>%
  arrange(desc(cases_per100k)) %>%  
  head(10) %>%  
  select(county, population, cases_per100k)

top_deaths <- df_normalized %>%
  arrange(desc(deaths_per100k)) %>%  
  head(10) %>% 
  select(county, population, deaths_per100k)

top_cases
```

    ## # A tibble: 10 × 3
    ##    county population cases_per100k
    ##    <chr>       <dbl>         <dbl>
    ##  1 Loving        102       192157.
    ##  2 Loving        102       192157.
    ##  3 Loving        102       191176.
    ##  4 Loving        102       191176.
    ##  5 Loving        102       191176.
    ##  6 Loving        102       190196.
    ##  7 Loving        102       188235.
    ##  8 Loving        102       187255.
    ##  9 Loving        102       187255.
    ## 10 Loving        102       186275.

``` r
top_deaths
```

    ## # A tibble: 10 × 3
    ##    county   population deaths_per100k
    ##    <chr>         <dbl>          <dbl>
    ##  1 McMullen        662          1360.
    ##  2 McMullen        662          1360.
    ##  3 McMullen        662          1360.
    ##  4 McMullen        662          1360.
    ##  5 McMullen        662          1360.
    ##  6 McMullen        662          1360.
    ##  7 McMullen        662          1360.
    ##  8 McMullen        662          1360.
    ##  9 McMullen        662          1360.
    ## 10 McMullen        662          1360.

**Observations**:

- It’s all the same two counties reported 10 times???
- And they all have slightly different values as well.
- When did these “largest values” occur?
  - these largest values occurred when the population was very low, as I
    theorized above

## Self-directed EDA

<!-- ------------------------- -->

### **q8** Drive your own ship: You’ve just put together a very rich dataset; you now get to explore! Pick your own direction and generate at least one punchline figure to document an interesting finding. I give a couple tips & ideas below:

### Ideas

<!-- ------------------------- -->

- Look for outliers.
- Try web searching for news stories in some of the outlier counties.
- Investigate relationships between county population and counts.
- Do a deep-dive on counties that are important to you (e.g. where you
  or your family live).
- Fix the *geographic exceptions* noted below to study New York City.
- Your own idea!

**DO YOUR OWN ANALYSIS HERE**

``` r
library(scales)
```

    ## 
    ## Attaching package: 'scales'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     discard

    ## The following object is masked from 'package:readr':
    ## 
    ##     col_factor

``` r
counties_to_include = c("Lancaster", "Boone", "Kearney", "Seward", "Sioux")


df_normalized %>%
  filter(
    state == "Nebraska",
    !is.na(fips), 
    county %in% counties_to_include 
  ) %>%

  ggplot(
    aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
  ) +
  geom_line() +
  scale_y_log10(labels = label_number(accuracy = 1)) +
  scale_color_discrete(name = "County") +
  theme_minimal() +
  labs(
    x = "Date",
    y = "Cases (per 100,000 persons)"
  )
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

These are the counties in Nebraska that my family is from.

Observations:

- They all have basically the same values except for sioux county is a
  bit lower?

- There’s an interesting drop with Kearney and Sioux halfway into the
  data - no idea how to explain that.

  - If you look at all the counties, you see that a whole bunch of the
    counties also exhibit this behavior

- Lancaster has a curvier, less sporadic curve, which could be explained
  by the fact that it is much larger population than the others, so it
  is more regulated so to speak

- Lancaster county has a rise at the very end - if the data went
  further, would we see a second wave?

- 

### Aside: Some visualization tricks

<!-- ------------------------- -->

These data get a little busy, so it’s helpful to know a few `ggplot`
tricks to help with the visualization. Here’s an example focused on
Massachusetts.

`{# {r ma-example} # ## NOTE: No need to change this; just an example # df_normalized %>% #   filter( #     state == "Massachusetts", # Focus on Mass only #     !is.na(fips), # fct_reorder2 can choke with missing data #   ) %>% #  #   ggplot( #     aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k)) #   ) + #   geom_line() + #   scale_y_log10(labels = scales::label_number_si()) + #   scale_color_discrete(name = "County") + #   theme_minimal() + #   labs( #     x = "Date", #     y = "Cases (per 100,000 persons)" #   )`

*Tricks*:

- I use `fct_reorder2` to *re-order* the color labels such that the
  color in the legend on the right is ordered the same as the vertical
  order of rightmost points on the curves. This makes it easier to
  reference the legend.
- I manually set the `name` of the color scale in order to avoid
  reporting the `fct_reorder2` call.
- I use `scales::label_number_si` to make the vertical labels more
  readable.
- I use `theme_minimal()` to clean up the theme a bit.
- I use `labs()` to give manual labels.

### Geographic exceptions

<!-- ------------------------- -->

The NYT repo documents some [geographic
exceptions](https://github.com/nytimes/covid-19-data#geographic-exceptions);
the data for New York, Kings, Queens, Bronx and Richmond counties are
consolidated under “New York City” *without* a fips code. Thus the
normalized counts in `df_normalized` are `NA`. To fix this, you would
need to merge the population data from the New York City counties, and
manually normalize the data.

# Notes

<!-- -------------------------------------------------- -->

\[1\] The census used to have many, many questions, but the ACS was
created in 2010 to remove some questions and shorten the census. You can
learn more in [this wonderful visual
history](https://pudding.cool/2020/03/census-history/) of the census.

\[2\] FIPS stands for [Federal Information Processing
Standards](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards);
these are computer standards issued by NIST for things such as
government data.

\[3\] Demographers often report statistics not in percentages (per 100
people), but rather in per 100,000 persons. This is [not always the
case](https://stats.stackexchange.com/questions/12810/why-do-demographers-give-rates-per-100-000-people)
though!
