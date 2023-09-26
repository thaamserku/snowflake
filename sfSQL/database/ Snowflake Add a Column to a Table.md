---
created: 2023-09-25T18:49:00 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-add-a-column-in-snowflake
author: 
---

# Snowflake: Add a Column to a Table

## How to Add a Column in Snowflake using Alter Table

Adding a column in Snowflake involves using the `ALTER TABLE` command.

Adding a brand\_id smallint column:

```sql
alter table products
add brand_id smallint;
```

Adding a brand\_id smallint column with a default value:

```sql
alter table products
add column brand_id smallint default 1;
```

Adding a string (varchar) column with a not null constraint:

```sql
-- note: this is possible only if the table contains no data!
alter table products
add description varchar(100) not null;
```

Adding more columns at the same time:

```sql
alter table products
add
  brand_id smallint default 1,
  description varchar(100) not null;
```
