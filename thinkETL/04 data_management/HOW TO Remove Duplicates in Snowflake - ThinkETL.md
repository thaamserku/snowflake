---
created: 2023-09-07T21:04:50 (UTC -04:00)
tags: []
source: https://thinketl.com/how-to-remove-duplicates-in-snowflakes/
author: ThinkETL
---

# HOW TO: Remove Duplicates in Snowflake?

> [!Excerpt]
> Learn different ways to remove duplicate records from a Snowflake table using SWAP WITH and INSERT OVERWRITE commands.

---
## **Introduction**

There are several ways in which we can remove duplicates from a Snowflake table. Consider below sample Employee data for the demonstration purpose.

```sql
CREATE OR REPLACE TABLE EMPLOYEE (
  EMPLOYEE_ID NUMBER(6,0),
  NAME VARCHAR2(20),
  SALARY NUMBER(8,2)
);

INSERT INTO EMPLOYEE(EMPLOYEE_ID,NAME,SALARY) VALUES
(100,'Jennifer',4400),
(100,'Jennifer',4400),
(101,'Michael',13000),
(101,'Michael',13000),
(101,'Michael',13000),
(102,'Pat',6000),
(102,'Pat',6000),
(103,'Den',11000)
;

SELECT * FROM EMPLOYEE;
```

**RESULT :**

<table><tbody><tr><td><strong>EMPLOYEE_ID</strong></td><td><strong>NAME</strong></td><td><strong>SALARY</strong></td></tr><tr><td>100</td><td>Jennifer</td><td>4400</td></tr><tr><td>100</td><td>Jennifer</td><td>4400</td></tr><tr><td>101</td><td>Michael</td><td>13000</td></tr><tr><td>101</td><td>Michael</td><td>13000</td></tr><tr><td>101</td><td>Michael</td><td>13000</td></tr><tr><td>102</td><td>Pat</td><td>6000</td></tr><tr><td>102</td><td>Pat</td><td>6000</td></tr><tr><td>103</td><td>Den</td><td>11000</td></tr></tbody></table>

Let us first understand different ways of extracting unique records from a table.

The following methods can be used to extract unique records from a Snowflake table.

-   Using DISTINCT Keyword
-   Using GROUP BY Clause
-   Using ROW\_NUMBER Analytic function

### **Using DISTINCT Keyword**

The **DISTINCT** keyword is used in conjunction with **SELECT** is used to return only distinct (unique) values from a dataset.

The below SQL query returns unique records from the EMPLOYEE table using DISTINCT keyword.

```sql
SELECT DISTINCT * FROM EMPLOYEE;
```

**RESULT :**

| EMPLOYEE_ID | NAME     | SALARY |
| ----------- | -------- | ------ |
| 100         | Jennifer | 4400   |
| 101         | Michael  | 13000  |
| 102         | Pat      | 6000   |
| 103         | Den      | 11000  |

### **Using GROUP BY Clause**

The **GROUP BY** clause is used with **SELECT** statement to collect data from multiple records and group the results by one or more columns. By applying GROUP BY function on all the source columns, unique records can be extracted from dataset.

The below SQL query returns unique records from the EMPLOYEE table using GROUP BY clause.

```sql
SELECT
  EMPLOYEE_ID,
  NAME,
  SALARY
FROM EMPLOYEE
GROUP BY EMPLOYEE_ID, NAME, SALARY;
```

**RESULT :**

| EMPLOYEE_ID | NAME     | SALARY |
| ----------- | -------- | ------ |
| 100         | Jennifer | 4400   |
| 101         | Michael  | 13000  |
| 102         | Pat      | 6000   |
| 103         | Den      | 11000  |
### **Using ROW\_NUMBER Analytic function**

The **ROW\_NUMBER** analytic function is used to assign sequential numbering to the rows within a window partition of the result set.

The below SQL query assigns row numbers to each unique set of records using ROW\_NUMBER analytic function.

```sql
SELECT
  EMPLOYEE_ID,
  NAME,
  SALARY,
  ROW_NUMBER() OVER(PARTITION BY EMPLOYEE_ID, NAME, SALARY ORDER BY EMPLOYEE_ID) AS ROW_NUMBER
FROM EMPLOYEE;
```

**RESULT :**

| EMPLOYEE_ID | NAME     | SALARY | ROW_NUMBER |
| ----------- | -------- | ------ | ---------- |
| 100         | Jennifer | 4400   | 1          |
| 100         | Jennifer | 4400   | 2          |
| 101         | Michael  | 13000  | 1          |
| 101         | Michael  | 13000  | 2          |
| 101         | Michael  | 13000  | 3          |
| 102         | Pat      | 6000   | 1          |
| 102         | Pat      | 6000   | 2          |
| 103         | Den      | 11000  | 1          |


> Once row numbers are assigned, the unique records from the table can be extracted by querying the rows with row number 1.

The below SQL query extracts unique records from EMPLOYEE table using ROW\_NUMBER analytic function.

```sql
SELECT EMPLOYEE_ID, NAME, SALARY
FROM(
  SELECT
    EMPLOYEE_ID,
    NAME,
    SALARY,
    ROW_NUMBER() OVER(PARTITION BY EMPLOYEE_ID, NAME, SALARY ORDER BY EMPLOYEE_ID) AS ROW_NUMBER
  FROM EMPLOYEE)
WHERE ROW_NUMBER = 1;
```

**RESULT :**

| EMPLOYEE_ID | NAME     | SALARY |
| ----------- | -------- | ------ |
| 101         | Michael  | 13000  |
| 100         | Jennifer | 4400   |
| 102         | Pat      | 6000   |
| 103         | Den      | 11000  | 

## **Removing Duplicate records from a Snowflake table**

The following methods can be used to remove exact row duplicates from a Snowflake table.

-   Using SWAP WITH command
-   Using OVERWRITE command

### **Using SWAP WITH command**

The **SWAP WITH** command swaps all content and metadata between two specified tables, including any integrity constraints defined for the tables. The two tables are essentially renamed in a single transaction.

Removing duplicate using this method involves four steps.

1.  Create a new table with the structure of source table without any data.
2.  Insert unique records from the source table into the newly created table.
3.  Swap the data between two tables.
4.  Delete the table created in first step.

**STEP-1:**

The Snowflake **CREATE TABLE LIKE** statement creates a new table with just the structure of the existing table without copying the data with exact same column names, data types, default values and constraints.

The below SQL statement creates EMPLOYEE\_DUP table with same structure as the EMPLOYEE table without any data.

```sql
CREATE OR REPLACE TABLE EMPLOYEE_DUP LIKE EMPLOYEE;
```

**STEP-2:**

The below SQL statement inserts unique records into the intermediate table created in earlier step.

```sql
INSERT INTO EMPLOYEE_DUP SELECT DISTINCT * FROM EMPLOYEE;
```

> _Note that you could use any of the three methods discussed in the earlier section in the select clause to insert unique records into the intermediate table._

**STEP-3:**

The below SQL statement swaps the data between two tables EMPLOYEE and EMPLOYEE\_DUP.

```sql
ALTER TABLE EMPLOYEE_DUP SWAP WITH EMPLOYEE;
```

This results in the actual source table EMPLOYEE having only unique records.

**STEP-4:**

The below statement drops the intermediate table used which now contains the entire data from source table.

### **Insert using OVERWRITE command**

The **INSERT** statement with **OVERWRITE** command deletes the existing records in the table before inserting the data into the table.

The below SQL statement overrides EMPLOYEE data with unique records truncating existing data with duplicates.

```sql
INSERT OVERWRITE INTO EMPLOYEE SELECT DISTINCT * FROM EMPLOYEE;
```

This method avoids steps of creating an intermediate table to hold the unique records from the table and swapping the data between them. All these actions are handled in a single SQL statement using OVERWRITE command.

> _Note that you could use any of the three methods discussed in the earlier section in the select clause to override data in the table with unique records._

## **Removing Duplicate records based on a Key field from a Snowflake table**

In most data warehouse tables, a surrogate key column is used. The surrogate keys are simple, system generated, incremental unique values. This column is used as an identifier for each row rather than relying on pre-existing attributes.

Let us make a small change to the EMPLOYEE table used as an example for demonstration in earlier sections by adding a surrogate key column EMPLOYEE\_KEY to understand working with tables containing surrogate keys.

```sql
CREATE OR REPLACE TABLE EMPLOYEE (
  EMPLOYEE_KEY NUMBER(6,0),
  EMPLOYEE_ID NUMBER(6,0),
  NAME VARCHAR2(20),
  SALARY NUMBER(8,2)
);

INSERT INTO EMPLOYEE(EMPLOYEE_KEY,EMPLOYEE_ID,NAME,SALARY) VALUES
(1,100,'Jennifer',4400),
(2,100,'Jennifer',4400),
(3,101,'Michael',13000),
(4,101,'Michael',13000),
(5,101,'Michael',13000),
(6,102,'Pat',6000),
(7,102,'Pat',6000),
(8,103,'Den',11000)
;

SELECT * FROM EMPLOYEE;
```

**RESULT :**

| EMPLOYEE_KEY | EMPLOYEE_ID | NAME     | SALARY |
| ------------ | ----------- | -------- | ------ |
| 1            | 100         | Jennifer | 4400   |
| 2            | 100         | Jennifer | 4400   |
| 3            | 101         | Michael  | 13000  |
| 4            | 101         | Michael  | 13000  |
| 5            | 101         | Michael  | 13000  |
| 6            | 102         | Pat      | 6000   |
| 7            | 102         | Pat      | 6000   |
| 8            | 103         | Den      | 11000  | 

> _Note that if you use any of the three methods discussed in the initial section of the article as is to extract unique records, it will **not** remove duplicates as the EMPLOYEE\_KEY is unique for each record._

**If we are considering the record with highest EMPLOYEE\_KEY value as the latest record out of records with same EMPLOYEE\_ID, then we can make use of ROW\_NUMBER analytic function to assign row numbers to records with same EMPLOYEE\_ID.**

The below SQL query assigns row numbers to the records in EMPLOYEE table partitioned by EMPLOYEE\_ID starting with highest EMPLOYEE\_KEY value.

```sql
SELECT
    EMPLOYEE_KEY,
    EMPLOYEE_ID,
    NAME,
    SALARY,
    ROW_NUMBER() OVER(PARTITION BY EMPLOYEE_ID ORDER BY EMPLOYEE_KEY DESC) AS ROW_NUMBER
FROM EMPLOYEE
;
```

**RESULT :**

| EMPLOYEE_KEY | EMPLOYEE_ID | NAME     | SALARY | ROW_NUMBER |
| ------------ | ----------- | -------- | ------ | ---------- |
| 2            | 100         | Jennifer | 4400   | 1          |
| 1            | 100         | Jennifer | 4400   | 2          |
| 5            | 101         | Michael  | 13000  | 1          |
| 4            | 101         | Michael  | 13000  | 2          |
| 3            | 101         | Michael  | 13000  | 3          |
| 7            | 102         | Pat      | 6000   | 1          |
| 6            | 102         | Pat      | 6000   | 2          |
| 8            | 103         | Den      | 11000  | 1          |


The records with row number 1 represent the unique records in the table. The below SQL query extracts unique records from EMPLOYEE table using ROW\_NUMBER analytic function.

```sql
SELECT EMPLOYEE_KEY, EMPLOYEE_ID, NAME, SALARY
FROM(
  SELECT
    EMPLOYEE_KEY,
    EMPLOYEE_ID,
    NAME,
    SALARY,
    ROW_NUMBER() OVER(PARTITION BY EMPLOYEE_ID ORDER BY EMPLOYEE_KEY DESC) AS ROW_NUMBER
  FROM EMPLOYEE
)
WHERE ROW_NUMBER = 1
;
```

**RESULT :**

| EMPLOYEE_KEY | EMPLOYEE_ID | NAME     | SALARY |
| ------------ | ----------- | -------- | ------ |
| 2            | 100         | Jennifer | 4400   |
| 5            | 101         | Michael  | 13000  |
| 7            | 102         | Pat      | 6000   |
| 8            | 103         | Den      | 11000       |


Once we have the query which gets the unique records from the table, we can use any of the two methods to replace records with unique records in the table.

The below SQL query overrides the data in the EMPLOYEE table with unique records using OVERWRITE command.

```sql
INSERT OVERWRITE INTO EMPLOYEE
SELECT EMPLOYEE_KEY, EMPLOYEE_ID, NAME, SALARY
FROM(
  SELECT
    EMPLOYEE_KEY,
    EMPLOYEE_ID,
    NAME,
    SALARY,
    ROW_NUMBER() OVER(PARTITION BY EMPLOYEE_ID ORDER BY EMPLOYEE_KEY DESC) AS ROW_NUMBER
  FROM EMPLOYEE
)
WHERE ROW_NUMBER = 1
;
```
