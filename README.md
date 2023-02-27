#Replicate a MySQL Database in ClickHouse

##What Is ClickHouse?

ClickHouse is a column-oriented SQL database management system (DBMS) for online analytical processing (OLAP). It is available as both an open-source software and a cloud offering.

##Column-Oriented vs Row-Oriented Databases​
In a row-oriented DBMS, data is stored in rows, with all the values related to a row physically stored next to each other.
In a column-oriented DBMS, data is stored in columns, with values from the same columns stored together.

#1. Configure MySQL

###Configure the MySQL database to allow for replication and native authentication. ClickHouse only works with native password authentication. Add the following entries to /etc/my.cnf:
```default-authentication-plugin = mysql_native_password
gtid-mode = ON
enforce-gtid-consistency = ON
```
###Create a user to connect from ClickHouse:
```CREATE USER clickhouse_user IDENTIFIED With 'mysql_native_password' BY 'ClickHouse_123';
```
###Grant the needed permissions to the new user. For demonstration purposes, full admin rights have been granted here:
```GRANT ALL PRIVILEGES ON *.* TO 'clickhouse_user'@'%';

###Create a database in MySQL:
```CREATE DATABASE db1;
```

###Create a table:
```CREATE TABLE db1.table_1 (
    id INT,
    column1 VARCHAR(10),
    PRIMARY KEY (`id`)
) ENGINE = InnoDB;
```
###Insert a few sample rows:

```INSERT INTO db1.table_1
  (id, column1)
VALUES
  (1, 'abc'),
  (2, 'def'),
  (3, 'ghi');
  ```
 #2. Configure ClickHouse
 ##Set parameter to allow use of experimental feature
```Set parameter to allow use of experimental feature
```
##Create a database that uses the MaterializedMySQL database engine:
 ``` CREATE DATABASE db1_mysql
ENGINE = MaterializedMySQL(
  'mysql-host.domain.com:3306',
  'db1',
  'clickhouse_user',
  'ClickHouse_123'
) SETTINGS (allows_query_when_mysql_lost =true);
```

#The minimum parameters are:
#Parameter	Description	example
host:port	hostname or IP and port	mysql-host.domain.com
database	mysql database name	db1
user	username to connect to mysql	clickhouse_user
password	password to connect to mysql	ClickHouse_123

#The MaterializedMySQL database engine allows you to define a database in ClickHouse that contains all the existing tables in a MySQL database, along with all the data in those tables. On the MySQL side, DDL and DML operations can continue to made and ClickHouse detects the changes and acts as a replica to MySQL database.
##3.Test the Integration
#In MySQL, insert a sample row:
``` INSERT INTO db1.table_1
  (id, column1)
VALUES
  (4, 'jkl');
  ```
 #Notice the new row appears in the ClickHouse table:
 ```SELECT
    id,
    column1
FROM db1_mysql.table_1```
#Suppose the table in MySQL is modified. Let's a column to db1.table_1 in MySQL:
```alter table db1.table_1 add column column2 varchar(10) after column1;
```
#Now let's insert a row to the modified table:
```INSERT INTO db1.table_1
  (id, column1, column2)
VALUES
  (5, 'mno', 'pqr');
  ```
  #Notice the talbe in ClickHouse now has the new column and the new row:
  ```
  SELECT
    id,
    column1,
    column2
FROM db1_mysql.table_1```
