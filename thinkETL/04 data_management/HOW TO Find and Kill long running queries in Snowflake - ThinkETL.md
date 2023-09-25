---
created: 2023-09-07T21:05:15 (UTC -04:00)
tags: []
source: https://thinketl.com/how-to-find-and-kill-long-running-queries-in-snowflake/
author: ThinkETL
---

# HOW TO: Find and Kill long running queries in Snowflake? - ThinkETL

> [!Excerpt]
> Learn how to find and kill long running queries in Snowflake using the QUERY_HISTORY table functions available under Information Schema.


## **1. Introduction**

The Snowflake **[INFORMATION\_SCHEMA](https://thinketl.com/snowflake-information-schema/)** consists of a set of system-defined views and **table functions** that provide extensive metadata information about the objects created in your account.

There is a table function that is made available under Snowflake information schema which provides historical information of queries triggered which helps in finding and killing the long running queries.

## **2. QUERY_HISTORY table function**

The **QUERY\_HISTORY** table function can be used to query Snowflake query history along various dimensions. The functions returns query activity within the last 7 days.

There is a family of table functions available under QUERY\_HISTORY

-   **QUERY\_HISTORY**: Returns queries within a specified time range.
-   **QUERY\_HISTORY\_BY\_SESSION**:  Returns queries within a specified session and time range.
-   **QUERY\_HISTORY\_BY\_USER**:  Returns queries submitted by a specified user within a specified time range.
-   **QUERY\_HISTORY\_BY\_WAREHOUSE**:  Returns queries executed by a specified warehouse within a specified time range.

## **3. Retrieving query history using QUERY_HISTORY table function**

The below query retrieves up to the last 100 queries run by the current user or run by any user on any warehouse on which the current user has the MONITOR privilege.

```sql
select *
from table(information_schema.query_history())
order by start_time;
```

## **4. Finding long queries within a time range**

### **4.1. Without arguments in table function**

The below query retrieves the long running queries which ran more than 5 minutes in last 1 hour.

```sql
select
    query_id,
    query_text,
    user_name,
    warehouse_name,
    start_time,
    end_time,
    datediff(second, start_time, end_time) as run_time_in_seconds
from table(information_schema.query_history())
where datediff(minute, start_time, end_time) > 5
and start_time > dateadd(hour, -1, current_timestamp())
order by start_time;
```

### **4.2. With arguments in table function**

Alternatively the above query can also written by passing arguments (**END\_TIME\_RANGE\_START**, **END\_TIME\_RANGE\_END**)  to the **QUERY\_HISTORY** table function as shown below.

```sql
select
    query_id,
    query_text,
    user_name,
    warehouse_name,
    start_time,
    end_time,
    datediff(second, start_time, end_time) as run_time_in_seconds
from table(information_schema.query_history(
    END_TIME_RANGE_START => dateadd(hour,-1,current_timestamp()),
    END_TIME_RANGE_END => current_timestamp() ))
where datediff(minute, start_time, end_time) > 5
order by start_time;
```

> _The arguments in table function are not mandatory. If END\_TIME\_RANGE\_END is not specified, the function returns all queries, including those that are still running currently._

The below query retrieves the queries started in last 5 minutes and still running.

```sql
select
    query_id,
    query_text,
    user_name,
    warehouse_name,
    start_time,
    datediff(second, start_time, current_timestamp) as run_time_in_seconds
from table(information_schema.query_history(END_TIME_RANGE_START => dateadd(minute, -5, current_timestamp)))
where execution_status='RUNNING'
order by start_time;
```

## **5. Finding long running queries by User**

The below query retrieves the queries started in last 5 minutes and still running by user named “TONY”. The user name is passed through the argument **USER\_NAME** to the **QUERY\_HISTORY\_BY\_USER** table function.

```sql
select
    query_id,
    query_text,
    user_name,
    warehouse_name,
    start_time,
    datediff(second, start_time, current_timestamp) as run_time_in_seconds
from table(information_schema.query_history_by_user(USER_NAME => 'TONY'))
where execution_status='RUNNING'
order by start_time;
```

> _Arguments used in QUERY\_HISTORY table function can be used in all of its family of table functions. Refer the example below._

## **6. Finding long running queries by Warehouse**

The below query retrieves the queries which are triggered in last 1 hour and still running on warehouse named “’COMPUTE\_WH”. The warehouse name is passed through the argument **WAREHOUSE\_NAME** to the **QUERY\_HISTORY\_BY\_ WAREHOUSE** table function.

```sql
select
    query_id,
    query_text,
    user_name,
    warehouse_name,
    start_time,
    datediff(second, start_time, current_timestamp) as run_time_in_seconds
from table(information_schema.query_history_by_warehouse(
    END_TIME_RANGE_START => dateadd(hour,-1,current_timestamp()),
    END_TIME_RANGE_END => current_timestamp(),
    WAREHOUSE_NAME => 'COMPUTE_WH'))
where execution_status='RUNNING'
order by start_time;
```

## **7. Killing a long running query in Snowflake using SYSTEM$CANCEL_QUERY**

SYSTEM$CANCEL\_QUERY kills/cancels the specified query (or statement) if it is currently active/running. The id of the query can be extracted by using the queries shared above.

```sql
select system$cancel_query('<query_id>');
```

A user can cancel their own running SQL queries. Cancelling queries executed by another user requires a role with one of the following privileges.

-   OWNERSHIP on the user who executed the query.
-   OPERATE or OWNERSHIP on the warehouse that is running the query.

