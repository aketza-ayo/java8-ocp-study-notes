# Introducing Relational Databases and SQL
A *Database* is an organized collection of data. In the real world, a type cabinet is a type of database. A *relational database* is a database that is organized into tables, which consist of rows and columns. You can think of a table as a spreadsheet. There are two main ways to access a relational database from Java:
- Java Database Connectivity Language (JDBC): Accesses data as rows and columns. JDBC is the API covered in this chapter.
- Java Persistence API (JPA): Accesses data through Java objects using concept called object-relational mapping (ORM). The idea is that you don't have to write as much code, and you get your data in Java objects. JAP is not on the exam, and therefore it is not covered in this chapter.
- A relational database is accessed through Structured Query Language (SQL).
- In addition to relational database, there is another type of database called NoSQL database. This is for database that store their data in format other than tables. NoSQL is out of the scope for the exam as well.
- In the following  section, we introduce a small relational database that we will be using for the examples in this chapter and present the SQL to access it. We will also cover some vocabulary that you need to know.

## Indentifying the Structure of a Relational Database
Our smaple database has two tables. One has a row for each species that is in our zoo. The other has a row for each animal. These two relate to each other because an animal belong to a species. These relationships are why this type of databases is called a relational database. The figure below shows the structure if the database.

![Tables in our relational database](img/tablesInDB.png)

As you can see we have two tables. One is species and the other ios named animal. Each table has a primary key. After all two animals might have the same name but they cannot have the same ID. In this case the primary is just one column in some situations it is a combination of columns. For example, a student identifier and year might be a key.

## Writing Basic SQL Statements
The only thing you need to know for SQL in the exam is that there are 4 types of statements for working with data in tables. Pay attention to the first word of each:

INSERT: Add a new row to the table
SELECT: Retrieve data from the table
UPDATE: Change zero or more rows in the table
DELETE: Remove zero or more rows from the table

That's it. You are not expected to determine if SQL statements are correct. You are not expected to spot syntax errors in SQL statements. You are not expected to write SQL statements. As far as the exam is concerned, joining two tables is a concept that doesn't exist.
Unlike Java, SQL keywords are case insensitive. This means select, SELECT, and Select are all equivalent. Most people use uppercase for the database so that they stand out. It is also common practice to use underscore to separate "words" in column names. We follow these conventions. Now it's time to write some code.

SQL has a number of types. Most are self explanatory like INTEGER, TIMESTAMP. There is also DECIMAL which is like a double in Java. The strangest one is VARCHAR, standing for "variable character" which is like a String in Java. The variable part means that the database should only use as much as it needs to store the value. 
The INSERT statement is usually used to create one new row in a table, for example:

```
INSERT INTO species VALUES (3, 'Asian Elephant', 7.5);
```

If there are two rows in the table before this command is run, then there are three afterwards. The INSERT statement lists the values that we want to insert. By default, it uses the same order in which the columns were defined, String data is enclosed in single quotes.

The SELECT statement reads data from the table.
```
SELECT * FROM species WHERE ID = 3;
```
The WHERE clause is optional. If you omit, the contents of the entire table are returned. The * indicates to return all of the columns in the order in which they are defined. Alternatively, you can list out the columns that you want returned:
```
SELECT name, num_acres FROM species WHERE ID = 3;
```
it is preferable to list the column names for clarity. It also helps if table changes in the database. 

You can also get information about the whole result without returning individual rows using special SQL functions:

```
SELECT COUNT(*), SUM(num_acres) FROM species;
```
This query tells us how many species we have and how much space we need for them. It returns only one row since it is combining information. Even if there are no rows in the table, the query returns one row that contains zero as the answer. 

The UPDATE statement changes one or more rows in the database:

```
UPDATE species SET num_acres = num_acres + .5 WHERE name = 'Asian Elephant';
```

Again, the WHERE clause is optional. If it is omitted, all rows in the table will be updated. The UPDATE statement always specifies the table to update and the column to update.

The DELETE statement deletes one or more rows in the database:

```
DELETE FORM species WHERE name = 'Asian Elephant';
```

And yet again, the WHERE clause is optional. If it is omitted, the entire table will be emptied. So be careful! All of the SQL shown in this section is common across the database. 

# Introducing the Interfaces of JDBC

# Connecting to a Database
## Building a JDBC URL
## Getting a Database Connection

# Obtaining a Statement
## Choosing a ResultSet Type
## Choosing a ResultSet Concurrency Mode

# Executing a Statement

# Getting Data from a ResultSet
## Reading a ResultSet
## Getting Data for a Column
## Scrolling ResultSet

# Closing Database Resources

# Dealing with Exceptions
