---
created: 2023-09-25T20:15:05 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-round-timestamps-in-snowflake
author: 
---

# Snowflake: date_trunc Function to Round Timestamps 

Rounding and/or truncating timestamps is useful when you're [grouping by time](https://popsql.com/learn-sql/snowflake/how-to-group-by-time-in-snowflake/). There are a few approaches.

## The DATE_TRUNC function

```sql
select date_trunc('second', now()) -- or minute, hour, day, month
```

## Predefined functions in Snowflake

If you are rounding by year, you can use the `year()` function (or `month()`, `week()`, `day()`, etc:

```sql
select year(getdate()) as year;
```

Be careful though. Using the `month()` function will, for example, make January 2020 and January 2019 both just translate to `1`. That may not be what you want

## Round and format result as a string, e.g. '05-2020'

However, if you want to distinguish between months of different years, you need to use `to_varchar()` function:

```sql
select to_varchar(getdate(), 'mm-yyyy'); -- round to month
select to_varchar(getdate(),'dd-mm-yyyy'); -- round to day
select to_varchar(getdate(),'dd-mm-yyyy hh'); -- round to hour
select to_varchar(getdate(),'dd-mm-yyyy hh:mm'); -- round to minute
select to_varchar(getdate(),'dd-mm-yyyy hh:mm:ss'); -- round to second
```
