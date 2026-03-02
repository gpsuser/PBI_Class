# Power BI - Part 2: Data Modelling, Star Schemas, and DAX

---

## Table of Contents

1. [Section 1: Data Modelling Concepts](#section-1-data-modelling-concepts)
2. [Section 2: Dimensional Modelling](#section-2-dimensional-modelling)
3. [Section 3: The Business Case for Dimensional Modelling](#section-3-the-business-case-for-dimensional-modelling)
4. [Section 4: Designing a Dimensional Data Model](#section-4-designing-a-dimensional-data-model)
5. [Section 5: Implementing the Star Schema in Power BI](#section-5-implementing-the-star-schema-in-power-bi)
6. [Section 6: Extending Dimensions](#section-6-extending-dimensions)
7. [Section 7: Creating Measures with DAX](#section-7-creating-measures-with-dax)
8. [Section 8: DAX Aggregation within a Dimensional Model](#section-8-dax-aggregation-within-a-dimensional-model)
9. [Section 9: Generating Reports](#section-9-generating-reports)
10. [Section 10: Hands-On Lab](#section-10-hands-on-lab)
11. [Further Reading and Resources](#further-reading-and-resources)

---

## Section 1: Data Modelling Concepts

**Overview:** This section recaps core data modelling concepts, introducing the idea of a data warehouse, data models, and multidimensional schema as the foundation for everything that follows.

### What is a Data Warehouse?

A data warehouse is a large, centralised store of structured data, designed primarily for analytics and reporting rather than for day-to-day transaction processing. Unlike an operational database (which is optimised for inserts, updates, and deletes), a data warehouse is optimised for reading and querying large volumes of historical data.

Key characteristics of a data warehouse:
- Subject-oriented: data is organised around key business subjects such as sales, products, and customers
- Integrated: data is pulled from multiple source systems and standardised
- Non-volatile: once data is loaded, it is not frequently changed
- Time-variant: historical data is retained so trends can be analysed over time

### What is a Data Model?

A data model is the structured and logical organisation of data, showing the relationships between data elements. A good data model makes it easy to query the data, maintain it, and build reliable reports on top of it.

### Multidimensional Schema

A multidimensional schema is a way of modelling different dimensions to track entities and actions within a data warehouse. This approach is commonly referred to as **dimensional modelling**.

The most widely used and accessible starting point for dimensional modelling is the **Star Schema**.

### ETL vs ELT Architectures

Data typically arrives in a data warehouse through one of two architectural patterns:

**On-Premises Pattern - ETL (Extract, Transform, Load)**

```
Source Systems --> [Extract] --> [Transform] --> [Load] --> Data Warehouse
```

In ETL, data is extracted from source systems, transformed (cleaned, reshaped, enriched) before it arrives in the warehouse. This is the traditional on-premises approach.

**Cloud Pattern - ELT (Extract, Load, Transform)**

```
Source Systems --> [Extract] --> [Load] --> Cloud Storage --> [Transform] --> Data Warehouse
```

In ELT, raw data is loaded into cloud storage first, and transformation happens within the cloud environment. This is the modern cloud-based approach, taking advantage of cheap storage and scalable compute.

Power BI can connect to data warehouses built on either pattern.

---

[Back to Table of Contents](#table-of-contents)

---

## Section 2: Dimensional Modelling

**Overview:** This section explains the core concepts of dimensional modelling: normalisation, denormalisation, keys, and the mechanics of transforming a single flat table into a proper star schema.

### Starting Point: The Denormalised Table

In many real-world data scenarios, data begins life as a single, flat, denormalised table. All information is stored in one place, resulting in repeated or redundant data.

Example raw (denormalised) data:

```
SaleID | ProductName | CustomerName | SaleDate   | Amount
-------|-------------|--------------|------------|-------
1      | Widget A    | Alice        | 2024-01-01 | 100.00
2      | Widget B    | Bob          | 2024-01-02 | 150.00
3      | Widget C    | Charlie      | 2024-01-03 | 200.00
4      | Widget A    | David        | 2024-01-04 | 100.00
5      | Widget B    | Eve          | 2024-01-05 | 150.00
```

Notice that "Widget A" and "Widget B" appear more than once. If the product name ever changed, every row containing that product would need to be updated. This is the problem of redundancy.

### Normalisation

Normalisation is the process of organising data to reduce redundancy. The resulting tables are said to be in **Normal Form**. There are varying degrees of normal form (1NF, 2NF, 3NF, and beyond), each addressing a different type of data anomaly.

Dimensional modelling applies normalisation by splitting the single flat table into:
- One central **Fact Table**
- Multiple surrounding **Dimension Tables**

### Keys

Keys are columns that establish relationships between tables:

- **Primary Key (PK):** A unique identifier for each row in a table. Every dimension table and the fact table has a primary key.
- **Foreign Key (FK):** A column in one table that references the primary key of another table. The fact table holds foreign keys that point to each dimension table.

### The Process: From One Table to Many

The transformation approach is:

1. Start with a single denormalised table
2. Identify the distinct entities: products, customers, dates, cities, etc.
3. Extract each entity into its own dimension table with a primary key
4. Replace the descriptive columns in the original table with foreign key references
5. The resulting central table becomes the Fact table

---

[Back to Table of Contents](#table-of-contents)

---

## Section 3: The Business Case for Dimensional Modelling

**Overview:** This section explains why dimensional models are the preferred structure for business intelligence and reporting, covering seven key business justifications.

Dimensional models are widely used in data warehousing and business intelligence because they deliver tangible benefits across performance, management, analysis, and usability.

### 1. Improved Query Performance

Fact tables store large volumes of transactional data, while dimension tables store descriptive attributes. This separation allows the database engine to efficiently join smaller dimension tables with the fact table, rather than scanning a single enormous flat file.

### 2. Simplified Data Management

By organising data into fact and dimension tables, redundancy is reduced and consistency is improved. Dimension tables centralise descriptive data, so a product name change only needs to be made in one place.

### 3. Enhanced Data Analysis

Star schemas allow analysts to easily slice and dice data along different dimensions such as time, product, and customer. This flexibility makes complex multi-dimensional queries straightforward to write.

### 4. Scalability

Fact tables can grow significantly over time as new transactions occur, but dimension tables typically remain relatively small. This structure supports scalability, allowing the data warehouse to handle large data volumes without performance degradation.

### 5. Flexibility

Star schemas allow new dimensions or facts to be added without disrupting existing queries and reports. This makes the model adaptable to changing business requirements.

### 6. Data Integrity

Normalising data into fact and dimension tables reduces the risk of inconsistencies. Each piece of descriptive information is stored once, in a dedicated table.

### 7. User-Friendly Structure

Star schemas are intuitive and easier for business users to understand. The clear separation between facts (what happened) and dimensions (the context of what happened) aligns well with how people naturally think about their data.

---

[Back to Table of Contents](#table-of-contents)

---

## Section 4: Designing a Dimensional Data Model

**Overview:** This section walks through the conceptual design of a star schema, covering the roles of the fact table and dimension tables, and illustrating the structure with a worked example.

### The Star Schema Structure

The star schema gets its name from the visual layout: a central fact table surrounded by dimension tables, connected by keys - resembling a star.

```
                             +------------------+
                             |   Dim_Date       |
                             |  PK: DateID      |
                             +--------+---------+
                                      |
                                      |
+------------------+        +---------+---------+        +------------------+
|  Dim_Customer    |        |   Fact_Sales      |        |   Dim_Product    |
|  PK: CustomerID  +--------+  PK: SaleID       +--------+  PK: ProductID   |
+------------------+        |  FK: CustomerID   |        +------------------+
                            |  FK: ProductID    |
                            |  FK: DateID       |
                            |  FK: CityID       |
                            |  Amount           |
                            |  Items            |
                            |  Total Spend      |
                            +---------+---------+
                                      |
                                      |
                             +--------+---------+
                             |   Dim_City       |
                             |  PK: CityID      |
                             +------------------+
```

### The Fact Table

The fact table is the central table in a star schema. It:
- Contains measurable information relating to a business event (such as a sales transaction)
- Can contain quantitative information (amounts, quantities, counts)
- Can also contain string information
- Holds a PRIMARY KEY and multiple FOREIGN KEYS
- Connects to all dimension tables through those FOREIGN KEYS

Example Fact Table (Fact_Sales):

```
SaleID | ProductID | CustomerID | SaleDate   | Amount | Items | Total Spend
-------|-----------|------------|------------|--------|-------|------------
1      | 1         | 1          | 2024-01-01 | 100.00 | 10    | 1000.00
2      | 2         | 2          | 2024-01-02 | 150.00 | 30    | 4500.00
3      | 3         | 3          | 2024-01-03 | 200.00 | 5     | 1000.00
4      | 1         | 4          | 2024-01-04 | 100.00 | 28    | 2800.00
5      | 2         | 5          | 2024-01-05 | 150.00 | 12    | 1800.00
```

### Dimension Tables

Dimension tables surround the fact table and provide descriptive context. They:
- Hold PRIMARY KEYS that connect to FOREIGN KEYS in the fact table
- Contain descriptive, qualitative information about the entity they represent

Example Dimension Tables:

**Dim_Product:**
```
ProductID | ProductName
----------|------------
1         | Widget A
2         | Widget B
3         | Widget C
```

**Dim_Customer:**
```
CustomerID | CustomerName
-----------|-------------
1          | Alice
2          | Bob
3          | Charlie
4          | David
5          | Eve
```

---

[Back to Table of Contents](#table-of-contents)

---

## Section 5: Implementing the Star Schema in Power BI

**Overview:** This section covers the practical steps for building a star schema in Power BI, including importing data, managing relationships, and correctly structuring the model view.

### Downloading the Data

The data files for this session are available at:

```
https://github.com/gpsuser/PBI_Class
data > Part2 data.zip
```

Download and extract the zip file before starting.

### Importing Data into Power BI

Power BI Desktop provides a straightforward import process for Excel workbooks.

**Importing the Raw Data file:**
1. Go to Home > Get Data > Excel Workbook
2. Navigate to the project folder
3. Select the Raw Data file > Open
4. Select the Raw Data sheet > Transform Data
5. In Power Query, locate the DateTime column > Change Data Type > Date/Time > Replace Current
6. Click Close & Apply > Close & Apply

**Importing the Fact Sales file:**
1. Go to Home > Get Data > Excel Workbook
2. Navigate to the project folder
3. Select the Fact Sales file > Open
4. Select the Fact Sales sheet > Transform Data
5. In Power Query, locate the DateTime column > Change Data Type > Date/Time > Replace Current
6. Click Close & Apply > Close & Apply

Repeat the import process for each dimension table (Dim_Customer, Dim_Product, Dim_City, Dim_Date, etc.) included in the data package.

### Managing Relationships in Power BI

When multiple tables are imported, Power BI will attempt to auto-detect relationships. In practice, you will often need to manually disconnect automatically created relationships and recreate them correctly to match the intended star schema design.

**To manage relationships:**
1. Go to the Model view in Power BI Desktop
2. Review any automatically created relationships
3. Delete any incorrect or unwanted relationships by right-clicking and selecting Delete
4. Recreate relationships by dragging a column from a dimension table's primary key to the matching foreign key column in the fact table

The goal is to end up with a clean star schema where the fact table sits at the centre and each dimension table connects to it through the appropriate key columns.

```
Dim_Customer.CustomerID  <----> Fact_Sales.CustomerID
Dim_Product.ProductID    <----> Fact_Sales.ProductID
Dim_Date.DateID          <----> Fact_Sales.SaleDate
Dim_City.CityID          <----> Fact_Sales.CityID
```

All relationships in a standard star schema flow from the dimension table (the "one" side) to the fact table (the "many" side), creating one-to-many relationships.

---

[Back to Table of Contents](#table-of-contents)

---

## Section 6: Extending Dimensions

**Overview:** This section covers how to enrich dimension tables with additional derived or calculated columns, with a focus on the Date dimension as a practical and common example.

### Why Extend Dimensions?

Source data often contains minimal information. For example, a date column in the fact table stores a raw date value, but for reporting you may need the year, month name, quarter, day of the week, and more. Rather than performing these calculations repeatedly in every report, they can be derived once and stored as additional columns in the dimension table.

### The Extended Date Dimension

The Date dimension is one of the most commonly extended dimensions in any data model. An extended Dim_Date table might include:

```
DateID | Date       | Year | Quarter | MonthNumber | MonthName  | DayOfWeek | IsWeekend
-------|------------|------|---------|-------------|------------|-----------|----------
1      | 2024-01-01 | 2024 | Q1      | 1           | January    | Monday    | No
2      | 2024-01-02 | 2024 | Q1      | 1           | January    | Tuesday   | No
...
```

In Power BI, you can extend a dimension table by:
- Adding calculated columns in the Data view using DAX
- Adding columns during the Power Query transformation stage

### Adding a Calculated Column in DAX

To add a Month Name column to Dim_Date in DAX:

```dax
MonthName = FORMAT(Dim_Date[Date], "MMMM")
```

To add a Year column:

```dax
Year = YEAR(Dim_Date[Date])
```

To add a Quarter column:

```dax
Quarter = "Q" & QUARTER(Dim_Date[Date])
```

To add a Day of Week column:

```dax
DayOfWeek = FORMAT(Dim_Date[Date], "dddd")
```

These additional columns become available in reports as filters and slicers, greatly enhancing the analytical capabilities of the model without any changes to the fact table.

---

[Back to Table of Contents](#table-of-contents)

---

## Section 7: Creating Measures with DAX

**Overview:** This section introduces DAX Measures: what they are, how to create them, and how to use them in Power BI reports. Several practical measure examples are covered.

### What is a Measure?

A Measure is a DAX calculation that is evaluated dynamically in the context of a report. Unlike a calculated column (which computes a value for each row and stores it in the table), a measure is computed on the fly when it is used in a visual, responding to any filters or slicers applied.

Measures are stored in the table they are associated with, but they do not add a column to that table.

### Creating a Measure

To create a measure in Power BI:

1. In the Data panel on the right, navigate to and click on the Fact table
2. Right-click the Fact table > New Measure
3. Type your DAX expression in the formula bar
4. Press Enter

### Measure Example 1: Total Spend for Platinum Tier Customers

This measure calculates the total spend for customers whose service tier is "Platinum":

```dax
msr_platinum_tier =
CALCULATE(
    SUM(Fact_Sales[Total Spend]),
    FILTER(Dim_Customer, [Service Tier] = "Platinum")
)
```

**How it works:**
- `SUM(Fact_Sales[Total Spend])` - sums the Total Spend column from the Fact_Sales table
- `CALCULATE(...)` - evaluates the expression within a modified filter context
- `FILTER(Dim_Customer, [Service Tier] = "Platinum")` - restricts the calculation to only rows where the Service Tier in Dim_Customer equals "Platinum"

**To use this measure:**
- Go to Page 1
- Insert a Card visual
- Drag and drop the measure onto the Card visual's Fields well

Sample output (Card visual):

```
+--------------------+
|  msr_platinum_tier |
|      84,320.00     |
+--------------------+
```

### Measure Example 2: Count of Customers Buying Specific Products with High Item Counts

This measure counts the number of distinct customers who purchased Product ID 1 or Product ID 2 and where the Items quantity was greater than 25:

```dax
msr_customers_prod_items =
CALCULATE(
    COUNT(Fact_Sales[Customer ID]),
    FILTER(
        Fact_Sales,
        (Fact_Sales[Product ID] = 2 || Fact_Sales[Product ID] = 1)
        && Fact_Sales[Items] > 25
    )
)
```

**How it works:**
- `COUNT(Fact_Sales[Customer ID])` - counts non-blank values in the Customer ID column
- `FILTER(Fact_Sales, ...)` - applies a row-level filter to the Fact_Sales table
- The filter condition combines two criteria using `||` (OR) and `&&` (AND) operators:
  - Product ID must be 1 or 2
  - Items count must be greater than 25

**To use this measure:**
- Go to Page 1
- Insert a Card visual
- Drag and drop the measure onto the Card Fields well

Sample output (Card visual):

```
+---------------------------+
| msr_customers_prod_items  |
|            47             |
+---------------------------+
```

### Key DAX Functions Used in Measures

| Function    | Purpose                                                              |
|-------------|----------------------------------------------------------------------|
| CALCULATE   | Evaluates an expression in a modified filter context                 |
| SUM         | Returns the sum of all values in a column                            |
| COUNT       | Counts the number of non-blank values in a column                    |
| FILTER      | Returns a table that is a subset of another table based on a condition|
| RELATED     | Returns a related value from another table (used across relationships)|

---

[Back to Table of Contents](#table-of-contents)

---

## Section 8: DAX Aggregation within a Dimensional Model

**Overview:** This section covers how to write DAX queries that perform grouped aggregations across multiple dimension tables, using VAR blocks, ADDCOLUMNS, SUMMARIZE, FILTER, and RELATED to build complex analytical tables directly in DAX.

### The Aggregation Challenge

Sometimes you need to aggregate data across multiple dimensions while applying filters from related dimension tables. For example:

> Group by Customer and Product, calculate the sum of Total Spend, but only include sales where the city's population is greater than 200,000 - sorted by customer and then by product.

This crosses the Fact_Sales table with both Dim_Customer, Dim_Product, and Dim_City simultaneously.

### Key DAX Functions for Aggregation

| Function      | Purpose                                                                         |
|---------------|---------------------------------------------------------------------------------|
| VAR           | Declares a named variable to store an intermediate result                       |
| RETURN        | Specifies the final output of a VAR block                                       |
| FILTER        | Returns a filtered subset of a table                                            |
| ADDCOLUMNS    | Adds calculated columns to an existing table (in memory, not stored)            |
| RELATED       | Retrieves a value from a related table via the model relationship               |
| SUMMARIZE     | Groups a table by specified columns and computes aggregated expressions         |
| SUM           | Sums a numeric column                                                           |

### Building the Aggregation Step by Step

**Step 1: Add a column from a related dimension table**

To filter by city population, the population size from Dim_City needs to be brought into the context of Fact_Sales rows. `ADDCOLUMNS` can extend a table in memory without altering the underlying table:

```dax
ADDCOLUMNS(
    Fact_Sales,
    "Population Size", RELATED(Dim_City[Population Size])
)
```

**Step 2: Filter the extended table**

Wrap the ADDCOLUMNS expression inside FILTER to restrict to cities with a population over 200,000:

```dax
FILTER(
    ADDCOLUMNS(
        Fact_Sales,
        "Population Size", RELATED(Dim_City[Population Size])
    ),
    [Population Size] > 200000
)
```

**Step 3: Group and aggregate**

Use SUMMARIZE to group the filtered result by Customer and Product, and compute the Total Spend:

```dax
SUMMARIZE(
    FilteredSales,
    Dim_Customer[Customer],
    Dim_Product[Product],
    "Total Spend", SUM(Fact_Sales[Total Spend])
)
```

### Creating a Calculated Table from DAX

The full DAX expression to create a calculated table named `tbl_GroupByTest` (without sorting):

```dax
tbl_GroupByTest =
VAR FilteredSales =
    FILTER(
        ADDCOLUMNS(
            Fact_Sales,
            "Population Size", RELATED(Dim_City[Population Size])
        ),
        [Population Size] > 200000
    )

VAR GroupedSales =
    SUMMARIZE(
        FilteredSales,
        Dim_Customer[Customer],
        Dim_Product[Product],
        "Total Spend", SUM(Fact_Sales[Total Spend])
    )

RETURN
    GroupedSales
```

To create this table in Power BI:
1. Go to Table tools (or Modeling tab)
2. Select New Table
3. Enter the DAX expression above into the formula bar

Sample output:

```
Customer   | Product   | Total Spend
-----------|-----------|------------
Customer A | Product A |       2542
Customer C | Product B |      18381
Customer C | Product E |       5474
Customer E | Product D |      16082
Customer H | Product E |      12748
Customer I | Product E |        379
Customer J | Product B |       9033
Customer J | Product C |       4722
```

### Adding Sorting to the Result

To produce the same result but sorted by Customer ascending and then Product ascending, the query version with ORDER BY can be used in the DAX query editor (via External Tools such as DAX Studio):

```dax
EVALUATE
VAR FilteredSales =
    FILTER(
        ADDCOLUMNS(
            Fact_Sales,
            "Population Size", RELATED(Dim_City[Population Size])
        ),
        [Population Size] > 200000
    )

VAR GroupedSales =
    SUMMARIZE(
        FilteredSales,
        Dim_Customer[Customer],
        Dim_Product[Product],
        "Total Spend", SUM(Fact_Sales[Total Spend])
    )

RETURN
    GroupedSales
ORDER BY
    Dim_Customer[Customer] ASC,
    Dim_Product[Product] ASC
```

Note: `ORDER BY` is a DAX query clause and is supported in DAX Studio or the Power BI performance analyser. When creating calculated tables directly in Power BI, `ADDCOLUMNS` with `SELECTCOLUMNS` or `TOPN` may be used instead to control ordering.

---

[Back to Table of Contents](#table-of-contents)

---

## Section 9: Generating Reports

**Overview:** This section covers how to build Power BI reports on top of the dimensional model, using calculated measures and the relationships established in the star schema to create meaningful visuals.

### Report Building Principles

Once the star schema is in place and measures have been defined, Power BI reports are built in the Report view. Key principles:

- **Visuals pull data from the model:** When you place a field or measure into a visual, Power BI uses the relationships in the model to aggregate data correctly across dimension boundaries.
- **Filters apply across the model:** A slicer on Dim_Date[Year] will automatically filter every visual on the page that uses data from Fact_Sales, because the relationship is defined in the model.
- **Measures respond to context:** The value shown in a measure will automatically update based on any filters or slicers applied to the report page.

### Suggested Report Structure

A simple sales report using the star schema might include:

**Page 1 - Summary KPIs**
- Card visual: `msr_platinum_tier` (Total spend for Platinum customers)
- Card visual: `msr_customers_prod_items` (Customer count for products 1 & 2 with high item volumes)
- Card visual: Total Sales (a basic SUM measure)

To add a Card visual:
1. In the Report view, select the Card visual from the Visualisations pane
2. Drag a measure from the Data panel into the Fields well of the Card
3. Format as required using the Format pane

**Page 2 - Sales by Dimension**
- Table or Matrix visual showing Sales by Customer and Product
- Slicer on Dim_Date[Year] or Dim_Date[MonthName] to filter by time
- Bar chart showing Total Spend by Product

### Using the Calculated Table in a Report

The `tbl_GroupByTest` calculated table created in the previous section can also be used directly as a data source for a Table visual:

1. In the Report view, insert a Table visual
2. Drag Dim_Customer[Customer], Dim_Product[Product], and the Total Spend column from `tbl_GroupByTest` into the visual's Fields well

This produces a table in the report showing aggregated spend by customer and product, filtered to cities with a population over 200,000.

---

[Back to Table of Contents](#table-of-contents)

---

## Section 10: Hands-On Lab

**Overview:** This section describes the practical lab activities that bring together everything covered in the lecture, giving hands-on experience building a complete star schema, writing DAX measures, and generating a report.

### Lab Setup

Download the data files from:

```
https://github.com/gpsuser/PBI_Class
data > Part2 data.zip
```

### Lab Tasks

**Task 1: Create a Star Schema in Power BI**

1. Open Power BI Desktop
2. Import each of the provided data files (Raw Data, Fact Sales, and all dimension tables)
3. Open the Model view and verify the structure of each table
4. Disconnect any automatically generated relationships that are incorrect
5. Recreate the correct relationships between the Fact table and each Dimension table:
   - Fact_Sales.ProductID to Dim_Product.ProductID
   - Fact_Sales.CustomerID to Dim_Customer.CustomerID
   - Fact_Sales.SaleDate to Dim_Date.DateID
   - Fact_Sales.CityID to Dim_City.CityID

Expected Model view layout:

```
  Dim_Date   Dim_Product   Dim_Customer   Dim_City
      \           |              |           /
       \          |              |          /
        \         |              |         /
         +--------+--------------+---------+
                      Fact_Sales
```

**Task 2: Extend the Date Dimension**

Add the following calculated columns to Dim_Date using DAX:

```dax
Year = YEAR(Dim_Date[Date])
```

```dax
MonthName = FORMAT(Dim_Date[Date], "MMMM")
```

```dax
Quarter = "Q" & QUARTER(Dim_Date[Date])
```

**Task 3: Create Measures**

In the Fact_Sales table, create the following measures:

Measure 1 - Total Spend for Platinum tier customers:

```dax
msr_platinum_tier =
CALCULATE(
    SUM(Fact_Sales[Total Spend]),
    FILTER(Dim_Customer, [Service Tier] = "Platinum")
)
```

Measure 2 - Count of customers buying Products 1 or 2 with more than 25 items:

```dax
msr_customers_prod_items =
CALCULATE(
    COUNT(Fact_Sales[Customer ID]),
    FILTER(
        Fact_Sales,
        (Fact_Sales[Product ID] = 2 || Fact_Sales[Product ID] = 1)
        && Fact_Sales[Items] > 25
    )
)
```

Display each measure using a Card visual on Page 1.

**Task 4: Create a Table Using DAX**

Create a new calculated table named `tbl_GroupByTest`:

```dax
tbl_GroupByTest =
VAR FilteredSales =
    FILTER(
        ADDCOLUMNS(
            Fact_Sales,
            "Population Size", RELATED(Dim_City[Population Size])
        ),
        [Population Size] > 200000
    )

VAR GroupedSales =
    SUMMARIZE(
        FilteredSales,
        Dim_Customer[Customer],
        Dim_Product[Product],
        "Total Spend", SUM(Fact_Sales[Total Spend])
    )

RETURN
    GroupedSales
```

Add a Table visual to the report and display the results.

**Task 5: Generate a Report**

Build a report page that includes:
- Two Card visuals showing the measures created in Task 3
- A Table visual showing the results of `tbl_GroupByTest`
- A slicer based on Dim_Date[Year] or Dim_Date[MonthName]
- A Bar Chart showing Total Spend by Product

---

[Back to Table of Contents](#table-of-contents)

---

## Further Reading and Resources

The following resources provide further reading, documentation, and practice opportunities. No video resources are listed.

### Official Documentation

- **Microsoft DAX Reference** - Complete reference for all DAX functions, syntax, and examples:
  https://learn.microsoft.com/en-us/dax/

- **Power BI documentation** - Official Microsoft documentation covering all aspects of Power BI Desktop and the Power BI service:
  https://learn.microsoft.com/en-us/power-bi/

- **Data Analysis Expressions (DAX) in Power BI Desktop** - Getting started guide:
  https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-quickstart-learn-dax-basics

- **Create and manage relationships in Power BI Desktop** - How to build and manage model relationships:
  https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-create-and-manage-relationships

- **Star Schema and the importance for Power BI** - Microsoft guidance on dimensional modelling in Power BI:
  https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

### DAX Practice and Reference

- **DAX Guide** - Community-driven reference covering every DAX function with examples and notes:
  https://dax.guide/

- **SQLBI DAX Patterns** - A library of common analytical patterns implemented in DAX, covering running totals, year-to-date, ranking, and more:
  https://www.daxpatterns.com/

- **DAX Formatter** - A free online tool that formats DAX code for readability:
  https://www.daxformatter.com/

### Data Modelling Background

- **The Data Warehouse Toolkit (Kimball & Ross)** - The foundational text on dimensional modelling. The associated website provides articles and patterns:
  https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/

- **SQLShack - Star Schema in Data Warehouse modelling** - A readable overview of star schema design principles:
  https://www.sqlshack.com/star-schema-in-data-warehouse-modeling/

### Practice Datasets

- **Microsoft sample datasets for Power BI** - Official sample .pbix files and Excel datasets for practice:
  https://learn.microsoft.com/en-us/power-bi/create-reports/sample-datasets

- **Kaggle public datasets** - Large library of freely available datasets for building practice data models:
  https://www.kaggle.com/datasets

### Community

- **Power BI Community forums** - Active community for asking questions and finding solutions:
  https://community.fabric.microsoft.com/t5/Power-BI-forums/ct-p/pbi_en

- **SQLBI Blog** - In-depth technical articles on DAX, Power BI, and data modelling:
  https://www.sqlbi.com/articles/

---

[Back to Table of Contents](#table-of-contents)