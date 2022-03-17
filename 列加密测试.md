[TOC]



## 列加密测试

本次采用 oracle 12.1.0.2 hr 示例数据进行测试。

示例数据导入：https://github.com/oracle/db-sample-schemas/tree/v21.1

>  @mksample Password123 Password123 Password123 Password123 Password123 Password123 Password123 Password123 users temp /tmp/log localhost:1521/TDEPDB

|                 |                              |                                                              |      |
| --------------- | ---------------------------- | :----------------------------------------------------------- | ---- |
| 列加密          | 增加一个加密列               | 为了TDE能够正常工作，钱夹必须被打开。如果钱夹被关闭，还是可以访问没有加密的列，但不能够访问加密的列。在表已存在情况下增加一加密的列。 | √    |
|                 | 修改表中未加密的列           | 对数据库中业务用户下的表中普通列进行加密。                   | √    |
|                 | 取消加密列                   | 取消数据库中业务用户下的表中列的加密。                       | √    |
|                 | 索引列加密                   | 对数据库中业务用户下表的索引列进行加密，测试加密后对索引类型，索引使用，SQL执行效率，SQL执行计划的影响。 | √    |
|                 | 主外键加密                   | 对数据库中业务用户下表的主外键进行加密，测试主外键加密对业务，SQL执行计划等影响。 |      |
|                 | LOB字段加密                  | 对数据库中业务用户下表的LOB字段加密，测试加密后业务对LOB访问和日常维护。 |      |
|                 | 列加密支持的数据库类型       | 注释1                                                        |      |
| 加密与DBLINK    | 加密后测试数据库之前的DBLINK | 数据库加密后测试不同数据库版本直接使用DBLINK                 | √    |
| 加密与IN-Mmeory | 加密后IN-Memory功能测试      | 数据库开启加密后IN-Memory功能的测试                          |      |
| 加密与物化视图  | 加密物化视图的测试           | 数据库开启加密后物化视图的维护和使用测试                     | √    |
| 加密与分区表    | 加密后分区表的测试           | 加密后分区表的维护和使用测试                                 |      |

[^注释1]: Oracle官方支持列加密的字段类型有如下几种： • BINARY_DOUBLE • BINARY_FLOAT • CHAR • DATE • INTERVAL DAY TO SECOND • INTERVAL YEAR TO MONTH • LOBs (Internal LOBs and SECUREFILE LOBs Only) • NCHAR • NUMBER • NVARCHAR2 • RAW • tmcSTAMP (includes tmcSTAMP WITH tmc ZONE and tmcSTAMP WITH LOCAL tmc ZONE) • VARCHAR2增加一个加密列

### 增加一个加密列

```
SQL> desc EMPLOYEES;
 Name					   Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID				   NOT NULL NUMBER(6)
 FIRST_NAME					    VARCHAR2(20)
 LAST_NAME				   NOT NULL VARCHAR2(25)
 EMAIL					   NOT NULL VARCHAR2(25)
 PHONE_NUMBER					    VARCHAR2(20)
 HIRE_DATE				   NOT NULL DATE
 JOB_ID 				   NOT NULL VARCHAR2(10)
 SALARY 					    NUMBER(8,2)
 COMMISSION_PCT 				    NUMBER(2,2)
 MANAGER_ID					    NUMBER(6)
 DEPARTMENT_ID					    NUMBER(4)
```

```
ALTER TABLE employee ADD (ssn VARCHAR2(11) ENCRYPT);
```

```
SQL> ALTER TABLE EMPLOYEES  ADD (ssn VARCHAR2(11) ENCRYPT);

Table altered.

SQL>  desc EMPLOYEES;
 Name					   Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID				   NOT NULL NUMBER(6)
 FIRST_NAME					    VARCHAR2(20)
 LAST_NAME				   NOT NULL VARCHAR2(25)
 EMAIL					   NOT NULL VARCHAR2(25)
 PHONE_NUMBER					    VARCHAR2(20)
 HIRE_DATE				   NOT NULL DATE
 JOB_ID 				   NOT NULL VARCHAR2(10)
 SALARY 					    NUMBER(8,2)
 COMMISSION_PCT 				    NUMBER(2,2)
 MANAGER_ID					    NUMBER(6)
 DEPARTMENT_ID					    NUMBER(4)
 SSN						    VARCHAR2(11) ENCRYPT
```

### 更新加密列 

```
update EMPLOYEES set ssn='sn'||EMPLOYEE_ID

SQL> update EMPLOYEES set ssn='sn'||EMPLOYEE_ID
  2  ;

107 rows updated.

SQL> commit;

Commit complete.

SQL> select EMPLOYEE_ID,ssn from EMPLOYEES where rownum<2;

EMPLOYEE_ID SSN

----------- -----------

	100 sn100
```

```
 select EMPLOYEE_ID ,EMAIL,PHONE_NUMBER,SALARY from EMPLOYEES;
```

### 测试钱包关闭/打开的情况下访问非加密列及加密列的情况

#### 钱包打开的情况访问数据测试

```
SQL> set lines 200
SQL> col WRL_PARAMETER for a60
SQL> /

WRL_TYPE	     WRL_PARAMETER						  STATUS
-------------------- ------------------------------------------------------------ ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/				  OPEN


SQL> col LAST_NAME for a20
SQL> select EMPLOYEE_ID,last_name,PHONE_NUMBER,SALARY,ssn  from EMPLOYEES where rownum<3;

EMPLOYEE_ID LAST_NAME		 PHONE_NUMBER		  SALARY SSN
----------- -------------------- -------------------- ---------- -----------
	100 King		 515.123.4567		   24000 sn100
	101 Kochhar		 515.123.4568		   17000 sn101

```

#### 钱包关闭的情况访问数据测试

```
login as sysdba
 ADMINISTER KEY MANAGEMENT SET KEYSTORE CLOSE IDENTIFIED BY "Password23" ;
```

```
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE CLOSE IDENTIFIED BY "Password23" ;

keystore altered.

SQL> select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER						  STATUS
-------------------- ------------------------------------------------------------ ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/				  CLOSED
SQL> col LAST_NAME for a20
SQL> select EMPLOYEE_ID,last_name,PHONE_NUMBER,SALARY,ssn  from EMPLOYEES where rownum<3;
select EMPLOYEE_ID,last_name,PHONE_NUMBER,SALARY,ssn  from EMPLOYEES where rownum<3
*
ERROR at line 1:
ORA-28365: wallet is not open


SQL> select EMPLOYEE_ID,last_name,PHONE_NUMBER,SALARY from EMPLOYEES where rownum<3;

EMPLOYEE_ID LAST_NAME		 PHONE_NUMBER		  SALARY
----------- -------------------- -------------------- ----------
	100 King		 515.123.4567		   24000
	101 Kochhar		 515.123.4568		   17000


SQL>   update EMPLOYEES set ssn='sn'||EMPLOYEE_ID;
  update EMPLOYEES set ssn='sn'||EMPLOYEE_ID
         *
ERROR at line 1:
ORA-28365: wallet is not open

SQL>  insert into EMPLOYEES as select * from EMPLOYEES where rownum<2;
 insert into EMPLOYEES as select * from EMPLOYEES where rownum<2
                       *
ERROR at line 1:
ORA-00926: missing VALUES keyword


```

结论：

钱包关闭的情况测试发现：在钱包关闭的情况下可以访问没有加密的列，但不能够访问加密的列，同样 dml 操作涉及到加密列也是无法进行操作加密列，但是如果 dml 操作中没有加密列 dml操作是可以进行的，比如删除、更新数据。

```
SQL> update EMPLOYEES set LAST_NAME=LAST_NAME||'tdeclose';
106 rows updated.
SQL> select LAST_NAME from EMPLOYEES where rownum<2;
LAST_NAME
--------------------
Abeltdetdeclose
SQL> delete from EMPLOYEES where EMPLOYEE_ID=206;
1 row deleted.
SQL> delete from EMPLOYEES where ssn='sn100';
delete from EMPLOYEES where ssn='sn100'
            *
ERROR at line 1:
ORA-28365: wallet is not open
但是如果删除表数据的时候 where 条件有加密列是无法进行删除操作的。但是可以通过其他where 条件来删除数据。
```

### 修改表中未加密的列



> ```
> --包含分区、索引、索引字段
> --先替换掉下面define值
> define owner=HR
> define table_name=EMPLOYEES
> --先替换掉上面define值
> set linesize 160
> col DATA_TYPE for a15
> set pagesize 10000
> col COLUMN_NAME for a30
> col col for a30
> select TABLE_NAME,    NUM_ROWS,    BLOCKS,    EMPTY_BLOCKS,    CHAIN_CNT,    AVG_ROW_LEN,    GLOBAL_STATS,    SAMPLE_SIZE,   to_char(t.last_analyzed,'MM-DD-YYYY') from dba_tables t where    owner = upper('&owner') and table_name = upper('&table_name');
> select COLUMN_NAME, DATA_TYPE,   NUM_DISTINCT,    DENSITY,    NUM_BUCKETS,    NUM_NULLS,    SAMPLE_SIZE,    to_char(t.last_analyzed,'MM-DD-YYYY') from dba_tab_columns t where   owner = upper('&owner') and table_name = upper('&table_name');
> select INDEX_NAME,    BLEVEL BLev,    LEAF_BLOCKS,    DISTINCT_KEYS,    NUM_ROWS,    AVG_LEAF_BLOCKS_PER_KEY,    AVG_DATA_BLOCKS_PER_KEY,    CLUSTERING_FACTOR,    to_char(t.last_analyzed,'MM-DD-YYYY') from    dba_indexes t where    table_name =  upper('&table_name') and table_owner = upper('&owner');
> select /*+ first_rows use_nl(i,t)*/ i.INDEX_NAME,    i.COLUMN_NAME,    i.COLUMN_POSITION,    decode(t.DATA_TYPE,           'NUMBER',t.DATA_TYPE||'('||           decode(t.DATA_PRECISION,                  null,t.DATA_LENGTH||')',                  t.DATA_PRECISION||','||t.DATA_SCALE||')'),                  'DATE',t.DATA_TYPE,                  'LONG',t.DATA_TYPE,                  'LONG RAW',t.DATA_TYPE,                  'ROWID',t.DATA_TYPE,                  'MLSLABEL',t.DATA_TYPE,                  t.DATA_TYPE||'('||t.DATA_LENGTH||')') ||' '||           decode(t.nullable,                  'N','NOT NULL',                  'n','NOT NULL',                  NULL) col   from     dba_ind_columns i,    dba_tab_columns t where i.index_owner=t.owner and     i.table_name = upper('&table_name') and i.index_owner = upper('&owner') and i.table_name = t.table_name and i.column_name = t.column_name order by index_name,column_position;
> 
> ```

```
COLUMN_NAME		       DATA_TYPE       NUM_DISTINCT    DENSITY NUM_BUCKETS  NUM_NULLS SAMPLE_SIZE TO_CHAR(T.
------------------------------ --------------- ------------ ---------- ----------- ---------- ----------- ----------
SSN			       VARCHAR2
EMPLOYEE_ID		       NUMBER			107 .009345794		 1	    0	      107 03-02-2022
FIRST_NAME		       VARCHAR2 		 91 .010989011		 1	    0	      107 03-02-2022
LAST_NAME		       VARCHAR2 		102 .009803922		 1	    0	      107 03-02-2022
EMAIL			       VARCHAR2 		107 .009345794		 1	    0	      107 03-02-2022
PHONE_NUMBER		       VARCHAR2 		107 .009345794		 1	    0	      107 03-02-2022
HIRE_DATE		       DATE			 98 .010204082		 1	    0	      107 03-02-2022
JOB_ID			       VARCHAR2 		 19 .004672897		19	    0	      107 03-02-2022
SALARY			       NUMBER			 58 .017241379		 1	    0	      107 03-02-2022
COMMISSION_PCT		       NUMBER			  7 .142857143		 1	   72	       35 03-02-2022
MANAGER_ID		       NUMBER			 18 .004716981		18	    1	      106 03-02-2022
DEPARTMENT_ID		       NUMBER			 11 .004716981		11	    1	      106 03-02-2022

INDEX_NAME	     COLUMN_NAME		    COLUMN_POSITION COL
-------------------- ------------------------------ --------------- ------------------------------
EMP_DEPARTMENT_IX    DEPARTMENT_ID				  1 NUMBER(4,0)
EMP_EMAIL_UK	     EMAIL					  1 VARCHAR2(25) NOT NULL
EMP_EMP_ID_PK	     EMPLOYEE_ID				  1 NUMBER(6,0) NOT NULL
EMP_JOB_IX	     JOB_ID					  1 VARCHAR2(10) NOT NULL
EMP_MANAGER_IX	     MANAGER_ID 				  1 NUMBER(6,0)
EMP_NAME_IX	     LAST_NAME					  1 VARCHAR2(25) NOT NULL
EMP_NAME_IX	     FIRST_NAME 				  2 VARCHAR2(20)

7 rows selected.
INDEX_NAME		   BLEV LEAF_BLOCKS DISTINCT_KEYS   NUM_ROWS AVG_LEAF_BLOCKS_PER_KEY AVG_DATA_BLOCKS_PER_KEY CLUSTERING_FACTOR TO_CHAR(T.
-------------------- ---------- ----------- ------------- ---------- ----------------------- ----------------------- ----------------- ----------
EMP_NAME_IX		      0 	  1	      107	 107			   1			   1		    15 03-02-2022
EMP_MANAGER_IX		      0 	  1	       18	 106			   1			   1		     8 03-02-2022
EMP_JOB_IX		      0 	  1	       19	 107			   1			   1		     8 03-02-2022
EMP_DEPARTMENT_IX	      0 	  1	       11	 106			   1			   1		     9 03-02-2022
EMP_EMP_ID_PK		      0 	  1	      107	 107			   1			   1		     2 03-02-2022
EMP_EMAIL_UK		      0 	  1	      107	 107			   1			   1		    19 03-02-2022

```

### 修改普通列为加密列

```
ALTER TABLE EMPLOYEES MODIFY (first_name ENCRYPT);
```

EMPLOYEES 表的 SALARY 列为普通列没有索引，修改为加密列

```
ALTER TABLE EMPLOYEES MODIFY (SALARY ENCRYPT);

SQL> ALTER TABLE EMPLOYEES MODIFY (SALARY ENCRYPT);

Table altered.

SQL> desc EMPLOYEES
 Name					   Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID				   NOT NULL NUMBER(6)
 FIRST_NAME					    VARCHAR2(20)
 LAST_NAME				   NOT NULL VARCHAR2(25)
 EMAIL					   NOT NULL VARCHAR2(25)
 PHONE_NUMBER					    VARCHAR2(20)
 HIRE_DATE				   NOT NULL DATE
 JOB_ID 				   NOT NULL VARCHAR2(10)
 SALARY 					    NUMBER(8,2) ENCRYPT
 COMMISSION_PCT 				    NUMBER(2,2)
 MANAGER_ID					    NUMBER(6)
 DEPARTMENT_ID					    NUMBER(4)
 SSN						    VARCHAR2(11) ENCRYPT

SQL>
```

### 取消加密列

```
ALTER TABLE EMPLOYEES MODIFY (ssn DECRYPT);
```

```
SQL> ALTER TABLE EMPLOYEES MODIFY (ssn DECRYPT);

Table altered.

SQL> desc EMPLOYEES
 Name					   Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID				   NOT NULL NUMBER(6)
 FIRST_NAME					    VARCHAR2(20)
 LAST_NAME				   NOT NULL VARCHAR2(25)
 EMAIL					   NOT NULL VARCHAR2(25)
 PHONE_NUMBER					    VARCHAR2(20)
 HIRE_DATE				   NOT NULL DATE
 JOB_ID 				   NOT NULL VARCHAR2(10)
 SALARY 					    NUMBER(8,2) ENCRYPT
 COMMISSION_PCT 				    NUMBER(2,2)
 MANAGER_ID					    NUMBER(6)
 DEPARTMENT_ID					    NUMBER(4)
 SSN						    VARCHAR2(11)

SQL>
```

### 索引列加密

EMPLOYEES 表中 first_name 是索引列，下面测试加密对索引列进行加密。

```
ALTER TABLE EMPLOYEES MODIFY (EMAIL ENCRYPT);
```

> ALTER TABLE EMPLOYEES MODIFY (first_name ENCRYPT);
>
> 默认采用`AES192` 加密算法。默认会使用 salt 加密数据，如果要对索引列进行加密需要制定 `NO SALT`，有可以使用 `NOMAC`  参数。
>
> Salt is added to the data, by default. You can encrypt the column using a different algorithm. If you want to index a column, then you must specify `NO SALT`. You can also bypass integrity checks by using the `NOMAC` parameter.

```
SQL> ALTER TABLE EMPLOYEES MODIFY (first_name ENCRYPT);
ALTER TABLE EMPLOYEES MODIFY (first_name ENCRYPT)
                              *
ERROR at line 1:
ORA-28338: Column(s) cannot be both indexed and encrypted with salt

```

索引列必须采用 no salt 的方式进行加密。

```
SQL> ALTER TABLE EMPLOYEES MODIFY (first_name ENCRYPT  NO SALT);
Table altered.
```

加密列上创建索引测试

```
create BITMAP index id_idx on EMPLOYEES(ssn)
```

```


hr@(601_CBTDEPDB)> c/first_name/ssn
  1* ALTER TABLE EMPLOYEES MODIFY (ssn ENCRYPT)
hr@(601_CBTDEPDB)> /

Table altered.

hr@(601_CBTDEPDB)> desc EMPLOYEES
 Name											   Null?    Type
 ----------------------------------------------------------------------------------------- -------- ------------------------------------------------------------
 EMPLOYEE_ID										   NOT NULL NUMBER(6)
 FIRST_NAME											    VARCHAR2(20) ENCRYPT
 LAST_NAME										   NOT NULL VARCHAR2(25)
 EMAIL											   NOT NULL VARCHAR2(25)
 PHONE_NUMBER											    VARCHAR2(20)
 HIRE_DATE										   NOT NULL DATE
 JOB_ID 										   NOT NULL VARCHAR2(10)
 SALARY 											    NUMBER(8,2) ENCRYPT
 COMMISSION_PCT 										    NUMBER(2,2)
 MANAGER_ID											    NUMBER(6)
 DEPARTMENT_ID											    NUMBER(4)
 SSN												    VARCHAR2(11) ENCRYPT

hr@(601_CBTDEPDB)>  create BITMAP index id_idx on EMPLOYEES(ssn)
  2  ;
 create BITMAP index id_idx on EMPLOYEES(ssn)
                                         *
ERROR at line 1:
ORA-28337: the specified index may not be defined on an encrypted column

```

无法在加密列上创建位图索引。

解决方法是通过 表空间进行加密，相当表上取消加密列来实现。

```
hr@(601_CBTDEPDB)>   create table tde1 (c number) tablespace TDETBS;

Table created.

hr@(601_CBTDEPDB)> create bitmap index i_tde on tde1(c);

Index created.
```



###  加密与物化视图

具体操作

创建物化视图需要的权限：
grant create materialized view to user_name; 本次使用的hr 有dba权限

在源表建立物化视图日志：

```
hr@(14_CBTDEPDB)> create materialized view log on EMPLOYEES  tablespace TDETBS with primary key;

Materialized view log created.

hr@(14_CBTDEPDB)>
```

在目标数据库上创建MATERIALIZED VIEW：

```
hr@(14_CBTDEPDB)> create materialized view mv_EMPLOYEES_test refresh force on demand start with sysdate next
to_date(concat(to_char(sysdate+1,'dd-mm-yyyy'),'10:25:00'),'dd-mm-yyyy hh24:mi:ss') as
select * from EMPLOYEES;   2    3

Materialized view created.

hr@(14_CBTDEPDB)>
```

修改刷新时间：

```
hr@(14_CBTDEPDB)> alter materialized view mv_EMPLOYEES_test refresh force on demand start with sysdate
next to_date(concat(to_char(sysdate+1,'dd-mm-yyyy'),' 23:00:00'),'dd-mm-yyyy hh24:mi:ss');  2

Materialized view altered.

hr@(14_CBTDEPDB)> alter materialized view mv_EMPLOYEES_test refresh force on demand start with sysdate
next trunc(sysdate,'dd')+1+1/24;  2

Materialized view altered.
```

建立索引：

```
hr@(14_CBTDEPDB)> create index IDX_MMT_IU_TEST on mv_EMPLOYEES_test(SSN,JOB_ID)   tablespace TDETBS;

Index created.
```

删除物化视图及日志：

```
drop materialized view log on EMPLOYEES;    --删除物化视图日志： 
drop materialized view mv_EMPLOYEES_test; --删除物化视图
hr@(14_CBTDEPDB)>  drop materialized view log on EMPLOYEES;

Materialized view log dropped.

hr@(14_CBTDEPDB)> drop materialized view mv_EMPLOYEES_test;

Materialized view dropped.

hr@(14_CBTDEPDB)>
```

总结：物化视图跟没采用tde加密操作方式相同。

### 加密与DBLINK

由于目前 tde 配置之后，数据库的连接跟普通的连接方式相同，索引dblink的创建方式跟不创建tde的方式相同。

```sql
CREATE DATABASE LINK dl4tdepdb CONNECT TO system IDENTIFIED BY Password123 USING 'TDEPDB';
```

### 分区表测试

创建分区表

```
CREATE TABLE hr.tbspar_ORDER
(
  odb_ID             NUMBER(12)               NOT NULL,
  odb_EXTERNALID     VARCHAR2(40 BYTE)        NOT NULL,
  odb_USERID         NUMBER(11)               NOT NULL,
  odb_STATUS         NUMBER(1)                NOT NULL,
  odb_STARTtmc      DATE                     DEFAULT sysdate               NOT NULL,
  odb_ENDtmc        DATE                     DEFAULT sysdate               NOT NULL,
  odb_USER_PRICE     NUMBER(12,4)             NOT NULL,
  odb_RETURNtmc     DATE                     DEFAULT sysdate               NOT NULL
)
TABLESPACE TDETBS
PARTITION BY RANGE (odb_STARTtmc)
INTERVAL( NUMTODSINTERVAL(7,'DAY'))
(  
  PARTITION VALUES LESS THAN (TO_DATE(' 2022-03-20 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    LOGGING
    NOCOMPRESS
)
;

CREATE UNIQUE INDEX hr.tbspar_odb_ID ON hr.tbspar_ORDER
(odb_ID)
TABLESPACE TDETBS
;

CREATE UNIQUE INDEX hr.tbspar_odb_USERID_EXID ON hr.tbspar_ORDER
(odb_EXTERNALID, odb_USERID)
TABLESPACE TDETBS
;

ALTER TABLE hr.tbspar_ORDER ADD (
  CONSTRAINT tbspar_odb_ID
  PRIMARY KEY
  (odb_ID)
  USING INDEX hr.tbspar_odb_ID
  ENABLE VALIDATE,
  CONSTRAINT tbspar_odb_USERID_EXID
  UNIQUE (odb_EXTERNALID, odb_USERID)
  USING INDEX hr.tbspar_odb_USERID_EXID
  ENABLE VALIDATE);
```

分区表日常维护

