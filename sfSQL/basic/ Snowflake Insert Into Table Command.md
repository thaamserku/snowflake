---
created: 2023-09-25T18:35:13 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-insert-data-in-snowflake
author: 
---

# Snowflake: Insert Into Table Command 

Here's the shortest and easiest way to insert data into a Snowflake table. You only have to specify the values, but you have to pass all values _in order_. If you have 10 columns, you have to specify 10 values.

```sql
-- assuming the sessions table has only four columns:
-- id, start_date, and end_date, and category, in that order
insert into sessions values (1, '2020-04-02 14:05:15.400', '2020-04-03 14:25:15.400', 1);
```

It's optional, but specifying a column list before the `VALUES` keyword is **highly recommended**:

```sql
insert into sessions (id, start_date, end_date, category)
values (12, '2020-04-02 14:05:15.400', '2020-04-04 16:57:53.653', 2);
```

By specifying a column list, you don't have to remember the column order as defined in the table:

```sql
insert into sessions (id, category, start_date, end_date)
values (3, 2, '2020-04-02 14:05:15.400', '2020-04-04 16:57:53.653');
```

Having a column list has more advantages:

-   You don't have to specify a value for all columns, just the required ones.
-   In case there are many columns, it is easier to match a value to the intended column when you see it in the statement, rather than having to look at the table definition.
-   `INSERT` statements without a column lists are invalidated once a column is added or removed from the table. You need to modify your query to reflect the new or deleted column in order for them to work again.

If you have many columns, but only want to specify some:

```sql
insert into sessions(id, start_date) values (4, '2020-04-02 14:05:15.400');
```

### **Inserting Multiple Rows**

You can insert multiple rows in one `INSERT` statement by having multiple sets of values enclosed in parentheses:

```sql
insert into sessions (id, start_date, end_date, category)
values
  (5, '2020-04-02 15:05:15.400','2020-04-03 15:14:30.400', 3),
  (6, '2020-04-02 17:07:16.300','2020-04-02 19:10:15.400', 4),
  (7, '2020-04-03 15:05:45.127','2020-04-04 18:05:15.400', 2);
```

You can also use `CREATE TABLE` with a `SELECT` command to copy data from an existing table. Note the `VALUES` keyword is omitted:

```sql
--ommiting a cloumn list specification
create table sessions_dm_1 as
select *
from sessions
where id <= 5;

--specifying a column list
create table sessions_dm_2 (id, start_date, end_date, category) as
select *
from sessions
where id > 5;
```

### **Inserting JSON Values**

If you want to insert into a JSON column, just wrap the valid JSON in a single quoted string:

```sql
insert into sessions(dates) values('{ "start_date": "2020-04-02 14:05:15.400", "end_date": "2020-04-02 14:57:45.127" }');
```

See also: our tutorial on [querying JSON columns](https://popsql.com/learn-sql/snowflake/how-to-query-a-json-object-in-snowflake/).

### **Handling Conflicts/Duplicates**

If inserting a row would violate a unique constraint, there is a way to handle it.

```sql
--if you want to insert the row if doesn't exists, otherwise ignore
insert into sessions(id, dates)
select
  1,
  '{ "start_date": "2020-04-02 14:05:15.400", "end_date": "2020-04-04 14:57:45.127" }'
where not exists (select id from sessions where id=1);


--if you want to update sessions_db_1 row if exists with sessions_db_2 row value, otherwise insert in sessions_db_1:
merge into sessions_db_1 a using (select id, dates from sessions_db_2 where id=3) as b on a.id=b.id
  when matched then update set a.dates=null --id=3 exists in both tables: sessions_db_1 will be updated with sessions_db_2 row value
  when not matched then insert (id, dates) values (b.id, b.dates); --id=3 exists in sessions_db_2, but doesn't exist in sessions_db_1: the row from sessions_db_2 will be inserted into sessions_db_1
```
