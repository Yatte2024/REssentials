
# Tidy Evaluation in R – A Practical Guide

## 1. Background
## 1.1 What is Non-Standard Evaluation (NSE)?

In base R, **expressions like `filter(df, mpg > 20)` seem to "know" what `mpg` is** — even though it's not quoted, and not defined globally. This is possible due to **non-standard evaluation** (NSE).

---

#### ✅ Standard evaluation (SE)

```r
filter(df, "mpg > 20")  # Need to quote, evaluate explicitly
```

#### ✅ Non-standard evaluation (NSE)

```r
filter(df, mpg > 20)    # `mpg > 20` is captured as an unevaluated expression
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
| Usage         | `filter(df, "mpg > 20")`   | `filter(df, mpg > 20)`          |
| Programming   | Easy                       | Complex unless using rlang      |



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
| `ensym()`      | Capture a single symbol (e.g., column name) |
| `enquo()`      | Capture an expression with its environment |
| `quo()`        | Manually create a quosure                  |
| `expr()`       | Create a pure expression (no environment)  |
| `!!`           | Unquote a quosure/symbol to insert into expression |
| `!!!`          | Unquote-splice multiple expressions        |
| `syms()`       | Turn string vector into symbols list       |
| `as_name()`    | Turn symbol into string                    |
| `eval_tidy()`  | Evaluate in a tidy (data-aware) context    |

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

## 4. Injecting Expressions with `!!` and `!!!`

### `!!` – Unquote a single symbol/quosure

```r
col <- ensym(mpg)
filter(mtcars, !!col > 20)
```

### `!!!` – Unquote-splice a list (useful for multiple columns)

```r
cols <- syms(c("mpg", "hp"))
select(mtcars, !!!cols)
```

---

## 5. Building Expressions: `quo()` vs `expr()`

### What's the difference?

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

Summary:
- `quo()` = expression + environment
- `expr()` = just syntax (no context)

---

## 6. `eval()` vs `eval_tidy()`: Choosing the Right Tool

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

## 7. Dynamic Column Names from Strings

```r
cols <- c("mpg", "hp")

# Method 1: syms + !!!
select(mtcars, !!!syms(cols))

# Method 2: dplyr's all_of()
select(mtcars, all_of(cols))
```

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

