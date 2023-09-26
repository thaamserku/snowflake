---
created: 2023-09-25T20:39:10 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-do-type-casting-in-snowflake
author: 
---

# Snowflake: Cast to String and Integer 

By default, Snowflake is not strict with type casting. For example, adding a numeric value in string quotes to another numeric value with not give the usual errors other databases and programming languages will give:

```sql
select 10 + '10';
```

However, should the need arise, you can use the `cast()` function to force the type of a value.

```sql
-- cast float to integer
select cast(1.0123456789 as int);

-- cast string to date
select cast('2020-04-22' as date);

-- cast string to decimal
select cast('12.345' as decimal(5,2));

-- cast string to time
select cast('12:45' as time);
```

You can cast to many Snowflake types such as `binary`, `char`, `varchar`, `date`, `datetime`, `time`, `decimal`, `bigint`, and `float`.
