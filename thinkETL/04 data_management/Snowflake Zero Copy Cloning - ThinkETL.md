---
created: 2023-09-24T12:20:18 (UTC -04:00)
tags: []
source: https://thinketl.com/snowflake-zero-copy-cloning/
author: ThinkETL
---

# Snowflake Zero Copy Cloning 

> Snowflake’s Zero Copy Cloning feature is a quick and easy way to create copies of database objects without incurring any additional costs.

## **Introduction**

It is often periodically required to replicate real time data from production environment into your dev or staging environments to have accurate unit test results for the latest changes planned in the project. One of the biggest difficulties in traditional databases is replicating your database objects from one environment to another. It involves proper planning, additional storage costs and long waiting times for the entire process to complete.

Snowflake simplifies the process of replicating your data without incurring any additional storage costs or long waiting times through Zero Copy Cloning.

## **What is Zero Copy Cloning in Snowflake?**

> **Snowflake’s Zero Copy Cloning is a feature which provides a quick and easy way to create a copy of any table, schema, or an entire database without incurring any additional costs as the derived copy shares the underlying storage with the original object.** 

The most powerful feature of Zero Copy Cloning is that _the cloned and original objects are independent from each other_ i.e., any changes done on either of the objects do not impact others. Until you make any changes, the cloned object shares the same storage as original. This can be quite useful for quickly producing backups that don’t cost anything extra until the copied object is changed.

> **However, any changes done on cloned snapshot creates additional storage components which results in additional costs.**

Cloning in Snowflake is much faster than cloning in other databases. The days of waiting an entire day or two for an environment provision are long gone. Depending on the size of the database objects, it could only take few seconds to several minutes.

## **How does Snowflake’s Zero Copy Cloning works?**

All data in Snowflake tables is automatically divided into micro-partitions, which are smallest continuous units of storage. Each micro-partition contains between 50 MB and 500 MB of uncompressed data (Note: The actual size in Snowflake is smaller because data is always stored compressed). This Micro-partitioning is automatically performed on all Snowflake tables. 

When a database object is cloned, Snowflake creates new metadata information pointing the micro-partitions of the original source object instead of creating copies of existing micro-partitions. Hence the name Zero copy cloning. All these operations are performed by Snowflake’s Cloud Services Layer and no intervention from the user is required.

> **Creates copies of database objects without actually copying the data.**

Let us understand this with an example

Consider a database table TABLE-A and its cloned snapshot TABLE-B.

The below image shows that the Snowflake’s metadata layer in which the metadata of TABLE-A is pointing to the micro-partitions in storage layer. The clone TABLE-B is nothing more than a new set of metadata pointing to the same micro-partitions that store TABLE-A data.

![Zero-Copy Cloning illustration](https://thinketl.com/wp-content/uploads/2022/05/Zero-copy-cloning-illustration.png)

Zero-Copy Cloning illustration

> **_Micro-partitions in Snowflake are immutable i.e., once created lasts in the same state until table is dropped. Hence, for any change in the data of a micro-partition, a new micro-partition is created with updated changes and metadata will point to the newly created micro partition. The older partition is retained for time-travel and Fail-safe purposes._**

Now consider you have made changes by updating data which belongs to micro-partition MP-3 in TABLE-B. The changes are captured in a new micro-partition MP-4 which is now referenced only by TABLE-B. So, additional storage costs are levied only for the modified data but not for the complete clone.

![Illustration showing change in cloned data](https://thinketl.com/wp-content/uploads/2022/05/Change-in-cloned-data-illustration.png)

Illustration showing change in cloned data

## **Which objects are supported in Snowflake Zero Copy Cloning?**

Before we get into understanding how to clone an object in Snowflake, it is worth considering what objects are supported for cloning. Below is the list of all objects which can be cloned in Snowflake.

-   **Data Containment Objects**
    -   Databases
    -   Schemas
    -   Tables
    -   Streams  
        
-   **Data Configuration and Transformation Objects**
    -   Stages
    -   File Formats
    -   Sequences
    -   Tasks

**Note**: Internal named stages cannot be cloned.

## **How to Clone objects in Snowflake?**

Cloning objects in Snowflake is simple and can be achieved using a simple SQL statement as shown below.

```sql
CREATE <OBJECT_TYPE> <OBJECT_NAME>
  CLONE <SOURCE_OBJECT_NAME>
```

The above one is a simplified version. The complete syntax to clone a Snowflake object is as shown below.

```sql
CREATE [ OR REPLACE ] { DATABASE | SCHEMA | TABLE | STREAM | STAGE | FILE FORMAT | SEQUENCE | TASK } [ IF NOT EXISTS ] <OBJECT_NAME>
  CLONE <SOURCE_OBJECT_NAME>
```

The above statements will create a new object cloned from a source object. Though the metadata of the cloned object and the underlying data it is holding is same as the source object, both objects have their own life cycle and are independent from each other.

## **Snowflake Cloning with Time Travel**

Snowflake time-travel is another great feature of which enables accessing historical data that has been changed or deleted at any point within a defined period.

If you accidentally dropped a table or some of the records in a table, they can be recovered back with in the configured time-travel period of the table. This is achieved using the **AT** or **BEFORE** clauses, which supports specific timestamps, offsets from the current time or statement IDs to use as a reference. 

This syntax to clone a table with Time travel as shown below.

```sql
CREATE [ OR REPLACE ] { DATABASE | SCHEMA | TABLE | STREAM } [ IF NOT EXISTS ] <OBJECT_NAME>
  CLONE <SOURCE_OBJECT_NAME>
          [ { AT | BEFORE } ( { TIMESTAMP => <TIMESTAMP> | OFFSET => <TIME_DIFFERENCE> | STATEMENT => <ID> } ) ]
```

> **This enables you to clone a table as it was at a specific period of time or as it was a few minutes/hours ago.**

The below are few examples showing how to clone tables using time-travel in Snowflake

```sql
CREATE TABLE EMPLOYEE_CLONE CLONE EMPLOYEE
  AT(OFFSET => -60*5);

CREATE TABLE EMPLOYEE_CLONE CLONE EMPLOYEE
  AT(TIMESTAMP => 'SUN, 06 MAR 2022 13:45:00 +0530'::TIMESTAMP_TZ);
```

We have discussed in detail about Snowflake Time-Travel feature in a separate article. Read the complete article **[here](https://thinketl.com/overview-of-snowflake-time-travel/)** for more details.

## **Summary**

That is all about Snowflake Zero Copy Cloning. It is a one of the great Snowflake features which is simple to understand and easy to implement.

To summarize everything we have discussed, in short, when a table is cloned in Snowflake, the clone utilizes no additional data storage because it shares all the existing micro-partitions of the original table at the time it was cloned. However, rows can then be added, deleted, or updated in the clone independently from the original table. Each change to the clone results in new micro-partitions that are owned exclusively by the clone and are protected through CDP.

