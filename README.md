# About
Created by <a href="https://www.linkedin.com/in/maiconcarneiro" target="_blank">Maicon Carneiro</a> (dibiei.blog)


### What is that ?
The queryTester is an Free SQL Stress Test tool that can run the same SQL statement how many times you want using concurrent/parallel connections in Oracle Database.

The tool is intended for DBAs and developers run yours custom test case with your own SQL query against an Oracle database and then analyze the impact of the test by using ASH, AWR, or even using the metrics that are automatically captured by this tool.

This is an Java application and can be used in MacOS, Linux and Windows systems. No additional JDBC driver is required as it already included in the tool. This is a command line tool e no GUI support is required, you only need a terminal with Java 8 or higher installed.

When executed, the tool will generate a output like this:
```
$ java -jar queryTester.jar --config=demo.cfg --executions=500 --threads=10 --sqlMetrics=with_value

queryTester - A Free Oracle SQL Stress Test Tool
Visit https://github.com/maiconcarneiro/queryTester for more information.


==========================================================================================
Configuration file..........: fetch_demo.cfg
Database Information........: localhost:1521/freepdb1
SQL Query Script File ......: sql/select_fetch_test.sql
SQL Query Binds File .......: sql/select_fetch_test.bind
SQL Preview SQL ID..........: yes
SQL Compute Metrics.........: yes
Session Compute Metrics.....: yes
Session Metrics Filter......: value >0 and name like '%client%'
Session Parameters File.....:
Session Translation Profile.:
Session Action ID...........: 8677d3a89556e1d3a98ad56786baab5d

# of Threads................: 10
# of Execs per Thread.......: 50
# of Executions total.......: 500
# of Rows per Fetch.........: 10
==========================================================================================

27/12/2024 20:01:36: Testing database connection...
27/12/2024 20:01:37: Validating Session Metrics query
27/12/2024 20:01:37: Getting SQL Id
27/12/2024 20:01:37: Calculated SQL ID (Preview): 108njrn7uc683
27/12/2024 20:01:37: Getting initial SQL Metrics
27/12/2024 20:01:37: SQL Execution in progress ...
27/12/2024 20:01:37: SQL Execution completed.
27/12/2024 20:01:37: Getting final SQL Metrics
27/12/2024 20:01:37: Test completed.

------------------------------------------------------------------------------------------
Elapsed time: 277.85 (ms)
------------------------------------------------------------------------------------------

Phase Name                       Elap Avg (ms)  Elap Max (ms)   Elap Min (ms)        %
------------------------------  --------------- --------------- --------------- --------
Compute Session Metrics                    2.40            3.84            1.86     1.38
Open DB Connection                       141.35          156.76          100.84    56.42
SQL Step 1: Prepare                         .17             .37             .05      .13
SQL Step 2: Execution                     64.56           75.57           55.11    27.20
SQL Step 3: Fetch Data                    59.23           77.73           52.96    27.98
------------------------------------------------------------------------------------------

========================================================================================
Computed SQL Metrics from GV$SQL for SQL ID 108njrn7uc683
========================================================================================
Metric Name                         | Begin      | End        | Delta      | Per exec
---------------------------------------------------------------------------------------
Buffer Gets                         |      5,783 |      7,277 |      1,494 |       3.03
CPU Time (ms)                       |        110 |        122 |         12 |       0.03
Elapsed Time (ms)                   |        182 |        215 |         33 |       0.07
Executions                          |      1,493 |      1,986 |        493 |       1.00
Fetches                             |      2,996 |      3,994 |        998 |       2.02
Pinned Total                        |      1,493 |      1,992 |        499 |       1.01
Rows Processed                      |     14,970 |     19,950 |      4,980 |      10.10
---------------------------------------------------------------------------------------



                          Computed Session Metrics (V$MYSTAT)
------------------------------------------------------------------------------------------
          Value Diff : Metric Name
------------------------------------------------------------------------------------------
               1,040 : Requests to/from client
               1,040 : SQL*Net roundtrips to/from client
                   0 : bytes received via SQL*Net from client (converted to MBytes)
                   0 : bytes sent via SQL*Net to client (converted to MBytes)
------------------------------------------------------------------------------------------
```

## A) Get started
#### Prereq: Ensure that you have java 1.8 or higher in machine where you will run queryTester:
``` shell
java -version
```

#### 1) Download the tool from my blog repository:
``` shell
mkdir ~/queryTester
cd  ~/queryTester
wget "https://github.com/maiconcarneiro/queryTester/blob/main/queryTester.jar"
```

#### 2) Create a script file with SQL to be executed
###### Example: my_select.sql
``` sql
SELECT SYSDATE FROM DUAL;
```

#### 3) Create a configuration file
###### Example using the helper command line:
``` shell
 java -jar queryTester.jar --create_sample_config
```

#### 4) Update the configuration file
###### Example of sample_config.cfg with minimal required parameters:
``` conf
db.url=cluster01-scan.dibiei.com:1521/racdb_short.dibiei.com
db.user=usr_test
sql.query=my_select.sql
```

#### 5) Execute the queryTester
###### Example using sample_config.cfg:
``` bash
java -jar queryTester.jar --config=sample_config.cfg
```

## B) SQL with Bind Variables
To work with BIND variables, you need to create an additional file with the bind values using positional format.

#### 1) Create a script file with the SQL using Bind variables.
###### Example: sql/insertMyTable.sql
``` sql
INSERT INTO usr_test.my_table (
    id, column1, column2, column3, column4, column5, column6, column7, column8, column9, column10
) VALUES (
    usr_test.my_table_seq.NEXTVAL, :1, :2, :3, :4, :5, :6, :7, :8, :9, RPAD('X', 4000, 'X')
```

#### 2) Create a second file with bind mapping using :POSITION=VALUE format.
##### The variables must be listed in the same order as they appear in SQL_TEXT
###### Example: sql/insertMyTable.bind
``` sql
:1='Value1'
:2='Value2'
:3='Value3'
:4='Value4'
:5='Value5'
:6='Value6'
:7='Value7'
:8='Value8'
:9='Value9'
```

#### 3) Update the configuration file with sql.binds property
###### Example: inserts.cfg
``` shell
db.url=cluster01-scan.dibiei.com:1521/racdb_short.dibiei.com
db.user=usr_test
sql.query=sql/insertMyTable.sql
sql.binds=sql/insertMyTable.bind
```

#### 3) Execute the he queryTester using configuration file inserts.cfg

```
java -jar queryTester.jar --config=inserts.cfg
```

## C) Workload Control
You can control the workload as a number of concurrent connections and number of executions per connection.
You can also control the fetch size when executing query that return data from the database.
All parameters are optional and the default value is 1.

### Controling the number of executions and concurrency

#### 1) Using properties in configuration file:
Supported properties the govern the number of execution & concurrent threads.

- All the properties are optional and are overridden by the equivalent command line parameter.
``` conf
run.executions=500
run.threads=10
run.execsPerThread=50
```

#### 2) Examples using command line:

**Use Case 1:** Execute the query 500 times with 10 concurrent threads/connections (will result in 50 executions per thread):
``` bash
--executions=500 --threads=10
```

**Use Case 2:**  Execute the query 500 times with 25 executions per thread (will result in 20 concurrent threads):
``` bash
--executions=500 --execsPerThread=25
```

**Use Case 3:**  Start 50 concurrent threads with 100 executions per thread (will result a total of 5.000 executions):
``` bash
--threads=50 --execsPerThread=100
```

### Rules of precedence

- When **--executions** is used with **--threads**, the **--execsPerThread** value will be derived automatically as **executions / threads**.
- When **--executions** is used with **--execsPerThread**, the **--threads** value will be derived automatically as **executions / execsPerThread**.
- When **--threads** and **--execsPerThread** are used together, the value of **--executions** will be ignored.
- If the value of execsPerThread cannot be divided equally between all the threads, the first thread will assume the additional executions to complete the task.



### Controlling the Fetch Size
When testing an SELECT statement, you can change the default Fetch Size (10) used by the tool.

This option is helpfully to demonstrate the impact of tuning this attribute for batch routines that select and return a high number of rows.

Using property file:
```
run.fetch_size=200
```

Using command line:
```
--fetchSize=200
```

###### TIP: When testing the fetch size parameter, enable Session Metrics and look for '%client%' metrics ;)


### Control Reuse of Database Connection
By default, the tool will open one connection per thread.

Example, considering the previous example with 500 executions and 10 threads, the tool will open 10 database connections and execute the query 50 times within each connection.

You can change this to reproduce a very bad behavior of some applications that open and close one connection for every SQL execution.

``` shell
--reuseConnection=no
```

With this parameter, for the previous example, the tool will open and close a total of 500 connections (one per SQL execution).

However, note that only 50 connections should be opened at same time, as the number of concurrent threads is 50.

If you want to reproduce the worst scenario, you can do something like this:
``` shell
---threads=500 --execsPerThread=1 --reuseConnection=no
```

This example will result in 500 connections being opened at the same time (or trying)

## D) Session Control
You can use a file with custom 'ALTER SESSION' statements.

#### 1) Create a .sql file with your own ALTER SESSION:
###### Example: my_alter_sessions1.sql
``` sql
alter session set statistics_level=ALL;
alter session set cell_offload_plan_display=NEVER;
alter session set optimizer_index_caching=100;
alter session set optimizer_index_cost_adj=5;
```

#### 2) Then update the configuration file with this property:
``` conf
session.parameters_file=my_alter_sessions1.sql
```

## E) Getting Execution Metrics
The tool can capture the BEGIN and END values from views V$MYSTAT, GV$SQL and compute the DELTA values automatically.


### 1) Session Metrics (V$MYSTAT)
For concurrent sessions, the values from V$MYSTAT will be summarized from all sessions and presented as an global aggregated value.

#### 1.1) Enable or disable the session metrics capture:
You can specify yes or true value to enable this feature. To disable it, you can use no or false value.
``` conf
session.compute_metrics=yes
```

You can also use command line parameter to enable this:
``` bash
--sessionMetrics=yes
```

#### 1.2) Filtering the metrics names
If you are interested in specific metrics, then you can use any SQL where expression to apply your own filters:
###### Using like:
``` conf
session.metrics_filter="name like 'cell%'"
```
###### Using =:
``` conf
session.metrics_filter="name = 'cell physical IO bytes eligible for smart IOs'"
```

Additionally, you can combine multiple filter using AND / OR expressions.
##### NOTE: When you specify a custom filter, the output will include all metrics that corresponds to your filter. Otherwise, the output will include all metrics where Delta value is greater than zero.

### 3) SQL Metrics (GV$SQL):
This option will preview the SQL_ID for the SQL_TEXT using Oracle SQL Translation feature (supported with 12c+).

Using this SQL_ID, the tool will collect metrics from GV$SQL at the BEGIN and the END of test, and then calculate the DELTA values.

#### 3.1) Enabling SQL metrics capture
Using property file:
``` conf
sql.compute_metrics=yes
```
Using command line:
``` bash
--sqlMetrics=yes
```

#### 3.2) Showing Only Metrics with Value
You can restrict the result to output only metrics with value greater than 0 (zero):
``` bash
--sqlMetrics=with_value
```

## F) Using SQL Translation Profile
You can enable the SQL Translation feature in the queryTester sessions by specifying an profile name in the  --translationProfileName parameter.

##### Example:
``` shell
java -jar queryTester.jar --config=example.cfg --translationProfileName=USR_TEST.MY_PROFILE1
```

## G) Using Client-Side Connection Pool
By default, the tool will open new Database connections at runtime without using connection pool.
In this default configuration, you can notice a relevant elapsed time in "Open DB Connection" phase.

You can change this behavior by enabling the use of connection pooling using the following parameter:
``` shell
 --useConnectionPool=yes
```

When connection pool is enabled, the tool will use Oracle Universal Connection Pool (UCP) and pre-open the connections before start the SQL executions, simulating an real-world scenario where is expected to have the connections already opened and available to use. So, you should notice a lower elapsed times for "DB Open Connection" phase.


## H) Advanced JDBC Options
###### **Note:** This section is optional, you need only inform a EZCONNECT in **db.url** parameter in the config file.
###### However, if you need more custom or advanced JDBC and or TNS options, this section is intended to provide this flexbility to you.

#
This tool is developed with Oracle JDBC (OJDBC) Driver.
By default, the driver will follow the same rules as any other Oracle Client, so you can use existing TNS configuration.

#### TNS Admin
The application will use the first TNS using the precedence:

```
1: --tnsAdmin parameter
2: $TNS_ADMIN (OS environment variable)
3: $ORACLE_HOME/network/admin (OS environment variable)
```

How to override the default TNS Admin:
``` shell
java -jar queryTester.jar --config=demo.cfg --tnsAdmin=/home/oracle/tns
```

#### OJDBC Properties
You can customize the JDBC configuration using your own OJDBC properties files.

Step 1) Create an new ojdbc property file, example 'my_ojdbc.properties' :
``` config
oracle.jdbc.allowedLogonVersion=11
oracle.jdbc.thinForceDNSLoadBalancing=true
```

Step 2) Update the queryTester config file to include the 'jdbc.property_file' as below:
``` config
jdbc.property_file=/home/oracle/test/my_ojdbc.properties
```

## I) Others
### SQL ID Preview
If you want to seed a preview of SQL_ID generated to the query being tested, enable this option in the config file:
``` config
sql.preview_sqlid=yes
```

And you should see anything like this in the output when start the test:
``` shell
27/12/2024 18:02:12: Getting SQL Id
27/12/2024 18:02:12: Calculated SQL ID (Preview): f52s6k68h0rxm
```

### Output behavior
If you want to omit the detail of messages generated by the tool, you can disable them with these options in config file:
``` config
options.show_phase_times=no
options.show_progress_messages=no
options.show_summary_header=no
```
Where **show_phase_times** control if the Elapsed time will be calculated per phase, and **show_progress_messages** control if the log messages will be printed in the output.



##  Finding sessions of queryTester in the target Database
All sessions opened in the Database is identified with PROGRAM and MODULE equals to 'queryTester', with can be used to track this connections in GV$SESSION view or even in Listener logs.

Also the sessions that execute the SQL query is identified with the "Session Action ID" value in ACTION column of GV$SESSION view.

### Example
The output should contain the Session Action ID like this:
``` 
Session Action ID...........: 731e96a012c456023e76229132b424c0
```

So you can use this information to query GV$SESSION and GV$SQL views:
##### Find my SQL_ID by using the ACTION:
``` sql
SELECT SQL_ID, PLAN_HASH_VALUE 
FROM GV$SQL 
WHERE ACTION='731e96a012c456023e76229132b424c0';
```
##### Find my sessions by using the ACTION:
``` sql
SELECT INST_ID, SID, PREV_SQL_ID, STATUS, EVENT 
FROM GV$SESSION 
WHERE ACTION='731e96a012c456023e76229132b424c0';
```
##### Find my sessions by using the MODULE:
``` sql
SELECT INST_ID, SID, PREV_SQL_ID, STATUS, EVENT 
FROM GV$SESSION 
WHERE MODULE='queryTester';
```


## Limitations
Current version has the following restrictions:
- Named Bind Variables are not supported (you can use positional variables).
- Only one SQL script file can be used at same time.
- Only one SQL statement is supported per SQL script file.

NOTE: Review all bind variables if you get 'Missing IN or OUT parameter at index' error message.st.sql
 IN or OUT parameter at index' error message.
