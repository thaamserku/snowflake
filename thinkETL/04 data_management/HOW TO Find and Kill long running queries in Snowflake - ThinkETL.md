---
created: 2023-09-07T21:05:15 (UTC -04:00)
tags: []
source: https://thinketl.com/how-to-find-and-kill-long-running-queries-in-snowflake/
author: ThinkETL
---

# HOW TO: Find and Kill long running queries in Snowflake? 


> Learn how to find and kill long running queries in Snowflake using the QUERY_HISTORY table functions available under Information Schema.


## **1. Introduction**

The Snowflake **[INFORMATION_SCHEMA](https://thinketl.com/snowflake-information-schema/)** consists of a set of system-defined views and **table functions** that provide extensive metadata information about the objects created in your account.

There is a table function that is made available under Snowflake information schema which provides historical information of queries triggered which helps in finding and killing the long running queries.

## **2. QUERY_HISTORY table function**

The **QUERY_HISTORY** table function can be used to query Snowflake query history along various dimensions. The functions returns query activity within the last 7 days.

There is a family of table functions available under QUERY_HISTORY

-   **`QUERY_HISTORY`**: Returns queries within a specified time range.
-   **`QUERY_HISTORY_BY_SESSION`**:  Returns queries within a specified session and time range.
-   **`QUERY_HISTORY_BY_USER`**:  Returns queries submitted by a specified user within a specified time range.
-   **`QUERY_HISTORY_BY_WAREHOUSE`**:  Returns queries executed by a specified warehouse within a specified time range.

## **3. Retrieving query history using QUERY_HISTORY table function**

The below query retrieves up to the last 100 queries run by the current user or run by any user on any warehouse on which the current user has the MONITOR privilege.

```sql
SELECT *
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
ORDER BY START_TIME;
```

## **4. Finding long queries within a time range**

### **4.1. Without arguments in table function**

The below query retrieves the long running queries which ran more than 5 minutes in last 1 hour.

```sql
SELECT
    QUERY_ID,
    QUERY_TEXT,
    USER_NAME,
    WAREHOUSE_NAME,
    START_TIME,
    END_TIME,
    DATEDIFF(SECOND, START_TIME, END_TIME) AS RUN_TIME_IN_SECONDS
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
WHERE DATEDIFF(MINUTE, START_TIME, END_TIME) > 5
AND START_TIME > DATEADD(HOUR, -1, CURRENT_TIMESTAMP())
ORDER BY START_TIME;
```

### **4.2. With arguments in table function**

Alternatively the above query can also written by passing arguments (**`END_TIME_RANGE_START`**, **`END_TIME_RANGE_END`**)  to the **`QUERY_HISTORY`** table function as shown below.

```sql
SELECT
    QUERY_ID,
    QUERY_TEXT,
    USER_NAME,
    WAREHOUSE_NAME,
    START_TIME,
    END_TIME,
    DATEDIFF(SECOND, START_TIME, END_TIME) AS RUN_TIME_IN_SECONDS
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY(
    END_TIME_RANGE_START => DATEADD(HOUR,-1,CURRENT_TIMESTAMP()),
    END_TIME_RANGE_END => CURRENT_TIMESTAMP() ))
WHERE DATEDIFF(MINUTE, START_TIME, END_TIME) > 5
ORDER BY START_TIME;
```

> _The arguments in table function are not mandatory. If END_TIME_RANGE_END is not specified, the function returns all queries, including those that are still running currently._

The below query retrieves the queries started in last 5 minutes and still running.

```sql
SELECT
    QUERY_ID,
    QUERY_TEXT,
    USER_NAME,
    WAREHOUSE_NAME,
    START_TIME,
    DATEDIFF(SECOND, START_TIME, CURRENT_TIMESTAMP) AS RUN_TIME_IN_SECONDS
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY(END_TIME_RANGE_START => DATEADD(MINUTE, -5, CURRENT_TIMESTAMP)))
WHERE EXECUTION_STATUS='RUNNING'
ORDER BY START_TIME;
```

## **5. Finding long running queries by User**

The below query retrieves the queries started in last 5 minutes and still running by user named “TONY”. The user name is passed through the argument **USER_NAME** to the **`QUERY_HISTORY_BY_USER`** table function.

```sql
SELECT
    QUERY_ID,
    QUERY_TEXT,
    USER_NAME,
    WAREHOUSE_NAME,
    START_TIME,
    DATEDIFF(SECOND, START_TIME, CURRENT_TIMESTAMP) AS RUN_TIME_IN_SECONDS
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY_BY_USER(USER_NAME => 'TONY'))
WHERE EXECUTION_STATUS='RUNNING'
ORDER BY START_TIME;
```

> _Arguments used in QUERY_HISTORY table function can be used in all of its family of table functions. Refer the example below._

## **6. Finding long running queries by Warehouse**

The below query retrieves the queries which are triggered in last 1 hour and still running on warehouse named “’COMPUTE_WH”. The warehouse name is passed through the argument **WAREHOUSE_NAME** to the **`QUERY_HISTORY_BY_ WAREHOUSE`** table function.

```sql
SELECT
    QUERY_ID,
    QUERY_TEXT,
    USER_NAME,
    WAREHOUSE_NAME,
    START_TIME,
    DATEDIFF(SECOND, START_TIME, CURRENT_TIMESTAMP) AS RUN_TIME_IN_SECONDS
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY_BY_WAREHOUSE(
    END_TIME_RANGE_START => DATEADD(HOUR,-1,CURRENT_TIMESTAMP()),
    END_TIME_RANGE_END => CURRENT_TIMESTAMP(),
    WAREHOUSE_NAME => 'COMPUTE_WH'))
WHERE EXECUTION_STATUS='RUNNING'
ORDER BY START_TIME;
```

## **7. Killing a long running query in Snowflake using `SYSTEM$CANCEL_QUERY`**

**`SYSTEM$CANCEL_QUERY`** kills/cancels the specified query (or statement) if it is currently active/running. The id of the query can be extracted by using the queries shared above.

```sql
SELECT SYSTEM$CANCEL_QUERY('<QUERY_ID>');
```

A user can cancel their own running SQL queries. Cancelling queries executed by another user requires a role with one of the following privileges.

-   OWNERSHIP on the user who executed the query.
-   OPERATE or OWNERSHIP on the warehouse that is running the query.

