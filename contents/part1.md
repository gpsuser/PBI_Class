# Power BI - Part 1

## data

### flatfile



## To Do


* Add `Unit Price` column

```dax
unit_price = [Total Spend] / [Items]
```

* Add `Month` column

```dax
Month = FORMAT([Date Time], "MMMM")
```

* Add `Data Date` column 

```dax
data_date = VALUE(FORMAT([Date Time], "YYYYMMDD"))
```

* Add `Revenue Tier` column


 Revenue Tier - Categorized as Low (<$2,000), Medium ($2,000-$6,000), or High (>$6,000)

 ```dax
 Revenue Tier = 
SWITCH(
    TRUE(),
    [Total Spend] < 2000, "Low",
    [Total Spend] <= 6000, "Medium",
    "High"
)
 ```

### Manually create `weight` table

Copy and paste the below into `Enter Data`, right lick on Column 1 :  `Paste >  Name: weight > Load`

```csv

Product	Weight_grams
Product A	115
Product B	220
Product C	3000
Product D	410
Product E	525
```


### Use DAX Studio to "test" aggregation: sum of weight by city  

```dax
EVALUATE
SUMMARIZE(
    'sales_data',
    'sales_data'[City],
    "Total Weight", 
    SUMX(
        'sales_data',
        RELATED('weight'[Weight_grams])
    )
)
```

### Sorting in DAX 

```dax
EVALUATE
SUMMARIZE(
    'sales_data',
    'sales_data'[City],
    "Total Weight", 
    SUMX(
        'sales_data',
        RELATED('weight'[Weight_grams])
    )
)
ORDER BY [Total Weight] DESC

```

### Use DAX Studio to "test" aggregation: sum of weight by city and product, sorted by city and total weight

```dax
EVALUATE
SUMMARIZE(
    'sales_data',
    'sales_data'[City],
    'sales_data'[Product],
    "Total Weight", 
    SUMX(
        'sales_data',
        RELATED('weight'[Weight_grams])
    )
)
ORDER BY 'sales_data'[City], [Total Weight] DESC

```

![image.png](/img/weight_by_city_by_product.png)

### Creating and using measures in DAX

A measure is a calculation that is performed on the data in your model. Measures are typically used to create aggregated values that can be used in visuals, such as sums, averages, counts, etc.

Example 1: Create a measure to calculate total weight

```dax
msr_Total_Weight = 
SUMX(
    'sales_data',
    RELATED('weight'[Weight_grams])
)

```

![image.png](/img/msr_total_weight.png)

Use the measure in a visual:

1. Create a table visual in Power BI
2. Add `City` from `sales_data` to the rows
3. Add the `msr_Total_Weight` measure to the values
4. Sort the table by `msr_Total_Weight` in descending order

Example 2: Create a measure to calculate average weight per item

```dax
Average Weight per Item =
DIVIDE(
    [Total Weight],
    SUM('sales_data'[Items])
)
```

Use the average weight measure in a visual:

1. Create a table visual in Power BI
2. Add `City` from `sales_data` to the rows
3. Add the `Average Weight per Item` measure to the values
4. Sort the table by `Average Weight per Item` in descending order


### Working with AI to help with DAX code generation

Prompt: "Write a DAX measure to calculate the total weight of products sold, using the 'weight' table to get the weight of each product and the 'sales_data' table to get the quantity sold."

```dax
Total Weight =
SUMX(
    'sales_data',
    RELATED('weight'[Weight_grams]) * 'sales_data'[Items]
)
```

Prompt: "Below is some dax code to sum total weight by city - how do i do this total sum of weight by city and product, given the attached screenshots for the two tables `weight` and `sales_data`?  Here is the initial dax: """ SUMMARIZE( 'sales_data', 'sales_data'[City], "Total Weight", SUMX( 'sales_data', RELATED('weight'[Weight_grams]) ) )  """ 
"

```dax
EVALUATE
SUMMARIZE(
    'sales_data',
    'sales_data'[City],
    'sales_data'[Product],
    "Total Weight", 
    SUMX(
        'sales_data',
        RELATED('weight'[Weight_grams])
    )
)
```