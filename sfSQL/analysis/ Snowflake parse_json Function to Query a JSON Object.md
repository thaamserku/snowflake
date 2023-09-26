---
created: 2023-09-25T20:56:11 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-query-a-json-object-in-snowflake
author: 
---

# Snowflake parse_json Function to Query a JSON Object

Snowflake supports querying JSON columns. This gives the advantage of storing and querying unstructured data. Here's how you can query a JSON column in Snowflake.

Get only `salesperson.name` from the employees table:

```sql
--level 2 element: get salesperson.name from the customers table
select parse_json(text):salesperson.name as sales_person_name
from customers
```

Get only customers related to a particular sales person:

```sql
select *
from customers
where parse_json(text):salesperson.name='Alice Miller'
```

Get first key and its value of a JSON data set:

```sql
select top 1 parse_json(text):customer as customer_key
from customers

--going deeper to get only customer name
select top 1 parse_json(text):customer.name as customer_name_key
from customers
```

Get JSON keys as columns populated with key values:

```sql
select
  parse_json(text):customer.name as customer_name, -- level 2 element having one value
  parse_json(text):customer.address as customer_address,
  parse_json(text):customer.phone as customer_phone,
  parse_json(text):dealership as dealership, -- level 1 element having one value
  parse_json(text):salesperson.id as sales_person_id,
  parse_json(text):salesperson.name as sales_person_name,
  parse_json(text):vehicle.extras[0] as extras_1, -- first value of level 2 element having three values
  parse_json(text):vehicle.extras[1] as extras_2, -- second value of level 2 element having three values
  parse_json(text):vehicle.extras[2] as extras_3  -- third value of level 2 element having three values
from customers
```
