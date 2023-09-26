---
created: 2023-09-25T18:48:31 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-add-a-default-value-to-a-column-in-snowflake
author: 
---

# How to Add a Default Value to a Column in Snowflake 

Adding a default value to a table column in Snowflake can be tricky. Unfortunately, we can't add a default for an existing column or modify it unless the default value is the sequence.

To modify a default for a column which already contains a sequence default, use the `ALTER TABLE <table_name> ALTER <column_name> SET DEFAULT` command:

```sql
-- example: default id values changed from one sequence to another
alter table products
alter id
set default id_seq.nextval;
```
