# Power BI DAX Worksheet - Solutions

## Exercise 1: Basic Calculated Columns - Arithmetic & Text Operations

### Task 1.1: Revenue per Item

```dax
Revenue_Per_Item = DIVIDE([Total Spend], [Items], 0)
```

**Explanation**: 
- `DIVIDE()` safely divides the first argument by the second
- The third parameter (0) is the default value if division by zero occurs
- This is safer than using the `/` operator directly

**Alternative Solution**:
```dax
Revenue_Per_Item = [Total Spend] / [Items]
```

---

### Task 1.2: Product-City Combination

```dax
Product_City_Label = [Product] & " - " & [City]
```

**Explanation**: 
- The `&` operator concatenates (joins) text strings
- You can include literal text in quotes like " - "
- This creates descriptive labels for each transaction

**Alternative Solution using CONCATENATE**:
```dax
Product_City_Label = CONCATENATE(CONCATENATE([Product], " - "), [City])
```
Note: Using `&` is simpler and more readable.

---

## Exercise 2: Date & Time Extraction

### Task 2.1: Extract Quarter

```dax
Quarter = "Q" & QUARTER([Date Time])
```

**Explanation**: 
- `QUARTER()` function returns 1, 2, 3, or 4
- We concatenate "Q" with the number to get "Q1", "Q2", etc.
- This creates readable quarter labels

**Alternative Solution using FORMAT**:
```dax
Quarter = FORMAT([Date Time], "\QQ")
```
This produces "Q1", "Q2", "Q3", or "Q4" directly.

---

### Task 2.2: Extract Day of Week Name

```dax
Day_of_Week = FORMAT([Date Time], "DDDD")
```

**Explanation**: 
- `FORMAT()` converts a value to text using a format string
- "DDDD" returns the full day name (Monday, Tuesday, etc.)
- "DDD" would return abbreviated names (Mon, Tue, etc.)

**Alternative Solution**:
```dax
Day_of_Week = 
SWITCH(
    WEEKDAY([Date Time], 2),
    1, "Monday",
    2, "Tuesday",
    3, "Wednesday",
    4, "Thursday",
    5, "Friday",
    6, "Saturday",
    7, "Sunday"
)
```

---

## Exercise 3: Conditional Logic & Categorization

### Task 3.1: Volume Tier Column

```dax
Order_Size = 
SWITCH(
    TRUE(),
    [Items] < 10, "Small Order",
    [Items] <= 20, "Medium Order",
    "Large Order"
)
```

**Explanation**: 
- `SWITCH(TRUE(), ...)` pattern evaluates conditions in order
- First condition that evaluates to TRUE returns its value
- "Large Order" is the default for anything > 20

**Alternative Solution using IF**:
```dax
Order_Size = 
IF(
    [Items] < 10,
    "Small Order",
    IF([Items] <= 20, "Medium Order", "Large Order")
)
```

---

### Task 3.2: Time of Day Category

```dax
Time_of_Day = 
VAR HourOfDay = HOUR([Date Time])
RETURN
SWITCH(
    TRUE(),
    HourOfDay >= 6 && HourOfDay <= 11, "Morning",
    HourOfDay >= 12 && HourOfDay <= 17, "Afternoon",
    HourOfDay >= 18 && HourOfDay <= 23, "Evening",
    "Night"
)
```

**Explanation**: 
- `VAR` creates a variable to store the hour (0-23)
- `HOUR()` extracts the hour from the datetime
- We use `&&` (logical AND) to check if hour is in a range
- Cleaner code by extracting hour once into a variable

**Alternative Solution without VAR**:
```dax
Time_of_Day = 
SWITCH(
    TRUE(),
    HOUR([Date Time]) >= 6 && HOUR([Date Time]) <= 11, "Morning",
    HOUR([Date Time]) >= 12 && HOUR([Date Time]) <= 17, "Afternoon",
    HOUR([Date Time]) >= 18 && HOUR([Date Time]) <= 23, "Evening",
    "Night"
)
```

---

## Exercise 4: Working with Related Tables

### Task 4.1: Weight per Item

```dax
Weight_Per_Unit = RELATED(weight[Weight_grams])
```

**Explanation**: 
- `RELATED()` traverses the relationship to get matching values
- Works from the "many" side to the "one" side of the relationship
- Requires a valid relationship between tables on Product column

---

### Task 4.2: Determine if Product is Heavy

```dax
Product_Weight_Category = 
IF(
    RELATED(weight[Weight_grams]) > 500,
    "Heavy",
    "Light"
)
```

**Explanation**: 
- Uses `RELATED()` to get the weight from the weight table
- Compares it to 500 grams threshold
- Returns appropriate category label

**Alternative Solution (if Weight_Per_Unit column exists)**:
```dax
Product_Weight_Category = IF([Weight_Per_Unit] > 500, "Heavy", "Light")
```

---

## Exercise 5: Measures for Business Intelligence

### Task 5.1: Maximum Single Transaction

```dax
Max_Transaction_Value = MAX(sales_data[Total Spend])
```

**Explanation**: 
- `MAX()` returns the largest value in a column
- Responds to filter context (slicers, visuals)
- Always reference `TableName[ColumnName]` in measures

**Expected Result**: $9,838

---

### Task 5.2: Minimum Single Transaction

```dax
Min_Transaction_Value = MIN(sales_data[Total Spend])
```

**Explanation**: 
- `MIN()` returns the smallest value in a column
- Works like MAX() but finds the minimum instead

**Expected Result**: $162

---

### Task 5.3: Average Items per Transaction

```dax
Avg_Items_Per_Sale = AVERAGE(sales_data[Items])
```

**Explanation**: 
- `AVERAGE()` calculates the mean of all values
- Automatically handles filter context

**Expected Result**: ~13.41 items per transaction

**Alternative Solution (manual calculation)**:
```dax
Avg_Items_Per_Sale = DIVIDE(SUM(sales_data[Items]), COUNTROWS(sales_data))
```

---

### Task 5.4: Count of Unique Customers

```dax
Unique_Customer_Count = DISTINCTCOUNT(sales_data[Customer])
```

**Explanation**: 
- `DISTINCTCOUNT()` counts unique values only
- Different from `COUNT()` which counts non-blank values (including duplicates)
- Perfect for counting unique customers, products, cities, etc.

**Expected Result**: 10 unique customers

**Alternative Solution**:
```dax
Unique_Customer_Count = COUNTROWS(DISTINCT(sales_data[Customer]))
```

---

### Task 5.5: Revenue per Customer (Average)

```dax
Revenue_Per_Customer = 
DIVIDE(
    SUM(sales_data[Total Spend]),
    DISTINCTCOUNT(sales_data[Customer]),
    0
)
```

**Explanation**: 
- Divides total revenue by number of unique customers
- `DIVIDE()` handles any division by zero gracefully
- This measure responds to filters (e.g., by Product or City)

**Expected Result**: ~$49,109 per customer

**Alternative using variables**:
```dax
Revenue_Per_Customer = 
VAR TotalRevenue = SUM(sales_data[Total Spend])
VAR CustomerCount = DISTINCTCOUNT(sales_data[Customer])
RETURN
DIVIDE(TotalRevenue, CustomerCount, 0)
```

---

## Bonus Challenge: Advanced Filtering & Context

### Task 6.1: Count of Large Transactions

```dax
Large_Order_Count = 
COUNTROWS(
    FILTER(
        sales_data,
        sales_data[Items] > 15
    )
)
```

**Explanation**: 
- `FILTER()` returns a table with only rows meeting the condition
- `COUNTROWS()` counts the rows in that filtered table
- Still responds to external filters from slicers/visuals

**Alternative using CALCULATE**:
```dax
Large_Order_Count = 
CALCULATE(
    COUNTROWS(sales_data),
    sales_data[Items] > 15
)
```

**Expected Result**: 52 transactions with > 15 items

---

### Task 6.2: Revenue from City 1 Only

```dax
City_1_Revenue = 
CALCULATE(
    SUM(sales_data[Total Spend]),
    sales_data[City] = "City 1"
)
```

**Explanation**: 
- `CALCULATE()` modifies the filter context
- The second parameter overrides any City filter
- Will always show City 1 revenue even if user filters to other cities

**Expected Result**: $77,175 from City 1

**Alternative using FILTER**:
```dax
City_1_Revenue = 
CALCULATE(
    SUM(sales_data[Total Spend]),
    FILTER(ALL(sales_data[City]), sales_data[City] = "City 1")
)
```

---

### Task 6.3: Percentage of Total Revenue

```dax
Revenue_Percentage = 
DIVIDE(
    SUM(sales_data[Total Spend]),
    CALCULATE(SUM(sales_data[Total Spend]), ALL(sales_data)),
    0
)
```

**Explanation**: 
- Numerator: revenue in current filter context
- Denominator: revenue with ALL filters removed using `ALL()`
- `ALL(sales_data)` ignores all filters on the sales_data table
- Returns a decimal (0.25 = 25%)

**To display as percentage**: Format the measure as percentage in Power BI.

**Alternative (percentage without formatting)**:
```dax
Revenue_Percentage = 
VAR CurrentRevenue = SUM(sales_data[Total Spend])
VAR TotalRevenue = CALCULATE(SUM(sales_data[Total Spend]), ALL(sales_data))
RETURN
DIVIDE(CurrentRevenue, TotalRevenue, 0) * 100
```

---

## Testing Your Solutions with DAX Studio

You can test your work using DAX Studio with EVALUATE queries:

### Test 1: Revenue by Product
```dax
EVALUATE
SUMMARIZE(
    sales_data,
    sales_data[Product],
    "Total Revenue", SUM(sales_data[Total Spend]),
    "Avg Items", AVERAGE(sales_data[Items]),
    "Transaction Count", COUNTROWS(sales_data)
)
ORDER BY [Total Revenue] DESC
```

---

### Test 2: Customer Analysis
```dax
EVALUATE
ADDCOLUMNS(
    SUMMARIZE(
        sales_data,
        sales_data[Customer]
    ),
    "Total Spent", CALCULATE(SUM(sales_data[Total Spend])),
    "Transaction Count", CALCULATE(COUNTROWS(sales_data)),
    "Avg Transaction", CALCULATE(AVERAGE(sales_data[Total Spend]))
)
ORDER BY [Total Spent] DESC
```

---

### Test 3: Time of Day Analysis
```dax
EVALUATE
SUMMARIZE(
    sales_data,
    sales_data[Time_of_Day],
    "Revenue", SUM(sales_data[Total Spend]),
    "Orders", COUNTROWS(sales_data),
    "Avg Order Value", AVERAGE(sales_data[Total Spend])
)
ORDER BY [Revenue] DESC
```

---

### Test 4: Large Orders by City
```dax
EVALUATE
ADDCOLUMNS(
    SUMMARIZE(
        FILTER(sales_data, sales_data[Items] > 20),
        sales_data[City]
    ),
    "Large Orders", CALCULATE(COUNTROWS(sales_data)),
    "Total Revenue", CALCULATE(SUM(sales_data[Total Spend]))
)
ORDER BY [Large Orders] DESC
```

---

## Common DAX Mistakes to Avoid

1. **Using calculated columns when you need measures**
   - Columns: Stored in memory, computed row-by-row
   - Measures: Computed on-demand based on context

2. **Forgetting table names in measures**
   - ✗ `SUM([Total Spend])`
   - ✓ `SUM(sales_data[Total Spend])`

3. **Not handling division by zero**
   - ✗ `[Revenue] / [Count]`
   - ✓ `DIVIDE([Revenue], [Count], 0)`

4. **Confusing FILTER vs. CALCULATE**
   - `FILTER()` returns a table
   - `CALCULATE()` evaluates an expression in modified context

5. **Not using VAR for complex calculations**
   - Makes code cleaner and faster (values calculated once)
   - Easier to debug

6. **Wrong RELATED direction**
   - `RELATED()`: many to one
   - `RELATEDTABLE()`: one to many

---

## Performance Tips

✓ **Use measures instead of calculated columns** where possible (saves memory)  
✓ **Use variables (VAR)** for values used multiple times  
✓ **Use DIVIDE()** instead of `/` to avoid errors  
✓ **Filter early** - apply filters before aggregating when possible  
✓ **Avoid calculated columns** with RELATED if you can use measures instead  
✓ **Use DISTINCTCOUNT** instead of counting DISTINCT table rows  

---

## Key Takeaways

✓ **Text concatenation**: Use `&` operator or CONCATENATE()  
✓ **Date extraction**: QUARTER(), HOUR(), FORMAT()  
✓ **Conditional logic**: IF() for simple, SWITCH() for multiple conditions  
✓ **Variables**: VAR makes code cleaner and more efficient  
✓ **Aggregations**: MAX, MIN, AVERAGE, DISTINCTCOUNT  
✓ **RELATED**: Pulls data from related tables (many-to-one)  
✓ **CALCULATE**: Most powerful function for modifying filter context  
✓ **ALL**: Removes filters for percentage calculations  
✓ **DIVIDE**: Safe division with default value handling  

---

**Congratulations on completing the worksheet!** Continue practicing with your own datasets to master DAX.
