---
created: 2023-09-25T19:48:19 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-remove-a-default-value-to-a-column-in-snowflake
author: 
---

# How to Remove a Default Value to a Column in Snowflake 

To remove a default value to a column in Snowflake, use the `ALTER TABLE <table_name> ALTER <column_name> DROP DEFAULT` command:

```sql
alter table products 
alter id 
drop default;
```