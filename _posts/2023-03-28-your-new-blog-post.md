# Using Stored Procedures, Imputation, and Variables in SQL.

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

### Variables




### Stored Procedures
