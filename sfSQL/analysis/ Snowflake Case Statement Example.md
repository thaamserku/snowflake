---
created: 2023-09-25T20:56:01 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-write-a-case-statement-in-snowflake
author: 
---

# Snowflake: Case Statement 

## How to Write a Snowflake Case When

Case statements are useful when you're reaching for an if statement in your `select`clause.

```sql
select
  id,
  name,
  category,
  unit_price,
  case
    when category = 5 then 'Premium'
    when category = 4 then 'Gold'
    when category = 3 then 'Standard'
    when category <= 2 then 'Basic'
    else 'unknown'
  end as quality_level
from products;
```
