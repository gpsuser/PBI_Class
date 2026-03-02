# Power BI DAX Worksheet for Beginners

## Introduction

Welcome to this hands-on Power BI DAX worksheet! This exercise set will help you learn the fundamentals of DAX (Data Analysis Expressions) using a real sales dataset.

**Dataset**: `sales_data.csv`

**Columns**:
- Sales ID
- Date Time (format: DD/MM/YYYY HH:MM)
- Items
- Total Spend
- Product (Product A, B, C, D, E)
- City (City 1-10)
- Customer

**Setup Instructions**:
1. Open Power BI Desktop
2. Import the `sales_data.csv` file
3. Create a separate `weight` table using "Enter Data" with the following:

| Product | Weight_grams |
|---------|--------------|
| Product A | 115 |
| Product B | 220 |
| Product C | 3000 |
| Product D | 410 |
| Product E | 525 |

4. Create a relationship between `sales_data[Product]` and `weight[Product]`

---

## Exercise 1: Basic Calculated Columns - Arithmetic & Text Operations

**Objective**: Learn to create calculated columns using arithmetic operations and text manipulation.

### Task 1.1: Calculate Revenue per Item
Calculate how much revenue each individual item generates in a transaction.

**Column Name**: `Revenue_Per_Item`

**Your DAX Code**:
```dax
Revenue_Per_Item = 


```

**Expected Result**: A column showing Total Spend divided by the number of items (e.g., a $1000 sale with 10 items = $100 per item).

---

### Task 1.2: Create Product-City Combination
Create a text column that combines the Product and City into a single descriptor (e.g., "Product A - City 1").

**Column Name**: `Product_City_Label`

**Your DAX Code**:
```dax
Product_City_Label = 


```

**Expected Result**: Text values like "Product A - City 1", "Product B - City 3", etc.

---

## Exercise 2: Date & Time Extraction

**Objective**: Extract different components from date/time values.

### Task 2.1: Extract Quarter
Create a column that shows which quarter of the year the sale occurred in (Q1, Q2, Q3, Q4).

**Column Name**: `Quarter`

**Your DAX Code**:
```dax
Quarter = 


```

**Expected Result**: Values like "Q1", "Q2", "Q3", "Q4".

---

### Task 2.2: Extract Day of Week Name
Create a column showing the day of the week as text (Monday, Tuesday, etc.).

**Column Name**: `Day_of_Week`

**Your DAX Code**:
```dax
Day_of_Week = 


```

**Expected Result**: Day names like "Monday", "Tuesday", "Wednesday", etc.

---

## Exercise 3: Conditional Logic & Categorization

**Objective**: Use conditional functions to create business categories.

### Task 3.1: Create Volume Tier Column
Categorize transactions based on the number of items sold:
- "Small Order" if Items < 10
- "Medium Order" if Items between 10 and 20
- "Large Order" if Items > 20

**Column Name**: `Order_Size`

**Your DAX Code**:
```dax
Order_Size = 


```

**Expected Result**: Each sale categorized as Small Order, Medium Order, or Large Order.

---

### Task 3.2: Create Time of Day Category
Categorize transactions by the time they occurred:
- "Morning" for hours 6-11
- "Afternoon" for hours 12-17
- "Evening" for hours 18-23
- "Night" for hours 0-5

**Column Name**: `Time_of_Day`

**Your DAX Code**:
```dax
Time_of_Day = 


```

**Expected Result**: Categories like "Morning", "Afternoon", "Evening", "Night".

---

## Exercise 4: Working with Related Tables

**Objective**: Use RELATED function and perform calculations with data from multiple tables.

### Task 4.1: Calculate Weight per Item
Calculate the weight of a single item by getting the product weight from the weight table.

**Column Name**: `Weight_Per_Unit`

**Your DAX Code**:
```dax
Weight_Per_Unit = 


```

**Expected Result**: The weight in grams for one unit of each product.

**Hint**: This should return the same value as the weight table for each product.

---

### Task 4.2: Determine if Product is Heavy
Create a flag that marks products as "Heavy" if they weigh more than 500 grams, otherwise "Light".

**Column Name**: `Product_Weight_Category`

**Your DAX Code**:
```dax
Product_Weight_Category = 


```

**Expected Result**: "Heavy" for Products C, D, E; "Light" for Products A, B.

---

## Exercise 5: Measures for Business Intelligence

**Objective**: Create measures for dynamic aggregations and KPIs.

### Task 5.1: Maximum Single Transaction
Create a measure that finds the highest transaction value.

**Measure Name**: `Max_Transaction_Value`

**Your DAX Code**:
```dax
Max_Transaction_Value = 


```

**Expected Result**: The largest Total Spend value in the dataset (~$9,838).

---

### Task 5.2: Minimum Single Transaction
Create a measure that finds the lowest transaction value.

**Measure Name**: `Min_Transaction_Value`

**Your DAX Code**:
```dax
Min_Transaction_Value = 


```

**Expected Result**: The smallest Total Spend value in the dataset (~$162).

---

### Task 5.3: Average Items per Transaction
Create a measure that calculates the average number of items sold per transaction.

**Measure Name**: `Avg_Items_Per_Sale`

**Your DAX Code**:
```dax
Avg_Items_Per_Sale = 


```

**Expected Result**: Average quantity of items (~13 items per transaction).

---

### Task 5.4: Count of Unique Customers
Create a measure that counts how many distinct customers made purchases.

**Measure Name**: `Unique_Customer_Count`

**Your DAX Code**:
```dax
Unique_Customer_Count = 


```

**Expected Result**: Number of different customers in the dataset (10).

---

### Task 5.5: Revenue per Customer (Average)
Create a measure that calculates the average revenue per customer.

**Measure Name**: `Revenue_Per_Customer`

**Your DAX Code**:
```dax
Revenue_Per_Customer = 


```

**Expected Result**: Total revenue divided by unique customer count (~$49,109).

---

## Bonus Challenge: Advanced Filtering & Context

### Task 6.1: Count of Large Transactions
Create a measure that counts only transactions where more than 15 items were sold.

**Measure Name**: `Large_Order_Count`

**Your DAX Code**:
```dax
Large_Order_Count = 


```

---

### Task 6.2: Revenue from City 1 Only
Create a measure that calculates total revenue specifically from City 1, regardless of any filters applied.

**Measure Name**: `City_1_Revenue`

**Your DAX Code**:
```dax
City_1_Revenue = 


```

---

### Task 6.3: Percentage of Total Revenue
Create a measure that shows what percentage of total revenue is represented by the current filter context.

**Measure Name**: `Revenue_Percentage`

**Your DAX Code**:
```dax
Revenue_Percentage = 


```

**Hint**: Compare the revenue in the current context to revenue with all filters removed.

---

## Testing Your Work

After completing the exercises:

1. Create a table visual showing your calculated columns
2. Create card visuals for each measure to verify the results
3. Test your measures with slicers (City, Product, Customer) to see how they respond to filters
4. Create a matrix visual with Products in rows and your measures in values
5. Use DAX Studio to run test queries (optional)

---

## Learning Checklist

By completing this worksheet, you should now understand:

- ✓ How to create calculated columns vs. measures
- ✓ Basic arithmetic and text operations in DAX
- ✓ Date and time extraction functions (QUARTER, HOUR, FORMAT)
- ✓ Conditional logic with IF and SWITCH
- ✓ The RELATED function for joining data from related tables
- ✓ Aggregation functions (SUM, MAX, MIN, AVERAGE, DISTINCTCOUNT)
- ✓ Filter context and the CALCULATE function
- ✓ Percentage calculations with ALL function

---


