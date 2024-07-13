---
created: 2023-09-04T15:07:10 (UTC -04:00)
tags: [snowflake, storedproc, looping]
source: https://thinketl.com/looping-in-snowflake-stored-procedures/
updated: 2024-04-21 13:07:07
---

# Looping in Snowflake Stored Procedures

## 1. Loops

The following are the different type of Loops supported in Snowflake Stored Procedures.

- FOR
- WHILE
- REPEAT
- LOOP

In this article let us discuss about the different loops in Snowflake Stored Procedures with examples.

## 2. For Loop

**A FOR loop enables a particular set of steps to be executed for a specified number of times until a condition is satisfied.**

The following is the syntax of FOR Loop in Snowflake Stored Procedures.

```sql
FOR <counter_variable> IN [ REVERSE ] <start> TO <end> { DO | LOOP }
    <statement>;
END { FOR | LOOP } [ <label> ] ;
```

The keyword **DO** should be paired with **END FOR** and the keyword **LOOP** should be paired with **END LOOP**. For example:

```sql
FOR...DO
  ...
END FOR;

FOR...LOOP
  ...
END LOOP;
```

In FOR Loop:

-   A **<counter\_variable>** loops from the values defined for **<start>** till the value defined for **<end>** in the syntax.
-   Note that if a variable with the same name as **<counter\_variable>** is declared outside the loop, the outer variable and the loop variable are independent.
-   Use **REVERSE** keyword to loop the values starting from **<end>** till **<start>.**
-   If there are multiple loops defined in the procedure, use the **<label>** to identify loops individually. This also helps to jump loops using BREAK and CONTINUE statements.

The following is an example of Snowflake Stored Procedure which calculates the sum of first n numbers using FOR Loop.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_for_loop(n NUMBER)
  RETURNS NUMBER
  LANGUAGE SQL
AS
$$
  DECLARE
    total_sum INTEGER DEFAULT 0;
  BEGIN
    FOR i IN 1 TO n DO
      total_sum := total_sum + i ;
    END FOR;
    RETURN total_sum;
  END;
$$
;
```

The output of the procedure with **FOR** Loop is as follows.

```sql
CALL sp_demo_for_loop(5);
```


SP_DEMO_FOR_LOOP
15


The following is an example of Snowflake Stored Procedure which prints the numbers in backwards using REVERSE keyword in FOR Loop.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_reverse_for_loop(n NUMBER)
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
  DECLARE
    reverse_series VARCHAR DEFAULT '';
  BEGIN
    FOR i IN REVERSE 1 TO n LOOP
      reverse_series := reverse_series ||' '|| i::VARCHAR ;
    END LOOP;
    RETURN reverse_series;
  END;
$$
;
```

The output of the procedure with **REVERSE** keyword in **FOR** Loop is as follows.

```sql
CALL sp_demo_reverse_for_loop(5);
```


SP_DEMO_REVERSE_FOR_LOOP
5 4 3 2 1


## 2. WHILE Loop

**A WHILE loop iterates _while_ a specified condition is true. The condition for the loop is tested immediately before executing the body of the loop in WHILE loop. If the condition is false, the loop is not executed even once.**

The following is the syntax of **WHILE** Loop in Snowflake Stored Procedures.

```sql
WHILE ( <condition> ) { DO | LOOP }
  <statement>;
END { WHILE | LOOP } [ <label> ] ;
```

In a WHILE Loop:

-   The **<condition>** is an expression that evaluates to a BOOLEAN.
-   The keyword **DO** should be paired with **END WHILE** and the keyword **LOOP** should be paired with **END LOOP**.
-   If there are multiple loops defined in the procedure, use the **<label>** to identify loops individually. This also helps to jump loops using BREAK and CONTINUE statements.

> _**Note that if the <condition> never evaluates to FALSE, and the loop does not contain a BREAK command (or equivalent), then the loop will run and consume credits indefinitely.**_

The following is an example of Snowflake Stored Procedure which calculates the sum of first n numbers using **WHILE** Loop.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_while_loop(n NUMBER)
  RETURNS NUMBER
  LANGUAGE SQL
AS
$$
  DECLARE
    total_sum INTEGER DEFAULT 0;
  BEGIN
    LET counter := 1;
    WHILE (counter <= n) DO
      total_sum := total_sum + counter;
      counter := counter + 1;
    END WHILE;
    RETURN total_sum;
  END;
$$
;
```

The output of the procedure with **WHILE** Loop is as follows.

```sql
CALL sp_demo_while_loop(6);
```


SP_DEMO_WHILE_LOOP
21



## 3. REPEAT Loop 

**A REPEAT loop iterates _until_ a specified condition is true. This is similar to DO WHILE loop in other programming languages which tests the condition at the end of the loop. This means that the body of a REPEAT loop always executes at least once.**

The following is the syntax of **REPEAT** Loop in Snowflake Stored Procedures.

```sql
REPEAT
    <statement>;
UNTIL ( <condition> )
END REPEAT [ <label> ] ;
```

In a **REPEAT** Loop:

-   The **<condition>** is an expression that evaluates to a BOOLEAN.
-   The **<condition>** is evaluated at the end of the loop and is defined using **UNTIL** keyword.

The following is an example of Snowflake Stored Procedure which calculates the sum of first n numbers using **REPEAT** Loop.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_repeat(n NUMBER)
  RETURNS NUMBER
  LANGUAGE SQL
AS
$$
  DECLARE
    total_sum INTEGER DEFAULT 0;
  BEGIN
    LET counter := 1;
    REPEAT
      total_sum := total_sum + counter;
      counter := counter + 1;
    UNTIL(counter > n)
    END REPEAT;
    RETURN total_sum;
  END;
$$
;
```

The output of the procedure with **REPEAT** Loop is as follows.


SP_DEMO_REPEAT
28


## 4. LOOP Loop 

**A LOOP loop executes until a BREAK command is executed. It does not specify a number of iterations or a terminating condition.**

The following is the syntax of **LOOP** Loop in Snowflake Stored Procedures.

```sql
LOOP
    <statement>;
END LOOP [ <label> ] ;
```

In a **LOOP** Loop:

-   The user must explicitly exit the loop by using **BREAK** command in the loop.
-   The **BREAK** command is normally embedded inside branching logic (e.g. [**IF**](https://thinketl.com/if-else-case-statements-in-snowflake-stored-procedures/#IF_Statement) Statements or **[CASE](https://thinketl.com/if-else-case-statements-in-snowflake-stored-procedures/#CASE_Statement)** Statements).
-   The **BREAK** command immediately stops the current iteration, and skips any remaining iterations.

The following is an example of Snowflake Stored Procedure which calculates the sum of first n numbers using **LOOP** Loop.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_loop(n NUMBER)
  RETURNS NUMBER
  LANGUAGE SQL
AS
$$
  DECLARE
    total_sum INTEGER DEFAULT 0;
  BEGIN
    LET counter := 1;
    LOOP
      IF(counter > n) THEN
        BREAK;
      END IF;
      total_sum := total_sum + counter;
      counter := counter + 1;
    END LOOP;
    RETURN total_sum;
  END;
$$
;
```

The output of the procedure with **LOOP** Loop is as follows.
SP_DEMO_LOOP
36


