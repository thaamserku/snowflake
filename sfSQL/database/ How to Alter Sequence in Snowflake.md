---
created: 2023-09-25T18:50:13 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-alter-sequence-in-snowflake
author: 
---

# How to Alter Sequence in Snowflake 

Auto-incrementing columns start at 1 by default. Sometimes you want them to start at a different number and/or increment by a different amount. These numbers are known as “sequences”. Here is how to create them in Snowflake:

```sql
-- syntax
create sequence sequence_name
start = number
increment = number;
```

Some examples:

```sql
create sequence even_numbers
start = 2
increment = 2;

create sequence negative_numbers
start = 0
increment = -1; -- sequence that increments backward
```

To alter the sequence so that IDs start with a different number, you can't just do an `UPDATE`. You have to use the `ALTER SEQUENCE` command:

```sql
--change the increment number of a sequence
alter sequence even_numbers
set increment = 10;

--set a comment for a sequence
alter sequence even_numbers
set comment = 'even dozens';

--rename a sequence
alter sequence even_numbers rename to even_numbers_dozens;
```
