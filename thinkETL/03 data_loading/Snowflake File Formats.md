---
created: 2023-09-24T12:23:02 (UTC -04:00)
tags: []
source: https://thinketl.com/snowflake-file-formats/
updated: 2024-04-21 12:13:45
---

# Snowflake File Formats 

> Snowflake File format simplifies the process of accessing the staged data and streamlines data loading/unloading in database tables.

---
## **1. What is Snowflake File Format?**

**Snowflake File format is a named database object that can be used to simplify the process of accessing the [staged](https://thinketl.com/types-of-snowflake-stages-data-loading-and-unloading-features/) data and streamlines loading data into and unloading data out of database tables.** **A Snowflake File format encapsulates information of data files, such as file type (CSV, JSON, etc.) and formatting options specific to each type used for bulk loading/unloading.**

## **2. What file formats are supported in Snowflake?**

The following are the file formats supported in Snowflake that can be accessed from Snowflake Stages.

1.  CSV
2.  JSON
3.  AVRO
4.  ORC
5.  PARQUET
6.  XML

The following file formats are supported for both loading data into and unloading data out of database tables.

1.  CSV
2.  JSON
3.  PARQUET

The following file formats are supported for loading only. The data cannot be unloaded from tables to below file formats.

1.  AVRO
2.  ORC
3.  XML

## **3. How to create Snowflake File Formats?**

The Snowflake file formats can be created using two different methods.

1.  Using Snowflake WEB UI
2.  Using SQL statements

### **METHOD-1: Creating Snowflake file formats form WEB UI**

1. Navigate to _Databases_ in the Snowflake classic console present on the top of the page.

2. Select the _database_ in which you wanted to create a file format.

3. Go to _File Formats_ and click _Create_.

![Creating Snowflake File Format from WEB UI](https://thinketl.com/wp-content/uploads/2022/07/88-1-Creating-Snowflake-file-formats-from-WEB-UI.png)

Creating Snowflake File Format from WEB UI

4. In the create file format menu, complete the following mandatory parameters.

-   **Name**: Specify a name for the file format. It must be unique for the schema in which the file format is created.
-   **Schema**: The Schema in which the file format to be created.
-   **Format** **Type**: Select the format of the file residing in stage you which to access.

Rest of the parameters are optional and vary according to the file format type you choose.

The below image shows a file format MY_CSV_FORMAT of type CSV created in schema MY_SCHEMA.

![Specifying File Format parameters](https://thinketl.com/wp-content/uploads/2022/07/88-2-Creating-Snowflake-file-formats-from-WEB-UI.png)

Specifying File Format parameters

5. Click _Finish_ to create the File Format.

The created file formats are available under _File Formats_. As an owner of the file format, you have rights to clone, edit drop and transfer its ownership to other roles and users.

![Created File Formats in Snowflake](https://thinketl.com/wp-content/uploads/2022/07/88-3-Creating-Snowflake-file-formats-from-WEB-UI.png)

Created File Formats in Snowflake

### **METHOD-2: Creating Snowflake file formats using SQL**

The syntax to create Snowflake file format using SQL statement is as shown below.

```sql
CREATE OR REPLACE FILE FORMAT <FILE FORMAT NAME>
  TYPE = <FILE FORMAT TYPE>
  <OPTIONAL PARAMETERS>
;
```

The below example shows a CSV file format named _my_csv_format_ with filed delimiter as comma which includes a single header line that will be skipped.

```sql
CREATE OR REPLACE FILE FORMAT MY_CSV_FORMAT
  TYPE = CSV
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
;
```

The below example shows a JSON file format named _my_json_format_ that uses all the default JSON format options.

```sql
CREATE OR REPLACE FILE FORMAT MY_JSON_FORMAT
  TYPE = JSON;
```

> _When you don’t specify the optional file format parameters, default values of those parameters are considered._

## **4. Where Snowflake File formats are used?**

Snowflake file formats are used while loading/unloading data from Snowflake stages into tables using [**COPY INTO**](https://thinketl.com/types-of-snowflake-stages-data-loading-and-unloading-features/#53_COPY_INTO_command) command and while creating [**EXTERNAL TABLES**](https://thinketl.com/how-to-create-snowflake-external-tables/) on files present in stages.

The following example shows loading data into Employee table from stage named my_stage using file format my_csv_format.

```sql
COPY INTO EMPLOYEE FROM @MY_STAGE/INPUT.CSV
FILE_FORMAT = (FORMAT_NAME = MY_CSV_FORMAT);
```

The following example shows creating an external table my_ext_table on top of employee files using file format my_csv_format.

```sql
CREATE OR REPLACE EXTERNAL TABLE MY_EXT_TABLE
  WITH LOCATION = @MY_AZURE_STAGE/
  FILE_FORMAT = (FORMAT_NAME = MY_CSV_FORMAT)
  PATTERN='.*EMPLOYEE.*[.]CSV';
```

## **5. Specifying Adhoc File Formats in Snowflake**

Alternatively the file formats can be specified on adhoc basis while creating external tables or performing data loading/unloading operations.

The following example shows loading data into Employee table from stage named my_stage using an adhoc CSV file format.

```sql
COPY INTO EMPLOYEE FROM @MY_STAGE/INPUT.CSV
FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1);
```

The following example shows creating an external table my_ext_table on top of employee files using file adhoc CSV file format.

```sql
CREATE OR REPLACE EXTERNAL TABLE MY_EXT_TABLE
  WITH LOCATION = @MY_AZURE_STAGE/
  FILE_FORMAT = (TYPE = CSV  SKIP_HEADER = 1)  
  PATTERN='.*EMPLOYEE.*[.]CSV';
```

## **6. Closing Points**

In order to able to create file formats in Snowflake, the users must be assigned with a role having CREATE FILE FORMAT privileges at a minimum. Note that operating on any object (file formats, tables, sequences etc.) in a schema also requires the USAGE privilege on the parent database and schema.

Recreating a file format (using CREATE OR REPLACE FILE FORMAT) breaks the association between the file format and any external table that references it. If you must recreate a file format after it has been linked to one or more external tables, you must recreate each of the external tables (using CREATE OR REPLACE EXTERNAL TABLE) to re-establish the association.

