---
created: 2023-09-04T15:04:29 (UTC -04:00)
tags: [if-else,case,conditoinal]
source: https://thinketl.com/if-else-case-statements-in-snowflake-stored-procedures/
author: ThinkETL
---

# IF-ELSE, CASE Statements in Snowflake Stored Procedures


## Introduction

Snowflake Stored Procedures supports following branching constructs in the stored procedure definition.

-   IF ELSE
-   CASE

## IF Statement

**IF statement in Snowflake provides a way to execute a set of statements if a condition is met.**

The following is the syntax to the IF statement in Snowflake.

```sql
IF ( <condition> ) THEN
    <statement>;
ELSEIF ( <condition> ) THEN
    <statement>;
ELSE
    <statement>;
END IF;
```

In an **IF** statement:

- The **ELSEIF** and **ELSE** clauses are optional.
- If an additional condition needs to be evaluated, add statements under **ELSEIF** clause.
- Multiple conditions can be evaluated using multiple **ELSEIF** clauses.
- If none of the provided conditions are true, specify statements to execute in **ELSE** clause.

The following is an example of Snowflake stored procedure calculating the maximum among the three numbers using **IF** statement.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_if(p NUMBER, q NUMBER, r NUMBER)
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
  DECLARE
    var_string VARCHAR DEFAULT 'Maximum Number: ';
  BEGIN
    IF ( p>=q AND p>=r ) THEN
      RETURN var_string || p ;
    ELSEIF ( q>=p AND q>=r ) THEN
      RETURN var_string || q ;
    ELSE
      RETURN var_string || r ;
    END IF;
  END;
$$
;
```

The output of the procedure is as follows

```sql
CALL sp_demo_if(11,425,35);
```


SP_DEMO_IF

Maximum Number: 435

## CASE Statement

**CASE statement in Snowflake lets you define different conditions using WHEN clause and returns a value when first condition is met. This is also referred as Searched CASE statement.**

The following is the syntax to the **CASE** statement in Snowflake.

```sql
CASE
    WHEN <condition 1> THEN
        <statement>;
    WHEN <condition 2> THEN
        <statement>;
    ELSE
        <statement>;
END;
```

In a CASE statement:

-   The conditions specified in the **WHEN** clause are executed in the order they are defined.
-   Whenever a condition is met, the statement configured in the **THEN** clause is executed.
-   If none of the conditions configured in WHEN clauses are met, the statement specified under **ELSE** clause is executed.

The following is an example of Snowflake stored procedure calculating the maximum among the three numbers using **CASE** statement.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_case(p NUMBER, q NUMBER, r NUMBER)
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
  DECLARE
    var_string VARCHAR DEFAULT 'Maximum Number: ';
  BEGIN
    CASE
      WHEN ( p>=q AND p>=r ) THEN
        RETURN var_string || p ;
      WHEN ( q>=p AND q>=r ) THEN
        RETURN var_string || q ;
      ELSE
        RETURN var_string || r ;
    END;
  END;
$$
;
```

The output of the procedure is as follows

```sql
CALL sp_demo_case(11,425,35);
```

 SP_DEMO_CASE
 
 Maximum Number: 425


## Simple CASE Statement

**A Simple CASE Statement allows you to define a single condition and all the possible output values of defined condition under different branches using WHEN clause.**

The following is the syntax to the Simple CASE statement in Snowflake.

```sql
CASE <condition>
    WHEN <value 1> THEN
        <statement>;
    WHEN <value 2> THEN
        <statement>;
    ELSE
        <statement>;
END;
```

In a **Simple CASE** statement:

- The condition expression is defined only once in **CASE** statement.
- All the possible values of the expression are defined in different **WHEN** clauses.
- Whenever a value defined in a **WHEN** clause is a match, then the statement configured in the **THEN** clause is executed.
- If none of the values configured in **WHEN** clauses are a match, the statements specified under **ELSE** clause are executed.

The following is an example of Snowflake Stored Procedure fetching the currency of a country using **Searched CASE** statement.

Note that the same condition expression is defined in every **WHEN** clause of the **CASE** statment in the example below.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_searched_case(v_country VARCHAR)
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
---------------------Example of Searched Case Statement---------------------
  DECLARE
    v_output VARCHAR DEFAULT 'The currency of country '|| UPPER(v_country) || ' is ';
  BEGIN
    CASE
      WHEN UPPER(v_country) = 'INDIA' THEN
          RETURN v_output || 'RUPEE';
      WHEN UPPER(v_country) = 'USA' THEN
          RETURN v_output || 'DOLLAR';
      WHEN UPPER(v_country) = 'UK' THEN
          RETURN v_output || 'POUND';
      ELSE
          RETURN 'The country '|| v_country || ' is not defined in procedure';
    END;
  END;
$$
;
```

The output of the procedure with **Searched CASE** statement is as follows.

```sql
CALL sp_demo_searched_case('India');
```


|SP_DEMO_SEARCHED_CASE|
|-|
|The currency of country INDIA is RUPEE|

The following is the same procedure built using the **Simple CASE** statement.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_simple_case(v_country VARCHAR)
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
---------------------Example of Simple Case Statement---------------------
  DECLARE
    v_output VARCHAR DEFAULT 'The currency of country '|| UPPER(v_country) || ' is ';
  BEGIN
    CASE( UPPER(v_country) )
      WHEN 'INDIA' THEN
          RETURN v_output || 'RUPEE';
      WHEN 'USA' THEN
          RETURN v_output || 'DOLLAR';
      WHEN 'UK' THEN
          RETURN v_output || 'POUND';
      ELSE
          RETURN 'The country '|| v_country || ' is not defined in procedure';
    END;
  END;
$$
;
```

The output of the procedure with **Simple CASE** statement is as follows.

```sql
CALL sp_demo_simple_case('USA');
```


|SP_DEMO_SIMPLE_CASE|
|-|
|The currency of country USA is DOLLAR|
