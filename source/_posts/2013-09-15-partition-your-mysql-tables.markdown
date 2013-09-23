---
layout: post
title: "Partition Your (MySQL) Tables"
date: 2013-09-15 19:40
comments: true
categories: [mysql]
published: false 
---

Many years ago, you built your web site and launched it to the world. Time went by, traffic went up but performance went down. That's when you realized your forgot to put that index on the foreign key in that database column. Quick bit of ALTER TABLE and Voila! Performance is up.

Jump forward to the present day. Your little website is now more popular than ever. The dataset in your database has grown to rather substantial levels. You have tables with millions of records and despite all the SQL tuning and indexing the queries are getting slower and slower. Fortunately for you there may be another optimization step you can take before looking at sharding your database onto multiple servers (and paying for all that hardware!).

[MySQL Partitioning](http://dev.mysql.com/doc/refman/5.6/en/partitioning.html) is a relatively simple idea that can greatly improve the performance of you growing database. You need MySQL verion 5.1 or greater (you are right?) and need to have the partitioning plugin enabled (which most distributions have on by default) in order to take advantage of it. The [MySQL documentaion](http://dev.mysql.com/doc/refman/5.6/en/partitioning.html) can explain how to determine if you have partitioning in your installation.

### What is partitioning?

{% blockquote MySQL Documentation http://dev.mysql.com/doc/refman/5.5/en/partitioning-overview.html %}
Partitioning takes this notion a step further, by enabling you to distribute portions of individual tables across a file system according to rules which you can set largely as needed. In effect, different portions of a table are stored as separate tables in different locations. The user-selected rule by which the division of data is accomplished is known as a partitioning function, which in MySQL can be the modulus, simple matching against a set of ranges or value lists, an internal hashing function, or a linear hashing function. The function is selected according to the partitioning type specified by the user, and takes as its parameter the value of a user-supplied expression. This expression can be a column value, a function acting on one or more column values, or a set of one or more column values, depending on the type of partitioning that is used.
{% endblockquote %}

In laymans terms, this says that the user can define a function in their table definition that splits the data up into, you guessed it, partitions. Each partition will carry a subset of the table data that matches the value returned by the defined function. For example, when using [HASH partitioning](http://dev.mysql.com/doc/refman/5.1/en/partitioning-hash.html) you can set your function to be `MONTH(some_date_column)`. When MySQL inserts (or updates) a row into the table it will execute this function and the result will determine into which partition the row will be placed.

```sql
-- Create Orders Table
CREATE TABLE orders (
  id INT NOT NULL,
  name VARCHAR(50),
  ...
  ordered_date DATE NOT NULL DEFAULT '2000-01-01'
)
PARTITION BY HASH( MONTH(ordered_date) )
PARTITIONS 12;

-- INSERT a row
-- MySQL evaluates MONTH('2013-05-13') to '5'
-- Row is inserted into Partition 'p5' (partition names are prefixed with 'p')
INSERT INTO orders (name, ordered_date) VALUES ('Joe Smith','2013-05-13');
```

### That's kind of neat, but how does it help me with performance?  

Once your table is partitioned you have essentially allowed the database engine to optimize your queries even more. When you run a `SELECT` statement with a condition based on `ordered_date` MySQL will evaluate those dates using the same `MONTH` function, determine which partition to use and only search that partition. Depending on your table size and number of partitions (12 in our example) that could cut the dataset needed to be searched significantly. 


```sql
SELECT COUNT(*) FROM orders;
-- Result: 5,000,000

SELECT COUNT(*) FROM orders WHERE ordered_date BETWEEN '2012-05-01' AND '2012-05-31';
-- Result: 350,000 (equals COUNT(*) on only one partition, 'p5')

-- Perform select on an unindexed column in orders
SELECT 
  COUNT(*) 
FROM 
  orders 
WHERE 
  ordered_date BETWEEN '2012-05-01' AND '2012-05-31' AND 
  name='Joe Smith' -- not indexed
;

-- MySQL optimizes the query and identified the ordered_date condition
-- it evaluates the dates using MONTH and get the result 5
-- The query is further optimized to use only partition 'p5'
-- therefore, only 350,000 rows are searched instead of 5 million
```

This is only one (contrived) example of one of the MySQL partitioning 
types, HASH. The MySQL documentaion has much more detail on the other 
[types of partitioning](http://dev.mysql.com/doc/refman/5.5/en/partitioning-types.html) 
that are available. Make sure you look over each one carefully as there
can be varying system requirements for each (some are only available in
newer versions of MySQL).

After reading all that you are probably thinking that you wouldn't see 
much of a difference in the above example with a properly indexed table 
and you would probably  be right. However, here are other advantages to 
having partitioned tables.

### Archiving

Ever tried to do a archive or purge old data from a large table? Waited
until the middle of the night just to run the `DELETE` command because 
you know its going to lock up the table for 20 minutes and take down the
system while those rows are deleted?

Partitioning can save you from that nightmare too. As of MySQL 5.5 you
can execute `TRUNCATE` on a single partition. 

{% blockquote MySQL Documentation http://dev.mysql.com/doc/refman/5.5/en/partitioning-maintenance.html %}
Beginning with MySQL 5.5.0, you can also truncate partitions using ALTER TABLE ... TRUNCATE PARTITION. This statement can be used to delete all rows from one or more partitions in much the same way that TRUNCATE TABLE deletes all rows from a table.
{% endblockquote %}

This allows you to _instantly_ (or very nearly) erase a partition no
matter how large it is withoug impacting your system. For example, if
you have a large logging table of which you only need to keep the last 3
months of data, you can easily `TRUNCATE` all the partitions older than
3 months and shrink your dataset down.
