---
layout: single
permalink: /fSQL/docs/1.3/tutorial
title: fSQL Tutorial
sidebar:
    nav: "fsql_side_bar"
---

This tutorial is designed to give a brief overview of the PHP fSQL API since version 1.2.
The syntax and function of the SQL queries understood by fSQL will be
addressed in another tutorial.

## I. The Basics

Every script that wishes to load fSQL needs to only include one file, the fSQL.php
located in the distribution.  If fSQL.php is in the same directory as the script using it,
this is all you need:

```php
require_once("Environment.php");
```

Once the file is included, there is only one class you need for all of your SQL needs:
fSQLEnvironment.  This contains information on the different databases in the program
and their location on the file system.  fSQLEnvironment is a simple API that is very similar to the PHP mysql
API in almost every way.  For example, fSQLEnvironment's fetch_assoc() is modeled directly
after mysql_fetch_assoc() so if you're having trouble understanding fSQL's documentation,
the PHP mysql API could also help you understand what each function does.

To create a new fSQLEnvironment class, it has a simple constructor with no parameters:

```php
$fsql = new fSQLEnvironment();
```

## II. Defining Databases

Databases in fSQL are directories on the file system with an associated name
given to them.  To define one, call the following:

```php
$fsql->define_db("mydb", "/path/to/db");
```

<p>The first parameter is the database name and the second is the path to that database.
This tells the environment that the database to be called "mydb" will be
store its files in the directory /path/to/db on the file system.  In other words,
all table information and data for "mydb" should be loaded from and stored to
the directory /path/to/db.  If the supplied path does not exist, fSQL will attempt
to create it and set the appropriate permissions.

As of PHP fSQL v1.2, fSQL allows you have multiple databases defined for
fSQL.  For example, one could define several databases like so:

```php
$fsql->define_db("db1", "/path/to/db1");
$fsql->define_db("db2", "/path/to/db2");
$fsql->define_db("db3", "/path/to/db3");
```

To select which database is to be the default when using queries and other function
calls, use the select_db() function.

```php
$fsql->select_db("db2");
```

This function is the equivalent to MySQL's "USE `db2`" query.  You should always
select a default database before using any other functions in fSQLEnvironment.

## III. Data Definition and Manipulation

rom here on out, the most important method for dealing with the databases'
data is the query() method.  The query method takes one parameter and that is the string
fSQL query to execute.  The simplest form is data definition and manipulation
performs just the query and returns either a true value on sucess or a false
value on failure.</p>

```php
$fsql->query("CREATE TABLE example(
   id INT NOT NULL AUTO_INCREMENT,
   name VARCHAR(30), 
   age INT,
   PRIMARY KEY(id)
)");
```

The other types of queries worth mentioning: data manipulation (like INSERT, UPDATE, etc).
On these queries, the method affected_rows() returns the number of rows added or modified
by the last data manipulation query.  For example:</p>

```php
$fsql->query("DELETE FROM example WHERE id &lt; 5");
echo "Deleted Rows: ".$fsql->affected_rows();
```

## IV. Data Selection

Executing data retrieval queries like SELECT are also performed using the query() method.
Except on data retrieval queries, the query method returns a handle to a result set
which can be iterated through row by row using the fetch methods.  Below we see an example.

```php
$results = $fsql->query("SELECT id, name FROM example WHERE age > 30");
while($row = $fsql->fetch_array($results))
{
	echo $row['id']." ".$row['name']."\r\n";
}
$fsql->free_result($results);
```

<p>fetch_array($results) returns the next row in the result set until the last row
has passed and then it returns NULL to stop the loop.<br /><br />
There are several ways to iterate through a result set:</p>
1. Return the row as an associative array using the names/aliases of the columns as the row's keys:<br/>
    * $fsql->fetch_array($results)
    * $fsql->fetch_array($results, FSQL_ASSOC)
    * $fsql->fetch_assoc($results)
2. Returns the row as a normal array with integer indexes:
    * $fsql->fetch_row($results)
    * $fsql->fetch_array($results, FSQL_NUM)
3. Returns the row with both column name keys and integer keys:
    * $fsql->fetch_both($results)
    * $fsql->fetch_array($results, FSQL_BOTH)
4. Returns the row as "class-less object" using the names/aliases of the columns as the object's member variables:
    * $fsql->fetch_object($results)

Other result set methods of interest:
* num_rows($results) - Returns the number of rows in the result set
* num_fields($results) - Returns the number of columns in the result set
* data_seek($results, $n) - Sets the internal cursor of the result set so that the next fetch method returns the nth row of the result set.
* free_result($results) - Frees the result set and any memory it used
