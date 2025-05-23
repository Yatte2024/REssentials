
# Tidy Evaluation in R – A Practical Guide

## 1. Background
## 1.1 What is Non-Standard Evaluation (NSE)?

In base R, **expressions like `filter(df, mpg > 20)` seem to "know" what `mpg` is** — even though it's not quoted, and not defined globally. This is possible due to **non-standard evaluation** (NSE).

---

#### ✅ Standard evaluation (SE)

```r
filter(mtcars, "mpg > 20")  # Need to quote, evaluate explicitly
```

#### ✅ Non-standard evaluation (NSE)

```r
filter(mtcars, mpg > 20)    # `mpg > 20` is captured as an unevaluated expression
```

In NSE:

- The **code is captured**, not evaluated immediately.
- The function (like `filter`) decides **how** and **where** to evaluate it.

This gives great flexibility, but can be **confusing to program with**. That's why **Tidy Evaluation** was introduced: to make NSE predictable and programmable.

---

#### 💡 Key difference

|               | Standard Evaluation (SE)  | Non-Standard Evaluation (NSE)   |
|---------------|----------------------------|----------------------------------|
| Input         | Actual values              | Code expressions                |
| Timing        | Immediate                  | Deferred                        |
| Usage         | `filter(mtcars, "mpg > 20")`   | `filter(mtcars, mpg > 20)`          |


### 1.2 What is Tidy Evaluation?

Tidy evaluation is a modern framework in the **tidyverse** for **non-standard evaluation (NSE)**—a way to write functions that refer to column names and expressions **without using quotes**.

This makes code both elegant and user-friendly.

#### Example:

```r
# Without tidy evaluation
filter(mtcars, mtcars$mpg > 20)

# With tidy evaluation
filter(mtcars, mpg > 20)  # cleaner!
```

---

## 2. Core Functions in Tidy Evaluation

| Function       | Description                                |
|----------------|--------------------------------------------|
| `rlang::ensym()`      | Capture a single symbol (e.g., column name) |
| `rlang::enquo()`      | Capture an expression with its environment |
| `rlang::quo()`        | Manually create a quosure                  |
| `rlang::expr()`       | Create a pure expression (no environment)  |
| `!!`           | Unquote a quosure/symbol to insert into expression |
| `!!!`          | Unquote-splice multiple expressions        |
| `rlang::syms()`       | Turn string vector into symbols list       |
| `rlang::as_name()`    | Turn symbol into string                    |
| `rlang::eval_tidy()`  | Evaluate in a tidy (data-aware) context    |

---

## 3. Symbol vs Expression Capture

### `ensym()` – for variable names

Captures a single **column name** in a function argument.

```r
select_one <- function(data, col) {
  col <- ensym(col)
  data %>% select(!!col)
}

select_one(mtcars, mpg)
```

### `enquo()` – for expressions

Captures **expressions like `x > 10`**, preserving the environment.

```r
filter_by_condition <- function(data, condition) {
  condition <- enquo(condition)
  data %>% filter(!!condition)
}

filter_by_condition(mtcars, mpg > 20)
```

---

## 4. `ensym()` vs `sym()`

Both `rlang::ensym()` and `rlang::sym()` deal with symbols in R, but they are used in different contexts.

| Feature        | `rlang::sym()`                             | `rlang::ensym()`                                  |
|----------------|--------------------------------------------|---------------------------------------------------|
| Package        | rlang                                       | rlang                                            |
| Purpose        | Convert a string to a symbol               | Capture a symbol passed as a function argument   |
| Input type     | String                                      | Bare symbol (from user input)                   |
| Usage context  | Programming context                         | User-facing function interface                   |
| Environment    | None (you provide symbol)                  | Captures from caller's environment               |

### 🔹 Example: `rlang::sym()`

#### `rlang::sym()`

```r
var <- "age"
df %>% dplyr::select(!!rlang::sym(var))  # programming with strings
```

#### `rlang::ensym()`

```r
my_select <- function(df, var) {
  var_sym <- rlang::ensym(var)
  df %>% dplyr::select(!!var_sym)
}

my_select(df, age)  # user-friendly interface
```

- `rlang::sym("age")` converts `"age"` (a string) into a symbol.
- `rlang::ensym(age)` captures the unquoted user input `age` as a symbol.


## 5. Injecting Expressions with `!!` and `!!!`

### `!!` – Unquote a single symbol/quosure

```r
col <- ensym(mpg)
filter(data, !!col > 20)
```

### `!!!` – Unquote-splice a list (useful for multiple columns)

```r
cols <- c("mpg", "hp")
select(mtcars, !!!syms(cols))
```
or using **all_of()**

```r
select(mtcars, all_of(cols))
```

---

## 6. Building Expressions: `quo()` vs `expr()`

| Function   | Captures environment? | Used for…                            | Returns     |
|------------|------------------------|--------------------------------------|-------------|
| `quo()`    | ✅ Yes                 | Capturing user input or tidyverse-compatible code | quosure     |
| `expr()`   | ❌ No                  | Programmatic code building, AST work | expression (call) |

### Why it matters: environment retention

```r
x <- 10

q <- quo(x + 1)
e <- expr(x + 1)

eval_tidy(q)  # => 11
eval(e)       # => 11 (if x is defined)
eval(e, envir = emptyenv())  # => Error: object 'x' not found
```

```r
get_env(q)  # environment where `x` is defined
get_env(e)  # empty_env()
```

- `quo()` = expression + environment
- `expr()` = just syntax (no context)

---

## 7. `eval()` vs `eval_tidy()`: Choosing the Right Tool

| Feature               | `eval()` (base R)             | `eval_tidy()` (rlang)            |
|-----------------------|-------------------------------|----------------------------------|
| Evaluates in          | A given environment           | A data-aware context (data mask) |
| Knows about data frames | ❌ Needs `df$col`             | ✅ Can refer to column names directly |
| Respects quosures?    | ❌ No                         | ✅ Yes                           |
| Typical use case      | Programmatic R expressions    | Tidyverse functions (`filter()`, `mutate()`, etc.) |

### Example:
```r
expr <- expr(mpg + 1)

eval(expr)                   # Error: object 'mpg' not found
eval_tidy(expr, data = mtcars)  # → mpg column + 1
```
**Use `eval()` when**:
- Writing low-level or base R tools
- Managing environments manually

**Use `eval_tidy()` when**:
- Evaluating expressions in data context
- Working with quosures / tidyverse
---

## 8. Common Issues

| Symptom                              | Cause                        | Fix                         |
|--------------------------------------|------------------------------|-----------------------------|
| "object not found" in `filter()`     | Using `eval()` on col name   | Use `eval_tidy()` instead   |
| Passing string column names fails    | Used raw strings in `select()` | Use `syms()` + `!!!`       |
| Expression not evaluated             | Forgot `!!`                  | Unquote with `!!`           |
| `ensym()` used for expression        | Only captures symbols        | Use `enquo()` for expressions |

---

## 9. Flowchart: Tidy Evaluation Process

```
                                tidy evaluation
                                        |
          -------------------------------------------------------------------
          |                       |                    |                   |
       ensym()                 enquo()              quo()              expr()
   (capture symbol)     (capture expr + env)   (manual quosure)     (raw expr)
          |                       |                    |                   |
          ------------------------|--------------------|-------------------
                                   |
                             quosure / symbol
                                   |
                                unquote (!!)
                            ----->   !!   <-----
                           |                       |
                      Single insert           Multiple insert
                           |                       |
                       dplyr verbs             !!! + syms()
                                         |
                                     Evaluation
                                ---------------------
                                |                   |
                            base::eval()     rlang::eval_tidy()
                        (manual context)     (data-aware context)
      ```

