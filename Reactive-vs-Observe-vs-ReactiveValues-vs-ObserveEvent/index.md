---
title: "Deep Dive: Reactive vs Observe vs ReactiveValues vs observeEvent"
layout: default
---

Shiny’s reactive programming model is built on four key tools: `reactive()`, `observe()`, `reactiveValues()`, and `observeEvent()`. Each has a distinct role in making your app interactive, efficient, and responsive.

Let’s look at their **functional differences**, **common use cases**, and **complete Shiny examples** to help you use them correctly and confidently.

---

## 1. `reactive()`: Reactive Expression That Returns a Value

### What It Does

- `reactive()` defines an **expression that tracks reactive inputs and returns a value**.
- It is **lazy**, meaning it only runs **when called**.
- Think of it as a function that **remembers its dependencies** and recalculates only when necessary.

### Key Features

- Returns a value (must be called like `my_reactive()`).
- Only re-executes when:
  - A dependency (like `input$x`) changes **and**
  - The expression is **actually called**.

### When to Use

- When you want to compute a value based on inputs.
- When the same logic is needed in multiple places.
- When you want to avoid unnecessary recalculations.

### Example

```r
library(shiny)

ui <- fluidPage(
  titlePanel("Calculate Square Value:"),
  numericInput("num", "Enter a number:", 1),
  verbatimTextOutput("square")
)

server <- function(input, output) {
  squared <- reactive({
    input$num ^ 2
  })

  output$square <- renderPrint({
    squared()
  })
}

shinyApp(ui, server)
```

---

## 2. `observe()`: React to Changes and Perform Side Effects

### What It Does

- `observe()` **monitors inputs or reactive expressions** and **runs side-effect code** when those dependencies change.
- It does **not return a value** and cannot be used in expressions like `reactive()`.
- It runs **eagerly**—as soon as a dependency changes, it executes.

### Key Features

- Used for **side effects**: printing, updating UI elements, writing files, etc.
- Does not return anything.
- Runs **every time a dependency updates**.

### When to Use

- To update UI elements dynamically.
- To log information or debug.
- When you need something to happen automatically when inputs change.

### Example: Dynamically update a dropdown

```r
library(shiny)

ui <- fluidPage(
  titlePanel("When the pets is changed, the breed updates automatically."),
  selectInput("Pets", "Select pets", choices = c("Dogs", "Cats")),
  selectInput("Breed", "Select model", choices = NULL)
)

server <- function(input, output, session) {
  observe({
    models <- if (input$Pets == "Dogs") {
      c("Maltese", "Maltipoo", "Cavapoo")
    } else {
      c("Ragdoll", "Siberian")
    }
    
    updateSelectInput(session, "Breed", choices = models)
  })
}

shinyApp(ui, server)
```

---

## 3. `reactiveValues()`: A Mutable Reactive State Container

### What It Does

- `reactiveValues()` creates an **object similar to a list** that can store **reactive values**.
- It acts like a reactive global variable for your app.
- Works well for app state: counters, data selections, steps in a process, etc.

### Key Features

- Use `$` to **access or modify** fields (e.g. `values$count <- values$count + 1`).
- Automatically tracks dependencies.
- Often used with `observeEvent()` for event-based updates.

### When to Use

- To store and update values over time.
- For use cases like a shopping cart, counter, or workflow stepper.
- When you need shared state that persists across reactive blocks.

### Example

```r
library(shiny)

ui <- fluidPage(
  actionButton("add", "Add 1"),
  verbatimTextOutput("count")
)

server <- function(input, output) {
  values <- reactiveValues(count = 0)

  observeEvent(input$add, {
    values$count <- values$count + 1
  })

  output$count <- renderPrint({
    values$count
  })
}

shinyApp(ui, server)
```

---

## 4. `observeEvent()`: React to Specific Events (e.g., Button Clicks)

### What It Does

- `observeEvent()` runs code **only when a specific event occurs**.
- Unlike `observe()`, it **ignores reactivity until the event is triggered**.
- Ideal for discrete actions like button clicks, file uploads, or tabs switching.

### Key Features

- Triggers only when a **specific input changes**.
- Can include `ignoreInit = TRUE` to skip running on startup.
- Prevents unnecessary re-runs due to unrelated input changes.

### When to Use

- When you want to run code only on demand (e.g., after clicking a button).
- When you need full control over when reactivity is triggered.
- For user-triggered computations or updates.

### Example

```r
library(shiny)

ui <- fluidPage(
  numericInput("n", "How many numbers?", 100),
  actionButton("go", "Generate"),
  plotOutput("hist")
)

server <- function(input, output) {
  observeEvent(input$go, {
    data <- rnorm(input$n)
    output$hist <- renderPlot({
      hist(data)
    })
  })
}

shinyApp(ui, server)
```

---

## Summary: Comparing Core Reactive Tools

| Function           | Returns Value | Lazy? | When It Runs                      | Primary Use                             |
|--------------------|---------------|--------|------------------------------------|------------------------------------------|
| `reactive()`        | Yes           | Yes    | When called, after dependencies update | Return computed values for reuse         |
| `observe()`         | No            | No     | Immediately when dependencies update | Side effects (print, UI update, etc.)    |
| `reactiveValues()`  | Yes (like list) | N/A  | When accessed or updated            | Store and track reactive state           |
| `observeEvent()`    | No            | No     | Only when specified event occurs    | Trigger logic on specific user events    |
