---
created: 2023-09-25T19:48:49 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-rename-a-table-in-snowflake
author: 
---

# Snowflake: Rename a Table 

## How to Rename a Table in Snowflake

Renaming a table in Snowflake is performed by using `ALTER TABLE .. RENAME TO` statement:

```sql
--the syntax
alter table old_table_name rename to new_table_name;

--rename users_marketing to users table
alter table sessions_db1 rename to sessions_db_1;
```
