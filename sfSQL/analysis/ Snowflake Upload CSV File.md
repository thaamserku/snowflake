---
created: 2023-09-25T20:36:07 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-import-a-csv-in-snowflake
author: 
---

# Snowflake: Upload CSV File

Snowflake allows you to upload a CSV file from your local machines that run on Windows, Linux, or MacOS.

This tutorial will show you how to upload a CSV file from all three platforms to a Snowflake database table.

In this example, the CSV file to be imported is called `Enterprises`. It contains three columns (`id`, `name`, and `location`), is located in "test" folder of our local machine, and has the following structure:

```txt
 1,Microsoft,Washington
 2,Apple,California
 3,IBM,New York
 ...
```

Steps:

1.  Create a Snowflake stage

```sql
create or replace stage enterprises_stage;
```

2.  Create a file format using the `FILE FORMAT` command to describe the format of the file to be imported

```sql
create or replace file format enterprises_format type = 'csv' field_delimiter = ',';
```

3.  Upload your CSV file from local folder to a Snowflake stage using the `PUT` command

```sql
-- this step can not be performed by running the command from the Worksheets page on the Snowflake web interface. You'll need to install and use SnowSQL client to do this

-- Windows
put file://C:\test\Enterprises.csv @enterprises_stage;

-- Linux/Mac
put file:///tmp/data/Enterprises.csv @enterprises_stage;
```

4.  Check to see if the Snowflake stage is populated with the data from the file

```sql
select
  c.$1,
  c.$2,
  c.$3
from @enterprises_stage (file_format => enterprises_format) c;
```

5.  You need to create a table within Snowflake database that has the same structure as the CSV file we want to import prior to running the `COPY INTO` command.

```sql
create or replace table enterprises (
  id integer,
  name varchar (100),
  location varchar(100)
)
```

6.  Load data from a Snowflake stage into a Snowflake database table using a `COPY INTO` command

```sql
-- load data as it is organized in a CSV file
copy into test.enterprises from @enterprises_stage;

-- if you want to filter out data from a stage and import only particular columns
copy into test.enterprises from (select c.$1, c.$2 from @enterprises_stage (file_format => enterprises_format) c);
```

7.  Check to see if the Snowflake database table is populated with the data

```sql
select * from enterprises;

      id    | name        | location
------------+-------------+---------------
 1          | Microsoft   | Washington
 2          | Apple       | California
 3          | IBM         | New York
```
