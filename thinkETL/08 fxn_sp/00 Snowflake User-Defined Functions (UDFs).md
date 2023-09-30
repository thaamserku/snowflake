---
created: 2023-09-07T21:20:31 (UTC -04:00)
tags: [snowflake, udf,functions]
source: https://thinketl.com/snowflake-user-defined-functions-udfs/
author: ThinkETL
---

# Snowflake User-Defined Functions (UDFs)

> ## Excerpt
> UDF is a reusable component defined by user to perform a specific task which can be called from a SQL statement.

---
## 1. Introduction

There are instances where there are certain requirements that cannot be fulfilled with the existing built-in system defined functions provided by Snowflake. In such cases, Snowflake allows users to create their own functions based on their requirement called User-Defined functions. Once created these functions can be reused multiple times.

In this article, let us deep-dive into understanding User-Defined Functions (UDFs) in Snowflake.

## 2. What is a User-Defined Function?

**A Snowflake User-Defined Function is a reusable component defined by user to perform a specific task which can be called from a SQL statement. Similar to built-in functions, the user-defined functions can be called from a SQL repeatedly from multiple places in a code.**

## 3. UDF Supported Languages in Snowflake

Though the UDFs are created using SQL, Snowflake supports writing the body of the function which holds the logic in multiple languages. Each language allows you to manipulate data within the constraints of the language and its runtime environment.

Below are the languages supported by Snowflake for writing UDFs:

1.  Java
2.  JavaScript
3.  Python
4.  Scala
5.  SQL

In this article, we will focus on building UDFs using SQL Scripting.

## 4. Types of User-Defined Functions

A SQL UDF evaluates an arbitrary SQL expression and returns the result(s) of the expression. Based on the return value(s) provided by a function, the UDFs are of two different types.

1.  Scalar Function (UDF) – returns a single value.
2.  Table Function (a user-defined table function, or UDTF) – returns a set of rows as tabular value.

Let us understand these UDF types with examples in the further sections of the article.

## 5. Creating User-Defined Function in Snowflake

The User-Defined Functions (UDFs) are created using a CREATE FUNCTION command. Below is the syntax to create UDFs in Snowflake.

```sql
CREATE OR REPLACE FUNCTION <name> ( [ <arg_name> <arg_data_type> ] [ , ... ] )
RETURNS <result_data_type>
LANGUAGE <language>
AS
   $$
      <function_body>
   $$
;
```

The syntax to create UDFs is similar to the creating Stored Procedures in Snowflake. For more details about the various parameters used in the syntax, refer our [**previous article**](https://thinketl.com/introduction-to-snowflake-stored-procedures).

## 6. Calling User-Defined Function in Snowflake

A User-defined function (UDF) or a User-defined table function (UDTF) can be called in the same way that you call other functions.

**Calling a UDF:**

A UDF can be called using a SELECT statement as shown below. If a UDF has arguments, you can specify those arguments by name or by position.

```sql
SELECT udf_name(udf_arguments) ;
```

**Calling a UDTF:**

A UDTF can be called in a way any table function would be called. A UDTF is called in a FROM clause of a query using **TABLE** keyword followed by UDTF name and its arguments wrapped inside a parentheses as shown below

```sql
SELECT ...
  FROM TABLE ( udtf_name (udtf_arguments) ) ;
```

## 7. Scalar Function with Examples

**A Scalar Function (UDF) returns a single row as output for each input row. The returned row consists of a single column/value.**

### **Example-1: Convert Datetimestamp value into a Date value using a UDF.**

The below UDF `_get_date_` takes Datetimestamp value as an input argument and returns a date value.

```sql
CREATE OR REPLACE FUNCTION get_date(business_date timestamp)
RETURNS DATE
LANGUAGE SQL
AS
$$
    TO_DATE(SUBSTR(TO_CHAR(business_date),1,10))
$$;
```

The call to the UDF can be made as shown below.

```sql
SELECT get_date('2023-01-01 12:53:22.000');
```

Output:

<table><tbody><tr><td><strong>GET_DATE(‘2023-01-01 12:53:22.000’)</strong><strong></strong></td></tr><tr><td>2023-01-01</td></tr></tbody></table>

### **Example-2: Calling UDF in a SELECT query.**

Consider below _`Sales`_ table as an example for demonstration purpose.

```sql
CREATE OR REPLACE TABLE SALES(
   sale_datetime TIMESTAMP,
   sale_amount NUMBER(19,4)
);

INSERT INTO SALES VALUES
('2023-01-01 12:53:22.000','2876.93'),
('2023-01-02 01:14:55.000','3509.75'),
('2023-01-03 01:05:12.000','2971.66'),
('2023-01-04 12:47:49.000','3328.32');
```

The same UDF _`get_date`_ created in example-1 is used for converting the datetimestamp values into date value while reading data from `_Sales_` table by calling UDF in the SELECT statement as shown below.

```sql
SELECT
   get_date(sale_datetime) AS sale_date,
   sale_amount
FROM SALES;
```

Output:

<table><tbody><tr><td><strong>SALE_DATE</strong></td><td><strong>SALE_AMOUNT</strong></td></tr><tr><td>2023-01-01</td><td>2876.9300</td></tr><tr><td>2023-01-02</td><td>3509.7500</td></tr><tr><td>2023-01-03</td><td>2971.6600</td></tr><tr><td>2023-01-04</td><td>3328.3200</td></tr></tbody></table>

### **Example-3: Calling UDF in a WHERE clause of a SELECT query.**

The SQL query below is an example where UDF is applied on a field used in the WHERE clause of SELECT statement.

```sql
SELECT * FROM sales
WHERE  get_date(sale_datetime) > '2023-01-02';
```

Output:

<table><tbody><tr><td><strong>SALE_DATE</strong></td><td><strong>SALE_AMOUNT</strong></td></tr><tr><td>2023-01-03</td><td>2971.6600</td></tr><tr><td>2023-01-04</td><td>3328.3200</td></tr></tbody></table>

### **Example-4: UDF with a Query Expression with SELECT Statement.**

Below is an example of UDF which when called provides the sum of total sales amount from _`Sales`_ table.

```sql
CREATE OR REPLACE FUNCTION get_total_sales()
RETURNS NUMBER(19,4)
LANGUAGE SQL
AS
$$
    SELECT SUM(sale_amount) FROM SALES
$$;
```

> **When using a query expression in a SQL UDF, do not include a semicolon (;) within the UDF body to terminate the query expression.**

Calling the UDF using SELECT.

```sql
SELECT get_total_sales();
```

Output:

<table><tbody><tr><td><strong>GET_TOTAL_SALES()</strong><strong></strong></td></tr><tr><td>12686.6600</td></tr></tbody></table>

> **Although the body of a UDF can contain a complete SELECT statement, it cannot contain DDL statements or any DML statement other than SELECT.**

## 8. Table Function with Examples

**Table Functions or a Tabular SQL UDFs (UDTFs) returns a set of rows consisting of 0, 1 or more rows each of which has 1 or more columns.**

While creating UDTFs using CREATE FUNCTION command, the `<_result_data_type>_` should be **TABLE(…)**. Inside the parentheses specify the output column names along with the expected data type.

Consider below tables _`sales_by_country`_, _`currency`_ as an example for demonstration purpose.

```sql
CREATE OR REPLACE TABLE sales_by_country(
year NUMBER(4),
country VARCHAR(50),
sale_amount NUMBER
);

INSERT INTO SALES_BY_COUNTRY VALUES
('2022','US','90000'),
('2022','UK','75000'),
('2022','FR','55000'),
('2023','US','100000'),
('2023','UK','80000'),
('2023','FR','70000');

CREATE OR REPLACE TABLE currency(
country VARCHAR(50),
currency VARCHAR(3)
);

INSERT INTO CURRENCY VALUES
('US','USD'),
('UK','GBP'),
('FR','EUR');
```

### **Example-1: Basic example of a UDTF which returns data from a table.**

The below UDTF is an example which returns data from a table based on an input argument value.

```sql
CREATE OR REPLACE FUNCTION get_sales(country_name VARCHAR)
RETURNS TABLE (year NUMBER, sale_amount NUMBER, country VARCHAR)
AS
$$
    SELECT year, sale_amount, country
    FROM sales_by_country
    WHERE country = country_name
$$
;
```

Calling the UDTF to return the data of **US** country.

```sql
SELECT * FROM TABLE(get_sales('US'));
```

Output:

| **YEAR** | **SALES\_AMOUNT** | **COUNTRY** |
| --- | --- | --- |
| 2022 | 90000 | US |
| 2023 | 100000 | US |

### **Example-2: UDTF with Joins in a SQL Query.**

The below UDTF is an example which joins two tables to return the required data from both the tables based on an input argument value.

```sql
CREATE OR REPLACE FUNCTION get_sales_with_currency(country_name VARCHAR)
RETURNS TABLE (year NUMBER, sale_amount NUMBER, country VARCHAR, currency VARCHAR)
AS
$$
    SELECT a.year, a.sale_amount, a.country, b.currency
    FROM sales_by_country a
    JOIN currency b
    ON a.country = b.country
    WHERE a.country = country_name
$$
;
```

Calling the UDTF to return the data of **US** country.

```sql
SELECT * FROM TABLE(get_sales_with_currency ('US'));
```

Output:

| **YEAR** | **SALES\_AMOUNT** | **COUNTRY** | **CURRENCY** |
| --- | --- | --- | --- |
| 2022 | 90000 | US | USD |
| 2023 | 100000 | US | USD |

### **Example-3: Using UDTF in a Join of a SQL Query.**

The below SQL statement is an example where data from a UDTF (_`get_sales`_) is joined with a table (_`Currency`_).

```sql
SELECT a.year, a.sale_amount, a.country, b.currency
FROM TABLE(get_sales('US')) a
JOIN currency b
ON a.country = b.country
;
```

Output:

| **YEAR** | **SALES\_AMOUNT** | **COUNTRY** | **CURRENCY** |
| --- | --- | --- | --- |
| 2022 | 90000 | US | USD |
| 2023 | 100000 | US | USD |

## 9. Difference between UDF and Stored Procedure

**1. UDFs Return a Value. Stored Procedures Need Not.**

-   The main purpose of a UDF is to calculate and return value. Whereas a Stored Procedure is used to perform administrative operations by executing SQL statements where it is not required to explicitly return a value.

**2. UDF Return Values Are directly usable in SQL. Stored Procedure Return Values may not be.**

-   The value returned by a stored procedure, unlike the value returned by a function, cannot be used directly in SQL.

**3. UDFs do not support DDL, DML queries like Stored Procedures.**

-   UDFs only support SELECT statements in the function body whereas Stored Procedures also allow DDL, DML queries inside the procedure body.

**4. UDFs can be called in the context of another statement. Stored Procedures are called Independently.**

-   A stored procedure is called as an independent statement whereas a UDF is always called inside the context of a SELECT statement.

```sql
SELECT MyStoredProcedure(argument_1);  -- Not Supported

CALL MyStoredProcedure(argument_1);

SELECT MyUDF(column_1) FROM table1;
```

**5. Multiple UDFs may be called within one statement. A Single Stored Procedure is called as one statement.**

-   A single executable statement can call only one stored procedure. In contrast, a single SQL statement can call multiple functions.

```sql
CALL MyStoredProcedure(argument_1);

SELECT MyUDF_1(column_1), MyUDF_2(column_2)  FROM table1;
```
