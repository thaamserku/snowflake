---
created: 2023-09-07T21:02:43 (UTC -04:00)
tags: []
source: https://thinketl.com/change-data-capture-using-snowflake-streams/
author: ThinkETL
---

# Change Data Capture using Snowflake Streams - ThinkETL

> ## Excerpt
> A Stream is a Snowflake object that provides change data capture (CDC) capabilities to track the changes in a table.

---
## **Introduction**

Change Data Capture (CDC) is a process that identifies and captures changes made to data in a database and then delivers those changes in real-time to a downstream process or system. There are several proven methods through which CDC can be implemented like making use of Audit Columns to identify the data that has been modified since the data last extracted.

In this article let us discuss about Snowflake’s own offering which provides CDC capabilities.

## **What are Snowflake Streams?**

A Stream is a Snowflake object that provides Change Data Capture (CDC) capabilities to track the changes made to tables including inserts, updates, and deletes, as well as metadata about each change, so that actions can be taken using the changed data. 

> _A Stream creates a “**Change table**” which tracks all the DML changes at the row level, between two transactional points of time in a table. The Change table mirrors the column structure of the tracked source object and includes additional metadata columns that describe each change event._

Streams can be created to query change data on the following objects.

-   Standard tables, including shared tables.
-   Views, including secure views
-   Directory tables
-   External tables

Below are the additional metadata fields included in Streams along with Source object fields to track changes

| **COLUMN NAME** | **DESCRIPTION** |
| --- | --- |
| **METADATA$ACTION** | Indicates the DML operation (**INSERT, DELETE**) recorded. |
| **METADATA$ISUPDATE** | Indicates whether the operation was part of an **UPDATE** statement. Updates to rows in the source object are **_represented as a pair of DELETE and INSERT_** records in the stream with a metadata column METADATA$ISUPDATE values set to TRUE. |
| **METADATA$ROW\_ID** | Specifies the unique and immutable ID for the row, which can be used to track changes to specific rows over time. |

## **Stream** **Offset**

Streams work by keeping track of an **offset**, a pointer which indicates the point in time since you last read the stream which is referred as a transaction.

> _Once the data in the Stream is consumed by a downstream table or a data pipeline in a **DML** transaction, the change data is no longer available in Stream. Only the subsequent changes made on the table are captured by Stream until they are again consumed by a consumer._

As stream advances its offset only when it is involved in a DML transaction, the data consumed by streams is very minimal. Therefore, you can create any number of streams for an object without incurring significant cost. Hence Snowflake also recommends creating separate stream for each consumer.

## **Types of Snowflake Streams**

There are three different types of Streams supported in Snowflake

1.  Standard
2.  Append-only
3.  Insert-only

### **1\. Standard Streams**

A Standard (i.e. delta) stream tracks all DML changes to the source object, including inserts, updates, and deletes (including table truncates). Supported on standard tables, directory tables and views.

The syntax to create Standard stream is as below

```
CREATE OR REPLACE STREAM my_stream ON TABLE my_table;
```

### **2\. Append-only Streams**

An Append-only stream tracks row inserts only. Update and delete operations (including table truncates) are not recorded. Supported on standard tables, directory tables and views.

The syntax to create Append-only streams similar to Standard streams except that the APPEND\_ONLY parameter value needs to be set to TRUE as below

```
CREATE OR REPLACE STREAM my_stream ON TABLE my_table
APPEND_ONLY = TRUE;
```

### **3\. Insert-only Streams**

Supported for **External tables** only. An insert-only stream tracks row inserts only. They do not record delete operations that remove rows from an inserted set.

The syntax to create Insert-only stream is as below

```
CREATE OR REPLACE STREAM my_stream ON EXTERNAL TABLE my_table
INSERT_ONLY = TRUE;
```

## **Demo on how Snowflake Streams Work**

Let us understand how streams work through an example. Consider a table named EMPLOYEES\_RAW where the raw data is staged and the data finally needs to be loaded into EMPLOYEES tables.

The table named EMPLOYEES\_RAW is created and three records are inserted.

```
--create employees_raw table
CREATE OR REPLACE TABLE EMPLOYEES_RAW(
    ID NUMBER,
    NAME VARCHAR(50),
    SALARY NUMBER
);

--Insert three records into table
INSERT INTO EMPLOYEES_RAW VALUES (101,'Tony',25000);
INSERT INTO EMPLOYEES_RAW VALUES (102,'Chris',55000);
INSERT INTO EMPLOYEES_RAW VALUES (103,'Bruce',40000);
```

Let us create EMPLOYEES table and the for the first load lets copy data from EMPLOYEES\_RAW directly using Insert statement.

```
--create employees table
CREATE OR REPLACE TABLE EMPLOYEES(
    ID NUMBER,
    NAME VARCHAR(50),
    SALARY NUMBER
);

--Inserting initial set of records from raw table
INSERT INTO EMPLOYEES SELECT * FROM EMPLOYEES_RAW;
```

The data now in EMPLOYEES\_RAW and EMPLOYEES table are in sync. Let us make few changes in data of raw table and track the changes through a Stream.

```
--create stream
CREATE OR REPLACE STREAM MY_STREAM ON TABLE EMPLOYEES_RAW;
```

Initially when you query a stream, it will return null records as there are no DML operations performed yet on the raw table.

![](https://thinketl.com/wp-content/uploads/2022/06/79-1-Stream.png)

Let us insert two records and update two records in the raw table and verify the contents of the stream MY\_STREAM.

```
--Insert two records
INSERT INTO EMPLOYEES_RAW VALUES (104,'Clark',35000);
INSERT INTO EMPLOYEES_RAW VALUES (105,'Steve',30000);

--Update two records
UPDATE EMPLOYEES_RAW SET SALARY = '50000' WHERE ID = '102';
UPDATE EMPLOYEES_RAW SET SALARY = '45000' WHERE ID = '103';
```

![](https://thinketl.com/wp-content/uploads/2022/06/79-2-Stream.png)

In the Stream we can observe that

-   Employee records with ID 104 and 105 are inserted.  
    The **METADATA$ACTION** for these records is set as **INSERT** and **METADATA$UPDATE** is set as **FALSE**.

-   The employee records with IDs 102 and 103 which got updated have two set of records, one with **METADATA$ACTION** set as **INSERT** and the other as **DELETE**.  
    The field **METADATA$UPDATE** is set as **TRUE** for both the records indicating that these records are part of UPDATE operation.

Before consuming the data from stream, let us also perform another DML operation on already modified data and see how stream updates its data.

```
--Delete one record
DELETE FROM EMPLOYEES_RAW WHERE ID = '102';
```

![](https://thinketl.com/wp-content/uploads/2022/06/79-3-Stream.png)

> _Note that streams record the differences between two offsets. If a row is updated and then deleted in the current offset, the delta change is a delete row._ 

The same can be noticed for the employee record with ID = 102 where the data is first updated and then the row is deleted. So the stream only capture the delta change which is delete.

## **Consuming data from a Snowflake Stream**

In the above discussed example, though we have inserted 2 records, updated 2 records and deleted 1 record, the final changes required to be captured from raw table are 2 inserts, 1 update and 1 delete.

The below select statements on the stream gives the details of records which needs to be inserted, updated and deleted in the target table.

```
--INSERT
SELECT * FROM MY_STREAM
WHERE metadata$action = 'INSERT'
AND metadata$isupdate = 'FALSE';

--UPDATE
SELECT * FROM MY_STREAM
WHERE metadata$action = 'INSERT'
AND metadata$isupdate = 'TRUE';

--DELETE
SELECT * FROM MY_STREAM
WHERE metadata$action = 'DELETE'
AND metadata$isupdate = 'FALSE';
```

Finally we can use a **MERGE** statement with the Stream using these filters to perform the insert, update and delete operations on target table as shown below.

```
MERGE INTO EMPLOYEES a USING MY_STREAM b ON a.ID = b.ID
  WHEN MATCHED AND metadata$action = 'DELETE' AND metadata$isupdate = 'FALSE' 
    THEN DELETE
  WHEN MATCHED AND metadata$action = 'INSERT' AND metadata$isupdate = 'TRUE' 
    THEN UPDATE SET a.NAME = b. NAME, a.SALARY = b.SALARY
  WHEN NOT MATCHED AND metadata$action = 'INSERT' AND metadata$isupdate = 'FALSE'
    THEN INSERT (ID, NAME, SALARY) VALUES (b.ID, b.NAME, b.SALARY)
;
```

The below image show that the merge operation inserted 2 records, updated 1 record and deleted 1 record as expected.

![](https://thinketl.com/wp-content/uploads/2022/06/79-4-Merge.png)

> _Now that we have consumed the stream in a DML transaction, the stream now do not return any records and is set to new offset. So if you need to consume for multiple downstream systems from a stream, build multiple streams on the table, one for each consumer._

## **Stream Staleness**

A stream becomes stale when its offset is outside of the data retention period for its source table. When a stream becomes stale, the historical data for the source table is no longer accessible, including any unconsumed change records.

To view the current staleness status of a stream, execute the **DESCRIBE STREAM** or **SHOW STREAMS** command. The **STALE\_AFTER** column timestamp indicates when the stream is currently predicted to become stale.

![](https://thinketl.com/wp-content/uploads/2022/06/79-5-Show-streams.png)

> _To avoid having a stream become stale, it is strongly recommended to regularly consume the changed data before its **STALE\_AFTER** timestamp._

If the data retention period for a table is **less than 14 days**, and a stream has not been consumed, Snowflake temporarily extends this period to prevent it from going stale. The period is extended to the stream’s offset, up to a maximum of 14 days by default, regardless of the Snowflake edition for your account.

Note that this restriction does not apply to streams on **Directory tables** or **External tables**, which have no data retention period.

## **Conclusion**

To summarize, Snowflake streams are a powerful way to handle changing data sets. Streams can be queried just like a table which makes it easy to grab the new data in a table. A stream only stores an offset for the source object and returns CDC records by leveraging the versioning history for the source object.

Snowflake table streams are also often used in conjunction with other features, such as Snowflake [**Snowpipe**](https://thinketl.com/introduction-to-snowpipe-on-azure/) and Snowflake tasks. Be sure to check out our other blog posts for more details about Snowflake features.

**Subscribe to our Newsletter !!**

**Related Articles :**

-   [![Snowflake Zero Copy Cloning](https://thinketl.com/wp-content/uploads/2022/05/zero-copy-cloning.png)](https://thinketl.com/snowflake-zero-copy-cloning/)
    
    Snowflake’s Zero Copy Cloning feature is a quick and easy way to create copies of database objects without incurring any additional costs.
    
    [**READ MORE**](https://thinketl.com/snowflake-zero-copy-cloning/)
    
-   [![Introduction to Snowpipe on Azure](https://thinketl.com/wp-content/uploads/2022/05/Snowpipe.png)](https://thinketl.com/introduction-to-snowpipe-on-azure/)
    
    A step by step guide on automating continuous data loading into Snowflake through Snowpipe on Microsoft Azure.
    
    [**READ MORE**](https://thinketl.com/introduction-to-snowpipe-on-azure/)
    
-   [![Types of Views in Snowflake](https://thinketl.com/wp-content/uploads/2022/05/TYPES-OF-SNOWFLAKE-VIEWS.png)](https://thinketl.com/types-of-views-in-snowflake/)
    
    There are three different types of views in Snowflake – Non-Materialized, Materialized and Secure Views.
    
    [**READ MORE**](https://thinketl.com/types-of-views-in-snowflake/)
#capture