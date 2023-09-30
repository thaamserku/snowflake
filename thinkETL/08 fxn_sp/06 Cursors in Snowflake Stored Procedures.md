---
created: 2023-09-04T15:08:32 (UTC -04:00)
tags: [storedproc, cursors, snowflake]
source: https://thinketl.com/cursors-in-snowflake-stored-procedures/
author: ThinkETL
---

# Cursors in Snowflake Stored Procedures


**A Cursor is a named object in a stored procedure which allows you to loop through a set of rows of a query result set, one row at a time. It allows to perform same set of defined actions for each row individually while looping through a result of a SQL query.**

Working with Cursors in Snowflake Stored Procedures includes following steps

1.  Declaring a cursor either in **DECLARE** or **BEGIN…END** section of the stored procedure.
2.  Opening a cursor using **OPEN** command.
3.  Fetching rows from cursors using **FETCH** command.
4.  Closing a cursor using **CLOSE** command.

## **2. Syntax of Cursors in Snowflake Stored Procedures**

### **2.1. Declaring a Cursor**

**A Cursor must be declared before using it. Declaring a Cursor defines the cursor with a name and the associated SELECT statement.**

The syntax for declaring a **CURSOR** in **DECLARE** section of the procedure is as follows.

```sql
DECLARE
  <cursor_name> CURSOR FOR <select_statement>;

-- Example:
DECLARE
  my_cursor CURSOR FOR SELECT id, firstname FROM employees;
```

The syntax for declaring a **CURSOR** in **BEGIN…END** section of the procedure is as follows.

```sql
BEGIN
  …
  LET <cursor_name> CURSOR FOR <select_statement>;
  …
END;

-- Example:
BEGIN
  …
  LET my_cursor CURSOR FOR SELECT id, firstname FROM employees;
  …
END;
```

### **2.2. Opening a Cursor**

**The cursor must be explicitly opened before fetching rows from it using OPEN command. The query associated with cursor is not executed until it is opened.**

The syntax to **OPEN** a **CURSOR** in stored procedure is as follows.

```sql
OPEN <cursor_name>;

-- Example:
BEGIN
  OPEN my_cursor;
  …
END;
```

### **2.3. Fetching data from Cursor**

**The FETCH command retrieves row by row from the result set of the query associated with cursor. Each FETCH command that you execute fetches a single row and increments the internal counter to next row.**

As a result the FETCH command must be executed multiple times until last row is fetched using looping commands in stored procedures. If a FETCH command is executed after all rows are fetched, it retrieves null values.

The syntax to **FETCH** data from **CURSOR** in stored procedures is as follows.

```sql
FETCH <cursor_name> INTO <variable_1>,<variable_2>,…;

-- Example:
BEGIN
  …
  FETCH my_cursor INTO my_variable_1, my_variable_1;
  …
END;
```

### **2.4. Closing a Cursor**

**The cursor must be closed once all rows are fetched using the CLOSE command.**

The syntax to **CLOSE** a **CURSOR** in stored procedures is as follows.

```sql
CLOSE <cursor_name>;

-- Example:
BEGIN
  …
  CLOSE my_cursor;
END;
```

## **3. Setting up a query for Cursor demonstration**

The following **SELECT** query fetches all the tables present in **PUBLIC** schema of **DEMO_DB** database. The query uses **INFORMATION_SCHEMA** which is a data dictionary schema available under each database.

```sql
SELECT table_name, table_type
FROM demo_db.information_schema.tables
WHERE table_schema = 'PUBLIC' AND table_type= 'BASE TABLE'
ORDER BY table_name;
```

![Output of query providing list of tables present in PUBLIC schema](https://thinketl.com/wp-content/uploads/2023/03/118-1-Get-all-tables-list.png)

Output of query providing list of tables present in PUBLIC schema

Let us use this query as an example and fetch the details of all tables using CURSOR.

## **4. Cursors using OPEN, FETCH and CLOSE**

The following stored procedure fetches details of all the tables present in the PUBLIC schema and lists them as output using Cursors.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_cursor()
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
DECLARE
  table_cursor CURSOR FOR
    SELECT table_name FROM demo_db.information_schema.tables
    WHERE table_schema = 'PUBLIC' AND table_type= 'BASE TABLE' ORDER BY table_name;
  res VARCHAR DEFAULT '';
  var_table_name VARCHAR;
BEGIN
  OPEN table_cursor;
  LOOP
    FETCH table_cursor into var_table_name;
    IF(var_table_name <> '') THEN
      res := var_table_name ||' '||res;
    ELSE
      BREAK;
    END IF;
  END LOOP;
  CLOSE table_cursor;
  RETURN res;
END;
$$;
```

In the above example:

-   In the line **7**, the cursor is defined in the DECLARE section of the stored procedure. The cursor is named as `table_cursor` and the query discussed in section-3 is associated with cursor.
-   In the line **13**, the cursor is opened using OPEN command.
-   From line **14-21**, LOOP command is used to loop until all records are fetched from the cursor.
-   In the line **15**, the FETCH command fetches the `table_name` from the result set of cursor into variable `var_table_name`.
-   From line **16-19**, IF-ELSE clause is used to concatenate the table name values read from the variable `var_table_name` as long as the variable value is not null. Once all the values in the result set are fetched, the logic to explicitly exit the loop using the BREAK command is embedded in the ELSE clause.
-   In the line **22**, the cursor `table_cursor` is closed.
-   In the line **23**, the final concatenated value of all table names is returned as output.

The output of the stored procedure `sp_demo_cursor` is as follows.

<table><tbody><tr><td><strong>SP_DEMO_CURSOR</strong></td></tr><tr><td>LOCATIONS EMPLOYEES DEPARTMENTS</td></tr></tbody></table>

## **5. Cursors using FOR Loop command**

**A FOR loop can also be used to iterate over a result set of a Cursor instead of FETCH command. The number of iterations of the FOR loop is determined by the number of rows in the result set of cursor.**

> Note that when using a FOR loop to iterate the cursor, the cursor need not be opened explicitly using OPEN command.

The syntax of **Cursor-based FOR loops** is as follows.

```sql
FOR <row_variable> IN <cursor_name> DO
  <statement>;
END FOR [ <label> ] ;
```

The `<row_variable>` holds data of all the columns of the row it is iterating. The individual columns can be accessed as below

```sql
<row_variable>.<column1>, <row_variable>.<column2>, ...
```

The following stored procedure fetches details of all the tables present in the PUBLIC schema and lists them as output using **Cursor-based FOR loops**.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_cursor_using_for()
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
DECLARE
  table_cursor CURSOR FOR
    SELECT table_name, table_type FROM demo_db.information_schema.tables
    WHERE table_schema = 'PUBLIC' and table_type= 'BASE TABLE' ORDER BY table_name;
   res VARCHAR DEFAULT '';
BEGIN
  FOR var in table_cursor DO
    res := var.table_name||' '||res;
  END FOR;
  RETURN res;
END;
$$
;
```

In the above example:

-   From line **12-14**, the FOR loop is used iterate over the result set of the cursor `table_cursor`.
-   In the line **12**, the row variable `var` is used to hold the result set values of the query associated with cursor.
-   In the line **13**, the `table_name` field from the result set is accessed using the row variable as `var.table_name`_._
-   In the line **15**, Once all rows in the result set are iterated, the FOR loop exits and the final concatenated value of all table names is returned.

The output of the stored procedure `sp_demo_cursor_using_for` is as follows.

```sql
CALL sp_demo_cursor_using_for();
```

<table><tbody><tr><td><strong>SP_DEMO_CURSOR_USING_FOR</strong></td></tr><tr><td>LOCATIONS EMPLOYEES DEPARTMENTS</td></tr></tbody></table>

## **6. Cursors using RESULTSET**

**Instead of assigning the SELECT query directly to the CURSOR, you can assign a variable of type RESULTSET which holds the query.**

The syntax to assign SELECT query to **CURSOR** using **RESULTSET** is as follows.

```
BEGIN
  LET <variable_name> RESULTSET := (<select_query>);
  LET <cursor_name> CURSOR FOR <variable_name>;
  …
END;
```

The following stored procedure fetches details of all the tables present in the PUBLIC schema and lists them as output using RESULTSET to pass query to CURSOR.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_cursor_using_resultset()
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
DECLARE
  res VARCHAR DEFAULT '';
BEGIN
  LET res_set RESULTSET := ( SELECT table_name, table_type FROM demo_db.information_schema.tables
    WHERE table_schema = 'PUBLIC' and table_type= 'BASE TABLE' ORDER BY table_name);
  LET table_cursor CURSOR FOR res_set;
  FOR var in table_cursor DO
    res := var.table_name||' '||res;
  END FOR;
  RETURN res;
END;
$$
;
```

In the above example:

-   In the line **9**, variable of type RESULTSET named `res_set` is assigned with the query is to fetch the list of all table names.
-   In the line **11**, the cursor is assigned with the `res_set` variable instead of query.
-   The rest of the procedure is same as the example discussed in earlier section.

The output of the stored procedure `sp_demo_cursor_using_resultset` is as follows.

```sql
CALL sp_demo_cursor_using_for();
```

<table><tbody><tr><td><strong>SP_DEMO_CURSOR_USING_RESULTSET</strong></td></tr><tr><td>LOCATIONS EMPLOYEES DEPARTMENTS</td></tr></tbody></table>

## **7. Cursors using USING clause to pass Bind variables**

**In the SELECT statement, we can pass bind variables to which values can be passed while opening the cursor in the `USING` clause of `OPEN` command.**

The following stored procedure fetches details of all the tables present in a schema whose value is passed as a bind variable to the cursor query.

```sql
CREATE OR REPLACE PROCEDURE sp_demo_cursor_using_bindvariables(var_schema VARCHAR)
  RETURNS VARCHAR
  LANGUAGE SQL
AS
$$
DECLARE
  table_cursor CURSOR FOR
    SELECT table_name, table_type FROM demo_db.information_schema.tables
    WHERE table_schema = ? AND table_type= 'BASE TABLE' ORDER BY table_name;
  res VARCHAR DEFAULT '';
  var_table_name VARCHAR;
BEGIN
  OPEN table_cursor using (var_schema);
  LOOP
    FETCH table_cursor into var_table_name;
    IF(var_table_name <> '') THEN
      res := var_table_name ||' '||res;
    ELSE
      BREAK;
    END IF;
  END LOOP;
  CLOSE table_cursor;
  RETURN res;
END;
$$
;
```

In the above example:

-   In the line **1**, the stored procedure is expected to be called by passing value to the input variable `var_schema` which holds the schema name.
-   In the line **9**, you could see that the schema is passed as bind variable (`**?**`) in the query assigned to the cursor.
-   In the line **13**, the value to the bind variable is passed through `USING` clause of `OPEN` command.
-   The rest of the procedure is same as what we discussed in the section-3.

The stored procedure `sp_demo_cursor_using_bindvariables` is executed as below.

```sql
CALL sp_demo_cursor_using_bindvariables('PUBLIC');
```

The output of the stored procedure `sp_demo_cursor_using_bindvariables` is as follows.

<table><tbody><tr><td><strong>SP_DEMO_CURSOR_USING_BINDVARIABLES</strong></td></tr><tr><td>LOCATIONS EMPLOYEES DEPARTMENTS</td></tr></tbody></table>

## **8. Real-time scenario using a Cursor in Stored Procedures**

In all the above examples, we have looped through a SELECT query which lists the names of all tables present in a schema. You might be wondering do we even need cursors to achieve this output.
Well, definitely not !!!

> These examples are designed to make you understand about the concept of the cursors. But in real-time, every time you extract the row from a result set of a cursor, an action would be associated with it.

Those actions could be executing any DML statements like UPDATE, DELETE etc.. or calling another stored procedure based on the logic.

So let us also end this article with a real-time useful scenario. In all the above examples we have just listed the table names. Let us extend the same example by **extracting the DDL of each table** and provide them as output.

The following stored procedure takes database and schema names as input and fetches the DDL of all the tables present inside them as output using Cursors.

```sql
CREATE OR REPLACE PROCEDURE sp_get_ddl( var_db_name VARCHAR, var_schema_name VARCHAR)
RETURNS TABLE(DDL VARCHAR)
LANGUAGE SQL
AS
$$
DECLARE
  cursor_sql VARCHAR DEFAULT 'SELECT table_name, table_type FROM '||var_db_name||'.information_schema.tables
    WHERE table_schema = '''||var_schema_name||''' and table_type= ''BASE TABLE'' ORDER BY table_name';
  cursor_resultset RESULTSET DEFAULT (EXECUTE IMMEDIATE :cursor_sql);
  table_cursor CURSOR FOR cursor_resultset;
  my_sql VARCHAR;
  my_union_sql VARCHAR;
  res RESULTSET;
  counter NUMBER DEFAULT 1;
BEGIN
  FOR var in table_cursor DO
    my_sql := 'SELECT GET_DDL(''TABLE'','''||var.table_name||''')';
    IF(counter=1) THEN
      my_union_sql := :my_sql;
    ELSE
      my_union_sql := :my_union_sql || ' UNION ALL ' || :my_sql;
    END IF;
    counter := counter + 1;
  END FOR;
  res := (EXECUTE IMMEDIATE :my_union_sql);
  RETURN table(res);
END;
$$
;
```

In the above example:

-   In the line **1**, the stored procedure `sp_get_ddl` accepts two inputs as variables `var_db_name` and `var_schema_name`, the database and the schema name respectively from which you wanted to extract the DDL of tables.
-   In the line **2**, the RETURN TYPE of the procedure is of type TABLE since we output a bunch of rows each with the DDL of a table.
-   In the line **7**, we created variable `cursor_sql` of type VARCHAR which holds the SQL to be assigned to cursor as an expression.

> The reason for passing SELECT statement as a string expression is that we are passing database name also as a variable in the query which cannot be passed as a bind variable since it is not used as a filter value in the query, but the part of the query itself.

-   In the line **9**, we are executing the SQL string expression using EXECUTE IMMEDIATE and assigning it to a RESULTSET `cursor_result_set`.
-   In the line **10**, finally the cursor `table_cursor` is declared and assigned a result set `cursor_result_set` which holds the query.
-   In the line **16**, we are using the FOR command to loop through the result set of the cursor.
-   In the line **17**, we are creating another SQL string expression `my_sql` which provides the DDL of a table.

> Since we cannot provide an output in each loop we are performing in Snowflake Stored Procedures, we have to concatenate queries using UNION ALL and form a single query at the end of the loop. The final query which holds the statement to provide DDL of all tables is executed outside the loop and provided as output.

-   In the line **18-19**, when the counter is 1, we assign the SQL string expression `my_sql` as value to the variable `my_union_sql`. The counter is also incremented at the end of iteration.
-   In the line **20-21**, after the initial iteration all the conditions go to ELSE clause and the SQL expression is concatenated to previous values using UNION ALL.
-   In the line **25**, the output of the final query present in the `my_union_sql` is executed and assigned to a variable of type RESULTSET `res`.
-   In the line **26**, the variable `res` is returned as an output in the form of TABLE.

The stored procedure `sp_get_ddl`  is executed as below with database name as `DEMO_DB` and schema name as `PUBLIC`.

```sql
CALL sp_get_ddl('DEMO_DB','PUBLIC');
```

The output of the stored procedure `sp_get_ddl` is as follows.

![Output of procedure sp_get_ddl providing DDL of all tables](https://thinketl.com/wp-content/uploads/2023/03/118-2-get-ddl.png)

Output of procedure sp_get_ddl providing DDL of all tables
