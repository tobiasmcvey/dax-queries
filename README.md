# Data Analysis Expression Queries

Formulas for learning DAX in Microsoft excel and Power BI.

**Learning resources**
* [Learn DAX Microsoft Docs](https://docs.microsoft.com/en-us/dax/data-analysis-expressions-dax-reference)
* [Dax Function Reference](https://docs.microsoft.com/en-us/dax/dax-function-reference)

**Contents**

- [Date transformations](https://github.com/tobmcv/dax-queries/blob/master/formulas.md#date-transformations)
- [Simple operations](https://github.com/tobmcv/dax-queries/blob/master/formulas.md#simple-operations)
- [Combining functions for transformations](https://github.com/tobmcv/dax-queries/blob/master/formulas.md#combining-functions-for-calculations)
- [Ecommerce calculations](https://github.com/tobmcv/dax-queries/blob/master/formulas.md#ecommerce-calculations)

### Date transformations

**Extract Year and Month from a date**
```
Year-Mon = FORMAT('Dimdate'[Date]; "YYYY-MM")
```
This can be very useful for aggregating a value in a chart by years and months.

**Extract Week number**
```
WeekNum = weeknum('Database orderlevel'[orderPlacedDate]; 2)
```


### Simple operations

**Multiplying, dividing, addition and subtraction over tables and columns**
```
daysToPickup = ('Database orderlevel'[collectionStartTime] - 'Database orderlevel'[orderPlacedDate])
```


### Combining Functions for calculations

**Using inactive relationships in data model for calculations**

Sometimes you will encounter a need to use both the active and inactive relationships in your data model. 

This is typical if you have multiple types of date entries, for example if you run a store that collects
* Order Placed Date - the date and time the customer placed their order
* Order Updated Date - the date and time the customer updated their order with a change
* Order Collect Date - the range of dates a customer can collect their order
* Order Collected Date - the date and time the customer collected their order

In these cases we are limited since Power BI and Excel do not handle multiple date relationships well. 

For example, if you want a table and visuals for both the Order Placed Date and the Order Collect Date, you can do the following
* Active relationship: DimDate and Order Placed Date - to power most of the tables and visuals, since this is commonly the date we want for our KPIs
* Inactive relationship: DimDate and Order Collect Date - to power specific calculations about the order collection date range


Example 1
```
AmountCollectionStartTime = CALCULATE(SUM('Database orderlevel'[Amount]);USERELATIONSHIP('Database orderlevel[collectionStartTime];Dimdate[Date]))
```

Example 2
```
Click&CollectRate = IFERROR(CALCULATE(SUM('Database orderlevel'[Amount]) / SUM('OtherDatabase'[Profit]); USERELATIONSHIP('Database orderlevel'[collectionStartTime]; 'Dimdate'[Date])); BLANK())
```
In this example we use 
* `IFERROR` to replaced values such as INFINITY with BLANK - INFINITY will distort tables and graph visuals
* `CALCULATE` to perform a calculation with multiple steps across several tables in our PBIX file


**Average order size**
```
Average order size = SUMX('Database orderlevel'; 'Database orderlevel'[Amount] / DISTINCTCOUNT('Database orderlevel'[orderId]))
```

Assuming you have 2 tables, containing 
* Orders
* Orderitems

You will find it's hard to count the unique number of items per order - the number of products in a single order - without creating a table to quantify number of products per order

**Create a table for number of orderitems per order**
```
Transactions = FILTER(GROUPBY('Database orderitems'; 'Database orderitems'[column_orderId]; 'Database orderitems'[column_category]; "Number of orderitems"; SUMX( CURRENTGROUP(); 'Database orderitems'[quantity])); 'Database orderitems'[column_category]="online_purchase")
```

Alternatively you can create a table in a DAX formula and return its value without creating an additional permanent table in your PBIX file. 

**Average number of orderitems**
```
Average num orderitems = SUMX('Transactions'; 'Transactions'[Number of orderitems] / DISTINCTCOUNT('Transactions'[Database orderlevel_orderId]))
```

### Ecommerce calculations


**Dropoff Rate Shopping Cart**
```
% Dropoff Cart = (CALCULATE(SUM(Ecommerce[Sessions]);Ecommerce[Shopping Stage] = "ADD_TO_CART") - CALCULATE(SUM(Ecommerce[Sessions]);Ecommerce[Shopping Stage] = "TRANSACTION")) / CALCULATE(SUM(Ecommerce[Sessions]);Ecommerce[Shopping Stage] = "ADD_TO_CART")
```
This formula should work for Google Analytics and other web analytics data sources in Power BI.

**Add to Cart Rate**
```
Add to Cart = CALCULATE(SUM(Ecommerce[Sessions]);Ecommerce[Shopping Stage] = "ADD_TO_CART") / CALCULATE(SUM(Ecommerce[Sessions]);Ecommerce[Shopping Stage] = "ALL_VISITS")
```

**Returning Customers**
```
Returning Customers = CALCULATE(SUM(Customers[Sessions]);Customers[U: Existing Customer Online Store] = "1")/SUM('Sessions'[Sessions])
```

**Returning Visitors**
```
Returning Visitors = CALCULATE(SUM('Returning Visitors'[Sessions]);'Returning Visitors'[User Type] = "Returning Visitor")/SUM('Sessions'[Sessions])
```

**Average orderitems**
```
Average num orderitems = DIVIDE(SUM(Sales[Quantity]);SUM(Sales[Number of transactions]))
```

**Average order size**
```
Average order size = DIVIDE(SUM(Sales[Revenue]);SUM(Sales[Number of transactions]))
```

**Conversion Rate**
```
Conversion Rate = SUM(Sales[Number of transactions]) / SUM('Sessions'[Sessions])
```