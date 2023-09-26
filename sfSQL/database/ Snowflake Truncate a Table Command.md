---
created: 2023-09-25T19:48:57 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-truncate-a-table-in-snowflake
author: 
---

# Snowflake Truncate a Table Command

Be super careful with `TRUNCATE` command. It will empty the content of your Snowflake table. This is useful in development, but you'll seldom want to do this in production.

```sql
--syntax
truncate table_name;

truncate users;
```

If the table contains an `AUTO_INCREMENT` column, the counter for it will not get reset. Note that this behavior is different vs some other databases like [SQLServer](https://popsql.com/learn-sql/sql-server/how-to-alter-sequence-in-sql-server/) for example, where you don't need to reset the counter yourself.
