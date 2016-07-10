# SQLcl

* [Intro](#intro)
* [Commands](#commands)
* [Scripting](#scripting)
* Advanced usage

## Intro

What is SQLcl? From their own web portal:

> Oracle SQLcl is a free command line interface for Oracle Database. It allows you to interactively or batch execute it's own commands alongside with SQL, PL/SQL, operating system commands, and JavaScript. SQLcl offers a feature-rich experience in a shell environment, while also supporting all of the previous scripts our customers have developed for their Oracle environments..

Whilst SQL*Plus is still around, no new features have been added in a while. SQLcl aims to bridge the divide. The purpose of this project is to provide some high level documentation in lieu of any current official documentation - currently, there is a lot of content out there - but most of it is located in blog posts, so this should provide a single point of reference.

New commands that have been made available can be identified with the help command. Any new commands (new, as in, not available in SQL*Plus) will be styled with bold and underline when running the `help` command. They currently include:

* ALIAS
* APEX
* BRIDGE
* CD
* CTAS
* [DDL](#ddl)
* FORMAT
* HISTORY
* INFORMATION
* LOAD
* NET
* NOHISTORY
* OERR
* REPEAT
* REST
* SCRIPT
* SODA
* SSHTUNNEL
* TNSPING

You can get more help on each command by typing `help <cmd>`, which will provide a basic overview of the command.

One of the great new features is the ability to run SQLcl commands through a script. Typically, the scripting language would be JavaScript - but SQLcl is Java based, so any language that can be run in Java will be supported.

## Commands

### DDL

DDL can be used to pull the DDL for any database object that is available to the connected user. The syntax is `DDL [<object_name> [<type>] [SAVE <filename>]]`

So, as a basic example - if we connect to `HR` we can generate the DDL for our employees table with `DDL employees`. We could additionally specify it's a table: `DDL employees table`. Or if we want to save it to a file (avoiding spools), `DDL employees save emp.sql`.

If you run this as is, you will notice that all the storage options are included. This is all configurable. If you run the command `ddl options` you will see what's available to be set:

```sql
SQL> show ddl
STORAGE : ON
INHERIT : ON
SQLTERMINATOR : ON
OID : ON
SPECIFICATION : ON
TABLESPACE : ON
SIZE_BYTE_KEYWORD : ON
PRETTY : ON
REF_CONSTRAINTS : ON
FORCE : ON
PARTITIONING : ON
CONSTRAINTS : ON
INSERT : ON
BODY : ON
CONSTRAINTS_AS_ALTER : ON
SEGMENT_ATTRIBUTES : ON
```

So, we can turn any of these options off with: `set ddl <option> <value>`. So, if we turn storage off with: `set ddl storage off`, we will have our DDL without storage. Other ones I like to turn off are `tablespace` and `segment_attributes`.

```sql
set ddl storage off
set ddl tablespace off
set ddl segment_attributes off

ddl employees
```

The documentation states the `type` field is only used for materialized views. I've had success specifying the type for other object types, however not when used with the save option. For this, you may need to make use of spooling. e.g.

```
SQL> spool foo.sql
SQL> ddl foo package
  CREATE OR REPLACE PACKAGE "HR"."FOO"
as
    lc_bar NUMBER := 1001;
end foo;
/
SQL> spool off
SQL> !ls | grep foo
foo.sql

SQL> !cat foo.sql
  CREATE OR REPLACE PACKAGE "HR"."FOO"
as
    lc_bar NUMBER := 1001;
end foo;
/
SQL> spool off
```

### Information

The information command expands on the desc command. The desc command provides an overview of all the columns in a table, where info will go one step further and list all table indexes and constraints. If you post fix the command with a `+` you will also get the column statistics.

```
SQL> desc employees
Name           Null?    Type         
-------------- -------- ------------
EMPLOYEE_ID    NOT NULL NUMBER(6)    
FIRST_NAME              VARCHAR2(20)
LAST_NAME      NOT NULL VARCHAR2(25)
EMAIL          NOT NULL VARCHAR2(25)
PHONE_NUMBER            VARCHAR2(20)
HIRE_DATE      NOT NULL DATE         
JOB_ID         NOT NULL VARCHAR2(10)
SALARY                  NUMBER(8,2)  
COMMISSION_PCT          NUMBER(2,2)  
MANAGER_ID              NUMBER(6)    
DEPARTMENT_ID           NUMBER(4)    
SQL>
SQL> info employees
Columns
NAME             DATA TYPE           NULL  DEFAULT    COMMENTS
*EMPLOYEE_ID     NUMBER(6,0)         No                   Primary key of employees table.
 FIRST_NAME      VARCHAR2(20 BYTE)   Yes                  First name of the employee. A not null column.
 LAST_NAME       VARCHAR2(25 BYTE)   No                   Last name of the employee. A not null column.
 EMAIL           VARCHAR2(25 BYTE)   No                   Email id of the employee
 PHONE_NUMBER    VARCHAR2(20 BYTE)   Yes                  Phone number of the employee; includes country code and area code
 HIRE_DATE       DATE                No                   Date when the employee started on this job. A not null column.
 JOB_ID          VARCHAR2(10 BYTE)   No                   Current job of the employee; foreign key to job_id column of the
                                                                    jobs table. A not null column.
 SALARY          NUMBER(8,2)         Yes                  Monthly salary of the employee. Must be greater
                                                                    than zero (enforced by constraint emp_salary_min)
 COMMISSION_PCT  NUMBER(2,2)         Yes                  Commission percentage of the employee; Only employees in sales
                                                                    department elgible for commission percentage
 MANAGER_ID      NUMBER(6,0)         Yes                  Manager id of the employee; has same domain as manager_id in
                                                                    departments table. Foreign key to employee_id column of employees table.
                                                                    (useful for reflexive joins and CONNECT BY query)
 DEPARTMENT_ID   NUMBER(4,0)         Yes                  Department id where employee works; foreign key to department_id
                                                                    column of the departments table

Indexes
INDEX_NAME            UNIQUENESS  STATUS  FUNCIDX_STATUS  COLUMNS                COLUMN_EXPRESSION  
HR.EMP_JOB_IX         NONUNIQUE   VALID                   JOB_ID                                    
HR.EMP_NAME_IX        NONUNIQUE   VALID                   LAST_NAME, FIRST_NAME                     
HR.EMP_EMAIL_UK       UNIQUE      VALID                   EMAIL                                     
HR.EMP_EMP_ID_PK      UNIQUE      VALID                   EMPLOYEE_ID                               
HR.EMP_MANAGER_IX     NONUNIQUE   VALID                   MANAGER_ID                                
HR.EMP_DEPARTMENT_IX  NONUNIQUE   VALID                   DEPARTMENT_ID                             


References
TABLE_NAME   CONSTRAINT_NAME  DELETE_RULE  STATUS   DEFERRABLE      VALIDATED  GENERATED  
DEPARTMENTS  DEPT_MGR_FK      NO ACTION    ENABLED  NOT DEFERRABLE  VALIDATED  USER NAME  
EMPLOYEES    EMP_MANAGER_FK   NO ACTION    ENABLED  NOT DEFERRABLE  VALIDATED  USER NAME  
JOB_HISTORY  JHIST_EMP_FK     NO ACTION    ENABLED  NOT DEFERRABLE  VALIDATED  USER NAME  

SQL>
```

### Loading data

The load command allows you to load data from a CSV file.

## Scripting

You could build scripts to run particiular SQLcl commands, which can be run in any language that can be run through the JVM (See JSR-232). This will most typically be with JavaScipt. To run a script you have developed, you just prefix the script filename with `script`. So, if I have a script called `demo.js` I would run `script demo` (not, leaving off the file prefix assumed javascript).

## Advanced usage

### Pre and post commands

You can set up a script to run before and after every command/script you run. This is done with `set precommand|postcommand <script|command>`.

```
SQL> set precommand select 'begin' start_msg from dual
SQL> set postcommand select 'end' end_msg from dual
SQL> select count(1) from employees;

START
-----
begin


  COUNT(1)
----------
       107


END
---
end
```

### Output formats

You can easily display query data in a number of different formats:

* csv
* html
* xml
* json
* ansiconsole
* insert
* loader
* fixed
* default

To change the active output format, you just run: `set sqlformat <format>`. For example, to output the data in CSV, you can run:

```
SQL> help set sqlformat
SET SQLFORMAT
  SET SQLFORMAT { csv,html,xml,json,ansiconsole,insert,loader,fixed,default}   

SQL> set sqlformat csv
SQL> select * from departments;
"DEPARTMENT_ID","DEPARTMENT_NAME","MANAGER_ID","LOCATION_ID"
10,"Administration",200,1700
20,"Marketing",201,1800
30,"Purchasing",114,1700
40,"Human Resources",203,2400
50,"Shipping",121,1500
--output ommitted

27 rows selected.

SQL>
```

double check this:
An additional argument can also be specified if you want to specify a default number format when querying data.

todo: `set sqlformat [num format]`
source: http://krisrice.blogspot.com.au/2015/10/sqlcl-oct-5th-edition.html

### History

History is self explanatory - it shows a list of recently run operations. It comes with a list of extra opererations, so it is run with: `history <command>`, where command is one of:

* time - time to run the operation
* usage - the number of times a comamnd has been run
* script
* full
* clear
* fails

Or, instead of the command, you can pass in index to output the entry from the history in that index location.

### Connection strings

Typically to connect, you would pass in a string such as: `user/pass@//server:port/sid`. With SQLcl you can pass in various jdbc connection strings.

* jdbc:oracle:thin
* jdbc:oracle:oci8
* jdbc:oracle:kprb
* jdbc:default:connection
* jdbc:oracle:kprb:
* jdbc:default:connection:

So, with that, we could use the connection string: `user/pass@jdbc:oracle:thin:@server:port/sid`

Source: http://krisrice.blogspot.com.au/2015/10/sqlcl-oct-5th-edition.html
