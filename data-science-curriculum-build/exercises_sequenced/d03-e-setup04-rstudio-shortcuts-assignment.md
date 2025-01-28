Setup: RStudio Shortcuts
================
Zachary del Rosario
2020-05-07

# Setup: RStudio Shortcuts

*Purpose*: Your ability to get stuff done is highly dependent on your
fluency with your tools. One aspect of fluency is knowing and *using*
keyboard shortcuts. In this exercise, we’ll go over some of the most
important ones.

*Reading*: [Keyboard
shortcuts](https://support.rstudio.com/hc/en-us/articles/200711853-Keyboard-Shortcuts);
[Code Chunk Options](https://rmarkdown.rstudio.com/lesson-3.html)
*Note*: Use this reading to look up answers to the questions below.
Rather than *memorizing* this information, I recommend you download a
[cheatsheet](https://rstudio.com/wp-content/uploads/2016/01/rstudio-IDE-cheatsheet.pdf),
and either print it out or save it in a convenient place on your
computer. Get used to referencing your cheatsheets while doing data
science—practice makes perfect!

### **q1** What do the following keyboard shortcuts do?

- Within the script editor or a chunk

  - `Alt` + `-`

    - insert “\<-”

  - `Shift` + `Cmd/Ctrl` + `M`

    - insert “%\>%”

  - `Cmd/Ctrl` + `Enter`

    - run current line/section

  - `F1` (Note: on a Mac you need to press `fn` + `F1`)

    - show help for current function

  - `Cmd/Ctrl` + `Shift` + `C`

    - comment/uncomment lines

- Within R Markdown

  - `Cmd/Ctrl` + `Alt` + `I`
    - insert code chunk

- Within a chunk

  - `Shift` + `Cmd/Ctrl` + `Enter`

    - source with echo

  - `Ctrl` + `I` (`Cmd` + `I`)

    - reindent lines

Try this below!

``` r
## Re-indent these lines
c(
  "foo",
  "bar",
  "goo",
  "gah"
)
```

    ## [1] "foo" "bar" "goo" "gah"

### **q2** For a chunk, what header option do you use to

- Run the code, don’t display it, but show its results?

  - echo = FALSE

- Run the code, but don’t display it or its results?

  - include = FALSE

### **q3** How do stop the code in a chunk from running once it has started?

ESCAPE

### **q4** How do you show the “Document Outline” in RStudio?

*Hint*: Try googling “rstudio document outline”

outline button in top right corner of this window.

<!-- include-exit-ticket -->

# Exit Ticket

<!-- -------------------------------------------------- -->

Once you have completed this exercise, make sure to fill out the **exit
ticket survey**, [linked
here](https://docs.google.com/forms/d/e/1FAIpQLSeuq2LFIwWcm05e8-JU84A3irdEL7JkXhMq5Xtoalib36LFHw/viewform?usp=pp_url&entry.693978880=e-setup04-rstudio-shortcuts-assignment.Rmd).
