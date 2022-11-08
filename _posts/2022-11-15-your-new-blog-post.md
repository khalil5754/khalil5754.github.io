# Using PostgreSQL to Build an Object-relational Database and Write Parametric Queries

I'll be honest: while I've been ecstatic about writing all of the tutorials/posts I've written thus far, PostgreSQL is my least favourite database management language. *With that being said*, it is undoubtedly a very powerful tool in its own right, especially in situations where you need something larger or more complex than what MySQL can offer. 
For that reason, let us begin our PostgreSQL tutorial with the first step: 

## Building the Database

Unlike writing queries, building a database in Postgres is markedly similar to doing it in MySQL, with a few important syntax differences. Because of this, let's take a different approach to this tutorial than usual. Instead of walking through every block of code, this time, I'm going to show you the code for building your database (remember CREATE TABLE and INSERT INTO from the MySQL post?) and explain what's different in Postgres compared to MySQL.

Assuming you have Postgres installed (or you're using a virtual machine), we can begin. Fair warning: this will be the largest single block of code I've posted thus far, but the majority of the commands I've gone over in the previous SQL post, so it's redundant to go through it step-by-step again.


Let's create "Vendor", "Customer", and "Transactions" tables, and insert into each some simple records:

```
CREATE TABLE IF NOT EXISTS Vendor (
  Vno char(5),
  Vname char(20),
  City char(10),
  Vbalance numeric(10,2),
  PRIMARY KEY (Vno)
);
INSERT INTO Vendor (Vno, Vname, City, Vbalance) VALUES
  ('V1', 'IKEA', 'Toronto', 200.00),
  ('V2', 'Walmart', 'Waterloo', 671.05),
  ('V3', 'Esso', 'Windsor', 0.00),
  ('V4', 'Esso', 'Waterloo', 225.00);
  
CREATE TABLE IF NOT EXISTS Customer (
  Account char(5),
  Cname char(20),
  Province char(10),
  Cbalance numeric(10,2),
  Crlimit integer,
  PRIMARY KEY (Account)
);
INSERT INTO Customer (Account, Cname, Province, Cbalance, Crlimit) VALUES
  ('A1', 'Smith', 'ONT', 2515.00, 2000),
  ('A2', 'Jones', 'BC', 2014.00, 2500),
  ('A3', 'Doc', 'ONT', 150, 1000);
 
 CREATE TABLE IF NOT EXISTS Transactions (
  Tno char(5),
  Vno char(5),
  Account char(5),
  T_date DATE,
  Amount numeric(10,2), 
  PRIMARY KEY (Tno)
);
INSERT INTO Transactions (Tno, Vno, Account, T_date, Amount) VALUES
  ('T1', 'V2', 'A1', '2022-07-15', 1325.00),
  ('T2', 'V2', 'A3', '2021-12-16', 1900.00),
  ('T3', 'V3', 'A1', '2022-09-01', 2500.00),
  ('T4', 'V4', 'A2', '2022-03-20', 1613.00),
  ('T5', 'V4', 'A3', '2022-07-31', 2212.00);
CREATE TABLE IF NOT EXISTS Vendor (
  Vno char(5),
  Vname char(20),
  City char(10),
  Vbalance numeric(10,2),
  PRIMARY KEY (Vno)
);
INSERT INTO Vendor (Vno, Vname, City, Vbalance) VALUES
  ('V1', 'IKEA', 'Toronto', 200.00),
  ('V2', 'Walmart', 'Waterloo', 671.05),
  ('V3', 'Esso', 'Windsor', 0.00),
  ('V4', 'Esso', 'Waterloo', 225.00);
  
CREATE TABLE IF NOT EXISTS Customer (
  Account char(5),
  Cname char(20),
  Province char(10),
  Cbalance numeric(10,2),
  Crlimit integer,
  PRIMARY KEY (Account)
);
INSERT INTO Customer (Account, Cname, Province, Cbalance, Crlimit) VALUES
  ('A1', 'Smith', 'ONT', 2515.00, 2000),
  ('A2', 'Jones', 'BC', 2014.00, 2500),
  ('A3', 'Doc', 'ONT', 150, 1000);
 
 CREATE TABLE IF NOT EXISTS Transactions (
  Tno char(5),
  Vno char(5),
  Account char(5),
  T_date DATE,
  Amount numeric(10,2), 
  PRIMARY KEY (Tno)
);
INSERT INTO Transactions (Tno, Vno, Account, T_date, Amount) VALUES
  ('T1', 'V2', 'A1', '2022-07-15', 1325.00),
  ('T2', 'V2', 'A3', '2021-12-16', 1900.00),
  ('T3', 'V3', 'A1', '2022-09-01', 2500.00),
  ('T4', 'V4', 'A2', '2022-03-20', 1613.00),
  ('T5', 'V4', 'A3', '2022-07-31', 2212.00);
  ```
  
 Right off the bat, you'll notice there are no single quotes around the names of the tables we create, nor around the names of the variables declared. 
 In the INSERT INTO function, we still don't use quotes around the table name, nor around the variables declared in the CREATE TABLE function. But there *are* quotes around the strings we insert into the tables.
 It is, however, crucial to remember integers (both with and without decimals) are to be fed to your function without quotes, but dates are not the same. 
  
Honestly, that's largely it as far as differences go in the table-building phase of using Postgres. Fairly easy to follow so far right?


## Writing Queries in PostgreSQL

  This is where it gets interesting. There are more powerful tools at your disposal through PostgreSQL than there are through MySQL, such as loops and variables. There are also many ways to write PostgreSQL queries, including writing functions with or without parameters straight through your terminal. I'll show an example of this, as well as a couple of different methods of writing Postgres queeries.
  
  
  ### Writing a function to insert a new customer tuple.
  
  Say we have a new customer that we would like to add to our database. Let's also assume this is a common occurrence at our analytics company, and we want to ensure anyone at the company - including non-programmers - can add this customer's data to the database simply by inputting parameters.
  
  I know what you're thinking: "Accepting input? Parameters? In *SQL*??"
  
  It seems crazy, but I'll show you:
  
``` 
CREATE FUNCTION new_customer(theAccount CHAR, theName CHAR, theProvince CHAR, theCrlimit integer) RETURNS TABLE (Account CHAR, Cname CHAR, Province CHAR, Cbalance numeric(10,2), Crlimit integer) AS $results$

BEGIN
INSERT INTO Customer VALUES
  (theAccount, theName, theProvince, 0.00, theCrlimit);  
RETURN QUERY

    SELECT *
    FROM Customer;
END;

$results$ LANGUAGE plpgsql;
SELECT * FROM new_customer('A4', 'Khalil', 'MAN', 900);
 ```
 
 If you've read the MySQL post (two posts back), then you'll notice the above query is not as similar to a MySQL query as the creation of the table was.
 Right off the bat we do something quite different: create a function that accepts parameters, and name each of those parameters (for example, theAccount and theName are user-input assigned variables). To do this you use the CREATE FUNCTION command, followed by the name of your function, and then in brackets, each of the parameters you want the program to accept. Then RETURN TABLE tells the function what to return to the user. In this case, we want each customer's records including: Account, Name, Province, Balance, and Credit Limit.
 Then, we can use BEGIN to write whatever we want the query to know before it runs the query. 
 To put it more simply, using BEGIN followed by inserting our user inputs into the "Customer" table, we're telling Postgres to pre-emptively prepare the table for the subsequent query, which begins at RETURN QUERY.
 
 Then we simply SELECT * - of course selecting every record from the previously updated "Customer" table, followed by the ever-important "END;", which serves as a "break" and tells Postgres when we're ready for the next section to begin.
 
 Next, we tell the program what language we're using, in this case plpgsql, which is the procedural Postgres language.
 Finally, we can write the code which serves as the user input, calling the "new_customer" function and giving it the parameters it needs.
 
 In case you haven't noticed, something interesting about PostgreSQL is that after every full command, we need to give it a semi-colon. This is because of the inherent requirement in Postgres where functions need to know exactly when and where each command ends. However, once we run this code, we're presented with a problem. If I try to pull the records of transactions for each customer but our new customer has yet to create a transaction, what will be displayed? Well, nothing. Or NULL to be more specific. We don't want that because it makes it difficult to tell when nothing was pulled due to an error, or if there's simply no record for the transaction. This makes bug-hunting unnecessarily difficult. To remedy this, we can employ loops and variables using a different Postgres method.
 
 ### Writing a query to display no transaction
 Let's say we want the program to display "no transaction" for any customer with no transaction, such as our recently added tuple.
 For that, there is RAISE NOTICE, LOOP, and DECLARE RECORD:
 
 ```
 CREATE OR REPLACE FUNCTION display_transactions() RETURNS TABLE(Account CHAR, Cname CHAR, Amount NUMERIC, Vname CHAR) AS $results$

DECLARE notran RECORD;

BEGIN 
    FOR notran in SELECT c.Account FROM customer AS c WHERE c.Account NOT IN (SELECT t.Account FROM transactions AS t)

    LOOP
        RAISE NOTICE 'Customer % has no transactions', notran.Account;
    END LOOP;

    RETURN QUERY 
    SELECT t.Account AS Account, c.Cname AS Customer_Name, t.Amount AS Amount, v.Vname AS Vendor_Name
    FROM transactions AS t 
    NATURAL JOIN (
        SELECT t.Account, Max(t_date) AS t_date 
        FROM transactions AS t 
        GROUP BY t.Account
    ) AS t2
    NATURAL JOIN customer AS c 
    NATURAL JOIN vendor AS v; 
END;

$results$ LANGUAGE plpgsql;
 ```
 
First, the CREATE OR REPLACE function ensures every time this function is run, it replaces the previous time - ensuring the function isn't replicated. 
Then, we declare a new RECORD which will be used in our FOR statement. This new record - "notran" - serves as a variable of sorts. As you likely already know, SQL doesn't usually allow the declaration of variables (nor does it allow loops) - but Postgres has a few _loop_ holes (get it?) which allow you to accomplish these incredibly useful tasks. Using DECLARE notran RECORD, we assign to notran any account that is not in the "transactions" table. Through our notran variable we can then tell the program to loop through and display "Customer '_x_' has no transactions" if their account is not in the transactions record. All of this occurs prior to the actual main Postgres query running, as the query needs the results of this loop to return what we're looking for. In the main query, we select the account, customer name, amount, and vendor name. We then NATURAL JOIN (this is just like the everyday SQL join you're familiar with, except it tells the program to automatically choose the common record to join the tables on) a subquery that takes the most recent date from transactions for each account...
 
 
 ### Let's combine both types of queries and do one more.
 
 ```
CREATE OR REPLACE FUNCTION p8(theVno CHAR, theAccount CHAR, theAmount NUMERIC(10, 2)) RETURNS TABLE(Tno CHAR, Vno CHAR, Account CHAR, T_date DATE, Amount NUMERIC, Vbalance NUMERIC, Cbalance NUMERIC) AS $results$

DECLARE theTno CHAR(5);

BEGIN 
    -- Validate the vendor number
    IF NOT (SELECT EXISTS (SELECT * FROM vendor as v WHERE v.Vno = theVno)) THEN 
        RAISE EXCEPTION 'Invalid vendor number';
    END IF;

    -- Validate the customer account
    IF NOT (SELECT EXISTS (SELECT * FROM customer as c WHERE c.Account = theAccount)) THEN 
        RAISE EXCEPTION 'Invalid account';
    END IF;

    -- Generate a new transaction number
    SELECT MAX(t.Tno) INTO theTno FROM transaction as t;

    IF theTno IS NULL THEN 
        theTno := 'T1';
    ELSE 
        theTno := 'T' || (SUBSTRING(theTno, 2, 4)::INTEGER + 1);
    END IF;

    -- Insert the new transaction
    INSERT INTO transaction VALUES (theTno, theVno, theAccount, CURRENT_DATE, theAmount);

    -- Add the transaction amount to the vendor's balance
    UPDATE vendor as v SET Vbalance = v.Vbalance + theAmount WHERE v.Vno = theVno;

    -- Subtract the transaction amount from the customer's balance
    UPDATE customer as c SET Cbalance = c.Cbalance - theAmount WHERE c.Account = theAccount;

    RETURN QUERY SELECT t.Tno, t.Vno, t.Account, t.t_date, t.Amount, v.Vbalance, c.CBalance FROM transaction as t NATURAL JOIN vendor as v NATURAL JOIN customer as c WHERE t.Tno = theTno AND t.Vno = theVno AND t.Account = theAccount;
END;
$results$ LANGUAGE plpgsql;
```  

In the above code (which I've commented to make it easier to understand, because it's quite long) we add a new transaction. 
Every time the program is run, the progtam takes as user-input: 
- vendor number
- account number
- amount 

It validates these numbers, ensuring they're not a random string of characters, and that they follow a similar format to previous records in the same column. 
The program then generates a transaction number for this new transaction, the date of which will be selected based on whatever date the program is run. The program then stores this transaction, newly made, into the "transactions" table - also updating the balances of the related customer and vendor with the amount of the new transaction. 

Finally, the program displays the new transaction and the updated customer and vendor records.
  
  
  Just like that, we're done! As you can see, writing queries in Postgres is very different to writing them in MySQL. While I don't hate writing in Postgres, I personally certainly prefer MySQL or any other form of a Database Management language (although I'm sure you're tired of hearing that by now).
  
  All in all, Postgres is an important tool in a data scientist's toolbox, and I hope I've shown you in this post why that is, and how to utilize it.
  
  
  Until next time,
  
  Khalil (TheDataNerd)
