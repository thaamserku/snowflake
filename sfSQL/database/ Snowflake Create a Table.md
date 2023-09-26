---
created: 2023-09-25T18:50:46 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-create-a-table-in-snowflake
author: 
---

# Snowflake: Create a Table  

Here's an example of creating a `users` table in Snowflake:

```sql
create table users (
  id integer autoincrement, -- auto incrementing IDs
  name varchar (100),  -- variable string column
  preferences variant, -- column used to store JSON type of data
  created_at timestamp
);
```

Within the parentheses are what's called _column definitions_, separated by commas. The minimum required fields for a column definition are column name and data type (shown above for columns `name`, `preferences`, and `created_at`). The `id` column has extra field to use an auto-incrementing feature to assign it values.

This is also a chance to specify [not null constraints](https://popsql.com/learn-sql/snowflake/how-to-add-a-not-null-constraint-in-snowflake/) and [default values](https://popsql.com/learn-sql/snowflake/how-to-add-a-default-value-to-a-column-in-snowflake/):

```sql
create table users (
  id integer autoincrement,
  name varchar(100) not null,
  active boolean default true
);
```

You can also create temporary tables that will stick around for the duration of your session. This is helpful to break down your analysis into smaller pieces.

```sql
create temporary table inactive_users (
  id integer autoincrement,
  name varchar(100) not null,
  active boolean default false
);
```

Snowflake allows us to create transient tables which are a mix of permanent and temporary tables. They are used to store temporary data outside our session without having the need to implement a high level of data security and data recovery.

```sql
create transient table active_users (
  id integer autoincrement,
  name varchar(100) not null,
  active boolean default true
);
```
