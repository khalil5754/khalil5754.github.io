## Using MS SQL and Power BI to Deliver Value to Stakeholders:

What is normalization?

To avoid redundant data, normalization transforms a dataset so it has a standard scale. The goal is to make the dataset easier to compare and analyze. For example if we had our penguin weight in grams, but our penguin's heights were in feet (or inches), we could choose to normalize the two by converting both to percentiles or the same unit of measure. More commonly, normalization is used to turn a skewed distribution into a normalized or "symmetrical" distribution, becoming easier to apply statistical analysis to.
Denormalization, on the other hand, our focus is not on reducing redundancy but improving search capability. This is because when one reduces redundancy in a SQL table, the redundancies are moved to external tables, thus it becomes more time-consuming and resource-heavy to run queries on these multiple tables. Denormalization takes the split tables and merges them into one single table.

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

Simple, primary keys can not have nulls, but unique keys can have nulls. Also, there can only be one primary key while there can be many unique keys.

Why are we using char instead of varchar or nchar? Well char is a fixed character length, while varchar is flexible. The n in ncode, on the other hand, makes it so non-english inputs work - however, this means unicode is supporting which will make each char 2 bytes. 


## How can we increase the performance of our MS SQL query? 

This is an _excellent_ question. First, you can use indexes to increase search performance in MS SQL. Imagine you have a dataset that spans numbers 1-100. What indexing does is build two nodes to make searching through the dataset faster. The first node can be values less than or equal to 50 and of course spans 1-50, while the second node would be all values greater than 50, and spans 51-99. This way, if we want SQL to pull the number 68, it knows immediately to ignore the first node and thus cancels out half of the work.
That index is referred to as a clustered index, but indexes can also be more high-level and store no actual data but instead point to the clustered index nodes which do store the actual data.

To create an index, MS SQL has the CREATE INDEX function, which just needs to know which column and which table you want to use to create the index. Here's an example using our previous table: 

```
CREATE INDEX idx_total_purchase ON TotalPurchase (Customers);
```

Important to note is that while indexes do increase performance, there is still some drawbacks to consider. Indexing requires more storage space for our newly created nodes, and they will also create a slight slowdown in write speed.



