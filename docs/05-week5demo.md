# Week 5 Demo

## Setup
First, we'll load the packages we'll be using in this week's brief demo. 


```r
#devtools::install_github("conjugateprior/austin")
library(austin)
library(quanteda)
library(quanteda.textstats)
```

## Wordscores

We can inspect the function for the wordscores model by @laver_extracting_2003 in the following way:


```r
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
## <bytecode: 0x7fdf05908830>
## <environment: namespace:austin>
```

We can then take some example data included in the `austin` package.


```r
data(lbg)
ref <- getdocs(lbg, 1:5)
```

And here our reference documents are all those documents marked with an "R" for reference; i.e., columns one to five.


```r
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


```r
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


```r
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

```r
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



```r
wordfish
```

We can then simulate some data, formatted appropriately for wordfiash estimation in the following way:


```r
dd <- sim.wordfish()

dd
```

```
## $Y
##      docs
## words D01 D02 D03 D04 D05 D06 D07 D08 D09 D10
##   W01  18  23  13  22  18  10  12   4   8   6
##   W02  21  11  13  15  10  20  13   8   6   4
##   W03  22  17  15  14  23  11   6   8   3   4
##   W04  20  18  30  18  11  23  12   5   9  12
##   W05  12  19  19  13  14  14   8   4   5   2
##   W06   5   6  13   9   9   9  13  22  24  21
##   W07   2  11   9   6   6  15  21  20  19  23
##   W08   9   4   7  11   9  20  16  23  19  31
##   W09   6   8   5   3  13   9  17  16  18  18
##   W10   9   9   9  13  13   8  22  15  26  24
##   W11  50  51  52  47  43  34  22  20  29  17
##   W12  73  54  51  59  38  36  28  24  19  25
##   W13  64  61  54  47  43  31  28  17  22  18
##   W14  60  53  43  39  41  26  29  22  15  13
##   W15  65  59  46  52  45  27  27  18  21  17
##   W16   7  22  16  24  28  45  48  53  59  56
##   W17  21  16  34  30  31  35  43  53  53  39
##   W18  12  19  17  15  45  42  50  49  49  54
##   W19  10  23  24  33  30  38  42  62  60  57
##   W20  14  16  30  30  30  47  43  57  36  59
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


```r
wf <- wordfish(dd$Y)
summary(wf)
```

```
## Call:
## 	wordfish(wfm = dd$Y)
## 
## Document Positions:
##     Estimate Std. Error    Lower   Upper
## D01  -1.5465    0.12153 -1.78470 -1.3083
## D02  -1.0437    0.10712 -1.25370 -0.8338
## D03  -0.7454    0.10129 -0.94391 -0.5469
## D04  -0.6526    0.09987 -0.84830 -0.4568
## D05  -0.3093    0.09618 -0.49781 -0.1208
## D06   0.2180    0.09519  0.03143  0.4046
## D07   0.5943    0.09793  0.40232  0.7862
## D08   1.1218    0.10672  0.91265  1.3310
## D09   1.0667    0.10552  0.85988  1.2735
## D10   1.2956    0.11094  1.07821  1.5131
```

## Using `quanteda`

We can also use `quanteda` to implement the same scaling techniques, as demonstrated in Exercise 4. 

