Chapter 5: Control flow
================
Erika Duan
2021-01-04

  - [Chapter goals](#chapter-goals)
  - [Choices](#choices)
      - [Using `if` statements](#using-if-statements)
          - [Handling invalid inputs to `if`
            statements](#handling-invalid-inputs-to-if-statements)
          - [Handling a vector of logical values as an
            input](#handling-a-vector-of-logical-values-as-an-input)
      - [Using `switch()` calls](#using-switch-calls)
  - [Loops](#loops)
      - [Common pitfalls of using `for`
        loops](#common-pitfalls-of-using-for-loops)
      - [Alternatives to `for` loops](#alternatives-to-for-loops)

``` r
#-----load R libraries-----   
if (!require(pacman)) install.packages("pacman")
p_load(tidyverse)  
```

# Chapter goals

Understanding what **control flow** is helps you to:

  - Understand how to control code flow using two different types of
    tools - choices and loops.  
  - Understand how `if` statements and `switch()` calls operate in R.  
  - Understand how `for` loops and `while` loops operate in R.

# Choices

## Using `if` statements

Choices comprise `if` statements and `switch()` calls and these tools
alter code flow by allowing you to run different code depending on the
input.

The basic form of an `if` statement is:

  - `if (condition) {true_action}`  
  - `if (condition) {true_action} else {false_action}`

In the first form, if the condition is `TRUE`, `true_action` is
evaluated.  
In the second form, if the condition is `TRUE`, `true_action` is
evaluated and if the condition is `FALSE`, `false_action` is evaluated.

In reality, choices usually comprise compound `ifelse` statements
containing multiple conditions.

``` r
#-----coding best practice for writing compound if statements-----  
student_grade <- function(score) {
  if (score >= 90) {
    "A"
  } else if (score >= 80) {
    "B"
  } else if (score >= 50) {
    "C"
  } else {
    "F"
  }
}

grades <- c(92, 45, 75, 80)  

vapply(grades, student_grade, FUN.VALUE = "chr") 
#> [1] "A" "F" "C" "B"  
```

**Note:** An excellent tutorial on the differences between `apply()`,
`lapply()` and `sapply()` can be found
[here](https://www.guru99.com/r-apply-sapply-tapply.html). A great
explanation of `vapply` usage can also be found
[here](https://stackoverflow.com/questions/51657605/fun-value-argument-in-vapply).

Since an `if` statement returns a value, its result can be assigned.
However, assignment is only recommended if the `if` statement fits on a
single line for readability.

``` r
#-----you can assign the ouput of single line if statements-----  
# the global evaluation of if (TRUE) is always TRUE   

if (TRUE) {print(1)}
#> [1] 1

if (TRUE) print(1)  
#> [1] 1 

# {print(1)} and print(1) and 1 return the same output 
# we do not need to use {} if the statement fits on a single line  

x1 <- if (TRUE) print("eval TRUE") else print("eval FALSE")     
x2 <- if (FALSE) print("eval FALSE") else print("eval TRUE")     

c(x1, x2)    
#> [1] "eval TRUE" "eval TRUE"      
```

When you evaluate a single `if` statement that does not contain an
`else` condition, your `if` statement invisibly returns `NULL` if the
condition is `FALSE`. As functions like `c()` and `paste()` invisibly
drop `NULL` inputs, this allows for compact outputs.

``` r
#-----c() and paste() invisibly drops NULL inputs-----   
c("id1", NULL, "id3", NULL, "id5")     
#> [1] "id1" "id3" "id5"              

paste0("Hello ", NULL, "world", NULL, "!", NULL)    
#> [1] "Hello world!"      
```

``` r
#-----compact outputs generated by single if arguments----- 
greet_person <- function(person, birthday = FALSE) {
  paste0(
    "Hi ", person,
    if (birthday == TRUE) " and HAPPY BIRTHDAY!"  
  )
}   

greet_person("Erika", birthday = FALSE)
#> [1] "Hi Erika"     

greet_person("Dan", birthday = TRUE)   
#> [1] "Hi Dan and HAPPY BIRTHDAY!"     
```

### Handling invalid inputs to `if` statements

Since a single `if` statement should always evaluate to a single `TRUE`
or `FALSE` (a single logical type), most other inputs will generate an
error.

``` r
#-----invalid inputs to if statements-----  
if (TRUE) print(1)     
#> [1] 1      

if (1) print(1) 
#> [1] 1  

# if (x) print(1)  
#> Error: object 'x' not found

x <- 5
if (x) print(1)
#> [1] 1  

# if("x") print(1)
#> Error in if ("x") { : argument is not interpretable as logical   
```

An exception to this occurs when a logical vector of length greater than
1 is the output. In R, the first element of the logical vector is then
used alongside a warning message. It is recommmended to suppress this
behaviour by setting the environment variable
`Sys.setenv("_R_CHECK_LENGTH_1_CONDITION_" = "true")`, as you might
accidentally miss this warning.

``` r
#-----be careful of length(logical vector) > 1 outputs-----   
if (c(TRUE, FALSE, TRUE)) print(1)
#> ## Warning in if (c(TRUE, FALSE, TRUE)) print(1): the condition has length > 1    
#> ## and only the first element will be used    
#> [1] 1    
```

``` r
#-----recommended practice to prevent length(logical vector) > 1 output evaluation-----
Sys.setenv("_R_CHECK_LENGTH_1_CONDITION_" = "true")  

# if (c(TRUE, FALSE, TRUE)) print(1)
#> Error in if (c(TRUE, FALSE, TRUE)) print(1) : 
#>   the condition has length > 1
```

### Handling a vector of logical values as an input

When your input is a vector of logical values i.e. when you would like
to evaluate a series of elements within a vector, `ifelse()` or
`case_when` can be used to handle the input. The function `ifelse()` is
a vectorised function containing `test`, `yes` and `no` vectors which
can be recycled to the same length.

From R documentation on `iselse(test, yes, no)`:

> ifelse returns a value with the same shape as test which is filled
> with elements selected from either yes or no depending on whether the
> element of test is TRUE or FALSE.

``` r
#-----ifelse() handles a vector of logical values-----  
x <- 1:6

ifelse(x %% 3 == 0, "divisible", as.character(x))
#> [1] "1"         "2"         "divisible" "4"         "5"         "divisible"  

# yes vector output and no vector output are both character type    
```

**Note:** The usage of `ifelse()` is only recommended for scenarios
where `yes` and `no` outputs are the same type, as it can be hard to
predict the output type of complex conditions.

The function `case_when()` uses a special syntax to allow any number of
condition-vector pairs i.e. the function is not limited to just
outputting `yes` and `no` vectors. A consequence is that we need to
order `case_when()` condition-vector pairs so that the most restricted
conditions are first evaluated.

``` r
#-----case_when() handles a vector of logical values-----  
y <- 1:6  

case_when(
  y %% 6 == 0 ~ "div by 6",
  y %% 4 == 0 ~ "div by 4", 
  y %% 3 == 0 ~ "div by 3",
  TRUE ~ as.character(y) # the remaining elements all evaluated to TRUE    
)  
#> [1] "1"        "2"        "div by 3" "div by 4" "5"        "div by 6"   

# the first TRUE condition that applies for each element is outputted         
```

## Using `switch()` calls

The function `switch()` can be used as an alternative to `ifelse()` as
it evaluates an expression and accordingly chooses one of the further
arguments. Using `switch()` instead of `ifelse()` can improve the
readability of compound `ifelse` statements.

``` r
#-----using ifelse() to write compound statements-----   
clean_x <- function(x) {
  if (x == "a") {
    "option 1"
  } else if (x == "b") {
    "option 2"
  } else if (x == "c") {
    "option 3"  
  } else {
    stop("Invalid x value")   
  }
}  

clean_x("a")
#> [1] "option 1"  

# clean_x("d")
#> Error in clean_x("d") : Invalid x value
```

``` r
#-----using switch() to write compound statements-----     
cleaner_x <- function(x) {
  switch (x,
    a = "option 1",
    b = "option 2",
    c = "option 3",
    stop("Invalid x value")
  )
}

cleaner_x("a")
#> [1] "option 1"  

# cleaner_x("d")   
#> Error in cleaner_x("d") : Invalid x value   
```

**Note:** The last component of `switch()` should always throw an error
i.e. force `stop()` and print an error message. Otherwise, unmatched
inputs from `switch()` will invisibly return `NULL`.

``` r
#-----when the last component of switch() does not throw an error message-----  
cleaner_x_e <- function(x) {
  switch (x,
          a = "option 1",
          b = "option 2",
          c = "option 3"
  )
}

cleaner_x_e("d")

# no output is created for unmatched inputs as NULL is returned invisibly  
# this is an undesirable behaviour  
```

When multiple inputs share the same output, `switch()` allows us to use
a shorthand that mimics the behaviour of C’s `switch` statement.

``` r
#-----when multiple inputs have the same output-----  
count_legs <- function(x) {
  switch (x,
          cow = ,
          horse = ,
          sheep = ,
          dog = 4 ,
          human = , 
          bird = , 
          stop("Unknown input entered")
  )
}

count_legs("cow")
#> [1] 4     

count_legs("dog")  
#> [1] 4     

# leaving the right hand side of = empty allows the input to fall through to the next value  
```

**Note:** The recommendation is to use `switch()` only when you have
character inputs, as numeric inputs have unpredictable failure modes.

``` r
#-----exercise 5.2.4.1-----   
class(ifelse(TRUE, 1, "no")) 
#> [1] "numeric"  

class(ifelse(FALSE, 1, "no"))
#> [1] "character"  

ifelse(NA, 1, "no")
#> [1] NA  

# when ifelse(test = NA, yes, no), NA is returned  

class(ifelse(NA, 1, "no"))
#> [1] "logical"  

# NA is a special logical type which denotes missingness    

#-----exercise 5.2.4.2-----   
x <- 1:10

length(x)
#> [1] 10  

# length(x) can be evaluated and all non-zero outputs are interpreted as TRUE       

if (length(x)) "not empty" else "empty"
#> [1] "not empty"

y <- numeric()

length(y)
#> [1] 0

if (length(y)) "not empty" else "empty"
#> [1] "empty"  

# length(y) can be evaluated and outputs 0 which is interpreted as FALSE       
```

# Loops

In R, `for` loops are used to iterate over items in a vector.

The basic form of a `for` loop is:

  - `for (item in vector) perform_action`
  - For each item in the vector, `perform_action` is called once and the
    value of `item` is updated.

The operation `for` assigns the `item` to the current environment,
overwriting any existing variables with the same name.

``` r
#-----for overrides existing variable names-----  
i <- 100 

for (i in 1:3) {
  print(i + 2)
}

#> [1] 3
#> [1] 4
#> [1] 5
```

There are two ways to terminate a `for` loop early:

  - Using `next` exits the current iteration.  
  - USing `break` exits the entire `for` loop.

<!-- end list -->

``` r
#-----using next to exit a for loop-----
for (i in 1:5) {
  if (i %% 2 == 0)
    next
  print(paste(i, "is not divisible by 2"))
}
#> [1] "1 is not divisible by 2"
#> [1] "3 is not divisible by 2"
#> [1] "5 is not divisible by 2"

#-----using break to exit the entire loop-----  
for (i in 1:5) {
  print(paste(i, "is less than 4"))
  if (i > 3) 
    break
}
#> [1] "1 is less than 4"
#> [1] "2 is less than 4"
#> [1] "3 is less than 4"
#> [1] "4 is less than 4"  

# when i = 4, the for loop is first actioned and then the vector exits the for loop 
```

## Common pitfalls of using `for` loops

There are 3 common pitfalls to using `for` loops in R:

  - When generating data, make sure to preallocate the output container
    using the function `vector()` or your output will be extremely
    slow.  
  - Avoid iterating over `1:length(x)` as the `for` loop will fail in
    unhelpful ways if x has length 0.  
  - Always call single inputs and assign single outputs using `[[i]]`
    inside for loops.

<!-- end list -->

``` r
#-----preallocate output container when generating data-----  
query <- c(1, 4, 16, 25) 

output <- vector("double", length = length(query)) 

# use vector() to choose atomic vectors or lists as the efficient output type  

for (i in seq_along(query)){
  output[[i]] <- sqrt(query[[i]])  
}
output
#> [1] 1 2 4 5  
```

``` r
#-----avoid using length() as it outputs an error for length(input) is 0-----
query <- c()

1:length(query)
#> [1] 1 0

#> output <- vector("double", length = length(query)) 
#> for (i in length(query)){
#>   output[[i]] <- sqrt(query[[i]])  
#> }
#> Error in sqrt(query[[i]]) : non-numeric argument to mathematical function  

# this happens as 1:length(query) outputs a nonsensical task sequence    

#-----use seq_along() instead of length()-----  
#> 1:seq_along(query)
#> Error in 1:seq_along(query) : argument of length 0

output <- vector("double", length = length(query)) 
for (i in seq_along(query)){
  output[[i]] <- sqrt(query[[i]])  
}
output
#> numeric(0)  
```

``` r
#-----for loops will strip S3 vector attributes-----  
dates <- as.Date(c("2020-01-01", "2010-01-01"))

for (i in dates) {
  print(i)
}
#> [1] 18262
#> [1] 14610

# remember to assign single outputs using [[i]] to preserve S3 vector attributes  

for (i in seq_along(dates)) {
  print(dates[[i]])
}
#> [1] "2020-01-01"
#> [1] "2010-01-01"
```

**Note:** More examples of writing efficient `for` loops can be found in
the [chapter on iteration](https://r4ds.had.co.nz/iteration.html) in the
R for Data Science textbook.

## Alternatives to `for` loops

In R, `for` loops are useful when you already know the set of elements
that you want to iterate over.

You can use two alternative iterative approaches when you do not know
the set of elements in advance:

  - `while(condition) perform_action` - this calls `perform_action`
    while `condition` is `TRUE`.  
  - `repeat(action)` - this repeats an action forever, until it
    encounters a `break`.

You can rewrite any `for` loop to use `while` instead. You can rewrite
any `while` loop to use `repeat` instead. Although the commands `repeat`
and `while` are more flexible than a `for` loop, it is best practice to
use the least flexible solution when performing iterations.

**Note:** In general, there should be no need to write your own `for`
loops as the existing functions `purrr::map()` and `apply()` should be
sufficient for most problems.

``` r
#-----rewrite for loop into while loop-----  
# for loop version  
x <- c(1:5)  
output <- vector("numeric", length(x))

for (i in seq_along(x)) {
output[[i]] <- x[[i]] ^ 2  
}
output
#> [1]  1  4  9 16 25

# while loop version  

x <- c(1:5)  
i <- 1
output <- vector("numeric", length(x))

while (i <= length(x)) {
  output[[i]] = x[[i]] ^ 2  
  i = i + 1
}
output
#> [1]  1  4  9 16 25      
```

``` r
#-----exercise 5.3.3.1-----  
x <- numeric()
out <- vector("list", length(x))
for (i in 1:length(x)) {
  out[i] <- x[i] ^ 2
}
out

x[1] ^ 2
#> [1] NA     

x[0] ^ 2
#> numeric(0)     

# output of length(x) is 0  
# first iteration - x[1] will generate NA 
# second iteration - x[0] returns numeric(0)

#-----exercise 5.3.3.2-----  
xs <- c(1, 2, 3)
for (x in xs) {
  xs <- c(xs, x * 2)
}
xs
#> [1] 1 2 3 2 4 6

# this should read as an infinite loop as it has the potential to keep growing    
# to prevent this, the loop appends x * 2 onto the pre-existing vector just once for x = c(1, 2, 3)   

#-----exercise 5.3.3.3-----   
for (i in 1:3) {
  i <- i * 2
  print(i) 
}  
#> [1] 2
#> [1] 4
#> [1] 6

# this is outputting the value for each original item in the original index      
# the original index is used for the for loop 
# reassigning the index does not affect the following iterations  
```