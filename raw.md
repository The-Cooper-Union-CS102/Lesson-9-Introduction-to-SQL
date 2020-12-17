# Introduction to SQL

SQL stands for "Structures Query Language."  It is the language of databases.
A database is a piece of software which can store a large amount of data and
give users quick access to it in an organized way.

In this introduction to SQL, we will be using a particular database software
called SQLite, a relatively simple database implementation written in C.  We
will start with some simple examples on a fake database that already exists,
and work our way up in complexity.

This tutorial closely follows https://www.sqlitetutorial.net, kudos to them!

## Getting Started with SQLite

To use SQLite on macOS, you simply type `sqlite3` into the terminal.

It is also installable in the usual manner on cygwin.

There is a SQLite repl. [here](https://repl.it/languages/sqlite)

In the `samples` directory of this repo, there are some sample database files.
We will first look at `customers.csv` which contains some fake data on some
hypothetical customers of a business.

To load this file into a database, start SQLite in csv mode and import it like
this:

```
$ sqlite3 -csv
SQLite version 3.30.1 2019-10-10 20:19:45
Enter ".help" for usage hints.
.import customers.csv customers
.schema
CREATE TABLE customers(
  "CustomerId" TEXT,
  "FirstName" TEXT,
  "LastName" TEXT,
  "Company" TEXT,
  "Address" TEXT,
  "City" TEXT,
  "State" TEXT,
  "Country" TEXT,
  "PostalCode" TEXT,
  "Phone" TEXT,
  "Fax" TEXT,
  "Email" TEXT,
  "SupportRepId" TEXT
);
```

`.import customers.csv customers` imports the file `customers.csv` into a **table**
called `customers`.

`.schema` shows the structure of the table, which was determined automatically from
the file.

## SELECT Statement

The `SELECT` statement is your bread and butter for getting data out of the
database.  Let's look at a simple example in our `customers` table.

```
SELECT * FROM customers;
```

```
1,"Luís","Gonçalves","Embraer - Empresa Brasileira de Aeronáutica S.A.","Av. Brigadeiro Faria Lima, 2170","São José dos Campos",SP,Brazil,12227-000,"+55 (12) 3923-5555","+55 (12) 3923-5566",luisg@embraer.com.br,3
2,Leonie,"Köhler","","Theodor-Heuss-Straße 34",Stuttgart,"",Germany,70174,"+49 0711 2842222","",leonekohler@surfeu.de,5
3,"François",Tremblay,"","1498 rue Bélanger","Montréal",QC,Canada,"H2G 1A7","+1 (514) 721-4711","",ftremblay@gmail.com,3
4,"Bjørn",Hansen,"","Ullevålsveien 14",Oslo,"",Norway,0171,"+47 22 44 22 22","",bjorn.hansen@yahoo.no,4
5,"František","Wichterlová","JetBrains s.r.o.","Klanova 9/506",Prague,"","Czech Republic",14700,"+420 2 4172 5555","+420 2 4172 5555",frantisekw@jetbrains.com,4
...
```

The statement printed all of the information we had on every customer.

The `*` is similar to the unix `*` in that it means "get everything."  In
particular, it means get `CustomerId`, `FirstName`, `LastName`, `Company`, and
so on.  The `FROM customers` means get all this from the `customers` table.
This is our only table currently, but you could have more.

Note by the way, that SQL is generally case insensitive, meaning the following
query is just equivalent to our original one.

```
select * from customers;
```

Suppose we only wanted three customers from the table.  We could modify our statement
to look like this:

```
SELECT * FROM customers LIMIT 3;
```

```
1,"Luís","Gonçalves","Embraer - Empresa Brasileira de Aeronáutica S.A.","Av. Brigadeiro Faria Lima, 2170","São José dos Campos",SP,Brazil,12227-000,"+55 (12) 3923-5555","+55 (12) 3923-5566",luisg@embraer.com.br,3
2,Leonie,"Köhler","","Theodor-Heuss-Straße 34",Stuttgart,"",Germany,70174,"+49 0711 2842222","",leonekohler@surfeu.de,5
3,"François",Tremblay,"","1498 rue Bélanger","Montréal",QC,Canada,"H2G 1A7","+1 (514) 721-4711","",ftremblay@gmail.com,3
```

And now suppose we wanted only to select the `FirstName` and `LastName` of each of these
three.  We could modify our statement like this:

```
SELECT FirstName, LastName FROM customers LIMIT 3;
```

```
"Luís","Gonçalves"
Leonie,"Köhler"
"François",Tremblay
```

In general, it is good practice to avoid the use of `*` and only use it for
debugging.

We can modify the `SELECT` statement with a couple of clauses (just meaning
a phrase that follows the statement).  We will go over a few important ones now.

## ORDER BY Clause

The `ORDER BY` clause allows us to sort the rows, ordered by some column or
group of columns.  Here we create a query that order the customers by their first name.

```
SELECT FirstName, LastName FROM customers ORDER BY FirstName LIMIT 3;
```

```
Aaron,Mitchell
Alexandre,Rocha
Astrid,Gruber
```

Note that even though we limited the output to three rows, it is as if every
row was sorted and the first three results of that sort were returned.  By
default the sort happens to be ascending (increasing) but we could also set
it to descending (decreasing).

```
SELECT FirstName, LastName FROM customers ORDER BY FirstName DESC LIMIT 3;
```

```
Wyatt,Girard
Victor,Stevens
Tim,Goyer
```

We can also sort by both `FirstName` and `LastName` at the same time, meaning if there
is a tie in the `FirstName` column, the `LastName` will decide the winner.

```
SELECT Country, FirstName, LastName FROM customers ORDER BY Country DESC, FirstName LIMIT 3;
```

```
"United Kingdom",Emma,Jones
"United Kingdom",Phil,Hughes
"United Kingdom",Steve,Murray
```

# DISTINCT Clause

The `DISTINCT` clause allows us to remove duplicates, like a **set** in Python.

We could use this, for example, to get all of the unique countries of each customer.

```
SELECT DISTINCT Country FROM customers;
```

```
Brazil
Germany
Canada
Norway
"Czech Republic"
Austria
Belgium
Denmark
USA
Portugal
France
Finland
Hungary
Ireland
Italy
Netherlands
Poland
Spain
Sweden
"United Kingdom"
Australia
Argentina
Chile
India
```

We can use this to check duplicates among groups of columns as well.  For example
if we want a list of unique cites and states, we could do:

```
SELECT DISTINCT City, Country FROM customers;
```

```
"São José dos Campos",Brazil
Stuttgart,Germany
"Montréal",Canada
Oslo,Norway
Prague,"Czech Republic"
Vienne,Austria
Brussels,Belgium
Copenhagen,Denmark
...
```

## WHERE Clause

The `WHERE` clause is used for filtering your data based on some condition.  It
is similar to a conditional `continue` statement at the start of a loop, or the
`if` part of a Python list comprehension like 
`a = [x for x in range(10) if x < 4]`.

As an example, what if we wanted to get a list of all customers from the United Kingdom?

```
SELECT FirstName, LastName FROM customers WHERE Country = 'United Kingdom';
```

```
Emma,Jones
Phil,Hughes
Steve,Murray
```

Note that SQL breaks from the typical pattern of using `==` for equality.

There are a couple other operators we can use as well.  Some examples:

```
SELECT FirstName, LastName FROM customers WHERE Country in ('Germany', 'United Kingdom');
```

```
Leonie,"Köhler"
Hannah,Schneider
Fynn,Zimmermann
Niklas,"Schröder"
Emma,Jones
Phil,Hughes
Steve,Murray
```


```
SELECT CustomerId FROM customers WHERE CustomerId BETWEEN 0 AND 3;
```

```
1
2
3
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
```

Oh...  That's weird...  Because our schema detected the CustomerId as `TEXT`,
it is performing a string comparison.  We can fix this with `CONVERT`

```
SELECT CustomerId FROM customers WHERE CAST(CustomerId AS INT) BETWEEN 0 AND 3;
```

```
1
2
3
```

Note that however, in this case, this casting is inefficient, we will have to
cast convert every single row into an integer before getting our answer.  It
would be better to make sure the value is stored correctly in the first place.

Like in other languages, we can combine these conditions with logical operators:

```
SELECT FirstName FROM customers WHERE Country in ('Germany', 'Canada') OR
CAST(CustomerId AS INT) BETWEEN 0 AND 10;
```

```
"Luís"
Leonie
"François"
"Bjørn"
"František"
Helena
Astrid
Daan
Kara
Eduardo
Mark
Jennifer
Robert
Edward
Martha
Aaron
Ellie
Hannah
Fynn
Niklas
```

## Nesting Queries

We can use queries as values inside other queries.  For example, suppose
we wanted to get the name of every customer who is from the same country
as "Marc Dubois" or "Steve Murray".  We can write:

```
SELECT Country, FirstName, LastName FROM customers
WHERE Country in (
    SELECT Country from customers
    WHERE FirstName = 'Marc' AND LastName = 'Dubois' OR
          FirstName = 'Steve' AND LastName = 'Murray'
);
```

## Joining Tables

All of the examples we have done so far have been on only one table, but in
most cases you will be dealing with data across multiple tables.  This can be
done through a "join" operation which combines data from both tables into one.

We will take a look at a new example database containing one table for
albums and one table for artists.  We open them with the following commands:

```
$ sqlite3 -csv
.import artists.csv artists
.import albums.csv albums
```

Now lets take a look at some data

```
SELECT ArtistId FROM albums LIMIT 10;
```

```
1
2
2
1
3
4
5
6
7
8
```

```
SELECT ArtistId FROM artists LIMIT 10;
```

```
1
2
3
4
5
6
7
8
9
10
```

You can see each column has a row that serves as a connection, the `ArtistId`.
We can connect the data in each table with a **join** as follows:

```
SELECT albums.Title, artists.Name FROM
albums INNER JOIN artists on artists.ArtistId = albums.ArtistId
LIMIT 10;
```

```
"For Those About To Rock We Salute You",AC/DC
"Balls to the Wall",Accept
"Restless and Wild",Accept
"Let There Be Rock",AC/DC
"Big Ones",Aerosmith
"Jagged Little Pill","Alanis Morissette"
Facelift,"Alice In Chains"
"Warner 25 Anos","Antônio Carlos Jobim"
"Plays Metallica By Four Cellos",Apocalyptica
Audioslave,Audioslave
```

Essentially, `albums INNER JOIN artists` creates a new table that has
all of the rows of both tables combined.  the `on artist.ArtistId =
albums.ArtistId` part means that for each row, the `ArtistId` will
be the same.  Conceptually, you could think of the process like this:
Look at every possible pair of rows from table 1 and table 2.  If
the condition holds (`artist.ArtistId = albums.ArtistId`), create a new
row taking columns from each table.

**Question:**  How many rows will be created when we do this?

```
SELECT albums.Title, artists.Name FROM
albums INNER JOIN artists on 1 = 1
LIMIT 10;
```

**Question** Is this operation symmetric?

## Left Join

A left join is like an inner join, except it also gives you ALL rows from the
first table, regardless of if they match your condition.  That is, you get all
the rows from the "left" table, but you also get columns from the "right" table
that match the left table.  If there is a row in the "left" table which does
not match any row in the right table, the values from the "right" table will
be null in that row.  Let's look at an example.

```
SELECT albums.Title, artists.Name FROM
artists LEFT JOIN albums on albums.ArtistId = artists.ArtistId
ORDER BY albums.AlbumId
LIMIT 20 OFFSET 60;
```

(Random note, `OFFSET 60` just means go 60 items down in the list)

```
,"Pedro Luís E A Parede"
,"Los Hermanos"
,"Mundo Livre S/A"
,Otto
,Instituto
,"Nação Zumbi"
,"DJ Dolores & Orchestra Santa Massa"
,"Seu Jorge"
,"Sabotage E Instituto"
,"Stereo Maracana"
,"Academy of St. Martin in the Fields, Sir Neville Marriner & William Bennett"
"For Those About To Rock We Salute You",AC/DC
Audioslave,Audioslave
"Iron Maiden","Iron Maiden"
Killers,"Iron Maiden"
"Live After Death","Iron Maiden"
"Live At Donington 1992 (Disc 1)","Iron Maiden"
"Live At Donington 1992 (Disc 2)","Iron Maiden"
"No Prayer For The Dying","Iron Maiden"
"Piece Of Mind","Iron Maiden"
```

Note that in the first several rows, there is nothing before the comma `,`
indicating a null value.  This means there was an artist, say "Otto" with no
associated albums in the database.  In an inner join, this row would not
be shown at all.

Note that a 'right join' is just a left join with the order of the tables
switched.

## Cross Join

There is something called a cross join, but its really just an inner join
with an always-true condition, so we won't go over it.

## Self Join

There is also something called a self join, but that is really just an inner
join on a table with intself, so we don't go over that either.

## Full Outer Join

The full outer join is another fundamental type of join which combines the
result of a left join and a right join.  That is, the result will have every
row from both tables, will nulls for columns which do not correspond to the
other row.

Unfortunately, SQLite does not support this operation directly, beause it
is too lite.  But we can emulate it by combining a left join and a right
join.

```
SELECT artists.ArtistId, artists.Name, albums.AlbumId, albums.Title FROM
albums LEFT JOIN artists on albums.ArtistId = artists.ArtistId
UNION ALL
SELECT artists.ArtistId, artists.Name, albums.AlbumId, albums.Title FROM
artists LEFT JOIN albums on albums.ArtistId = artists.ArtistId
WHERE albums.ArtistId IS NULL
LIMIT 10;
```

Notice that we have to use `WHERE albums.ArtistId IS NULL` because otherwise
the rows available from both tables would be duplicated.

## GROUP BY Clause

The `GROUP BY` clause allows us to combine multiple rows into one result.
For example, suppose we wanted to count the number of albums one artist has.
We want to count the number of albums, but grouped by the artist ID.  In
SQL we write:

```
SELECT ArtistId, COUNT(AlbumId) FROM albums
GROUP BY ArtistId
ORDER BY COUNT(ArtistId) DESC
LIMIT 10;
```

```
90,21
22,14
58,11
50,10
150,10
114,6
118,5
84,4
82,4
21,4
```

This gives us the top ten artists by how many albums they have.  We can
combine this result with the artist table to get the names using a join:

```
SELECT Name, COUNT(AlbumId) FROM
albums LEFT JOIN artists on artists.ArtistId = albums.ArtistId
GROUP BY albums.ArtistId
ORDER BY COUNT(AlbumId) DESC
LIMIT 10;
```

```
"Iron Maiden",21
"Led Zeppelin",14
"Deep Purple",11
Metallica,10
U2,10
"Ozzy Osbourne",6
"Pearl Jam",5
"Foo Fighters",4
"Faith No More",4
"Various Artists",4
```

We can also filter groups on arbitrary conditions.  Say we only want
artists with 5 or more albums:

```
SELECT Name, COUNT(AlbumId) FROM
albums LEFT JOIN artists on artists.ArtistId = albums.ArtistId
GROUP BY albums.ArtistId HAVING count(AlbumId) >= 5
ORDER BY COUNT(AlbumId) DESC;
```

## INSERT Statement

So far we have covered a bunch of way for getting data out of a table, but that
does beg the question the, how can we insert data into the table?  These next
sections go over just that.

This is how we can simply insert data with the `INSERT` statement:

```
INSERT INTO artists (ArtistId, Name)
VALUES (999, 'Cory Nezin');
```

```
SELECT Name FROM artists WHERE ArtistId = 999;
```

```
"Cory Nezin"
```

You could also insert multiple rows at the same time:

```
INSERT INTO artists (ArtistId, Name)
VALUES
    (997, 'Cory Aezin'),
    (998, 'Cory Bezin'),
    (999, 'Cory Cezin');
```

```
SELECT ArtistId, Name FROM artists WHERE Name LIKE 'Cory%';
```

```
999,"Cory Nezin"
997,"Cory Aezin"
998,"Cory Bezin"
999,"Cory Cezin"
```

Uh oh, we accidentally added two artists with the same ID.  How can we fix it?

## UPDATE Statement

Option number two to fix our mistake is to update one of the rows with a new ID.
We can do that with the `UPDATE` statement like this:

```
UPDATE artists
SET ArtistId = 1000
WHERE Name = "Cory Nezin";
```

And observe that our issue is fixed:

```
SELECT ArtistId, Name FROM artists WHERE Name LIKE 'Cory%';
```

```
1000,"Cory Nezin"
997,"Cory Aezin"
998,"Cory Bezin"
999,"Cory Cezin"
```

## DELETE Statement

Option number two to fix our mistake is to delete the problem row.  We can do this
as follows:

```
DELETE FROM artists
WHERE Name = "Cory Nezin";
```

```
SELECT ArtistId, Name FROM artists WHERE Name LIKE 'Cory%';
```

```
997,"Cory Aezin"
998,"Cory Bezin"
999,"Cory Cezin"
```

## Table Definition

Those are the basics of interacting with an existing database, however we took
for granted the creation of the database and its structure.  This section
will go over some basics on defining a "schema" for a database.

### Types

First, we have a few basic types similar to other languages.  The types are
`NULL`, `INTEGER`, `REAL`, `TEXT` and `BLOB` which are similiar to Python's
`None`, `int`, `float`, `str` and `bytes`.  `NULL` represents a lack of a value,
`INTEGER` represents a signed integer value, `REAL` represnts a floating point
number, `TEXT` represents a small(ish) amount of text and `BLOB` represents
arbitrary binary data which could be used to store anything like an audio
recording, image, or video.

### Creating a Table

We can create a new, emtpy table with the following SQL statement.


```
CREATE TABLE artists (
    "ArtistId" INTEGER,
    "Name" TEXT
);
```

```
.schema
```

```
CREATE TABLE artists (
    "ArtistId" INTEGER,
    "Name" TEXT
);
```

Now we can add data in the same way we did before:

```
INSERT INTO artists (ArtistId, Name)
VALUES (0, 'Cory');
```

```
SELECT * FROM artists;
```

```
0|Cory
```

### Contraints

We could also create the table with a "primary key", meaning a column
which uniquely identifies its row, like this:

```
CREATE TABLE artists (
    "ArtistId" INTEGER PRIMARY KEY,
    "Name" TEXT
);
```

Now if we try to insert a duplicate:

```
INSERT INTO artists (ArtistId, Name)
VALUES (0, 'Cory');
INSERT INTO artists (ArtistId, Name)
VALUES (0, 'Nezin');
```

We will get an error

```
Error: UNIQUE constraint failed: artists.ArtistId
```

You can also add a `UNIQUE` constraint explicitly

```
CREATE TABLE artists (
    "ArtistId" INTEGER PRIMARY KEY,
    "Name" TEXT UNIQUE
);
```

```
INSERT INTO artists (ArtistId, Name)
VALUES (0, 'Cory');
INSERT INTO artists (ArtistId, Name)
VALUES (1, 'Cory');
```

```
Error: UNIQUE constraint failed: artists.Name
```

We can also enforce constraints across tables with the use of a **foreign key**.
For example let's say we now want to add an "albums" table which relies on the
"artist" table, and we want to impose the constraint that every album must
reference a valid artist.  We can do it like this:

```
CREATE TABLE artists (
    "ArtistId" INTEGER PRIMARY KEY,
    "Name" TEXT UNIQUE
);
```

```
CREATE TABLE albums (
    "AlbumId" INTEGER PRIMARY KEY,
    "ArtistId" INTEGER,
    "Name" TEXT,
    FOREIGN KEY (ArtistId) REFERENCES artists (ArtistId)
);
```

The `artists` table is called the parent table and `albums` is the child table.

To enable foreign key constraints we must run this command

```
PRAGMA foreign_keys = ON;
```

Now we populate our parent table:

```
INSERT INTO artists (ArtistId, Name)
VALUES (0, 'Cory');
```

And insert an item into the child table

```
INSERT INTO albums (AlbumId, ArtistId, Name)
VALUES (1, 1, "Plans");
```

```
Error: FOREIGN KEY constraint failed
```
