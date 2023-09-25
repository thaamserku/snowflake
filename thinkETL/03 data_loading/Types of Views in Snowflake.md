---
created: 2023-09-24T12:23:24 (UTC -04:00)
tags: [snowflake, view]
source: https://thinketl.com/types-of-views-in-snowflake/
author: ThinkETL
---

# Types of Views in Snowflake

>[!Excerpt]
> There are three different types of views in Snowflake - Non-Materialized, Materialized and Secure Views.


In this article, let us discuss what is a view, types of views in Snowflake and their use cases.

## **What is a View?**

**A View is nothing more than a saved SQL query associated with a name stored in your database.**

A View can be considered as a virtual table and can be used almost anywhere that a table can be used (joins, subqueries, etc.). Whenever you query a view, the underlying SQL query associated with the view gets executed dynamically and the results were displayed.

A View can be created using **CREATE VIEW** command using below syntax in Snowflake.

```sql
CREATE OR REPLACE VIEW my_view AS

SELECT * FROM my_table;
```

There are three broad categories of views in Snowflake.

1.  Non-Materialized views (Regular views)
2.  Materialized Views
3.  Secure Views

Let us discuss in detail about each one of them listed above.

Consider below patient’s data for the demo on views.

![PATIENT_DETAILS table](https://thinketl.com/wp-content/uploads/2022/05/77-1-Patient_details-1024x225.png)

PATIENT\_DETAILS table

## **Non-Materialized Views**

**A Non-Materialized view’s results are created by executing the query at the time that the view is referenced in a query. The term “Views” generally refers to Non-Materialized views.**

-   The results of non-materialized views are not stored for future use.
-   Performance is slower than compared to materialized views.
-   The syntax to create a non-materialized view is same as creating a view.

Below is an example of non-materialized view created on top of PATIENT\_DETAILS table selecting only the details(fields) which are required for a doctor.

```sql
CREATE OR REPLACE VIEW doctor_view AS
SELECT patient_id, patient_name, diagnosis, treatment FROM PATIENT_DETAILS;
```

## **Materialized Views**

**A Materialized view is a database object that stores the pre-computed results of a query definition of a view. While simple views allow us to save complicated queries for future use, materialized views store a copy of the query results.**

-   Materialized Views are used when data is to be accessed frequently and data in table does **not** get updated on frequent basis. Whereas Views are generally used when data is to be accessed infrequently and data in table get updated on frequent basis.
-   Materialized views are automatically and transparently maintained by Snowflake. The automatic maintenance of materialized views consumes credits.
-   Materialized views in Snowflake have a lot of limitations compared to a view.

    -   A materialized view can query only a single table.

    -   Joins, including self-joins, are not supported.
    -   For full list of limitations of materialized views refer [Snowflake Documentation](https://docs.snowflake.com/en/user-guide/views-materialized.html#limitations-on-creating-materialized-views).

To create a materialized view, use the **MATERIALIZED** keyword with the standard DDL of non-materialized-views.

Below is an example of materialized view created on top of PATIENT\_DETAILS table selecting only the details(fields) which are required for finance team.

```sql
CREATE OR REPLACE MATERIALIZED VIEW accountant_view AS
     SELECT patient_id, patient_name, billing_address, cost FROM PATIENT_DETAILS;
```

## **Secure Views**

**A Secure View limit access to the data definition of the view so that the sensitive data that should not be exposed to all users of the underlying table(s) stays hidden.**

-   Both non-materialized and materialized views can be defined as **secure.**
-   Secure views have advantages over standard views, including improved data privacy and data sharing. However, they also have some performance impacts to take into consideration.
-   The definition of a secure view is only exposed to authorized users i.e. users who have been granted the role that owns the view.

To create a Secure view, use the **SECURE** keyword with the standard DDL for views.

Below is an example of secure view created on top of PATIENT\_DETAILS

```sql
CREATE OR REPLACE SECURE VIEW patient_view AS
    SELECT * FROM PATIENT_DETAILS;
```

### **SHOW VIEWS**

**SHOW VIEWS** command in Snowflake lists the views, including secure views, for which you have access privileges.

The below image shows SHOW VIEWS command listing all the views we have created as part of demo in this article.

![Executing SHOW VIEWS with owner role](https://thinketl.com/wp-content/uploads/2022/05/77-2-SHOW-VIEWS-edited-1024x269.png)

Executing SHOW VIEWS with owner role

Since the ACCOUNT\_ADMIN is owner of the Secure View PATIENT\_VIEW, the view definition is visible under the column text.

Check the **IS\_SECURE** column in the output of the SHOW VIEWS command to find out whether a view is secure or not.

### **Accessing Secure Views from a different role**

To fully understand how secure views work, let us grant access on secure views to a different role and see how the data definition is secured.

The below commands provide **read only** access to SYSADMIN on all the views created by ACCOUNTADMIN.

```sql
USE ROLE ACCOUNTADMIN;
GRANT SELECT ON ALL VIEWS IN SCHEMA analytics.health_care TO ROLE SYSADMIN;
```

Let us verify the view definition from SYSADMIN role.

```sql
USE ROLE SYSADMIN;
SHOW VIEWS;
```

![Executing SHOW VIEWS with read-only role](https://thinketl.com/wp-content/uploads/2022/05/77-3-SHOW-VIEWS-SYSADMIN-edited-1024x240.png)

Executing SHOW VIEWS with read-only role

> **Since the SYSADMIN has read-only access on secure view, the view definition is hidden whereas the view definition of regular views is visible.**

## **Conclusion**

-   Views enable writing more modular code and granting access to a subset of a Table. For example, in the demo views we have created, we have created separate views for Doctors and Accountants with access to different details of same patient.
-   Materialized Views are designed to improve performance but also have a lot of limitations compared to non-materialized views in Snowflake.
-   When deciding whether to use a secure view, you should consider the purpose of the view and weigh the trade-off between data privacy/security and query performance.
-   There is also a fourth category of views which we didn’t discuss in the article called Recursive views (i.e. the view can refer to itself). Instead of using a [recursive CTE](https://docs.snowflake.com/en/user-guide/queries-cte.html#recursive-ctes-and-hierarchical-data) in a view , you can create a recursive view with the keyword **RECURSIVE**.
