# Week 6 Demo

## Setup
First, we'll load the packages we'll be using in this week's brief demo. 


``` r
library(topicmodels)
# there are sometimes problem with installing topicmodels on Mac OS X. You can find help on Ken benoit's page here: https://kenbenoit.net/how-to-install-the-r-package-topicmodels-on-os-x/. For me, this required installing gsl and modeltools.

library(dplyr)
library(tidytext)
library(ggplot2)
library(ggthemes)
```

Estimating a topic model requires us first to have our data in the form of a document-term-matrix. This is another term for what we have referred to in previous weeks as a document-feature-matrix.

We can take some example data from the `topicmodels` package. This text is from news releases by the Associated Press. It consists of around 2,200 articles (documents) and over 10,000 terms (words).


``` r
data("AssociatedPress", 
     package = "topicmodels")
```

To estimate the topic model we need only to specify the document-term-matrix we are using, and the number (`k`) of topics that we are estimating. To speed up estimation, we are here only estimating it on 100 articles.


``` r
lda_output <- LDA(AssociatedPress[1:100,], k = 10)
```

We can then inspect the contents of each topic as follows.


``` r
terms(lda_output, 10)
```

```
##       Topic 1     Topic 2      Topic 3      Topic 4       Topic 5     
##  [1,] "bush"      "new"        "percent"    "police"      "i"         
##  [2,] "state"     "noriega"    "fire"       "i"           "people"    
##  [3,] "campaign"  "california" "two"        "immigration" "dukakis"   
##  [4,] "dukakis"   "panama"     "north"      "rating"      "barry"     
##  [5,] "monday"    "skins"      "government" "year"        "new"       
##  [6,] "president" "states"     "officials"  "new"         "waste"     
##  [7,] "i"         "york"       "production" "man"         "government"
##  [8,] "record"    "officials"  "monday"     "mrs"         "moore"     
##  [9,] "school"    "receptor"   "kim"        "national"    "state"     
## [10,] "animals"   "state"      "rate"       "like"        "asked"     
##       Topic 6      Topic 7          Topic 8   Topic 9     Topic 10  
##  [1,] "percent"    "new"            "i"       "soviet"    "bank"    
##  [2,] "prices"     "people"         "warming" "years"     "new"     
##  [3,] "year"       "administration" "embassy" "roberts"   "company" 
##  [4,] "price"      "government"     "global"  "year"      "central" 
##  [5,] "rate"       "greyhound"      "city"    "agents"    "duracell"
##  [6,] "magellan"   "i"              "four"    "peres"     "snow"    
##  [7,] "spacecraft" "union"          "grain"   "official"  "billion" 
##  [8,] "inflation"  "farmer"         "people"  "people"    "england" 
##  [9,] "oil"        "man"            "plant"   "gorbachev" "gas"     
## [10,] "months"     "soviet"         "summit"  "program"   "million"
```

We can then use the `tidy()` function from `tidytext` to gather the relevant parameters we've estimated. To get the $\beta$ per-topic-per-word probabilities (i.e., the probability that the given term belongs to a given topic) we can do the following.


``` r
lda_beta <- tidy(lda_output, matrix = "beta")

lda_beta %>%
  arrange(-beta)
```

```
## # A tibble: 104,730 × 3
##    topic term      beta
##    <int> <chr>    <dbl>
##  1     6 percent 0.0294
##  2     3 percent 0.0260
##  3     1 bush    0.0195
##  4     9 soviet  0.0160
##  5     6 prices  0.0144
##  6    10 bank    0.0143
##  7     6 year    0.0135
##  8     4 police  0.0130
##  9     2 new     0.0124
## 10    10 new     0.0113
## # ℹ 104,720 more rows
```

And to get the $\gamma$ per-document-per-topic probabilities (i.e., the probability that a given document (here: article) belongs to a particular topic) we do the following.



``` r
lda_gamma <- tidy(lda_output, matrix = "gamma")

lda_gamma %>%
  arrange(-gamma)
```

```
## # A tibble: 1,000 × 3
##    document topic gamma
##       <int> <int> <dbl>
##  1       76     7  1.00
##  2       81     5  1.00
##  3        6     2  1.00
##  4       43     9  1.00
##  5       31     6  1.00
##  6       95     2  1.00
##  7       77     1  1.00
##  8       29     9  1.00
##  9       80     1  1.00
## 10       57     7  1.00
## # ℹ 990 more rows
```

And we can easily plot our $\beta$ estimates as follows.


``` r
lda_beta %>%
  group_by(topic) %>%
  top_n(10, beta) %>%
  ungroup() %>%
  arrange(topic, -beta) %>%
  mutate(term = reorder_within(term, beta, topic)) %>%
  ggplot(aes(beta, term, fill = factor(topic))) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ topic, scales = "free", ncol = 4) +
  scale_y_reordered() +
  theme_tufte(base_family = "Helvetica")
```

<img src="06-week6demo_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Which shows us the words associated with each topic, and the size of the associated $\beta$ coefficient. 
