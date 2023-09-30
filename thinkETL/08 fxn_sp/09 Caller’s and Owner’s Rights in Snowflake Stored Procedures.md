---
created: 2023-09-04T15:14:11 (UTC -04:00)
tags: []
source: https://thinketl.com/callers-and-owners-rights-in-snowflake-stored-procedures/
author: ThinkETL
---

# Caller’s and Owner’s Rights in Snowflake Stored Procedures

## **1\. Introduction**

The stored procedures in Snowflake runs either with caller’s rights or the owner’s rights which helps in defining the privileges with which the statements in the stored procedure executes. By default, when a stored procedure is created in Snowflake without specifying the rights with which it should be executed, it runs with owner’s rights.

In this article let us discuss what are caller’s rights and owner’s rights, the differences between the both and how to implement them in Snowflake stored procedures.

A caller’s rights stored procedure runs with the privileges of the role that called the stored procedure. The term “Caller” in this context refers to the user executing the stored procedure, who may or may not be the creator of the procedure.

**Any statement that the caller could not execute outside the stored procedure cannot be executed inside the stored procedure with caller’s rights.**

At the time of creation of stored procedure, the creator has to specify if the stored procedure runs with caller’s rights. The default is owner’s rights.

The syntax to create a stored procedure with caller’s rights is as shown below.

```sql
CREATE OR REPLACE PROCEDURE <procedure_name>()
  RETURNS <data_type>
  LANGUAGE SQL
  EXECUTE AS CALLER
AS
$$
  …
$$;
```

## **3\. Owner’s Rights in Snowflake Stored Procedures**

An Owner’s rights stored procedure runs with the privileges of the role that created the stored procedure. The term “Owner” in this context refers to the user who created the stored procedure, who may or may not be executing the procedure.

**The primary advantage of Owner’s rights is that the owner can delegate the privileges to another role through stored procedure without actually granting privileges outside the procedure.**

For example, if a user do not have access to clean up data in a table is granted access to a stored procedure (with owner’s rights) which does it. **The user who do not have any privileges on table can clean up the data in the table by executing the stored procedure**. But the same statements in the procedure when executed outside the procedure, cannot be executed by the user.

The syntax to create a stored procedure with owner’s rights is as shown below.

```sql
CREATE OR REPLACE PROCEDURE <procedure_name>()
  RETURNS <data_type>
  LANGUAGE SQL
  EXECUTE AS OWNER
AS
$$
  …
$$;
```

Note “**EXECUTE AS OWNER**” is optional. Even if the statement is not specified, the procedure is created with owner’s rights.

## **4\. Difference between Caller’s and Owner’s Rights in Snowflake**

The below are the differences between Caller’s and Owner’s Rights in Snowflake.

<table><tbody><tr><td><strong>Caller’s Rights</strong></td><td><strong>Owner’s Rights</strong></td></tr><tr><td>Runs with the privileges of the caller.</td><td>Runs with the privileges of the owner.</td></tr><tr><td>Inherit the current warehouse of the caller.</td><td>Inherit the current warehouse of the caller.</td></tr><tr><td>Use the database and schema that the caller is currently using.</td><td>Use the database and schema that the stored procedure is created in, not the database and schema that the caller is currently using.</td></tr></tbody></table>

## **5\. Demonstration of Caller’s and Owner’s Rights**

Let us understand how Caller’s and Owner’s Rights work with an example using **ACCOUNTADMIN** and **SYSADMIN** roles.

Using **ACCOUNTADMIN** role, let us create a table named `Organization` for demonstration.

```sql
USE ROLE ACCOUNTADMIN;
CREATE TABLE organization(id NUMBER, org_name VARCHAR(50));
```

When the table is queried using **SYSADMIN** role, it throws an errors as shown below since no grants on this table are provided to SYSADMIN.

```sql
USE ROLE SYSADMIN;
SELECT * FROM organization;
```

![Output of querying table with SYSADMIN role](https://thinketl.com/wp-content/uploads/2023/03/121-1-Querying-the-table-with-SYSADMIN-role.png)

Let us create a stored procedure with Caller’s rights using ACCOUNTADMIN role to delete data from Organization table.

```sql
USE ROLE ACCOUNTADMIN;

CREATE OR REPLACE PROCEDURE sp_demo_callers_rights()
  RETURNS VARCHAR
  LANGUAGE SQL
  EXECUTE AS CALLER
AS
$$
  BEGIN
    DELETE FROM ORGANIZATION WHERE ID = '101';
    RETURN 'Data cleaned up from table.';
  END;
$$
;
```

The output of the caller’s rights stored procedure with ACCOUNTADMIN role is as below.

```sql
USE ROLE ACCOUNTADMIN;
CALL sp_demo_callers_rights();
```

![Output of executing stored procedure with ACCOUNTADMIN role](https://thinketl.com/wp-content/uploads/2023/03/121-2-Output-of-stored-procedure-with-ACCOUNTADMIN-role.png)

Assign the grants to execute the stored procedure to the SYSADMIN role.

```sql
USE ROLE ACCOUNTADMIN;
GRANT USAGE ON PROCEDURE DEMO_DB.PUBLIC.sp_demo_callers_rights() TO ROLE SYSADMIN;
```

The output of the caller’s rights stored procedure with SYSADMIN role is as below.

```sql
USE ROLE SYSADMIN;
CALL sp_demo_callers_rights();
```

![Output of executing caller's rights stored procedure with SYSADMIN role](https://thinketl.com/wp-content/uploads/2023/03/121-3-Calling-Stored-Procedure-with-SYSADMIN-role.png)

**Since the SYSADMIN role do not have any privileges on Organization table, the execution of procedure with caller’s rights also fails.**

The owner of the stored procedure can change the procedure from an owner’s rights stored procedure to a caller’s rights stored procedure (or vice-versa) by executing an **ALTER PROCEDURE** command as shown below.

```sql
ALTER PROCEDURE sp_demo_callers_rights() EXECUTE AS OWNER;
```

The output of the owner’s rights stored procedure with **SYSADMIN** role is as below.

```sql
USE ROLE SYSADMIN;
CALL sp_demo_callers_rights();
```

![Output of executing owner's rights stored procedure with SYSADMIN role](https://thinketl.com/wp-content/uploads/2023/03/121-2-Output-of-stored-procedure-with-ACCOUNTADMIN-role.png)

**Though the SYSADMIN role do not have privileges on Organization table, the execution of the procedure which deletes data from the Organization table succeeds because the procedure executes with Owner’s rights.**
