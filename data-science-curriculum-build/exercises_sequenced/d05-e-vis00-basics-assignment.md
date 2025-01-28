Vis: Data Visualization Basics
================
Zach del Rosario
2020-05-03

# Vis: Data Visualization Basics

*Purpose*: The most powerful way for us to learn about a dataset is to
*visualize the data*. Throughout this class we will make extensive use
of the *grammar of graphics*, a powerful graphical programming *grammar*
that will allow us to create just about any graph you can imagine!

*Reading*: [Data Visualization
Basics](https://rstudio.cloud/learn/primers/1.1). *Note*: In RStudio use
`Ctrl + Click` (Mac `Command + Click`) to follow the link. *Topics*:
`Welcome`, `A code template`, `Aesthetic mappings`. *Reading Time*: ~ 30
minutes

### **q1** Inspect the `diamonds` dataset. What do the `cut`, `color`, and `clarity` variables mean?

*Hint*: We learned how to inspect a dataset in `e-data-00-basics`!

``` r
?diamonds
```

cut is the quality of cut (e.g. fair, good, very good, …)

color is the diamond color from best to worst (D-J)

clarity is a measurement of how clear the diamond is (I1-IF)

### **q2** Use your “standard checks” to determine what variables the dataset has.

Now that we have the list of variables in the dataset, we know what we
can visualize!

``` r
summary(diamonds)
```

    ##      carat               cut        color        clarity          depth      
    ##  Min.   :0.2000   Fair     : 1610   D: 6775   SI1    :13065   Min.   :43.00  
    ##  1st Qu.:0.4000   Good     : 4906   E: 9797   VS2    :12258   1st Qu.:61.00  
    ##  Median :0.7000   Very Good:12082   F: 9542   SI2    : 9194   Median :61.80  
    ##  Mean   :0.7979   Premium  :13791   G:11292   VS1    : 8171   Mean   :61.75  
    ##  3rd Qu.:1.0400   Ideal    :21551   H: 8304   VVS2   : 5066   3rd Qu.:62.50  
    ##  Max.   :5.0100                     I: 5422   VVS1   : 3655   Max.   :79.00  
    ##                                     J: 2808   (Other): 2531                  
    ##      table           price             x                y         
    ##  Min.   :43.00   Min.   :  326   Min.   : 0.000   Min.   : 0.000  
    ##  1st Qu.:56.00   1st Qu.:  950   1st Qu.: 4.710   1st Qu.: 4.720  
    ##  Median :57.00   Median : 2401   Median : 5.700   Median : 5.710  
    ##  Mean   :57.46   Mean   : 3933   Mean   : 5.731   Mean   : 5.735  
    ##  3rd Qu.:59.00   3rd Qu.: 5324   3rd Qu.: 6.540   3rd Qu.: 6.540  
    ##  Max.   :95.00   Max.   :18823   Max.   :10.740   Max.   :58.900  
    ##                                                                   
    ##        z         
    ##  Min.   : 0.000  
    ##  1st Qu.: 2.910  
    ##  Median : 3.530  
    ##  Mean   : 3.539  
    ##  3rd Qu.: 4.040  
    ##  Max.   :31.800  
    ## 

### **q3** Using `ggplot`, visualize `price` vs `carat` with points. What trend do

you observe?

*Hint*: Usually the language `y` vs `x` refers to the `vertical axis` vs
`horizontal axis`. This is the opposite order from the way we often
specify `x, y` pairs. Language is hard!

``` r
## TODO: Complete this code
diamonds %>% 
  ggplot(aes(carat, price)) +
  geom_point()
```

![](d05-e-vis00-basics-assignment_files/figure-gfm/q3-task-1.png)<!-- -->

**Observations**:

- having all black is hard to read, not fully linear, spread as you go
  right

## A note on *aesthetics*

The function `aes()` is short for *aesthetics*. Aesthetics in ggplot are
the mapping of variables in a dataframe to visual elements in the graph.
For instance, in the plot above you assigned `carat` to the `x`
aesthetic, and `price` to the `y` aesthetic. But there are *many more*
aesthetics you can set, some of which vary based on the `geom_` you are
using to visualize. The next question will explore this idea more.

### **q4** Create a new graph to visualize `price`, `carat`, and `cut`

simultaneously.

*Hint*: Remember that you can add additional aesthetic mappings in
`aes()`. Some options include `size`, `color`, and `shape`.

``` r
## TODO: Complete this code
diamonds %>% 
  ggplot(aes(carat, price)) +
  geom_point(mapping = aes(color = cut))
```

![](d05-e-vis00-basics-assignment_files/figure-gfm/q4-task-1.png)<!-- -->

**Observations**:

- it is somewhat helpful to have color

<!-- include-exit-ticket -->

# Exit Ticket

<!-- -------------------------------------------------- -->

Once you have completed this exercise, make sure to fill out the **exit
ticket survey**, [linked
here](https://docs.google.com/forms/d/e/1FAIpQLSeuq2LFIwWcm05e8-JU84A3irdEL7JkXhMq5Xtoalib36LFHw/viewform?usp=pp_url&entry.693978880=e-vis00-basics-assignment.Rmd).
