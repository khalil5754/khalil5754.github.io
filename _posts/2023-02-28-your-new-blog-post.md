## Using MS SQL and Power BI: The Correlation Between Home Runs and Total Career Earnings

### What is Normalization?

Sometimes tables have a lot of redundant data, whether it's a country code _and_ full country name, or something even less useful, this can be a strain on the performance of a SQL query. To avoid redundant data, normalization transforms a dataset so it has a standard scale. The goal is to make the dataset easier to compare and analyze. For example if we had our penguin weight in grams, but our penguin's heights were in feet (or inches), we could choose to normalize the two by converting both to percentiles or the same unit of measure. More commonly, normalization is used to turn a skewed distribution into a normalized or "symmetrical" distribution, becoming easier to apply statistical analysis to.

Denormalization, on the other hand, is quite the opposite. Our focus is not on reducing redundancy but improving search capability. This is because when one reduces redundancy in a SQL table, the redundancies are moved to external tables, thus it becomes more time-consuming and resource-heavy to run queries on these multiple tables. Denormalization takes the split tables and merges them into one single table.

When you have a system wherein you are inputting or loading data into a database, typically you want to normalize the data to make it more lightweight and less redundant. However, when you want to pull data from the table, you want the searching time for SQL to be as short as possible, which is when you would want to implement de-normalization.

In high-capacity systems, such as databases with millions of lines of code, you will typically need two databases. The OLTP database that is normalized for maximum updating, deleting, or inserting efficiency, and the OLAP system which is denormalized is intended to be for fast SELECT statements and queries. These databases work in tandem and form the bedrock of the ETL (Extract, Transform, Load) process, which we'll go over in further detail in a future post.


## The normalization process: 1st, 2nd, and 3rd normal forms

Remember APT: Atomic, Partial, Transient.

1st normal form: No repeating groups. This essentially means if you load your dataset as a csv, there should be no more than one input per cell. If this is not the case the columns should be split to ensure atomic (unique) values.

2nd normal form: 1st normal form rules still apply, but all non-key columns should be _fully_ dependent on the primary key. This can be achieved by splitting a table to ensure each column is fully dependent on the PK.

3rd normal form: All previous form rules still apply, but without transient dependency. This means no non-key column should depend on any other non-key column.

To achieve normalization in SQL, we can create a series of tables and relationships using SQL statements such as CREATE TABLE, ALTER TABLE, and JOIN. For example, in order to create a normalized database for a customer orders system, we can create separate tables for customers, orders, and order details, with relationships between them based on the customer ID and order ID.

Here's an example of a CREATE TABLE statement for a normalized customer table:

```
CREATE TABLE Customers (
   CustomerID int PRIMARY KEY,
   FirstName varchar(50),
   LastName varchar(50),
   Email varchar(100),
   Phone varchar(20),
   Province varchar(50),
   TotalPurchase int(10)
);
```

## Unique Key VS. Primary Key

The difference is simple, but important - primary keys can not have nulls, while unique keys can have nulls. Also, there can only be one primary key while there can be many unique keys.

Why are we using char instead of varchar or nchar? Well char is a fixed character length, while varchar is flexible. While I'm at it let me tell you that the n in "ncode", on the other hand, makes it so non-english inputs work - however, this means unicode is supported which will make each char 2 bytes, thus increasing computational load.


## How can we increase the performance of our MS SQL query? 

This is an _excellent_ question. First, you can use indexes to increase search performance in MS SQL. Imagine you have a dataset that spans numbers 1-100. What indexing does is build two nodes to make searching through the dataset faster. The first node can be values less than or equal to 50 and of course spans 1-50, while the second node would be all values greater than 50, and spans 51-99. This way, if we want SQL to pull the number 68, it knows immediately to ignore the first node and thus cancels out half of the work.
That index is referred to as a clustered index, but indexes can also be more high-level and store no actual data but instead point to the clustered index nodes which do store the actual data.

To create an index, MS SQL has the CREATE INDEX function, which just needs to know which column and which table you want to use to create the index. Here's an example using our previous table: 

```
CREATE INDEX idx_total_purchase ON TotalPurchase (Customers);
```

Important to note is that while indexes do increase performance, there are still some drawbacks to consider. Indexing requires more storage space for our newly created nodes, and they will also create a slight slowdown in write speed.


##SQL Server's Unique Merge Function

SQL Server and MySQL share many similarities - their differences mainly come from scalability (SQL Server scales better) and security wherein SQL Server is also inherently more secure due to built-in encryption and role-based access control. There are small functionality differences, such as the use of a MERGE command in MS SQL which can perform INSERT, UPDATE, and DELETE functions in just one line. The syntax differences are relatively small - LIMIT in MySQL is TOP in MS SQL and instead of CAST, MS SQL uses TRY_CONVERT. Overall for an enterprise focused on security, performance, and scalability, MS SQL is likely the right choice. 

Okay. Now let's work on a mini-project leveraging MS SQL - then see what Power BI can do with the data we pull. 


We have a table with the salary data of baseball players spanning about 40 years. Another table has the statistical performance of each baseball player spanning over a century. Let's look into our batters in particular. Only looking at years after 1984, which is when salary data became publicly available in the MLB, what is the correlation between the average number of home runs a player hits throughout their career, and their total career earnings?

We have but four columns we're concerned with: Year, Home Runs, PlayerID (PK), and Salary.
Let's see what we can do using the power of Common Table Expressions: 

```
WITH cte_homeruns AS (
    SELECT playerID, 
           SUM(NumberOfHomeruns) AS sum_homeruns 
    FROM batting
    WHERE TRY_CONVERT(int, year) > 1985
    GROUP BY playerID
),
cte_salaries AS (
    SELECT playerID, 
           SUM(salary) AS total_salary 
    FROM salaries
    GROUP BY playerID
)
SELECT cte_homeruns.playerID,
       cte_homeruns.sum_homeruns,
       cte_salaries.total_salary
FROM cte_homeruns
INNER JOIN cte_salaries
ON cte_homeruns.playerID = cte_salaries.playerID
ORDER BY cte_homeruns.sum_homeruns DESC;

```
This code uses a CTE to first filter out any records with invalid birthdates by casting the birthdate column as a date datatype using the TRY_CONVERT() function to determine if the cast was successful. If the cast was successful, the record is included in the main query, which pulls data from the CTE. The cte_salaries query simply exists to calculate the SUM of each player's career earnings. Finally, we select all columns from the PlayerStats CTE and order the results by the average home runs in descending order.

To improve performance, we can also add indexes on the batting and salaries tables on the playerID and year columns, as these are used in the WHERE clauses for filtering the data. For example, we can create the following indexes:

```
CREATE INDEX idx_batting_player_year ON batting(playerID, year);
CREATE INDEX idx_salaries_player_year ON salaries(playerID, year);
```

These indexes can improve query performance by allowing the database to quickly find the relevant rows for each player and year, without having to scan the entire table.

Now let's create a Power BI model to put the data in perspective:

![image](https://user-images.githubusercontent.com/44441178/222619161-0be5e274-5b43-4089-b104-f71cde7e6bf8.png)

Of course, we expected a correlation - but it is always fun to see what we can do with a SQL-manipulated dataset. To be clear, this whole project could have been done entirely in Power BI (or Tableau), but to keep a skill sharp you have to use it, and I like my SQL skills sharp.

You may have noticed there is one big problem with our data, however. It is intuitive that a baseball player who has scored more home runs, is also more likely to have been in the MLB longer, thus also earning more money! While we're at it note that having the size of markers be tied to the amount of salary is good, but not when we have already visualized what we want to sufficiently. All we're doing after that point is adding unneccesary visual clutter.

Instead of committing to SUMS of values, let us work with averages. Namely: Average home runs plotted against average yearly salary. 

![Screenshot 2023-03-02 at 9 55 52 PM](https://user-images.githubusercontent.com/44441178/222620215-a03b7d53-eba2-4609-9530-55bb7c82ddaf.png)

Much better. Remember to always question your own approach if things don't seem right! There's no shame in catching a mistake. In case you were curious, I used slicers in Power BI to only target years after 1984 and had the report's filters exclude any batters who hadn't scored a single home run.

In the next section of this blog we'll be further diving into some baseball datasets and using some more powerful Power BI tools such as DAX functions to manipulate and pull more actionable data. Stay tuned for an update this week!


Thanks for reading,

Khalil (TheStatsGuy)
