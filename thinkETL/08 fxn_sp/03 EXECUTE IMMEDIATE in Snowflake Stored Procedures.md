---
created: 2023-09-04T15:03:51 (UTC -04:00)
tags: [execimmediate, snowflake, storedproc, snowscripting]
source: https://thinketl.com/execute-immediate-in-snowflake-stored-procedures/
author: ThinkETL
---

## 1.EXECUTE IMMEDIATE in Snowflake Stored Procedures


**EXECUTE IMMEDIATE** command in Snowflake executes SQL statements present in form a string literal (character sets enclosed by single quotes or double dollar signs). EXECUTE IMMEDIATE returns the result of the executed SQL statement.

The string literal that EXECUTE IMMEDIATE executes be can any of the following.

- A single SQL statement
- A Stored Procedure call
- An Anonymous Stored Procedure block

## 2. Executing SQL statements using EXECUTE IMMEDIATE

The syntax to execute SQL statements using EXECUTE IMMEDIATE is as follows.

```sql
EXECUTE IMMEDIATE '<sql_query>';

EXECUTE IMMEDIATE $$ <sql_query> $$;
```

The below example executes the SQL statement defined in a string literal using EXECUTE IMMEDIATE command.

```sql
EXECUTE IMMEDIATE 'SELECT COUNT(*) FROM employees';
```

## 3. Executing Stored Procedures using EXECUTE IMMEDIATE

The syntax to execute Stored Procedures using EXECUTE IMMEDIATE is as follows.

```sql
EXECUTE IMMEDIATE 'CALL <stored_procedure_name>';

EXECUTE IMMEDIATE $$ CALL <stored_procedure_name> $$;
```

The below example executes a stored procedure named `my_procedure()` using EXECUTE IMMEDIATE command.

```sql
EXECUTE IMMEDIATE
 $$
    CALL my_procedure();
 $$;
```

## 4. Executing procedure blocks using EXECUTE IMMEDIATE

If you are running an anonymous block in Snowsight, you can execute the block directly without the need of EXECUTE IMMEDIATE command.

The following is an example of an anonymous block that you can run in **Snowsight**.

```sql
DECLARE
  net_sales NUMBER(38, 2);
  tax NUMBER(38, 2);
  gross_sales NUMBER(38, 2) DEFAULT 0.0;
BEGIN
  net_sales := 98.67;
  tax := 1.33;
  gross_sales := net_sales + tax;
  RETURN gross_sales;
END;
```

The output of the above anonymous block would be as follows:


| anonymous block |
| --------------- |
| 100             |

But if you are using **SnowSQL** or the **classic web interface**, you must specify the block as a string literal **(enclosed in single quotes or double dollar signs)**, and you must pass the block to the EXECUTE IMMEDIATE command as shown below.

```sql
EXECUTE IMMEDIATE
$$
  DECLARE
    net_sales NUMBER(38, 2);
    tax NUMBER(38, 2);
    gross_sales NUMBER(38, 2) DEFAULT 0.0;
  BEGIN
    net_sales := 98.67;
    tax := 1.33;
    gross_sales := net_sales + tax;
    RETURN gross_sales;
  END;
$$
;
```

## 5. Executing SQL statements in Stored Procedures using EXECUTE IMMEDIATE

The below stored procedure is an example which executes the SQL statements using EXECUTE IMMEDIATE.

- The first EXECUTE IMMEDIATE command executes the CREATE statement declared in the variable `**create_stmt**`.
- The second EXECUTE IMMEDIATE command executes the DELETE statement declared in variable `**delete_stmt**` in concatenation with a filter condition passed as a string.

> This also demonstrates that EXECUTE IMMEDIATE works not only with a string literal, but also with an expression that evaluates to a string (VARCHAR).

```sql
CREATE OR REPLACE PROCEDURE sp_execute_immediate_demo()
  RETURNS NUMBER
  LANGUAGE SQL
AS
$$
  DECLARE
    create_stmt VARCHAR DEFAULT 'CREATE OR REPLACE TABLE temp_emp AS SELECT * FROM employees';
    delete_stmt VARCHAR DEFAULT 'DELETE FROM temp_emp';
    result NUMBER DEFAULT 0;
  BEGIN
    EXECUTE IMMEDIATE create_stmt;
    EXECUTE IMMEDIATE delete_stmt || ' WHERE status= ''INACTIVE'' ';
    result := (SELECT COUNT(*) FROM temp_emp);
    RETURN result;
  END;
$$
;
```

The output of the procedures gives the record count in table `**temp_emp**` after removing the INACTIVE records.

## 6. EXECUTE IMMEDIATE with USING clause in Snowflake

The **EXECUTE IMMEDIATE** command is used in conjunction with **USING** clause to pass bind variables to the SQL query passed as a string literal to it. The bind variables are passed as a list separated by comma and enclosed in brackets.

The syntax to use EXECUTE IMMEDIATE with USING clause in Snowflake is as follows.

```sql
EXECUTE IMMEDIATE '<sql_query>' USING (bind_variable1, bind_variable2,…);
```

> A bind variable holds a value to be used in SQL query executed by EXECUTE IMMEDIATE command.

The below stored procedure is an example in which values to the filter condition of a SQL query executed by EXECETE IMMEDIATE command are passed through bind variables defined in USING clause.

```sql
CREATE OR REPLACE PROCEDURE purge_data_by_date()
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
  DECLARE
    sql_stmt VARCHAR DEFAULT 'DELETE FROM employees WHERE hire_date BETWEEN :1 AND :2';
    min_date DATE DEFAULT '2015-01-01';
    max_date DATE DEFAULT '2017-12-31';
    result VARCHAR;
  BEGIN
    EXECUTE IMMEDIATE sql_stmt USING (min_date, max_date);
    result := 'Data deleted between '|| min_date || 'and '|| max_date;
    return result;
  end;
$$
;
```

## 7. EXECUTE IMMEDIATE with INTO clause in Snowflake

Using INTO clause in conjunction with EXECUTE IMMEDIATE, we can specify the list of the user defined variables which will hold the values returned by SELECT statement in Oracle. But currently **EXECUTE IMMEDIATE with INTO clause is not supported in Snowflake** like in Oracle.

Instead we can still assign the values to the user defined variables from the result of a SQL statement by using INTO clause directly in SELECT statement as shown below.

```sql
SELECT id, firstname INTO :id_variable, :name_variable FROM employees WHERE id = 101;
```
