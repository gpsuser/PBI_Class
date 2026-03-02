# DAX Quick Reference Cheat Sheet

## Calculated Columns vs. Measures

| Calculated Columns | Measures |
|-------------------|----------|
| Computed row-by-row | Computed on-demand |
| Stored in model (use memory) | Not stored (calculated dynamically) |
| Use `[ColumnName]` syntax | Use `TableName[ColumnName]` syntax |
| Created in Data view | Created in Report/Data view |
| Good for: categorization, flags | Good for: aggregations, KPIs |

---

## Basic Aggregation Functions

```dax
SUM(TableName[Column])              // Sums all values
AVERAGE(TableName[Column])          // Calculates mean
MIN(TableName[Column])              // Finds minimum value
MAX(TableName[Column])              // Finds maximum value
COUNT(TableName[Column])            // Counts non-blank values
COUNTROWS(TableName)                // Counts all rows
DISTINCTCOUNT(TableName[Column])    // Counts unique values
```

---

## Iterator Functions (Row-by-Row)

```dax
SUMX(Table, Expression)             // Iterates and sums
AVERAGEX(Table, Expression)         // Iterates and averages
MINX(Table, Expression)             // Iterates and finds min
MAXX(Table, Expression)             // Iterates and finds max
COUNTX(Table, Expression)           // Iterates and counts

// Example:
Total_Weight = SUMX(sales_data, [Items] * [Unit_Weight])
```

---

## Date & Time Functions

```dax
TODAY()                             // Current date
NOW()                               // Current date and time
YEAR([DateColumn])                  // Extracts year (2023)
MONTH([DateColumn])                 // Extracts month (1-12)
DAY([DateColumn])                   // Extracts day (1-31)
WEEKDAY([DateColumn], 2)            // Day of week (1=Mon, 7=Sun)
EOMONTH([DateColumn], 0)            // End of month
FORMAT([DateColumn], "YYYYMMDD")    // Format as text
DATE(2023, 12, 25)                  // Creates date (Dec 25, 2023)
DATEDIFF([StartDate], [EndDate], DAY)  // Difference in days
```

---

## Text Functions

```dax
FORMAT(Value, "FormatString")       // Converts to formatted text
VALUE("12345")                      // Converts text to number
CONCATENATE("Hello", "World")       // Joins text (or use &)
LEFT([Column], 5)                   // First 5 characters
RIGHT([Column], 3)                  // Last 3 characters
LEN([Column])                       // Length of text
UPPER([Column])                     // Converts to uppercase
LOWER([Column])                     // Converts to lowercase
TRIM([Column])                      // Removes extra spaces
SUBSTITUTE([Column], "old", "new")  // Replaces text
```

---

## Conditional Logic

```dax
// IF Statement
IF(Condition, ValueIfTrue, ValueIfFalse)

// Example:
Status = IF([Amount] > 1000, "High", "Low")

// Nested IF
Tier = IF([Score] >= 90, "A",
       IF([Score] >= 80, "B",
       IF([Score] >= 70, "C", "F")))

// SWITCH (cleaner for multiple conditions)
Grade = SWITCH(
    TRUE(),
    [Score] >= 90, "A",
    [Score] >= 80, "B",
    [Score] >= 70, "C",
    "F"  // Default value
)

// SWITCH with exact match
Day_Name = SWITCH([DayNumber],
    1, "Monday",
    2, "Tuesday",
    3, "Wednesday",
    "Other"  // Default
)
```

---

## Filter Functions

```dax
FILTER(Table, Condition)            // Returns filtered table
ALL(Table or Column)                // Removes filters
ALLEXCEPT(Table, Column1, ...)      // Removes all filters except specified
VALUES(Column)                      // Distinct values in current context
DISTINCT(Column)                    // Distinct values (similar to VALUES)

// Example:
High_Sales = FILTER(sales_data, sales_data[Amount] > 5000)
```

---

## CALCULATE - Modify Filter Context

```dax
CALCULATE(Expression, Filter1, Filter2, ...)

// Examples:
Total_2023 = CALCULATE(
    SUM(sales_data[Amount]),
    YEAR(sales_data[Date]) = 2023
)

Product_A_Revenue = CALCULATE(
    SUM(sales_data[Revenue]),
    sales_data[Product] = "Product A"
)

All_City_Revenue = CALCULATE(
    SUM(sales_data[Revenue]),
    ALL(sales_data[City])  // Ignores City filter
)
```

---

## Relationship Functions

```dax
RELATED(TableName[Column])          // Gets value from related table (many-to-one)
RELATEDTABLE(TableName)             // Gets related table (one-to-many)
USERELATIONSHIP(Column1, Column2)   // Activates inactive relationship

// Example:
Product_Weight = RELATED(weight_table[Weight])

Total_Orders = COUNTROWS(RELATEDTABLE(orders))
```

---

## Logical Functions

```dax
AND(Condition1, Condition2)         // Both must be TRUE
OR(Condition1, Condition2)          // At least one TRUE
NOT(Condition)                      // Reverses TRUE/FALSE
TRUE()                              // Returns TRUE
FALSE()                             // Returns FALSE

// Example:
Valid = AND([Age] >= 18, [Status] = "Active")

// Using && and || (recommended)
Valid = [Age] >= 18 && [Status] = "Active"
Alert = [Stock] < 10 || [Expired] = TRUE()
```

---

## Math Functions

```dax
DIVIDE(Numerator, Denominator, [AltResult])  // Safe division
ABS(Number)                         // Absolute value
ROUND(Number, Decimals)             // Rounds number
ROUNDUP(Number, Decimals)           // Rounds up
ROUNDDOWN(Number, Decimals)         // Rounds down
MOD(Number, Divisor)                // Remainder after division
POWER(Number, Power)                // Raises to power
SQRT(Number)                        // Square root

// Example:
Unit_Price = DIVIDE([Total_Spend], [Items], 0)  // Returns 0 if divide by zero
```

---

## Information Functions

```dax
ISBLANK(Value)                      // Checks if blank
ISNUMBER(Value)                     // Checks if number
ISTEXT(Value)                       // Checks if text
ISERROR(Expression)                 // Checks if error
HASONEVALUE(Column)                 // Checks if single value in context

// Example:
Safe_Divide = IF(ISBLANK([Denominator]), 0, [Numerator]/[Denominator])
```

---

## Time Intelligence Functions

**Requirements**: Need a proper Date table with continuous dates

```dax
TOTALYTD(Expression, Dates)         // Year-to-date total
TOTALMTD(Expression, Dates)         // Month-to-date total
TOTALQTD(Expression, Dates)         // Quarter-to-date total

SAMEPERIODLASTYEAR(Dates)          // Same period previous year
DATEADD(Dates, -1, MONTH)          // Shift dates by interval
PARALLELPERIOD(Dates, -1, YEAR)    // Complete parallel period

// Examples:
YTD_Sales = TOTALYTD(SUM(sales_data[Amount]), dim_date[Date])

Last_Year = CALCULATE(
    SUM(sales_data[Amount]),
    SAMEPERIODLASTYEAR(dim_date[Date])
)

Previous_Month = CALCULATE(
    SUM(sales_data[Amount]),
    DATEADD(dim_date[Date], -1, MONTH)
)
```

---

## Common Patterns

### Pattern 1: Percentage of Total
```dax
Pct_of_Total = 
DIVIDE(
    SUM(sales_data[Amount]),
    CALCULATE(SUM(sales_data[Amount]), ALL(sales_data))
)
```

### Pattern 2: Running Total
```dax
Running_Total = 
CALCULATE(
    SUM(sales_data[Amount]),
    FILTER(
        ALL(sales_data[Date]),
        sales_data[Date] <= MAX(sales_data[Date])
    )
)
```

### Pattern 3: Rank
```dax
Product_Rank = 
RANKX(
    ALL(sales_data[Product]),
    [Total_Revenue],
    ,
    DESC
)
```

### Pattern 4: Previous Value
```dax
Previous_Month_Sales = 
CALCULATE(
    [Total_Sales],
    DATEADD(dim_date[Date], -1, MONTH)
)
```

### Pattern 5: Moving Average
```dax
Moving_Avg_3M = 
CALCULATE(
    [Total_Sales],
    DATESINPERIOD(dim_date[Date], LASTDATE(dim_date[Date]), -3, MONTH)
) / 3
```

---

## Syntax Tips

✓ Use `TableName[ColumnName]` in measures  
✓ Use `[ColumnName]` in calculated columns  
✓ Always handle division by zero with DIVIDE()  
✓ Use `// Comment` for single-line comments  
✓ Use `/* Comment */` for multi-line comments  
✓ DAX is case-insensitive but use consistent casing  
✓ Press Shift+Enter for multi-line formulas  

---

## Common Errors & Fixes

| Error | Likely Cause | Solution |
|-------|--------------|----------|
| "Column not found" | Missing table name | Use `Table[Column]` |
| Circular dependency | Column references itself | Check calculation dependencies |
| Wrong data type | Text vs. number mismatch | Use VALUE() or FORMAT() |
| BLANK results | Division by zero or missing data | Use DIVIDE() or handle BLANKs |
| Slow performance | Too many calculated columns | Convert to measures where possible |

---

## Best Practices

1. **Use measures instead of calculated columns** when possible (saves memory)
2. **Create a proper Date table** for time intelligence
3. **Use DIVIDE()** instead of `/` operator
4. **Use variables** (VAR) for complex calculations (cleaner, faster)
5. **Format measures** appropriately (currency, percentage, etc.)
6. **Use meaningful names** (avoid spaces, use underscores)
7. **Comment your code** for complex formulas
8. **Test incrementally** - build complex formulas step by step

---

## Variable Usage (VAR)

```dax
Total_Profit = 
VAR TotalRevenue = SUM(sales_data[Revenue])
VAR TotalCost = SUM(sales_data[Cost])
RETURN
    TotalRevenue - TotalCost
```

Benefits: Cleaner code, better performance (calculated once), easier debugging

---

**Quick Reference Keywords**: SUM, CALCULATE, FILTER, RELATED, IF, SWITCH, FORMAT, DIVIDE, SUMX, ALL, VALUES, YEAR, MONTH, COUNTROWS, AVERAGE
