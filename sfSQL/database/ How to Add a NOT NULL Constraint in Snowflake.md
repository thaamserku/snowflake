---
created: 2023-09-25T18:49:16 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-add-a-not-null-constraint-in-snowflake
author: 
---

# How to Add a NOT NULL Constraint in Snowflake 

Not null constraints add another layer of validation to your data.

While you could perform this validation in your application layer, be aware that inconsistencies will happen: someone will forget to add the validation, or remove it by accident, or bypass validations in a console and insert nulls, etc. The only sure way is to enforce it in your column definition. If you're validating nulls on the database layer as well, you're protected.

To enforce `NOT NULL` for a column in Snowflake, use the `ALTER TABLE <table_name> ALTER <column_name>` command and restate the column definition, adding the `NOT NULL` attribute.

```sql
alter table products
alter category not null;
```
