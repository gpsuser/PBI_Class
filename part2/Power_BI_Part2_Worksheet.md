# Power BI - Data Modelling and DAX

---

## Section 1: Quiz

---

### Questions 1-4: Fill in the Blanks

For each question, choose the correct word from the word bank provided and fill in the blank.

---

**Question 1**

In a star schema, the __________ table sits at the centre and contains measurable information relating to an event, such as a sales transaction.

**Word Bank (scrambled):** primary / fact / dimension / foreign

---

**Question 2**

The initial raw data table, before it is split into fact and dimension tables, is referred to as being __________ because it contains repeated or redundant data.

**Word Bank (scrambled):** normalised / aggregated / denormalised / indexed

---

**Question 3**

Dimension tables hold __________ keys that connect to the foreign keys stored in the fact table.

**Word Bank (scrambled):** composite / foreign / primary / surrogate

---

**Question 4**

In DAX, the __________ function is used to evaluate an expression under a modified filter context, making it the foundation for many calculated measures.

**Word Bank (scrambled):** RELATED / SUMMARIZE / CALCULATE / FILTER

---

### Questions 5-7: Multiple Choice

Circle or write the letter of the best answer.

---

**Question 5**

Which of the following best describes the purpose of normalisation in dimensional modelling?

- A) To increase the number of rows in the fact table
- B) To reduce data redundancy by organising data into separate related tables
- C) To combine all data into a single flat table for faster loading
- D) To remove primary keys from dimension tables

---

**Question 6**

Consider the following DAX measure:

```
msr_platinum_tier = CALCULATE(
    SUM(Fact_Sales[Total Spend]),
    FILTER(Dim_Customer, [Service Tier] = "Platinum")
)
```

What does this measure calculate?

- A) The count of all customers in the Platinum tier
- B) The total spend across all customers regardless of tier
- C) The total spend for customers in the Platinum service tier only
- D) The average spend per Platinum customer

---

**Question 7**

In a star schema diagram, which of the following best represents the correct structure?

```
Option A:                    Option B:

  Dim_A --- Dim_B            Dim_A
    |           |               \
  Fact ------Dim_C           Fact_Sales --- Dim_B
    |                           /
  Dim_D                     Dim_C
```

- A) Option A, where dimension tables connect to each other
- B) Option B, where all dimension tables connect directly to the central fact table
- C) Both options are equally valid star schema designs
- D) Neither option represents a star schema

---

## Section 2: Tasks

---

### Task 1 - Identifying Star Schema Components

The table below contains a mix of columns from a sales dataset. Your task is to classify each column and indicate whether it belongs in the **Fact table**, a **Dimension table**, or could appear in **both**.

For each column, also state **which dimension table** it would belong to if it is a dimension attribute (e.g., Dim_Product, Dim_Customer, Dim_Date).

**Raw data columns to classify:**

| Column Name     | Fact / Dimension / Both | If Dimension: Table Name |
|-----------------|------------------------|--------------------------|
| SaleID          |                        |                          |
| CustomerName    |                        |                          |
| ProductName     |                        |                          |
| SaleDate        |                        |                          |
| Amount          |                        |                          |
| ProductID       |                        |                          |
| CustomerID      |                        |                          |
| City            |                        |                          |
| Quarter         |                        |                          |

**Scaffolding - Star Schema Structure:**

```
         Dim_Customer         Dim_Product
         +------------+       +------------+
         | CustomerID |       | ProductID  |
         | CustomerName|      | ProductName|
         +-----+------+       +------+-----+
               |                     |
        FK_CustomerID         FK_ProductID
               |                     |
         +-----+---------------------+------+
         |           Fact_Sales             |
         |  SaleID (PK)                     |
         |  FK_CustomerID                   |
         |  FK_ProductID                    |
         |  FK_DateID                       |
         |  Amount                          |
         +------------------+---------------+
                            |
                       FK_DateID
                            |
                    +-------+------+
                    |  Dim_Date    |
                    |  DateID (PK) |
                    |  SaleDate    |
                    |  Year        |
                    |  Month       |
                    |  Quarter     |
                    +--------------+
```

**Sample Input (raw denormalised data):**

| SaleID | CustomerName | ProductName | SaleDate   | Amount | City       |
|--------|--------------|-------------|------------|--------|------------|
| 1      | Alice        | Widget A    | 2024-01-01 | 100.00 | Manchester |
| 2      | Bob          | Widget B    | 2024-01-02 | 150.00 | London     |
| 3      | Alice        | Widget C    | 2024-01-03 | 200.00 | Manchester |

**Expected Output (partial - Dim_Customer example):**

| CustomerID | CustomerName | City       |
|------------|--------------|------------|
| 1          | Alice        | Manchester |
| 2          | Bob          | London     |

Complete the classification table and then write out what the Fact_Sales table and at least one other dimension table would look like using the sample data above.

---

### Task 2 - Writing DAX Measures

Using the star schema below, write two DAX measures as described.

**Schema:**

```
Fact_Sales: SaleID, FK_CustomerID, FK_ProductID, FK_DateID, Quantity, UnitPrice, TotalAmount
Dim_Customer: CustomerID, CustomerName, ServiceTier, Region
Dim_Product: ProductID, ProductName, Category
Dim_Date: DateID, SaleDate, Year, Month, Quarter
```

**Measure 1 - Total Revenue**

Write a DAX measure called `msr_total_revenue` that calculates the total of `Fact_Sales[TotalAmount]`.

```dax
-- Scaffolding: fill in the blanks
msr_total_revenue = __________(Fact_Sales[TotalAmount])
```

**Sample output (as a Card visual):**

```
+---------------------+
|   Total Revenue     |
|                     |
|     75,430.00       |
+---------------------+
```

**Measure 2 - Gold Tier Revenue**

Write a DAX measure called `msr_gold_revenue` that calculates the total revenue for customers whose `ServiceTier` is "Gold".

```dax
-- Scaffolding: use CALCULATE and FILTER
msr_gold_revenue = CALCULATE(
    SUM(Fact_Sales[TotalAmount]),
    FILTER(__________, [ServiceTier] = "________")
)
```

**Sample output (as a Card visual):**

```
+---------------------+
|  Gold Tier Revenue  |
|                     |
|     22,150.00       |
+---------------------+
```

---

### Challenge Task - DAX Table with SUMMARIZE and FILTER

This task requires you to write a complete DAX expression to create a calculated table in Power BI.

**Scenario:**

You need to create a summary table called `tbl_RegionSummary` that:

1. Filters `Fact_Sales` to include only rows where `Dim_Product[Category]` is "Electronics"
2. Groups the results by `Dim_Customer[Region]` and `Dim_Product[ProductName]`
3. Calculates the total `Fact_Sales[TotalAmount]` for each group
4. Returns the grouped result

**Schema reminder:**

```
Fact_Sales: SaleID, FK_CustomerID, FK_ProductID, TotalAmount
Dim_Customer: CustomerID, CustomerName, Region
Dim_Product: ProductID, ProductName, Category
```

**Scaffolding:**

```dax
tbl_RegionSummary =

VAR FilteredSales =
    FILTER(
        ADDCOLUMNS(
            Fact_Sales,
            "Category", RELATED(__________[__________])
        ),
        [Category] = "__________"
    )

VAR GroupedData =
    SUMMARIZE(
        FilteredSales,
        Dim_Customer[__________],
        Dim_Product[__________],
        "Total Amount", SUM(Fact_Sales[__________])
    )

RETURN
    __________
```

**Sample Input Data:**

| SaleID | Region | ProductName    | Category    | TotalAmount |
|--------|--------|----------------|-------------|-------------|
| 1      | North  | Laptop Pro     | Electronics | 1200.00     |
| 2      | South  | Widget A       | Accessories | 50.00       |
| 3      | North  | Tablet X       | Electronics | 750.00      |
| 4      | East   | Laptop Pro     | Electronics | 1200.00     |
| 5      | South  | Widget A       | Accessories | 50.00       |

**Expected Output (tbl_RegionSummary):**

| Region | ProductName | Total Amount |
|--------|-------------|--------------|
| East   | Laptop Pro  | 1200.00      |
| North  | Laptop Pro  | 1200.00      |
| North  | Tablet X    | 750.00       |

Complete the scaffolded DAX expression above to produce this result.
