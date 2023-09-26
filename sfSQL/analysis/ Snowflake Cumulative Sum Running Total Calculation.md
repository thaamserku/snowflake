---
created: 2023-09-25T20:36:37 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-calculate-cumulative-sum-running-total-in-snowflake
author: 
---

# Snowflake: Cumulative Sum/Running Total Calculation

Let's say we want a graph that's always "up and to the right" in Snowflake. Say, a graph of our cumulative sessions by day. First, we'll need a table with a day column and a count column:

```sql
select
  to_date(start_date) as day,
  count(1)
from sessions
group by to_date(start_date);
```

```txt
     day    | count
------------+-------
 2020-05-01 | 1
 2020-05-02 | 4
 2020-05-03 | 2
```

Next, we'll write a [Snowflake common table expression (CTE)](https://popsql.com/learn-sql/snowflake/how-to-write-a-common-table-expression-in-snowflake/) and use a window function to keep track of the cumulative sum/running total:

```sql
select
  to_date(start_date) as day,
  count(1)
from sessions
group by to_date(start_date);

with data as (
  select
    to_date(start_date) as day,
    count(1)  as number_of_sessions
  from sessions
  group by to_date(start_date)
)

select
  day,
  sum(number_of_sessions) over (order by day asc rows between unbounded preceding and current row)
from data;
```

```txt
      day    | sum
------------+-------
 2020-05-01 | 1
 2020-05-02 | 5
 2020-05-03 | 7
```

Voila! 📈
