---
title: "R Notebook"
output: html_notebook
---


```r
library(tidyverse)
```

```
## Loading tidyverse: ggplot2
## Loading tidyverse: tibble
## Loading tidyverse: tidyr
## Loading tidyverse: readr
## Loading tidyverse: purrr
## Loading tidyverse: dplyr
```

```
## Conflicts with tidy packages ----------------------------------------------
```

```
## filter(): dplyr, stats
## lag():    dplyr, stats
```

```r
library(ggplot2)
library(copula)
```



```r
meanSummary <- dataMultiSub %>% gather(condition,power,-Subject) %>% group_by(Subject,condition) %>% summarise(power = mean(power)) 
```

```
## Error in eval(expr, envir, enclos): object 'dataMultiSub' not found
```

```r
meanBaselineP <- dataMultiSub %>% gather(condition,power,-Subject) %>% group_by(Subject) %>% summarise(mean = mean(power))
```

```
## Error in eval(expr, envir, enclos): object 'dataMultiSub' not found
```

```r
meanBLP2 <- rep(meanBaselineP$mean,each =2)
```

```
## Error in eval(expr, envir, enclos): object 'meanBaselineP' not found
```

```r
meanBaseline <- dataMultiSub %>% gather(condition,power,-Subject) %>% group_by(Subject) %>% mutate(power = power-mean(power)) %>% group_by(Subject,condition) %>% summarise(power = mean(power)) %>% ungroup() %>% mutate(blPower = meanSummary$power-meanBLP2)
```

```
## Error in eval(expr, envir, enclos): object 'dataMultiSub' not found
```

```r
meanBaselineP2 <- dataMultiSub %>% gather(condition,power,-Subject) %>% group_by(Subject) %>% group_by(Subject,condition) %>% summarise(power = mean(power)) %>%ungroup()
```

```
## Error in eval(expr, envir, enclos): object 'dataMultiSub' not found
```

```r
medianBaseline <- dataMultiSub %>% gather(condition,power,-Subject) %>% group_by(Subject) %>% mutate(power = power-median(power)) %>% group_by(Subject,condition) %>% summarise(power = mean(power)) %>%ungroup()
```

```
## Error in eval(expr, envir, enclos): object 'dataMultiSub' not found
```

```r
meanSummary %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

```
## Error in eval(expr, envir, enclos): object 'meanSummary' not found
```

```r
meanBaseline %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

```
## Error in eval(expr, envir, enclos): object 'meanBaseline' not found
```

```r
medianBaseline %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

```
## Error in eval(expr, envir, enclos): object 'medianBaseline' not found
```

```r
meanSummary %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x =difference))+geom_density(alpha =.2)
```

```
## Error in eval(expr, envir, enclos): object 'meanSummary' not found
```

```r
meanBaseline %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

```
## Error in eval(expr, envir, enclos): object 'meanBaseline' not found
```

```r
medianBaseline %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

```
## Error in eval(expr, envir, enclos): object 'medianBaseline' not found
```

```r
pairwise.t.test(meanSummary$power,meanSummary$condition,paired = TRUE)
```

```
## Error in factor(g): object 'meanSummary' not found
```

```r
pairwise.t.test(meanBaseline$power,meanBaseline$condition,paired = TRUE)
```

```
## Error in factor(g): object 'meanBaseline' not found
```

```r
pairwise.t.test(medianBaseline$power,medianBaseline$condition,paired = TRUE)
```

```
## Error in factor(g): object 'medianBaseline' not found
```



```r
nSubs <- 20
nTrials <- 100

myCop<- normalCopula(param=0.5, dim = 2, dispstr = "ex")
myMvd <- mvdc(copula=myCop, margins=c("lnorm", "lnorm"),
              paramMargins=list(list(meanlog=0.05, sdlog=1),
                                list(meanlog=0.02, sdlog=1)
                                ))

MVNdata <- replicate(nSubs,rMvdc(nTrials,myMvd),simplify = FALSE)
MVNdata <- bind_rows(lapply(MVNdata, data.frame, stringsAsFactors = FALSE)) %>%
  mutate(Subject = rep(1:nSubs,each = nTrials)) 
  names(MVNdata) <- c("condA","condB","Subject")

pairs.panels(MVNdata[1:2],method = "spearman")
```

```
## Error in eval(expr, envir, enclos): could not find function "pairs.panels"
```

```r
MVNdata %>% gather(condition,power,-Subject) %>%
ggplot(aes(x = power,fill = condition))+geom_histogram(position = "dodge")+facet_wrap(~Subject)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk unnamed-chunk-3](/figure/./EEG-baselineCorrection/unnamed-chunk-3-1.png)


```r
meanSummary <- MVNdata %>% gather(condition,power,-Subject) %>% group_by(Subject,condition) %>% summarise(power = mean(power)) 

actualDifference <- MVNdata %>% mutate(difference = log(condA)-log(condB),diffUnt = condA - condB) %>% group_by(Subject) %>% summarise(logDiff = mean(difference),realDiff = mean(diffUnt)) 
mean(actualDifference$logDiff)
```

```
## [1] 0.01680475
```

```r
mean(actualDifference$realDiff)
```

```
## [1] 0.01582332
```

```r
meanBLP <- MVNdata  %>% 
  gather(condition,power,-Subject) %>%
  group_by(Subject) %>%
  summarise(mean = mean(power)) 

meanBLP2 <- rep(meanBLP$mean,each =2)

meanBaseline <- MVNdata %>% 
  gather(condition,power,-Subject) %>% 
  group_by(Subject) %>% 
  mutate(power = power-mean(power)) %>%
  group_by(Subject,condition) %>%
  summarise(power = mean(power)) %>%
  ungroup() %>%
  mutate(meanBL = meanSummary$power-meanBLP2)

meanSummary %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-5](/figure/./EEG-baselineCorrection/unnamed-chunk-5-1.png)

```r
meanBaseline %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-5](/figure/./EEG-baselineCorrection/unnamed-chunk-5-2.png)

```r
meanBaseline %>% ggplot(aes(x = meanBL,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-5](/figure/./EEG-baselineCorrection/unnamed-chunk-5-3.png)

```r
meanBaseline[1:3] %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-5](/figure/./EEG-baselineCorrection/unnamed-chunk-5-4.png)

```r
pairwise.t.test(meanSummary$power,meanSummary$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanSummary$power and meanSummary$condition 
## 
##       condA
## condB 0.76 
## 
## P value adjustment method: holm
```

```r
pairwise.t.test(meanBaseline$power,meanBaseline$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanBaseline$power and meanBaseline$condition 
## 
##       condA
## condB 0.76 
## 
## P value adjustment method: holm
```

```r
bb <- meanBaseline[1:3] %>% spread(condition,power)
t.test(bb$condA,bb$condB,paired = TRUE)
```

```
## 
## 	Paired t-test
## 
## data:  bb$condA and bb$condB
## t = 0.30727, df = 19, p-value = 0.762
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.09196004  0.12360668
## sample estimates:
## mean of the differences 
##              0.01582332
```


```r
ZScoreBaseline <- MVNdata %>% 
  gather(condition,power,-Subject) %>% 
  group_by(Subject) %>% 
  mutate(power = (power-mean(power))/sd(power)) %>%
  group_by(Subject,condition) %>%
  summarise(power = mean(power)) %>%
  ungroup()

medianBaseline <- MVNdata %>% gather(condition,power,-Subject) %>% group_by(Subject) %>% mutate(power = power-median(power)) %>% group_by(Subject,condition) %>% summarise(power = mean(power)) %>%ungroup()

meanSummary %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-1.png)

```r
meanBaseline %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-2.png)

```r
ZScoreBaseline %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-3.png)

```r
medianBaseline %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-4.png)

```r
meanSummary %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x =difference))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-5.png)

```r
meanBaseline %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

```
## Warning: Removed 40 rows containing non-finite values (stat_density).
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-6.png)

```r
ZScoreBaseline %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-7.png)

```r
medianBaseline %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-6](/figure/./EEG-baselineCorrection/unnamed-chunk-6-8.png)

```r
pairwise.t.test(meanSummary$power,meanSummary$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanSummary$power and meanSummary$condition 
## 
##       condA
## condB 0.76 
## 
## P value adjustment method: holm
```

```r
pairwise.t.test(ZScoreBaseline$power,meanBaseline$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  ZScoreBaseline$power and meanBaseline$condition 
## 
##       condA
## condB 0.6  
## 
## P value adjustment method: holm
```

```r
pairwise.t.test(medianBaseline$power,medianBaseline$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  medianBaseline$power and medianBaseline$condition 
## 
##       condA
## condB 0.76 
## 
## P value adjustment method: holm
```


```r
meanDivBaseline <- MVNdata %>%
  gather(condition,power,-Subject) %>%
  group_by(Subject) %>%
  mutate(power = power/mean(power)) %>%
  group_by(Subject,condition) %>%
  summarise(power = mean(power)) %>%
  ungroup() 

meanSummary %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-7](/figure/./EEG-baselineCorrection/unnamed-chunk-7-1.png)

```r
meanDivBaseline %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-7](/figure/./EEG-baselineCorrection/unnamed-chunk-7-2.png)

```r
meanDivBaseline[1:3] %>% spread(condition, power) %>% mutate(difference = condA -
  condB) %>% ggplot(aes(x = difference)) + geom_density()
```

![plot of chunk unnamed-chunk-7](/figure/./EEG-baselineCorrection/unnamed-chunk-7-3.png)

```r
pairwise.t.test(meanDivBaseline$power,meanDivBaseline$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanDivBaseline$power and meanDivBaseline$condition 
## 
##       condA
## condB 0.7  
## 
## P value adjustment method: holm
```


```r
meanDbBaseline <- MVNdata %>%
  gather(condition, power, -Subject) %>%
  group_by(Subject) %>%
  mutate(power = log(power) - log(mean(power))) %>%
  group_by(Subject, condition) %>%
  summarise(power = mean(power)) %>%
  ungroup() %>%
  mutate(DbBL = log(meanSummary$power/meanBLP2))
  
meanDbBaseline %>%
    ggplot(aes(x = power, fill = condition)) + geom_density(alpha =  .2)
```

![plot of chunk unnamed-chunk-8](/figure/./EEG-baselineCorrection/unnamed-chunk-8-1.png)

```r
meanDbBaseline %>%
    ggplot(aes(x = DbBL, fill = condition)) + geom_density(alpha =  .2)
```

![plot of chunk unnamed-chunk-8](/figure/./EEG-baselineCorrection/unnamed-chunk-8-2.png)

```r
meanDbBaseline[1:3] %>% spread(condition, power) %>% mutate(difference = condA -
  condB) %>% ggplot(aes(x = difference)) + geom_density()
```

![plot of chunk unnamed-chunk-8](/figure/./EEG-baselineCorrection/unnamed-chunk-8-3.png)

```r
meanDbBaseline[c(1:2,4)] %>% spread(condition, DbBL) %>% mutate(difference = condA -
  condB) %>% ggplot(aes(x = difference)) + geom_density()
```

![plot of chunk unnamed-chunk-8](/figure/./EEG-baselineCorrection/unnamed-chunk-8-4.png)

```r
pairwise.t.test(meanDbBaseline$power,meanDbBaseline$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanDbBaseline$power and meanDbBaseline$condition 
## 
##       condA
## condB 0.39 
## 
## P value adjustment method: holm
```

```r
pairwise.t.test(meanDbBaseline$DbBL,meanDbBaseline$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanDbBaseline$DbBL and meanDbBaseline$condition 
## 
##       condA
## condB 0.71 
## 
## P value adjustment method: holm
```


```r
  medianDivBaseline <- MVNdata %>%
    gather(condition, power, -Subject) %>%
    group_by(Subject) %>%
    mutate(power = power /median(power)) %>%
    group_by(Subject, condition) %>% summarise(power = mean(power)) %>%
    ungroup()
  
  meanSummary %>%
    ggplot(aes(x = power, fill = condition)) + geom_density(alpha = .2)
```

![plot of chunk unnamed-chunk-9](/figure/./EEG-baselineCorrection/unnamed-chunk-9-1.png)

```r
  meanDivBaseline %>% ggplot(aes(x = power, fill = condition)) + geom_density(alpha = .2)
```

![plot of chunk unnamed-chunk-9](/figure/./EEG-baselineCorrection/unnamed-chunk-9-2.png)

```r
  meanDbBaseline %>%
    ggplot(aes(x = power, fill = condition)) + geom_density(alpha =  .2)
```

![plot of chunk unnamed-chunk-9](/figure/./EEG-baselineCorrection/unnamed-chunk-9-3.png)

```r
  medianDivBaseline %>%
    ggplot(aes(x = power, fill = condition)) + geom_density(alpha = .2)
```

![plot of chunk unnamed-chunk-9](/figure/./EEG-baselineCorrection/unnamed-chunk-9-4.png)

```r
  meanSummary %>% spread(condition, power) %>% mutate(difference = condA -
  condB) %>% ggplot(aes(x = difference)) + geom_density(alpha = .2)
```

![plot of chunk unnamed-chunk-9](/figure/./EEG-baselineCorrection/unnamed-chunk-9-5.png)

```r
  meanDbBaseline %>% spread(condition, power) %>% mutate(difference = condA -
  condB) %>% ggplot(aes(x = difference)) + geom_density(alpha = .2)
```

```
## Warning: Removed 40 rows containing non-finite values (stat_density).
```

![plot of chunk unnamed-chunk-9](/figure/./EEG-baselineCorrection/unnamed-chunk-9-6.png)

```r
  medianDivBaseline %>% spread(condition, power) %>% mutate(difference = condA -
  condB) %>% ggplot(aes(x = difference)) + geom_density(alpha = .2)
```

![plot of chunk unnamed-chunk-9](/figure/./EEG-baselineCorrection/unnamed-chunk-9-7.png)

```r
  pairwise.t.test(meanSummary$power, meanSummary$condition, paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanSummary$power and meanSummary$condition 
## 
##       condA
## condB 0.76 
## 
## P value adjustment method: holm
```

```r
  pairwise.t.test(meanDivBaseline$power, meanDivBaseline$condition, paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanDivBaseline$power and meanDivBaseline$condition 
## 
##       condA
## condB 0.7  
## 
## P value adjustment method: holm
```

```r
  pairwise.t.test(meanDbBaseline$power, meanDbBaseline$condition, paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanDbBaseline$power and meanDbBaseline$condition 
## 
##       condA
## condB 0.39 
## 
## P value adjustment method: holm
```

```r
  pairwise.t.test(medianDivBaseline$power,
  medianDivBaseline$condition,
  paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  medianDivBaseline$power and medianDivBaseline$condition 
## 
##       condA
## condB 0.72 
## 
## P value adjustment method: holm
```



```r
meanLogSummary <- MVNdata %>% gather(condition,power,-Subject) %>% group_by(Subject,condition) %>% summarise(power = mean(log10(power))) 

meanBLPlog <- MVNdata  %>% 
  gather(condition,power,-Subject) %>%
  group_by(Subject) %>%
  summarise(mean = mean(log10(power)))

meanBLP2log <- rep(meanBLPlog$mean,each =2)

meanBaselineLog <- MVNdata %>% 
  gather(condition,power,-Subject) %>% 
  group_by(Subject) %>% 
  mutate(power = log10(power)-mean(log10(power))) %>%
  group_by(Subject,condition) %>%
  summarise(power = mean(power)) %>%
  ungroup() %>%
  mutate(meanBL = meanLogSummary$power-meanBLP2log)


meanBaselineLogDiv <- MVNdata %>% 
  gather(condition,power,-Subject) %>% 
  group_by(Subject) %>% 
  mutate(power = log(power)/mean(log(power))) %>%
  group_by(Subject,condition) %>%
  summarise(power = mean(power)) %>%
  ungroup() %>%
  mutate(meanBL = meanLogSummary$power/meanBLP2log)

meanLogSummary %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-10](/figure/./EEG-baselineCorrection/unnamed-chunk-10-1.png)

```r
meanBaselineLog %>% ggplot(aes(x = power,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-10](/figure/./EEG-baselineCorrection/unnamed-chunk-10-2.png)

```r
meanBaselineLog %>% ggplot(aes(x = meanBL,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-10](/figure/./EEG-baselineCorrection/unnamed-chunk-10-3.png)

```r
meanBaselineLogDiv %>% ggplot(aes(x = meanBL,fill = condition))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-10](/figure/./EEG-baselineCorrection/unnamed-chunk-10-4.png)

```r
meanBaselineLog[1:3] %>% spread(condition,power) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-10](/figure/./EEG-baselineCorrection/unnamed-chunk-10-5.png)

```r
meanBaselineLog[c(1,2,4)] %>% spread(condition,meanBL) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-10](/figure/./EEG-baselineCorrection/unnamed-chunk-10-6.png)

```r
meanBaselineLogDiv[c(1,2,4)] %>% spread(condition,meanBL) %>% mutate(difference = condA-condB) %>% ggplot(aes(x = difference))+geom_density(alpha =.2)
```

![plot of chunk unnamed-chunk-10](/figure/./EEG-baselineCorrection/unnamed-chunk-10-7.png)

```r
pairwise.t.test(meanLogSummary$power,meanLogSummary$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanLogSummary$power and meanLogSummary$condition 
## 
##       condA
## condB 0.39 
## 
## P value adjustment method: holm
```

```r
pairwise.t.test(meanBaselineLog$power,meanBaselineLog$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanBaselineLog$power and meanBaselineLog$condition 
## 
##       condA
## condB 0.39 
## 
## P value adjustment method: holm
```

```r
pairwise.t.test(meanBaselineLog$meanBL,meanBaselineLog$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanBaselineLog$meanBL and meanBaselineLog$condition 
## 
##       condA
## condB 0.39 
## 
## P value adjustment method: holm
```

```r
pairwise.t.test(meanBaselineLogDiv$power,meanBaselineLog$condition,paired = TRUE)
```

```
## 
## 	Pairwise comparisons using paired t tests 
## 
## data:  meanBaselineLogDiv$power and meanBaselineLog$condition 
## 
##       condA
## condB 0.74 
## 
## P value adjustment method: holm
```

```r
bb <- meanBaselineLog[1:3] %>% spread(condition,power)
t.test(bb$condA,bb$condB,paired = TRUE)
```

```
## 
## 	Paired t-test
## 
## data:  bb$condA and bb$condB
## t = 0.8716, df = 19, p-value = 0.3943
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.01022732  0.02482373
## sample estimates:
## mean of the differences 
##             0.007298208
```

```r
bb <- meanBaselineLog[c(1,2,4)] %>% spread(condition,meanBL)
t.test(bb$condA,bb$condB,paired = TRUE)
```

```
## 
## 	Paired t-test
## 
## data:  bb$condA and bb$condB
## t = 0.8716, df = 19, p-value = 0.3943
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.01022732  0.02482373
## sample estimates:
## mean of the differences 
##             0.007298208
```


