---
created: 2023-09-25T19:13:28 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-drop-a-table-in-snowflake
author: 
---

# Snowflake: Drop a Table 

## How to Drop a Table in Snowflake

Dropping a table in Snowflake is simple:

```sql
drop table users;
```

Unlike other technologies, Snowflake offers the "undo" option for this command. If you want to restore the table, you can do it using the `undrop table` command:

```sql
undrop table users;
```

With the `undrop table` command, data that resided in the table will also be restored.
