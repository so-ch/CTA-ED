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
##       Topic 1      Topic 2    Topic 3     Topic 4      Topic 5      Topic 6    
##  [1,] "percent"    "new"      "bush"      "new"        "rating"     "soviet"   
##  [2,] "prices"     "i"        "campaign"  "california" "roberts"    "central"  
##  [3,] "rate"       "fire"     "police"    "summit"     "new"        "official" 
##  [4,] "report"     "year"     "i"         "warming"    "greyhound"  "peres"    
##  [5,] "new"        "duracell" "president" "air"        "magellan"   "waste"    
##  [6,] "rose"       "children" "dukakis"   "global"     "spacecraft" "snow"     
##  [7,] "month"      "like"     "people"    "leaders"    "study"      "soviets"  
##  [8,] "year"       "mrs"      "man"       "north"      "union"      "gorbachev"
##  [9,] "government" "back"     "years"     "york"       "contact"    "northern" 
## [10,] "economy"    "church"   "city"      "carbon"     "people"     "polish"   
##       Topic 7    Topic 8    Topic 9       Topic 10        
##  [1,] "barry"    "bank"     "year"        "i"             
##  [2,] "national" "new"      "immigration" "administration"
##  [3,] "fbi"      "million"  "officials"   "people"        
##  [4,] "moore"    "company"  "congress"    "state"         
##  [5,] "agents"   "last"     "last"        "dukakis"       
##  [6,] "years"    "oil"      "states"      "noriega"       
##  [7,] "died"     "gas"      "baker"       "thats"         
##  [8,] "i"        "two"      "company"     "american"      
##  [9,] "three"    "florio"   "jews"        "think"         
## [10,] "people"   "reported" "united"      "farmer"
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
##  1     1 percent 0.0471
##  2     6 soviet  0.0198
##  3    10 i       0.0171
##  4     3 bush    0.0166
##  5     8 bank    0.0140
##  6     4 new     0.0137
##  7     1 prices  0.0120
##  8     8 new     0.0115
##  9     1 rate    0.0111
## 10     1 report  0.0109
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
##  1       76    10  1.00
##  2       81     2  1.00
##  3        6    10  1.00
##  4       43     5  1.00
##  5       31     1  1.00
##  6       95     4  1.00
##  7       77     3  1.00
##  8       29     6  1.00
##  9       80     4  1.00
## 10       57     9  1.00
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
