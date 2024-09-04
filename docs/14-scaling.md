# Exercise 4: Scaling techniques

## Introduction

The hands-on exercise for this week focuses on: 1) scaling texts ; 2) implementing scaling techniques using `quanteda`. 

In this tutorial, you will learn how to:
  
* Scale texts using the "wordfish" algorithm
* Scale texts gathered from online sources
* Replicate analyses by @kaneko_estimating_2021

Before proceeding, we'll load the packages we will need for this tutorial.




``` r
library(dplyr)
library(quanteda) # includes functions to implement Lexicoder
library(quanteda.textmodels) # for estimating similarity and complexity measures
library(quanteda.textplots) #for visualizing text modelling results
```

In this exercise we'll be using the dataset we used for the sentiment analysis exercise. The data were collected from the Twitter accounts of the top eight newspapers in the UK by circulation. The tweets include any tweets by the news outlet from their main account. 

## Importing data

We can download the dataset with:


``` r
tweets <- readRDS("data/sentanalysis/newstweets.rds")
```

If you're working on this document from your own computer ("locally") you can download the tweets data in the following way:


``` r
tweets  <- readRDS(gzcon(url("https://github.com/cjbarrie/CTA-ED/blob/main/data/sentanalysis/newstweets.rds?raw=true")))
```

We first take a sample from these data to speed up the runtime of some of the analyses. 


``` r
tweets <- tweets %>%
  sample_n(20000)
```

## Construct `dfm` object

Then, as in the previous exercise, we create a corpus object, specify the document-level variables by which we want to group, and generate our document feature matrix. 


``` r
#make corpus object, specifying tweet as text field
tweets_corpus <- corpus(tweets, text_field = "text")

#add in username document-level information
docvars(tweets_corpus, "newspaper") <- tweets$user_username

dfm_tweets <- dfm(tokens(tweets_corpus),
                  remove_punct = TRUE, 
                  remove = stopwords("english"))
```



We can then have a look at the number of documents (tweets) we have per newspaper Twitter account. 


``` r
## number of tweets per newspaper
table(docvars(dfm_tweets, "newspaper"))
```

```
## 
##     DailyMailUK     DailyMirror EveningStandard        guardian         MetroUK 
##            2052            5834            2182            2939             966 
##       Telegraph          TheSun        thetimes 
##            1519            3840             668
```

And this is what our document feature matrix looks like, where each word has a count for each of our eight newspapers. 


``` r
dfm_tweets
```

```
## Document-feature matrix of: 20,000 documents, 48,967 features (99.98% sparse) and 31 docvars.
##        features
## docs    rt @standardnews breaking coronavirus outbreak declared pandemic world
##   text1  1             1        1           1        1        1        1     1
##   text2  1             0        0           0        0        0        0     0
##   text3  0             0        0           0        0        0        0     0
##   text4  0             0        0           0        0        0        0     0
##   text5  0             0        0           0        0        0        0     0
##   text6  0             0        0           0        0        0        0     0
##        features
## docs    health organisation
##   text1      1            1
##   text2      0            0
##   text3      0            0
##   text4      0            0
##   text5      0            0
##   text6      0            0
## [ reached max_ndoc ... 19,994 more documents, reached max_nfeat ... 48,957 more features ]
```

## Estimate wordfish model

Once we have our data in this format, we are able to group and trim the document feature matrix before estimating the wordfish model.


``` r
# compress the document-feature matrix at the newspaper level
dfm_newstweets <- dfm_group(dfm_tweets, groups = newspaper)
# remove words not used by two or more newspapers
dfm_newstweets <- dfm_trim(dfm_newstweets, 
                                min_docfreq = 2, docfreq_type = "count")

## size of the document-feature matrix
dim(dfm_newstweets)
```

```
## [1]     8 11111
```

``` r
#### estimate the Wordfish model ####
set.seed(123L)
dfm_newstweets_results <- textmodel_wordfish(dfm_newstweets #, 
                                             #sparse = TRUE
                                             )
```

And this is what results.


``` r
summary(dfm_newstweets_results)
```

```
## 
## Call:
## textmodel_wordfish.dfm(x = dfm_newstweets)
## 
## Estimated Document Positions:
##                    theta       se
## DailyMailUK      0.65364 0.012735
## DailyMirror      1.18293 0.006521
## EveningStandard -0.22655 0.015934
## guardian        -0.95519 0.010427
## MetroUK         -0.04606 0.022694
## Telegraph       -1.05472 0.010489
## TheSun           1.44685 0.005847
## thetimes        -1.00090 0.014902
## 
## Estimated Feature Scores:
##          rt breaking coronavirus outbreak declared pandemic   world  health
## beta 0.4751   0.1252    0.002689  -0.3348  -0.1318  -0.2693 -0.3874 -0.3975
## psi  5.3564   3.5675    5.841223   3.1632   0.5091   3.2026  3.3967  3.2332
##      organisation genuinely interested      see     one     cos     fair
## beta      -0.4825   -0.3554    -0.3194 -0.06736 -0.1312 -0.3455 -0.09754
## psi        0.5515   -0.5487    -1.4682  2.79755  3.8944 -1.4657  0.35543
##       german    care system protect troubled children #covid19 anxiety  shows
## beta -0.8156 -0.3948  -1.18 -0.1784 -0.54265 -0.05455  -0.7486  0.3587 0.3523
## psi   1.1094  3.1323   1.27  1.9079 -0.08176  2.87554   2.9966  0.5950 2.8632
##         sign    man   behind    app explains    tips
## beta -0.1896 0.4486 -0.01174 0.2052   0.6086 -0.2779
## psi   1.9594 3.6107  2.45969 1.3882   1.2844  1.5468
```

We can then plot our estimates of the $\theta$s---i.e., the estimates of the latent newspaper position---as so.


``` r
textplot_scale1d(dfm_newstweets_results)
```

<img src="14-scaling_files/figure-html/unnamed-chunk-12-1.png" width="672" />

Interestingly, we seem not to have captured ideology but some other tonal dimension. We see that the tabloid newspapers are scored similarly, and grouped toward the right hand side of this latent dimension; whereas the broadsheet newspapers have an estimated theta further to the left.

Plotting the "features," i.e., the word-level betas shows how words are positioned along this dimension, and which words help discriminate between news outlets.


``` r
textplot_scale1d(dfm_newstweets_results, margin = "features")
```

<img src="14-scaling_files/figure-html/unnamed-chunk-13-1.png" width="672" />

And we can also look at these features.


``` r
features <- dfm_newstweets_results[["features"]]

betas <- dfm_newstweets_results[["beta"]]

feat_betas <- as.data.frame(cbind(features, betas))
feat_betas$betas <- as.numeric(feat_betas$betas)

feat_betas %>%
  arrange(desc(betas)) %>%
  top_n(20, betas) %>% 
  kbl() %>%
  kable_styling(bootstrap_options = "striped")
```

<table class="table table-striped" style="color: black; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> features </th>
   <th style="text-align:right;"> betas </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> ig </td>
   <td style="text-align:right;"> 9.089320 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 🎥 </td>
   <td style="text-align:right;"> 7.890592 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diver </td>
   <td style="text-align:right;"> 7.102809 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alien-looking </td>
   <td style="text-align:right;"> 6.121832 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wwe </td>
   <td style="text-align:right;"> 5.356255 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cutest </td>
   <td style="text-align:right;"> 5.356255 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lad </td>
   <td style="text-align:right;"> 5.057634 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bargains </td>
   <td style="text-align:right;"> 4.886314 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> partner's </td>
   <td style="text-align:right;"> 4.731173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ronaldo </td>
   <td style="text-align:right;"> 4.535202 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> clever </td>
   <td style="text-align:right;"> 4.390910 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wheelchair </td>
   <td style="text-align:right;"> 4.371903 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mcguinness </td>
   <td style="text-align:right;"> 4.371903 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spider </td>
   <td style="text-align:right;"> 4.224946 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nails </td>
   <td style="text-align:right;"> 4.224946 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rides </td>
   <td style="text-align:right;"> 3.973758 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ghostly </td>
   <td style="text-align:right;"> 3.973758 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> extensions </td>
   <td style="text-align:right;"> 3.973758 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corrie's </td>
   <td style="text-align:right;"> 3.973758 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lion </td>
   <td style="text-align:right;"> 3.889909 </td>
  </tr>
</tbody>
</table>

These words do seem to belong to more tabloid-style reportage, and include emojis relating to film, sports reporting on "cristiano" as well as more colloquial terms like "saucy."

## Replicating Kaneko et al.

This section adapts code from the replication data provided for @kaneko_estimating_2021 [here](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/EL3KYD). We can access data from the first study by @kaneko_estimating_2021 in the following way. 


``` r
kaneko_dfm <- readRDS("data/wordscaling/study1_kaneko.rds")
```

If you're working locally, you can download the `dfm` data with:


``` r
kaneko_dfm  <- readRDS(gzcon(url("https://github.com/cjbarrie/CTA-ED/blob/main/data/wordscaling/study1_kaneko.rds?raw=true")))
```

This data is in the form a document-feature-matrix. We can first manipulate it in the same way as @kaneko_estimating_2021 by grouping at the level of newspaper and removing infrequent words.


``` r
table(docvars(kaneko_dfm, "Newspaper"))
```

```
## 
##       Asahi     Chugoku    Chunichi    Hokkaido      Kahoku    Mainichi 
##          38          24          47          46          18          26 
##      Nikkei Nishinippon      Sankei     Yomiuri 
##          13          27          14          30
```

``` r
## prepare the newspaper-level document-feature matrix
# compress the document-feature matrix at the newspaper level
kaneko_dfm_study1 <- dfm_group(kaneko_dfm, groups = Newspaper)
# remove words not used by two or more newspapers
kaneko_dfm_study1 <- dfm_trim(kaneko_dfm_study1, min_docfreq = 2, docfreq_type = "count")

## size of the document-feature matrix
dim(kaneko_dfm_study1)
```

```
## [1]   10 4660
```

## Exercises

1. Estimate a wordfish model for the @kaneko_estimating_2021 data
2. Visualize the results
