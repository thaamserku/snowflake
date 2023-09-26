---
created: 2023-09-25T18:51:08 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-create-an-index-in-snowflake
author: 
---

# Snowflake Indexes: How to Create an Index in Snowflake 


>[!note]
Let's immediately clarify one thing: Snowflake doesn't support indices. Instead of creating or dropping an index in Snowflake, you can use clustering keys to accomplish query performance. This tutorial will show you how to define a clustering key for a particular table.

To create clustering key, use the `ALTER TABLE .. CLUSTER BY` command:

```sql
-- syntax
alter table table_name cluster by (column1, column2, .., columnN);

-- create clustering key on one column
alter table active_users
cluster by (id)

-- create clustering on multiple columns
alter table active_users
cluster by (id, active)
```

However, you should be very careful about using clustering keys. Clustering keys should be used only in case you are dealing with tables of the following properties:

-   Tables containing huge amount of data (i.e. measured in terabytes).
-   Data from tables are filtered on a regular basis.

Note: Unlike other technologies that allow us creation of multiple indices on table, Snowflake lets us create only one clustering key per table!
