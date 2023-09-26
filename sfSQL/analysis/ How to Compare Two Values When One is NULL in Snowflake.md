---
created: 2023-09-25T20:38:58 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-compare-two-values-when-one-is-null-in-snowflake
author: 
---

# How to Compare Two Values When One is NULL in Snowflake - PopSQL


Say you're comparing two Snowflake columns and you want to know how many rows are different. No problem, you think:

```sql
select count(1)
from sessions
where start_date != end_date;
```

Wait a minute. If some of the start or end dates are null, they won't be counted! 😬 That's definitely not what you wanted. That's why you need to use the `INTERSECT` operator:

```sql
select count(1)
from sessions
where not exists(select start_date from sessions intersect select end_date from sessions);
```

Now, your count will be “null aware” and you'll get the result you want.
