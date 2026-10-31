# DAX Measures & Calculated Columns

**Project:** Sales Performance Tracker\
**Author:** Himanshu\
**Tool:** Power BI Desktop\
**Source:** Extracted from the uploaded Power BI semantic model (TMDL)

This document records the explicit measures and calculated columns found
in the supplied semantic model. Formulas are transcribed from the model
definition.

## 1. DAX measures

### 1.1 `customer_avg_age`

**Table:** `customers`\
**Concepts:** `AVERAGE`

``` dax
customer_avg_age = AVERAGE(customers[customer_age])
```

Returns the average customer age in the current filter context.

### 1.2 `total_revenue`

**Table:** `sales`\
**Concepts:** `SUM`

``` dax
total_revenue = SUM(sales[discounted_price])
```

Sums the `discounted_price` column in the sales table.

### 1.3 `total_units_sold`

**Table:** `sales`\
**Concepts:** `SUM`

``` dax
total_units_sold = SUM(sales[sale_quantity])
```

Sums the sale quantity across the sales rows in the current filter
context.

### 1.4 `discounted_revenue`

**Table:** `sales`\
**Concepts:** `SUMX`, row context

``` dax
discounted_revenue =
SUMX(
    sales,
    sales[sale_quantity] *
    sales[discounted_price]
)
```

Iterates over sales rows and sums quantity multiplied by discounted
price.

### 1.5 `revenue_in_feb`

**Table:** `sales`\
**Concepts:** `CALCULATE`, date filtering, filter context

``` dax
revenue_in_feb =
CALCULATE(
    sales[discounted_revenue],
    dates[date] >= DATE(2025, 02, 01),
    dates[date] <= DATE(2025, 02, 28)
)
```

Evaluates the model's `discounted_revenue` measure for February 1--28,
2025.

### 1.6 `oldest_store_open_date`

**Table:** `stores`\
**Concepts:** `MIN`

``` dax
oldest_store_open_date = MIN(stores[store_open_date])
```

Returns the earliest store opening date in the current filter context.

### 1.7 `session_quality_score_avg_scaled`

**Table:** `website_sessions`\
**Concepts:** `AVERAGEX`, `INT`, iterator

``` dax
session_quality_score_avg_scaled =
AVERAGEX(
    website_sessions,
    INT(website_sessions[session_quality_score] * 100)
)
```

Averages the integer-converted session quality score after multiplying
each row's score by 100.

### 1.8 `target_quality_score`

**Table:** `website_sessions`\
**Concepts:** Scalar constant

``` dax
target_quality_score = 50
```

Returns a fixed target quality score of 50.

### 1.9 `max_quality_score`

**Table:** `website_sessions`\
**Concepts:** Scalar constant

``` dax
max_quality_score = 100
```

Returns a fixed maximum quality score of 100.

## 2. Calculated columns

The following four calculated columns were found in the `sales` table.

### 2.1 `price`

**Concepts:** `RELATED`, row context

``` dax
price =
sales[sale_quantity] *
RELATED(products[product_list_price])
```

Calculates quantity multiplied by the related product's list price.

### 2.2 `discounted_price`

**Concepts:** Row-level arithmetic

``` dax
discounted_price =
sales[price] * (1 - sales[sale_discount])
```

Calculates the row price after applying the discount rate.

### 2.3 `bulk_order`

**Concepts:** `IF`, conditional logic

``` dax
bulk_order =
IF(
    sales[sale_quantity] > 3,
    "True",
    "False"
)
```

Labels a row as a bulk order when its sale quantity is greater than 3.

### 2.4 `order_size`

**Concepts:** `SWITCH`, `TRUE()`, conditional categorization

``` dax
order_size =
SWITCH(
    TRUE(),
    sales[discounted_price] < 100, "Tiny",
    sales[discounted_price] < 200, "Mid",
    "Massive"
)
```

Categorizes the row's discounted price as Tiny (below 100), Mid (100 to
below 200), or Massive (200 and above).

## 3. DAX concepts demonstrated

  DAX concept               Functions / examples
  ------------------------- -----------------------------
  Aggregation               `SUM`, `MIN`, `AVERAGE`
  Iterator functions        `SUMX`, `AVERAGEX`
  Filter context            `CALCULATE`
  Date filtering            `DATE` and date comparisons
  Relationship navigation   `RELATED`
  Conditional logic         `IF`, `SWITCH`, `TRUE`
  Type conversion           `INT`
  Row-level calculations    `price`, `discounted_price`

## 4. Review notes before publishing

These are verification points, not edits to the extracted model.

1.  **`total_revenue` versus `discounted_revenue`:** The
    `discounted_price` calculated column is already quantity × list
    price × (1 − discount). The `discounted_revenue` measure multiplies
    that result by quantity again. Confirm whether this second quantity
    multiplication is intentional. The CEO dashboard's displayed revenue
    KPI uses `total_revenue`, not `discounted_revenue`.
2.  **`bulk_order` data type:** The expression returns text values
    `"True"` and `"False"`, while model metadata marks the column as
    Boolean. Verify the data type and expression are consistent in Power
    BI Desktop.
3.  **`revenue_in_feb`:** This measure is hard-coded to February 2025,
    so it is a fixed-period calculation rather than a dynamic
    month-selection measure.
4.  **Scaled session quality score:** The measure averages per-row
    integer-converted values after multiplying by 100. This can differ
    from multiplying the unrounded average by 100.

## 5. Extraction scope

The semantic model contains **nine explicit measures and four calculated
columns**. Automatically generated local date tables and the built-in
date hierarchy columns are not included as user-authored measures.
