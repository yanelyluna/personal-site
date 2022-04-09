---
title: Popular songs (Taylor's Version)
author: Yanely Luna
date: '2022-02-24'
slug: []
categories: []
tags: []
subtitle: ''
excerpt: 'Exploring songs by my favorite artist in the world.'
images: ~
series: ~
layout: single
---

If you have known me in person, I think it would take you less than ten minutes to realize that I love Taylor Swift's music (and work in general) more that any music from any other artist ever. In that case, is no surprise that I dedicated this post to analyse her discography data. I will be using a dataset from [kaggle](https://www.kaggle.com/thespacefreak/taylor-swift-spotify-data) to first do some exploration and then to try to determine the song's popularity based on its properties.



# Libraries


```r
library(dplyr) # For manipulating data
library(tidyr) # same
library(ggplot2) # For graphs
library(gridExtra) # same
library(ggtextures) # For textured graphs
```

# Dataset


```r
taylor <- read.csv("spotify_taylorswift.csv")
str(taylor)
```

```
## 'data.frame':	171 obs. of  16 variables:
##  $ X               : int  0 1 2 3 4 5 6 7 8 9 ...
##  $ name            : chr  "Tim McGraw" "Picture To Burn" "Teardrops On My Guitar - Radio Single Remix" "A Place in this World" ...
##  $ album           : chr  "Taylor Swift" "Taylor Swift" "Taylor Swift" "Taylor Swift" ...
##  $ artist          : chr  "Taylor Swift" "Taylor Swift" "Taylor Swift" "Taylor Swift" ...
##  $ release_date    : chr  "2006-10-24" "2006-10-24" "2006-10-24" "2006-10-24" ...
##  $ length          : int  232106 173066 203040 199200 239013 207106 248106 236053 242200 213080 ...
##  $ popularity      : int  49 54 59 49 50 47 47 48 53 50 ...
##  $ danceability    : num  0.58 0.658 0.621 0.576 0.418 0.589 0.479 0.594 0.476 0.403 ...
##  $ acousticness    : num  0.575 0.173 0.288 0.051 0.217 0.00491 0.525 0.0868 0.0103 0.0177 ...
##  $ energy          : num  0.491 0.877 0.417 0.777 0.482 0.805 0.578 0.629 0.777 0.627 ...
##  $ instrumentalness: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ liveness        : num  0.121 0.0962 0.119 0.32 0.123 0.24 0.0841 0.137 0.196 0.182 ...
##  $ loudness        : num  -6.46 -2.1 -6.94 -2.88 -5.77 ...
##  $ speechiness     : num  0.0251 0.0323 0.0231 0.0324 0.0266 0.0293 0.0294 0.0246 0.0289 0.0292 ...
##  $ valence         : num  0.425 0.821 0.289 0.428 0.261 0.591 0.192 0.504 0.472 0.374 ...
##  $ tempo           : num  76 106 100 115 176 ...
```

This dataset contains 171 observations from 16 variables, which according to the source represent the following:

+ `name`: The name of the song
+ `album`: Name of the album
+ `artist`: Name of artist/s involved
+ `release_date`: Release date of album
+ `length`: Song length in milliseconds
+ `popularity`: Percent popularity of the song based on Spotify's algorithm (posibly the number of stream at a certain period of time).
+ `danceability`: How suitable a track is for dancing based on a combination of musical elements.
+ `acousticness`: How acoustic a song is.
+ `energy`: A perceptual measure of intensity and activity. 
+ `instrumentalness`: The amount of vocals in the song.
+ `liveness`: Probability that the song was recorded with a live audience.
+ `loudness`: Tendency of music to be recorded at steadily higher volumes.
+ `speechiness`: Presence of spoken words in a track (if the speechiness of a song is above 0.66, it is probably made of spoken words, a score between 0.33 and 0.66 is a song that may contain both music and words, and a score below 0.33 means the song does not have any speech)
+ `valence`: A measure of how sad or happy the song sounds.
+ `tempo`: Beats per minute.

We can drop the variable `X` since it does not give us information and look for other variables that we may not need.

When we take a closer look to the _folklore_ album (or any other album) we notice that the `artist` column has only one value even when other artist participate on the song (e.g. _exile_). We check that is the case for the whole dataset, so we can drop that variable later.


```r
taylor %>% count(album)
```

```
##                         album  n
## 1               1989 (Deluxe) 19
## 2   evermore (deluxe version) 17
## 3 Fearless (Taylor's Version) 26
## 4   folklore (deluxe version) 17
## 5                       Lover 18
## 6        Red (Deluxe Edition) 22
## 7                  reputation 15
## 8  Speak Now (Deluxe Package) 22
## 9                Taylor Swift 15
```

```r
taylor %>% filter(album=="folklore (deluxe version)") %>% select(name, artist)
```

```
##                               name       artist
## 1                            the 1 Taylor Swift
## 2                         cardigan Taylor Swift
## 3  the last great american dynasty Taylor Swift
## 4           exile (feat. Bon Iver) Taylor Swift
## 5                my tears ricochet Taylor Swift
## 6                       mirrorball Taylor Swift
## 7                            seven Taylor Swift
## 8                           august Taylor Swift
## 9                this is me trying Taylor Swift
## 10                 illicit affairs Taylor Swift
## 11                invisible string Taylor Swift
## 12                       mad woman Taylor Swift
## 13                        epiphany Taylor Swift
## 14                           betty Taylor Swift
## 15                           peace Taylor Swift
## 16                            hoax Taylor Swift
## 17         the lakes - bonus track Taylor Swift
```

```r
taylor %>% count(artist)
```

```
##         artist   n
## 1 Taylor Swift 171
```

We know each of the nine albums in this dataset were released on different dates so we don't need the `release_date` variable for now.


```r
taylor %>% group_by(album) %>% count(release_date)
```

```
## # A tibble: 9 x 3
## # Groups:   album [9]
##   album                       release_date     n
##   <chr>                       <chr>        <int>
## 1 1989 (Deluxe)               2014-01-01      19
## 2 evermore (deluxe version)   2021-01-07      17
## 3 Fearless (Taylor's Version) 2021-04-09      26
## 4 folklore (deluxe version)   2020-08-18      17
## 5 Lover                       2019-08-23      18
## 6 Red (Deluxe Edition)        2012-10-22      22
## 7 reputation                  2017-11-10      15
## 8 Speak Now (Deluxe Package)  2010-01-01      22
## 9 Taylor Swift                2006-10-24      15
```

```r
#drop variables
taylor <- taylor %>% select(-X,-artist,-release_date)

# Change the album names to be shorter and rearrange them according to realse date.
taylor$album <- recode_factor(taylor$album, "Taylor Swift"="Debut",
                              "Fearless (Taylor's Version)" = "FearlessTV",
                              "Speak Now (Deluxe Package)" = "Speak Now",
                              "Red (Deluxe Edition)" = "Red",
                              "1989 (Deluxe)" = "1989",
                              "reputation"="reputation",
                              "Lover"="Lover",
                              "folklore (deluxe version)" = "folklore",
                              "evermore (deluxe version)"="evermore")
```

# Exploratory Data Analysis

Now, we can start exploring the numeric variables. Firstly, the `popularity` variable has a mean of 61.23 and we notice some songs has a value of 0. It turns out, these are tracks included in the deluxe version of _1989_ but they are not actual songs. I chose to drop these three observations.


```r
ggplot(taylor, aes(x=popularity)) +
  geom_histogram(fill=colores[2],binwidth = 3) +
  theme_classic() + ggtitle("Popularity distribution")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/popularity-1.png" width="672" />

```r
summary(taylor$popularity)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00   58.00   63.00   61.23   67.00   82.00
```

```r
filter(taylor, popularity==0) %>% select(name, album, popularity)
```

```
##                            name album popularity
## 1    I Know Places - Voice Memo  1989          0
## 2 I Wish You Would - Voice Memo  1989          0
## 3      Blank Space - Voice Memo  1989          0
```

```r
taylor <- taylor %>% filter(popularity>0)

ggplot(taylor, aes(x=popularity)) +
  geom_histogram(fill=colores[2],binwidth = 3) +
  theme_classic() + ggtitle("Popularity distribution (without non-songs tracks)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/popularity-2.png" width="672" />

We can compare the popularity of each album by taking the mean of its songs' popularity (and let's add some glitter).


```r
img = "https://t3.ftcdn.net/jpg/04/44/39/80/360_F_444398090_ek1YrGhZa2AZmiUZwALuMe3cRafivme9.jpg"
taylor %>% group_by(album) %>% 
  summarise(album_popularity=round(mean(popularity),2)) %>%
  ggplot(aes(x=album,y=album_popularity)) +
  geom_textured_col(image = img, color = "white", width = 0.8) +
  geom_text(aes(label=album_popularity),size=4, nudge_y = 5) +
  theme_classic() +
  ggtitle("Album's Popularity") + 
  theme(axis.title.x = element_blank(),
        axis.title.y = element_blank()) +
  coord_flip()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/geom_textured-1.png" width="672" />

Now, let's have a quick look at the distribution of the other numeric variables.


```r
tay_long <- taylor %>%
  select(popularity,length, instrumentalness, danceability,acousticness,energy,liveness,loudness,
         speechiness,valence, tempo) %>% 
  pivot_longer(cols = length:tempo)

ggplot(tay_long, aes(x=value)) +
  geom_histogram(aes(fill=name)) +
  facet_wrap(~name, scales = "free") +
  theme_classic() + 
  theme(legend.position = "none") +
  scale_fill_manual(values = colores)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/tay_long-1.png" width="672" />

Let's take a closer look at `instrumentalness` since it seems most of its values are zero.


```r
summary(taylor$acousticness)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
## 0.000191 0.027975 0.142000 0.313407 0.661000 0.971000
```

```r
ggplot(taylor, aes(instrumentalness, popularity)) +
  geom_point(color=colores[5]) +
  theme_classic()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/instr-1.png" width="672" />
We can see that the values of `instrumentalness` are very close to zero and the variable doesn't seem to influence our response (`popularity`) so we decide to drop it.

Now we explore the correlation between the remaining continuous variables. We notice that `energy` and `loudness` have the strongest positive correlation of all pairs of variables; meanwhile, `loudness` and `acousticness` have the strongest negative correlation. This could be an indicative of collinearity problems for a regression model. 


```r
library(corrplot)
cont_vars <- c(3:7,9:13)
taylor_cont_vars <- taylor[,cont_vars]

cor_m <- cor(taylor_cont_vars)
corrplot(cor_m, method = 'shade', order = 'AOE', type = "lower")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/cor-1.png" width="672" />

Let's look at the other continuous variables compared with `popularity`. These scatter plots give us an idea of how much does a certain variable influences the response variable. We notice that `loudness` and `length` seem to have a negative relation with `popularity`; meanwhile, `speechiness` and `danceability` seem to have a positive one.


```r
tay_long <- filter(tay_long, name!= "instrumentalness") #%>% 
  #mutate(value = ifelse(name=="length",format(value, scientific = TRUE), value))

ggplot(tay_long, aes(x=value, y=popularity)) +
  geom_point(aes(color=name)) +
  geom_smooth(method="lm", se=FALSE) +
  facet_wrap(~name, scales = "free") +
  theme_classic() + 
  theme(legend.position = "none",
        axis.text.x = element_text(size=7)) +
  scale_fill_manual(values = colores)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/pairs-1.png" width="672" />

# Modeling

We can start by fitting a multiple linear regression model with all continuos variables.


```r
mod_rlm1 <- lm(popularity~., data = taylor_cont_vars)
summary(mod_rlm1)
```

```
## 
## Call:
## lm(formula = popularity ~ ., data = taylor_cont_vars)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -16.7974  -5.2335   0.3421   4.8021  18.3626 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   5.415e+01  9.054e+00   5.981 1.43e-08 ***
## length       -4.344e-05  1.778e-05  -2.444   0.0156 *  
## danceability  7.404e+00  6.022e+00   1.230   0.2207    
## acousticness -6.882e+00  2.943e+00  -2.339   0.0206 *  
## energy        9.084e+00  6.104e+00   1.488   0.1387    
## liveness     -9.601e+00  8.197e+00  -1.171   0.2432    
## loudness     -1.863e+00  4.212e-01  -4.422 1.81e-05 ***
## speechiness   2.707e+01  1.229e+01   2.203   0.0291 *  
## valence      -2.028e+00  4.277e+00  -0.474   0.6361    
## tempo        -1.313e-02  1.993e-02  -0.659   0.5111    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 7.658 on 158 degrees of freedom
## Multiple R-squared:  0.2672,	Adjusted R-squared:  0.2255 
## F-statistic: 6.402 on 9 and 158 DF,  p-value: 9.961e-08
```


```r
mod_rlm2 <- lm(popularity~., data = select(taylor,-name,-instrumentalness))
summary(mod_rlm2)
```

```
## 
## Call:
## lm(formula = popularity ~ ., data = select(taylor, -name, -instrumentalness))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -8.6685 -2.3496 -0.5954  1.9373 15.6737 
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)      5.531e+01  5.306e+00  10.424  < 2e-16 ***
## albumFearlessTV  1.600e+01  1.499e+00  10.673  < 2e-16 ***
## albumSpeak Now  -7.444e-01  1.653e+00  -0.450   0.6531    
## albumRed         1.067e+01  1.708e+00   6.246 4.13e-09 ***
## album1989        1.528e+01  1.710e+00   8.936 1.37e-15 ***
## albumreputation  2.381e+01  1.924e+00  12.373  < 2e-16 ***
## albumLover       2.332e+01  1.772e+00  13.160  < 2e-16 ***
## albumfolklore    1.546e+01  1.902e+00   8.130 1.49e-13 ***
## albumevermore    1.788e+01  1.971e+00   9.068 6.27e-16 ***
## length           1.025e-05  1.218e-05   0.841   0.4017    
## danceability    -3.370e+00  3.761e+00  -0.896   0.3716    
## acousticness    -2.055e+00  2.099e+00  -0.979   0.3290    
## energy          -2.887e+00  3.672e+00  -0.786   0.4330    
## liveness        -5.707e+00  4.725e+00  -1.208   0.2291    
## loudness         5.116e-01  3.025e-01   1.692   0.0928 .  
## speechiness      1.505e+00  7.539e+00   0.200   0.8420    
## valence          5.436e+00  2.698e+00   2.015   0.0457 *  
## tempo           -1.797e-02  1.148e-02  -1.565   0.1196    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.341 on 150 degrees of freedom
## Multiple R-squared:  0.7764,	Adjusted R-squared:  0.7511 
## F-statistic: 30.64 on 17 and 150 DF,  p-value: < 2.2e-16
```

