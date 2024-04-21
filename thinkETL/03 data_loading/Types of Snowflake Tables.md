---
created: 2023-09-24T12:23:14 (UTC -04:00)
tags: [snowflake, table]
source: https://thinketl.com/types-of-snowflake-tables/
updated: 2024-04-21 12:13:57
---

# Types of Snowflake Tables


> Snowflake supports three types of tables namely Permanent, Temporary and Transient tables.


Tables are database objects logically structured as a collection of rows and columns. All data in Snowflake is stored in database tables. Apart from standard database tables, Snowflake supports other table types that are especially useful for storing data that does not need to be maintained for extended periods of time.

Snowflake supports three types of tables

-   Permanent table
-   Transient table
-   Temporary table

## **Permanent Table**

**These are the standard, regular database tables. Permanent tables are the default table type in Snowflake and do not need any additional syntax while creating to make them permanent.**

The data stored in permanent tables consumes space and contributes to the storage charges that Snowflake bills your account. It also comes with additional features like [**Time-Travel**](https://thinketl.com/types-of-snowflake-tables/#What_is_Snowflake_Time-Travel) and **[Fail-Safe](https://thinketl.com/types-of-snowflake-tables/#What_is_Snowflake_Fail-Safe_period)** which helps in data availability and recovery.

To create a Permanent table in Snowflake

```sql
CREATE TABLE EMPLOYEE (ID NUMBER, NAME VARCHAR(50));
```

## **Transient Table**

**Transient tables in Snowflake are similar to permanent tables except that that they do not have a Fail-safe period and only have a very limited Time-Travel period. These are best suited in scenarios where the data in your table is not critical and can be recovered from external means if required.**

Transient tables, like permanent tables, contribute to your account’s overall storage expenses. However, since Transient Tables do not use Fail-safe, there are no Fail-safe costs (i.e. the costs associated with maintaining the data required for Fail-safe disaster recovery).

To create a Transient table in Snowflake, You need to mention **_transient_** in the create table syntax.

```sql
CREATE TRANSIENT TABLE EMPLOYEE (ID NUMBER, NAME VARCHAR(50));
```

## **Temporary Table**

**Temporary Tables in Snowflake exist only within the session in which they were created and available only for the remainder of the session. They are not visible to other users or sessions. Once the session ends, data stored in the table is dropped completely and is not recoverable.**

Like Transient tables, temporary tables do not have a Fail-safe period and have a very limited Time-Travel period. These are best suited for storing non-permanent, transitory data which lasts only during the session they were created.

Though Temporary tables are dropped at the end of the session, Snowflake recommends explicitly dropping these tables once they are no longer needed to prevent any unexpected storage changes when working with large temporary tables.

To create a Temporary table in Snowflake, you need to mention **_temporary_** in the create table syntax.

```sql
CREATE TEMPORARY TABLE EMPLOYEE (ID NUMBER, NAME VARCHAR(50));
```

## **Naming Conflicts** **between temporary and non-temporary tables**

Snowflake supports creating a temporary table that has the same name as an existing permanent/transient table in the same schema. However, note that the temporary table takes precedence in the session over any other table with the same name in the same schema.

**When you create a temporary table that has the same name as an existing table in the same schema**

-   Temporary table take precedence and hides the existing non-temporary table.

**When you create a table that has the same name as an existing temporary table in the same schema**

-   The newly-created table is hidden by the temporary table.

All queries and other operations performed in the session on the table effect only the temporary table.

## **Comparison of Snowflake Table Types**

The below table summarizes the differences between the three table types, particularly with regard to their impact on Time Travel and Fail-safe:

<table><tbody><tr><td><strong>Type</strong></td><td><strong>Availability</strong></td><td><strong>Time-Travel Retention period in days</strong></td><td><strong>Fail-Safe period in days</strong></td></tr><tr><td>Temporary</td><td>Remainder of session</td><td>0 or 1 (default is 1)</td><td>0</td></tr><tr><td>Transient</td><td>Until explicitly dropped</td><td>0 or 1 (default is 1)</td><td>0</td></tr><tr><td>Permanent( Standard Edition)</td><td>Until explicitly dropped</td><td>0 or 1 (default is 1)</td><td>7</td></tr><tr><td>Permanent( Enterprise and higher Edition)</td><td>Until explicitly dropped</td><td>0 to 90 (default is configurable)</td><td>7</td></tr></tbody></table>

## **How to find what type a Snowflake table is?**

Below are some of the ways you can find what type a Snowflake is

### **1. SHOW TABLES**

Lists the tables for which you have access privileges, including dropped tables that are still within the Time Travel retention period and, therefore, can be undropped.

Syntax to execute show tables in Snowflake is as below

To get table details from a particular database and Schema

```sql
SHOW TABLES IN <DATABASE_NAME>.<SCHEMA_NAME>;
```

To show all tables that start with ‘dim’

```sql
SHOW TABLES LIKE 'DIM%' IN <DATABASE_NAME>.<SCHEMA_NAME>;
```

![show tables in Snowflake](https://thinketl.com/wp-content/uploads/2022/02/69-1-Show-tables.png)

> _The table type can be obtained from the field **kind** from the output._

### **2. GET_DDL**

Returns a DDL statement that can be used to recreate the specified object. The DDL statement contains the type of the Snowflake table.

Syntax to get DDL of a table

```sql
SELECT GET_DDL('TABLE','<TABLE_NAME>');
```

![get_ddl in Snowflake](https://thinketl.com/wp-content/uploads/2022/02/69-2-get-ddl.png)

### **3. Information Schema**

The Snowflake Information Schema (aka “Data Dictionary”) consists of a set of system-defined views and table functions that provide extensive metadata information about the objects created in your account.

Syntax to use Information Schema

```sql
SELECT * FROM <DATABASE>.INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = '<TABLE_NAME>';
```

![Information Schema in Snowflake](https://thinketl.com/wp-content/uploads/2022/02/69-information-schema.png)

> _Information Schema do not show temporary tables._

## **What is Snowflake Time-Travel?**

Snowflake Time Travel enables accessing historical data that has been changed or deleted at any point within a defined period. If you accidentally dropped a table or some of the records in a table, they can be recovered back with in the configured time-travel period of the table.

This requires a separate discussion in another article to understand how Time-Travel works and how to recover data using time-travel.

**Read complete Article: [Snowflake Time Travel](https://thinketl.com/overview-of-snowflake-time-travel/)**

## **What is Snowflake Fail-Safe period?**

Fail-safe provides a (non-configurable) 7-day period during which historical data may be recoverable by Snowflake. This period starts immediately after the Time Travel retention period ends.

Fail-safe is **not** provided as a means for accessing historical data after the Time Travel retention period has ended. It is for use only by Snowflake to recover data that may have been lost or damaged due to extreme operational failures.

> Fail-Safe requires intervention by Snowflake Support team to restore data.

## **Snowflake External Tables**

You might be wondering what this external table is or why this external table category is not added along with Permanent, Temporary and Transient tables. These are not the typical type of tables that can be created directly in Snowflake.

Snowflake External tables allow you to access the data from files stored in external stage such as Amazon S3, Azure blob storage or GCP bucket as a regular table. This allows you to query a file as if it is a table in Snowflake.

Again this requires a separate discussion in another article on how to work with Snowflake external tables. But it is good to know that these tables do exist in Snowflake.

