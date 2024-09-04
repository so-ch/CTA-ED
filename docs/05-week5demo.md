# Week 5 Demo

## Setup
First, we'll load the packages we'll be using in this week's brief demo. 


``` r
#devtools::install_github("conjugateprior/austin")
library(austin)
library(quanteda)
library(quanteda.textstats)
```

## Wordscores

We can inspect the function for the wordscores model by @laver_extracting_2003 in the following way:


``` r
classic.wordscores
```

```
## function (wfm, scores) 
## {
##     if (!is.wfm(wfm)) 
##         stop("Function not applicable to this object")
##     if (length(scores) != length(docs(wfm))) 
##         stop("There are not the same number of documents as scores")
##     if (any(is.na(scores))) 
##         stop("One of the reference document scores is NA\nFit the model with known scores and use 'predict' to get virgin score estimates")
##     thecall <- match.call()
##     C.all <- as.worddoc(wfm)
##     C <- C.all[rowSums(C.all) > 0, ]
##     F <- scale(C, center = FALSE, scale = colSums(C))
##     ws <- apply(F, 1, function(x) {
##         sum(scores * x)
##     })/rowSums(F)
##     pi <- matrix(ws, nrow = length(ws))
##     rownames(pi) <- rownames(C)
##     colnames(pi) <- c("Score")
##     val <- list(pi = pi, theta = scores, data = wfm, call = thecall)
##     class(val) <- c("classic.wordscores", "wordscores", class(val))
##     return(val)
## }
## <bytecode: 0x118b8e3e0>
## <environment: namespace:austin>
```

We can then take some example data included in the `austin` package.


``` r
data(lbg)
ref <- getdocs(lbg, 1:5)
```

And here our reference documents are all those documents marked with an "R" for reference; i.e., columns one to five.


``` r
ref
```

```
##      docs
## words  R1  R2  R3  R4  R5
##    A    2   0   0   0   0
##    B    3   0   0   0   0
##    C   10   0   0   0   0
##    D   22   0   0   0   0
##    E   45   0   0   0   0
##    F   78   2   0   0   0
##    G  115   3   0   0   0
##    H  146  10   0   0   0
##    I  158  22   0   0   0
##    J  146  45   0   0   0
##    K  115  78   2   0   0
##    L   78 115   3   0   0
##    M   45 146  10   0   0
##    N   22 158  22   0   0
##    O   10 146  45   0   0
##    P    3 115  78   2   0
##    Q    2  78 115   3   0
##    R    0  45 146  10   0
##    S    0  22 158  22   0
##    T    0  10 146  45   0
##    U    0   3 115  78   2
##    V    0   2  78 115   3
##    W    0   0  45 146  10
##    X    0   0  22 158  22
##    Y    0   0  10 146  45
##    Z    0   0   3 115  78
##    ZA   0   0   2  78 115
##    ZB   0   0   0  45 146
##    ZC   0   0   0  22 158
##    ZD   0   0   0  10 146
##    ZE   0   0   0   3 115
##    ZF   0   0   0   2  78
##    ZG   0   0   0   0  45
##    ZH   0   0   0   0  22
##    ZI   0   0   0   0  10
##    ZJ   0   0   0   0   3
##    ZK   0   0   0   0   2
```

This matrix is simply a series of words (here: letters) and reference texts with word counts for each of them. 

We can then look at the wordscores for each of the words, calculated using the reference dimensions for each of the reference documents.


``` r
ws <- classic.wordscores(ref, scores=seq(-1.5,1.5,by=0.75))
ws
```

```
## $pi
##         Score
## A  -1.5000000
## B  -1.5000000
## C  -1.5000000
## D  -1.5000000
## E  -1.5000000
## F  -1.4812500
## G  -1.4809322
## H  -1.4519231
## I  -1.4083333
## J  -1.3232984
## K  -1.1846154
## L  -1.0369898
## M  -0.8805970
## N  -0.7500000
## O  -0.6194030
## P  -0.4507576
## Q  -0.2992424
## R  -0.1305970
## S   0.0000000
## T   0.1305970
## U   0.2992424
## V   0.4507576
## W   0.6194030
## X   0.7500000
## Y   0.8805970
## Z   1.0369898
## ZA  1.1846154
## ZB  1.3232984
## ZC  1.4083333
## ZD  1.4519231
## ZE  1.4809322
## ZF  1.4812500
## ZG  1.5000000
## ZH  1.5000000
## ZI  1.5000000
## ZJ  1.5000000
## ZK  1.5000000
## 
## $theta
## [1] -1.50 -0.75  0.00  0.75  1.50
## 
## $data
##      docs
## words  R1  R2  R3  R4  R5
##    A    2   0   0   0   0
##    B    3   0   0   0   0
##    C   10   0   0   0   0
##    D   22   0   0   0   0
##    E   45   0   0   0   0
##    F   78   2   0   0   0
##    G  115   3   0   0   0
##    H  146  10   0   0   0
##    I  158  22   0   0   0
##    J  146  45   0   0   0
##    K  115  78   2   0   0
##    L   78 115   3   0   0
##    M   45 146  10   0   0
##    N   22 158  22   0   0
##    O   10 146  45   0   0
##    P    3 115  78   2   0
##    Q    2  78 115   3   0
##    R    0  45 146  10   0
##    S    0  22 158  22   0
##    T    0  10 146  45   0
##    U    0   3 115  78   2
##    V    0   2  78 115   3
##    W    0   0  45 146  10
##    X    0   0  22 158  22
##    Y    0   0  10 146  45
##    Z    0   0   3 115  78
##    ZA   0   0   2  78 115
##    ZB   0   0   0  45 146
##    ZC   0   0   0  22 158
##    ZD   0   0   0  10 146
##    ZE   0   0   0   3 115
##    ZF   0   0   0   2  78
##    ZG   0   0   0   0  45
##    ZH   0   0   0   0  22
##    ZI   0   0   0   0  10
##    ZJ   0   0   0   0   3
##    ZK   0   0   0   0   2
## 
## $call
## classic.wordscores(wfm = ref, scores = seq(-1.5, 1.5, by = 0.75))
## 
## attr(,"class")
## [1] "classic.wordscores" "wordscores"         "list"
```

And here we see the thetas contained in this wordscores object, i.e., the reference dimensions for each of the reference documents and the pis, i.e., the estimated wordscores for each word. 

We can now use these to score the so-called "virgin" texts as follows. 


``` r
#get "virgin" documents
vir <- getdocs(lbg, 'V1')
vir
```

```
##      docs
## words  V1
##    A    0
##    B    0
##    C    0
##    D    0
##    E    0
##    F    0
##    G    0
##    H    2
##    I    3
##    J   10
##    K   22
##    L   45
##    M   78
##    N  115
##    O  146
##    P  158
##    Q  146
##    R  115
##    S   78
##    T   45
##    U   22
##    V   10
##    W    3
##    X    2
##    Y    0
##    Z    0
##    ZA   0
##    ZB   0
##    ZC   0
##    ZD   0
##    ZE   0
##    ZF   0
##    ZG   0
##    ZH   0
##    ZI   0
##    ZJ   0
##    ZK   0
```

``` r
# predict textscores for the virgin documents
predict(ws, newdata=vir)
```

```
## 37 of 37 words (100%) are scorable
## 
##     Score Std. Err. Rescaled  Lower  Upper
## V1 -0.448    0.0119   -0.448 -0.459 -0.437
```

## Wordfish


If we wish, we can inspect the function for the wordscores model by @slapin_scaling_2008 in the following way. This is a much more complex algorithm, which is not printed here, but you can inspect on your own devices. 



``` r
wordfish
```

We can then simulate some data, formatted appropriately for wordfiash estimation in the following way:


``` r
dd <- sim.wordfish()

dd
```

```
## $Y
##      docs
## words D01 D02 D03 D04 D05 D06 D07 D08 D09 D10
##   W01  19  21  27  12  25  12   6   9   7   8
##   W02  22  17  19  12  10  15  13  13   5   3
##   W03  25  11  19  11  12  16  12   5   6   3
##   W04  18  18  13  15  14  13  13   7   8   3
##   W05  16  20  18  25  18  21  13   7   9  12
##   W06   5   8   8  11  11  16  15  15  27  25
##   W07   5   6   6  11   6  13  19  24  19  18
##   W08   4   4   9  12  12  12  19  17   8  15
##   W09   4   5  11   9  19  14  19  21  20  14
##   W10  12   7   8   7  13  14  10  18  20  26
##   W11  65  61  47  47  42  37  26  26   7  14
##   W12  48  50  47  43  38  29  30  17  20   7
##   W13  58  51  54  48  50  32  31  18  15  16
##   W14  60  73  41  49  32  23  32  24  17  19
##   W15  62  58  50  51  35  23  31  22  20   8
##   W16  10  15  27  24  34  46  31  58  57  49
##   W17  18  23  24  24  27  42  40  49  52  68
##   W18   9  15  27  28  32  38  53  40  72  58
##   W19  21  20  22  37  35  46  47  43  53  76
##   W20  19  17  23  24  35  38  40  67  58  58
## 
## $theta
##  [1] -1.4863011 -1.1560120 -0.8257228 -0.4954337 -0.1651446  0.1651446
##  [7]  0.4954337  0.8257228  1.1560120  1.4863011
## 
## $doclen
## D01 D02 D03 D04 D05 D06 D07 D08 D09 D10 
## 500 500 500 500 500 500 500 500 500 500 
## 
## $psi
##  [1] 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1
## 
## $beta
##  [1] 0 0 0 0 0 1 1 1 1 1 0 0 0 0 0 1 1 1 1 1
## 
## attr(,"class")
## [1] "wordfish.simdata" "list"
```

Here we can see the document and word-level FEs, as well as the specified range of the thetas to be estimates. 

Then estimating the document positions is simply a matter of implementing this algorithm.


``` r
wf <- wordfish(dd$Y)
summary(wf)
```

```
## Call:
## 	wordfish(wfm = dd$Y)
## 
## Document Positions:
##     Estimate Std. Error    Lower    Upper
## D01  -1.3859    0.11111 -1.60369 -1.16814
## D02  -1.2149    0.10635 -1.42339 -1.00649
## D03  -0.7517    0.09692 -0.94164 -0.56172
## D04  -0.5570    0.09440 -0.74203 -0.37200
## D05  -0.2453    0.09206 -0.42573 -0.06487
## D06   0.2287    0.09246  0.04744  0.40989
## D07   0.3226    0.09311  0.14010  0.50508
## D08   0.8440    0.10019  0.64761  1.04036
## D09   1.2636    0.11041  1.04721  1.48002
## D10   1.4953    0.11792  1.26420  1.72644
```

## Using `quanteda`

We can also use `quanteda` to implement the same scaling techniques, as demonstrated in Exercise 4. 

