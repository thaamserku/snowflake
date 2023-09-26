---
created: 2023-09-25T20:14:54 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-query-date-and-time-in-snowflake
author: 
---

# Snowflake: Query Date and Time with dateadd, getdate, datediff

Get the date and time right now (where Snowflake is running):

```sql
select current_timestamp;
select getdate();
select systimestamp();
select localtimestamp;
```

Find rows between two dates or timestamps:

```sql
select *
from events
where event_date between '2020-04-18' and '2020-04-20';

-- can include time by specifying in YYYY-MM-DD hh:mm:ss format:
select  *
from events
where event_date between '2020-04-18 12:00:00' and '2020-04-20 23:30:00';
```

> _Note: always put the dates in the 'between .. and' statement in ascending order(from smaller to larger date value)_

Find rows created within the last week:

```sql
select *
from events
where event_date > (select dateadd(week, -1, getdate()));
```

Find events scheduled between one week ago and 3 days from now:

```sql
select *
from events
where event_date between (select dateadd(week, -1, getdate())) and (select dateadd(day, +3, getdate()));
```

Extracting part of a timestamp and returning an integer:

```sql
select day(getdate()); -- or month() or year()
select date_part(hour, getdate()); -- or hour, week, month, quarter, year
```

Extracting part of a timestamp and returning a string (e.g. "Feb" or "Mon"):

```sql
select dayname(getdate());
select monthname(getdate());
```

Get the day of the week from a timestamp:

```sql
select date_part(weekday, getdate()); -- returns 0-6 (integer), starting with 0 as Sunday
```

How to convert a timestamp to a unix timestamp:

```sql
select datediff(second, '1970-01-01' , current_timestamp())
```

To calculate the difference between two timestamps, convert them to unix timestamps then subtract:

```sql
select datediff(second, '1970-01-01' , '2020-04-20 00:00:00.000 -0700') - datediff(second, '1970-01-01' , '2020-04-18 12:04:00.000 -0700') -- output in seconds
```
