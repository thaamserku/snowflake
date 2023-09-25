---
created: 2023-09-07T21:01:39 (UTC -04:00)
tags: []
source: https://thinketl.com/continuous-data-loading-and-monitoring-using-snowpipe/
author: ThinkETL
---

# Continuous Data Loading and Monitoring using Snowpipe - ThinkETL

> ## Excerpt
> Learn how to load files from external storage using the Snowpipe and monitor the load status of the files in Snowflake.

---
## **1\. Introduction**

Snowpipe is Snowflake’s continuous data ingestion service which enables loading data from files into database tables as soon as they are available in a stage.

In our previous articles, we have discussed about setting up **[Snowpipe on AWS](https://thinketl.com/introduction-to-snowflake-snowpipe-on-aws/)** and **[Snowpipe on Azure](https://thinketl.com/introduction-to-snowpipe-on-azure/)** for automatic data ingestion. In this article, let us focus on how to load the files from external storage using the Snowpipe and monitor the load status of these files in Snowflake.

For the demonstration, we will load the data from files in **AWS S3** location into Snowflake using

-   A Snowpipe named **MY\_SNOWPIPE.**
-   A database table named **EMPLOYEES** to which the data will be loaded.
-   The automatic data loads using Snowpipe are configured using **Event Notifications**.

The below image shows the source files present in the AWS S3 location to be loaded into Snowflake.

![AWS S3 Source Files](https://thinketl.com/wp-content/uploads/2023/05/127-1-AWS-S3-Source-Files.png)

AWS S3 Source Files

## **2\. Loading Historic files older than 7 days**

When a Snowpipe is created on top of an external stage, it will not process the files which are already existing in the location before the creation of Snowpipe. This is because the historic data files do not trigger event notifications and hence they are not picked up for ingestion by Snowpipe.

The files in the external storage location which are placed 7 days prior to the creation of the Snowpipe can be loaded manually by executing the **COPY INTO <table>** statement. This COPY INTO statement can be extracted from the DDL of the Snowpipe definition.

The below COPY INTO command loads the data from CSV files present in the Inbox folder of external stage MY\_S3\_STAGE into EMPLOYEES table.

```
COPY INTO EMPLOYEES
FROM @MY_S3_STAGE/Inbox
FILE_FORMAT = (TYPE = 'CSV' skip_header = 1);
```

The below image shows output of the **COPY INTO** statement which provides the loads status of the files processed into table.

![COPY INTO statement output](https://thinketl.com/wp-content/uploads/2023/05/127-2-COPY-INTO-statement-output.png)

COPY INTO statement output

The below image shows that a total of 1 million records are loaded into EMPLOYEES table.

![Employees table record count after processing Historical files](https://thinketl.com/wp-content/uploads/2023/05/127-3-Employess-table-record-count.png)

Employees table record count after processing Historical files

## **3\. Loading Historic files staged within the previous 7 days**

The files in the external storage location which are placed within 7 days prior to the creation of the Snowpipe can still be loaded using the Snowpipe by executing the **ALTER PIPE…REFRESH** statement.

The below SQL statement copies files staged within the previous 7 days to the Snowpipe ingest queue for loading into the target table.

```
ALTER PIPE MY_SNOWPIPE REFRESH;
```

## **4\. Loading Incremental files from External Stage using Snowpipe**

For the Snowpipe to automatically pick the files placed after its creation, an Event Notification should be configured on your external location to notify Snowpipe when new data is available to load.

Refer our previous articles for more details on setting up [**Event Notifications on AWS**](https://thinketl.com/introduction-to-snowflake-snowpipe-on-aws/#44_Create_AWS_S3_Event_Notification_to_Automate_Snowpipe) and [**Event Notifications on Azure**](https://thinketl.com/introduction-to-snowpipe-on-azure/#4_Setting_up_Azure_Storage_Event_Notifications) to Automate Snowpipe.

Once new files are placed in the external location, a configured event notification service informs Snowpipe that files are ready to load. Snowpipe copies the files into a queue and loads data from the queued files into the target table.

Let us place a new file in the same external location discussed earlier and verify how its load status can be tracked.

![Incremental file placed in external storage location](https://thinketl.com/wp-content/uploads/2023/05/127-4-Incremental-file-placed-in-external-storage-location.png)

Incremental file placed in external storage location

The below image shows that the total records in table increased from 1 million to 1.2 million after the new file is processed.

![Employees table record count after processing incremental files using Snowpipe](https://thinketl.com/wp-content/uploads/2023/05/127-5-Employess-table-record-count-after-loading-data-from-Snowpipe.png)

Employees table record count after processing incremental files using Snowpipe

## **5\. Monitoring Load status of Snowpipe**

There are several ways through which the files processed through Snowpipe can be tracked as listed below.

-   PIPE\_USAGE\_HISTORY
-   COPY\_HISTORY
-   LOAD\_HISTORY
-   Snowsight Copy History Page

### **5.1. PIPE\_USAGE\_HISTORY**

**PIPE\_USAGE\_HISTORY** is a table function that can be used to query the history of data loaded into Snowflake tables using Snowpipe within a specified date range.

> **_PIPE\_USAGE\_HISTORY_** _table function returns pipe activity within the last **14** days._

The following are the optional parameters that can be passed along with the function to narrow the search of the load history.

-   DATE\_RANGE\_START
-   DATE\_RANGE\_END
-   PIPE\_NAME

The below SQL query returns the data load history of your account through Snowpipe for a 30 minute range using PIPE\_USAGE\_HISTORY table function.

```
SELECT * FROM TABLE(INFORMATION_SCHEMA.PIPE_USAGE_HISTORY(
    DATE_RANGE_START=>TO_TIMESTAMP_TZ('2023-05-14 16:00:00.000 +0530'),
    DATE_RANGE_END=>TO_TIMESTAMP_TZ('2023-05-14 16:30:00.000 +0530'),
    PIPE_NAME=>'MY_SNOWPIPE'));
```

The below SQL query returns the data load history of of your account through Snowpipe of last 1 hour using PIPE\_USAGE\_HISTORY table function.

```
SELECT * FROM TABLE(INFORMATION_SCHEMA.PIPE_USAGE_HISTORY(
    DATE_RANGE_START=>DATEADD('HOUR',-1,CURRENT_DATE()),
    PIPE_NAME=>'MY_SNOWPIPE'));
```

The below image shows the output of the PIPE\_USAGE\_HISTORY queries provided above.

![PIPE_USAGE_HISTORY output](https://thinketl.com/wp-content/uploads/2023/05/127-6-PIPE_USAGE_HISTORY-output.png)

PIPE\_USAGE\_HISTORY output

### **5.2. COPY\_HISTORY**

**COPY\_HISTORY** is a table function that can be used to query the history of data loaded into Snowflake tables through both bulk data loading using **COPY INTO** statements and continuous data loading using **Snowpipe** within a specified date range.

> **_COPY\_HISTORY_** _table function returns copy activity within the last **14** days._

The following are the parameters that can be passed along with the function to narrow the search of the copy activity.

-   TABLE\_NAME
-   START\_TIME
-   END\_TIME (optional)

The below SQL query returns the data load history of last 1 hour of your account through Snowpipe using COPY\_HISTORY table function.

```
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME=>'EMPLOYEES',
    START_TIME=> DATEADD(HOURS, -1, CURRENT_TIMESTAMP())));
```

![COPY_HISTORY output](https://thinketl.com/wp-content/uploads/2023/05/127-7-COPY_HISTORY-output.png)

COPY\_HISTORY output

### **5.3. LOAD\_HISTORY**

**LOAD\_HISTORY** is an Information Schema View that can be used to query the history of data loaded into Snowflake tables using COPY INTO statements.

> **_LOAD\_HISTORY_** _view returns copy activity within the last **14** days._  
> _This view does not return the history of data loaded using Snowpipe._

The below SQL query returns the history of data loaded into the EMPLOYEES table using LOAD\_HISTORY view.

```
SELECT *
  FROM INFORMATION_SCHEMA.LOAD_HISTORY
  WHERE TABLE_NAME = 'EMPLOYEES'
  ORDER BY LAST_LOAD_TIME DESC;
```

![LOAD_HISTORY output](https://thinketl.com/wp-content/uploads/2023/05/127-8-LOAD_HISTORY-output.png)

LOAD\_HISTORY output

Note that the historical data for COPY INTO commands is removed from the view when a table is dropped.

### **5.4. Snowsight Copy History Page**

The Snowsight Copy History page provides details of both bulk data loading activity using **COPY INTO** statements and continuous data loading using **Snowpipe**. The Copy History page provides a detailed table of data loads for your tables.

> **_Copy History page_** _provides details of data loading activity that has occurred over the last **365** days for all tables in your account._

To access data load history information in Snowsight,

-   Login to Snowsight > Activity > Copy History

The below image shows the data load information from the Copy History page in Snowsight.

![Snowsight Copy History Page](https://thinketl.com/wp-content/uploads/2023/05/127-9-Snowsight-Copy-History-Page.png)

Snowsight Copy History Page

## **6\. Summary**

To summarize everything related to loading files using Snowpipe.

-   The Historical files which are older than 7 days cannot be processed by Snowpipe. They should be manually processed using **COPY INTO** statement.
-   The Historical files which are placed within the previous 7 days can be processed by executing the **ALTER PIPE…REFRESH** statement.
-   The Incremental files are processed by Snowpipe automatically by setting up an Event Notification service which notifies Snowpipe when new files are arrived.

The below table summarizes the details related to monitoring load status of Snowpipe

<table><tbody><tr><td><strong>Monitoring Method</strong></td><td><strong>Type</strong></td><td><strong>Captures load status of COPY INTO statement</strong></td><td><strong>Captures load status of SNOWPIPE</strong></td><td><strong>No of days of history captured</strong></td></tr><tr><td>PIPE_USAGE_HISTORY</td><td>Table Function</td><td>No</td><td>Yes</td><td>14</td></tr><tr><td>COPY_HISTORY</td><td>Table Function</td><td>Yes</td><td>Yes</td><td>14</td></tr><tr><td>LOAD_HISTORY</td><td>View</td><td>Yes</td><td>No</td><td>14</td></tr><tr><td>Copy History Page</td><td>Snowsight UI feature</td><td>Yes</td><td>Yes</td><td>365</td></tr></tbody></table>

**Subscribe to our Newsletter !!**

**Related Articles:**

-   [![Introduction to Snowflake SQL REST API using Postman](https://thinketl.com/wp-content/uploads/2023/04/Introduction-to-Snowflake-SQL-REST-API-using-Postman.png)](https://thinketl.com/introduction-to-snowflake-sql-rest-api-using-postman/)
    
    Snowflake SQL REST API allows users to interact with Snowflake through HTTP requests, making it easy to integrate with other systems.
    
    [**READ MORE**](https://thinketl.com/introduction-to-snowflake-sql-rest-api-using-postman/)
    
-   [![Execute multiple SQL statements in a single Snowflake API request](https://thinketl.com/wp-content/uploads/2023/04/Execute-multiple-SQL-statements-in-a-single-Snowflake-API-request.png)](https://thinketl.com/execute-multiple-sql-statements-in-a-single-snowflake-api-request/)
    
    Learn how to submit an API request containing multiple statements to execute to the Snowflake SQL REST API using Postman.
    
    [**READ MORE**](https://thinketl.com/execute-multiple-sql-statements-in-a-single-snowflake-api-request/)
    
-   [![Introduction to Snowflake Snowpipe on AWS](https://thinketl.com/wp-content/uploads/2023/05/Introduction-to-Snowflake-Snowpipe-on-AWS.png)](https://thinketl.com/introduction-to-snowflake-snowpipe-on-aws/)
    
    A step by step guide on automating continuous data loading into Snowflake through Snowpipe on AWS S3.
    
    [**READ MORE**](https://thinketl.com/introduction-to-snowflake-snowpipe-on-aws/)
#capture