Setup: Types
================
Zachary del Rosario
2020-06-26

# Setup: Types

*Purpose*: Vectors can hold data of only one *type*. While this isn’t a
course on computer science, there are some type “gotchas” to look out
for when doing data science. This exercise will help us get ahead of
those issues.

*Reading*: [Types](https://rstudio.cloud/learn/primers/1.2)

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

## Objects vs Strings

<!-- -------------------------------------------------- -->

### **q1** Describe what is wrong with the code below.

``` r
## TASK: Describe what went wrong here
## Set our airport
airport <- "BOS"

## Check our airport value
airport == "ATL"
```

**Observations**:

- it was trying to compare a string “BOS” to an object ATL, which are
  incompatible for comparison

## Casting

<!-- -------------------------------------------------- -->

Sometimes our data will not be in the form we want; in this case we may
need to *cast* the data to another format.

- `as.integer(x)` converts to integer
- `as.numeric(x)` converts to real (floating point)
- `as.character(x)` converts to character (string)
- `as.logical(x)` converts to logical (boolean)

### **q2** Cast the following vector `v_string` to integers.

``` r
v_string <- c("00", "45", "90")
v_integer <- as.integer(v_string)
v_integer
```

    ## [1]  0 45 90

Use the following test to check your work.

``` r
## NOTE: No need to change this!
assertthat::assert_that(
  assertthat::are_equal(
                v_integer,
                c(0L, 45L, 90L)
  )
)
```

    ## [1] TRUE

``` r
print("Great job!")
```

    ## [1] "Great job!"

<!-- include-exit-ticket -->

# Exit Ticket

<!-- -------------------------------------------------- -->

Once you have completed this exercise, make sure to fill out the **exit
ticket survey**, [linked
here](https://docs.google.com/forms/d/e/1FAIpQLSeuq2LFIwWcm05e8-JU84A3irdEL7JkXhMq5Xtoalib36LFHw/viewform?usp=pp_url&entry.693978880=e-setup06-types-assignment.Rmd).
