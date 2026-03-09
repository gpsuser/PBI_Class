# Power BI: Building a Dimensional Model from Raw Data

---

## Part 1: Quiz

---

### Questions 1-4: Fill in the Blanks

For each question below, complete the sentence by choosing the correct word or phrase from the word bank provided. Each word bank is specific to that question and has been scrambled.

---

**Question 1**

A __________ table creates a live link back to the original source table, meaning that when the underlying data is refreshed, changes automatically flow through to any tables built from it. By contrast, a __________ creates a snapshot at the time of copying and does not update when the source changes. When building dimension tables in Power BI, the correct approach is to use a __________ so that the model stays in sync with the raw data.

**Word bank:** `snapshot` | `reference` | `duplicate`

---

**Question 2**

In a Star Schema, the central table is called the __________ table. It stores transactional or event-driven records and connects to surrounding __________ tables via __________ keys. Each of those surrounding tables holds a unique __________ key that is used to make the connection.

**Word bank:** `primary` | `dimension` | `foreign` | `fact`

---

**Question 3**

In Power Query, when you merge two tables and want to keep all rows from the left table regardless of whether a match exists in the right table, you should use a __________ join. If a match is not found, the columns from the right table are filled with __________. If you only want rows where a match exists in both tables, you would instead use an __________ join.

**Word bank:** `null` | `inner` | `left outer`

---

**Question 4**

Every transformation step applied in the Power Query Editor is automatically recorded as __________ language code. This code is visible in the __________ bar at the top of the editor, or in full inside the __________ Editor. Unlike DAX, this language runs __________ data is loaded into the model.

**Word bank:** `before` | `formula` | `M` | `Advanced`

---

### Questions 5-7: Multiple Choice

Circle or mark the single best answer for each question.

---

**Question 5**

You are building a DIM-customer table from a raw data source. The raw data contains 1,500 transaction rows, but only 40 unique customers. After referencing the raw table and selecting only the Customer column, what step should you perform next before adding a surrogate key?

- A) Sort the Customer column alphabetically
- B) Remove duplicate rows on the Customer column
- C) Filter the Customer column to remove nulls using an Inner Join
- D) Merge the Customer column with the FACT table

---

**Question 6**

Look at the two tables below and the join result shown.

```
Table A (Left)          Table B (Right)
+----+---------+        +----+----------+
| ID | City    |        | ID | Country  |
+----+---------+        +----+----------+
|  1 | London  |        |  1 | England  |
|  2 | Dublin  |        |  3 | Ireland  |
|  3 | Cardiff |        +----+----------+
+----+---------+

Join Result:
+----+---------+----------+
| ID | City    | Country  |
+----+---------+----------+
|  1 | London  | England  |
|  3 | Cardiff | Ireland  |
+----+---------+----------+
```

Which join type produced this result?

- A) Left Outer Join
- B) Full Outer Join
- C) Inner Join
- D) Left Anti Join

---

**Question 7**

A colleague has built a summary table called TotSalesByRegion using the Group By feature in Power Query. They now want this table to respond dynamically to slicers and filters when building a report. Which of the following statements is most accurate?

- A) The Group By table will respond dynamically because it is connected to the Star Schema via relationships
- B) The Group By table produces a static, pre-aggregated result and will not respond dynamically to slicers; DAX measures would be needed for dynamic filtering
- C) The Group By table will respond dynamically as long as it is placed in the same group folder as the FACT table in Power Query
- D) The Group By table will respond dynamically once the surrogate key columns are removed

---

## Part 2: Tasks

---

### Task 1: Trace the M Code

The Power Query Advanced Editor for a query called DIM-product contains the following M script. Read it carefully and answer the questions below.

```
let
    Source = Excel.Workbook(File.Contents("C:\Data\SalesData.xlsx"), null, true),
    Sales_Sheet = Source{[Item="Sales",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(Sales_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",
                        {{"Product", type text}, {"Unit Price", type number}}),
    #"Kept Columns" = Table.SelectColumns(#"Changed Type", {"Product", "Unit Price"}),
    #"Removed Duplicates" = Table.Distinct(#"Kept Columns", {"Product"}),
    #"Added Index" = Table.AddIndexColumn(#"Removed Duplicates", "Index", 1, 1, Int64.Type)
in
    #"Added Index"
```

**Question 1a**

List the transformation steps in the order they are applied, using only the step names (not the M function names). The first one has been done for you.

```
Step 1: Source
Step 2: _______________
Step 3: _______________
Step 4: _______________
Step 5: _______________
Step 6: _______________
Step 7: _______________
```

**Question 1b**

The `#"Added Index"` step adds a surrogate key to the table. Based on the arguments passed to `Table.AddIndexColumn`, answer the following:

- What is the name given to the new index column?
- What number does the index start from?
- What data type is the index column?

**Question 1c**

Rewrite the `#"Kept Columns"` step so that it keeps only the `Product` column and drops `Unit Price`.

```
#"Kept Columns" = 
```

**Question 1d**

The query above does not rename the index column to `id-product`. Write the M step that would rename `Index` to `id-product`. Use the scaffold below.

```
#"Renamed Column" = Table.RenameColumns( ___________ , {{ ___________ , ___________ }})
```

---

### Task 2: Design and Trace a Merge

You have the two tables shown below and you need to enrich DIM-region by adding the `Region Manager` column from the Managers table.

```
DIM-region (current state)
+----------+-----------+------------------+---------+
| id-region | City      | Population Size  | State   |
+----------+-----------+------------------+---------+
|         1 | Bath      | 88,000           | England |
|         2 | Bristol   | 470,000          | England |
|         3 | Cork      | 210,000          | Ireland |
|         4 | Galway    | 80,000           | Ireland |
+----------+-----------+------------------+---------+

Managers table
+---------+------------------+
| State   | Region Manager   |
+---------+------------------+
| England | Sarah Okafor     |
| Ireland | Liam Brennan     |
+---------+------------------+
```

**Question 2a**

What column should be used as the matching key when merging Managers into DIM-region? Explain in one sentence why you chose this column.

**Question 2b**

Which join type should you use for this merge? In one sentence, explain why this is the correct choice.

**Question 2c**

Draw the expected output table after the merge is complete and the `Region Manager` column has been expanded. Use the ASCII table format shown above.

**Question 2d**

After expanding the merged column, Power Query sometimes prefixes the new column name with the source table name (e.g., `Managers.Region Manager`). Write the single M function call that renames it to `Region Manager`. Use the scaffold below.

```
#"Renamed Column" = Table.RenameColumns( ___________ , {{ ___________ , ___________ }})
```

---

### Challenge Task: Build a Custom M Step from Scratch

You are working with the following pre-aggregated summary table in Power Query called `TotSalesByCustomerByProduct`.

```
+------------------+------------------+-------------+
| Customer         | Product          | Total Sales |
+------------------+------------------+-------------+
| Acme Corp        | Widget A         |      900.00 |
| Acme Corp        | Widget B         |      450.00 |
| Brendan Ltd      | Service Package  |     2100.00 |
| Clancy Supplies  | Widget A         |      300.00 |
| Clancy Supplies  | Widget B         |      750.00 |
+------------------+------------------+-------------+
```

Your task is to extend this table using M language custom column steps.

---

**Part A: Concatenated Label Column**

Using what you know about the `&` operator and `each` in M, write the custom column formula that creates a new column called `Summary Label` that combines Customer and Product in this format:

```
Acme Corp -- Widget A
```

(That is: Customer, space, two hyphens, space, Product.)

```
= Table.AddColumn( ___________ , "Summary Label", each ___________ )
```

---

**Part B: Sales Band Column**

Write a custom column M formula that creates a new column called `Sales Band` using the following logic:

```
If Total Sales >= 1000 then "High"
else if Total Sales >= 500 then "Medium"
else "Low"
```

Use the scaffold below. Fill in the condition expressions.

```
= Table.AddColumn( #"Added Summary Label", "Sales Band", each
    if ___________ then "High"
    else if ___________ then "Medium"
    else "Low"
)
```

---

**Part C: Expected Output**

Draw the expected output table after both new columns have been added. Include all five original rows. Use the ASCII table format shown above.

```
+------------------+------------------+-------------+----------------------------+------------+
| Customer         | Product          | Total Sales | Summary Label              | Sales Band |
+------------------+------------------+-------------+----------------------------+------------+
|                  |                  |             |                            |            |
...
```

---

**Part D: Reflect**

In two to three sentences, explain why it is generally better practice to add calculated or categorisation columns like `Sales Band` using DAX measures in the data model rather than as Power Query custom columns. Consider what happens when a user applies a slicer to a report.

---

*End of worksheet.*
