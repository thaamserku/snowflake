---
created: 2023-09-25T19:48:39 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-rename-a-column-in-snowflake
author: 
---

# Snowflake: Rename a Column using Alter Table 

## How to Rename a Column in Snowflake

Renaming a column in Snowflake involves using the `ALTER TABLE .. RENAME COLUMN` command.

```sql
-- syntax
alter table table_name rename column old_name to new_name;

-- rename product_category column in products table
alter table products rename column products_category to products;
```
