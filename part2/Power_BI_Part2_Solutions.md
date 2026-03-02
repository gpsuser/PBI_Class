# Power BI - Data Modelling and DAX
# Answer Key

---

## Section 1: Quiz Answers

---

**Question 1** - Answer: **fact**

In a star schema, the **fact** table sits at the centre and contains measurable information relating to an event, such as a sales transaction.

---

**Question 2** - Answer: **denormalised**

The initial raw data table is referred to as being **denormalised** because it contains repeated or redundant data.

---

**Question 3** - Answer: **primary**

Dimension tables hold **primary** keys that connect to the foreign keys stored in the fact table.

---

**Question 4** - Answer: **CALCULATE**

The **CALCULATE** function is used to evaluate an expression under a modified filter context.

---

**Question 5** - Answer: **B**

Normalisation reduces data redundancy by organising data into separate related tables. This is the core purpose of splitting raw denormalised data into fact and dimension tables in a star schema.

---

**Question 6** - Answer: **C**

The measure uses CALCULATE with a FILTER that restricts Dim_Customer to rows where [Service Tier] = "Platinum". SUM then totals Fact_Sales[Total Spend] for only those matching rows. The result is the total spend for Platinum-tier customers only.

---

**Question 7** - Answer: **B**

In a star schema all dimension tables connect directly to the central fact table. Dimension tables do not connect to each other. Option A shows dimensions connected to each other, which would represent a snowflake schema rather than a star schema.

---

## Section 2: Task Answers

---

### Task 1 - Identifying Star Schema Components

**Classification Table:**

| Column Name     | Fact / Dimension / Both | If Dimension: Table Name |
|-----------------|------------------------|--------------------------|
| SaleID          | Fact (Primary Key)      | N/A                      |
| CustomerName    | Dimension               | Dim_Customer             |
| ProductName     | Dimension               | Dim_Product              |
| SaleDate        | Dimension               | Dim_Date                 |
| Amount          | Fact (measure)          | N/A                      |
| ProductID       | Both                    | Dim_Product (as PK), Fact_Sales (as FK) |
| CustomerID      | Both                    | Dim_Customer (as PK), Fact_Sales (as FK)|
| City            | Dimension               | Dim_Customer or Dim_City |
| Quarter         | Dimension               | Dim_Date                 |

**Fact_Sales table (from sample data):**

| SaleID | FK_CustomerID | FK_ProductID | FK_DateID | Amount |
|--------|---------------|--------------|-----------|--------|
| 1      | 1             | 1            | 1         | 100.00 |
| 2      | 2             | 2            | 2         | 150.00 |
| 3      | 1             | 3            | 3         | 200.00 |

**Dim_Product table (from sample data):**

| ProductID | ProductName |
|-----------|-------------|
| 1         | Widget A    |
| 2         | Widget B    |
| 3         | Widget C    |

**Dim_Date table (from sample data):**

| DateID | SaleDate   |
|--------|------------|
| 1      | 2024-01-01 |
| 2      | 2024-01-02 |
| 3      | 2024-01-03 |

---

### Task 2 - Writing DAX Measures

**Measure 1 - Total Revenue:**

```dax
msr_total_revenue = SUM(Fact_Sales[TotalAmount])
```

The blank is filled with `SUM`. The SUM function aggregates all values in the specified column across the current filter context.

---

**Measure 2 - Gold Tier Revenue:**

```dax
msr_gold_revenue = CALCULATE(
    SUM(Fact_Sales[TotalAmount]),
    FILTER(Dim_Customer, [ServiceTier] = "Gold")
)
```

- The first blank is `Dim_Customer` - this is the table being filtered.
- The second blank is `Gold` - the value being matched in the ServiceTier column.
- CALCULATE modifies the filter context so that SUM only operates over sales linked to Gold-tier customers.

---

### Challenge Task - DAX Table with SUMMARIZE and FILTER

**Complete solution:**

```dax
tbl_RegionSummary =

VAR FilteredSales =
    FILTER(
        ADDCOLUMNS(
            Fact_Sales,
            "Category", RELATED(Dim_Product[Category])
        ),
        [Category] = "Electronics"
    )

VAR GroupedData =
    SUMMARIZE(
        FilteredSales,
        Dim_Customer[Region],
        Dim_Product[ProductName],
        "Total Amount", SUM(Fact_Sales[TotalAmount])
    )

RETURN
    GroupedData
```

**Blank answers in order:**

1. `Dim_Product` - the related table containing the Category column
2. `Category` - the column name in Dim_Product
3. `"Electronics"` - the filter value
4. `Region` - the column from Dim_Customer to group by
5. `ProductName` - the column from Dim_Product to group by
6. `TotalAmount` - the measure column from Fact_Sales to sum
7. `GroupedData` - the variable returned

**Explanation:**

- `ADDCOLUMNS` extends the Fact_Sales table with a new virtual column called "Category" by using `RELATED` to look up the Category from Dim_Product via the existing relationship.
- `FILTER` then restricts the extended table to rows where Category equals "Electronics".
- `SUMMARIZE` groups the filtered result by Region and ProductName, calculating the sum of TotalAmount for each group.
- `RETURN GroupedData` outputs the grouped variable as the table result.

The rows containing "Accessories" (Widget A, SaleID 2 and 5) are excluded by the filter, leaving only the three Electronics rows in the output.
