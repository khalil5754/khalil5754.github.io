# Using Stored Procedures, Imputation, Variables, and While loops in SQL Server.

Many of SQL's most useful tools are much less well known than one would expect. 

### Working with NULL values.
In this blog post I'd like to get progressively more complex and interesting as we get deeper in the post. So before we get into COALESCE or the even more interesting stored procedures, let me show you one very easy way to replace a singular column's values with a value of your choice in the case that the value is missing. Using the ISNULL(x, 'y') function allows you to select a column _x_, and in the case that a value in column _x_ is null, it will be replaced with the value 'y'. Simple. You can also choose to replace the null value with another column in the same table. To do this, just ensure the second argument in the ISNULL() function is a column name (without quotes, of course). 

Now, more interesting - using Coalesce! What happens if we'd like to replace NULL values in one column with another column's value, *and* we want to ensure the column we're using to replace the NULL values isn't itself a NULL value?

COALESCE's syntax may be a bit confusing to grasp due to the fact that it takes multiple arguments. Let's say we have an airport's table with country codes in one column, province names in another, city names, and airport codes. 

In case we want to select airport codes but we're worried about NULL values, we can use COALESCE like this: 

```
COALESCE(airport_code, city_name, province_name)
```

This single line is checking for the first non-null value among three columns: "airport_code", "city_name", and "province_name".
So if the "airport_code" column has a non-null value, its value will be returned. If "airport_code" is null, the function will check if "city_name" has a non-null value and return that. If both "airport_code" and "city_name" are null, it will check for a non-null value in the "province_name" column and return that.

### Variables & While Loops
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

First, I think it's fairly easy to tell what's going on in the above code if you've had practically any exposure to programming. That being said, the BREAK command is unnecessary and just serves to show you that the while loop can be broken before it finishes its loop.

### Stored Procedures
