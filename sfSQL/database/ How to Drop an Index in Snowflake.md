---
created: 2023-09-25T19:13:47 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-drop-an-index-in-snowflake
author: 
---

# How to Drop an Index in Snowflake

As previously mentioned, Snowflake doesn't support the concept of indices.

As a substitute in particular situations, you can use clustering keys. Creation of clustering keys is explained [here](https://popsql.com/learn-sql/snowflake/how-to-create-an-index-in-snowflake/), and this article will show you how to drop a clustering key for a particular table.

To drop a clustering key, use the `ALTER TABLE .. DROP CLUSTERING KEY` command:

```sql
-- syntax
alter table table_name drop clustering key;

alter table active_users drop clustering key;
```
