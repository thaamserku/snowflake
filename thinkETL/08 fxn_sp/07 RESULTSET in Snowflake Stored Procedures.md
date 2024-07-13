---
created: 2023-09-04T15:11:42 (UTC -04:00)
tags: []
source: https://thinketl.com/resultset-in-snowflake-stored-procedures/
author: ThinkETL
---

# `RESULTSET` in Snowflake Stored Procedures 


## 1. Introduction

Snowflake allows storing the entire rows present in the result set of a SELECT statement and return them as output in the form a table using `RESULTSET`. `RESULTSET` is a SQL data type supported only in Snowflake Stored Procedures that points to the result set of a query.

The results that `RESULTSET` points to can be accessed using one of the following ways.

-   Return the results as a table using the TABLE() syntax.
-   Assigning the `RESULTSET` to a CURSOR and looping over it.

The syntax of `RESULTSET` includes two different parts

1.  Declaring a `RESULTSET` and assigning a SQL statement.
2.  Accessing data from a `RESULTSET`
    -   Returning the data of a `RESULTSET` as a table.
    -   Accessing data from a `RESULTSET` using a Cursor

### 2.1. Declaring a `RESULTSET`

The `RESULTSET` can be declared either in the `DECLARE` or `BEGIN…END `section of the stored procedures.

The below is the syntax to declare a **`RESULTSET`** in the **`DECLARE`** section of the stored procedure.

```sql
DECLARE
  <resultset_name> RESULTSET DEFAULT ( <query> );
```

The below is the syntax to declare a **`RESULTSET`** in the **`BEGIN…END`** section of the stored procedure.

```sql
BEGIN
  LET <resultset_name> := ( <query> );
                    (or)
  LET <resultset_name> RESULTSET := ( <query> );
END;
```

### 2.2. Returning the data of a RESULTSET as a Table

In order to return the results of a RESULTSET as an output, pass the RESULTSET to `TABLE()`. In the CREATE PROCEDURE, the return type should be declared as a TABLE along with the columns and their data types.

The below is the syntax to return a **RESULTSET** as a **Table** in a stored procedure.

```sql
CREATE PROCEDURE sp_demo()
RETURNS TABLE( column_1 <data_type>, column_2 <data_type>,…)
…
BEGIN
  …
  RETURN TABLE(<resultset_name>);
END;
```

### 2.3. Accessing data from a RESULTSET using a Cursor

Instead of assigning the SELECT query directly to the CURSOR, you can assign a variable of type RESULTSET which holds the query.

The below is the syntax to access data from a **`RESULTSET`** using a **`CURSOR`**.

```sql
BEGIN
  LET <resultset_name> RESULTSET := ( <query> );
  LET <cursor_name> CURSOR FOR <resultset_name>;
  …
END;
```

To learn more about using RESULTSET with CURSORS, refer our previous article on cursors.

## 3. Return output of a SELECT statement in Stored Procedures using RESULTSET

In order to return the output of a SELECT statement in stored procedures, we can assign the SQL statement to a variable of type `RESULTSET` and return the data from `RESULTSET` as a table.

Let us understand how to implement `RESULTSET`s in stored procedures using examples. Consider below EMPLOYEES data as an example for demonstration purpose.

```sql
CREATE OR REPLACE TABLE EMPLOYEES(
    ID NUMBER,
    EMP_NAME VARCHAR(50)
);

INSERT INTO EMPLOYEES VALUES (101, 'TONY'), (102, 'STEVE');
```

### 3.1. Declaring RESULTSET with a DEFAULT Clause

A `RESULTSET` can be declared in `DECLARE` section of the stored procedure and assigned with a `DEFAULT` `SELECT` query while declaration as shown below.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_resultset()
RETURNS TABLE(ID NUMBER, ENAME VARCHAR)
LANGUAGE SQL
AS
$$
  DECLARE
    res RESULTSET DEFAULT (SELECT ID, EMP_NAME FROM EMPLOYEES);
  BEGIN
    RETURN TABLE(res);
  END;
$$
;
```

The output of the stored procedure is as follows.

```sql
CALL sp_demo_resultset();
```

<table><tbody><tr><td><strong>ID</strong></td><td><strong>ENAME</strong></td></tr><tr><td>101</td><td>TONY</td></tr><tr><td>102</td><td>STEVE</td></tr></tbody></table>

### 3.2. Declaring RESULTSET without a DEFAULT Clause

Instead of assigning the `SELECT` query while declaring using DEFAULT clause, the query can be assigned after declaration in BEGIN…END section of the procedure as below.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_resultset2()
RETURNS TABLE(ID NUMBER, ENAME VARCHAR)
LANGUAGE SQL
AS
$$
  DECLARE
    res RESULTSET;
  BEGIN
    res := (SELECT ID, EMP_NAME FROM EMPLOYEES);
    RETURN TABLE(res);
  END;
$$
;
```

The output of the stored procedure is as follows.

```sql
CALL sp_demo_resultset2();
```

<table><tbody><tr><td><strong>ID</strong></td><td><strong>ENAME</strong></td></tr><tr><td>101</td><td>TONY</td></tr><tr><td>102</td><td>STEVE</td></tr></tbody></table>

### 3.3. Constructing the SQL statement dynamically for RESULTSET

A SQL statement can also be constructed dynamically as a string expression. But it cannot be directly assigned to a variable of type RESULTSET as shown in previous examples.

The SQL expression should be executed using **[`EXECUTE IMMEDIATE`](https://thinketl.com/execute-immediate-in-snowflake-stored-procedures/)** before assigning it to a RESULTSET.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_resultset_dynamic_query(var_id number)
RETURNS TABLE(ID NUMBER, ENAME VARCHAR)
LANGUAGE SQL
AS
$$
  DECLARE
    res RESULTSET;
    sql_query VARCHAR;
  BEGIN
    sql_query := 'SELECT ID, EMP_NAME FROM EMPLOYEES WHERE ID ='|| var_id;
    res := (EXECUTE IMMEDIATE :sql_query);
    RETURN TABLE(res);
  END;
$$
;
```

The output of the stored procedure is as follows.

```sql
CALL sp_demo_resultset_dynamic_query(101);
```

<table><tbody><tr><td><strong>ID</strong></td><td><strong>ENAME</strong></td></tr><tr><td>101</td><td>TONY</td></tr></tbody></table>

## 4. Difference between a RESULTSET and CURSOR in Snowflake Stored Procedures

Though both **RESULTSET** and **[CURSOR](https://thinketl.com/cursors-in-snowflake-stored-procedures/)** provide access to the result set of a query, they differ in following ways.

**1.** The main difference between a `RESULTSET` and `CURSOR` in Snowflake Stored Procedures is that _the Cursors allow you to loop through each row of the query result set and apply certain actions on each row._ Whereas it is not supported with the RESULTSET.

In order to loop through each row of the result set of a query that RESULTSET points to, it again needs to be assigned to a Cursor.

**2.** The query assigned to the `RESULTSET` is executed when the query is assigned to it either in DECLARE or `BEGIN…END` section of the procedure.

For Cursors, the query is not executed during assignment. The query is executed when you open the cursor using `OPEN` command.

**3.** Binding variables to the query assigned is supported in Cursors whereas it is not supported in `RESULTSET`

## 5. Conclusion

Snowflake supports `RESULTSET` only inside Stored Procedures with SQL Scripting.

Snowflake does not support

-   Declaring an input parameter of type `RESULTSET`.
-   Declaring a stored procedure’s return type as `RESULTSET`.
-   Declaring column of type of `RESULTSET`.
-   Querying `RESULTSET` as a table using `SELECT`.
