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

# Just to order albums by date
taylor$album <- ordered(taylor$album, 
      levels = c("Taylor Swift", "Fearless (Taylor's Version)",
                 "Speak Now (Deluxe Package)", "Red (Deluxe Edition)",
                 "1989 (Deluxe)", "reputation", "Lover", "folklore (deluxe version)",
                 "evermore (deluxe version)"))
```

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
##                            name         album popularity
## 1    I Know Places - Voice Memo 1989 (Deluxe)          0
## 2 I Wish You Would - Voice Memo 1989 (Deluxe)          0
## 3      Blank Space - Voice Memo 1989 (Deluxe)          0
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

<img src="{{< blogdown/postref >}}index_files/figure-html/num_vars-1.png" width="672" />

