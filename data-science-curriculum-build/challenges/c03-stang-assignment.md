Aluminum Data
================
Cooper A. Penkava
2025-02-16

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Submission](#submission)
- [Loading and Wrangle](#loading-and-wrangle)
  - [**q1** Tidy `df_stang` to produce `df_stang_long`. You should have
    column names `thick, alloy, angle, E, nu`. Make sure the `angle`
    variable is of correct type. Filter out any invalid
    values.](#q1-tidy-df_stang-to-produce-df_stang_long-you-should-have-column-names-thick-alloy-angle-e-nu-make-sure-the-angle-variable-is-of-correct-type-filter-out-any-invalid-values)
- [EDA](#eda)
  - [Initial checks](#initial-checks)
    - [**q2** Perform a basic EDA on the aluminum data *without
      visualization*. Use your analysis to answer the questions under
      *observations* below. In addition, add your own *specific*
      question that you’d like to answer about the data—you’ll answer it
      below in
      q3.](#q2-perform-a-basic-eda-on-the-aluminum-data-without-visualization-use-your-analysis-to-answer-the-questions-under-observations-below-in-addition-add-your-own-specific-question-that-youd-like-to-answer-about-the-datayoull-answer-it-below-in-q3)
  - [Visualize](#visualize)
    - [**q3** Create a visualization to investigate your question from
      q2 above. Can you find an answer to your question using the
      dataset? Would you need additional information to answer your
      question?](#q3-create-a-visualization-to-investigate-your-question-from-q2-above-can-you-find-an-answer-to-your-question-using-the-dataset-would-you-need-additional-information-to-answer-your-question)
    - [**q4** Consider the following
      statement:](#q4-consider-the-following-statement)
- [References](#references)

*Purpose*: When designing structures such as bridges, boats, and planes,
the design team needs data about *material properties*. Often when we
engineers first learn about material properties through coursework, we
talk about abstract ideas and look up values in tables without ever
looking at the data that gave rise to published properties. In this
challenge you’ll study an aluminum alloy dataset: Studying these data
will give you a better sense of the challenges underlying published
material values.

In this challenge, you will load a real dataset, wrangle it into tidy
form, and perform EDA to learn more about the data.

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

*Background*: In 1946, scientists at the Bureau of Standards tested a
number of Aluminum plates to determine their
[elasticity](https://en.wikipedia.org/wiki/Elastic_modulus) and
[Poisson’s ratio](https://en.wikipedia.org/wiki/Poisson%27s_ratio).
These are key quantities used in the design of structural members, such
as aircraft skin under [buckling
loads](https://en.wikipedia.org/wiki/Buckling). These scientists tested
plats of various thicknesses, and at different angles with respect to
the [rolling](https://en.wikipedia.org/wiki/Rolling_(metalworking))
direction.

# Loading and Wrangle

<!-- -------------------------------------------------- -->

The `readr` package in the Tidyverse contains functions to load data
form many sources. The `read_csv()` function will help us load the data
for this challenge.

``` r
## NOTE: If you extracted all challenges to the same location,
## you shouldn't have to change this filename
filename <- "./data/stang.csv"

## Load the data
df_stang <- read_csv(filename)
```

    ## Rows: 9 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): alloy
    ## dbl (7): thick, E_00, nu_00, E_45, nu_45, E_90, nu_90
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
df_stang
```

    ## # A tibble: 9 × 8
    ##   thick  E_00 nu_00  E_45  nu_45  E_90 nu_90 alloy  
    ##   <dbl> <dbl> <dbl> <dbl>  <dbl> <dbl> <dbl> <chr>  
    ## 1 0.022 10600 0.321 10700  0.329 10500 0.31  al_24st
    ## 2 0.022 10600 0.323 10500  0.331 10700 0.323 al_24st
    ## 3 0.032 10400 0.329 10400  0.318 10300 0.322 al_24st
    ## 4 0.032 10300 0.319 10500  0.326 10400 0.33  al_24st
    ## 5 0.064 10500 0.323 10400  0.331 10400 0.327 al_24st
    ## 6 0.064 10700 0.328 10500  0.328 10500 0.32  al_24st
    ## 7 0.081 10000 0.315 10000  0.32   9900 0.314 al_24st
    ## 8 0.081 10100 0.312  9900  0.312 10000 0.316 al_24st
    ## 9 0.081 10000 0.311    -1 -1      9900 0.314 al_24st

Note that these data are not tidy! The data in this form are convenient
for reporting in a table, but are not ideal for analysis.

### **q1** Tidy `df_stang` to produce `df_stang_long`. You should have column names `thick, alloy, angle, E, nu`. Make sure the `angle` variable is of correct type. Filter out any invalid values.

*Hint*: You can reshape in one `pivot` using the `".value"` special
value for `names_to`.

``` r
## TASK: Tidy `df_stang`
df_stang_long <-
  df_stang %>% 
  pivot_longer(
    names_to = c(".value", "angle"),
    names_sep = "_",
    values_to = "val",
    starts_with("E") | starts_with("nu")
  ) %>% 
  mutate(angle = as.integer(angle)) %>%
  filter(E >= 0, nu >= 0)

df_stang_long
```

    ## # A tibble: 26 × 5
    ##    thick alloy   angle     E    nu
    ##    <dbl> <chr>   <int> <dbl> <dbl>
    ##  1 0.022 al_24st     0 10600 0.321
    ##  2 0.022 al_24st    45 10700 0.329
    ##  3 0.022 al_24st    90 10500 0.31 
    ##  4 0.022 al_24st     0 10600 0.323
    ##  5 0.022 al_24st    45 10500 0.331
    ##  6 0.022 al_24st    90 10700 0.323
    ##  7 0.032 al_24st     0 10400 0.329
    ##  8 0.032 al_24st    45 10400 0.318
    ##  9 0.032 al_24st    90 10300 0.322
    ## 10 0.032 al_24st     0 10300 0.319
    ## # ℹ 16 more rows

Use the following tests to check your work.

``` r
## NOTE: No need to change this
## Names
assertthat::assert_that(
              setequal(
                df_stang_long %>% names,
                c("thick", "alloy", "angle", "E", "nu")
              )
            )
```

    ## [1] TRUE

``` r
## Dimensions
assertthat::assert_that(all(dim(df_stang_long) == c(26, 5)))
```

    ## [1] TRUE

``` r
## Type
assertthat::assert_that(
              (df_stang_long %>% pull(angle) %>% typeof()) == "integer"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

# EDA

<!-- -------------------------------------------------- -->

## Initial checks

<!-- ------------------------- -->

### **q2** Perform a basic EDA on the aluminum data *without visualization*. Use your analysis to answer the questions under *observations* below. In addition, add your own *specific* question that you’d like to answer about the data—you’ll answer it below in q3.

``` r
##
glimpse(df_stang)
```

    ## Rows: 9
    ## Columns: 8
    ## $ thick <dbl> 0.022, 0.022, 0.032, 0.032, 0.064, 0.064, 0.081, 0.081, 0.081
    ## $ E_00  <dbl> 10600, 10600, 10400, 10300, 10500, 10700, 10000, 10100, 10000
    ## $ nu_00 <dbl> 0.321, 0.323, 0.329, 0.319, 0.323, 0.328, 0.315, 0.312, 0.311
    ## $ E_45  <dbl> 10700, 10500, 10400, 10500, 10400, 10500, 10000, 9900, -1
    ## $ nu_45 <dbl> 0.329, 0.331, 0.318, 0.326, 0.331, 0.328, 0.320, 0.312, -1.000
    ## $ E_90  <dbl> 10500, 10700, 10300, 10400, 10400, 10500, 9900, 10000, 9900
    ## $ nu_90 <dbl> 0.310, 0.323, 0.322, 0.330, 0.327, 0.320, 0.314, 0.316, 0.314
    ## $ alloy <chr> "al_24st", "al_24st", "al_24st", "al_24st", "al_24st", "al_24st"…

``` r
summary(df_stang)
```

    ##      thick              E_00           nu_00             E_45      
    ##  Min.   :0.02200   Min.   :10000   Min.   :0.3110   Min.   :   -1  
    ##  1st Qu.:0.03200   1st Qu.:10100   1st Qu.:0.3150   1st Qu.:10000  
    ##  Median :0.06400   Median :10400   Median :0.3210   Median :10400  
    ##  Mean   :0.05322   Mean   :10356   Mean   :0.3201   Mean   : 9211  
    ##  3rd Qu.:0.08100   3rd Qu.:10600   3rd Qu.:0.3230   3rd Qu.:10500  
    ##  Max.   :0.08100   Max.   :10700   Max.   :0.3290   Max.   :10700  
    ##      nu_45              E_90           nu_90           alloy          
    ##  Min.   :-1.0000   Min.   : 9900   Min.   :0.3100   Length:9          
    ##  1st Qu.: 0.3180   1st Qu.:10000   1st Qu.:0.3140   Class :character  
    ##  Median : 0.3260   Median :10400   Median :0.3200   Mode  :character  
    ##  Mean   : 0.1772   Mean   :10289   Mean   :0.3196                     
    ##  3rd Qu.: 0.3290   3rd Qu.:10500   3rd Qu.:0.3230                     
    ##  Max.   : 0.3310   Max.   :10700   Max.   :0.3300

``` r
df_stang_long %>%
  distinct(alloy)
```

    ## # A tibble: 1 × 1
    ##   alloy  
    ##   <chr>  
    ## 1 al_24st

``` r
df_stang_long %>%
  distinct(angle)
```

    ## # A tibble: 3 × 1
    ##   angle
    ##   <int>
    ## 1     0
    ## 2    45
    ## 3    90

``` r
df_stang_long %>%
  distinct(thick)
```

    ## # A tibble: 4 × 1
    ##   thick
    ##   <dbl>
    ## 1 0.022
    ## 2 0.032
    ## 3 0.064
    ## 4 0.081

**Observations**:

- Is there “one true value” for the material properties of Aluminum?
  - no - each different sample has a different value for each of the
    variables - there is variance, which can come from a multitude of
    sources all along the manufacturing process of these metal samples
- How many aluminum alloys are in this dataset? How do you know?
  - 1 - summary lists the alloy column as length 9, but each of those is
    the same type, as seen when using the glimpse function
- What angles were tested?
  - 0, 45, and 90, most likely in degrees
- What thicknesses were tested?
  - thicknesses including 0.022, 0.032, 0.064, and 0.081
- Does thickness impact the modulus of elasticity, E? Materials science
  says it shouldn’t, but what does the data say?
  - let’s make a visualization for this one.

## Visualize

<!-- ------------------------- -->

### **q3** Create a visualization to investigate your question from q2 above. Can you find an answer to your question using the dataset? Would you need additional information to answer your question?

``` r
## TASK: Investigate your question from q2 here
df_stang_long %>% 
  ggplot(aes(thick, E)) +
  geom_point()
```

![](c03-stang-assignment_files/figure-gfm/q3-task-1.png)<!-- -->

**Observations**:

- There is not a clear pattern here - I would say the data visualized in
  this way does not show a trend between sample thickness and modulus of
  elasticity, E. All the points are around 10,000, with a bit of
  deviation - we need to define what is a reasonable variance for this
  problem, but…
  - I also wonder why 0.081 thickness deviate a little more from the
    line than the other three groups - is this a coincidence, or is it
    something unrelated entirely? I think that 500 is a significant
    difference, but this would need more contextual thinking to confirm?
    Is this due to variance or different testing behaviors?

### **q4** Consider the following statement:

> “A material’s property (or material property) is an intensive property
> of some material, i.e. a physical property that does not depend on the
> amount of the material.”\[2\]

Note that the “amount of material” would vary with the thickness of a
tested plate. Does the following graph support or contradict the claim
that “elasticity `E` is an intensive material property.” Why or why not?
Is this evidence *conclusive* one way or another? Why or why not?

``` r
## NOTE: No need to change; run this chunk
df_stang_long %>%

  ggplot(aes(nu, E, color = as_factor(thick))) +
  geom_point(size = 3) +
  theme_minimal()
```

![](c03-stang-assignment_files/figure-gfm/q4-vis-1.png)<!-- -->

**Observations**:

first of all, bruh how did i come up with the same question :( and then
you went and made a better graph

- Does this graph support or contradict the claim above?
  - I would argue it tends to support it if we make the assumption that
    the outlier should be thrown out - the first three categories are
    all basically completely on top each other and the fourth is
    extremely close.
  - it would be nice to have a more zoomed in version of this graph
    after disregarding the outlier just to make sure there are no funny
    shapes
- Is this evidence *conclusive* one way or another?
  - absolutely not - there’s only a couple data points here, and one of
    them is a major outlier and the thickest category is ever so
    slightly off. I would not trust my bridges, skyscrapers, heavy
    machinery, and aerospace craft on this evidence.

**redoing q4:**

- this graph without the outlier now appears as a linear relationship
  between E and mu

- also, looking at the thickness, we can somewhat observe an inverse
  relationship between thickness and E/mu - the thinnest one has a lot
  of points up in the high corners, and the thickest has all in the
  lowest corner

- because of this, I would tend to say now that it actually contradicts
  the statement - there’s divided groupings of thicknesses with
  differing properties.

- I would shy away from saying that this data is conclusive - it’s
  pretty limited, and the data wasn’t even collected in the same way.
  further testing is definitely necessary, because each thickness only
  has 6 data points.

# References

<!-- -------------------------------------------------- -->

\[1\] Stang, Greenspan, and Newman, “Poisson’s ratio of some structural
alloys for large strains” (1946) Journal of Research of the National
Bureau of Standards, (pdf
link)\[<https://nvlpubs.nist.gov/nistpubs/jres/37/jresv37n4p211_A1b.pdf>\]

\[2\] Wikipedia, *List of material properties*, accessed 2020-06-26,
(link)\[<https://en.wikipedia.org/wiki/List_of_materials_properties>\]
