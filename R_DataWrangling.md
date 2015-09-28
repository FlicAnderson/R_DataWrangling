Data Wrangling & Tidy Data in R
========================================================
author: Flic Anderson
date: 29 September 2015
font-import: http://fonts.googleapis.com/css?family=Telex
font-family: 'Telex'

RBGE Code Group    
This presentation at: https://github.com/FlicAnderson/R_DataWrangling    
GitHub: @FlicAnderson



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
        Tr1=sample(0:1, 3, replace=TRUE),  
        Gr1=sample(0:1, 3, replace=TRUE),  
        Hr1=sample(0:1, 3, replace=TRUE),  
        Hr2=sample(0:1, 3, replace=TRUE)
)
```



Messy Data: Problem 1
========================================================

1) Column headings are values, not variable names

This can be helpful when displaying data, but isn't great for analyses.

Here, the taxa (Tr1, Gr1, He2 etc) along the top are technically *values*, not variables.


```
  vegplot altM Tr1 Gr1 Hr1 Hr2
1   S1T1A  100   0   1   0   1
2   S1T1B  200   0   1   0   0
3   S1T1C  300   0   0   1   0
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
1   S1T1A  100   Tr1        0
2   S1T1B  200   Tr1        0
3   S1T1C  300   Tr1        0
4   S1T1A  100   Gr1        1
5   S1T1B  200   Gr1        1
6   S1T1C  300   Gr1        0
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
1   S1       T1      A  100   Tr1        0
2   S1       T1      B  200   Tr1        0
3   S1       T1      C  300   Tr1        0
4   S1       T1      A  100   Gr1        1
5   S1       T1      B  200   Gr1        1
6   S1       T1      C  300   Gr1        0
```



Example Data Set: treeData
========================================================

This is **fictional** botanical survey data, based on my MSc project data.  
Yes, I did end up with this shambles of a table... I hadn't thought out my headings before I started measuring!


```r
treeData <- data.frame(
   #vegplot
   site=c("S1T1A", "S1T1A", "S1T1B", "S1T1B"), 
   # measure
   measured=c("sps", "cop", "sps", "cop"),
   # nearest 3 trees
   near1=c("oak", "non-cop", "oak", "old-cop"),
   near2=c("pear", NA, "oak", "yng-cop"),
   near3=c("oak", "yng-cop", "prunus", NA)
)
```



Messy Data: Problem 3
========================================================

3) Variables in both rows and columns

At the 2 sites shown, the 3 nearest trees (near1, near2 & near3) were surveyed for:
- species (oak, pear, etc)
- if oak: evidence of coppice & age of coppicing if evidence found


```
  vegplot measured   near1   near2   near3
1   S1T1A      sps     oak    pear     oak
2   S1T1A      cop non-cop    <NA> yng-cop
3   S1T1B      sps     oak     oak    plum
4   S1T1B      cop old-cop yng-cop    <NA>
```


Messy Data: Fix 3
========================================================

Step 1: gather columns near1 - near3 into new variable "tree" using gather( ) function from the {tidyr} package


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

Step 1: gather columns near1 - near3 into new variable "tree" using gather( ) function from the {tidyr} package


```
   vegplot measured  tree collInfo
1    S1T1A      sps near1      oak
2    S1T1A      cop near1  non-cop
3    S1T1B      sps near1      oak
4    S1T1B      cop near1  old-cop
5    S1T1A      sps near2     pear
6    S1T1B      sps near2      oak
7    S1T1B      cop near2  yng-cop
8    S1T1A      sps near3      oak
9    S1T1A      cop near3  yng-cop
10   S1T1B      sps near3     plum
```



Messy Data: Fix 3
========================================================

Step 2: now spread 'collInfo' which contains data on 2 separate variables (species, coppicing) into separate columns using spread( ) function from the {tidyr} package


```r
treeData <-
   treeData %>% 
      # spread 'measured' column as new columns
      # fill with contents of 'collInfo' column
      spread(measured, collInfo) %>%
      # extract numerics from 'near1' etc
      mutate(tree=extract_numeric(tree)) %>%
   print
```



Messy Data: Fix 3
========================================================

Step 2: now spread 'collInfo' which contains data on 2 separate variables (species, coppicing) into separate columns using spread( ) function from the {tidyr} package


```
  vegplot tree     cop  sps
1   S1T1A    1 non-cop  oak
2   S1T1A    2    <NA> pear
3   S1T1A    3 yng-cop  oak
4   S1T1B    1 old-cop  oak
5   S1T1B    2 yng-cop  oak
6   S1T1B    3    <NA> plum
```



Messy Data: Fix 3
========================================================

Finally, we also want to fix the vegplots bit again, which we can do with the separate( ) function as before.


```r
   treeData %>% 
        # separate vegplot into site, transect, releve
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
     print
```



Messy Data: Fix 3
========================================================

Finally, we also want to fix the vegplots bit again, which we can do with the separate( ) function as before.


```
  site transect releve tree     cop  sps
1   S1       T1      A    1 non-cop  oak
2   S1       T1      A    2    <NA> pear
3   S1       T1      A    3 yng-cop  oak
4   S1       T1      B    1 old-cop  oak
5   S1       T1      B    2 yng-cop  oak
6   S1       T1      B    3    <NA> plum
```



Messy Data: Problem 4
========================================================

4) Multiple types of observational units are stored in the same table

Here, we had elevational data (altM) and species data (taxon, presence) stored in the same table.  That's not tidy data!   

Splitting these 'types' of data apart is called 'normalisation' & is done in relational databases


```
  site transect releve altM taxon presence
1   S1       T1      A  100   Tr1        0
2   S1       T1      B  200   Tr1        0
3   S1       T1      C  300   Tr1        0
4   S1       T1      A  100   Gr1        1
5   S1       T1      B  200   Gr1        1
6   S1       T1      C  300   Gr1        0
```



Messy Data: Fix 4
========================================================

Split elevation data (altM) amd species data (taxon, presence) off into separate tables.

```r
# environmental data  
envDat <- 
   datA %>% 
      select(site, transect, releve, altM) %>%
print
```

```
  site transect releve altM
1   S1       T1      A  100
2   S1       T1      B  200
3   S1       T1      C  300
4   S1       T1      A  100
5   S1       T1      B  200
6   S1       T1      C  300
```



Messy Data: Fix 4
========================================================

Split elevation data (altM) amd species data (taxon, presence) off into separate tables.

```r
# species data
spsDat <- 
   datA %>%
      select(site, transect, releve, taxon, presence) %>%
print
```

```
  site transect releve taxon presence
1   S1       T1      A   Tr1        0
2   S1       T1      B   Tr1        0
3   S1       T1      C   Tr1        0
4   S1       T1      A   Gr1        1
5   S1       T1      B   Gr1        1
6   S1       T1      C   Gr1        0
```



Messy Data: Problem 5
========================================================

5) A single observational unit is stored in multiple tables

Sometimes we'll accumulate several files which often store the same type of observation.

This can happen for various reasons, but often the names of the tables actually represent the value of a variable (eg. sampling location or date of data collection).





Messy Data: Problem 5
========================================================

5) A single observational unit is stored in multiple tables

Here we've got separate files from each sample location:

```r
# Site 2, Transect 1, releve A
S2T1A
```

```
  altM Tr1 Gr1 Hr1 Hr2
1  100   1   0   1   0
2  200   0   1   0   1
3  300   1   0   1   1
```

```r
# S2T1B has same format.
```
Here, the name of the table is actually the variable "vegplot".



Messy Data: Fix 5
========================================================

Before joining the tables (S2T1A & S2T1B), we want to add a new column to store that name variable information.

Do this using mutate() function from {dplyr}:


```r
# data <- mutate(data, newColumnName="value")
S2T1A <- mutate(S2T1A, vegplot="S2T1A")
S2T1A
```

```
  altM Tr1 Gr1 Hr1 Hr2 vegplot
1  100   1   0   1   0   S2T1A
2  200   0   1   0   1   S2T1A
3  300   1   0   1   1   S2T1A
```



Messy Data: Fix 5
========================================================

Then we use the bind_rows() function to join the tables:


```r
datB <- bind_rows(S2T1A, S2T1B)
datB
```

```
Source: local data frame [6 x 6]

  altM Tr1 Gr1 Hr1 Hr2 vegplot
1  100   1   0   1   0   S2T1A
2  200   0   1   0   1   S2T1A
3  300   1   0   1   1   S2T1A
4  100   0   1   0   0   S2T1B
5  200   0   0   1   0   S2T1B
6  300   1   0   1   0   S2T1B
```



Messy Data: Fix 5
========================================================

Now to tidy up the next messy data problem we've ended up with: multiple variables in 1 column!


```r
datB <- 
datB %>% 
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



Messy Data: Fix 5
========================================================

We've split the vegplot variable out into the site, transect and releve codes as we did before.


```
Source: local data frame [6 x 8]

  altM Tr1 Gr1 Hr1 Hr2 site transect releve
1  100   1   0   1   0   S2       T1      A
2  200   0   1   0   1   S2       T1      A
3  300   1   0   1   1   S2       T1      A
4  100   0   1   0   0   S2       T1      B
5  200   0   0   1   0   S2       T1      B
6  300   1   0   1   0   S2       T1      B
```



Now What?
========================================================

Now your data is tidy, it's far easier to analyse, plot & share!

Put all data tidying steps into a script & run this on the raw data each time you want to analyse it. Never change your raw data!

Comment like someone you've never met will read your code - it'll help you when you come back to it later!



References/Resources
========================================================

Tidy Data & Data Wrangling:
- http://vita.had.co.nz/papers/tidy-data.pdf
- http://blog.rstudio.org/2014/07/22/introducing-tidyr/
- https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf
- http://www.stat.purdue.edu/~varao/STAT598Z/lect13.pdf
- Vignettes at cran.r-project.org: dplyr & tidyr

For making this presentation (R Presentation / Rstudio):  
- https://support.rstudio.com/hc/en-us/articles/200486468-Authoring-R-Presentations)



Thanks!
========================================================

Anyone interested in future talk/demo on the {dplyr} package?  

Would probably focus more on 5 core manipulation functions:

- select( ) - subset data easily
- filter( ) - filter rows by groupings
- arrange( ) - reorder and arrange rows by sorting
- mutate( ) - add new columns as functions of others
- summarise( ) - collapse dataframe to single row

- distinct( ) - not a core manipulation function, but really useful. Similar to unique( ) in base R
