# purrr Usage Summary 
---

## 1. Basics (Iteration, Subsetting, Flavors of map, Complex Iterations)

| Topic | Function | Description | Example |
|-------|----------|-------------|---------|
| **Simplifying iteration** | `map(.x, .f)` | Apply `.f` to each element of `.x`, returns a list | `map(1:5, ~ .x^2)` → `list(1,4,9,16,25)` |
| | `map_dbl()` | Returns numeric vector | `map_dbl(1:5, ~ .x^2)` → `c(1,4,9,16,25)` |
| | `map_chr()` | Returns character vector | `map_chr(letters[1:3], toupper)` → `"A" "B" "C"` |
| | `map_lgl()` | Returns logical vector | `map_lgl(1:5, ~ .x > 3)` → `FALSE FALSE FALSE TRUE TRUE` |
| | `map_int()` | Returns integer vector | `map_int(c(1.2, 2.8, 3.5), floor)` → `1 2 3` |
| **Subsetting lists** | `[[ ]]` | Subset list elements by index or name | `L[[1]]`, `L[["df"]]` |
| | `map()` | Apply function to each list element | `map_dbl(survey_data, nrow)` |
| **Flavors of map()** | `map_dbl()` | Numeric output | `map_dbl(survey_data, nrow)` |
| | `map_lgl()` | Logical output | `map_lgl(survey_data, ~ nrow(.x) == 14)` |
| | `map_chr()` | Character output | `map_chr(survey_data, rownames)` |
| | `map_df()` | Bind results into tibble | `map_df(bird_measurements, ~ data.frame(weight=.x$weight, wing=.x$wing_length))` |
| **Complex iterations** | `map2(.x, .y, .f)` | Iterate over two inputs | `map2(1:3, 4:6, ~ .x + .y)` → `list(5,7,9)` |
| | `pmap(list_of_args, .f)` | Iterate over >2 inputs | `pmap(list(mean=c(5,10), sd=c(1,2), n=c(3,3)), ~ rnorm(n, mean, sd))` |
| | `%>%` (pipe) | Chain operations together | `survey_data %>% map(nrow) %>% map_dbl(identity)` |

---

## 2. Troubleshooting

```r
# --- Troubleshooting with purrr ---
# Requires purrr; load tidyverse or purrr
# library(tidyverse); or:
library(purrr)

# safely(): wrap a function so iteration continues even if some elements fail
safe_log <- safely(log)
map(list(1, 10, "oops"), safe_log)
# [[1]]$result = 0, $error = NULL
# [[2]]$result = 2.302585, $error = NULL
# [[3]]$result = NULL, $error = <error message>

# Use transpose() to separate results and errors
res <- map(list(1, 10, "oops"), safe_log) %>% transpose()
res$result
res$error

# possibly(): return result or a default value
safe_log2 <- possibly(log, otherwise = NA_real_)
map_dbl(list(1, 10, "oops"), safe_log2)
# 0 2.302585 NA

# compact(): remove NULL values from a list
list(1, NULL, 2, NULL) %>% compact()
# list(1, 2)
```

---

## 3. walk(): Side Effects

```r
# --- walk(): side effects ---
library(purrr)
library(ggplot2)

# walk() is like map(), but returns nothing.
# Use for side effects: printing, saving files, plotting.
walk(1:3, ~ print(.x * 2))
# prints 2, 4, 6

# Example with plots
plots <- list(
  ggplot(mtcars, aes(mpg, wt)) + geom_point(),
  ggplot(mtcars, aes(hp, wt)) + geom_point()
)
walk(plots, print)   # prints both plots (useful in RMarkdown)
```

---

## 4. Functional Programming Tools

```r
# --- Functional Programming Tools ---
library(purrr)

# Anonymous function with ~
map(1:5, ~ .x + 100)

# keep() / discard()
keep(1:10, ~ .x > 5)      # 6 7 8 9 10
discard(1:10, ~ .x > 5)   # 1 2 3 4 5

# every() / some()
every(1:5, is.numeric)    # TRUE
some(1:5, ~ .x > 3)       # TRUE

# detect() / detect_index()
detect(c(2,4,7,9), ~ .x > 5)        # 7
detect_index(c(2,4,7,9), ~ .x > 5)  # 3

# compose(): combine functions
f <- compose(log, sum)   # equivalent to log(sum(.))
f(1:10)                  # log(55)

# negate(): invert a predicate
not_na <- negate(is.na)
map_lgl(c(1, NA, 2), not_na)  # TRUE FALSE TRUE
```

---

## 5. List Columns

```r
# --- List Columns ---
library(dplyr)
library(tidyr)
library(purrr)

# A tibble column where each cell is a list.
# Useful for nested data (data frames, models, etc.)

# nest() + mutate() + map()
iris %>%
  group_by(Species) %>%
  nest() %>%
  mutate(model = map(data, ~ lm(Sepal.Length ~ Sepal.Width, data = .x)))

# unnest()
iris %>%
  group_by(Species) %>%
  nest() %>%
  mutate(n_rows = map_int(data, nrow)) %>%
  unnest(cols = data)
```

---

## 6. Graphing with purrr

```r
# --- Graphing with purrr ---
library(purrr)
library(ggplot2)

# ggplot() requires a data frame.
# Use map_df() to combine list elements into one data frame.

# Example list 'bird_measurements' is assumed: each element has $weight and $wing_length
# bird_measurements <- list(
#   robin = list(weight = c(16, 17, 16.5), wing_length = c(22, 23, 21.5)),
#   sora  = list(weight = c(70, 72, 68.5), wing_length = c(35, 36, 34.5))
# )

bird_measurements %>%
  map_df(~ data.frame(weight = .x$weight, wing = .x$wing_length)) %>%
  ggplot(aes(x = weight, y = wing)) +
  geom_point()
```

---

## Bonus: All-in-One (Single Block Copy)

```r
# --- All-in-One purrr Notes (Sections 1–6 in one block) ---

# Load pkgs
suppressPackageStartupMessages({
  library(purrr); library(dplyr); library(tidyr); library(ggplot2)
})

# 1) Basics --------------------------------------------------------------
# map() family
map(1:5, ~ .x^2); map_dbl(1:5, ~ .x^2); map_chr(letters[1:3], toupper)
map_lgl(1:5, ~ .x > 3); map_int(c(1.2, 2.8, 3.5), floor)

# Subsetting lists
L <- list(a = 1:3, b = data.frame(x=1:2))
L[[1]]; L[["b"]]; map_dbl(L, length)

# map_df()
map_df(list(mtcars, iris), ~ data.frame(rows=nrow(.x)))

# map2 / pmap
map2(1:3, 4:6, ~ .x + .y)
pmap(list(mean=c(5,10), sd=c(1,2), n=c(3,3)), ~ rnorm(n, mean, sd))

# Pipe
mtcars %>% map(nrow) %>% map_dbl(identity)

# 2) Troubleshooting -----------------------------------------------------
safe_log <- safely(log)
res_raw <- map(list(1, 10, "oops"), safe_log)
res <- res_raw %>% transpose()
res$result; res$error

safe_log2 <- possibly(log, otherwise = NA_real_)
map_dbl(list(1, 10, "oops"), safe_log2)
compact(list(1, NULL, 2, NULL))

# 3) walk() --------------------------------------------------------------
walk(1:3, ~ print(.x * 2))
plots <- list(
  ggplot(mtcars, aes(mpg, wt)) + geom_point(),
  ggplot(mtcars, aes(hp, wt)) + geom_point()
)
walk(plots, print)

# 4) FP Tools ------------------------------------------------------------
map(1:5, ~ .x + 100)
keep(1:10, ~ .x > 5); discard(1:10, ~ .x > 5)
every(1:5, is.numeric); some(1:5, ~ .x > 3)
detect(c(2,4,7,9), ~ .x > 5); detect_index(c(2,4,7,9), ~ .x > 5)
f <- compose(log, sum); f(1:10)
not_na <- negate(is.na); map_lgl(c(1, NA, 2), not_na)

# 5) List Columns --------------------------------------------------------
iris %>% group_by(Species) %>% nest() %>%
  mutate(model = map(data, ~ lm(Sepal.Length ~ Sepal.Width, data = .x))) -> iris_nested
iris_nested %>%
  mutate(n_rows = map_int(data, nrow)) %>%
  unnest(cols = data) -> iris_unnested

# 6) Graphing ------------------------------------------------------------
# bird_measurements %>%
#   map_df(~ data.frame(weight = .x$weight, wing = .x$wing_length)) %>%
#   ggplot(aes(x = weight, y = wing)) + geom_point()
```

