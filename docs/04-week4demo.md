# Week 4 Demo

## Setup
First, we'll load the packages we'll be using in this week's brief demo. 


```r
library(quanteda)
library(quanteda.textstats)
library(quanteda.textplots)
library(tidytext)
library(stringdist)
library(corrplot)
library(janeaustenr)
```

## Character-based similarity

A first measure of text similarity is at the level of characters. We can look *for the last time* (I promise) at the example from the lecture and see how similarity compares. 

We'll make two sentences and create two character objects from them. These are two thoughts imagined up from our classes. 


```r
a <- "We are all very happy to be at a lecture at 11AM"
b <- "We are all even happier that we don’t have two lectures a week"
```

We know that the "longest common substring measure" is, according to the [stringdist](https://cran.r-project.org/web/packages/stringdist/stringdist.pdf) package documentation, "the longest string that can be obtained by pairing characters from *a* and *b* while keeping the order of characters intact."

And we can easily get different distance/similarity measures by comparing our character objects `a` and `b` as so. 


```r
## longest common substring distance
stringdist(a, b,
           method = "lcs")
```

```
## [1] 36
```

```r
## levenshtein distance
stringdist(a, b,
           method = "lv")
```

```
## [1] 27
```

```r
## jaro distance
stringdist(a, b,
           method = "jw", p =0)
```

```
## [1] 0.2550103
```

## Term-based similarity

In this second example from the lecture, we're taking the opening line of *Pride and Prejudice* alongside my own versions of this same famous opening line. 

We can get the text of Jane Austen very easily thanks to the `janeaustenr` package.


```r
## similarity and distance example

text <- janeaustenr::prideprejudice

sentences <- text[10:11]

sentence1 <- paste(sentences[1], sentences[2], sep = " ")

sentence1
```

```
## [1] "It is a truth universally acknowledged, that a single man in possession of a good fortune, must be in want of a wife."
```

We're then going to specify our alternative versions of this same sentence. 


```r
sentence2 <- "Everyone knows that a rich man without wife will want a wife"

sentence3 <- "He's loaded so he wants to get married. Everyone knows that's what happens."
```

Finally, we're going to convert these into a document feature matrix. We're doing this with the `quanteda` package, which is a package that we'll begin using more and more over coming weeks as the analyses we're performing get gradually more technical. 


```r
dfmat <- dfm(tokens(c(sentence1,
                      sentence2,
                      sentence3)),
             remove_punct = TRUE, remove = stopwords("english"))

dfmat
```

```
## Document-feature matrix of: 3 documents, 20 features (58.33% sparse) and 0 docvars.
##        features
## docs    truth universally acknowledged single man possession good fortune must
##   text1     1           1            1      1   1          1    1       1    1
##   text2     0           0            0      0   1          0    0       0    0
##   text3     0           0            0      0   0          0    0       0    0
##        features
## docs    want
##   text1    1
##   text2    1
##   text3    0
## [ reached max_nfeat ... 10 more features ]
```

What do we see here?

Well, it's clear that `text2` and `text3` are not very similar to `text1` at all---they share few words. But we also see that `text2` does at least contain some words that are shared with `text1`, which is the original opening line of Jane Austen's *Pride and Prejudice*. 

So, how do we then measure the similarity or distance between these texts?

The first way is simply by correlating the two sets of ones and zeroes. We can do this with the `quanteda.textstats` package like so.


```r
## correlation
textstat_simil(dfmat, margin = "documents", method = "correlation")
```

```
## textstat_simil object; method = "correlation"
##         text1   text2  text3
## text1  1.0000 -0.0689 -0.811
## text2 -0.0689  1.0000 -0.144
## text3 -0.8112 -0.1438  1.000
```

And you'll see that this is the same as what we would get if we manipulated the data into tidy format (rows for words and columns of 1s and 0s).


```r
test <- tidy(dfmat)
test <- test %>%
  cast_dfm(term, document, count)
test <- as.data.frame(test)

res <- cor(test[,2:4])
res
```

```
##             text1       text2      text3
## text1  1.00000000 -0.06894503 -0.8112457
## text2 -0.06894503  1.00000000 -0.1438235
## text3 -0.81124574 -0.14382349  1.0000000
```

And we see that as expected `text2` is more highly correlated with `text1` than is `text3`. 


```r
corrplot(res, type = "upper", order = "hclust", 
         tl.col = "black", tl.srt = 45)
```

<img src="04-week4demo_files/figure-html/unnamed-chunk-9-1.png" width="672" />
As for Euclidean distances, we can again use `quanteda` as so.


```r
textstat_dist(dfmat, margin = "documents", method = "euclidean")
```

```
## textstat_dist object; method = "euclidean"
##       text1 text2 text3
## text1     0  3.61  4.24
## text2  3.61     0  3.61
## text3  4.24  3.61     0
```

Or we could define our own function just so we see what's going on behind the scenes.


```r
# function for Euclidean distance
euclidean <- function(a,b) sqrt(sum((a - b)^2))
# estimating the distance
euclidean(test$text1, test$text2)
```

```
## [1] 3.605551
```

```r
euclidean(test$text1, test$text3)
```

```
## [1] 4.242641
```

```r
euclidean(test$text2, test$text3)
```

```
## [1] 3.605551
```

For Manhattan distance, we could use `quanteda` again.


```r
textstat_dist(dfmat, margin = "documents", method = "manhattan")
```

```
## textstat_dist object; method = "manhattan"
##       text1 text2 text3
## text1     0    13    18
## text2    13     0    11
## text3    18    11     0
```

Or we could again define our own function.


```r
## manhattan
manhattan <- function(a, b){
  dist <- abs(a - b)
  dist <- sum(dist)
  return(dist)
}

manhattan(test$text1, test$text2)
```

```
## [1] 13
```

```r
manhattan(test$text1, test$text3)
```

```
## [1] 18
```

```r
manhattan(test$text2, test$text3)
```

```
## [1] 11
```

And for the cosine similarity, `quanteda` again makes this straightforward.


```r
textstat_simil(dfmat, margin = "documents", method = "cosine")
```

```
## textstat_simil object; method = "cosine"
##       text1 text2 text3
## text1 1.000 0.381     0
## text2 0.381 1.000 0.239
## text3     0 0.239 1.000
```

But to make clear what's going on here, we could again write our own function. 


```r
## cosine
cos.sim <- function(a, b) 
{
  return(sum(a*b)/sqrt(sum(a^2)*sum(b^2)) )
}  

cos.sim(test$text1, test$text2)
```

```
## [1] 0.381385
```

```r
cos.sim(test$text1, test$text3)
```

```
## [1] 0
```

```r
cos.sim(test$text2, test$text3)
```

```
## [1] 0.2390457
```

## Complexity

Note: this section borrows notation from the materials for the [`texstat_readability()` function](https://quanteda.io/reference/textstat_readability.html).

We also talked about different document-level measures of text characteristics. One of these is the "complexity" or readability of a text. One of the most frequently used is Flesch's Reading Ease Score (Flesch 1948).

This is computed as:


\item{\code{"Flesch"}:}{Flesch's Reading Ease Score (Flesch 1948).
\deqn{206.835 - (1.015 \times ASL) - (84.6 \times \frac{n_{sy}}{n_{w}})}{
  206.835 - (1.015 * ASL) - (84.6 * (Nsy / Nw))}}
  
We can estimate a readability score for our respective sentences as such. The Flesch score from 1948 is the default. 


```r
textstat_readability(sentence1)
```

```
##   document   Flesch
## 1    text1 62.10739
```

```r
textstat_readability(sentence2)
```

```
##   document Flesch
## 1    text1 88.905
```

```r
textstat_readability(sentence3)
```

```
##   document   Flesch
## 1    text1 83.09904
```

What do we see here? The original Austen opening line is marked lower in readability than our more colloquial alternatives. 

But there are other alternatives measures we might use. You can check these out by clicking through the links of the function `textstat_readability()`. Below I display a few of these. 

One such is the McLaughlin (1969) "Simple Measure of Gobbledygook, which is based on the recurrence of words with 3 syllables or more and is calculated as:


\item{\code{"SMOG"}:}{Simple Measure of Gobbledygook (SMOG) (McLaughlin 1969). \deqn{ 1.043
   \times \sqrt{n_{wsy>=3}} \times \frac{30}{n_{st}} + 3.1291}{1.043 * sqrt(Nwmin3sy
   * 30 / Nst) + 3.1291}

where \eqn{n_{wsy>=3}} = Nwmin3sy = the number of words with 3 syllables or more.
This measure is regression equation D in McLaughlin's original paper.}

We can calculate this for our three sentences as below.


```r
textstat_readability(sentence1, measure = "SMOG")
```

```
##   document     SMOG
## 1    text1 13.02387
```

```r
textstat_readability(sentence2, measure = "SMOG")
```

```
##   document     SMOG
## 1    text1 8.841846
```

```r
textstat_readability(sentence3, measure = "SMOG")
```

```
##   document     SMOG
## 1    text1 7.168622
```

Here, again, we see that the original Austen sentence has a higher level of complexity (or gobbledygook!).
