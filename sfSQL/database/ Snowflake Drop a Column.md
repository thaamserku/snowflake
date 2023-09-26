---
created: 2023-09-25T19:13:15 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-drop-a-column-in-snowflake
author: 
---

# Snowflake: Drop a Column 

## How to Drop a Column in Snowflake

Dropping a column in Snowflake involves using the `ALTER TABLE .. DROP COLUMN` command.

Drop one column:

```sql
alter table products
drop column description;
```

Drop multiple columns at the same time:

```sql
alter table products
drop column price, description;
```
