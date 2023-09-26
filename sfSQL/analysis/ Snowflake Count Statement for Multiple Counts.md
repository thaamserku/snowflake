---
created: 2023-09-25T20:56:23 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-have-multiple-counts-in-snowflake
author: 
---

# Snowflake Count Statement for Multiple Counts
## How to Have Multiple Counts in Snowflake

To do multiple counts in one query in Snowflake, you can combine `sum()` with the [CASE statement](https://popsql.com/learn-sql/snowflake/how-to-write-a-case-statement-in-snowflake/):

```sql
select
  count(1), -- count all products
  sum(case when name = 'Table' then 1 else 0 end), -- count all tables
  sum(case when category = 4 then 1 else 0 end) -- count all category 4 products
from products
```
