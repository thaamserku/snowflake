---
created: 2023-09-25T20:36:50 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-calculate-percentiles-in-snowflake
author: 
---

# Snowflake: Percentile Calculation with percentile_cont() Function
## How to Calculate Percentiles in Snowflake

Let's say you want to look at the percentiles for products. You can use Snowflake's `percentile_cont()` function to do that:

```sql
select
  percentile_cont(0.25) within group(order by unit_price) over () as p25,
  percentile_cont(0.50) within group(order by unit_price) over () as p50,
  percentile_cont(0.75) within group(order by unit_price) over () as p75,
  percentile_cont(0.95) within group(order by unit_price) over () as p95
from products;
```

If you want to view those percentiles by category:

```sql
select distinct category,
  percentile_cont(0.25) within group(order by unit_price) over (partition by category) as p25,
  percentile_cont(0.50) within group(order by unit_price) over (partition by category) as p50,
  percentile_cont(0.75) within group(order by unit_price) over (partition by category) as p75,
  percentile_cont(0.95) within group(order by unit_price) over (partition by category) as p95
from products
order by category;
```
