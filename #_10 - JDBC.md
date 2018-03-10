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
For the exam you need to know four key interfaces of JDBC. The interfaces are declared in the JDK. This is just like all of the other interfaces and classes that you have seen in this book. List is in the JDK, Path is in the JDK and so forth.

As you know, interfaces need a concrete class to implement them in order to be useful. These concrete classes come from the JDBC driver. Each database has a different JAR file with these classes. For example. PostgreSQL's JAR is called somthing like ```postgresql-9.4-1201-jdbc4.jar``` Whereas MySQL's JAR is called something like ```mysql-connector-java-5.1.36.jar``` The exact name depends on the version of the driver JAR. This driver JAR contqains an implementation of these key interfaces alonmg with a number of others. The key is that the provided implementation know how to communicate with a database. There are different types of drives: luckily, you don't need to know about this for the exam.

The figure below shows four key interfaces that you need to know. It also shows that the implementation is provided by an imaginary Foo driver JAR. They cleverly stick the name Foo in all classes.

You have probably noticed that we didn't tell you what the implementing classes are called in any real database. The main point is that you shouldn't know. With JDBC, you use only interfaces in your code and never implementation classes directly. In fact they might not even be public classes.

![Key JDBC interfaces](img/keyJDBCInterfaces.png)

What do these four interfaces do? On a very high level, we have the following:

Driver: Knows how to get a connection to the database
Connection: Knows how to communicate with the database
Statement: Knows how to run the SQL
ResultSet: knows what was retruned by a SELECT query

All database classes are in the package ```java.sql``` so will omit the imports going foward.

In thisnext example, we show you what JDBC code looks like end to end. If you are new to JDBC, just notice that three of the four interfaces are in the code. If you are experienced, this is what JDBC looks like accoridng to the OCP exam. Yes we know - you wouldn't and shouldn't write code like this. We wouldn't either. It's OK. Grumblr to yourself and it out of your system.

```java
package com.wiley.ocp.connection;

import java.sql.*;

public class MyFirstDatabaseConnection{
  public static void main(String[] args){
    String url = "jdbc:derby:zoo";
    try(Connection conn = DriverManager.getConnection(url);
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("select name from animal")){
    
        while(rs.next()){
          System.out.println(rs.getString(1));
        }
    }
  }
}

```

If the URL were using our imaginary Foo driver, DriverManager would return an instance of FooConnection. Calling createStatement() would return an instance of FooStatement, and calling executeQuery() would return an instance FooResultSet. Since the URL uses derby instead, it returns the implementations that derby has provided for these interfaces. You don't need to know their names. In the rest of the chpater, we will explain hoto use all four of the interfaces and go into more detail about what they do. By the end of the chapter, you will be writting code like this yourself.

# Connecting to a Database
The first step in doing anything with a database is connecting to it. First we will show you how toi build the JDBC URL. Then we will show yuu how the exam wants you to get a Connection to the database.

## Building a JDBC URL
To access a website, you need to know the URl if the website.To access you emails, you need to know username and passwprd. JDBC is no different. In order to access a database, you need to know this information about it. Unlike web urls, JDBC URL has a variety of formats. They have three parts in common, as shown in figure below. There are three if you can count third one as beign in commmon, The first piece is always the same. It is the protocol jdbc. The secodn part is the name of the database such as derby, mysql or postgres. The third part is the rest of it, which is database specific format. Colons separate the three parts.

![The JDBC URL format](img/urlFormat.png)

The third part typically contains the location and the name of the database. The syntax varies. You need to know about the three main parts. You don't need to memorize the vendor specific part. Phew! You've already seen one such URL:

```
jdbc:derby:zoo
```

Notice the three parts. It starts with jdbc, then comes derby and it ends with the database name. Other examples are shown here:

```
jdbc:postgres://localhost/zoo

jdbc:oracle:thin:@123.123.123.123:1521:zoo

jdbc:mysql://localhost:3306/zoo?profileSQL=true
```

You can see that each of these begin with jdbc, followed by a colon, and then followed by the vendor/product name. After that it varies. Notice how all of them include the location of the datbase, which are localhost, 123.123.123.123:1521 and localhost:3306, respectively. Also notice that the port is optional when using the default. Finally, notice that all of them include the name of the datbase, which is zoo.
To make sure you get this, do you see what is wrong with each of the following?

```
jdbc:postgresql://local/zoo

jdbc:mysql://1234567/zoo

jdbc;oracle;this;/lcoalhost/zoo
```

The first one uses local instead of localhost. Localhost is a specially defined name. You can't just make up a name. Granted it is possible for ourserver to be named local, but the exam will not have you assume names. If the database has a special name, the question will let you know. The second one says that the location of the database is 123456. This doesn't make sence. A localtion can be localhost or a IP address or a domain name. It can't be any random number. The third one is no good because it uses semicolons instead of colons.
In some databases, you use an alias rather than the database name. For the purpose of the exam, consider the alias to be logical database name.

## Getting a Database Connection
There are two main ways to get a Connection: DriverManager or DataSource. DriverManager is the one covered on the exam. Do not use a DriverManager in code someone is paying you to write. A DataSource is a factory, and it has more features than DriverManager. For example, it can pool connections or store the database connection info outside the application.

The DriverManager class is in JDK, as it is an API that comes with Java. It uses the factory pattern, which means that you call static method to get a Connection. As you learned in Chapter 2, "Design Patterns and Principles" the factory pattern means that you can get any implementation on the interface when calling the method. The good news is that the method has an easy-to-remember name - getConnection(). To get a connection form the embedded database, you write the following: 

```
import java.sql.*;

public class TestConnect{
  public static void main(String[] args) throws SqlException{
    Connection conn = DriverManager.getConnection("jdbc:derby:zoo");
    System.out.println(conn);
  }
}
```
Running this example as java TestConnect will give you an error that begins with this:

```
Exception in thread "main" java.sql.SQLException: No suitable driver found for jdbc:derby:zoo 
```

The class SQLException means "something went wrong when connecting to or accessing the database". In this case, we didn't tell java where to find the database driver JAR file. Remeber that the implementation class for Connection is found inside a driver JAR. We try this again by adding the classpath with ```java -cp <java_home>/db/lib/derby.jar``` TestConnect. Remember to substitute the location of where Java is installed on your computer for "java_home". (If you are on Windows, replace the colon woth semicolon) This time the program runs successfully and prints something like the following:
```
org.aparache.derby.impl.jdbc.EmbeddedConnection40@1377654657
(XID = 156), (SESSIONID = 1), (DATABASE = zoo), (DRDAID = null)
```

The details of the output arent important. Just notice that the class is not Conneciton. It is a vendor implementation of Connection. There is also a signature that takes a suername and password:

```java
import java.sql.*;

public class TestExternal{
  public static void main(String[] args) throws SQLException{
    Connection conn = DriverManager.getConnection(
      "jdbc:postgresql://localhost:5432/ocp-book",
      "username",
      "password");
      
    System.out.println(conn);
  }
}

```

Notice the three params that are passed to getConnection(). The first is the JDBC URL that you learned about in the previous section. The second is the username for accessing the database, and the third is the password for accessing the database. It should go without saying that our password is not "password". Also, don't put your password in real code. It is horrible practice and Oracle should not be encouraging.

This time, now that we have included the drived file, the pogram rins successfully and prints something like this:

```
org.postgresql.jdbc4.Jdbc4Connection@eed1f14
```

Again, notice that it is driver-specific implementation class. You can tell from the package name. Since the package is org.postgresql.jdbc4, it is part of the PostgresSQL driver. The command line tells Java where to find the driver JAR. It also includes the current directory so Java can find TestConnect itself!

Unless the exam specifies a command line, you can assume the the correct JDBC driver JAR is in the classpath. The exam creators explicitly ask about the driver JAR if they want you to think about it.

The nice thing about a factory is that it takes care of the logic of creating a class for you. You don't need to know the name of the class that implements Connection, and you don't need to know it is created. You are probably a bit curious, though.

The DriverManager class looks through the classpath for JARs that contains a Driver. DriverManager knows that a JAR is a driver because it contains a file called java.sql.Driver in the directory META-INF/services. In other words, a driver might contain this information: 

META-INF
-service
-java.sql.Driver
com
-wiley
-MyDriver.class

Inside the java.sql.Driver file is one line. If it is a fully qualified package name of the Driver implementation class. Remember thouse four key interfaces? Driver is the first one. DriverManager then looks through any driver it can find to see if they can handle the JDBC URL. If so, it creates a Connection using that Driver. If not it gives up and throws a SQLException.

**Real World Scenario** Using a DataSource. In real applications you should use a DataSource rather than a DriverManager to get a Connection. For one thing, there is no reason why you should have to know the database password. It's far better if the database team or another team can set up a data source that you can reference. Another reason is thta a DataSource mantains a connection pool so that you can keep reusing the same connection rather than needing to get a new one each time. Even the JavaDoc says DataSource is preferred over DriverManager. But DriverManager is in the exam objectives, so you still have to know it.

You might see Class.forName() used in older code before getting a Connection. It looks like this:

```
public static void main(String[] args) throws SQLException, ClassNotFoundException{
  Class.forName("org.posgressql.Driver");
  Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost:5432/ocp-book",
    "username",
    "password");
}

```
Class.forName() loads a class. This lets DriverManager use a Driver, even if the JAR doesn't have META-INF/services/java.sql.Driver file. There is no harm in including Class.forName(), even if the newer driver doesn't have a file. When Class.forName() is used, the error about an invalid class occurs on that line and throws a ClassNotFoundExpection:

```
public static void main(String[] args) throws ClassNotFoundException{
  Class.forName("not.a.driver");
}

```

Obviosuly, this is not a valid driver name. The output begins with the following code:

```
Exception in thread "main" java.lang.ClassNotFoundExpection: not.a.driver
...

```
Having a META-INF/service/java.sql.Driver inside the JAR became mandatory with JDBC 4.0 in Java 6. Before that, some drivers included it and some didn't. Table below sums up the current state of affairs:

![JDBC 3.0 vs 4.0 drivers](img/jdbcDrivers.png)

# Obtaining a Statement
In order to run SQL, you need to tell a Statement about it. Getting a Statement from a Connection is easy:

```
Statement stmt = conn.createStatement();
```

As you will remember, Statement is one of the four core interfaces on the exam. It represents a SQL statement that you want to run using the Connection.
That's the simple signature. There's another one that you need to know before the exam:

```
Statement stat = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
```
This signature takes two paramenter. The first is the ResultSet type, and the other is the ResultSet concurrency mode. You have to know all of the choices for these parameters and the order in which they are specified. Let's look at the choices for these parameters.

## Choosing a ResultSet Type
Bydefault, a ResultSet is in TYPE_FORWARD_ONLY mode. This is what you need most of the time. You can go through the data once in the order in which it was retrieved. The two other types are TYPE_SCROLL_INSENSITIVE and TYPE_SCROLL_SENSITIVE. Both allow you go through the data in any order. You can go forward and backward. You can even go to a specific spot in the data. Think of these like scrolling in a browser. You can scroll up and down. You can go to a specific spot in the result.
The difference between this scroll types is what happens when data changes in the actual database while you are busy scrolling. With TYPE_SCROLL_INSENSITIVE, you have a static view of what the resultSet looked like when you did the query. If the data changed in the table, you will see it as it was when you did the query. With TYPE_SCROLL_SENSITIVE, you would see the latest data when scrolling through the ResultSet. 

Note that you have to know forward only and scroll insensitive in detail for the exam. For scroll sentive, you only have to know the name and that it isn't well supported. You don't need to read or write code with it.

We say "would" because most databases and database drivers don't actually support the TYPE_SCROLL_SENSITIVE mode. That's right. You have to learn something for the exam that you are almost guaranteed never to use in practice.

If the type you request isn't available, the driver can "helpfully" downgrade to one that is. This means that if you ask for TYPE_SCROLL_SENSITIVE, you will likely get a Statement that is TYPE_SCROLL_INSENSITIVE. Isn't that great? 

Table below sumps up what you need to know about the ResultSet types.

![ResultSet type options](img/resultSetTypes.png)


## Choosing a ResultSet Concurrency Mode
By default, a ResultSet is in CONCUR_READ_ONLY mode. This is what you need most of the time. It means that you can't update the result set. Most of the time, you will use INSERT, UPDATE or DELETE SQL statements to change the database rather than a ResultSet.

There is one other mode that you can request when creating a Statement. Unsurprisingly, it lets you modify the database through the ResultSet. It is called CONCUR_UPDATABLE.

Note that you have to know read only mode in detail for the exam. For updatable, you only have to know the name and that it is not universally supported.

Databases and JDBC drivers are not required to support CONCUR_UPDATABLE. Unlike TYPE_SCROLL_SENSITIVE, an updatable result set is at least used on rare occasions. Most of the time, it is the wrong choice for your program, though.
Again, if the mode you request isn't available, the driver can downgrade you. This means that if you ask for CONCUR_UPDATABLE, you will likely get a statement that is CONCUR_READ_ONLY.

The table below sums up what you need to know about the result set concurrency modes:

![Result Set concurrency mode options](img/resultSetConcurrencyModes.png)



# Executing a Statement

# Getting Data from a ResultSet
## Reading a ResultSet
## Getting Data for a Column
## Scrolling ResultSet

# Closing Database Resources

# Dealing with Exceptions
