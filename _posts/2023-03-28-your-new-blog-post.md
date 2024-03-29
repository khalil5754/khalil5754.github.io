# Window Functions, Variables, Date Manipulation, Imputation, and Loops in SQL Server

Many of SQL's most useful tools are much less well known than one would expect. 

## Working With NULL Values (Imputation).
In this blog post I'd like to get progressively more complex and interesting as we get deeper in the post. So before we get into COALESCE or the even more interesting stored procedures, let me show you one very easy way to replace a singular column's values with a value of your choice in the case that the value is missing. Using the ISNULL(x, 'y') function allows you to select a column _x_, and in the case that a value in column _x_ is null, it will be replaced with the value 'y'. Simple. You can also choose to replace the null value with another column in the same table. To do this, just ensure the second argument in the ISNULL() function is a column name (without quotes, of course). 

Now, more interesting - using Coalesce! What happens if we'd like to replace NULL values in one column with another column's value, *and* we want to ensure the column we're using to replace the NULL values isn't itself a NULL value?

COALESCE's syntax may be a bit confusing to grasp due to the fact that it takes multiple arguments. Let's say we have an airport's table with country codes in one column, province names in another, city names, and airport codes. 

In case we want to select airport codes but we're worried about NULL values, we can use COALESCE like this: 

```
COALESCE(airport_code, city_name, province_name)
```

This single line is checking for the first non-null value among three columns: "airport_code", "city_name", and "province_name".
So if the "airport_code" column has a non-null value, its value will be returned. If "airport_code" is null, the function will check if "city_name" has a non-null value and return that. If both "airport_code" and "city_name" are null, it will check for a non-null value in the "province_name" column and return that.

## Variables & While Loops
Variables can be immensely useful in SQL, and they're very easy to use. To declare and set a variable:

```
DECLARE @pants VARCHAR(5)
DECLARE @size INT(3)

SET @pants = 'Jorts'
SET @size = '30'
```

Here we declare our variables using the DECLARE command followed by an '@' symbol to denote a variable, and then telling SQL what kind of variable we're declaring (in this case a string and an integer). Then we can simply use the SET command to set this variable equal to whatever we need. It's that easy. A quick and easy way to use the above variables we've declared is to simply have them in the WHERE clause of a query. 

For example, say we had an apparel retailer's dataset and we wanted to see jorts (jean shorts, for those fortunate enough to not know) in the retailer's inventory in size 30.

```
SELECT article, sizing
FROM retailer_articles
WHERE article = @pants AND sizing = @size
```
It's that easy.

Now, let me do something even more interesting and introduce WHILE loops, as well as explain how variables can come into play.

To build a while loop in SQL Server, we can use commands WHILE, BEGIN, IF, BREAK, and of course, END.

Let me build a basic while loop and walk you through it.

```
DECLARE @instance INT

SET @instance = 1

WHILE @instance < 5
  BEGIN 
  SET @instance = @instance + 1
  
  IF @instance = 3
    BREAK
END
```

It's fairly easy to tell what's going on in the above code if you've had practically any exposure to programming. That being said, the BREAK command is unnecessary and just serves to show you that the while loop can be broken before it finishes its loop.

Variables aren't limited to an explicit assignment; you can also set a variable equal to a SELECT statement! If we wanted to declare a variable for an orderdate, but we aren't quite sure what the order date is, you can do something like this: 

```
DECLARE @OrderDate as date
SET 
	@OrderDate = (
		SELECT TOP 1 OrderDate
		FROM orders
		ORDER BY date
		);
		
DECLARE @OrderDateTime as datetime
SET 
	@OrderTime =  
```


## Window Functions 

Window functions in SQL are a powerful tool for data scientist that allow for the calculation of aggregate functions over a specific window of rows simultaneously with a main query. They can provide additional insights into data such as calculating moving averages. One example of a use-case for window functions is to calculate the moving average of sales over a period of time, allowing businesses to track trends and make informed decisions.

The syntax for Window functions in SQL is very specific and must always be written out fully:

```
OVER (PARTITION BY SalesYear ORDER BY SalesYear)
```
The OVER clause creates the window, while PARTITION BY tells us the slice of the table we want to work with. Lastly, ORDER BY of course takes care of the order of the results. There are already some very useful pre-built functions baked into SQL, such as SUM() or RANK().

An important set of window functions to remember are LEAD() and LAG(). Lead can peek and pull the next row's data (very difficult without a window function) while LAG() pulls the previous row's data. For example, if we had a table of orders with columns for the customer ID and the order's date, we could use the following code to pull a customer's previous order date:

```
SELECT CustomerID, OrderDate, 
       LAG(OrderDate) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS previousorder
FROM orders
```
This will return a new column called "previousorder" which solely returns the customer's most recent order. Since we're partitioning by each customer, SQL will only show us the results for one customer at a time

## Date Functions

One difference in the syntax between SQL Server and other SQL languages is the way they handle dates converting types. For example, in MySQL one could only use CAST(), but in SQL Server we can also use CONVERT(). CONVERT takes two arguments. The first is the data type we'd like to convert to, while the second is the name of the column you want to convert. For example, say you want to figure out how many orders your company is getting per day but the column "OrderDate" is an annoyingly long date-time type that you'd need to drop the time from in order to use GROUP BY or just because it's an eyesore:

```
SELECT 
  CONVERT(DATE, OrderDate), 
  SELECT(*)
  
FROM orders

GROUP BY CONVERT(DATE, OrdertDate)
```

Now let's do something more fun with dates - pretend we sell ouija boards. Using DATEPART() allows us to pull a subset of the date of our choice. For example, let's see if we can figure out how many orders we got on the 13th day of each month to attempt (poorly) to figure how many of our customers are superstitious, and how many are not.

```
SELECT
	COUNT(OrderID) AS OrdersPlaced,
    "StartDate" = CASE WHEN DATEPART(DAY, OrderDate) = 13 THEN 'Non-superstitious'
					   WHEN DATEPART(DAY, OrderDate) > 0 THEN 'Superstitious' END
FROM orders
GROUP BY
	CASE WHEN DATEPART(DAY, OrderDate) = 13 THEN 'Non-superstitious'
					   WHEN DATEPART(DAY, OrderDate) > 0 THEN 'Superstitious' END
```            

If we wanted to figure out how many hours people spend on our website per week, we can combine two other functions - DATENAME() and DATEDIFF(). DATENAME serves a similar function as DATEPART, in that it extracts part of a date. However, DATENAME returns a character string that represents a specific part of a date or time value, while DATEPART returns an integer value that represents a specific part of a date or time value. Both functions can be useful depending on what you want to do with the extracted value.


SELECT GETDATE() is a crucial command you'll need for the next blog post (which will be all about Stored Procedures). This returns the current datetime from your computer, and is much more accurate and consistent than selecting this yourself. A good-practice way to use GETDATE() is to set it to a variable right away:

```
DECLARE @CurrentDateTime datetime
SET @CurrentDateTime = GETDATE()
```

Say we wanted to take this a step further and get the number of orders from yesterday's date. For this we can use DATEADD().

```
SELECT COUNT(*)
FROM orders
WHERE CAST(OrderDate AS date) = DATEADD(d,-1,GETDATE())
```
The WHERE clause casts OrderDate as a date type, then ensures that it's equal to yesterday's date. The syntax for DATEADD is first the part of the date you want to add/subtract to (in this case "d" which is day), then the amount you want to add/subtract, and finally the date you would like to perform this operation on.

