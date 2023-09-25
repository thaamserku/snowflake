---
created: 2023-09-07T20:58:11 (UTC -04:00)
tags: [snowflake, stages]
source: https://thinketl.com/types-of-snowflake-stages-data-loading-and-unloading-features/
author: ThinkETL
---

# Types of Snowflake Stages: Data Loading and Unloading Features

> [!Excerpt]
> A complete guide to types of Snowflake Stages and how to load data into and unload data from Snowflake tables using stages.

## **1. What are Snowflake Stages?**

**Snowflake Stages are locations where data files are stored (“staged”) which helps in loading data into and unloading data out of database tables. The stage locations could be internal or external to Snowflake environment.**

## **2. Types of Snowflake Stages

Snowflake supports two types of stages for storing data files used for loading/unloading

-   **Internal stages** store the files internally within Snowflake.
-   **External stages** store the files in an external location (AWS S3 bucket or Azure Containers or GCP Cloud storage) that is referenced by the stage.

## **3. Internal Stages**

Snowflake Internal Stages stores data files internally within Snowflake. Internal stages can be either permanent or temporary. Snowflake Internal Stages are further classified into

-   User
-   Table
-   Named

By default, each **user** and **table** in Snowflake is automatically allocated an internal stage for staging data files. In addition, you can create **internal** **named** stages.

### **3.1. User Stage**

Each user has a Snowflake stage allocated to them by default for storing files and these cannot be altered or dropped. These stages are unique to the user, meaning no other user can access the stage. User Stages are not suitable option if files needs to be accessed by multiple users.

Snowflake store all the worksheets created by the user in the User stage of the user.

> _User stages are referenced using **@~**_

The following statement lists all the file (including worksheets) present in a user stage.

### **3.2. Table Stage**

Each table has a Snowflake stage allocated to it by default for storing files and these cannot be altered or dropped. Table stages can be accessed by multiple users but can only load data into the table it is allocated to. Table stages are not suitable if the data needs to be loaded into multiple tables.

> _Table stages are referenced using **@%** and have the same name as table._

The following statement lists all the files present in the stage of table Employee

> **User** and **Table stages** are not separate database objects, rather they are implicit stages tied to the user and the table respectively.

### **3.3. Internal Named Stage**

Named stages are database objects that provide the greatest degree of flexibility for data loading. They overcome the limitations of both User and Table stages.

-   Named stages are accessible by all the users with appropriate privileges.
-   The data from Named stages can be loaded into multiple tables.

The Internal Named stages should be created manually unlike User and Table stages. You can create a named internal stage using either the web interface or SQL.

Follow below steps to create an Internal Named Stage from web interface.

1.  Navigate to **Databases** > **_DatabaseName_** > **Stages**.
2.  Click Create, select **Snowflake Managed Storage** and click _Next_.
3.  Enter the name of the stage, select schema and click _Finish_.

The following statement creates an internal stage in Snowflake.

```sql
create or replace stage my_internal_stage;
```

> _Named stages are referenced using **@**_

The following statement lists all the files present in the internal named stage

## **4. External Stages**

**External stages are similar to Internal Named stages except that the files are stored in an external location outside Snowflake environment like AWS S3, Azure containers or GCP cloud storage.**

The External stages can be created using either the web interface or SQL. This requires a separate discussion in another article to understand how to create External stages in Snowflake.

**Read Complete Article:** **[How to create External Stages in Snowflake?](https://thinketl.com/how-to-create-external-stages-in-snowflake/)**

The following statement lists all the files present in the external stage

## **5. Snowflake Data Loading/Unloading commands**

Below commands are used in Snowflake for loading data into and unloading data from Snowflake stages.

1.  PUT
2.  GET
3.  COPY INTO

> **_GET_** _and **PUT** command cannot be executed from the Worksheet tab page in the Snowflake web interface. Instead, use the **SnowSQL** command line tool to upload data files from your local machine into Snowflake Stages._

**Related Article: [How to download and configure Snowflake SnowSQL?](https://thinketl.com/snowflake-snowsql-command-line-tool-to-access-snowflake/)**

### **5.1. PUT command**

PUT command in Snowflake uploads (i.e. stages) data files from a local folder on client machine into one of the following Snowflake stages

-   Internal Named stage.
-   Internal User Stage
-   Internal Table Stage.

Once files are staged, the data in the files can be loaded into a table using the **COPY INTO** command.

### **5.2. GET command**

GET Command in Snowflake downloads data files from one of the following Snowflake stages to a local folder on a client machine:

-   Internal Named stage.
-   Internal User Stage
-   Internal Table Stage.

> **_GET_** _and **PUT** commands are not supported with External stages._

### **5.3. COPY INTO command**

COPY INTO command in Snowflake loads data from staged files to an existing table and vice versa. The files are staged in one of the following locations.

-   Internal Stages( User, Table and Named)
-   External Stages built on Amazon S3, Google Cloud Storage, or Microsoft Azure.
-   External location (Amazon S3, Google Cloud Storage, or Microsoft Azure).

## **6. Loading data from local folder into Snowflake Stages using PUT command**

The following statement uploads a file named input_._csv on your local machine to your **User stage** and prefixes the file with a folder named my\_stage\_dir.

```sh
put file://C:\SourceFiles\input.csv @~/my_stage_dir;
```

The following statement uploads a file named input.csv on your local machine to the **Table stage** for the table named EMPLOYEE.

```sh
put file://C:\SourceFiles\input.csv @%EMPLOYEE;
```

The following statement uploads a file named input.csv on your local machine to a **Named** **internal stage** called my\_internal\_stage.

```sh
put file://C:\SourceFiles\input.csv @my_internal_stage;
```

## **7. Loading data from Snowflake Stages into tables using COPY INTO command**

### **7.1. Loading from User Stage into table**

The following statement loads data from file named input.csv prefixed with my\_stage\_dir in your **User stage** into table named EMPLOYEE.

```sql
copy into EMPLOYEE from @~/my_stage_dir/input.csv;
```

The following statement loads data from file named input.csv prefixed with my\_stage\_dir in your **User stage** into table named EMPLOYEE by specifying a named file format.

```sql
copy into EMPLOYEE
from @~/my_stage_dir/input.csv
file_format = (format_name = 'my_csv_format');
```

### **7.2. Loading from Table Stage into table**

The following statement loads data from file named input.csv in your **Table stage** into table named EMPLOYEE.

```sql
copy into EMPLOYEE from @%EMPLOYEE/input.csv;
```

The **FROM** clause can be omitted while loading from **Table stage** because Snowflake automatically checks for files in the table stage.

### **7.3. Loading from Internal Named Stage into table**

The following statement loads data from file named input.csv in your **Internal Named stage** named my\_internal\_stage into table named EMPLOYEE.

```sql
copy into EMPLOYEE from @my_internal_stage/input.csv;
```

The following statement loads data from file named input.csv in your **Internal Named stage** named my\_internal\_stage into table named EMPLOYEE by specifying a adhoc file format.

```sql
copy into EMPLOYEE
from @my_internal_stage/input.csv
file_format = (type = csv field_delimiter = ',' skip_header = 0);
```

### **7.4. Loading from External Stage into table**

The following statement loads data from file named input.csv in your **External stage** named my\_azure\_stage into table named EMPLOYEE.

```sql
copy into EMPLOYEE from @my_azure_stage/input.csv;
```

The data can be loaded from external locations into Snowflake directly without the use of stages. The following statement loads data from file named input.csv in your Azure location into table named EMPLOYEE.

```sql
copy into EMPLOYEE
  from 'azure://myazurespace.blob.core.windows.net/snowflake/input.csv'
  storage_integration = azure_int
  file_format = (format_name = my_csv_format);
```

## **8. Unloading data into Snowflake Stages from tables using COPY INTO command**

The following statement unloads data into your **User stage** and prefixes the file with a folder named my\_stage\_dir from table named EMPLOYEE.

```sql
copy into @~/my_stage_dir from EMPLOYEE;
```

The following statement unloads data into your **Table stage** from table named EMPLOYEE.

```sql
copy into @%EMPLOYEE from EMPLOYEE;
```

The following statement unloads data into your **Internal Named stage** named my\_internal\_stage from table named EMPLOYEE.

```sql
copy into @my_internal_stage from EMPLOYEE;
```

The following statement unloads data into your **External stage** named my\_azure\_stage from table named EMPLOYEE.

```sql
copy into @my_azure_stage from EMPLOYEE;
```

## **9. Unloading data into local folder from Snowflake Stages using GET command**

The following statement unloads data from your **User stage** into local machine.

```sh
get @~/my_stage_dir file://C:\OutputFiles\User;
```

The following statement unloads data from **Table stage** named EMPLOYEE into local machine.

```sh
get @%EMPLOYEE file://C:\OutputFiles\Table;
```

The following statement unloads data from **Internal Named stage** named my\_internal\_stage into local machine.

```sh
get @my_internal_stage file://C:\OutputFiles\NamedInternal;
```

## **10\. Removing files from Snowflake Stages**

The following statment removes all the files with pattern csv in the file name from your **User Stag**e.

> Be careful while running remove statement on your User stage. Without specifying any pattern you might risk deleting all your worksheets in Snowflake.

The following statment removes all the files from **Table Stage** named EMPLOYEE.

The following statment removes all the files with pattern csv in the file name from **Internal Named Stage** named my\_internal\_stage.

```sh
rm @my_internal_stage pattern='.*csv.*';
```

The following statment removes all the files with pattern csv in the file name from **External Stage** named my\_azure\_stage.

```sh
rm @my_azure_stage pattern='.*csv.*';
```
