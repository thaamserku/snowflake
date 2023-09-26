---
created: 2023-09-25T20:55:47 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-use-coalesce-in-snowflake
author: 
---

# Snowflake: Coalesce() Function 

Imagine you're looking at a Snowflake integer column where some rows are null:

```sql
select day, tickets
from stats;
```

```txt
    day     | tickets
------------+---------
 2020-05-01 | 1
 2020-05-02 | null
 2020-05-03 | 3
```

Instead of having that null, you might want that row to be 0. To do that, use the `coalesce()` function, which returns the first non-null argument it's passed:

```sql
select day, coalesce(tickets, 0)
from stats;
```

```txt
    day     | tickets
------------+---------
 2020-05-01 | 1
 2020-05-02 | 0
 2020-05-03 | 3
```
