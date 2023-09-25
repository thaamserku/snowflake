---
created: 2023-09-07T21:03:08 (UTC -04:00)
tags: []
source: https://thinketl.com/snowflake-snowsql-command-line-tool-to-access-snowflake/
author: ThinkETL
---

# Snowflake SnowSQL: Command Line Tool to access Snowflake - ThinkETL

> ## Excerpt
> A definitive guide on how to download, install, configure and use Snowflake SnowSQL Command Line tool

---
## **1\. What is Snowflake SnowSQL?**

**SnowSQL is a command line tool for connecting to Snowflake from Windows, Linux and macOS. Snowflake SnowSQL lets you execute all SQL queries and perform DDL and DML operations, including loading data into and unloading data out of database tables.**

In this article let us discuss how to install and configure SnowSQL and the various command line options that are supported.

## **2\. How to download and Install Snowflake SnowSQL?**

Follow below steps to download and install Snowflake SnowSQL.

1\. Login to your Snowflake account.

2\. Navigate to **Help** > **Download.**

![Snowflake Help ](https://thinketl.com/wp-content/uploads/2022/03/71-1-download.png)

3\. Select **CLI Client(snowsql)** and click on **Snowflake Repository** in the Downloads Dialog box.

![Snowflake Downloads dialog](https://thinketl.com/wp-content/uploads/2022/03/71-2-download-1.png)

4\. Select the operating system on which you wanted to install SnowSQL and download the executable file.

![SnowSQL OS download options](https://thinketl.com/wp-content/uploads/2022/03/71-3-download.png)

5\. To install SnowSQL in Windows,

-   Double-click **.msi** file in the download location to run the installer MSI file.
-   Follow the instructions provided by the installer.

6\. To install SnowSQL in Linux,

-   Open a terminal window.
-   Run the Bash script installer from the download location.

```
$ bash snowsql-linux_x86_64.bash
```

-   Follow the instructions provided by the installer.

7\. To install SnowSQL in macOS,

-   Double-click **snowsql-darwin\_x86\_64.pkg** file in the download location to run the installer PKG file.
-   Follow the instructions provided by the installer.

8\. To verify the SnowSQL installation,

-   Open a terminal window.
-   Type below command to verify that installation completed successfully.

![Snowflake version](https://thinketl.com/wp-content/uploads/2022/03/71-4-snowflake-version-1.png)

-   For the first time, you might see that snowflake starts downloading the required packages and its auto-upgrade feature upgrading the client to the latest release irrespective of the version you downloaded and installed.

## **3\. How to Login into Snowflake SnowSQL?**

There are several ways in which you can log into Snowflake using SnowSQL. We will discuss the couple of simple methods to login.

### **3.1 Logging in through interactive Password prompt**

1\. Open a terminal Window.

2\. Use the below command to connect to Snowflake

```
$ snowsql -a <account_identifier> -u <username>
```

| Parameter | Description |
| --- | --- |
| \-a <account\_identifier> | Unique account name assigned by Snowflake. It can be obtained from your snowflake home page URL. |
| \-u <username> | User name with which you connect to the specified account. |

You can optionally specify the database, schema and warehouse in the login command as shown below

```
$ snowsql -a <account_identifier> -u <username> -d <database> -s <schema> -w<warehouse> 
```

3\. Enter the password of the account in interactive password prompt to login.

![Logging in Snowflake through interactive Password prompt](https://thinketl.com/wp-content/uploads/2022/03/71-5-snowflake-login.png)

Logging in Snowflake through interactive Password prompt

### **3.2 Logging in by configuring SnowSQL configuration file.**

It is recommended to configure the default connection parameters in the **SnowSQL** **config** file to simplify the connection process. Thereafter, when connecting to Snowflake, you can omit your Snowflake account identifier, username, and any other parameters you have configured as your default values.

Follow below steps to configure the default connection parameters.

1\. Navigate to location where SnowSQL configuration file is created. The default location of the file is:

```
Linux/macOS: ~/.snowsql/

Windows: C:\Users\<username>\.snowsql\
```

2\. Open the SnowSQL configuration file (named **config**) in a text editor. The config file consists of several default parameters configured providing you an example how to configure them in the file.

-   In Windows, open the file directly in a Notepad.
-   In Linux or macOS, open the file using a vi editor.

3\. You could use the default **\[connections\]** section to configure the default connection parameters by removing the comment symbol from any of the following parameters and specifying the correct values.

```
[connections]
#accountname = <string>   
#username = <string>      
#password = <string>      
#dbname = <string>        
#schemaname = <string>    
#warehousename = <string> 
#rolename = <string>      
#authenticator = <string> 
```

4\. If you wanted to configure different sets of connection configurations, you can define one or more **_named_** connections.

Add a separate **\[connections\]** section with a unique name for each named connection followed by a dot.

```
[connections.my_snowflake_connection]
accountname = myorganization-myaccount
username = jsmith
password = xxxxxxxxxxxxxxxxxxxx
dbname = mydb
schemaname = public
warehousename = mywh
```

5\. Once the login configurations are done in config file, use **–c connection parameter** to specify a named connection to login.

```
$ snowsql -c my_snowflake_connection
```

![Logging in Snowflake through preconfigured connection](https://thinketl.com/wp-content/uploads/2022/03/71-6-snowflake-login.png)

Logging in Snowflake through preconfigured connection

> The password is stored in plain text in the config file. You must explicitly secure the file to restrict access.

Once you log into SnowSQL execute **!help** command which displays Displays the help for SnowSQL commands. All supported commands are appended with **!(exclamation)**.

![Snowflake SnowSQL commands](https://thinketl.com/wp-content/uploads/2022/03/71-help.png)

Snowflake SnowSQL commands

## **4\. How do I run a SQL query in a SnowSQL?**

Running SQL queries in SnowSQL is simple and straight forward. Once you login using one of the methods discussed in above section, you can execute SQL queries and perform all DDL and DML operations.

The below image shows SnowSQL connected to ANALYTICS database, HR schema and COMPUTE\_WH warehouse running select statement on table Employee.

![Running SQL query in SnowSQL](https://thinketl.com/wp-content/uploads/2022/03/71-7-snowflake-running-queries.png)

Running SQL query in SnowSQL

## **5\. How to use variables in SnowSQL?**

You can use variables to store and reuse values in a Snowflake session. Variables enable you to use user-defined and database values in queries.

### **5.1 Enabling Variable Substitution**

To enable SnowSQL to substitute values for the variables, you must set the **variable\_substitution** configuration option to **true** in one of the following ways.

-   To set this option before you start SnowSQL, open the SnowSQL configuration file in a text editor, and set this option in the **\[options\]** section.

```
[options]
variable_substitution = True
```

-   To set this option when you start SnowSQL, specify the **\-o command-line flag**.

```
$ snowsql -a... -o variable_substitution=true
```

-   To set this option when in a SnowSQL session, execute the **!set** command in the session.

```
user#> !set variable_substitution=true
```

### **5.2 Defining and substituting SnowSQL Variables**

You can define and substitute SnowSQL variable in two different ways:

-   Using configuration file
-   Executing commands (on the command line in the Snowflake session)

#### **5.2.1 Define variables in SnowSQL configuration file**

Define any variables that you plan to use in the SnowSQL configuration file under \[variables\] section.

```
[variables]
<variable_name>=<variable_value>
```

For example, the below image shows variable **&my\_table** used in a SQL select statement for which the value is set as **employee** in config file.

```
[variables]
my_table=employee
```

![Running SQL query in SnowSQL from a SQL script](https://thinketl.com/wp-content/uploads/2022/03/71-8-snowflake-variables.png)

Running SQL query in SnowSQL using variables defined in config file

#### **5.2.2 Define variables after connecting to Snowflake**

To define a variable after connecting to Snowflake, execute the **!define** command in the session.

The below image shows variable **var\_table** defined with in a snowflake session and used in the Snowflake select statement.

```
user#> !define var_table=employee

user#> !set variable_substitution=true

user#> select * from &var_table;
```

![Running SQL query in SnowSQL using variables defined in the session](https://thinketl.com/wp-content/uploads/2022/03/71-9-snowflake-variables-runtime.png)

Running SQL query in SnowSQL using variables defined in the session

> SnowSQL variables need to be prefixed with **&(ampersand)** while using in queries.

## **6\. How to execute queries from a SQL Script in SnowSQL?**

You can run SQL scripts in two ways:

-   Using connection parameters (while connecting to Snowflake)
-   Executing commands (on the command line in the Snowflake session)

### **6.1 Using -f Connection Parameter while connecting**

To execute a SQL script while connecting to Snowflake, use the **\-f <input\_filename>** connection parameter.

For example:

```
snowsql –c my_connection -f /tmp/input_script.sql
```

The below image shows running SQL query from file **snowsql\_script.sql** while connecting to SnowSQL.

![Running SQL query in SnowSQL from a SQL script while connecting](https://thinketl.com/wp-content/uploads/2022/03/71-10-snowflake-running-queries-from-file.png)

Running SQL query in SnowSQL from a SQL script while connecting

### **6.2 Running in a Session (!source or !load Command)**

To run a SQL script after connecting to Snowflake, execute the **!source** (or **!load**) command in the session.

For example:

```
user#> !source example.sql
```

The below image shows running SQL query from file **snowsql\_script.sql** in a SnowSQL session using **!source**.

![Running SQL query in SnowSQL from a SQL script inside a session](https://thinketl.com/wp-content/uploads/2022/03/71-11-snowflake-running-queries-from-file.png)

Running SQL query in SnowSQL from a SQL script inside a session

## **7\. How to export data from SnowSQL?**

To export data using SnowSQL use -o or –options connection parameter. An output file and format for the script can be specified using below parameters.

-   \-o output\_file=<output\_filename>
-   \-o output\_format=<output\_format>

```
snowsql -c my_snowflake_connection -f /tmp/input_script.sql -o output_file=/tmp/output.csv -o output_format=csv -o quiet=true
```

Additionally you can use below options while exporting data from SnowSQL

<table><tbody><tr><td><strong>Options</strong></td><td><strong>Description</strong></td></tr><tr><td>-o quiet=true&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</td><td>Removes standard output after executing the command</td></tr><tr><td>-o friendly=false</td><td>Removes the Good Bye message in standard output</td></tr><tr><td>-o header=false</td><td>Removes header in the output file</td></tr><tr><td>-o timing=false</td><td>Removes timing details in the standard output</td></tr></tbody></table>

> Note that consecutive queries are appended to the output file using this method.

To avoid this issue, alternatively, redirect the query output to a file and overwrite the file with each new statement, using the greater-than sign (**\>**) in a script.

```
snowsql -c my_snowflake_connection -f /tmp/input_script.sql -o output_format=csv -o friendly=false -o timing=false > /tmp/output.csv
```

Alternatively you can use **–q <query>** parameter to pass the query directly in the login command.

```
snowsql -c my_snowflake_connection -q "select * from employee" -o output_format=csv -o friendly=false -o timing=false > /tmp/output.csv
```

> Note that **\-o quiet=true** should not be used in this method as it will result in output file creating empty.

## **8\. How to disconnect from SnowSQL?**

To exit a connection/session, use the **!exit** command (or its alias, **!disconnect**).

-   You can then connect again using **!connect <connection\_name>** if you have defined multiple connections in the SnowSQL config file.
-   Note that, if you only have one connection open, the **!exit** command also quits/stops SnowSQL.

To exit all connections and then quit/stop SnowSQL, use the **!quit** command (or its alias, **!q**). You can also type **\[CTRL\]-d** on your keyboard.

**Related Articles:**
#capture