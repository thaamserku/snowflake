---
created: 2023-09-25T18:44:07 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-update-data-in-snowflake
author: 
---

# Snowflake: Update a Table, Rows and Columns 

## How to Use Snowflake Update Statement

Usually you only want to update rows that match a certain condition. You do this by specifying a `WHERE` clause:

```sql
--this will update only one row that matches id = 1
update sessions
set start_date = '2020-04-20 10:12:15.653',
    end_date = '2020-04-22 15:40:30.123'
where id = 1;

--this will update multiple rows that match category = 1
update sessions
set end_date = null
where category = 1;
```

To update all rows in a Snowflake table, just use the `UPDATE` statement without a `WHERE` clause:

```sql
update sessions
set end_date = '2020-04-04 16:57:53.653';
```

You can also update multiple columns at a time:

```sql
update sessions
set start_date = '2020-04-02 14:05:15.400',
    end_date = '2020-04-04 16:57:53.653';
```
