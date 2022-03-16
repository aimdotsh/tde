   * [PDB 的创建](#pdb-的创建)
      * [1、 使用seed创建新的pdb](#1-使用seed创建新的pdb)
      * [2、 从本地 PDB 进行克隆](#2-从本地-pdb-进行克隆)
         * [2.1.源 pdb 未启用 tde 加密进行克隆](#21源-pdb-未启用-tde-加密进行克隆)
         * [2.2.源库启用 tde 加密进行克隆](#22源库启用-tde-加密进行克隆)
         * [2.2.1 钱包打开，克隆数据和不克隆数据进行测试](#221-钱包打开克隆数据和不克隆数据进行测试)
            * [克隆数据库不包含数据](#克隆数据库不包含数据)
            * [客户数据库包含数据](#客户数据库包含数据)
         * [2.2.2 钱包关闭进行测试](#222-钱包关闭进行测试)
         * [总结](#总结)
         * [数据库告警处理](#数据库告警处理)
            * [查看alert日志](#查看alert日志)
            * [查看视图信息](#查看视图信息)
            * [查看钱包状态](#查看钱包状态)
            * [首先从源库导出密钥](#首先从源库导出密钥)
            * [目标库导入](#目标库导入)
            * [然后导入密钥](#然后导入密钥)
            * [查看状态](#查看状态)
            * [重启数据库](#重启数据库)
            * [告警解决](#告警解决)
      * [3、 从远程 PDB 进行克隆](#3-从远程-pdb-进行克隆)
         * [3.1、创建 dblink](#31创建-dblink)
         * [3.2、确认dblink](#32确认dblink)
         * [3.4、启动数据库](#34启动数据库)
         * [3.5、导入密钥](#35导入密钥)
            * [3.5.1 源库导出密钥](#351-源库导出密钥)
            * [3.5.2 目标 PDB 打开 钱包](#352-目标-pdb-打开-钱包)
            * [3.5.3 目标 PDB 导入密钥](#353-目标-pdb-导入密钥)
            * [3.5.4目标 PDB 查看状态](#354目标-pdb-查看状态)
            * [3.5.5目标 PDB重启数据库](#355目标-pdb重启数据库)
      * [4、tde pdb 插拔测试](#4tde-pdb-插拔测试)
         * [4.1.插入PDB](#41插入pdb)
            * [4.1数据文件存储目录](#41数据文件存储目录)
            * [4.2操作示例](#42操作示例)
            * [目标库导入密钥](#目标库导入密钥)
            * [然后导入密钥](#然后导入密钥-1)
            * [查看状态](#查看状态-1)
            * [重启数据库](#重启数据库-1)
      * [总结](#总结-1)
         * [从源库导出密钥](#从源库导出密钥)
         * [目标库导入](#目标库导入-1)
            * [打开钱包](#打开钱包)
            * [导入密钥](#导入密钥)
         * [检查钱包状态](#检查钱包状态)
         * [重启数据库](#重启数据库-2)
         * [检查日志确认日志无误](#检查日志确认日志无误)



PDB的创建、插拔、克隆等功能测试

> https://docs.oracle.com/database/121/ADMIN/cdb_plug.htm#ADMIN13549
>
> 普通 pdb
>
> create pluggable database notedpdb admin user pdbadmin identified by "Password123";
> alter pluggable database notedpdb open  instances=all;
> alter pluggable database notedpdb save state instances=all;
>
> 导入示例数据
>
> @mksample Password123 Password123 Password123 Password123 Password123 Password123 Password123 Password123 users temp /tmp/log localhost:1521/notedpdb
>
> 通过 配置环境变量可以很方便的看到是在哪个 pdb 下面操作。
>
>  vi $ORACLE_HOME/sqlplus/admin/glogin.sql
>
> define _editor=vi
> set serveroutput on size 1000000
> set trimspool on
> set long 5000
> set pagesize 5000
> set linesize 256
> column plan_plus_exp format a80
> column global_name new_value gname
> alter session set nls_date_format='yyyy-mm-dd hh24:mi:ss';
> set termout off
> define gname=idle
> column global_name new_value gname
> select lower(user)||'@'||'('||(select distinct sid from v$mystat ) ||'_'||(select sys_context('USERENV','CON_NAME') from dual)||')' global_name from v$instance;
> set sqlprompt '&gname> '
> set termout on

## PDB 的创建 

### 1、 使用seed创建新的pdb



![Description of Figure 38-2 follows](http://pic.liups.com/images/2022/03/13/202203131603779.png)

最简单的创建pdb 语句

```sql
SQL> create pluggable database pdborcl admin user pdbadmin identified by pdbadmin ;
```



```sql
CREATE PLUGGABLE DATABASE "pdb01" ADMIN USER "pdbadm" IDENTIFIED BY "pdbadm"
  FILE_NAME_CONVERT=(
    '/oradata/tdecdb/sys/pdbseed/system01.dbf', '/oradata/tdecdb/sys/pdb01/system01.dbf',
    '/oradata/tdecdb/sys/pdbseed/pdbseed_temp012022-02-21_09-30-37-AM.dbf', '/oradata/tdecdb/sys/pdb01/temp01.dbf',
    '/oradata/tdecdb/sys/pdbseed/sysaux01.dbf', '/oradata/tdecdb/sys/pdb01/sysaux01.dbf'
  )
  STORAGE (
    MAXSIZE 20G
    MAX_SHARED_TEMP_SIZE 10G
  );
```



```sql
CREATE PLUGGABLE DATABASE salespdb ADMIN USER pdbadm IDENTIFIED BY pdbadm 
  STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 100M)
  DEFAULT TABLESPACE sales 
    DATAFILE '/oradata/tdecdb/salespdb/sales01.dbf' SIZE 20M AUTOEXTEND ON
  FILE_NAME_CONVERT = ('/oradata/tdecdb/sys/pdbseed/', '/oradata/tdecdb/salespdb/');
```



```sql
CREATE PLUGGABLE DATABASE pathprepdb ADMIN USER pdbadm IDENTIFIED BY pdbadm roles=(DBA)
  STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 10M)
  DEFAULT TABLESPACE sales 
    DATAFILE '/oradata/tdecdb/pathprepdb/sales01.dbf' SIZE 20M AUTOEXTEND ON ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT)
  PATH_PREFIX = '/oradata/tdecdb/pathprepdb/'
  FILE_NAME_CONVERT = ('/oradata/tdecdb/sys/pdbseed/', '/oradata/tdecdb/pathprepdb/');

  SQL> CREATE PLUGGABLE DATABASE pathprepdb ADMIN USER pdbadm IDENTIFIED BY pdbadm roles=(DBA)
  STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 10M)
  DEFAULT TABLESPACE sales
    DATAFILE '/oradata/tdecdb/pathprepdb/sales01.dbf' SIZE 20M AUTOEXTEND ON ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
  FILE_NAME_CONVERT = ('/oradata/tdecdb/sys/pdbseed/', '/oradata/tdecdb/pathprepdb/');  2    3    4
CREATE PLUGGABLE DATABASE pathprepdb ADMIN USER pdbadm IDENTIFIED BY pdbadm roles=(DBA)
*
ERROR at line 1:
ORA-03113: end-of-file on communication channel
Process ID: 5920
Session ID: 596 Serial number: 7339
```

 在创建表空间的时候指定加密表空间，竟然报07445了。应该是无法指定的，应该创建pdb之后 钱包是关闭的。

```shell
Dump file /u01/app/oracle/diag/rdbms/tdecdb/tdecdb/incident/incdir_30873/tdecdb_ora_6143_i30873.trc
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
ORACLE_HOME = /u01/app/oracle/product/12.1.0.2/dbhome_1
System name:    Linux
Node name:      tcloud
Release:        3.10.0-1160.45.1.el7.x86_64
Version:        #1 SMP Wed Oct 13 17:20:51 UTC 2021
Machine:        x86_64
Instance name: tdecdb
Redo thread mounted by this instance: 1
Oracle process number: 59
Unix process pid: 6143, image: oracle@tcloud (TNS V1-V3)


*** 2022-03-13 17:11:34.564
*** SESSION ID:(602.10788) 2022-03-13 17:11:34.564
*** CLIENT ID:() 2022-03-13 17:11:34.564
*** SERVICE NAME:(SYS$USERS) 2022-03-13 17:11:34.564
*** MODULE NAME:(sqlplus@tcloud (TNS V1-V3)) 2022-03-13 17:11:34.564
*** CLIENT DRIVER:(SQL*PLUS) 2022-03-13 17:11:34.564
*** ACTION NAME:() 2022-03-13 17:11:34.564
*** CONTAINER ID:(1) 2022-03-13 17:11:34.564

[TOC00000]
Jump to table of contents
Dump continued from file: /u01/app/oracle/diag/rdbms/tdecdb/tdecdb/trace/tdecdb_ora_6143.trc
[TOC00001]
ORA-07445: exception encountered: core dump [prsetsesc()+59] [SIGSEGV] [ADDR:0x40] [PC:0x80C08EB] [Address not mapped to object] []

[TOC00001-END]
[TOC00002]
========= Dump for incident 30873 (ORA 7445 [prsetsesc]) ========
[TOC00003]
----- Beginning of Customized Incident Dump(s) -----
Dumping swap information
Memory (Avail / Total) = 98.76M / 3788.92M
Swap (Avail / Total) = 2799.84M /  4096.00M
Exception [type: SIGSEGV, Address not mapped to object] [ADDR:0x40] [PC:0x80C08EB, prsetsesc()+59] [flags: 0x0, count: 1]
Registers:
%rax: 0x000000000000044b %rbx: 0x00007f793d560ac0 %rcx: 0x00007f7938b6e19f
%rdx: 0x00007f793d3e4af8 %rdi: 0x00007f793d548ca0 %rsi: 0x00007f793d560ac0
%rsp: 0x00007ffffbfa3ff0 %rbp: 0x00007ffffbfa4030  %r8: 0x0000000000000048
 %r9: 0x0000000005a95350 %r10: 0x0000000000000000 %r11: 0x000000000ef64fc8
"/u01/app/oracle/diag/rdbms/tdecdb/tdecdb/incident/incdir_30873/tdecdb_ora_6143_i30873.trc" 62161L, 3160770C                                                                              1,1           Top
Dump file /u01/app/oracle/diag/rdbms/tdecdb/tdecdb/incident/incdir_30873/tdecdb_ora_6143_i30873.trc
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
ORACLE_HOME = /u01/app/oracle/product/12.1.0.2/dbhome_1
System name:    Linux
```



```sql
CREATE PLUGGABLE DATABASE pathprepdb ADMIN USER pdbadm IDENTIFIED BY pdbadm roles=(DBA)
  STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 10M)
  DEFAULT TABLESPACE sales 
    DATAFILE '/oradata/tdecdb/pathprepdb/sales01.dbf' SIZE 20M AUTOEXTEND ON 
  FILE_NAME_CONVERT = ('/oradata/tdecdb/sys/pdbseed/', '/oradata/tdecdb/pathprepdb/');
```

```
CREATE PLUGGABLE DATABASE pathprepdb ADMIN USER pdbadm IDENTIFIED BY pdbadm roles=(DBA)
  STORAGE (MAXSIZE 2G MAX_SHARED_TEMP_SIZE 10M)
  DEFAULT TABLESPACE sales 
    DATAFILE '/oradata/tdecdb/pathprepdb/sales01.dbf' SIZE 20M AUTOEXTEND ON 
      PATH_PREFIX = '/oradata/tdecdb/pathprepdb/'
  FILE_NAME_CONVERT = ('/oradata/tdecdb/sys/pdbseed/', '/oradata/tdecdb/pathprepdb/');
```

创建pdb之后，pdb 是mount状态的，熟悉open database，查看 钱包状态 是 close

```sql
alter pluggable database pathprepdb open
```

```sql

sys@tdecdb(603)> select * from 	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID
-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------
FILE		     /etc/ORACLE/WALLETS/tdecdb/	      CLOSED			     UNKNOWN		  SINGLE    UNDEFINED	       0

sys@tdecdb(603)> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.

sys@tdecdb(603)> select * from 	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID
-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------
FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN_NO_MASTER_KEY	     PASSWORD		  SINGLE    UNDEFINED	       0


```



设置 MASTER_KEY

```sql
sys@tdecdb(603)>  ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY Password23 with backup;

keystore altered.

sys@tdecdb(603)> select * from 	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID
-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------
FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN			     PASSWORD		  SINGLE    NO		       0

```

就可以进行正常的pdb操作了。

使用seed创建新的 tde 加密pdb 完成。

删除 pdb

```sql
 alter Pluggable database PDBORCL close;
 drop pluggable database PDBORCL including datafiles;
```

### 2、 从本地 PDB 进行克隆

> ![Description of Figure 38-3 follows](http://pic.liups.com/images/2022/03/13/202203131726296.png)
>
> ```sql
> SQL>  CREATE PLUGGABLE DATABASE hrpdb FROM salespdb no data STORAGE unlimited;
> Pluggable database created.
> SYS@ora12c> col name for a15
> SYS@ora12c> select con_id, name,open_mode from v$containers;
>     CON_ID NAME            OPEN_MODE
> ---------- --------------- --------------------
>          1 CDB$ROOT        READ WRITE
>          2 PDB$SEED        READ ONLY
>          3 SALESPDB        READ ONLY
>          4 HRPDB           MOUNTED
> ```
>
> ```
>  show pdbs;
> 
>     CON_ID CON_NAME			  OPEN MODE  RESTRICTED
> ---------- ------------------------------ ---------- ----------
> 	 2 PDB$SEED			  READ ONLY  NO
> 	 3 TDEPDB			  READ WRITE NO
> 	 4 TDEPDB2			  READ WRITE NO
> 	 6 SALESPDB			  READ WRITE NO
> 
> CREATE PLUGGABLE DATABASE hrpdb_tde FROM TDEPDB  STORAGE unlimited;
> ```



#### 2.1.源 pdb 未启用 tde 加密进行克隆

```sql
SQL>  CREATE PLUGGABLE DATABASE hrpdb FROM salespdb no data STORAGE unlimited;
Pluggable database created.
	 2 PDB$SEED			  READ ONLY  NO
	 3 TDEPDB			  READ WRITE NO
	 4 TDEPDB2			  READ WRITE NO
	 5 HRPDB			  MOUNTED
	 6 SALESPDB			  READ WRITE NO
```

库能够正常打开。

#### 2.2.源库启用 tde 加密进行克隆

分两种情况进行测试

#### 2.2.1 钱包打开，克隆数据和不克隆数据进行测试

##### 克隆数据库不包含数据

```sql
 CREATE PLUGGABLE DATABASE hrpdbtdenodata FROM TDEPDB no data STORAGE unlimited;
Pluggable database created.
sys@tdecdb(26)>  alter Pluggable database hrpdbtdenodata open;

Warning: PDB altered with errors.
	6 HRPDBTDENODATA		  READ WRITE YES
```

##### 客户数据库包含数据

```sql
CREATE PLUGGABLE DATABASE hrpdb_tde FROM TDEPDB  STORAGE unlimited;
Pluggable database created.
sys@tdecdb(26)>  alter Pluggable database hrpdbtdenodata open;

Warning: PDB altered with errors.
	6 HRPDB_TDE		  READ WRITE YES
```

不管包含数据还是不包含数据，库可以 open，但是有警告，其状态 RESTRICTED 为yes 

```sql
  6 HRPDBTDENODATA		  READ WRITE YES
  7 HRPDB_TDE			      READ WRITE YES
```



#### 2.2.2 钱包关闭进行测试

源库钱包关闭

```sql
sys@tdecdb(23)> /

WRL_TYPE	     WRL_PARAMETER		    STATUS
-------------------- ------------------------------ ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/    CLOSED

sys@tdecdb(23)>
```

```sql
 CREATE PLUGGABLE DATABASE hrpdb_close FROM TDEPDB no data STORAGE unlimited;

Pluggable database created.
```

```sql
CREATE PLUGGABLE DATABASE hrpdb_tdeclose FROM TDEPDB  STORAGE unlimited;

Pluggable database created.
sys@tdecdb(23)> alter session set container=HRPDB_TDECLOSE;

Session altered.

sys@tdecdb(23)> startup

Warning: PDB altered with errors.

Pluggable Database opened.
sys@tdecdb(23)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 9 HRPDB_TDECLOSE		  READ WRITE YES
sys@tdecdb(23)> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
[oracle@tcloud_for_12c_tdecdb:/home/oracle]$ sqlplus hr/Password123@localhost:1521/HRPDB_TDECLOSE

SQL*Plus: Release 12.1.0.2.0 Production on Mon Mar 14 07:34:55 2022

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Last Successful login time: Mon Mar 14 2022 07:24:13 +08:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options


Session altered.

hr@tdecdb(23)> desc employees;
 Name																		    Null?    Type
 -------------------------------------------------------------------------------------------------------------------------------------------------- -------- ---------------------------------------------------------------------------------------------------
 EMPLOYEE_ID																	    NOT NULL NUMBER(6)
 FIRST_NAME																		     VARCHAR2(20) ENCRYPT
 LAST_NAME																	    NOT NULL VARCHAR2(25)
 EMAIL																		    NOT NULL VARCHAR2(25)
 PHONE_NUMBER																		     VARCHAR2(20)
 HIRE_DATE																	    NOT NULL DATE
 JOB_ID 																	    NOT NULL VARCHAR2(10)
 SALARY 																		     NUMBER(8,2) ENCRYPT
 COMMISSION_PCT 																	     NUMBER(2,2)
 MANAGER_ID																		     NUMBER(6)
 DEPARTMENT_ID																		     NUMBER(4)
 SSN																			     VARCHAR2(11)

hr@tdecdb(23)> select * from employees where rownum<=2;
select * from employees where rownum<=2
              *
ERROR at line 1:
ORA-28365: wallet is not open


hr@tdecdb(23)>
```

#### 总结

源库钱包不管是否关闭，克隆的数据库是否包含数据，都可以进行克隆，库可以 open，状态 RESTRICTED 为yes ，会有个  Warning: PDB altered with errors. 查询钱包状态是 close，无法直接查询加密数据

#### 数据库告警处理

##### 查看alert日志

```shell
***************************************************************
WARNING: Pluggable Database HRPDB_CLOSE with pdb id - 8 is
         altered with errors or warnings. Please look into
         PDB_PLUG_IN_VIOLATIONS view for more details.
***************************************************************
```

##### 查看视图信息

```sql
 select * from PDB_PLUG_IN_VIOLATIONS;
TIME			       NAME	       CAUSE		    TYPE      ERROR_NUMBER	 LINE MESSAGE						 STATUS    ACTION
------------------------------ --------------- -------------------- --------- ------------ ---------- -------------------------------------------------- --------- --------------------------------------------------
14-MAR-22 07.27.02.585205 AM   HRPDB_TDE       Wallet Key Needed    ERROR		 0	    1 PDB needs to import keys from source.		 PENDING   Import keys from source.
14-MAR-22 07.34.35.744234 AM   HRPDB_TDECLOSE  Wallet Key Needed    ERROR		 0	    1 PDB needs to import keys from source.		 PENDING   Import keys from source.
14-MAR-22 07.37.33.449191 AM   HRPDB_CLOSE     Wallet Key Needed    ERROR		 0	    1 PDB needs to import keys from source.		 PENDING   Import keys from source.

```

提示： PDB needs to import keys from source

##### 查看钱包状态

```
sys@tdecdb(219)>  select * from v$encryption_wallet;

WRL_TYPE	     WRL_PARAMETER					STATUS			       WALLET_TYPE	    WALLET_OR FULLY_BAC     CON_ID
-------------------- -------------------------------------------------- ------------------------------ -------------------- --------- --------- ----------
FILE		     /etc/ORACLE/WALLETS/tdecdb/			CLOSED			       UNKNOWN		    SINGLE    UNDEFINED 	 0
```

状态是 close，但是 WALLET_TYPE 为 UNKNOWN。

> 导入 key：
>
> ```sql
> administer key management import encryption keys with secret "password" from '/etc/ORACLE/WALLETS/tdecdb/ewallet.p12' force
> keystore identified by delphixclone with backup;
> ```
>
> ```sql
> select * from v$encryption_wallet;
> ```

##### 首先从源库导出密钥

源库导出密钥

```sql
ADMINISTER KEY MANAGEMENT EXPORT ENCRYPTION KEYS WITH SECRET "mySecret" TO '/tmp/export.p12' IDENTIFIED BY Password23;
```

##### 目标库导入

导入之前需要保证 钱包是 open 状态，否则

```shell
ERROR at line 1:
ORA-46658: keystore not open in the container
```

```sql
sys@tdecdb(26)> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

##### 然后导入密钥

```sql
sys@tdecdb(26)>  administer key management import encryption keys  with secret "mySecret" from '/tmp/export.p12' identified by  Password23 with backup;

keystore altered.
```

##### 查看状态

```sql
select * from 	V$ENCRYPTION_WALLET;
WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID

-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------

FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN			     PASSWORD		  SINGLE    NO		       0

```

##### 重启数据库

```sql
sys@tdecdb(592)> shutdown immediate
Pluggable Database closed.
sys@tdecdb(592)> startup
Pluggable Database opened.
可以看到启动的时候没有告警了。
sys@tdecdb(592)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 7 HRPDB_TDE			  READ WRITE NO
sys@tdecdb(592)>
```

导入数据之后查看日志告警

```sql
  1*  select * from PDB_PLUG_IN_VIOLATIONS
sys@tdecdb(26)> /

TIME			       NAME			 CAUSE		      TYPE	ERROR_NUMBER	   LINE MESSAGE 					   STATUS    ACTION
------------------------------ ------------------------- -------------------- --------- ------------ ---------- -------------------------------------------------- --------- -----------------------------------
14-MAR-22 07.34.35.744234 AM   HRPDB_TDECLOSE		 Wallet Key Needed    ERROR		   0	      1 PDB needs to import keys from source.		   PENDING   Import keys from source.
14-MAR-22 08.38.58.435334 AM   HRPDB_CLOSE		 Wallet Key Needed    ERROR		   0	      1 PDB needs to import keys from source.		   PENDING   Import keys from source.
14-MAR-22 08.56.37.229458 AM   HRPDB_TDE		 Wallet Key Needed    ERROR		   0	      1 PDB needs to import keys from source.		   RESOLVED  Import keys from source.
15-MAR-22 10.58.46.331944 AM   HRPDBTDENODATA		 Wallet Key Needed    ERROR		   0	      1 PDB needs to import keys from source.		   PENDING   Import keys from source.

sys@tdecdb(26)>
```

##### 告警解决

可以看到状态 变成了 **RESOLVED** 已解决。

```sql
hr@tdecdb(11)> desc employees;
 Name																		    Null?    Type
 -------------------------------------------------------------------------------------------------------------------------------------------------- -------- ---------------------------------------------------------------------------------------------------
 EMPLOYEE_ID																	    NOT NULL NUMBER(6)
 FIRST_NAME																		     VARCHAR2(20) ENCRYPT
 LAST_NAME																	    NOT NULL VARCHAR2(25)
 EMAIL																		    NOT NULL VARCHAR2(25)
 PHONE_NUMBER																		     VARCHAR2(20)
 HIRE_DATE																	    NOT NULL DATE
 JOB_ID 																	    NOT NULL VARCHAR2(10)
 SALARY 																		     NUMBER(8,2) ENCRYPT
 COMMISSION_PCT 																	     NUMBER(2,2)
 MANAGER_ID																		     NUMBER(6)
 DEPARTMENT_ID																		     NUMBER(4)
 SSN																			     VARCHAR2(11)

hr@tdecdb(11)>
```

### 3、 从远程 PDB 进行克隆

> 命令跟从本地克隆是差不多的，就是把本地的 pdb 名字换成了远程pdb名字@dblink。
>
> ```
>  CREATE PLUGGABLE DATABASE hrpdb FROM salespdb no data STORAGE unlimited;
>  CREATE PLUGGABLE DATABASE ORA12CPDB2 FROM PDBTEST@dlink NO DATA STORAGE unlimited;
> ```



#### 3.1、创建 dblink

```
CREATE DATABASE LINK dl4tdepdb CONNECT TO system IDENTIFIED BY Password123 USING 'TDEPDB';
```

#### 3.2、确认dblink

```
sys@(11_CDB$ROOT)> select NAME,OPEN_MODE,RESTRICTED from  v$pdbs@dl4tdepdb;
NAME			       OPEN_MODE  RESTRICTED
------------------------------ ---------- --------------------
TDEPDB			       READ WRITE NO
```

3.3、克隆远程 PDB

```
 CREATE PLUGGABLE DATABASE remtdepdb FROM TDEPDB@dl4tdepdb NO DATA STORAGE unlimited;
```

需要注意的是源 PDB 需要在open状态，否则会报

```
ERROR at line 1:
ORA-17627: ORA-01033: ORACLE initialization or shutdown in progress
ORA-17629: Cannot connect to the remote database server
```

```
sys@(11_CDB$ROOT)>  CREATE PLUGGABLE DATABASE remtdepdb FROM TDEPDB@dl4tdepdb NO DATA STORAGE unlimited;

Pluggable database created.

sys@(11_CDB$ROOT)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 TDEPDB			  READ WRITE NO
	12 REMTDEPDB			  MOUNTED
```

可以看到 REMTDEPDB 已经克隆完成，启动数据库

#### 3.4、启动数据库

```
alter pluggable database REMTDEPDB open  instances=all;
alter pluggable database REMTDEPDB save state instances=all;
```



```
sys@(11_CDB$ROOT)> alter pluggable database REMTDEPDB open  instances=all;

Warning: PDB altered with errors.

sys@(11_CDB$ROOT)>  select * from PDB_PLUG_IN_VIOLATIONS;

TIME					 NAME		      CAUSE		   TYPE      ERROR_NUMBER	LINE MESSAGE						STATUS	  ACTION
---------------------------------------- -------------------- -------------------- --------- ------------ ---------- -------------------------------------------------- --------- ------------------------------
14-MAR-22 07.34.35.744234 AM		 HRPDB_TDECLOSE       Wallet Key Needed    ERROR		0	   1 PDB needs to import keys from source.		PENDING   Import keys from source.
14-MAR-22 08.38.58.435334 AM		 HRPDB_CLOSE	      Wallet Key Needed    ERROR		0	   1 PDB needs to import keys from source.		PENDING   Import keys from source.
14-MAR-22 08.56.37.229458 AM		 HRPDB_TDE	      Wallet Key Needed    ERROR		0	   1 PDB needs to import keys from source.		RESOLVED  Import keys from source.
15-MAR-22 10.58.46.331944 AM		 HRPDBTDENODATA       Wallet Key Needed    ERROR		0	   1 PDB needs to import keys from source.		PENDING   Import keys from source.
15-MAR-22 02.55.22.545426 PM		 HRPDB		      Wallet Key Needed    ERROR		0	   1 PDB needs to import keys from source.		PENDING   Import keys from source.
16-MAR-22 02.49.04.258375 PM		 REMTDEPDB	      Wallet Key Needed    ERROR		0	   1 PDB needs to import keys from source.		PENDING   Import keys from source.

6 rows selected.

sys@(11_CDB$ROOT)> l
  1*  select * from PDB_PLUG_IN_VIOLATIONS
sys@(11_CDB$ROOT)>
```

启动数据库之后仍然有告警，通过查看 PDB_PLUG_IN_VIOLATIONS 视图，仍然需要导入密钥。

#### 3.5、导入密钥

首先从源库导出密钥

##### 3.5.1 源库导出密钥

```sql
ADMINISTER KEY MANAGEMENT EXPORT ENCRYPTION KEYS WITH SECRET "mySecret" TO '/tmp/export.p12' IDENTIFIED BY Password23;
```

目标库导入

导入之前需要保证 钱包是 open 状态，否则

```shell
ERROR at line 1:
ORA-46658: keystore not open in the container
```

##### 3.5.2 目标 PDB 打开 钱包

```sql
sys@(404_REMTDEPDB)> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

##### 3.5.3 目标 PDB 导入密钥

```sql
sys@(404_REMTDEPDB)> administer key management import encryption keys  with secret "mySecret" from '/tmp/export.p12' identified by  Password23 with backup;

keystore altered.
```

##### 3.5.4目标 PDB 查看状态

```sql

sys@(404_REMTDEPDB)> set lines 200
sys@(404_REMTDEPDB)> col WRL_PARAMETER for a50
sys@(404_REMTDEPDB)> select * from 	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER					STATUS			       WALLET_TYPE	    WALLET_OR FULLY_BAC     CON_ID
-------------------- -------------------------------------------------- ------------------------------ -------------------- --------- --------- ----------
FILE		     /etc/ORACLE/WALLETS/tdecdb/			OPEN			       PASSWORD 	    SINGLE    NO		 0

sys@(404_REMTDEPDB)>
```

##### 3.5.5目标 PDB重启数据库

```sql
sys@(404_REMTDEPDB)>  alter pluggable database REMTDEPDB close immediate  instances=all;
Pluggable database altered.
sys@(404_REMTDEPDB)>  alter pluggable database REMTDEPDB open   instances=all;
Pluggable database altered.
可以看到启动的时候没有告警了。
sys@tdecdb(592)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 7 HRPDB_TDE			  READ WRITE NO
sys@tdecdb(592)>
```

导入数据之后查看日志告警,已经标记为已解决。

总结：远程克隆和本地客户基本相同，都需要导入钱包密钥，克隆关键字有本地pdb名称换成了远程pdb的名称加dblink的方式。

### 4、tde pdb 插拔测试

> https://docs.oracle.com/database/121/ADMIN/cdb_plug.htm#ADMIN13855
>

#### 4.1.插入PDB

在插入PDB时有以下两个因素要考虑。

源CDB和目标CDB要有相同的字节存储顺序，字符集要么相同，要么是子集。本次先测试没有配置tde加密的pdb 插拔操作。

       操作时在create pluggable database 中使用using子句，指定一个XML的PDB元数据文件或一个压缩的PDB归档文件(.pdb类型的文件)来创建一个PDB。
       XML元数据文件描述了未插入的PDB和与PDB相关的文件(例如数据文件和钱包文件)。
       归档文件包括 XML 元数据文件和PDB文件。
       当指定 XML 元数据时，XML 文件包含PDB文件的完整路径。
       当指定 .pdb 归档文件时，XML 元数据文件只包含相对文件名。

##### 4.1数据文件存储目录

       通过.pdb归档文件插入PDB，不需要添加参数来指定数据文件的存放位置，因为oracle会将归档文件解压到当前目录，当前目录就是数据文件的存放位置。
    
        通过XML元数据文件插入PDB，需要添加source_file_name_convert 或者source_file_directory字句来指定文件存放路径。因为XML元数据文件里存放的是PDB之前的数据文件的绝对路径，该路径不一定存在所以需要指定该参数。

##### 4.2操作示例

本示例将 tdepdb 插拔到  相同cdb 实例下面的 cbtdepdb 。

**(1)拔出PDB  被拔出的PDB只能被删除,不能做其它操作,比如打开等.** 

```sql
sys@(25_TDEPDB)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 TDEPDB			  READ WRITE NO

```

拔出PDB有两种方式：XML方式和归档文件方式。

XML方式将PDB从CDB中拔出

```
sys@(25_TDEPDB)> select name from v$datafile;

NAME
--------------------------------------------------------------------------------
/oradata/tdecdb/undo/undotbs01.dbf
/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_system_k15x87v3_.dbf
/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_sysaux_k15x87vb_.dbf
/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_test_tde_k18b3y43_.dbf

关闭需要拔出的PDB
sys@(25_TDEPDB)> alter pluggable database TDEPDB close;
Pluggable database altered.
```

将名为 TDEPDB 的PDB拔出，并生成名为tdepdb.xml的xml文件

需要在 cdb 操作

```sql
sys@(25_CDB$ROOT)> show con_name

CON_NAME
------------------------------
CDB$ROOT
sys@(25_CDB$ROOT)>

sys@(25_CDB$ROOT)>  alter pluggable database tdepdb unplug into '/tmp/tdepdb.xml';
Pluggable database altered.


sys@(25_CDB$ROOT)> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 TDEPDB			  MOUNTED


sys@(25_CDB$ROOT)>  select PDB_ID,PDB_NAME,STATUS from dba_pdbs;
    PDB_ID PDB_NAME		STATUS
---------- -------------------- ---------
	 3 TDEPDB		UNPLUGGED
	 2 PDB$SEED		NORMAL
可以看到 TDEPDB 的状态为 UNPLUGGED 
```

将 pdb 拔下之后，是无法再打开次数据pdb，只能进行drop操作了。

```
sys@(214_CDB$ROOT)> alter session set container=TDEPDB;
Session altered.
sys@(214_CDB$ROOT)> startup
ORA-65086: cannot open/close the pluggable database
```



**(4)插入目标CDB中**

使用XML文件, 由于是在通一个cdb 中进行测试，需要将源pdb 删除掉，删除的时候保留数据文件，如果不删除源pdb 会报以下

```
sys@(591_CDB$ROOT)>  create pluggable database cbtdepdb using  '/tmp/tdepdb.xml'  nocopy tempfile reuse;
 create pluggable database cbtdepdb using  '/tmp/tdepdb.xml'  nocopy tempfile reuse
*
ERROR at line 1:
ORA-65122: Pluggable database GUID conflicts with the GUID of an existing container.
```

删除源 PDB保留数据文件

```
sys@(11_CDB$ROOT)>  drop pluggable database TDEPDB  keep datafiles;

Pluggable database dropped.
```

插入目标 CDB

```sql

sys@(11_CDB$ROOT)> create pluggable database cbtdepdb using  '/tmp/tdepdb.xml'  nocopy tempfile reuse;

Pluggable database created.

sys@(11_CDB$ROOT)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 CBTDEPDB			  MOUNTED
SQL> alter session set container=CBTDEPDB;
Session altered.
SQL> select name from v$datafile;
NAME
----------------------------------------------------------------------------------------------------
/oradata/tdecdb/undo/undotbs01.dbf
/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_system_k15x87v3_.dbf
/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_sysaux_k15x87vb_.dbf
/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_test_tde_k18b3y43_.dbf
```

检查插入的PDB

打开目标数据库

```
sys@(11_CDB$ROOT)> alter pluggable database CBTDEPDB open;
Warning: PDB altered with errors.
```

同样显示有告警

```
sys@(11_CDB$ROOT)> 
set linesize 1000;
column MESSAGE format a50;
column name format a16;
column ACTION format a64;
column type format a16;
column cause format a24;

sys@(11_CDB$ROOT)> select name,cause,type,status,message,action from pdb_plug_in_violations where name ='CBTDEPDB';

NAME		 CAUSE			  TYPE		   STATUS    MESSAGE						ACTION
---------------- ------------------------ ---------------- --------- -------------------------------------------------- ----------------------------------------------------------------
CBTDEPDB	 Wallet Key Needed	  ERROR 	   PENDING   PDB needs to import keys from source.		Import keys from source.

```

导入密钥key

```

```

##### 目标库导入密钥

导入之前需要保证 钱包是 open 状态，否则

```shell
ERROR at line 1:
ORA-46658: keystore not open in the container
```

```sql
sys@(11_CDB$ROOT)> conn sys/Password123@127.0.0.1/cbtdepdb as sysdba
Connected.
sys@(11_CBTDEPDB)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 CBTDEPDB			  READ WRITE YES
sys@(11_CBTDEPDB)>
sys@tdecdb(26)> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

##### 然后导入密钥

```sql
sys@tdecdb(26)>  administer key management import encryption keys  with secret "mySecret" from '/tmp/export.p12' identified by  Password23 with backup;

keystore altered.
```

##### 查看状态

```sql
select * from 	V$ENCRYPTION_WALLET;
WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID

-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------

FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN			     PASSWORD		  SINGLE    NO		       0

```

##### 重启数据库

```sql
sys@(11_CBTDEPDB)> alter pluggable database  cbtdepdb close immediate;

Pluggable database altered.

sys@(11_CBTDEPDB)> alter pluggable database  cbtdepdb open ;

Pluggable database altered.


sys@(11_CBTDEPDB)>  show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 CBTDEPDB			  READ WRITE NO
sys@(11_CBTDEPDB)>
```

插拔数据库测试完成。



### 总结

通过以上测试：数据库的创建、插拔、克隆等，如果源 PDB 启用了 tde，那么在跟普通PDB的操作多了一步导入密钥key的操作。

#### 从源库导出密钥

源库导出密钥

```sql
ADMINISTER KEY MANAGEMENT EXPORT ENCRYPTION KEYS WITH SECRET "mySecret" TO '/tmp/export.p12' IDENTIFIED BY Password23;
```

#### 目标库导入

导入之前需要保证 钱包是 open 状态，否则

```shell
ERROR at line 1:
ORA-46658: keystore not open in the container
```

##### 打开钱包

```sql
sys@tdecdb(26)> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

##### 导入密钥

```sql
sys@tdecdb(26)>  administer key management import encryption keys  with secret "mySecret" from '/tmp/export.p12' identified by  Password23 with backup;

keystore altered.
```

#### 检查钱包状态

```sql
select * from 	V$ENCRYPTION_WALLET;
WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID

-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------

FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN			     PASSWORD		  SINGLE    NO		       0

```

#### 重启数据库

```sql
sys@tdecdb(592)> shutdown immediate
Pluggable Database closed.
sys@tdecdb(592)> startup
Pluggable Database opened.
可以看到启动的时候没有告警了。
sys@tdecdb(592)> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 7 HRPDB_TDE			  READ WRITE NO
sys@tdecdb(592)>
```

#### 检查日志确认日志无误

```
set linesize 1000;
column MESSAGE format a50;
column name format a16;
column ACTION format a64;
column type format a16;
column cause format a24;

sys@(11_CDB$ROOT)> select name,cause,type,status,message,action from pdb_plug_in_violations where name ='CBTDEPDB';
```

