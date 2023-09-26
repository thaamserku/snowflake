---
created: 2023-09-25T18:50:59 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-create-a-view-in-snowflake
author: 
---

# Snowflake: Create a View 

Views let you to encapsulate or “hide” complexities, or allow limited read access to part of the data.

To create a view, use the `CREATE VIEW` command:

```sql
-- syntax
create view view_name
as select_statement;
```

Some examples:

1.  a view to show only products within category 1

```sql
create view category_1_products_v as
select *
from products
where category = 1;
```

2.  a view to limit read access to only certain columns

```sql
create view category_products_basic_v as
select
  name,
  category,
  unit_price
from products;
```

3.  a view that displays top 10 products that provide highest sold value

```sql
create view top_10_products_v as
select
  top 10 p.name,
  p.category,
  p.unit_price,
  ps.quantity_sold,
  p.unit_price * ps.quantity_sold as sold_value
from products p
left join products_sold ps on p.id = ps.product_id
order by sold_value desc;
```
