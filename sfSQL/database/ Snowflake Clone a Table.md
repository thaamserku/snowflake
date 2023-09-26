---
created: 2023-09-25T19:48:08 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-duplicate-a-table-in-snowflake
author: 
---

# Snowflake: Clone a Table - PopSQL

Sometimes you need to duplicate a table. There are various methods, depending on your intent.  Copy both the entire table structure and all the data inside:

Sometimes you need to duplicate a table. There are various methods, depending on your intent.

Copy both the entire table structure and all the data inside:

```
--method 1
create table sessions_copy clone sessions;

--method 2
create table sessions_copy as
select *
from sessions;
```

Copy entire table structure along with particular data set:

```
create table sessions_db_1_copy as
select *
from sessions_db_1
where dates is null;
```

Copy only particular columns into new table along with particular data set:

```
create table sessions_dm_1_copy as
select
  id,
  start_date,
  end_date
from sessions_dm_1
where category = 2;
```

Copy only particular columns from more tables into new table along with particular data set:

```
create table users_sessions_1_rpt as
select
  u.name,
  s.start_date as session_start_date,
  s.end_date as session_end_date
from sessions s
left join user_sessions us on s.id = us.session_id
left join users_1 u on us.user_id = u.id
where u.active = true;
```

Copy only table structure, no data copying:

```
create table users_copy like users;
```
#capture