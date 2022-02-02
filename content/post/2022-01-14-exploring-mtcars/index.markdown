---
title: Exploring mtcars
author: ''
date: '2022-01-14'
slug: []
categories: []
tags: []
---




```r
library(ggplot2)
library(GGally)
library(dplyr)
```

# The `mtcars` dataset

```r
str(mtcars)
```

```
## 'data.frame':	32 obs. of  11 variables:
##  $ mpg : num  21 21 22.8 21.4 18.7 18.1 14.3 24.4 22.8 19.2 ...
##  $ cyl : num  6 6 4 6 8 6 8 4 4 6 ...
##  $ disp: num  160 160 108 258 360 ...
##  $ hp  : num  110 110 93 110 175 105 245 62 95 123 ...
##  $ drat: num  3.9 3.9 3.85 3.08 3.15 2.76 3.21 3.69 3.92 3.92 ...
##  $ wt  : num  2.62 2.88 2.32 3.21 3.44 ...
##  $ qsec: num  16.5 17 18.6 19.4 17 ...
##  $ vs  : num  0 0 1 1 0 1 0 1 1 1 ...
##  $ am  : num  1 1 1 0 0 0 0 0 0 0 ...
##  $ gear: num  4 4 4 3 3 3 3 4 4 4 ...
##  $ carb: num  4 4 1 1 2 1 4 2 2 4 ...
```

```r
# Pais
ggpairs(mtcars,columns = c(1,3:7),mapping = aes(color=as.factor(cyl))) 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/mtcars-1.png" width="672" />

```r
save.image()

# Miles per galon (mpg)
ggplot(mtcars, aes(x=mpg)) +
  geom_histogram(bins = 7) +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/mtcars-2.png" width="672" />


# The perfect fit


```r
nc <- ncol(mtcars)
nr <- nc
perfect_fit <- lm(mpg~.,data=mtcars[1:nr,])

predictions <- mtcars[1:nr,] %>%
  mutate(pred = predict(perfect_fit))

ggplot(predictions, aes(x=pred,y=mpg)) +
  geom_point(color="green",size=1.5) +
  geom_abline(slope=1,color="green") +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />

# Unseen data

```r
nr <- nr +1
nr_full <- nrow(mtcars)
# Using the perfect_fit model on the unseen data [nr:nr_full,]
full_pred <- mtcars[nr:nr_full,] %>%
  mutate(pred = predict(perfect_fit,newdata = mtcars[nr:nr_full,]))

ggplot(full_pred, aes(x=pred,y=mpg)) +
  geom_point(color="green",size=1.5) +
  geom_smooth(method="lm",color="green",se=FALSE) +
  theme_bw()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

