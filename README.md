# mcQueryTester
###### By Maicon Carneiro (dibiei.blog)
### What is that ?
The mcQueryTester is an Free SQL Stress Test tool that can run the same SQL statement how many times you want using concurrent/parallel connections in Oracle Database.

This is an Java application and can be used in MacOS, Linux and Windows systems. No JDBC driver is required as it already included in the package.

The tool is intended to be used by Database Administrators via command line. You just need  to have a Java installed in your machine or database server, no additional installation or configuration is required.

## A) Get started
#### 1) Download the tool from Dibiei blog repository:
``` shell
mkdir ~/mcQueryTester
cd  ~/mcQueryTester
wget "https://github.com/maiconcarneiro/blog-dibiei/blob/main/mcQueryTester.jar"
```

#### 2) Create a script file with SQL to be executed
###### Example: my_select.sql
``` sql
SELECT SYSDATE FROM DUAL;
```

#### 3) Create a configuration file
###### Example using the helper command line:
``` shell
 java -jar mcQueryTester --create_sample_config
```

#### 4) Update the configuration file
###### Example of sample_config.cfg with minimal required parameters:
``` conf
db.url=cluster01-scan.dibiei.com:1521/racdb_short.dibiei.com
db.user=usr_test
sql.query=my_select.sql
```

#### 5) Execute the mcQueryTester
###### Example using sample_config.cfg:
``` bash
java -jar mcQueryTester.jar --config=sample_config.cfg
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

#### 3) Execute the he mcQueryTester using configuration file inserts.cfg

```
java -jar mcQueryTester.jar --config=inserts.cfg
```

## C) Workload Control
You can control the workload as a number of concurrent connections and number of executions per connection.

#### Using command line options:

``` bash
--numConnections=32 --numExecs=1000 
```
#### Using properties in configuration file:
``` conf
run.connectios=32
run.execsPerConnection=1000
```

In both examples, the SQL should get a total of 32.000 executions (32 connections X 1000 execs per connection).


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

## E) Session Metrics
The tool can capture the BEGIN and END values from V$MYSTAT and compute the DELTA values automatically. 

For concurrent sessions, the values will be summarized from all sessions and presented as an global aggregated value.

#### 1) Enable or disable the metrics capture:
You can specify yes or true value to enable this feature. To disable it, you can use no or false value.
``` conf
session.compute_metrics=yes
```

You can also use command line parameter to enable this:
``` bash
--computeMetrics=yes
```

#### 2) Filtering the metrics names
If you are interested in specific metrics, then you can use any SQL where expression to apply your own filters:
###### Using like:
``` conf
session.metrics_filter="name like 'cell%'"
```
###### Using =:
``` conf
session.metrics_filter="name = 'cell physical IO bytes eligible for smart IOs'"
```

Aditionally, you can combine multiple filter using AND / OR expressions.

Example output when metrics is enabled:
```

                          Computed Session Metrics (V$MYSTAT)
------------------------------------------------------------------------------------------
          Value Diff : Metric Name
------------------------------------------------------------------------------------------
                  69 : CPU used by this session
                  69 : CPU used when call started
                  77 : DB time
              10,902 : Requests to/from client
              10,702 : SQL*Net roundtrips to/from client
             135,414 : bytes received via SQL*Net from client
           1,139,520 : bytes sent via SQL*Net to client
              10,100 : calls to get snapshot scn: kcmgss
                 200 : calls to kcmgcs
                   1 : concurrency wait time
             995,904 : cumulative DB time in requests
                   1 : cursor authentications
              10,100 : execute count
                   8 : global enqueue gets sync
                   8 : global enqueue releases
              10,892 : non-idle wait count
                   4 : non-idle wait time
                 200 : opened cursors cumulative
                 100 : opened cursors current
                 200 : parse count (total)
                 100 : process last non-idle time
                 100 : session cursor cache count
             131,072 : session pga memory max
             327,440 : session uga memory
             258,072 : session uga memory max
                 100 : sorts (memory)
               6,404 : sorts (rows)
              10,802 : user calls
                 233 : workarea executions - optimal
------------------------------------------------------------------------------------------
```

## F) Enabling Oracle SQL Translation Framework
You can enable the SQL Translation feature in the mcQueryTester sessions by specifying an profile name in --translationProfileName.

##### Example:
``` shell
java -jar mcQueryTester.jar --config=example.cfg --translationProfileName=USR_TEST.MY_PROFILE1
```

## G) Finding sessions of mcQueryTester in the target Database
All sessios opened in the Database is identified with MODULE='mcQueryTester' 

Also the sessions that execute the SQL query is identified with the "Session Action ID" value in ACTION column of GV$SESSION view.

### Example
The output should contains the Session Action ID like this:
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
WHERE MODULE='mcQueryTester';
```

## Restrictions
Current version has the following restrictions:
- Named Bind Variables is not supported (but you can use positional variables)
- Only one SQL script file can be used at same time
- Only one SQL statement is supported per SQL script file, do not try to put multiple statements separated by ';'

NOTE: Review all bind variables if you get 'Missing IN or OUT parameter at index' error message.
