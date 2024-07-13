---
created: 2023-09-30T11:11:23 (UTC -04:00)
tags: [snowflake, storedproc, exceptions]
source: https://thinketl.com/exceptions-in-snowflake-stored-procedures/
---

# Exceptions in Snowflake Stored Procedures

## 1. Introduction

An exception occurs during the execution of a procedure when an instruction is encountered which cannot be executed at run-time due to an error. Apart from run-time errors, Snowflake also lets you raise an exception manually whenever an undesired result is encountered to prevent the next lines of code from executing.

Snowflake also allows catching an exception for the errors that can occur in our code and handle the exception by defining exception handlers for each exception. These exception handlers contain the code that needs to be executed when that particular exception arises.

In this article let us discuss how to declare, raise and catch exceptions in Snowflake Stored Procedures.

## 2. Declaring an Exception in Snowflake Stored Procedures

The user-defined Exception needs to be declared in the **DECLARE** section of the stored procedure.

The syntax to declaring an exception in **DECLARE** section of the stored procedure is as shown below.

```sql
DECLARE
  <exception_name> EXCEPTION ( <exception_number>, '<execption_message>') ;
```

**Exception\_name**:

The name of the user-defined exception provided by the user.

**Exception\_Number**:

This is the number to uniquely identify the exception which should be between -20000 to -20999. Same number should not ne user for multiple exceptions with in the same procedure.

If you don’t not specify a number for the exception, the default value used is -20000.

**Exception\_Message**:

This is the text that describes the exception. The text must not contain any double quote characters.

The below is an example of declaring exceptions with and without exception number and message.

```sql
DECLARE
  MY_EXCEPTION1 EXCEPTION;
  MY_EXCEPTION2 EXCEPTION(-20000,'Raised user defined exception MY_SP_EXCEPTION1.');
```

## 3. Raising an Exception in Snowflake Stored Procedures

An exception can be raised manually by executing the **RAISE** command.

The syntax to raise an exception using **RAISE** command in the **BEGIN..END** section of the stored procedure is as shown below.

The below is a simple example showing an exception being raised using RAISE command.

```sql
CREATE OR REPLACE PROCEDURE sp_raise_exception()
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
  DECLARE
    MY_SP_EXCEPTION EXCEPTION;
  BEGIN
    RAISE MY_SP_EXCEPTION;
  END;
$$
;
```

The output of the stored procedure is as follows.

```sql
CALL sp_raise_exception();
```

![](https://thinketl.com/wp-content/uploads/2023/03/120-1-Exception.png)

The below is another simple example showing an exception being raised using RAISE command where exception number and message are defined.

```sql
CREATE OR REPLACE PROCEDURE sp_raise_exception()
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
  DECLARE
    MY_SP_EXCEPTION EXCEPTION(-20001, 'Raised user defined exception MY_SP_EXCEPTION.');
  BEGIN
    RAISE MY_SP_EXCEPTION;
  END;
$$
;
```

The output of the stored procedure is as follows.

```sql
CALL sp_raise_exception();
```

![](https://thinketl.com/wp-content/uploads/2023/03/120-2-Exception.png)

> Note that the exception number and message are displayed as per the exception definition and are different from previous example.

## 4. Catching an Exception in Snowflake Stored Procedures

Whenever we are raising and exception using the **RAISE** command, the job fails providing the information of the error.

Instead of letting the job fail, we can also handle the exception by catching it using the **EXCEPTION** block of the stored procedure.

The syntax to catch an exception using the **EXCEPTION** block in the stored procedure is as shown below.

```sql
BEGIN
…
EXCEPTION
  WHEN <exception_name> THEN
  <statement>;
END;
```

The below is an example showing how to catch an exception using **EXCEPTION** block in stored procedures.

```sql
CREATE OR REPLACE PROCEDURE sp_raise_exception()
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
  DECLARE
    MY_SP_EXCEPTION EXCEPTION(-20001, 'Raised user defined exception MY_SP_EXCEPTION.');
  BEGIN
    RAISE MY_SP_EXCEPTION;
  EXCEPTION
    WHEN MY_SP_EXCEPTION THEN
      RETURN 'Raised user defined exception MY_SP_EXCEPTION';
  END;
$$
;
```

The output of the stored procedure is as follows.

```sql
CALL sp_raise_exception();
```

![](https://thinketl.com/wp-content/uploads/2023/03/120-3-Exception.png)

## 5. Built-in Exception Variables in Snowflake Stored Procedures

Snowflake provides some built-in variables which provides information about the exceptions raised in the stored procedure.

The three built-in exception variables are as follows:

1.  SQLCODE
2.  SQLERRM
3.  SQLSTATE

### 5.1. SQLCODE

The SQLCODE variable captures the exception number defined for the user defined exception while declaring.

### 5.2. SQLERRM

The SQLERRM variable captures the error message defined for the user defined exception while declaring.

### 5.3. SQLSTATE

The SQLSTATE variable is a 5-character code which indicates the return code of a call which accords to ANSI SQL [SQLSTATE](https://en.wikipedia.org/wiki/SQLSTATE) . Snowflake uses additional values beyond those in the ANSI SQL standard.

The below is an example which shows the usage of built-in exception variables in a stored procedure.

```sql
CREATE OR REPLACE PROCEDURE sp_raise_exception(var number)
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
DECLARE
  my_sp_exception1 EXCEPTION (-20001, 'Raised user defined exception MY_SP_EXCEPTION1.');
  my_sp_exception2 EXCEPTION (-20002, 'Raised user defined exception MY_SP_EXCEPTION2.');
BEGIN
  IF (var=0) THEN
    RAISE my_sp_exception1;
  ELSEIF (var=1) THEN
    RAISE my_sp_exception2;
  END IF;
  RETURN var;
EXCEPTION
 WHEN my_sp_exception1 THEN
    RETURN SQLSTATE||':'||SQLCODE||':'||SQLERRM;
 WHEN my_sp_exception2 THEN
    RETURN SQLSTATE||':'||SQLCODE||':'||SQLERRM;
END;
$$
;
```

In the above stored procedure

-   There are two user defined exceptions `my_sp_exception1` and `my_sp_exception2`.
-   If the value of the variable `var`\=0, the `my_sp_exception1` is raised.
-   If the value of the variable `var`\=1, the `my_sp_exception2` is raised.
-   If the value of the variable `var` is other than 0 and 1, the value of `var` is returned.
-   For each exception, the details are captured using the built-in variables in the EXCEPTION block of the stored procedure.

The output of the stored procedure with built-in variables for various inputs is as follows.

```sql
CALL sp_raise_exception(0);
```

![](https://thinketl.com/wp-content/uploads/2023/03/120-4-Exception.png)

```sql
CALL sp_raise_exception(1);
```

![](https://thinketl.com/wp-content/uploads/2023/03/120-5-Exception.png)

```sql
CALL sp_raise_exception(3);
```

![](https://thinketl.com/wp-content/uploads/2023/03/120-6-Exception.png)

Snowflake provides built-in exceptions which are able to define the cause of the error and helps in catching the error. These built-in exceptions can be used along with user defined exceptions in the stored procedures.

The built-in exceptions in the Snowflake stored procedures are as follows

### 6.1. STATEMENT\_ERROR

This exception indicates the error associated with executing a SQL statement. For example, if you perform any operation on a table which does not exist, this exception is raised.

### 6.2. EXPRESSION\_ERROR

This exception indicates an expression-related error. This error is raised, for instance, **dividing by zero** or if you construct an expression that evaluates to a VARCHAR and try to assign its value to a FLOAT.

### 6.3. OTHER

Though this exception do not capture only one particular error, this helps in catching the exceptions that are not specified in the stored procedure.

The below is an example of a stored procedure capturing the error from a delete statement using built-in exception STATEMENT\_ERROR.

```sql
CREATE OR REPLACE PROCEDURE sp_purge_data()
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
BEGIN
  DELETE FROM emp;
EXCEPTION
  WHEN STATEMENT_ERROR THEN
    RETURN 'STATEMENT_ERROR:'||SQLSTATE||':'||SQLCODE||':'||SQLERRM;
END;
$$;
```

The output of the stored procedure is as follows.

![](https://thinketl.com/wp-content/uploads/2023/03/120-7-Exception.png)

The below is an example of a stored procedure capturing the error using built-in exception EXPRESSION\_ERROR.

```sql
CREATE OR REPLACE PROCEDURE sp_expression_error()
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
DECLARE
  var1 FLOAT;
  var2 VARCHAR DEFAULT 'Some text';
BEGIN
  var1 := var2;
  RETURN var1;
EXCEPTION
  WHEN EXPRESSION_ERROR THEN
    RETURN 'STATEMENT_ERROR:'||SQLSTATE||':'||SQLCODE||':'||SQLERRM;
END;
$$;
```

The output of the stored procedure is as follows.

```sql
CALL sp_expression_error();
```

![](https://thinketl.com/wp-content/uploads/2023/03/120-9-Exception.png)

> Note that we have not declared any exception in the above examples and the error information is captured using a built-in exceptions.

In both the above two examples, you can replace the built-in exception with OTHER and it will still capture the error information. Not just the built-in exceptions, the OTHER exception catches any user-defined exception which is declared but not specified in EXCEPTION block.

The below is an example of a stored procedure capturing the error using built-in exception OTHER.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_other()
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
DECLARE
  var1 NUMBER;
BEGIN
  var1 := 1/0;
  RETURN var1;
EXCEPTION
  WHEN OTHER THEN
    RETURN 'OTHER_ERROR:'||SQLSTATE||':'||SQLCODE||':'||SQLERRM;
END;
$$;
```

The output of the stored procedure is as follows.

![](https://thinketl.com/wp-content/uploads/2023/03/120-8-Exception.png)

## 7. Closing Points

More than one exception can be handled using one exception handler using **OR**.

The below exception block shows that same value to be returned for multiple exceptions using OR.

```sql
EXCEPTION
  WHEN MY_EXCEPTION_1 OR MY_EXCEPTION_2 OR MY_EXCEPTION_3 THEN
    RETURN 123;
  WHEN MY_EXCEPTION_4 THEN
    RETURN 4;
  WHEN OTHER THEN
    RETURN 99;
```

The exception handler should be at the end of the block. If the block contains statements after the exception handler, those statements are not executed.

If more than one WHEN clause could match a specific exception, then the first WHEN clause that matches is the one that is executed. The other clauses are not executed.

If you want to raise the exception which you caught in your exception handler, execute RAISE command without any arguments.

```sql
BEGIN
  DELETE FROM emp;
EXCEPTION
  WHEN STATEMENT_ERROR THEN
    LET ERROR_MESSAGE := SQLCODE || ': ' || SQLERRM;
    INSERT INTO error_details VALUES (:ERROR_MESSAGE); -- Capture error details into a table.
    RAISE; -- Raise the same exception that you are handling.
END;
```
