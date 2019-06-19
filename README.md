# Presto demo with MS SQL Server, Hive on S3 (with Minio), and Kafka

## usage
1. Run with docker-compose
2. Create required buckets in minio (UI in port 9000)
3. Use presto in port 8080 (examples below)

## table join demo

* make sure bucket exists first!

```
presto:default> create schema hive.products_lab with (location='s3://products-bucket/');
CREATE SCHEMA

presto:web> create table hive.products_lab.categories(id integer, name varchar);
presto:web> insert into hive.products_lab.categories (id,name) values (1,'Water'), (2,'Earth'), (3, 'Air');
```

*  of course you can reference only the table name if you are in a schema context...

```
presto:web> use memory.default;
presto:default> create table products_categories (id_product integer,id_category integer);

presto:default> insert into memory.default.products_categories
             -> (id_product,id_category)
             -> values
             -> (1,2),
             -> (2,2),
             -> (3,2),
             -> (4,2),
             -> (5,2),
             -> (6,1),
             -> (7,3),
             -> (8,2),
             -> (9,2);
INSERT: 9 rows

Query 20190619_141227_00017_w79iz, FINISHED, 1 node
Splits: 35 total, 35 done (100.00%)
0:02 [0 rows, 0B] [0 rows/s, 0B/s]
```

* show join between memory, hive e sqlserver

> select p.productname as product, c.name as category
from memory.default.products_categories pc
left join hive.products_lab.categories c on c.id = pc.id_category
left join mssql.dbo.products p on p.id = pc.id_product;

```
presto:default> select p.productname as product, c.name as category
             -> from memory.default.products_categories pc
             -> left join hive.products_lab.categories c on c.id = pc.id_category
             -> left join mssql.dbo.products p on p.id = pc.id_product;
  product   | category
------------+----------
 Car        | Earth
 Truck      | Earth
 Motorcycle | Earth
 Bicycle    | Earth
 Horse      | Earth
 Boat       | Water
 Plane      | Air
 Scooter    | Earth
 Skateboard | Earth
(9 rows)

Query 20190619_142519_00036_w79iz, FINISHED, 1 node
Splits: 116 total, 116 done (100.00%)
0:04 [21 rows, 446B] [4 rows/s, 102B/s]
```

* creates an orc table in hive

> create table hive.products_lab.products_with_categories as
select p.productname as product, c.name as category
from memory.default.products_categories pc
left join hive.products_lab.categories c on c.id = pc.id_category
left join mssql.dbo.products p on p.id = pc.id_product;

```
presto:default> create table hive.products_lab.products_with_categories as
             -> select p.productname as product, c.name as category
             -> from memory.default.products_categories pc
             -> left join hive.products_lab.categories c on c.id = pc.id_category
             -> left join mssql.dbo.products p on p.id = pc.id_product;
CREATE TABLE: 9 rows

Query 20190619_142623_00040_w79iz, FINISHED, 1 node
Splits: 134 total, 134 done (100.00%)
0:08 [21 rows, 446B] [2 rows/s, 55B/s]
```

* look into minio and see files there....

## kafka demo

* Warning: Presto is not a streaming analytics solution. You should look into Flink et al...

```
    presto> select count(*), quoteauthor from kafka.default.test group by quoteauthor order by count(*) desc limit 10;
     _col0 |   quoteauthor
    -------+-----------------
        38 |
        23 | Buddha
        22 | Confucius
        17 | Byron Pulsifer
        15 | Richard Bach
        14 | Lao Tzu
        13 | Albert Einstein
        10 | Wayne Dyer
        10 | Ralph Emerson
         9 | Dalai Lama
    (10 rows)

Query 20190619_162400_00012_6thj9, FINISHED, 1 node
Splits: 50 total, 50 done (100.00%)
0:02 [656 rows, 80.2KB] [307 rows/s, 37.6KB/s]

presto>  select count(*), quoteauthor from kafka.default.test group by quoteauthor order by count(*) desc limit 10;
 _col0 |   quoteauthor
-------+-----------------
    54 |
    29 | Buddha
    28 | Confucius
    21 | Byron Pulsifer
    19 | Albert Einstein
    16 | Richard Bach
    16 | Lao Tzu
    15 | Dalai Lama
    13 | Wayne Dyer
    13 | Napoleon Hill
(10 rows)

Query 20190619_162702_00013_6thj9, FINISHED, 1 node
Splits: 50 total, 50 done (100.00%)
0:02 [838 rows, 103KB] [416 rows/s, 51KB/s]
```
