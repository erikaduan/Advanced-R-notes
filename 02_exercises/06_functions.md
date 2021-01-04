Chapter 6: Functions
================
Erika Duan
2021-01-04

  - [Chapter goals](#chapter-goals)
  - [Fundamental concepts](#fundamental-concepts)

``` r
#-----load R libraries-----   
if (!require(pacman)) install.packages("pacman")
p_load(tidyverse)  
```

# Chapter goals

Understanding how **functions** operate helps you to:

  - Understand the basic three components of a function.  
  - Understand the differences behind a primitive function.  
  - Discuss the strengths and weaknesses of function composition in R.
  - Understand how R finds the value associated within a given name
    locally inside a function.  
  - Understand how arguments, including the special argument `...`,
    operate.  
  - Understand the two ways that a function can exit.  
  - Understand the ways in which R disguises ordinary function calls.

# Fundamental concepts