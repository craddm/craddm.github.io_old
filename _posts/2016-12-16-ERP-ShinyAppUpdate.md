---
title: 'ERP visualization: Shiny Demo updated'
output: html_document
layout: post
comments: true
categories: [EEG, ERPs, statistics, R, ggplot2, shiny]
---



## Shiny app updated!

In my last [post](https://craddm.github.io/blog/2016/12/07/ERP-ShinyApp) unleashed the [Shiny](http://www.shinyapps.io/) app I'd knocked up in a few hours to do some basic display of different confidence interval types and difference waves. I've been hacking away at it on and off and I've now added some exciting new features!

You can now try loading up your own data. You'll need a .csv file with the following structure:

* No header
* Comma-separated values
* Each row should be one time-point, one subject, columns should be "condition1",  "condition2", "Time", "Subject"

Here's the first few lines of the example data I include (note this is already after import, so it's stripped the commas between values).


```
##        V1       V2      V3 V4
## 1 0.27208 -0.83944 -199.22  1
## 2 0.29759 -0.84020 -197.27  1
## 3 0.33535 -0.83444 -195.31  1
## 4 0.37724 -0.82867 -193.36  1
## 5 0.38822 -0.82690 -191.41  1
## 6 0.36169 -0.82391 -189.45  1
```

I'll allow more flexibility in the file format at some point.

You can now also add a line showing whether each time-point is significant either:

* Uncorrected
* Bonferroni-Holm corrected
* False Discovery Rate corrected

As usual the code is available over on my [Github page](https://github.com/craddm/ERPdemo). If you want to run the app locally rather than over on the Shinyapps website, download it from there. Obviously, you'll need R. The packages listed in the code below are also required to be able to run the app. In R set your working directory to the folder containing *app.R*, and simply type runApp() after loading the Shiny library.


```r
install.packages("shiny","tidyverse","magrittr","Rmisc","Cairo","shinythemes")
```


