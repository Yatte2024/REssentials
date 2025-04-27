---
title: "R Package Development Workflow Summary"
layout: default
---

This guide explains how to create, document, test, and finalize an R package, using professional best practices.

---

## 1. Package Core Structure Overview

| Component | Purpose | Example |
|:---------|:--------|:--------|
| `R/` | Store `.R` function scripts | `temperature_utils.R` |
| `man/` | Store `.Rd` help files | `temperature_utils.Rd` |
| `DESCRIPTION` | Metadata (name, version, author) | easyToolkit metadata |
| `NAMESPACE` | Manage exports and imports | `export(fahrenheit_to_celsius)` |
| `data/` | Built-in datasets | `weather_samples.rda` |
| `inst/` | Templates, reports | Custom R Markdown templates |
| `vignettes/` | Tutorials and guides | `intro_to_easyToolkit.Rmd` |
| `tests/` | Unit tests (testthat) | `test-temperature_utils.R` |

---

## 2. Setting Up the Package

```r
usethis::create_package("easyToolkit")
dir(".")
```

Essential structure created: `DESCRIPTION`, `NAMESPACE`, `R/`.

Optionally, you can initialize Git (`usethis::use_git()`) and publish to GitHub (`usethis::use_github()`).

---

## 3. Writing Functions (and use_r vs dump)
**Difference between `use_r()` and `dump()`:**

| Function | Description | Typical Usage |
|:---------|:------------|:--------------|
| `usethis::use_r("file_name")` | Creates a new, empty `.R` file in the `R/` folder for writing new functions. | Planned development, clean organization. |
| `dump("function_name", "R/file_name.R")` | Saves an existing function from the environment into an `.R` file. | Quick saving of interactively written code. |

✅ Prefer `use_r()` for structured development; use `dump()` for quick saving from console.

**Example Function**:

```r
fahrenheit_to_celsius <- function(temp_f) {
  (temp_f - 32) * 5/9
}

usethis::use_r("temperature_utils") # Create
# OR
dump("fahrenheit_to_celsius", "R/temperature_utils.R") # Save
```

---


## 4. Installing vs Loading the Package

| Command | Purpose | When to use |
|:-------|:--------|:------------|
| `devtools::install()` | Installs the package | After major changes |
| `devtools::load_all()` | Loads package without installing | During active development |

(devtools::install() compiles the package formally, devtools::load_all() only loads code for development without installation.)

---

## 5. Package Naming and Licensing

### Check availability:

```r
available::available("easyToolkit")
```

**What does available::available() do?**

- It checks if your chosen package name is already taken on CRAN or elsewhere.
- It suggests alternative names if the chosen name is unavailable.
- Helps you pick a unique, valid package name easily.

**Common mistakes when naming packages:**

| Good Example | Bad Example | Reason |
|:------------|:------------|:-------|
| `easyToolkit` | `data_tools` | Underscores `_` not allowed |
| `cleanR` | `data.helper` | Dots `.` are discouraged |
| `fastpkg` | `pkg!name` | Special characters `!` not allowed |
| `pkgdata` | `123pkg` | Cannot start with a number |

✅ Package names should be concise, use only letters and numbers, start with a letter, and avoid special characters.

### Apply license

```r
usethis::use_mit_license(name = "Your Name")
```

**Choosing a License:**

| License | Description | Use Case | Function |
|:--------|:------------|:---------|:---------|
| MIT License | Very permissive; allows reuse with minimal restrictions. | Good for open-source and wide distribution. | `use_mit_license()` |
| GPL-3 License | Requires derivative works to also be open source under GPL. | Good if you want to enforce open-source compliance. | `use_gpl3_license()` |
| CC0 License | Public domain dedication; no rights reserved. | Good if you want to give up all rights and encourage maximum reuse. | `use_cc0_license()` |

✅ Best Practice: Use MIT for maximum flexibility, GPL-3 if you want stricter sharing rules.



---

## 6. Managing Dependencies (Including Minimum Version)

Add packages:
| Type | Loaded automatically? | Notes |
|:------|:------|:------|
|Imports |    ✅ | Functions always available|
|Suggests |    ❌ | Optional, examples or vignettes only|
|Depends |    ⚠️ | Full attachment, best avoided|


```r
usethis::use_package("stringr")
usethis::use_package("kableExtra", type = "Suggests")
usethis::use_package("stringr", type = "Import", min_version = "1.5.0") #Set minimum version
```

---

## 7. Writing Documentation

Roxygen2 header:

```r
#' Convert Fahrenheit to Celsius
#'
#' @param temp_f Numeric temperature in Fahrenheit
#' @return Numeric temperature in Celsius
#' @export
#'
#' @examples
#' fahrenheit_to_celsius(32)
fahrenheit_to_celsius <- function(...) { ... }
```

Generate documentation:

```r
devtools::document()
# or lower level
roxygen2::roxygenize()
```

(devtools::document() also updates DESCRIPTION and NAMESPACE, while roxygen2::roxygenize() only updates documentation.)

---

## 8. Creating Vignettes (Vignette Building)

Create vignette:

```r
usethis::use_vignette("intro_to_easyToolkit", title = "Getting Started with easyToolkit")
```

Build vignettes:

```r
devtools::build_vignettes()
rmarkdown::render("vignettes/intro_to_easyToolkit.Rmd")
```

(devtools::build_vignettes() builds for package use, rmarkdown::render() is for quick checking during writing.)

---

## 9. Unit Testing with testthat

Write tests:

```r
testthat::test_that("fahrenheit_to_celsius works correctly", {
  testthat::expect_equal(fahrenheit_to_celsius(32), 0)
  testthat::expect_equal(fahrenheit_to_celsius(212), 100)
  testthat::expect_error(fahrenheit_to_celsius("abc"))
})
```

Setup testing framework:

```r
usethis::use_testthat()
usethis::use_test("temperature_utils")
```

Run all tests:

```r
devtools::test()
```

Optionally, you can check code coverage with `testthat::test_coverage()`.

Common test functions:

| Function | Purpose |
|:---------|:--------|
| `testthat::expect_equal()` | Allow for minor numerical differences |
| `testthat::expect_identical()` | Exact match |
| `testthat::expect_error()` | Check if error occurs |
| `testthat::expect_warning()` | Check if warning occurs |
| `testthat::expect_output()` | Match printed output |
| `testthat::expect_true()` | Check if result is TRUE |
| `testthat::expect_false()` | Check if result is FALSE |
| `testthat::expect_match()` | Check if string matches regex |
| `testthat::expect_s3_class()` | Check if object has expected S3 class |
| `testthat::expect_length()` | Check object length |
| `testthat::expect_named()` | Check if object is named |

Best practices for writing tests:

- Use small, focused `test_that()` blocks.
- Always test normal cases, edge cases, and invalid inputs.
- Cover both success and expected failure scenarios.
- Use `tolerance` argument when comparing floating-point numbers.
- Rerun tests after every meaningful code change.
- Keep tests independent from each other.

Example of advanced testing:

```r
testthat::test_that("fahrenheit_to_celsius handles numeric input correctly", {
  testthat::expect_equal(fahrenheit_to_celsius(98.6), 37, tolerance = 1e-1)
})

testthat::test_that("fahrenheit_to_celsius throws error on non-numeric input", {
  testthat::expect_error(fahrenheit_to_celsius("not_a_number"))
})

testthat::test_that("fahrenheit_to_celsius special cases", {
  testthat::expect_equal(fahrenheit_to_celsius(-40), -40) # Celsius and Fahrenheit meet at -40
})
```

---

## 10. Final Steps Before Release

- Update `DESCRIPTION` fields.
- Document datasets.
- Final check:

```r
devtools::check()
```

Goal: 0 Errors, 0 Warnings, 0 Notes.

---

## 11. Whole process

| Good Practice | Reminder |
|:-------------|:---------|
| One function per file | Easier maintenance |
| `load_all()` for dev, `install()` for release | Faster iteration |
| Document functions immediately | Don't delay |
| Cover edge cases in tests | Improve robustness |
| Run `check()` before publishing | Ensure quality |

