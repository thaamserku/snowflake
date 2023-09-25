---
created: 2023-09-24T12:20:28 (UTC -04:00)
tags: []
source: https://thinketl.com/snowflake-information-schema/
author: ThinkETL
---

# Snowflake Information Schema - ThinkETL

> ## Excerpt
> The INFORMATION_SCHEMA is a read-only schema available automatically under each database. It stores metadata of all the objects built under the database.

---
## **1\. Introduction**

Snowflake like any good database keeps track of all the defined objects within it and their associated metadata. In order to make it simple for users to inspect some of the information about the databases, schemas, and tables in the Snowflake, the metadata information is supplied as a collection of views against the metadata layer.

Information Schema is one such offering from Snowflake that provides extensive metadata information about the objects created in your account. 

The Snowflake Information Schema is a Data Dictionary schema available as a read-only schema named **INFORMATION\_SCHEMA** under each database created automatically by Snowflake. 

It consists of a set of **system-defined views** and **table functions** that provide extensive metadata information about the objects created in your account. 

## **3\. Snowflake Information Schema Views**

The views in INFORMATION\_SCHEMA display metadata about objects defined in the database, as well as metadata for non-database, account-level objects that are common across all databases.

-   There are **17** views available under INFORMATION\_SCHEMA that holds information of Database level objects.
-   There are **8** views that holds information of Account level objects.

> _The Snowflake Information Schema views are ANSI-standard._

![INFORMATION_SCHEMA available under a database named COVID19](https://thinketl.com/wp-content/uploads/2022/09/94-1-Information-Schema-Views.png)

INFORMATION\_SCHEMA available under a database named COVID19

Below are the various views available under INFORMATION\_SCHEMA

### **3.1. Databases, Schema, Tables and Views**

**DATABASES:** The databases that are accessible to the current user’s role.

**SCHEMATA:** The schemas defined in this database that are accessible to the current user’s role.

**TABLES:** The tables defined in this database that are accessible to the current user’s role.

**VIEWS:** The views defined in this database that are accessible to the current user’s role.

**TABLE\_STORAGE\_METRICS:** All tables within an account, including expired tables.

**INFORMATION\_SCHEMA\_CATALOG\_NAME:** Returns the name of the database in which Information\_schema resides.

### **3.2. Sequences and File Formats**

**SEQUENCES:** The sequences defined in this database that are accessible to the current user’s role.

**FILE\_FORMATS:** The file formats defined in this database that are accessible to the current user’s role.

### **3.3. Stages, External Tables and Pipes**

**STAGES:** Stages in this database that are accessible by the current user’s role.

**EXTERNAL\_TABLES:** The external tables defined in this database that are accessible to the current user’s role.

**PIPES:** The pipes defined in this database that are accessible to the current user’s role.

**LOAD\_HISTORY:** The history of data loaded into tables using the COPY INTO command with in last 14 days.

### **3.4. UDFs, Procedures and Packages**

**PROCEDURES:** The stored procedures defined in this database that are accessible to the current user’s role.

**FUNCTIONS:** The user-defined functions defined in this database that are accessible to the current user’s role.

**PACKAGES:** Available packages in current account.

### **3.5. Roles, Object Privileges and Usage Privileges**

**APPLICABLE\_ROLES:** The roles that can be applied to the current user.

**ENABLED\_ROLES:** The roles that are enabled to the current user.

**TABLE\_PRIVILEGES:** The privileges on tables defined in this database that are accessible to the current user’s role.

**USAGE\_PRIVILEGES:** The usage privileges on sequences defined in this database that are accessible to the current user’s role.

**OBJECT\_PRIVILEGES:** The privileges on all objects defined in this database that are accessible to the current user’s role.

### **3.6. Columns and Constraints**

**COLUMNS:** The columns of tables defined in this database that are accessible to the current user’s role.

**REFERENTIAL\_CONSTRAINTS:** Referential Constraints in this database that are accessible to the current user

**TABLE\_CONSTRAINTS:** Constraints defined on the tables in this database that are accessible to the current user

### **3.7. Replication Groups and Databases**

**REPLICATION\_DATABASES:** The databases for replication that are accessible to the current user’s role.

**REPLICATION\_GROUPS**: The replication groups that are accessible to the current user’s role.

## **4\. Snowflake Information Schema Views Usage**

Below are the few examples of the usage of Snowflake Information Schema views

### **4.1. Get the list of all tables in a Snowflake Schema**

```
SELECT table_name, table_type
FROM my_db.information_schema.tables
WHERE table_schema = 'my_schema'
ORDER BY table_name;
```

### **4.2. Get the Top 5 Table names with highest record count in each Snowflake Schema**

```
SELECT table_schema, table_name, row_count
FROM(
    SELECT
        table_schema, table_name, row_count ,
        dense_rank() over(partition by table_schema order by row_count desc) as rnk
    FROM my_db.information_schema.tables
    WHERE row_count IS NOT NULL
  )
WHERE rnk <=5
ORDER BY table_schema, row_count desc;
```

### **4.3. Get the size (in bytes) of all tables in all schemas in a Snowflake database**

```
SELECT table_schema,sum(bytes)
FROM my_db.information_schema.tables
GROUP BY table_schema;
```

### **4.4. Get the time travel duration of all tables in a Snowflake Schema**

```
SELECT table_name, retention_time
FROM my_db.information_schema.tables
WHERE table_schema = 'my_schema'
ORDER BY table_name;
```

### **4.5. Get the list of all Primary Keys in all tables present in a Snowflake Schema**

```
SELECT table_name, constraint_type, constraint_name
FROM my_db.information_schema.table_constraints
WHERE constraint_type = 'PRIMARY KEY' and table_schema = 'my_schema'
ORDER BY table_name;
```

### **4.6. Generate SQL statements to change ownership on all tables owned by a role to a new role in Snowflake**

```
SELECT 'grant ownership on table ' || table_name || ' to role my_new_role copy grants;' AS grant_statement
FROM my_db.information_schema.table_privileges
WHERE grantor = 'old_grant_role';
```

### **4.7. Generate SQL statements to drop all tables in a Snowflake schema**

```
SELECT 'drop table ' || table_name || ' cascade;'
FROM my_db.information_schema.tables
WHERE table_schema = 'my_schema'
ORDER BY table_name;
```

## **5\. Snowflake Information Schema Table Functions**

The table functions in INFORMATION\_SCHEMA can be used to return account-level usage and historical information for storage, warehouses, user logins, and queries.

Like Information Schema Views, the table functions are not visible directly under Information Schema. For more details about table functions refer [Snowflake Documentation](https://docs.snowflake.com/en/sql-reference/info-schema.html#list-of-table-functions).

## **6\. SHOW Commands vs Information Schema**

The same data presented by the SHOW <objects> commands is also available through a SQL interface using the INFORMATION SCHEMA views.

The SHOW commands can be replaced by the views, but before switching, you should be aware of the following significant differences:

-   To query the Information Schema views, the warehouse must be operating and actively being used where as it is not required for SHOW commands.
-   While Information Schema views display all items in the current specified database, most SHOW commands by default limit results to the current schema.

## **7\. Summary**

INFORMATION\_SCHEMA is a read-only schema available automatically under each database. It stores metadata of all Snowflake objects built under the database.

Running Queries on INFORMATION\_SCHEMA requires warehouse to be up and running which incurs Snowflake credits.

The output of a view or table function depend on the privileges granted to the user’s current role. When querying an INFORMATION\_SCHEMA view or table function, only objects for which the current role has been granted access privileges are returned.

**Subscribe to our Newsletter !!**

**Related Articles:**

-   [![Overview of Snowflake Role Based Access Control](https://thinketl.com/wp-content/uploads/2022/07/SNOWFLAKE-ROLE-BASED-ACCESS-CONTROL-1.png)](https://thinketl.com/overview-of-snowflake-role-based-access-control/)
    
    Learn various system define roles in Snowflake, their purpose and how you can create custom roles and Role Hierarchy.
    
    [**READ MORE**](https://thinketl.com/overview-of-snowflake-role-based-access-control/)
    
-   [![Snowflake File Formats](https://thinketl.com/wp-content/uploads/2022/07/Snowflake-FileFormats.png)](https://thinketl.com/snowflake-file-formats/)
    
    Snowflake File format simplifies the process of accessing the staged data and streamlines data loading/unloading in database tables.
    
    [**READ MORE**](https://thinketl.com/snowflake-file-formats/)
    
-   [![HOW TO: Load and Query JSON data in Snowflake?](https://thinketl.com/wp-content/uploads/2022/08/Loading-and-Querying-JSON-data.png)](https://thinketl.com/how-to-load-and-query-json-data-in-snowflake/)
    
    Snowflake supports loading JSON data in database tables and allows querying data along with flattening it into a columnar structure.
    
    [**READ MORE**](https://thinketl.com/how-to-load-and-query-json-data-in-snowflake/)
#capture