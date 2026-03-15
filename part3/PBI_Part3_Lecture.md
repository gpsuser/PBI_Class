# Power BI: Building a Dimensional Model from Raw Data

---

## Section 1: Introduction and Power BI Recap

### Overview

This section provides a brief recap of what Power BI is and what its core capabilities are. It sets the stage for the more hands-on work that follows, ensuring everyone shares a common understanding of the tool before building a dimensional model from scratch.

---

### What is Power BI?

Power BI is a business analytics platform developed by Microsoft. It enables users to connect to data sources, transform and clean data, build analytical data models, create interactive visualisations, and share insights across an organisation.

The core workflow in Power BI moves through several distinct phases:

```
+------------------+     +------------------+     +------------------+
|  Connect to Data |---->|  Transform Data  |---->|  Model Data      |
|  (Get Data)      |     |  (Power Query)   |     |  (Relationships) |
+------------------+     +------------------+     +------------------+
                                                          |
                                                          v
                                          +------------------+     +------------------+
                                          |  Visualise       |---->|  Share / Publish |
                                          |  (Reports)       |     |  (PBI Service)   |
                                          +------------------+     +------------------+
```

### Key Features

- **Data Connectivity**: Power BI connects to a wide range of sources including Excel spreadsheets, relational databases (SQL Server, MySQL, PostgreSQL), cloud services (Azure, Salesforce, Google Analytics), and flat files such as CSV.
- **Data Transformation**: The Power Query Editor allows you to clean, reshape, and prepare data before it is loaded into the data model. This step is critical and is where most data quality issues are resolved.
- **Data Modelling**: Once data is loaded, you define relationships between tables and write calculated columns or measures in DAX to enable analytical queries.
- **Visualisations**: Power BI provides a rich library of visual types (bar charts, line charts, maps, tables, KPI cards, etc.) that respond dynamically to user selections through slicers and filters.
- **Sharing and Collaboration**: Reports can be published to the Power BI Service (cloud) where colleagues can view, comment on, and interact with dashboards via a browser or the Power BI mobile app.

### The Two Languages in Power BI

Power BI uses two distinct formula languages, each serving a different purpose:

| Language | Where It Is Used | Purpose |
|---|---|---|
| M (Power Query Formula Language) | Power Query Editor | Data preparation and transformation before loading |
| DAX (Data Analysis Expressions) | Data Model (after loading) | Calculations, measures, and analytical logic |

It is important not to confuse these two languages. M prepares data; DAX analyses it. This session covers both at an introductory level.


---

## Section 2: Dimensional Modelling Recap

### Overview

Before building anything, it is important to revisit the theory behind dimensional modelling and the Star Schema. Understanding the business motivation will help you make the right design decisions when splitting a raw data table into facts and dimensions.

---

### The Business Motivation Behind Dimensional Modelling

In a typical business scenario, raw transactional data is collected in flat, wide tables. These tables contain everything in one place: customer names, product descriptions, dates, quantities, prices, and locations all sitting side by side in each row. This works fine for recording transactions, but it creates serious problems for analysis:

- **Redundancy**: Customer names, cities, and product descriptions are repeated for every transaction. A customer who places 500 orders will have their name stored 500 times.
- **Update anomalies**: If a product name changes, you must update hundreds or thousands of rows instead of one record.
- **Query performance**: Large, wide flat tables are slower to query than properly normalised structures.
- **Inflexibility**: It becomes difficult to ask questions like "What were total sales by region last quarter?" without writing complex, fragile queries.

Dimensional modelling solves these problems by organising data into two types of tables: fact tables and dimension tables.

### Fact Tables vs Dimension Tables

```
+------------------+         +------------------+
|   DIM-region     |         |   DIM-customer   |
|------------------|         |------------------|
| id-region (PK)   |         | id-customer (PK) |
| City             |         | Customer         |
| Population Size  |         | Age of Business  |
| State            |         | Service Tier     |
+------------------+         +------------------+
         |                            |
         |                            |
         v                            v
+------------------------------------------------------+
|                     FACT-sales                       |
|------------------------------------------------------|
| id-region     (FK) ----------------------------------|-->  DIM-region
| id-customer   (FK) ----------------------------------|-->  DIM-customer
| id-product    (FK) ----------------------------------|-->  DIM-product
| id-date       (FK) ----------------------------------|-->  DIM-date
| Total Spend                                          |
| Quantity                                             |
+------------------------------------------------------+
         |                            |
         v                            v
+------------------+         +------------------+
|   DIM-product    |         |   DIM-date       |
|------------------|         |------------------|
| id-product (PK)  |         | id-date (PK)     |
| Product          |         | Data Date        |
+------------------+         +------------------+
```

- **Dimension tables** hold descriptive, contextual, and categorical data. They answer the "who, what, where, when" questions. Examples include customers, products, dates, and geographic regions. Each dimension table has a unique surrogate primary key (PK).
- **Fact tables** hold quantitative, numeric, or event-driven data. They capture what happened - transactions, measurements, events. Each row in a fact table represents one business event (e.g. one sale). The fact table holds foreign keys (FK) that link to each dimension table's primary key.

### The Star Schema

The Star Schema gets its name from its visual appearance: a central fact table surrounded by dimension tables, resembling a star. It is the most widely used structure in business intelligence because it is:

- Easy for business users to understand
- Optimised for query performance in analytical tools like Power BI
- Flexible enough to support a wide variety of reporting questions

### Connections in Power BI

In Power BI, relationships between tables are defined in the **Model View**, not inside the Power Query Editor. The Query Editor is used to prepare and shape the tables; the Model View is where you draw the connections between them using drag-and-drop.

---

## Section 3: References vs Duplicates in Power Query

### Overview

When building a dimensional model from a single raw data table, the first practical question is: how do you safely create multiple working copies of that table? Power BI offers two approaches - **references** and **duplicates** - and choosing the wrong one can cause significant problems when source data is updated.

---

### Why You Need Multiple Copies

Starting from a single raw data table, you will create:
- DIM-region
- DIM-customer
- DIM-product
- DIM-date
- FACT-sales

Each of these is derived from the raw data. Rather than importing the data multiple times, you work from one source table and create linked or independent copies of it in the Power Query Editor.

### References

A **reference table** creates a live link back to the original (source) table.

- Any steps applied in the original table (filtering rows, renaming columns, changing types) are automatically inherited by the reference table.
- When the underlying data source (e.g., a CSV or Excel file) is updated and you refresh Power BI, the changes flow through the original table and then through all tables that reference it.
- This is the correct choice when you want dimension and fact tables to stay in sync with the raw source.

### Duplicates

A **duplicate table** creates a snapshot copy of the original table at the moment of duplication. It includes all the transformation steps from the original but has no ongoing link.

- Changes to the original table do not flow through to the duplicate.
- Use duplicates when you want an independent version that should not change when the source changes.
- In dimensional modelling, duplicates are generally the wrong choice because your dimension tables need to update when the raw data is refreshed.

### Summary Comparison

```
+----------------------+----------------------------------+----------------------------------+
| Feature              | Reference                        | Duplicate                        |
+----------------------+----------------------------------+----------------------------------+
| Link to original?    | Yes - live link                  | No - snapshot at time of copy    |
| Inherits steps?      | Yes - all original steps         | Yes - all steps at point of copy |
| Updates on refresh?  | Yes - inherits raw data changes  | No - remains unchanged           |
| Use case             | Dimension and fact tables        | Independent, static copies       |
+----------------------+----------------------------------+----------------------------------+
```

### Demonstrating the Update Behaviour

To see the difference in action:
1. Create a reference to the Raw Data table and name it DIM-region.
2. Open the source Excel file and add a new row of data.
3. In Power BI, click **Home > Refresh**.
4. Observe that the DIM-region table has picked up the new row automatically.

If you had used a duplicate instead, the new row would not appear until you manually recreated the copy.


---

## Section 4: Table Joins

### Overview

Before merging tables in Power Query, it is essential to understand the different types of joins available. A join is the operation that combines rows from two tables based on a matching column. Choosing the correct join type determines which rows are included in the output.

---

### What is a Table Join?

A join takes two tables - a left table and a right table - and combines them row by row wherever a specified column contains matching values. In Power BI's Power Query, this operation is called a **Merge**.

Consider two tables:

```
Table A (Left)         Table B (Right)
+----+-------+         +----+---------+
| ID | City  |         | ID | State   |
+----+-------+         +----+---------+
|  1 | Bath  |         |  1 | England |
|  2 | Cork  |         |  3 | Ireland |
|  3 | Galway|         +----+---------+
+----+-------+
```

Different join types produce different results:

### Inner Join

Returns only rows where there is a match in both tables. Rows with no match are excluded.

```
Result of Inner Join on ID:
+----+--------+---------+
| ID | City   | State   |
+----+--------+---------+
|  1 | Bath   | England |
|  3 | Galway | Ireland |
+----+--------+---------+
```

Use inner joins when you only want records that exist in both tables.

### Left Outer Join

Returns all rows from the left table, and matching rows from the right table. Where there is no match, the right table columns are filled with null.

```
Result of Left Outer Join on ID:
+----+--------+---------+
| ID | City   | State   |
+----+--------+---------+
|  1 | Bath   | England |
|  2 | Cork   | null    |
|  3 | Galway | Ireland |
+----+--------+---------+
```

This is the most commonly used join in dimensional modelling because you want to keep all records from your main table regardless of whether supplementary data exists.

### Right Outer Join

Returns all rows from the right table, and matching rows from the left table. Where there is no match on the left, those columns are filled with null.

### Full Outer Join

Returns all rows from both tables. Where there is no match on either side, the unmatched columns are null.

```
Result of Full Outer Join on ID:
+------+--------+---------+
| ID   | City   | State   |
+------+--------+---------+
|  1   | Bath   | England |
|  2   | Cork   | null    |
|  3   | Galway | Ireland |
| null | null   | Ireland |  (row 3 in B matched, but if ID 3 only in B with no match in A)
+------+--------+---------+
```

### Anti Joins

Anti joins return only the rows that do NOT have a match.

- **Left Anti Join**: Returns rows from the left table that have no match in the right table.
- **Right Anti Join**: Returns rows from the right table that have no match in the left table.

Anti joins are useful for identifying missing or orphaned records, such as finding customers in one system who do not exist in another.

### Summary of Join Types

```
+------------------+-------------------------------------------+
| Join Type        | What It Returns                           |
+------------------+-------------------------------------------+
| Inner            | Only matching rows from both tables       |
| Left Outer       | All left rows + matching right rows       |
| Right Outer      | All right rows + matching left rows       |
| Full Outer       | All rows from both tables                 |
| Left Anti        | Left rows with no match in right          |
| Right Anti       | Right rows with no match in left          |
+------------------+-------------------------------------------+
```


---

## Section 5: Organising and Hiding Source Data

### Overview

After building all your dimension and fact tables, the Power Query Editor can become cluttered with source tables that end users do not need to interact with. Power BI provides two tools for keeping things tidy: grouping tables into folders and hiding tables from the report view.

---

### Creating a Source Group in Power Query

It is good practice to group raw source tables separately from the modelled tables so anyone looking at the Query Editor can quickly distinguish source data from analytical tables.

1. In Power Query Editor, right-click in the Queries pane on an empty area
2. Select **New Group** and name it **source**
3. Drag the **Raw Data** table and the **City State** table into the **source** group
4. Click **Close and Apply**

**Before grouping:**
```
Queries
|-- Raw Data
|-- City State
|-- DIM-region
|-- DIM-customer
|-- DIM-product
|-- DIM-date
|-- FACT-sales
```

**After grouping:**
```
Queries
|-- source
|   |-- Raw Data
|   |-- City State
|-- DIM-region
|-- DIM-customer
|-- DIM-product
|-- DIM-date
|-- FACT-sales
```

### Hiding Tables from Report View

Source tables should be hidden from report authors so they do not accidentally use them for visualisations instead of the properly modelled tables.

1. Go to **Home > Data blade** (the table icon on the left side panel)
2. Right-click on **City State** and select **Hide**
3. Right-click on **Raw Data** and select **Hide**

Hidden tables remain available in the data model and the Power Query Editor, but they will not appear in the Fields pane when building reports. This prevents confusion and keeps the reporting experience clean.


---

## Section 6: Setting Up Relationships in Model View

### Overview

With all tables built and cleaned, the final step in constructing the Star Schema is to create relationships between the fact table and each dimension table in Power BI's Model View. This is where the foreign key - primary key connections are drawn.

---

### Opening Model View

Click the **Model View** icon (the diagram icon) on the left panel of the Power BI Desktop window. You will see all loaded tables displayed as cards showing their column names.

### Removing Old Relationships

Before creating new relationships, remove any auto-detected or leftover connections that Power BI may have created:

1. Select any existing relationship line
2. Press the **Delete** key to remove it

Clear all existing relationships before establishing the correct Star Schema connections.

### Creating the Star Schema Relationships

Relationships in Power BI represent many-to-one connections between the fact table (the "many" side) and each dimension table (the "one" side). Each foreign key in FACT-sales connects to a primary key in a dimension table.

**How to create a relationship:**
- Click on a foreign key column in FACT-sales (e.g., **id-region**)
- Drag it to the matching primary key in the relevant dimension table (e.g., **id-region** in DIM-region)
- Release to create the relationship

Repeat for all four dimension tables:

```
FACT-sales.id-region    --[*:1]--> DIM-region.id-region
FACT-sales.id-customer  --[*:1]--> DIM-customer.id-customer
FACT-sales.id-product   --[*:1]--> DIM-product.id-product
FACT-sales.id-date      --[*:1]--> DIM-date.id-date
```

After creating all four connections, the Model View should show a clear star pattern with FACT-sales at the centre.

```
         DIM-date
             |
             | *:1
             |
DIM-region ------ FACT-sales ------ DIM-customer
             |
             | *:1
             |
         DIM-product
```

### Why These Relationship Directions Matter

Power BI uses the relationship direction to determine how filters flow. In a standard Star Schema:
- Filters applied to a dimension (e.g., filtering by a specific region) flow downstream into the fact table, restricting which rows are aggregated.
- This enables all of your measures and aggregations to automatically respond to slicers and filters in a report.


---

## Section 7: Introduction to M Language

### Overview

Every transformation you apply in the Power Query Editor - renaming a column, removing blanks, merging tables - is automatically recorded as M language code behind the scenes. Understanding M helps you understand what Power Query is doing, allows you to inspect and debug transformations, and opens the door to writing custom logic that the graphical interface cannot handle directly.

---

### What is M?

M (formally the Power Query Formula Language) is a functional, case-sensitive scripting language designed specifically for data preparation and transformation. It is quite different from DAX:

```
+---------------------+-------------------------------+-------------------------------+
| Feature             | M (Power Query)               | DAX                           |
+---------------------+-------------------------------+-------------------------------+
| Where it runs       | Power Query Editor            | Data Model                    |
| When it runs        | Before data is loaded         | After data is loaded          |
| What it does        | Cleans and shapes data        | Calculates analytical values  |
| Similar to          | Functional programming        | Excel formula language         |
| Auto-generated?     | Yes - by UI interactions      | Mostly written manually        |
+---------------------+-------------------------------+-------------------------------+
```

### Seeing M Code in the Formula Bar

When you click on any step in the **Applied Steps** pane (on the right side of the Power Query Editor), the M expression for that step appears in the formula bar at the top of the editor.

For example, clicking on a "Removed Duplicates" step might show:

```
= Table.Distinct(#"Changed Type", {"City"})
```

You can expand the formula bar by clicking the down arrow on its right side to see more of the expression.

### The Advanced Editor

The **Advanced Editor** (found under the **Home** or **View** tab in Power Query) shows the complete M script for the entire query as a sequence of let...in statements:

```
let
    Source = Excel.Workbook(File.Contents("C:\Data\Raw Data.xlsx"), null, true),
    RawData_Sheet = Source{[Item="RawData",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(RawData_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"City", type text}}),
    #"Removed Duplicates" = Table.Distinct(#"Changed Type", {"City"}),
    #"Added Index" = Table.AddIndexColumn(#"Removed Duplicates", "Index", 1, 1, Int64.Type)
in
    #"Added Index"
```

Each step builds on the output of the previous step, referenced by name in the `let` block.

### Adding a Custom Column with M

You can create a new column by combining values from existing columns using the **Add Column > Custom Column** feature. Behind the scenes, this generates M code.

**Example - Concatenating Customer and Product names:**

In the Power Query Editor, with **TotSalesByCustomerByProduct** selected:

1. Go to **Add Column tab > Custom Column**
2. New Column Name: **Customer and Product**
3. In the formula box, enter: `= [Customer] & " " & [Product]`
4. Click **OK**

The M code generated and visible in the formula bar for this step is:

```
= Table.AddColumn(#"Renamed Columns", "Customer and Product", each [Customer] & " " & [Product])
```

**Sample output:**

```
+------------------+------------------+---------------------------+
| Customer         | Product          | Customer and Product      |
+------------------+------------------+---------------------------+
| Acme Corp        | Widget A         | Acme Corp Widget A        |
| Acme Corp        | Widget B         | Acme Corp Widget B        |
| Brendan Ltd      | Service Package  | Brendan Ltd Service Pack  |
+------------------+------------------+---------------------------+
```

### Key M Language Concepts

- **Each keyword**: The `each` keyword in M is shorthand for a function that takes one argument (the current row). `each [Customer]` means "for each row, return the value in the Customer column".
- **Column references**: Columns are referenced using square brackets: `[ColumnName]`
- **String concatenation**: The `&` operator joins text values
- **Step references**: Steps are referenced using `#"Step Name"` when the step name contains spaces
- **Case sensitivity**: M is case-sensitive. `table.addcolumn` will fail; it must be `Table.AddColumn`

### M is Mostly Auto-Generated

For most tasks in Power BI, you will not need to write M from scratch. The graphical interface generates it for you. However, being able to read M code is valuable because:
- It helps you debug unexpected query behaviour
- It allows you to make precise edits that are difficult through the interface alone
- It gives you insight into what Power BI is actually doing with your data at each step


---

## Section 8: Conclusion and Further Resources

### Overview

This section summarises everything covered in the session and points to resources for further practice and deeper learning.

---

### What Was Covered

Starting from a single flat raw data table, you have learned and applied the following:

1. **Dimensional Modelling Theory**: The business motivation behind separating data into fact and dimension tables, and the structure of a Star Schema.

2. **References vs Duplicates**: How to safely create multiple working copies of a source table in Power Query, and why reference tables are the correct choice for dimensional modelling.

3. **Table Joins**: The six join types available in Power BI (Inner, Left Outer, Right Outer, Full Outer, Left Anti, Right Anti) and when to use each.

4. **Creating Dimension Tables**: How to build DIM-region, DIM-customer, DIM-product, and DIM-date by selecting relevant columns, removing duplicates, and adding surrogate keys.

5. **Enriching Dimension Tables**: How to bring in additional data from a secondary file using a Merge (Left Outer Join) within Power Query.

6. **Building the Fact Table**: How to create FACT-sales and populate it with foreign keys by merging surrogate keys from each dimension table, then removing the original descriptive columns.

7. **Organising the Model**: Grouping source tables into folders and hiding them from the report view.

8. **Model View Relationships**: Drawing many-to-one connections between FACT-sales and each dimension table to complete the Star Schema.

9. **Aggregation Without DAX**: Using Group By in Power Query to create pre-aggregated summary tables, and enriching them with labels via Merge operations.

10. **M Language Basics**: Understanding that every Power Query step is backed by auto-generated M code, how to read it in the formula bar and Advanced Editor, and how to write a simple custom column formula.

---

### The Single Most Important Rule

**Practice.**

Building a Star Schema from raw data is a skill that improves with repetition. The steps become second nature quickly, but only if you work through them yourself with real data. Every time you apply these techniques to a new dataset, you will develop a sharper instinct for which joins to use, which columns belong in facts versus dimensions, and how to handle the edge cases that real-world data always presents.

---

### DAX Reference - Key Measure Patterns

Although this session focused on the model building layer, here are some DAX measure patterns that you will commonly apply once your Star Schema is in place.

**Total Sales Measure:**

```
Total Sales =
SUM ( 'FACT-sales'[Total Spend] )
```

**Average Sales Per Customer:**

```
Avg Sales Per Customer =
DIVIDE (
    SUM ( 'FACT-sales'[Total Spend] ),
    DISTINCTCOUNT ( 'FACT-sales'[id-customer] )
)
```

**Sales Filtered to a Specific Region:**

```
Sales - England =
CALCULATE (
    SUM ( 'FACT-sales'[Total Spend] ),
    'DIM-region'[State] = "England"
)
```

**Year-to-Date Sales (requires a proper date dimension with a date column):**

```
Sales YTD =
TOTALYTD (
    SUM ( 'FACT-sales'[Total Spend] ),
    'DIM-date'[Data Date]
)
```

These measures will work correctly because of the relationships you established in Model View. The Star Schema ensures that filters applied to dimension tables automatically flow into the fact table aggregations.

---

### Further Reading and Practice Resources

The following resources provide further depth on the topics covered in this session. No video resources are included - all are text-based documentation, tutorials, and practice materials.

**Official Microsoft Documentation**

- Power Query M Language Specification:
  https://learn.microsoft.com/en-us/powerquery-m/power-query-m-language-specification

- Power Query documentation (overview and function reference):
  https://learn.microsoft.com/en-us/power-query/

- DAX reference documentation:
  https://learn.microsoft.com/en-us/dax/

- Relationships in Power BI Desktop:
  https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-relationships-understand

- Data modelling best practices in Power BI:
  https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

**Practice Datasets**

- Adventure Works DW sample (.pbix file with a pre-built dimensional model):
  https://github.com/microsoft/powerbi-desktop-samples

- Contoso sample database (another widely used practice dataset):
  https://www.microsoft.com/en-us/download/details.aspx?id=18279

**DAX Reference and Practice**

- DAX Guide (community-maintained, comprehensive function reference with examples):
  https://dax.guide/

- SQLBI DAX Patterns (practical, real-world DAX patterns with explanations):
  https://www.daxpatterns.com/

- DAX Formatter (formats and validates your DAX code):
  https://www.daxformatter.com/

**Dimensional Modelling Theory**

- Ralph Kimball's dimensional modelling techniques (foundational theory):
  https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/

**Community and Forums**

- Power BI Community forums (post questions, browse solved examples):
  https://community.powerbi.com/

- Microsoft Fabric and Power BI blog (official announcements and how-to articles):
  https://powerbi.microsoft.com/en-us/blog/


---

*End of lecture notes.*
