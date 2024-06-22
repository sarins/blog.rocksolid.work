+++
title = 'Oracle基础操作'
subtitle = 'Oracle basis operations'
brief = ''
date = 2014-09-18
categories = ['Database', 'Oracle']
series = []
tags = ['Database', 'Oracle', 'sqlplus']
type = 'blog'
+++

## 1. 启动/停止

```bash
# 切换到oracle用户。
[root@Udb ~]# su - oracle

# 使用sqlplus登录到根。
[oracle@Udb ~]sqlplus /nolog

SQL> conn / as sysdba

# 启动数据库实例。
SQL> startup

# 停止数据库实例。
SQL> shutdown immediate

```

- - -

## 2. 监听器控制

```bash
# 以下命令均基于oracle用户环境下执行。

# 检查监听器状态
[oracle@Udb ~]$ lsnrctl status

# 启动监听器
[oracle@Udb ~]$ lsnrctl start
```

- - -

## 3. 建立数据库以及表空间

```bash
# 根据前文中介绍的方法，使用sqlplus登录到根执行以下操作。

# 建立表空间（正式/临时）
SQL> create tablespace testins datafile '/ora_ds/testins/testins.dbf' size 4096m;

SQL> create bigfile tablespace testins datafile '/ora_ds/testins/testins.dbf' size 409600m;

SQL> create temporary tablespace tmp_testins tempfile '/ora_ds/testins/tmp_testins.dbf' size 2048M autoextend on next 100M maxsize 4096M;

# 使用前文建立的表空间，建立新用户。
SQL> create user testins identified by "6yhn*IK<" default tablespace testins temporary tablespace tmp_testins;

# 向新建用户授权。
SQL> grant resource,connect,dba to testins;
```

- - -

## 4. 数据导出

```bash
# exp命令使用参考
[oracle@Udb ~]$ exp -help

Export: Release 11.2.0.1.0 - Production on Fri Sep 19 02:47:25 2014

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

You can let Export prompt you for parameters by entering the EXP
command followed by your username/password:

     Example: EXP SCOTT/TIGER

Or, you can control how Export runs by entering the EXP command followed
by various arguments. To specify parameters, you use keywords:

     Format:  EXP KEYWORD=value or KEYWORD=(value1,value2,...,valueN)
     Example: EXP SCOTT/TIGER GRANTS=Y TABLES=(EMP,DEPT,MGR)
               or TABLES=(T1:P1,T1:P2), if T1 is partitioned table

USERID must be the first parameter on the command line.

Keyword    Description (Default)      Keyword      Description (Default)
--------------------------------------------------------------------------
USERID     username/password          FULL         export entire file (N)
BUFFER     size of data buffer        OWNER        list of owner usernames
FILE       output files (EXPDAT.DMP)  TABLES       list of table names
COMPRESS   import into one extent (Y) RECORDLENGTH length of IO record
GRANTS     export grants (Y)          INCTYPE      incremental export type
INDEXES    export indexes (Y)         RECORD       track incr. export (Y)
DIRECT     direct path (N)            TRIGGERS     export triggers (Y)
LOG        log file of screen output  STATISTICS   analyze objects (ESTIMATE)
ROWS       export data rows (Y)       PARFILE      parameter filename
CONSISTENT cross-table consistency(N) CONSTRAINTS  export constraints (Y)

OBJECT_CONSISTENT    transaction set to read only during object export (N)
FEEDBACK             display progress every x rows (0)
FILESIZE             maximum size of each dump file
FLASHBACK_SCN        SCN used to set session snapshot back to
FLASHBACK_TIME       time used to get the SCN closest to the specified time
QUERY                select clause used to export a subset of a table
RESUMABLE            suspend when a space related error is encountered(N)
RESUMABLE_NAME       text string used to identify resumable statement
RESUMABLE_TIMEOUT    wait time for RESUMABLE 
TTS_FULL_CHECK       perform full or partial dependency check for TTS
VOLSIZE              number of bytes to write to each tape volume
TABLESPACES          list of tablespaces to export
TRANSPORT_TABLESPACE export transportable tablespace metadata (N)
TEMPLATE             template name which invokes iAS mode export

Export terminated successfully without warnings.

```

## 5. 数据导入

```bash
[oracle@Udb ~]$ imp -help

Import: Release 11.2.0.1.0 - Production on Fri Sep 19 02:51:11 2014

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

You can let Import prompt you for parameters by entering the IMP
command followed by your username/password:

     Example: IMP SCOTT/TIGER

Or, you can control how Import runs by entering the IMP command followed
by various arguments. To specify parameters, you use keywords:

     Format:  IMP KEYWORD=value or KEYWORD=(value1,value2,...,valueN)
     Example: IMP SCOTT/TIGER IGNORE=Y TABLES=(EMP,DEPT) FULL=N
               or TABLES=(T1:P1,T1:P2), if T1 is partitioned table

USERID must be the first parameter on the command line.

Keyword  Description (Default)       Keyword      Description (Default)
--------------------------------------------------------------------------
USERID   username/password           FULL         import entire file (N)
BUFFER   size of data buffer         FROMUSER     list of owner usernames
FILE     input files (EXPDAT.DMP)    TOUSER       list of usernames
SHOW     just list file contents (N) TABLES       list of table names
IGNORE   ignore create errors (N)    RECORDLENGTH length of IO record
GRANTS   import grants (Y)           INCTYPE      incremental import type
INDEXES  import indexes (Y)          COMMIT       commit array insert (N)
ROWS     import data rows (Y)        PARFILE      parameter filename
LOG      log file of screen output   CONSTRAINTS  import constraints (Y)
DESTROY                overwrite tablespace data file (N)
INDEXFILE              write table/index info to specified file
SKIP_UNUSABLE_INDEXES  skip maintenance of unusable indexes (N)
FEEDBACK               display progress every x rows(0)
TOID_NOVALIDATE        skip validation of specified type ids 
FILESIZE               maximum size of each dump file
STATISTICS             import precomputed statistics (always)
RESUMABLE              suspend when a space related error is encountered(N)
RESUMABLE_NAME         text string used to identify resumable statement
RESUMABLE_TIMEOUT      wait time for RESUMABLE 
COMPILE                compile procedures, packages, and functions (Y)
STREAMS_CONFIGURATION  import streams general metadata (Y)
STREAMS_INSTANTIATION  import streams instantiation metadata (N)
DATA_ONLY              import only data (N)
VOLSIZE                number of bytes in file on each volume of a file on tape

The following keywords only apply to transportable tablespaces
TRANSPORT_TABLESPACE import transportable tablespace metadata (N)
TABLESPACES tablespaces to be transported into database
DATAFILES datafiles to be transported into database
TTS_OWNERS users that own data in the transportable tablespace set

Import terminated successfully without warnings.
```
