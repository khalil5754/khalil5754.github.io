# Using MySQL to Create a Simple Database and Write Queries

## Looking for a fast solution to querying?
Look no further. 
My personal favourite RDBMS (Relational Database Management System) is MySQL. Notoriously faster (and simpler) than its cousin the Object-Relational Database Management System PostgreSQL, MySQL is practically written in plain english and as a result, is the easiest for a newbie to learn (in my opinion). That is why I'm going to walk you through how to create a small database from scratch in MySQL, followed by some sample queries.

Don't worry, we won't go too complex for the first SQL post. I will throw one or two small subqueries at you, but no Common Table Expressions (a type of subquery that comes _before_ the main query and makes subqueries much faster and more efficient to process) until the next SQL post. 
Of course, before you can query a database you have to create it. Assuming you've got your IDE of choice open, the first command you need is self-explanatory "CREATE TABLE".

Let's say we wanted to create a basic database for my University's courses and students. 
First, we'll create a table with all the courses offered by the University. Within that table we want a coloumn for the course number, the title of the course, and the department administering it:


```
CREATE TABLE IF NOT EXISTS `Course` (
  `CNum` int(6) unsigned NOT NULL,
  `Title` varchar(20) NOT NULL,
  `Dept` varchar(10) NOT NULL,
  PRIMARY KEY (`CNum`)
);
```

Ta-Da! Your first ever table has been created. Or at least, we've told SQL to create the coloumns which will soon form our database. "CNum" (the course number) has been indicated to be an integer, and the "Department" and "Title" coloumns are both strings of length n. If you've noticed, we also have a The primary key of a relation is a candidate key designated as the main key of the relation, essentially a coloumn with unique values that can be related to another table. This is something we can refer to more in our PostgreSQL analysis.

Now, you could go ahead and use this table already but you'll soon find that your queries aren't returning anything. Why is that? You haven't inserted any values into your table yet!

To remedy this, we use the INSERT INTO function, which is, again, self-explanatory. But let me show you:

```
INSERT INTO `Course` (`CNum`, `Title`, `Dept`) VALUES
  ('2310', 'Discrete', 'SoCS'),
  ('3340', 'Database', 'SoCS'),
  ('4930', 'Economics', 'ECON'),
  ('3400', 'Finance', 'FINANCE'),
  ('3360', 'LinAlg', 'MATH'),
  ('1500', 'IntroCS', 'SoCS'),
  ('4000', 'Stats', 'STATS');
```
###### (SoCS indicates the School of Computer Science)
After the INSERT INTO command, you simply tell MySQL which table you want to insert values into, explain which columns these values will be inserted into seperated by a comma, then call VALUES.
In this case, 'Cnum' (course number) is first, so the first value before the comma seperator in each list will be inserted into the 'Course' Database under the CNum coloumn. MySQL parses through each list and inserts the values accordingly.

We have 7 courses listed in no particular order (MySQL can order these with ease if we so request, but more on that later).
This time we've _really_ created a table using SQL!

To create a data_base_ let us now create two more tables, one for the students at the University, and one table indicating which courses each student is taking, as well as their grades and the semester they took each course.

Remember: first, CREATE TABLE, then INSERT INTO:

```
CREATE TABLE IF NOT EXISTS `Student` (
  `ID` int(8) unsigned NOT NULL,
  `Name` varchar(20) NOT NULL,
  `Program` varchar(20) NOT NULL,
  PRIMARY KEY (`ID`)
);
INSERT INTO `Student` (`ID`, `Name`, `Program`) VALUES
  ('1017402', 'Khalil', 'CS'),
  ('1008577', 'Raza', 'CS'),
  ('1018500', 'Joss', 'CS'),
  ('1696969', 'James', 'FIN'),
  ('1712948', 'Jones', 'FIN');
```
The primary key is 'ID' because 'Name' and 'Program' are going to inevitably have duplicates, but student IDs are unique. Right now, we don't have another table which relates to this one, but we'll remedy that in a quick second.
Here we've inserted into the Student table the individual ID numbers, names, and programs using myself, a few friends of mine, and two fake names (I'm not telling you which is which) as inspiration. 


Let's create our final table, "Taking" - which will relate to _both_ the Student table and the Course table:

```
CREATE TABLE IF NOT EXISTS `Taking` (
  `ID` int(8) unsigned NOT NULL,
  `Cnum` int(6) unsigned NOT NULL,
  `Grade` int(4) unsigned NOT NULL,
  `Semester` varchar(5) NOT NULL,
  PRIMARY KEY (`ID`,`Cnum`)
);
INSERT INTO `Taking` (`ID`, `Cnum`, `Grade`, `Semester`) VALUES
  ('1017402', '2310', '99', 'W22'),
  ('1017402', '1500', '90', 'F22'),
  ('1017402', '3340', '60', 'F22'),
  ('1008577', '3360', '51', 'W22'),
  ('1018500', '4930', '88', 'W22'),
  ('1668902', '3400', '70', 'W22'),
  ('1712948', '3400', '81', 'W22'),
  ('1712948', '4000', '81', 'F22'),
  ('1668902', '3340', '60', 'W22'),
  ('1668902', '1500', '98', 'W22');
```

Here you can see there are two unique coloumns SQL has been told to use as primary keys. We've finally completed creating our database, as this third table has primary keys that relate to both of our other tables!


## Now for the fun part: Querying.

For our first query, we'll do an INNER JOIN. This type of join selects the records that have common values in both coloumns being joined. We can do an INNER JOIN to combine all three columns, like so:

```
SELECT *
  FROM Taking AS t
  INNER JOIN Student AS s
  ON t.ID = s.ID
  INNER JOIN Course AS c
  ON t.CNum = c.CNum
 ```
The first line SELECT * selects every column, so long as it satisfies the query's restrictions. In this case, the query has no restriction on records so long as it satisfies the INNER JOIN.
Of course, you can't simply tell SQL to retrieve records without telling it from where. This is where the FROM command comes into play, telling SQL which Table you want to pull from. The AS command is also known as "aliasing" and essentially renames your table for the query in question, allowing for better readability and writability. 
The ON command _must_ come after any join, as without it SQL won't know what value to combine the tables on. In this case, first, SQL combines the Taking table and the Student table using IDs as the common key. Then, the Course table is added to the new aggregate table ON course numbers (CNum). 

This is the result of our query:

<img width="1139" alt="image" src="https://user-images.githubusercontent.com/44441178/197077430-d1b874b9-d4e8-452a-afee-36937ef7d8c8.png">

As you can see, all the records have been returned successfully!

Now, let's do something a little more interesting. Let's see the names and IDs of the students who achieved a minimum grade of 80 in at least one of their classes in the Winter 2022 semester.

 ```
 SELECT s.ID, s.Name, 80

    FROM Student AS s
    INNER JOIN Taking AS t 
    ON t.ID = s.ID
    WHERE t.Semester = 'W22'
    GROUP BY s.ID
    HAVING MIN(Grade) > 80
 ```
 Now we make a slight jump and introduce the crucial GROUP BY, HAVING and WHERE functions. These functions are powerful, but become more powerful when their power is combined. The WHERE function is simple. It indicates a condition which must be true for the query to return the records requested in the SELECT statement. The WHERE function must come before the GROUP BY function, which is a big reason why we cannot force it to bear the weight of an aggregation, such as MIN() - which returns the minimum integer value. 
 The GROUP BY function groups our records by a specific column, paving the way for us to compute aggregate functions - something the HAVING function was born to do. 
 
 Before the GROUP BY clause, the WHERE clause tells SQL we only want to return records WHERE the semester is equal to 'W22'. We don't want any other semesters. 
 Then, GROUP BY groups by students' IDs and shows the Names and IDs requested in the SELECT STATEMENT if and only if the student has a minimum grade of greater than 80.
 
 Here's the result:
 
 <img width="1059" alt="image" src="https://user-images.githubusercontent.com/44441178/197079685-337acb37-b3da-4cd3-b731-9b3f14f5f6f9.png">

 Look at Khalil, Joss, and Jones go! :)
 
 
 ## Subqueries and Common Table Expressions
 
 Let's go just a tad deeper now and see how you can embed a query _within_ a query using subqueries, and then how you can rewrite the same query to be more efficient and faster (and increase readability) using CTES in a later post dedicated to SQL efficiency.
 
 For this query, we're looking for students who took _no_ courses offered by the SoCS. We'll look at how to do this with a subquery:
 
 ```
 SELECT s.ID, s.Name, c.Title 
    FROM Student AS s
    INNER JOIN Taking AS t 
    USING(ID)
    INNER JOIN Course AS c
    USING(CNum)
    WHERE t.Semester = 'W22' AND s.ID NOT IN (SELECT s.ID 
                                                FROM Student AS s 
                                                INNER JOIN  
                                                Taking AS t 
                                                ON s.ID = t.ID
                                                INNER JOIN Course AS c
                                                ON c.Cnum = t.Cnum
                                                WHERE t.Semester = 'W22' AND c.Dept = 'SoCS')
    GROUP BY s.ID
  ```

If you have a sharp eye you'll notice the ON command has been replaced with USING. USING is useful when the common values you're using are the same. 
The more interesting portion of this specific query may not be the USING command, but the subquery at the end of of the WHERE clause. In a subquery, the main query's result is dependent on the records returned by the subquery. This allows for much greater flexibility and a wider array of possibilities by overcoming the limits of only having one query, as the subquery can act as a 'force multiplier' of sorts for the main query.
In this case, the subquery returns all the students' IDs that _did_ take a Winter '22 SoCS course. 
The WHERE clause ensures the semester in the records being pulled is 'W22' AND, using the NOT IN function, only pulls student IDs that aren't returned in the subquery. This ensures that any students who took a SoCS course are not included in the final result-set.

This returns:

<img width="1059" alt="image" src="https://user-images.githubusercontent.com/44441178/197080929-2e958846-c7c6-4f6b-9df8-83945afb15e8.png">

But we can make this more efficient using CTEs! Tune into that post soon.


Let's build one more query. I'm sure you're tired by now so can make this final one an easy one. Lets pull the course numbers, titles, and average grades of W22 courses having 2 or more students.

```
SELECT c.Cnum, c.Title, AVG(t.Grade) AS avg_grade
    FROM Course AS c 
    INNER JOIN Taking AS t 
    ON c.Cnum = t.Cnum
    GROUP BY c.Cnum
    HAVING COUNT(DISTINCT ID) >= 2
    ORDER BY c.Cnum DESC
```

We can use an aggregate function (in this case, the AVG function) when we SELECT the record that we need to pull. The AVG() function returns the average grade, aliased as avg_grade for readability in the resulting table.
However, don't forget that if you have an aggregate function you also need a GROUP BY function for SQL to run the aggregate function over. Otherwise, MySQL won't know what to you want it to find the average with respect to.
HAVING COUNT(DISTINCT ID) >= 2 may seem like a verbose line of code, but in reality it's quite simple. The COUNT command counts the number of **distinct** IDs taking each course. The HAVING command then only returns true if the number of **distinct** IDs in a course are greater than or equal to 2. That's it. Simple!
The last line in the query, the ORDER BY clause orders the queries results by course number, in descending order.

This returns:

<img width="1059" alt="image" src="https://user-images.githubusercontent.com/44441178/197108895-fd779018-84a3-49ba-989d-5156ca4625dc.png">

See? SQL isn't so bad! I love it because it feels like I'm talking to my computer.

This post _barely_ scraped the surface of the power SQL allows a skilled programmer to wield. There are many ways to make queries more efficient and more readable (you don't have to use AS everytime you alias, for example: "FROM Course c" is the same as "FROM Course AS c"). 
We also only touched on inner joins in this post, but there are tons of other kinds of joins which have their own use cases, such as left joins, full joins, self joins, natural joins, etc.. But we'll talk about those in the same post that I introduce you to CTEs.

For now: signing off,

Khalil (TheStatsGuy)



