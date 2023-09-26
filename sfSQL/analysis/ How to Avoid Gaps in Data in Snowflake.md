---
created: 2023-09-25T20:36:24 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-avoid-gaps-in-data-in-snowflake
author: 
---

# How to Avoid Gaps in Data in Snowflake 

Let's say you need to group by time in Snowflake but you don't want any gaps in your report data.

First, generate a series of date/time values that have no gaps using a [common table expression](https://popsql.com/learn-sql/snowflake/how-to-write-a-common-table-expression-in-snowflake/):

```sql
set start_date = '2020-04-01';
set end_date = '2020-04-30';

with cte_date (date_rec) as (
  select to_date($start_date)
  union all
  select to_date(dateadd(day, 1, date_rec)) --or week, month, week, hour, minute instead of day
  from  cte_date
  where  date_rec < $end_date
)
select date_rec
from cte_date;
```

> Note: you can [run multiple statements](https://popsql.com/docs/getting-started/writing-a-query/) in sequence in PopSQL with just one click. In the example above, you could highlight all 3 statements at once and then click `Run All`.

```txt
date_rec
------------------------
2020-04-01
2020-04-02
2020-04-03
2020-04-04
...
2020-04-30
```

Now you can left join your date series data against this gapless series. Here's how you could create a count of sessions for each day:

```sql
set start_date = '2020-04-01';
set end_date = '2020-04-30';

with cte_date (date_rec) as (
  select  to_date($start_date)
  union all
  select  to_date(dateadd(day, 1, date_rec))
  from  cte_date
  where  date_rec < $end_date
)

select
  cte_date.date_rec,
  count(s.id) as session_ct
from cte_date
left outer join sessions s on to_date(s.start_date) = cte_date.date_rec
group by date_rec;
```
