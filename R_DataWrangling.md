Data Wrangling in R
========================================================
author: Flic Anderson
date: 29 September 2015
font-import: http://fonts.googleapis.com/css?family=Telex
font-family: 'Telex'

RBGE Code Group 


What IS Data Wrangling?!
========================================================

For more details on authoring R presentations click the
**Help** button on the toolbar.

- Bullet 1
- Bullet 2
- Bullet 3

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

Slide With Plot
========================================================

![plot of chunk unnamed-chunk-2](R_DataWrangling-figure/unnamed-chunk-2-1.png) 
   
References/Resources
========================================================

- https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf
- other things
