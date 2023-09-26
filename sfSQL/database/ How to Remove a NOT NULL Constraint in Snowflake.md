---
created: 2023-09-25T19:48:29 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-remove-a-not-null-constraint-in-snowflake
author: 
---

# How to Remove a NOT NULL Constraint in Snowflake 

To remove a `NOT NULL` constraint for a column in Snowflake, you use the `ALTER TABLE <table_name> ALTER <column_name> DROP` command and restate the column definition, adding the `NOT NULL` attribute.

```sql
alter table products
alter category drop not null;
```
