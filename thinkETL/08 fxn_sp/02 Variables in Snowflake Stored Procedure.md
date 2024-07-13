---
created: 2023-09-04T15:00:42 (UTC -04:00)
tags: []
source: https://thinketl.com/variables-in-snowflake-stored-procedure/
updated: 2024-04-21 12:55:00
---

# Variables in Snowflake Stored Procedure

## 1.Variables

A Variable is a named object which holds a value of a specific data type whose value can change during the stored procedure execution. Variables in Snowflake stored procedures are local to stored procedures are used to hold intermediate results.

In this article let us discuss in-detail about declaring and using variables in Snowflake Stored Procedures. To learn more about creating a Stored Procedure, refer our article – **[Introduction to Snowflake Stored Procedures](https://thinketl.com/introduction-to-snowflake-stored-procedures/)**

## 2. Declaring Variables in Snowflake Stored Procedures

A Variable must be declared before using it in Stored Procedures. When a variable is declared, the type of the variable must be specified by either:

-   Explicitly specifying the data type. The data type of variable can be
    
    -   SQL data type
    
    -   CURSOR
    
    -   RESULTSET
    
    -   EXCEPTION
-   Specifying an initial value for the variable using **DEFAULT** command. Snowflake Scripting uses the DEFAULT value to determine the type of the variable.

A Variable in Snowflake can be declared either in **DECLARE** section or **BEGIN…END** section of the stored procedure body or both.

The below example shows variable declaration in **DECLARE** section of the stored procedure body.

```sql
-- Variable declaration in DECLARE section of body
<variable_name> <type>;

<variable_name> DEFAULT <expression> ;

<variable_name> <type> DEFAULT <expression> ;

-- Examples
net_sales NUMBER(38,2);

net_sales DEFAULT 98.67;

net_sales NUMBER(38,2) DEFAULT 98.67;
```

The below example shows variable declaration in **BEGIN…END** section of the stored procedure body. 

```sql
-- Variable declaration in BEGIN...END section of body
LET <variable_name> { DEFAULT | := } <expression> ;

LET <variable_name> <type> { DEFAULT | := } <expression> ;

-- Examples
LET net_sales := 98.67;

LET net_sales DEFAULT 98.67;

LET net_sales NUMBER(38,2) := 98.67;

LET net_sales NUMBER(38,2) DEFAULT 98.67;
```

> _Note that the variable should be preceded with **LET** command while declaring variables in BEGIN…END section of the body._

## 3. Assigning values to Declared Variables in Snowflake Stored Procedures

To assign a value to a variable that has already been declared, use the **:=** operator:

```sql
<variable_name> := <expression> ;
```

You can use another declared variables in the expression to assign the resulting value to the variable.

The below example shows

-   A variable named `gross_sales` declared under DECLARE section of the body with initial default value as `0.0` using DEFAULT command
-   Two variables declared in the BEGIN…END section of the body using LET command.
    
    -   The variable `net_sales` is assigned a value as `98.67` using **:=** operator.
    
    -   The variable `tax` is declared with an initial value of `1.33` using DEFAULT command.
-   The variable `gross_sales` is assigned a resulting value of the summation expression of variables `net_sales` and `tax`.
-   Finally the variable `gross_sales` is returned as an output using RETURN command.

```sql
DECLARE
    gross_sales NUMBER(38, 2) DEFAULT 0.0;
BEGIN
    LET net_sales NUMBER(38, 2) := 98.67;
    LET tax NUMBER(38, 2) DEFAULT 1.33;

    gross_sales := net_sales + tax;

    RETURN gross_sales;
END;
```

## 4. Using a Variable in a SQL Statement (Binding)

The variables declared in the stored procedure can be used in the SQL statements using **colon as prefix** to the variable name. For example:

```sql
DELETE FROM EMPLOYEES WHERE ID = :in_employeeid;
```

If you are using the variable as the name of an object, use the **IDENTIFIER** keyword to indicate that the variable represents an object identifier. For example:

```sql
DELETE FROM IDENTIFIER(:in_tablename) WHERE ID = : in_employeeid;
```

If you are building a SQL statement as a string to execute, the variable does not need the colon prefix. For example:

```sql
LET sql_stmt := 'DELETE FROM EMPLOYEES WHERE ID = ' || in_employeeid;
```

Note that if you are using the variable with RETURN, you do not need the colon prefix. For example:

## 5. Assigning result of a SQL statement to Variables using INTO clause in Snowflake Stored Procedures

You can assign expression result of a SELECT statement to Variables in Snowflake Stored Procedures using **INTO** clause.

The syntax to assign result of a SQL statement to variables is as below.

```sql
SELECT <expression1>, <expression2>, ... INTO :<variable1>, :<variable2>, ... FROM ... WHERE ...;
```

In the syntax:

-   The value of `<expression1>` is assigned to `<variable1>`.
-   The value of `<expression2>` is assigned to `<variable2>`.

> _Note that the SELECT statement used to assign values to variables must return only single output row._

Consider below data as an example to understand how it works.

```sql
CREATE OR REPLACE TABLE employees (id INTEGER, firstname VARCHAR);

INSERT INTO employees (id, firstname) VALUES
  (101, 'TONY'),
  (102, 'STEVE');
```

The below stored procedure assigns the **id** and **firstname** of the employee with id 101 into variables `id_variable` and `name_variable` respectively.

```sql
CREATE OR REPLACE PROCEDURE get_employeedata()
    RETURNS VARCHAR
    LANGUAGE SQL
AS
 $$
    DECLARE
      id_variable INTEGER;
      name_variable VARCHAR;
    BEGIN
      SELECT id, firstname INTO :id_variable, :name_variable FROM employees WHERE id = 101;
      RETURN id_variable || ' ' || name_variable;
    END;
$$
;
```

When the stored procedure is executed, the output returns the concatenated values of id and name of the employee with id 101.

<table><tbody><tr><td><strong>GET_EMPLOYEEDATA</strong></td></tr><tr><td>101 TONY</td></tr></tbody></table>

## 6. Variable Scope in Snowflake Stored Procedures

If you have nested blocks in your stored procedures and multiple variables with same name are declared in it, the scope of the variable will be local to the block its declared.

For example, if you have an outer block and inner block where you have declared a variable `my_variable` and assigned value as 5 in outer block and 7 in inner block. As long as the variable is used in the inner block, the value remains 7 and all operations outside the inner block, the value assigned to variable remains 5.

When a variable name is referenced, Snowflake looks for the variable by starting first in the current block, and then working outward one block at a time until a matching name is found.

