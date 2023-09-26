---
created: 2023-09-25T20:14:46 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-group-by-time-in-snowflake
author: 
---

# How to Group by Time in Snowflake 

When you need to group by minute, hour, day, week, etc., you may think you can simply group by your timestamp column.

If you do that, though, you'll get one group per second -- probably not what you want. Instead you need to “truncate” your timestamp to the granularity you want, like minute, hour, day, week, etc. The function you need here is`date_trunc()`:

```sql
-- returns number of sessions grouped by particular timestamp fragment
select
  date_trunc('DAY',start_date), --or WEEK, MONTH, YEAR, etc
  count(id) as number_of_sessions
from sessions
group by 1
;
```
