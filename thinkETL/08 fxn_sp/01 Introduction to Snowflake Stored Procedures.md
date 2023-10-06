---
created: 2023-09-04T14:58:49 (UTC -04:00)
tags: [stored proc, snowscripting, snowflake]
source: https://thinketl.com/introduction-to-snowflake-stored-procedures/
author: ThinkETL
---

# Introduction to Snowflake Stored Procedures 

## 1. What are Stored Procedures?

Stored procedures allow you to write procedural code that executes business logic by combining multiple SQL statements. In a stored procedure, you can use programmatic constructs to perform branching and looping.

A stored procedure is created with a **CREATE PROCEDURE** command and is executed with a **CALL** command.

Snowflake supports writing stored procedures in multiple languages. In this article we will discuss on writing stored procedures using **Snowflake SQL Scripting**.

## 2. Stored Procedure Syntax in Snowflake

The following is the basic syntax for creating Stored Procedures in Snowflake.

```sql
CREATE OR REPLACE PROCEDURE <name> ( [ <arg_name> <arg_data_type> ] [ , ... ] )
   RETURNS <result_data_type>
   LANGUAGE SQL
AS
   $$
      <procedure_body>
   $$
;
```

Note that you must use string literal delimiters `(‘ or $$)` around procedure definition(body) if you are creating a Snowflake Scripting procedure in Classic Web Interface or SnowSQL. The string literal delimiters `(‘ or $$)` are not mandatory when writing procedures in SnowSight.

Let us understand the various parameters in the stored procedure construct.

### 2.1. NAME <name>

Specifies the name of the stored procedure.

The name must start with an alphabetic character and cannot contain spaces or special characters unless the entire identifier string is enclosed in double quotes (e.g. “My Procedure”). Identifiers enclosed in double quotes are also case-sensitive.

2.2. INPUT PARAMETERS ( [ <arg_name> <arg_data_type> ] [ , … ] )

A Stored Procedures can be built which takes one or more arguments as input parameters or even without any input parameters.

-   The <arg_name> specifies the name of the input argument.
-   The <arg_data_type> specifies the SQL data type of the input argument.

```sql
-- Stored Procedure with multiple input arguments
CREATE OR REPLACE PROCEDURE my_proc( id NUMBER, name VARCHAR)

-- Stored Procedure with single input argument
CREATE OR REPLACE PROCEDURE my_proc( id NUMBER)

-- Stored Procedure with no input arguments
CREATE OR REPLACE PROCEDURE my_proc()
```


### 2.3. RETURNS <result_data_type>

Specifies the type of the result returned by the stored procedure.

```sql
CREATE OR REPLACE PROCEDURE my_proc()
   RETURNS VARCHAR
```

### 2.4. LANGUAGE SQL

Since Snowflake supports stored procedures in multiple languages, the **LANGUAGE** parameter specifies the language of the stored procedure definition. For Snowflake scripting, the value to the LANGUAGE parameter is passed as **SQL**.

```sql
CREATE OR REPLACE PROCEDURE my_proc()
   RETURNS VARCHAR
   LANGUAGE SQL
```

### 2.5. PROCEDURE BODY

The body defines the code executed by the stored procedure. The procedure definition is mentioned after the **AS** clause in the stored procedure construct. As mentioned earlier the body is wrapped between `$$` string literal delimiters if the procedure scripting is not done in SnowSight.

## 3. Understanding various sections in Stored Procedure Body

The Stored Procedure Body is made up of multiple sections. The various sections in the stored procedure body are as follows.

```sql
DECLARE
  ... (variable declarations, cursor declarations, etc.) ...
BEGIN
  ... (Snowflake Scripting and SQL statements) ...
EXCEPTION
  ... (statements for handling exceptions) ...
END;
```

### 3.1. DECLARE

The DECLARE section is used to define any variables, cursors etc. used in the body. Alternatively they can be declared in the BEGIN…END section of the body also.

### 3.2. BEGIN…END

The SQL statements and scripting constructs are written between the BEGIN and END sections of the body.

### 3.3. EXCEPTION

The EXCEPTION section of the body is used to hold any exception handling code you wanted to add.

> Note that DECLARE and EXCEPTION sections are not mandatory in every procedure definition.

A simple stored procedure body just requires BEGIN and END sections.

```sql
BEGIN
      CREATE TABLE employees(id NUMBER, firstname VARCHAR);
END;
```

## 4. Creating a Stored Procedure in Snowflake

Consider a use case where the requirement is to purge the inactive employees’ data from a database table. Let us build a Stored Procedure which performs this activity.

The below Stored Procedure deletes all records with status field value as ‘INACTIVE’ from the employees table.

```sql
CREATE OR REPLACE PROCEDURE purge_data()
    RETURNS VARCHAR
    LANGUAGE SQL
AS
 $$
    DECLARE
        message VARCHAR;
    BEGIN
        DELETE FROM employees WHERE status = 'INACTIVE';
        message := 'Inactive employees data deleted successfully';
        RETURN message;
    END;
 $$
;
```

Let us break down each block of the stored procedure below to understand better.

  - The name of the stored procedure is `_purge_data_` and do not take any input parameters.
  - The data type of the return value from store procedure is defined as `_varchar_`.
  - The language is defined as `_SQL_`, the language in which the procedure body is defined.
  - A variable named `_message_` of type `_varchar_` is defined under DECLARE section of the body.
  - Between BEGIN…END section of the procedure body,

    -   The statement to delete the records with INACTIVE status is defined. The variable message is assigned a string value. The assignment operator used is `:=` for assigning value to variable.

    -   The variable message is returned as the output from the stored procedure.

## 5. Creating a Stored Procedure with Input Parameters

Consider another scenario where you wanted to purge the data from a table based on an input you passed. Let us understand with an example.

The below Stored Procedure deletes all records with status value that matches the value passed as an input through an input parameter `_in_status_` from the employees table.

```sql
CREATE OR REPLACE PROCEDURE purge_data_by_status(in_status VARCHAR)
    RETURNS VARCHAR
    LANGUAGE SQL
AS
 $$
    DECLARE
        message VARCHAR;
    BEGIN
        DELETE FROM employees WHERE status = :in_status;
        message := in_status ||' employees data deleted sucessfully';
        RETURN message;
    END;
 $$
;
```

The above stored procedure is similar to that of the one defined under section-4 of the article except,

 - The name of the procedure is `_purge_data_by_status_` and accepts an input through parameter named `_in_status_` of type varchar.
 - The input parameter is used in the SQL statement which deletes the data from employees table. Prefix the input parameter with colon (**:in\_status**) to use in a SQL statement.
 - The same input parameter is also used in the string value assigned to message variable indicating records with which status are deleted.

The above stored procedure can be simplified by eliminating the DECLARE section as shown below.

```sql
CREATE OR REPLACE PROCEDURE purge_data_by_status(in_status VARCHAR)
	RETURNS VARCHAR
    LANGUAGE SQL
AS
$$
BEGIN
	DELETE FROM employees WHERE status = :in_status;
	RETURN in_status ||' employees data deleted successfully';
END;
$$
;
```

## 6. Calling a Stored Procedure in Snowflake

Use CALL command to execute a stored procedure in Snowflake.

The following is the syntax to CALL command

```sql
CALL <procedure_name> ( [ <arg1> , ... ] )
```

The below image shows calling a stored procedure named `_purge_data_` and the output of the stored procedure call.

![Call Stored Procedure without any Input Parameters](https://thinketl.com/wp-content/uploads/2023/02/113-1-Call-Stored-Procedure-without-any-Input-Parameters.png)

Call Stored Procedure without any Input Parameters

The below image shows calling a stored procedure named `_purge_data_by_status_` with a string input parameter _‘**INACTIVE**’_ and the output of the stored procedure call.

```sql
CALL purge_data_by_status('INACTIVE');
```

![Call Stored Procedure with Input Parameters](https://thinketl.com/wp-content/uploads/2023/02/113-2-Call-Stored-Procedure-with-Input-Parameters.png)

Call Stored Procedure with Input Parameters

## 7. Closing Points

As the name of the article suggests, it is just an introduction to building stored procedures in Snowflake. We have barely scratched the surface in understanding the complete concepts of stored procedures. Nevertheless, I hope it is a good starting point to begin learning Snowflake stored procedures.