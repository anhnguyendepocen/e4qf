
# Functions

## Introduction

- `&&`, `||`, `any`, `all`, `identical`, `dplyr::near`
- `switch`, `cut`
- `stop`, `stopifnot`
- `...`


## When should you write a function?


```r
library("lubridate")
#> Loading required package: methods
#> 
#> Attaching package: 'lubridate'
#> The following object is masked from 'package:base':
#> 
#>     date
```


### Practice

**Ex 1.** Why is TRUE not a parameter to `rescale01()`? What would happen if `x` contained a single missing value, and `na.rm` was `FALSE`?

**Note** By a single missing value, this means that `x` has at least one `NA` value.

If there were any `NA` values, and `na.rm = FALSE`, then the function would 
return `NA`.

I can confirm this by testing a function that allows for `na.rm` as an argument:

```r
rescale01_alt <- function(x, finite = TRUE) {
  rng <- range(x, na.rm = finite, finite = finite)
  (x - rng[1]) / (rng[2] - rng[1])
}
rescale01_alt(c(NA, 1:5), finite = FALSE)
#> [1] NA NA NA NA NA NA
rescale01_alt(c(NA, 1:5), finite = TRUE)
#> [1]   NA 0.00 0.25 0.50 0.75 1.00
```

**Ex 2.** In the second variant of `rescale01()`, infinite values are left unchanged. Rewrite `rescale01()` so that `-Inf` is mapped to 0, and `Inf` is mapped to 1.


```r
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE, finite = TRUE)
  y <- (x - rng[1]) / (rng[2] - rng[1])
  y[y == -Inf] <- 0
  y[y == Inf] <- 1
  y
}

rescale01(c(Inf, -Inf, 0:5, NA))
#> [1] 1.0 0.0 0.0 0.2 0.4 0.6 0.8 1.0  NA
```

**Ex 3.** Practice turning the following code snippets into functions. Think about what each function does. What would you call it? How many arguments does it need? Can you rewrite it to be more expressive or less duplicative?


```r
mean(is.na(x))

x / sum(x, na.rm = TRUE)

sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
```


This function calculates the proportion of `NA` values in a vector:

```r
prop_na <- function(x) {
  mean(is.na(x))
}
prop_na(c(NA, 0, NA, 0, NA))
#> [1] 0.6
```

This function standardizes a function to its weight. If all elements of `x` are non-negative, this will ensure the vector sums to 1. 

```r
weights <- function(x) {
  x / sum(x, na.rm = TRUE)
}
y <- weights(0:5)
y
#> [1] 0.0000 0.0667 0.1333 0.2000 0.2667 0.3333
sum(y)
#> [1] 1
```

This function calculates the [coefficient of variation](https://en.wikipedia.org/wiki/Coefficient_of_variation) (assuming that `x` can only take non-negative values). 
The coefficient of variation is the standard deviation devided by the mean:

```r
coef_variation <- function(x) {
  sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
}
coef_variation(runif(10))
#> [1] 0.672
```

**Ex 4.** Follow http://nicercode.github.io/intro/writing-functions.html to write your own functions to compute the variance and skew of a numeric vecto

**Note** The math in https://nicercode.github.io/intro/writing-functions.html seems not to be rendering, but I'll write functions for the variance and skewness.

The sample variance is defined as 
$$
Var(x) = \frac{1}{n - 1} \sum_{i=1}^n (x_i - \bar{x}) ^2
$$
where the sample mean is $\bar{x} = (\sum x_i) / n$.

```r
variance <- function(x) {
  # remove missing values
  x <- x[!is.na(x)]
  n <- length(x)
  m <- mean(x)
  sq_err <- (x - m) ^ 2
  sum(sq_err) / (n - 1)
}
var(1:10)
#> [1] 9.17
variance(1:10)
#> [1] 9.17
```

There are multiple definitions of [skewness](https://en.wikipedia.org/wiki/Skewness), but I'll use the method of moments estimator of the population skewness,
$$
b_1 =  \frac{m_3}{s^3} = \frac{\frac{1}{n} \sum (x_i - \bar{x}) ^ 3}{{\left(\frac{1}{n - 1} \sum (x_i - \bar{x}) ^ 2\right)} ^ \frac{3}{2}}
$$

```r
skewness <- function(x) {
  x <- x[!is.na(x)] 
  n <- length(x)
  m <- mean(x)
  m3 <- sum((x - m) ^ 3) / n
  s3 <- sqrt(sum((x - m) ^ 2) / (n - 1))
  m3 / s3
}
skewness(rgamma(10, 1, 1))
#> [1] 1.56
```

5. Write `both_na()`, a function that takes two vectors of the same length and returns the number of positions that have an `NA` in both vectors.


```r
both_na <- function(x, y) {
  sum(is.na(x) & is.na(y))
}
both_na(c(NA, NA,  1, 2),
        c(NA,  1, NA, 2))
#> [1] 1
both_na(c(NA, NA,  1, 2, NA, NA, 1), 
        c(NA,  1, NA, 2, NA, NA, 1))
#> [1] 3
```

**Ex. 6** What do the following functions do? Why are they useful even though they are so short?


```r
is_directory <- function(x) file.info(x)$isdir
is_readable <- function(x) file.access(x, 4) == 0
```

The function `is_directory` checks whether the path in `x` is a directory.
The function `is_readable` checks whether the path in `x` is readable, meaning that the file exists and the user has permission to open it.
These functions are useful even though they are short because their names make it much clearer what the code is doing.

**Ex 7.** Read the complete lyrics to “Little Bunny Foo Foo”. There’s a lot of duplication in this song. Extend the initial piping example to recreate the complete song, and use functions to reduce the duplication.

This could probably be done cleaner, but here's one version.

```r
threat <- function(chances) {
  give_chances(from = Good_Fairy,
               to = foo_foo,
               number = chances,
               condition = "Don't behave",
               consequence = turn_into_goon)  
}
  
lyric <- function() {
  foo_foo %>%
    hop(through = forest) %>%
    scoop(up = field_mouse) %>%
    bop(on = head)
  
  down_came(Good_Fairy)
  said(Good_Fairy, 
      c("Little bunny Foo Foo",
        "I don't want to see you",
        "Scooping up the field mice"
        "And bopping them on the head.")
}

lyric()
threat(3)
lyric()
threat(2)
lyric()
threat(1)
lyric()
turn_into_goon(Good_Fairy, foo_foo)
             
```


## Functions are for humans and computers

> There are only two hard problems in Computer Science: cache invalidation and naming things. - Phil Karlton (as quoted by many places online but no original source.)

See this Quora discussion: [Why is naming things hard in computer science, and how can it can be made easier?](https://www.quora.com/Why-is-naming-things-hard-in-computer-science-and-how-can-it-can-be-made-easier).

### Exercises

**Ex 1.** Read the source code for each of the following three functions, puzzle out what they do, and then brainstorm better names.


```r
f1 <- function(string, prefix) {
  substr(string, 1, nchar(prefix)) == prefix
}

f2 <- function(x) {
  if (length(x) <= 1) return(NULL)
  x[-length(x)]
}

f3 <- function(x, y) {
  rep(y, length.out = length(x))
}
```

The function `f1` returns whether a function has a common prefix.

```r
f1(c("str_c", "str_foo", "abc"), "str_")
#> [1]  TRUE  TRUE FALSE
```
A better name for `f1` is `has_prefix()`

The function `f2` drops the last element:

```r
f2(1:3)
#> [1] 1 2
f2(1:2)
#> [1] 1
f2(1)
#> NULL
```
A better name for `f2` is `drop_last()`.

The function `f3` repeats `y` once for each element of `x`.

```r
f3(1:3, 4)
#> [1] 4 4 4
```
This is a harder one to name. I would say something like `recycle` (R's name for this behavior), or `epxand`.

**Ex. 2** Take a function that you’ve written recently and spend 5 minutes brainstorming a better name for it and its arguments.

*Don't worry about it*

**Ex. 3** Compare and contrast `rnorm()` and `MASS::mvrnorm()`. How could you make them more consistent?

*You can ignore*

`rnorm` samples from the univariate normal distribution, while `MASS::mvrnorm` samples from the multivariate normal distribution.
The main arguments in `rnorm` are `n`, `mean`, `sd`.
The main arguments is `MASS::mvrnorm` are `n`, `mu`, `Sigma`. 
To be consistent they should have the same names.
However, this is difficult. 
In general, it is better to be consistent with more widely used functions, e.g. `rmvnorm` should follow the conventions of `rnorm`. 
However, while `mean` is correct in the multivariate case, `sd` does not make sense in the multivariate case. 
Both functions an internally consistent though; it would be bad to have `mu` and `sd` or `mean` and `Sigma`.

**Ex. 4**  Make a case for why `norm_r()`, `norm_d()` etc would be better than `rnorm()`, `dnorm()`. Make a case for the opposite.

If named `norm_r` and `norm_d`, it groups the family of functions related to the normal distribution.
If named `rnorm`, and `dnorm`, functions related to are grouped into families by the action they perform. `r*` functions always sample from distributions: `rnorm`, `rbinom`, `runif`, `rexp`. `d*` functions calculate the probability density or mass of a distribution: `dnorm`, `dbinom`, `dunif`, `dexp`.


## Conditional execution

### Exercises

**Ex. 1** What’s the difference between `if` and `ifelse()`? Carefully read the help and construct three examples that illustrate the key differences.

The keyword `if` tests a single condition, while `ifelse` tests each element.


**Ex. 2** Write a greeting function that says “good morning”, “good afternoon”, or “good evening”, depending on the time of day. (Hint: use a time argument that defaults to `lubridate::now()`. That will make it easier to test your function.)


```r
greet <- function(time = lubridate::now()) {
  hr <- hour(time)
  # I don't know what to do about times after midnight, 
  # are they evening or morning?
  if (hr < 12) {
    print("good morning")
  } else if (hr < 17) {
    print("good afternoon")
  } else {
    print("good evening")
  }
} 
greet()
#> [1] "good evening"
greet(ymd_h("2017-01-08:05"))
#> [1] "good morning"
greet(ymd_h("2017-01-08:13"))
#> [1] "good afternoon"
greet(ymd_h("2017-01-08:20"))
#> [1] "good evening"
```

**Ex. 3** Implement a `fizzbuzz` function. It takes a single number as input. If the number is divisible by three, it returns “fizz”. If it’s divisible by five it returns “buzz”. If it’s divisible by three and five, it returns “fizzbuzz”. Otherwise, it returns the number. Make sure you first write working code before you create the function.


```r
fizzbuzz <- function(x) {
  stopifnot(length(x) == 1)
  stopifnot(is.numeric(x))
  # this could be made more efficient by minimizing the
  # number of tests
  if (!(x %% 3) & !(x %% 5)) {
    print("fizzbuzz")
  } else if (!(x %% 3)) {
    print("fizz")
  } else if (!(x %% 5)) {
    print("buzz")
  }
}
fizzbuzz(6)
#> [1] "fizz"
fizzbuzz(10)
#> [1] "buzz"
fizzbuzz(15)
#> [1] "fizzbuzz"
fizzbuzz(2)
```

**Ex. 4** How could you use `cut()` to simplify this set of nested if-else statements?


```r
if (temp <= 0) {
  "freezing"
} else if (temp <= 10) {
  "cold"
} else if (temp <= 20) {
  "cool"
} else if (temp <= 30) {
  "warm"
} else {
  "hot"
}
```
How would you change the call to `cut()` if I’d used `<` instead of `<=`? What is the other chief advantage of cut() for this problem? (Hint: what happens if you have many values in temp?)


```r
temp <- seq(-10, 50, by = 5)
cut(temp, c(-Inf, 0, 10, 20, 30, Inf), right = TRUE,
    labels = c("freezing", "cold", "cool", "warm", "hot"))
#>  [1] freezing freezing freezing cold     cold     cool     cool    
#>  [8] warm     warm     hot      hot      hot      hot     
#> Levels: freezing cold cool warm hot
```

To have intervals open on the left (using `<`), I change the argument to `right = FALSE`:

```r
temp <- seq(-10, 50, by = 5)
cut(temp, c(-Inf, 0, 10, 20, 30, Inf), right = FALSE,
    labels = c("freezing", "cold", "cool", "warm", "hot"))
#>  [1] freezing freezing cold     cold     cool     cool     warm    
#>  [8] warm     hot      hot      hot      hot      hot     
#> Levels: freezing cold cool warm hot
```

Two advantages of using `cut` is that it works on vectors, whereas `if` only works on a single value (I already demonstrated this above),
and that to change comparisons I only needed to change the argument to `right`, but I would have had to change four operators in the `if` expression.

**Ex. 5** What happens if you use `switch()` with numeric values?

It selects that number argument from `...`.


```r
switch(2, "one", "two", "three")
#> [1] "two"
```


**Ex 6.** 
What does this `switch()` call do? What happens if `x` is `“e”`?

It will return the `"ab"` for `a` or `b`, `"cd"` for `c` or `d`, an `NULL` for `e`. It returns the first non-missing value for the first name it matches.

```r
x <- "e"
switch(x, 
  a = ,
  b = "ab",
  c = ,
  d = "cd"
)
```
Experiment, then carefully read the documentation.


```r
switcheroo <- function(x) {
  switch(x, 
  a = ,
  b = "ab",
  c = ,
  d = "cd"
  )
}
switcheroo("a")
#> [1] "ab"
switcheroo("b")
#> [1] "ab"
switcheroo("c")
#> [1] "cd"
switcheroo("d")
#> [1] "cd"
switcheroo("e")
```

## Function arguments

### Exercises

**Ex 1.** What does `commas(letters, collapse = "-")` do? Why?


```r
commas <- function(...) stringr::str_c(..., collapse = ", ")
```

It throws an error.

```r
commas(letters, collapse = "-")
#> Error in stringr::str_c(..., collapse = ", "): formal argument "collapse" matched by multiple actual arguments
```
The argument `collapse` is passed to `str_c` as part of `...`, 
so it tries to run `str_c(letters, collapse = "-", collapse = ", ")`.


**Ex 2.** It’d be nice if you could supply multiple characters to the pad argument, e.g. `rule("Title", pad = "-+")`. Why doesn’t this currently work? How could you fix it?


```r
rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}
```


```r
rule("Important output")
#> Important output ------------------------------------------------------
rule("Important output", pad = "-+")
#> Important output -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
It does not work because it duplicates pad by the width minus the length of the string.
This is implictly assuming that pad is only one character.
I could adjust the code to calculate the length of pad.
The trickiest part is handling what to do if width is not a multiple of the number of characters of `pad`.


```r
rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  padchar <- nchar(pad)
  cat(title, " ",
      stringr::str_dup(pad, width %/% padchar),
      # if not multiple, fill in the remaining characters
      stringr::str_sub(pad, 1, width %% padchar),
      "\n", sep = "")
}
rule("Important output")
#> Important output ------------------------------------------------------
rule("Important output", pad = "-+")
#> Important output -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
rule("Important output", pad = "-+-")
#> Important output -+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+-
```

**Ex 3.** What does the `trim` argument to `mean()` do? When might you use it?

The `trim` arguments trims a fraction of observations from each end of the vector (meaning the range) before calculating the mean.
This is useful for calculating a measure of central tendancy that is robust to outliers.

**Ex 4.** The default value for the `method` argument to `cor()` is `c("pearson", "kendall", "spearman")`. What does that mean? What value is used by default?

It means that the `method` argument can take one of those three values. 
The first value, `"pearson"`, is used by default.
