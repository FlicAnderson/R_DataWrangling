Data Wrangling & Tidy Data in R
========================================================
author: Flic Anderson
date: 29 September 2015
font-import: http://fonts.googleapis.com/css?family=Telex
font-family: 'Telex'

RBGE Code Group 
This presentation at: https://github.com/FlicAnderson/R_DataWrangling



What IS Data Wrangling?!
========================================================

The process of cleaning, transforming and generally converting raw data into a form which allows easier use later.

For reproducible science, this is best done programmatically.  

R is great for this & there are a few great packages which make life much easier (dplyr, tidyr, reshape/reshape2, etc)  

If you work with data, you probably do a lot of data wrangling - congrats! 



What IS Tidy Data?!
========================================================

Tidy data is a subset of data wrangling.

The aim is to get your untidy raw data into a structure which allows easy analysis

3 principles:
- Each variable forms a column
- Each observation forms a row
- Each type of observational unit forms a table



Common Messy Data Issues
========================================================

Your data might be messy, if:
- 1) Column headings are values, not variable names 
- 2) Multiple variables are stored in one column
- 3) Variables in both rows and columns
- 4) Multiple types of observational units are stored in the same table
- 5) A single observational unit is stored in multiple tables



Let's Go!
========================================================

Install & load packages we'll work with:


```r
# dplyr
if (!require(dplyr)){
  install.packages("dplyr")
  library(dplyr)
} 

# tidyr
if (!require(tidyr)){
  install.packages("tidyr")
  library(tidyr)
} 

# NB: for shorter/simpler code, can just use "library(dplyr)" if it's installed already.
```



A Brief Intro to {dplyr}
==========================================================

The {dplyr} package is one of a stable of packages from the 'Hadleyverse'...

Hadley Wickham's packages tend to have a particular grammar which is easy to use & quick to learn.

{dplyr} is a good example - when the grammar *clicks*, it can really simplify your code & make it easier to read & understand.  {tidyr} uses some of the same grammar.



Example Data Set: datA
========================================================

This is **randomly generated** botanical survey data, based on my MSc project data


```r
# data frame: 
datA <- data.frame(
        # sampling site codes
        vegplot=c("S1T1A", "S1T1B", "S1T1C"),  
        # elevation of sample
        altM=c(100, 200, 300), 
        # presence values for taxa
        T1=sample(0:1, 3, replace=TRUE),  
        G1=sample(0:1, 3, replace=TRUE),  
        H1=sample(0:1, 3, replace=TRUE),  
        H2=sample(0:1, 3, replace=TRUE)
)
```



Messy Data: Problem 1
========================================================

1) Column headings are values, not variable names

This can be helpful when displaying data, but isn't great for analyses.

Here, the taxa (T1, G1, H2 etc) along the top are technically *values*, not variables.


```
  vegplot altM T1 G1 H1 H2
1   S1T1A  100  0  0  0  0
2   S1T1B  200  1  1  1  1
3   S1T1C  300  0  1  1  0
```



Messy Data: Fix 1
========================================================

- pull species from headings into a column we're calling "taxon"
- 'gather' values under each heading into a column we've called "presence"
- don't gather site codes or altitude! (use minus symbol)


```r
datA <- 
datA %>% 
  gather(taxon, presence, -vegplot, -altM) %>%
  head %>%  # only show first 6 rows
print
```



Messy Data: Fix 1
========================================================

- pull species from headings into a column we're calling "taxon"
- **'gather'** values under each heading into a column we've called "presence"
- don't gather site codes or altitude! (use minus symbol)


```
  vegplot altM taxon presence
1   S1T1A  100    T1        0
2   S1T1B  200    T1        1
3   S1T1C  300    T1        0
4   S1T1A  100    G1        0
5   S1T1B  200    G1        1
6   S1T1C  300    G1        1
```



Messy Data: Problem 2
========================================================

2) Multiple variables are stored in one column

The vegplots variable is made up of a code for each sampling point & represents multiple variables:
- S1 = site 1 [1:3]
- T1 = transect 1  \[1:5] (replicates!)
- A = releve A  [A:F]
We'll need to split these up if we want to compare things like species by site.


```
     [,1]   
[1,] "S1T1A"
[2,] "S1T1B"
[3,] "S1T1C"
[4,] "S1T1A"
[5,] "S1T1B"
[6,] "S1T1C"
```



Messy Data: Fix 2
========================================================

- we separate site, transect & releve data into columns of their own


```r
datA <- 
datA %>% 
  separate(
          # column to split
          vegplot, 
          # new column names
          into=c("site", "transect", "releve"), 
          # numeric/position separator
          sep=c(2, 4), 
          # remove old vegplot column
          remove=TRUE  
          ) %>%
  head %>%   # only show first 6 rows
print
```



Messy Data: Fix 2
========================================================

- we separate site, transect & releve data into columns of their own


```
  site transect releve altM taxon presence
1   S1       T1      A  100    T1        0
2   S1       T1      B  200    T1        1
3   S1       T1      C  300    T1        0
4   S1       T1      A  100    G1        0
5   S1       T1      B  200    G1        1
6   S1       T1      C  300    G1        1
```



Messy Data: Problem 3
========================================================

3) Variables in both rows and columns

Here we've got some coppice & grazing data from a releve.


```r
treeData <- data.frame(
        #site
        site=c("S1", "S1", "S1"), 
        #transect
        trnsct=c("T1", "T1", "T1"), 
        # releve
        releve=c("A", "A", "A"), 
        # nearest 3 trees:
        near1=c("oak", "non-cop", "graz"),
        near2=c("pear", NA, "no-graz"),
        near3=c("oak", "yng-cop", "no-graz")
)
```



Messy Data: Problem 3
========================================================

3) Variables in both rows and columns

The 3 nearest trees (near1, near2 & near3) were surveyed for:
- species (oak, pear, etc)
- if oak: evidence of coppice & age of coppicing if evidence found
- evidence of grazing (graz = yes, no-graz = no)


```
  site trnsct releve   near1   near2   near3
1   S1     T1      A     oak    pear     oak
2   S1     T1      A non-cop    <NA> yng-cop
3   S1     T1      A    graz no-graz no-graz
```


Messy Data: Fix 3
========================================================

Step 1: gather the columns near1 - near3 into a new variable called "tree" using the gather( ) function from the {tidyr} package


```r
treeData <- 
   treeData %>% 
     gather(
     # gather near1:near3 into new column "tree"
             tree, 
     # dump info into new column "collInfo"
             collInfo, 
     # specify columns - near1: near3
             near1:near3, 
     # remove NA/missing values
             na.rm=TRUE) %>%
     print
```



Messy Data: Fix 3
========================================================

Step 1: gather the columns near1 - near3 into a new variable called "tree" using the gather( ) function from the {tidyr} package


```
  site trnsct releve  tree collInfo
1   S1     T1      A near1      oak
2   S1     T1      A near1  non-cop
3   S1     T1      A near1     graz
4   S1     T1      A near2     pear
5   S1     T1      A near2  no-graz
6   S1     T1      A near3      oak
7   S1     T1      A near3  yng-cop
8   S1     T1      A near3  no-graz
```



Messy Data: Fix 3
========================================================

Step 2: now we need to spread the 'collInfo' which still contains data on 3 separate kinds of variable (species, coppicing & grazing) into separate columns using the spread( ) function from the {tidyr} package

The spread( ) function spreads key-value pairs across multiple columns.






















```
Error in eval(expr, envir, enclos) : object 'near1' not found
```
