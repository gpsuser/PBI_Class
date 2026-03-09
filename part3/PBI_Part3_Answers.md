# Power BI: Building a Dimensional Model from Raw Data -- Answers

---

## Part 1: Quiz -- Answers

---

### Questions 1-4: Fill in the Blanks

---

**Question 1 -- Answer**

A **reference** table creates a live link back to the original source table, meaning that when the underlying data is refreshed, changes automatically flow through to any tables built from it. By contrast, a **duplicate** creates a snapshot at the time of copying and does not update when the source changes. When building dimension tables in Power BI, the correct approach is to use a **reference** so that the model stays in sync with the raw data.

---

**Question 2 -- Answer**

In a Star Schema, the central table is called the **fact** table. It stores transactional or event-driven records and connects to surrounding **dimension** tables via **foreign** keys. Each of those surrounding tables holds a unique **primary** key that is used to make the connection.

---

**Question 3 -- Answer**

In Power Query, when you merge two tables and want to keep all rows from the left table regardless of whether a match exists in the right table, you should use a **left outer** join. If a match is not found, the columns from the right table are filled with **null**. If you only want rows where a match exists in both tables, you would instead use an **inner** join.

---

**Question 4 -- Answer**

Every transformation step applied in the Power Query Editor is automatically recorded as **M** language code. This code is visible in the **formula** bar at the top of the editor, or in full inside the **Advanced** Editor. Unlike DAX, this language runs **before** data is loaded into the model.

---

### Questions 5-7: Multiple Choice -- Answers

---

**Question 5 -- Answer: B**

After selecting only the Customer column, the next step is to **remove duplicate rows on the Customer column**. This ensures that each customer appears exactly once in the dimension table before a surrogate key is added. Sorting (A) is not required for a dimension to function correctly. An Inner Join (C) is not a row-deduplication step. Merging with the FACT table (D) comes much later and serves a different purpose.

---

**Question 6 -- Answer: C**

The result is produced by an **Inner Join**. Only rows where the ID value exists in both Table A and Table B are returned: IDs 1 and 3. ID 2 (Dublin) is excluded because there is no matching row in Table B. A Left Outer Join (A) would have included Dublin with a null Country. A Full Outer Join (B) would have included all rows from both sides. A Left Anti Join (D) would have returned only Dublin (the unmatched left row).

---

**Question 7 -- Answer: B**

The Group By approach in Power Query produces a **static, pre-aggregated result**. Once loaded, the numbers in the table are fixed and do not recalculate in response to slicers or filters. DAX measures, written after the model is built, calculate dynamically at query time and respond to all active filter contexts. The table's folder grouping (C) and surrogate key presence (D) have no effect on dynamic filtering behaviour.

---

## Part 2: Tasks -- Answers

---

### Task 1: Trace the M Code -- Answers

**Question 1a -- Step names in order:**

```
Step 1: Source
Step 2: Sales_Sheet
Step 3: Promoted Headers
Step 4: Changed Type
Step 5: Kept Columns
Step 6: Removed Duplicates
Step 7: Added Index
```

---

**Question 1b -- Index column details:**

- Name given to the new column: `Index`
- Number the index starts from: `1`
- Data type of the index column: `Int64.Type` (64-bit integer)

---

**Question 1c -- Rewritten Kept Columns step:**

```
#"Kept Columns" = Table.SelectColumns(#"Changed Type", {"Product"})
```

Only `"Product"` is passed in the list. `"Unit Price"` is removed from the list of columns to keep.

---

**Question 1d -- Rename step:**

```
#"Renamed Column" = Table.RenameColumns(#"Added Index", {{"Index", "id-product"}})
```

The first argument is the previous step (`#"Added Index"`). The second argument is a list of rename pairs: `{old name, new name}`.

---

### Task 2: Design and Trace a Merge -- Answers

---

**Question 2a -- Matching key:**

The matching column is **State**. It is the column that appears in both DIM-region and the Managers table, allowing rows to be aligned so that each city is assigned the correct regional manager for its state.

---

**Question 2b -- Join type:**

A **Left Outer Join** should be used because you want to retain all rows from DIM-region (the left table), even if a state happened to have no matching entry in the Managers table. Using an Inner Join could silently drop cities if their state was missing from the Managers table.

---

**Question 2c -- Expected output table:**

```
+----------+-----------+------------------+---------+------------------+
| id-region | City      | Population Size  | State   | Region Manager   |
+----------+-----------+------------------+---------+------------------+
|         1 | Bath      | 88,000           | England | Sarah Okafor     |
|         2 | Bristol   | 470,000          | England | Sarah Okafor     |
|         3 | Cork      | 210,000          | Ireland | Liam Brennan     |
|         4 | Galway    | 80,000           | Ireland | Liam Brennan     |
+----------+-----------+------------------+---------+------------------+
```

Both English cities receive Sarah Okafor and both Irish cities receive Liam Brennan, because the join matched on State.

---

**Question 2d -- Rename step:**

```
#"Renamed Column" = Table.RenameColumns(#"Expanded Managers", {{"Managers.Region Manager", "Region Manager"}})
```

The previous step name (`#"Expanded Managers"`) is the first argument. The rename pair maps the prefixed column name to the clean target name.

---

### Challenge Task -- Answers

---

**Part A -- Summary Label formula:**

```
= Table.AddColumn(#"Previous Step", "Summary Label", each [Customer] & " -- " & [Product])
```

The `&` operator concatenates the three text fragments for each row. The exact previous step name will depend on what precedes this step in the query.

---

**Part B -- Sales Band formula:**

```
= Table.AddColumn(#"Added Summary Label", "Sales Band", each
    if [Total Sales] >= 1000 then "High"
    else if [Total Sales] >= 500 then "Medium"
    else "Low"
)
```

Column references use square brackets. The conditions are evaluated top-down, so a value of 1000 or above hits the first branch; a value between 500 and 999 hits the second; anything below 500 falls through to "Low".

---

**Part C -- Expected output table:**

```
+------------------+------------------+-------------+-------------------------------+------------+
| Customer         | Product          | Total Sales | Summary Label                 | Sales Band |
+------------------+------------------+-------------+-------------------------------+------------+
| Acme Corp        | Widget A         |      900.00 | Acme Corp -- Widget A         | Medium     |
| Acme Corp        | Widget B         |      450.00 | Acme Corp -- Widget B         | Low        |
| Brendan Ltd      | Service Package  |     2100.00 | Brendan Ltd -- Service Package | High       |
| Clancy Supplies  | Widget A         |      300.00 | Clancy Supplies -- Widget A   | Low        |
| Clancy Supplies  | Widget B         |      750.00 | Clancy Supplies -- Widget B   | Medium     |
+------------------+------------------+-------------+-------------------------------+------------+
```

---

**Part D -- Reflection answer (model answer):**

Power Query custom columns like `Sales Band` are calculated once when the data is loaded and are then fixed in the table. A DAX measure, by contrast, recalculates at query time based on whatever filter context is currently active in the report, meaning it automatically responds to slicers, page filters, and cross-highlights applied by the report user. If a user slices by region, a DAX-based Sales Band calculation would reflect only the sales within that region, whereas the Power Query column would always show the band computed across the full unfiltered dataset, giving potentially misleading results in a filtered context.

---

*End of answers.*
