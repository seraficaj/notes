# Intro to SQL

## Introduction To SQL and Databases

### Objectives

* Describe the purpose of a database
* Connect to a PostgreSQL database using `psql`
* Use SQL to create a database, tables, and entries
* Use SQL CRUD operations to insert, select, update, and delete data
* Define the purpose of Primary and Foreign Keys

### What is a database?

* It is a program that enforces structure on your data and allows a computer to quickly retreive data.
* A database should support CRUD operations.
  * **CRUD:** Create, Read, Update, Destroy
* Sometimes called a DBMS \(Database Management System\)

### Why Use a Database?

Discuss as a class. Why is it better than just writing to files?

* Data is structured
* Can store [master data and transactional data](https://www.quora.com/What-is-the-difference-between-transactional-data-and-master-data)
* Data retrevial is fast
* Has a system for remote access \(data is often stored on a remote server\)
* Has a system for backup

### Types of Databases

#### RDBMS

\(Relational Database Management System\) The most common type of database today is a **relational database**. Relational databases have tabular data with rows for each instance of data and columns for each attribute of that data. Tables may refer to one another. Relational databases typically use **SQL** \(Structured Query Language\).

![diagram of example relational database](https://image.slidesharecdn.com/databasemanagementsystem-161020091729/95/database-management-system-9-638.jpg?cb=1476955074)

**Brands of Relational Databases**

* Postgres
* MySQL
* Oracle \(Commercial Product with lots of features\)
* Microsoft SQL Server
* SQLite \(Good for mobile development/Small applications\)

#### Cloud Storage

This is a very vague term and can be used to mean lots of things. Typically it is a system in which your data is stored and managed by a company so you don't have to worry about losing it. Examples included AWS \(Amazon Web Services\), Rackspace, MS Azure

#### NoSQL

There is also a school of thought called NoSql \(literally Not SQL\). Instead of data being stored in tables, it is often a Key Value storage system and is not relational. This is typically used in applications where a database needs to scale well. Example technologies include MongoDB, Apache CouchDB, SimpleDB.

### How are databases used in the wild?

For learning and testing purposes, we will be using Postgres on the same machine that our web server is running. In the real world, your database will be on a separate machine, called a **database server**.

A database server is a computer or group of computers that is dedicated to storing your data and handling remote requests to retreive that data. Even in a very simple configuration, the database server will have at least 1 backup machine that keeps an exact copy of the database just in case the main database server goes down.

## psql

Today we're using [PostgreSQL](https://www.postgresql.org/download/macosx/), often called Postgres. Postgres is based off an older database system called Ingres. That's where the name comes from. We're using a "post-Ingres" database. PostgreSQL is the default database on macOS Server as of OS X Server version 10.7.

If you install [Postgres.app](https://postgresapp.com/downloads.html), you will have access to psql from the elephant icon at the top of the screen:

* ![image](../.gitbook/assets/Postgres.png)

You can start and stop running the server from here.

In order to access our server from our terminal, we need to install the path in our `.zshrc` or `.bashrc`. Open up your file from terminal by typing anywhere `open ~/.zshrc` and add this line to the bottom:

```text
export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/latest/bin
```

To apply this, either restart your terminal or type `source ~/.zshrc` in your zshell. To check if it works,

**psql** is a command line tool to interact with postgres databases, by default it connects to the localhost database with the name of the current user.

* In your terminal, type `psql` to begin using psql.

psql has some of it's own commands.

* type `\?` to view them all.

Use `q` to exit the help screen \(or any other screen that doesn't self-terminate\)

Note that all psql commands start with `\` except for `q`.

To _quit_ psql and return to the home terminal:

```bash
\q
```

### SQL: Structured Query Language

**A Brief History of Databases**

Before the notion of an RDBMS \(like PostreSQL\) and a standard language for querying that data was created \(SQL\), there were many database vendors. Each vendor had different ways of storing data and very different ways of retreiving the data afterwards. Moving data from one system to another was very costly. Luckly, in the 1970s, SQL was created and later turned into a standard. Modern relational databases are now based on the SQL standard, so moving from Postgres to Oracle is not nearly as much of a challenge as it used to be.

**CRUD**

Stands for Create, Read, Update and Destroy. This is the lifecycle of data in an applicatoin. In SQL, CRUD can be mapped to the following **INSERT, SELECT, UPDATE, DELETE**. We will walk through examples in this section.

### Creating a Database

Most database products have the notion of separate databases. Lets create one for the lesson.

```bash
CREATE DATABASE testdb;
```

Remember that the default DBMS on a mac is PostgreSQL. Type `psql` to connect to PostgreSQL via the command line.

To view all the databases that exist on your machine, type `\l`. You should see `testdb` in this list.

Connect to the database: `\connect testdb`

Once we connect, our command prompt should look similar to this: `testdb=#`

To view the tables in the database you're connected to, type `\dt`. \(This stands for "display tables".\)

At this point we should have a database with no tables in it. So now we need to create tables - using SQL **\(NOT to be confused with the psql app itself\)**

**ALL SQL COMMANDS MUST BE ENDED WITH A SEMICOLON IN THE PSQL SHELL** It doesn't matter how many lines you take up to write the SQL statements because it won't run until you type a semi-colon.

Note that psql will not accept values with double quotes, only single quotes.

#### CREATE-ing a Table

This is an example of a students table. \(We will talk about the primary key soon.\)

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name TEXT,
    phone VARCHAR(15),
    email TEXT
);
```

Check that it's there:

```text
\dt
```

Look at the table structure

```text
 \d students
```

#### INSERT-ing Data

```sql
INSERT INTO students
(name, phone, email)
VALUES
('William Smith', '(415)555-5555', 'bill@example.com');

INSERT INTO students
(name, phone, email)
VALUES
('Bob Jones', '(415)555-5555', 'bob@example.com');
```

#### SELECT-ing Data

```sql
SELECT * FROM students;

SELECT * FROM students WHERE name = 'Bob Jones';

SELECT id, name FROM students;
```

### UPDATE-ing Data

```sql
UPDATE students SET email='bobby@example.com' WHERE name = 'Bob Jones';
```

### DELETE-ing Data

```sql
DELETE FROM students WHERE name = 'Mary';
->DELETE 0
```

```sql
DELETE FROM students WHERE email = 'bobby@example.com';
->DELETE 1
```

#### DROP-ing a Table

```sql
DROP TABLE students;
```

### Database Schema Design

The **schema** of the database is the set of CREATE TABLE commands that specify what the tables are and how they relate to each other. For our very simple database example, here is the schema:

We typed

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name TEXT,
    phone VARCHAR(15),
    email TEXT
);
```

```text
testdb=# \d students
                                   Table "public.students"
 Column |         Type          | Collation | Nullable |               Default                
--------+-----------------------+-----------+----------+--------------------------------------
 id     | integer               |           | not null | nextval('students_id_seq'::regclass)
 name   | text                  |           |          | 
 phone  | character varying(15) |           |          | 
 email  | text                  |           |          | 
Indexes:
    "students_pkey" PRIMARY KEY, btree (id)
```

### What is a Primary Key?

It denotes an attribute on a table that can uniquely identify the row.

#### What does SERIAL Do?

SERIAL tells the database to automatically assign the next unused integer value to id whenever we insert into the database and do not specify id. In general, if you have a column that is set to SERIAL, it is a good idea to let the database assign the value for you.

### Data Types

Similar to how Javascript has types of data, SQL defines types that can be stored in the DB. Here are some common ones:

* Serial
* Integer
* Numeric // Numbers are exact, no rounding error
* Float // Rounding error is possible, but operations are faster than Numeric
* Text, Char\(set number of characters\), Varchar\(max number of characters\)
* Timestamp
* Boolean \(True or False\)

### Exercise Time

Design a table for a movie database. Discuss a few things that a movie table may have. Choose the appropriate data type for the data. Make the `CREATE TABLE` command and execute it in psql. Use `\dt` to verify that the table was created. Once you're satisfied that the table is there, get rid of it using the DROP TABLE command. Use `\dt` again to make sure that the table has been dropped.

### Selecting

A select statement allows you to get data from the database. Here are the [docs on select](http://www.postgresql.org/docs/9.0/static/sql-select.html). Also, postgres has a good [tutorial on select](http://www.postgresql.org/docs/7.3/static/tutorial-select.html). I'd recommend looking at the tutorial sometime after the lesson.

Create a new database to hold a movies table:

```sql
CREATE DATABASE testdb;
```

Connect to the new database:

```sql
\connect testdb;
```

Given this table:

```sql
CREATE TABLE movies (
  id SERIAL PRIMARY KEY,
  title TEXT,
  description TEXT,
  rating INTEGER
);
```

And these insert statements:

```sql
INSERT INTO movies (title, description, rating) VALUES('Cars', 'a movie', 9);
INSERT INTO movies (title, description, rating) VALUES('Back to the Future', 'another movie', 9);
INSERT INTO movies (title, description, rating) VALUES('Dude Wheres My Car', 'probably a bad movie', 3);
INSERT INTO movies (title, description, rating) VALUES('Godfather', 'good movie', 9);
INSERT INTO movies (title, description, rating) VALUES('Mystic River', 'did not see it', 7);
INSERT INTO movies (title, description, rating) VALUES('Jurassic Park', 'dinos and Jeff Goldblum', 10)
INSERT INTO movies (title, description, rating) VALUES('Argo', 'a movie', 8);
INSERT INTO movies (title, description, rating) VALUES('Gigli', 'really bad movie', 1);
```

This will select all the attributes from the movies table unconditionally. Make sure not to forget the semi-colon at the end of each statement.

```sql
SELECT * FROM movies;
`
```

We may not want all attribues though. Let's say instead we only care about the titles of the movie and the description. Here is the query we'd use:

```sql
SELECT title, description FROM movies;
```

This will select the titles from movies table where the rating is greater than 4.

```sql
SELECT title FROM movies WHERE rating > 4;
```

You can also have more complex queries to get data. The following query finds all the movies with a rating greater than 4 and with a title of Cars.

```sql
SELECT title FROM movies WHERE rating > 4 AND title = 'Cars';
```

SQL also supports an OR statement. The following query will return any movie with a rating greater than 4, or any movies with the title Gigli. In other words, if a movie called Gigli is found with a rating equal to 1, it will still be returned along with any movie with a rating greater than 4.

```sql
SELECT title FROM movies WHERE rating > 4 OR title = 'Gigli';
```

Let's say that we just want a list of the best movies of all time. We can do a select statement that ensures ordering. The DESC keyword tells it to order the rating in descending order. ASC is the default.

```sql
SELECT title, rating FROM movies ORDER BY rating DESC;
```

**Note:** If no order by clause is specified, the database does not give any guarantees on what order your data will be returned in. At times it may seem like data you are getting back is in sorted order, but make sure not to rely on that in your code. Only rely on a sort if you also have an ORDER BY clause.

We've gotten a list of movies back, but it's way too long for our uses. Let's instead only get the top 5 movies that are returned using LIMIT:

```sql
SELECT title, rating FROM movies ORDER BY rating DESC LIMIT 5;
```

#### Exercise

Write a query on the movie table to return the worst movie of all time. There should be only 1 result returned. The result should include the title, description and rating of the movie.

### Updating

The update statement is defined [here](http://www.postgresql.org/docs/9.1/static/sql-update.html) in the postgres docs. It is used to change existing data in our database.

For example, if we do not think Gigli was actually that bad, and we want to change the rating to a 2, we can use an update statement:

```sql
UPDATE movies SET rating=2 WHERE title='Gigli';
```

### Deleting

Deleting works similarly to a select statement. Here are the [docs on delete](http://www.postgresql.org/docs/8.1/static/sql-delete.html)

The statement below deletes the Dude Wheres My Car row from the database:

```sql
DELETE FROM movies WHERE title='Dude Wheres My Car';
```

We could also use compound statements here:

```sql
DELETE FROM movies WHERE id < 9 AND rating = 2;
```

### Foreign Keys

This is where the **relational** part comes in! Foreign keys allow entries in one table to refer to entires in another table.

What are some examples of when this would be useful?

* \(library\) books table references an authors table
* \(elementary school\) a students table refereces a classes table, which references teachers table, which references a schools table, which references a districts table, etc.

Let's build out the books and authors tables listed above:

```text
CREATE TABLE authors (
  id SERIAL PRIMARY KEY,
  first_name TEXT,
  last_name INT
);

CREATE TABLE books (
  id SERIAL PRIMARY KEY,
  title TEXT,
  author_id INT references authors(id)
);

INSERT INTO authors (first_name, last_name) VALUES ('Alexandre', 'Dumas');
INSERT INTO books (title, author_id) VALUES ('The Three Musketeers', 1);
```

Use select statements to view the tables and make sure everything worked as expected. Now try to delete the Hobbit book - what happened? If you delete the associated author, can you delete the book now?

Now practice planning out a more complex scenario! Use your own ideas, or try the following:

* customers \(id, name, email\)
* items
* merch\_order \(id, num\_items, customer\_id\)
* ordered\_items \(id, item\_id, quantity, merch\_order\)

### ER Diagrams

Creating an ER diagram can be useful if you are designing a DB with lots of tables and relationships to one another. It may be useful to revist ER Diagrams after you have a firm understanding of databases. Here are some useful resources:

* [Wikipedia - ER Diagram](http://en.wikipedia.org/wiki/Entity-relationship_model)
* [Ultimate Guide To ER Diagrams](http://creately.com/blog/diagrams/er-diagrams-tutorial/) - Not so ultimate, but a good intro.

### LAB:

[Where in the world is Carmen San Diego?](https://github.com/WDI-SEA/sql-carmen-san-diego)

