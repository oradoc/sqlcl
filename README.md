# SQLcl

* [Intro](#intro)
* [Commands](#commands)
* [Scripting](#scripting)

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

## Scripting
