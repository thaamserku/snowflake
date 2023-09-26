---
created: 2023-09-25T18:38:42 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-delete-data-in-snowflake
author: 
---

# Snowflake: Delete Data from a Table 

To delete rows in a Snowflake table, use the `DELETE` statement:

```sql
delete from sessions where id = 7;
```

The `WHERE` clause is optional, but you'll usually want it, unless you really want to delete every row from the table.

```sql
delete from sessions;
```
