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
## <bytecode: 0x121539be0>
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
##   W01  15  27  22  10  17  12  12   9   5   6
##   W02  23  22  17  12  11  10  10  11   5   3
##   W03  16  21  17  16  18  12   7  10   6   4
##   W04  25  23  15  14  14  12  13   9   6   3
##   W05  18  20  16  12   8   7  11   7   8   3
##   W06   6  11  12  11  13  18  19  15  15  24
##   W07   3   5  16  10  13  16  20  16  22  14
##   W08   5   6   8  13  13  13  15  18  11  14
##   W09   8   7   5  11  21  13  24  19  15  20
##   W10   3   5   6  14  14  16  19  18  29  24
##   W11  56  59  61  47  32  31  33  27  23  13
##   W12  68  61  57  53  34  30  23  24  18  19
##   W13  51  46  47  49  45  34  21  26  16  12
##   W14  68  52  41  52  45  35  26  25  18  14
##   W15  66  62  51  42  45  35  28  24  16  22
##   W16  18  13  18  20  36  39  49  58  51  61
##   W17  16  21  19  29  27  42  36  38  65  66
##   W18  17  14  17  25  31  48  49  49  55  47
##   W19   8  12  24  33  34  38  47  47  59  73
##   W20  10  13  31  27  29  39  38  50  57  58
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
##     Estimate Std. Error   Lower    Upper
## D01  -1.4783    0.11668 -1.7070 -1.24960
## D02  -1.3089    0.11093 -1.5263 -1.09150
## D03  -0.7767    0.09761 -0.9680 -0.58538
## D04  -0.4370    0.09261 -0.6185 -0.25546
## D05  -0.1297    0.09028 -0.3067  0.04723
## D06   0.2806    0.09036  0.1035  0.45767
## D07   0.5492    0.09237  0.3682  0.73026
## D08   0.6677    0.09376  0.4839  0.85145
## D09   1.1954    0.10376  0.9920  1.39875
## D10   1.4382    0.11056  1.2215  1.65490
```

## Using `quanteda`

We can also use `quanteda` to implement the same scaling techniques, as demonstrated in Exercise 4. 

