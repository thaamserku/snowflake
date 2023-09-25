---
created: 2023-09-07T21:05:02 (UTC -04:00)
tags: []
source: https://thinketl.com/how-to-get-ddl-of-database-objects-in-snowflake/
author: ThinkETL
---

# HOW TO: Get DDL of database objects in Snowflake? 
> [!Excerpt]
> Snowflake provides GET_DDL Function using which DDL of database objects like tables, views, procedures etc., can be extracted.

---
## **GET\_DDL Function**

Snowflake provides **GET\_DDL** Function using which DDL of database objects can be extracted. The GET\_DDL function returns a DDL statement as output that can be used to create the database objects. 

GET\_DDL function takes object type and object name as input to generate the DDL of the required object.

## **Syntax of GET\_DDL Function**

The syntax of GET\_DDL function in snowflake is as shown below

```sql
GET_DDL( '<object_type>' , '[<namespace>.]<object_name>' )
```

The **object\_type** argument specifies the type of the object for which DDL is required. Below are the valid object types that can be passed to the GET\_DDL function.

-   DATABASE
-   FILE\_FORMAT
-   FUNCTION (for UDFs, including external functions)
-   PIPE
-   POLICY (masking and row access policies)
-   PROCEDURE (for stored procedures)
-   SCHEMA
-   SEQUENCE
-   STREAM
-   TABLE (including for external tables)
-   TAG (object tagging)
-   TASK
-   VIEW (including for materialized views)

The **object\_name** argument specifies the name of the object for which DDL is required.

The **namespace** argument is the database and/or schema in which the object resides which is **optional** if database and schema are in use in current session in the worksheet.

Trigger the GET\_DDL function to extract the DDL of a Snowflake object as shown below.

```sql
SELECT GET_DDL('<object_type>' , '[<namespace>.]<object_name>');
```

The below example extracts the DDL of a table named **CUSTOMER**.

```sql
SELECT GET_DDL('table', 'customer');
```

If the database and schema are not in use, you can pass the complete namespace to extract DDL as shown below.

```sql
 SELECT GET_DDL('table', 'mydb.sales.customer');
```

For databases and schemas, GET\_DDL is recursive i.e. it returns the DDL statements of all supported objects within the specified database/schema.

The below example extracts the DDL of all supported objects present in the database **my\_db**.

```sql
SELECT GET_DDL('database', 'my_db');
```

The below example extracts the DDL of all supported objects present in the schema **my\_schema**.

```sql
SELECT GET_DDL('schema', 'my_db.my_schema');
```

## **GET\_DDL Output**

By default the DDL statement returned by GET\_DDL function do not use a fully-qualified named of the object.

The below example shows the output of query providing the DDL of customer table without database and schema information.

![DDL of Customer without fully-qualified name](https://thinketl.com/wp-content/uploads/2023/01/106-1-GET_DDL-output.png)

DDL of Customer without fully-qualified name

To return the DDL with fully-qualified named, use the **true** indicator in the syntax of GET\_DDL function as shown below.

```sql
GET_DDL( '<object_type>' , '[<namespace>.]<object_name>', true );
```

The below example shows the output of query providing the DDL of customer table with a fully-qualified named providing the database and schema information.

![DDL of Customer with fully-qualified name](https://thinketl.com/wp-content/uploads/2023/01/106-2-GET_DDL-with-fully-qualified-name.png)

DDL of Customer with fully-qualified name

