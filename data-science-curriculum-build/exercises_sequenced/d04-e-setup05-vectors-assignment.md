Setup: Vector Basics
================
Zachary del Rosario
2020-06-05

# Setup: Vector Basics

*Purpose*: *Vectors* are the most important object we’ll work with when
doing data science. To that end, let’s learn some basics about vectors.

*Reading*: [Programming
Basics](https://rstudio.cloud/learn/primers/1.2). *Topics*: `vectors`
*Reading Time*: ~10 minutes

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

### **q1** What single-letter `R` function do you use to create vectors with specific entries? Use that function to create a vector with the numbers `1, 2, 3` below.

``` r
## TODO: Assign the appropriate vector to vec_q1

vec_q1 <- c(1, 2, 3)
vec_q1
```

    ## [1] 1 2 3

Use the following tests to check your work:

``` r
## NOTE: No need to change this
assertthat::assert_that(length(vec_q1) == 3)
```

    ## [1] TRUE

``` r
assertthat::assert_that(mean(vec_q1) == 2)
```

    ## [1] TRUE

``` r
print("Nice!")
```

    ## [1] "Nice!"

### **q2** Did you know that you can use `c()` to *extend* a vector as well? Use this to add the extra entry `4` to `vec_q1`.

``` r
## TODO: Assign the appropriate vector to vec_q2

vec_q2 <- c(vec_q1,4)
vec_q2
```

    ## [1] 1 2 3 4

Use the following tests to check your work:

``` r
## NOTE: No need to change this
assertthat::assert_that(length(vec_q2) == 4)
```

    ## [1] TRUE

``` r
assertthat::assert_that(mean(vec_q2) == 2.5)
```

    ## [1] TRUE

``` r
print("Well done!")
```

    ## [1] "Well done!"

<!-- include-exit-ticket -->

# Exit Ticket

<!-- -------------------------------------------------- -->

Once you have completed this exercise, make sure to fill out the **exit
ticket survey**, [linked
here](https://docs.google.com/forms/d/e/1FAIpQLSeuq2LFIwWcm05e8-JU84A3irdEL7JkXhMq5Xtoalib36LFHw/viewform?usp=pp_url&entry.693978880=e-setup05-vectors-assignment.Rmd).
