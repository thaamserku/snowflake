---
created: 2023-09-25T20:37:10 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-write-a-common-table-expression-in-snowflake
author: 
---

# Snowflake CTE: Common Table Expression in Snowflake 

## How to Write a Common Table Expression in Snowflake

Common table expressions (CTEs) are a great way to break up complex queries. Snowflake also supports this functionality. Here's a simple query to illustrate how to write a CTE:

```sql
with free_users as (
  select *
  from users
  where plan = 'free'
)
select user_sessions.*
from user_sessions
inner join free_users on free_users.id = user_sessions.user_id
order by free_users.id;
```
