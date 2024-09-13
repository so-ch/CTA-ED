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
##       Topic 1      Topic 2         Topic 3          Topic 4   Topic 5   
##  [1,] "bank"       "percent"       "i"              "rating"  "new"     
##  [2,] "new"        "soviet"        "bush"           "saudi"   "central" 
##  [3,] "years"      "new"           "people"         "percent" "degrees" 
##  [4,] "percent"    "california"    "campaign"       "new"     "duracell"
##  [5,] "two"        "month"         "administration" "iraq"    "high"    
##  [6,] "blackowned" "manufacturing" "dukakis"        "study"   "snow"    
##  [7,] "businesses" "production"    "president"      "economy" "expected"
##  [8,] "year"       "rate"          "thats"          "soviet"  "northern"
##  [9,] "announced"  "military"      "farmer"         "two"     "record"  
## [10,] "city"       "air"           "agents"         "year"    "southern"
##       Topic 6     Topic 7    Topic 8    Topic 9   Topic 10   
##  [1,] "percent"   "people"   "police"   "barry"   "i"        
##  [2,] "year"      "peres"    "fire"     "warming" "people"   
##  [3,] "prices"    "waste"    "soviet"   "leaders" "state"    
##  [4,] "oil"       "official" "new"      "global"  "new"      
##  [5,] "million"   "congress" "officers" "moore"   "noriega"  
##  [6,] "average"   "israel"   "i"        "north"   "roberts"  
##  [7,] "price"     "jews"     "mrs"      "summit"  "dukakis"  
##  [8,] "gas"       "offer"    "polish"   "kim"     "greyhound"
##  [9,] "last"      "office"   "died"     "skins"   "president"
## [10,] "increased" "germany"  "company"  "man"     "jackson"
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
##  1     2 percent 0.0284
##  2     6 percent 0.0213
##  3     6 year    0.0175
##  4     1 bank    0.0172
##  5     3 i       0.0154
##  6     3 bush    0.0149
##  7     2 soviet  0.0131
##  8     5 new     0.0125
##  9     7 people  0.0117
## 10    10 i       0.0117
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
##  1       76     3  1.00
##  2       81    10  1.00
##  3        6    10  1.00
##  4       43    10  1.00
##  5       31     6  1.00
##  6       95     2  1.00
##  7       77     3  1.00
##  8       29     8  1.00
##  9       80     5  1.00
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
