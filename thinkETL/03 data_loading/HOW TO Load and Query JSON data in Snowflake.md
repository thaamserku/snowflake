---
created: 2023-09-07T21:00:10 (UTC -04:00)
tags: [json, snowflake]
source: https://thinketl.com/how-to-load-and-query-json-data-in-snowflake/
updated: 2024-04-21 12:13:40
---

# HOW TO: Load and Query JSON data in Snowflake?


> Snowflake supports loading JSON data in database tables and allows querying data along with flattening it into a columnar structure.

## **Introduction**

Snowflake supports loading semi structured data like JSON in database tables. Once the data is loaded from stage into a database table, Snowflake has an excellent functionality for directly querying semi-structured data along with flattening it into a columnar structure.

In this article, let us discuss the entire process of loading and parsing JSON data in Snowflake.

**Local Machine → Snowflake Internal Stage → Database table → Querying and Flattening the JSON data**

For the demonstration, we will consider below JSON data containing novel’s information of three authors.

```json
[
  {
    "AuthorName": "Author-1",
    "Category": [{
        "CategoryName": "Fiction",
        "Genre": [{
            "GenreName": "Action and Adventure",
            "Novel": [{
                "Novel": "Novel-1",
                "Sales": "200"
              },
              {
                "Novel": "Novel-2",
                "Sales": "850"
              }
            ]
          },
          {
            "GenreName": "Romance",
            "Novel": [{
              "Novel": "Novel-3",
              "Sales": "400"
            }]
          }
        ]
      },
      {
        "CategoryName": "NonFiction",
        "Genre": [{
          "GenreName": "Autobiography",
          "Novel": [{
            "Novel": "Novel-4",
            "Sales": "900"
          }]
        }]
      }
    ]
  },
  {
    "AuthorName": "Author-2",
    "Category": [{
        "CategoryName": "Fiction",
        "Genre": [{
            "GenreName": "Action and Adventure",
            "Novel": [{
              "Novel": "Novel-5",
              "Sales": "340"
            }]
          },
          {
            "GenreName": "Crime",
            "Novel": [{
                "Novel": "Novel-6",
                "Sales": "940"
              },
              {
                "Novel": "Novel-7",
                "Sales": "540"

              }
            ]
          }
        ]
      }
    ]
  },
  {
    "AuthorName": "Author-3",
    "Category": [{
        "CategoryName": "Fiction",
        "Genre": [{
            "GenreName": "Romance",
            "Novel": [{
                "Novel": "Novel-8",
                "Sales": "820"
              },
              {
                "Novel": "Novel-9",
                "Sales": "620"
              }
            ]
          },
          {
            "GenreName": "Thriller",
            "Novel": [{
              "Novel": "Novel-10",
              "Sales": "770"
            }]
          }
        ]
      },
      {
        "CategoryName": "NonFiction",
        "Genre": [{
            "GenreName": "History",
            "Novel": [{
              "Novel": "Novel-11",
              "Sales": "450"
            }]
          },
          {
            "GenreName": "Travel",
            "Novel": [{
              "Novel": "Novel-12",
              "Sales": "150"
            }]
          }
        ]
      }
    ]
  }
]
```

Our goal is to load the above JSON data into Snowflake and flatten it to achieve below columnar format of the data.

<table><tbody><tr><td><strong>Author</strong></td><td><strong>Category</strong></td><td><strong>Genre</strong></td><td><strong>Novel</strong></td><td><strong>Sales</strong></td></tr><tr><td>Author1</td><td>Fiction</td><td>Action and Adventure</td><td>Novel-1</td><td>200</td></tr><tr><td>Author1</td><td>Fiction</td><td>Action and Adventure</td><td>Novel-2</td><td>850</td></tr><tr><td>Author1</td><td>Fiction</td><td>Romance</td><td>Novel-3</td><td>400</td></tr><tr><td>Author1</td><td>Non-Fiction</td><td>Autobiography</td><td>Novel-4</td><td>900</td></tr><tr><td>Author2</td><td>Fiction</td><td>Action and Adventure</td><td>Novel-5</td><td>340</td></tr><tr><td>Author2</td><td>Fiction</td><td>Crime</td><td>Novel-6</td><td>940</td></tr><tr><td>Author2</td><td>Fiction</td><td>Crime</td><td>Novel-7</td><td>540</td></tr><tr><td>Author3</td><td>Fiction</td><td>Romance</td><td>Novel-8</td><td>820</td></tr><tr><td>Author3</td><td>Fiction</td><td>Romance</td><td>Novel-9</td><td>620</td></tr><tr><td>Author3</td><td>Fiction</td><td>Thriller</td><td>Novel-10</td><td>770</td></tr><tr><td>Author3</td><td>Non-Fiction</td><td>History</td><td>Novel-11</td><td>450</td></tr><tr><td>Author3</td><td>Non-Fiction</td><td>Travel</td><td>Novel-12</td><td>150</td></tr></tbody></table>

## **Loading JSON data from local machine into Snowflake Internal Stage**

Before we load the data, we must choose the database and schema where the data is staged and later loaded into table. For the demonstration we will be using MY\_DB database and MY\_DB.MY\_SCHEMA schema.

```sql
USE DATABASE my_db;
USE SCHEMA my_schema;
```

### **Step-1: Create a Snowflake Internal named Stage**

Create an **[Internal Named Stage](https://thinketl.com/types-of-snowflake-stages-data-loading-and-unloading-features/)** in Snowflake to hold the data loaded from local machine as shown below.

```sql
CREATE OR REPLACE STAGE my_internal_stage;
```

### **Step-2: Load data from local machine into Internal Stage using SnowSQL**

To load data from our local machine into the Snowflake Internal stage, we have to use the Snowflake’s CLI tool which is SnowSQL.

To know more about this, read our detailed article on **[Snowflake SnowSQL](https://thinketl.com/snowflake-snowsql-command-line-tool-to-access-snowflake/)**.

1. Login to Snowflake SnowSQL

2. Using PUT command, copy the files from the local folder into snowflake internal stage created in earlier step.

```sh
put file://C:\SourceFiles\authors.json @my_internal_stage;
```

![Loading JSON file from local machine into Snowflake Internal Stage](https://thinketl.com/wp-content/uploads/2022/08/90-1-SnowSQL-put.png)

Loading JSON file from local machine into Snowflake Internal Stage

### **Step-3: Verify the file in Internal Stage using List command**

Use **List** command in Snowflake worksheet to verify the file in internal stage loaded from SnowSQL as shown below.

![Listing files in Snowflake Internal Stage](https://thinketl.com/wp-content/uploads/2022/08/90-2-List.png)

Listing files in Snowflake Internal Stage

## **Loading JSON data from Snowflake internal Stage into database table**

### **Step-4: Create a JSON file format in Snowflake**

Create a **[Snowflake File format](https://thinketl.com/snowflake-file-formats/)** of type JSON which encapsulates information of data files which helps in processing the files from stage.

```sql
CREATE OR REPLACE FILE FORMAT my_json_format
type = json
strip_outer_array = true
;
```

We have used an option called **STRIP\_OUTER\_ARRAY** for this load. It helps in removing the outer set of square brackets **\[ \]** when loading the data, separating the initial array into multiple lines. Else the entire JSON data gets loaded into single record instead of multiple records.

### **Step-5: Create database table to load JSON data**

Create a table to load JSON data from Snowflake internal stage as shown below. Snowflake stores semi-structured data using the **VARIANT** field type in tables.

```sql
CREATE OR REPLACE TABLE Authors (
  JSON_DATA VARIANT
);
```

Note that the data of each author in the JSON file will be loaded into a single column **JSON\_DATA** of type **VARIANT** in the table named **Authors**.

### **Step-6: Load data from Internal Stage into database table using COPY command**

**COPY** command in Snowflake helps in loading data from staged files to an existing table. Load the data from a JSON file in internal stage to Authors database table using the **MY\_JSON\_FORMAT** file format as shown below

```sql
COPY INTO Authors
FROM @my_internal_stage/authors.json
FILE_FORMAT = (format_name = MY_JSON_FORMAT);
```

![Copying files from Snowflake internal stage into database table](https://thinketl.com/wp-content/uploads/2022/08/90-3-copy-into.png)

Copying files from Snowflake internal stage into database table

## **Querying JSON data from Snowflake database table**

The data from the Authors table can be queried directly as shown below. By using the **STRIP\_OUTER\_ARRAY** option, we were able remove this initial array **\[\]** and treat each object in the array as a row in Snowflake. Hence each author object loaded as a separate row.

![Querying data from database table with JSON data](https://thinketl.com/wp-content/uploads/2022/08/90-4-query-json-data.png)

Querying data from database table with JSON data

The individual elements in the column JSON\_DATA can be queried using standard **:** notation as shown below.

```sql
SELECT
    JSON_DATA:AuthorName,
    JSON_DATA:Category
FROM Authors;
```

![Querying individual JSON elements from database table](https://thinketl.com/wp-content/uploads/2022/08/90-5-query-json-data-using-colan-notation.png)

Querying individual JSON elements from database table

The data in the Category can be further drilled down and required elements information can be fetched as shown below.

```sql
SELECT
    JSON_DATA:AuthorName,
    JSON_DATA:Category[0]:CategoryName,
    JSON_DATA:Category[1]:CategoryName
FROM Authors;
```

![Querying individual JSON elements using index from database table](https://thinketl.com/wp-content/uploads/2022/08/90-6-query-json-data-using-indexes.png)

Querying individual JSON elements using index from database table

The novels of the authors are categorized into two different types in the JSON data. The details of each category can be accessed using the index \[0\], \[1\].. notation as shown above.

The outer quotes in the column data can be removed by using **::** notation which lets you define the end data type of the values being retrieved. Notice how in this example, the outer quotes “ are removed.

```sql
SELECT
JSON_DATA:AuthorName::string,
JSON_DATA:Category[0]:CategoryName::string,
JSON_DATA:Category[1]:CategoryName::string
FROM Authors;
```

![Casting the datatype of fields while querying JSON data](https://thinketl.com/wp-content/uploads/2022/08/90-7-query-json-data.png)

Casting the datatype of fields while querying JSON data

Further more details of author can be drilled down as shown below.

```sql
SELECT
JSON_DATA:AuthorName::string,
JSON_DATA:Category[0]:CategoryName::string,
JSON_DATA:Category[0]:Genre[0]:GenreName::string,
JSON_DATA:Category[0]:Genre[1]:GenreName::string,
JSON_DATA:Category[1]:CategoryName::string,
JSON_DATA:Category[1]:Genre[0]:GenreName::string,
JSON_DATA:Category[1]:Genre[1]:GenreName::string
FROM Authors;
```

![Querying details of each category from JSON data](https://thinketl.com/wp-content/uploads/2022/08/90-8-query-json-data.png)

Querying details of each category from JSON data

Unfortunately, this approach is not ideal. As the data increases, you need to add additional levels of category and genre in the query statement specifying the index values. Using the **:** and **\[\]** notation alone is not sufficient to dynamically get every object in an array.

## **Flattening Arrays in JSON data**

Flattening is a process of unpacking the semi-structured data into a columnar format by converting arrays into different rows of data.

Using the **LATERAL FLATTEN** function we can explode arrays into individual JSON objects. The **input** for the function is the **array in the JSON structure** that we want to flatten (In the example shown below, the array is Category). The flattened **output** is stored in a **VALUE** column. The individual elements from unpacked array can be accessed through the VALUE column as shown below.

```sql
SELECT
JSON_DATA:AuthorName::string AS Author,
VALUE:CategoryName::string AS CategoryName
FROM Authors
,LATERAL FLATTEN (input => JSON_DATA:Category);
```

![Flattening the Category array from JSON data](https://thinketl.com/wp-content/uploads/2022/08/90-9-flatten-json-data.png)

Flattening the Category array from JSON data

When there are multiple arrays which you need to flatten, it is mandatory to pass an **alias** to every input array. The VALUE column also should be used along with the alias you passed to the input array.

In our example, we need to flatten the Category, Genre and Novel arrays to get the desired output. Also note that the Novel array is present inside Genre array which is present inside Category array. So the flattened array output VALUE becomes input for the array present inside it.

The final query to get the desired output is as below.

```sql
SELECT
    JSON_DATA:AuthorName::string AS  Author_Name,
    Flatten_Category.VALUE:CategoryName::string AS  Category_Name,
    Flatten_Genre.VALUE:GenreName::string AS Genre_Name,
    Flatten_Novel.VALUE:Novel::string AS Novel_Name,
    Flatten_Novel.VALUE:Sales:: number AS Sales_in_Millions
FROM Authors
,LATERAL FLATTEN (input => JSON_DATA:Category) AS Flatten_Category
,LATERAL FLATTEN (input => Flatten_Category.VALUE:Genre) AS Flatten_Genre
,LATERAL FLATTEN (input => Flatten_Genre.VALUE:Novel) AS Flatten_Novel
;
```

![Final output after flattening the Category, Genre and Novel arrays in JSON data](https://thinketl.com/wp-content/uploads/2022/08/90-10-flatten-json-data-output.png)

Final output after flattening the Category, Genre and Novel arrays in JSON data

## **Summary**

Snowflake supports loading semi structured data files from external and internal stages into database tables. Once the data is loaded into the table, it is important to understand the data structure and identify the arrays to flatten which provides the required output. The transformed data can then be loaded into another database tables with proper field names and data types easily.

This is a good example of Snowflake’s ELT features which is extremely helpful as the semi structured data can be easily transformed once loaded without the help of external ETL tools.
