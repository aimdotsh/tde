[TOC]

   * [tde exp/imp 测试](#tde-expimp-测试)
      * [exp 导出数据测试](#exp-导出数据测试)
         * [钱包打开的情况下测试exp](#钱包打开的情况下测试exp)
         * [钱包关闭情况下exp导出](#钱包关闭情况下exp导出)
      * [imp 导入数据测试](#imp-导入数据测试)
      * [expdp 导出数据测试](#expdp-导出数据测试)
         * [钱包打开的情况expdp测试](#钱包打开的情况expdp测试)
         * [钱包关闭的情况测试](#钱包关闭的情况测试)
         * [impdp 在钱包关闭的情况下导入](#impdp-在钱包关闭的情况下导入)
         * [钱包打开的情况下导入](#钱包打开的情况下导入)
         * [总结](#总结)
   * [tde rman  测试](#tde-rman--测试)
      * [rman 不压缩备份测试](#rman-不压缩备份测试)
      * [总结](#总结-1)
      * [rman 压缩备份测试](#rman-压缩备份测试)
         * [钱包关闭 rman 压缩测试](#钱包关闭-rman-压缩测试)
         * [钱包打开 rman 压缩测试](#钱包打开-rman-压缩测试)
      * [总结](#总结-2)

## tde exp/imp 测试

### exp 导出数据测试

#### 钱包打开的情况下测试exp

```
$exp hr/Password123@cbtdepdb owner=hr log=/home/oracle/exphr.log file=/home/oracle/hrexp.dmp

Export: Release 12.1.0.2.0 - Production on Thu Mar 17 10:23:51 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.


Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
Export done in US7ASCII character set and AL16UTF16 NCHAR character set
server uses AL32UTF8 character set (possible charset conversion)

About to export specified users ...
. exporting pre-schema procedural objects and actions
. exporting foreign function library names for user HR
. exporting PUBLIC type synonyms
. exporting private type synonyms
. exporting object type definitions for user HR
About to export HR's objects ...
. exporting database links
. exporting sequence numbers
. exporting cluster definitions
. about to export HR's tables via Conventional Path ...
. . exporting table                      COUNTRIES         25 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                    DEPARTMENTS         27 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00107: Feature (COLUMN ENCRYPTION) of column FIRST_NAME in table HR.EMPLOYEES is not supported. The table will not be exported.
. . exporting table                           JOBS         19 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                    JOB_HISTORY         10 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                      LOCATIONS         23 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                        REGIONS          4 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                           TAB1     100000 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00113: Feature New Composite Partitioning Method is unsupported. Table HR.TBSPAR_ORDER could not be exported
EXP-00111: Table TDETAB resides in an Encrypted Tablespace TDETBS and will not be exported
EXP-00111: Table TDETAB2 resides in an Encrypted Tablespace TDETBS and will not be exported
EXP-00107: Feature (COLUMN ENCRYPTION) of column ID in table HR.TDETAB_COL is not supported. The table will not be exported.
. exporting synonyms
. exporting views
. exporting stored procedures
. exporting operators
. exporting referential integrity constraints
. exporting triggers
. exporting indextypes
. exporting bitmap, functional and extensible indexes
. exporting posttables actions
. exporting materialized views
. exporting snapshot logs
. exporting job queues
. exporting refresh groups and children
. exporting dimensions
. exporting post-schema procedural objects and actions
. exporting statistics
Export terminated successfully with warnings.

```



```
EXP-00107: Feature (COLUMN ENCRYPTION) of column FIRST_NAME in table HR.EMPLOYEES is not supported. The table will not be exported.
```

#### 钱包关闭情况下exp导出

```
  exp hr/Password123@cbtdepdb owner=hr  log=/home/oracle/exphr.log file=/home/oracle/hrexpclose.dmp

Export: Release 12.1.0.2.0 - Production on Thu Mar 17 11:28:13 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.


Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
Export done in US7ASCII character set and AL16UTF16 NCHAR character set
server uses AL32UTF8 character set (possible charset conversion)

About to export specified users ...
. exporting pre-schema procedural objects and actions
. exporting foreign function library names for user HR
. exporting PUBLIC type synonyms
. exporting private type synonyms
. exporting object type definitions for user HR
About to export HR's objects ...
. exporting database links
. exporting sequence numbers
. exporting cluster definitions
. about to export HR's tables via Conventional Path ...
. . exporting table                      COUNTRIES         25 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                    DEPARTMENTS         27 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00107: Feature (COLUMN ENCRYPTION) of column SSN in table HR.EMPLOYEES is not supported. The table will not be exported.
. . exporting table                           JOBS         19 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                    JOB_HISTORY         10 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                      LOCATIONS         23 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                        REGIONS          4 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. . exporting table                           TAB1     100000 rows exported
EXP-00091: Exporting questionable statistics.
EXP-00113: Feature New Composite Partitioning Method is unsupported. Table HR.TBSPAR_ORDER could not be exported
EXP-00111: Table TDE1 resides in an Encrypted Tablespace TDETBS and will not be exported
EXP-00111: Table TDETAB resides in an Encrypted Tablespace TDETBS and will not be exported
EXP-00111: Table TDETAB2 resides in an Encrypted Tablespace TDETBS and will not be exported
EXP-00107: Feature (COLUMN ENCRYPTION) of column ID in table HR.TDETAB_COL is not supported. The table will not be exported.
. exporting synonyms
. exporting views
. exporting stored procedures
. exporting operators
. exporting referential integrity constraints
. exporting triggers
. exporting indextypes
. exporting bitmap, functional and extensible indexes
EXP-00091: Exporting questionable statistics.
EXP-00091: Exporting questionable statistics.
. exporting posttables actions
. exporting materialized views
. exporting snapshot logs
. exporting job queues
. exporting refresh groups and children
. exporting dimensions
. exporting post-schema procedural objects and actions
. exporting statistics
```

结论

无论钱包是否开启，可以看到exp 都无法导出 加密的表。

### imp 导入数据测试

```
[oracle@tcloud_for_12c_tdecdb:/home/oracle]$ imp  hr/Password123@cbtdepdb fromuser=hr touser=hr5 file=/home/oracle/hrexp.dmp

Import: Release 12.1.0.2.0 - Production on Thu Mar 17 11:12:18 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.


Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

Export file created by EXPORT:V12.01.00 via conventional path
import done in US7ASCII character set and AL16UTF16 NCHAR character set
import server uses AL32UTF8 character set (possible charset conversion)
. importing HR's objects into HR5
. . importing table                    "COUNTRIES"         25 rows imported
. . importing table                  "DEPARTMENTS"         27 rows imported
. . importing table                         "JOBS"         19 rows imported
. . importing table                  "JOB_HISTORY"         10 rows imported
. . importing table                    "LOCATIONS"         23 rows imported
. . importing table                      "REGIONS"          4 rows imported
. . importing table                         "TAB1"     100000 rows imported

```

imp 可以正常imp导出的数据正常导入。

### expdp 导出数据测试

#### 钱包打开的情况expdp测试

```
expdp hr/Password123@cbtdepdb schemas=hr dumpfile=hrexpdp.dmp logfile=hr.log directory=LOG_FILE_DIR

Export: Release 12.1.0.2.0 - Production on Thu Mar 17 11:14:21 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
Starting "HR"."SYS_EXPORT_SCHEMA_01":  hr/********@cbtdepdb schemas=hr dumpfile=hrexpdp.dmp logfile=hr.log directory=LOG_FILE_DIR
Estimate in progress using BLOCKS method...
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 56.56 MB
Processing object type SCHEMA_EXPORT/USER
Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
Processing object type SCHEMA_EXPORT/ROLE_GRANT
Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
Processing object type SCHEMA_EXPORT/VIEW/VIEW
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/BITMAP_INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/BITMAP_INDEX/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/STATISTICS/MARKER
. . exported "HR"."TDETAB_COL"                           9.796 MB  100000 rows
. . exported "HR"."TAB1"                                 9.796 MB  100000 rows
. . exported "HR"."TDETAB"                               9.796 MB  100000 rows
. . exported "HR"."TDETAB2"                              9.796 MB  100000 rows
. . exported "HR"."COUNTRIES"                            6.460 KB      25 rows
. . exported "HR"."DEPARTMENTS"                          7.125 KB      27 rows
. . exported "HR"."EMPLOYEES"                            19.20 KB     106 rows
. . exported "HR"."JOBS"                                 7.109 KB      19 rows
. . exported "HR"."JOB_HISTORY"                          7.195 KB      10 rows
. . exported "HR"."LOCATIONS"                            8.437 KB      23 rows
. . exported "HR"."REGIONS"                              5.546 KB       4 rows
. . exported "HR"."TBSPAR_ORDER":"SYS_P601"                  0 KB       0 rows
. . exported "HR"."TDE1"                                     0 KB       0 rows
ORA-39173: Encrypted data has been stored unencrypted in dump file set.
Master table "HR"."SYS_EXPORT_SCHEMA_01" successfully loaded/unloaded
******************************************************************************
Dump file set for HR.SYS_EXPORT_SCHEMA_01 is:
  /tmp/log/hrexpdp.dmp
Job "HR"."SYS_EXPORT_SCHEMA_01" successfully completed at Thu Mar 17 11:15:01 2022 elapsed 0 00:00:38
```

expdp 在钱包打开的情况可以正常完整的导出数据。

#### 钱包关闭的情况测试

```
[oracle@tcloud_for_12c_tdecdb:/home/oracle]$  expdp hr/Password123@cbtdepdb schemas=hr dumpfile=hrexpdp2.dmp logfile=hr.log directory=LOG_FILE_DIR

Export: Release 12.1.0.2.0 - Production on Thu Mar 17 11:17:42 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
Starting "HR"."SYS_EXPORT_SCHEMA_01":  hr/********@cbtdepdb schemas=hr dumpfile=hrexpdp2.dmp logfile=hr.log directory=LOG_FILE_DIR
Estimate in progress using BLOCKS method...
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 56.56 MB
Processing object type SCHEMA_EXPORT/USER
Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
Processing object type SCHEMA_EXPORT/ROLE_GRANT
Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
Processing object type SCHEMA_EXPORT/VIEW/VIEW
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/BITMAP_INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/BITMAP_INDEX/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/STATISTICS/MARKER
ORA-31693: Table data object "HR"."TDETAB_COL" failed to load/unload and is being skipped due to error:
ORA-28365: wallet is not open
. . exported "HR"."TAB1"                                 9.796 MB  100000 rows
ORA-31693: Table data object "HR"."TDETAB" failed to load/unload and is being skipped due to error:
ORA-02354: error in exporting/importing data
ORA-28365: wallet is not open
ORA-31693: Table data object "HR"."TDETAB2" failed to load/unload and is being skipped due to error:
ORA-02354: error in exporting/importing data
ORA-28365: wallet is not open
. . exported "HR"."COUNTRIES"                            6.460 KB      25 rows
. . exported "HR"."DEPARTMENTS"                          7.125 KB      27 rows
ORA-31693: Table data object "HR"."EMPLOYEES" failed to load/unload and is being skipped due to error:
ORA-28365: wallet is not open
. . exported "HR"."JOBS"                                 7.109 KB      19 rows
. . exported "HR"."JOB_HISTORY"                          7.195 KB      10 rows
. . exported "HR"."LOCATIONS"                            8.437 KB      23 rows
. . exported "HR"."REGIONS"                              5.546 KB       4 rows
. . exported "HR"."TBSPAR_ORDER":"SYS_P601"              8.210 KB       0 rows
. . exported "HR"."TDE1"                                     0 KB       0 rows
ORA-39173: Encrypted data has been stored unencrypted in dump file set.
Master table "HR"."SYS_EXPORT_SCHEMA_01" successfully loaded/unloaded
******************************************************************************
Dump file set for HR.SYS_EXPORT_SCHEMA_01 is:
  /tmp/log/hrexpdp2.dmp
Job "HR"."SYS_EXPORT_SCHEMA_01" completed with 4 error(s) at Thu Mar 17 11:18:17 2022 elapsed 0 00:00:34

[oracle@tcloud_for_12c_tdecdb:/home/oracle]$
```

钱包关闭的情况，加密数据无法正常导出。

#### impdp 在钱包关闭的情况下导入

```
[oracle@tcloud_for_12c_tdecdb:/home/oracle]$  impdp  hr/Password123@cbtdepdb remap_schema=hr:hr6  dumpfile=hrexpdp2.dmp logfile=hr.log directory=LOG_FILE_DIR

Import: Release 12.1.0.2.0 - Production on Thu Mar 17 11:20:42 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
Master table "HR"."SYS_IMPORT_FULL_01" successfully loaded/unloaded
Starting "HR"."SYS_IMPORT_FULL_01":  hr/********@cbtdepdb remap_schema=hr:hr6 dumpfile=hrexpdp2.dmp logfile=hr.log directory=LOG_FILE_DIR
Processing object type SCHEMA_EXPORT/USER
Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
Processing object type SCHEMA_EXPORT/ROLE_GRANT
Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
Processing object type SCHEMA_EXPORT/TABLE/TABLE
ORA-39083: Object type TABLE:"HR6"."TBSPAR_ORDER" failed to create with error:
ORA-28365: wallet is not open
Failing sql is:
CREATE TABLE "HR6"."TBSPAR_ORDER" ("ODB_ID" NUMBER(12,0) NOT NULL ENABLE, "ODB_EXTERNALID" VARCHAR2(40 BYTE) NOT NULL ENABLE, "ODB_USERID" NUMBER(11,0) NOT NULL ENABLE, "ODB_STATUS" NUMBER(1,0) NOT NULL ENABLE, "ODB_STARTTMC" DATE DEFAULT sysdate NOT NULL ENABLE, "ODB_ENDTMC" DATE DEFAULT sysdate NOT NULL ENABLE, "ODB_USER_PRICE" NUMBER(12,4) NOT NULL ENABLE, "ODB_RETURNTMC" DATE DEFA
ORA-39083: Object type TABLE:"HR6"."TDETAB2" failed to create with error:
ORA-28365: wallet is not open
Failing sql is:
CREATE TABLE "HR6"."TDETAB2" ("ID" NUMBER, "C" VARCHAR2(100 BYTE), "E" VARCHAR2(50 BYTE)) SEGMENT CREATION IMMEDIATE PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 NOCOMPRESS LOGGING STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645 PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1 BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT) TABLESPACE "TDETBS"
ORA-39083: Object type TABLE:"HR6"."TDETAB" failed to create with error:
ORA-28365: wallet is not open
Failing sql is:
CREATE TABLE "HR6"."TDETAB" ("ID" NUMBER, "C" VARCHAR2(100 BYTE), "E" VARCHAR2(50 BYTE)) SEGMENT CREATION IMMEDIATE PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 NOCOMPRESS LOGGING STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645 PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1 BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT) TABLESPACE "TDETBS"
ORA-39083: Object type TABLE:"HR6"."TDE1" failed to create with error:
ORA-28365: wallet is not open
Failing sql is:
CREATE TABLE "HR6"."TDE1" ("C" NUMBER) SEGMENT CREATION IMMEDIATE PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 NOCOMPRESS LOGGING STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645 PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1 BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT) TABLESPACE "TDETBS"
ORA-39083: Object type TABLE:"HR6"."TDETAB_COL" failed to create with error:
ORA-28365: wallet is not open
Failing sql is:
CREATE TABLE "HR6"."TDETAB_COL" ("ID" NUMBER ENCRYPT USING 'AES256' 'SHA-1' NO SALT , "C" VARCHAR2(100 BYTE) ENCRYPT USING 'AES256' 'SHA-1' NO SALT , "E" VARCHAR2(50 BYTE) ENCRYPT USING 'AES256' 'SHA-1' NO SALT ) SEGMENT CREATION IMMEDIATE PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 NOCOMPRESS LOGGING STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645 PCTINCREASE 0
ORA-39083: Object type TABLE:"HR6"."EMPLOYEES" failed to create with error:
ORA-28365: wallet is not open
Failing sql is:
CREATE TABLE "HR6"."EMPLOYEES" ("EMPLOYEE_ID" NUMBER(6,0), "FIRST_NAME" VARCHAR2(20 BYTE) ENCRYPT USING 'AES192' 'SHA-1' NO SALT , "LAST_NAME" VARCHAR2(25 BYTE) CONSTRAINT "EMP_LAST_NAME_NN" NOT NULL ENABLE, "EMAIL" VARCHAR2(25 BYTE) CONSTRAINT "EMP_EMAIL_NN" NOT NULL ENABLE, "PHONE_NUMBER" VARCHAR2(20 BYTE), "HIRE_DATE" DATE CONSTRAINT "EMP_HIRE_DATE_NN" NOT NULL ENABLE, "JOB_ID" VARCHA
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "HR6"."TAB1"                                9.796 MB  100000 rows
. . imported "HR6"."COUNTRIES"                           6.460 KB      25 rows
. . imported "HR6"."DEPARTMENTS"                         7.125 KB      27 rows
. . imported "HR6"."JOBS"                                7.109 KB      19 rows
. . imported "HR6"."JOB_HISTORY"                         7.195 KB      10 rows
```

钱包关闭的情况加密数据及加密表空间里的表及数据无法正常导出。

#### 钱包打开的情况下导入

```
 impdp hr/Password123@cbtdepdb remap_schema=hr:hr7  dumpfile=hrexpdp2.dmp logfile=hr.log directory=LOG_FILE_DIR

Import: Release 12.1.0.2.0 - Production on Thu Mar 17 11:23:15 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
Master table "HR"."SYS_IMPORT_FULL_01" successfully loaded/unloaded
Starting "HR"."SYS_IMPORT_FULL_01":  hr/********@cbtdepdb remap_schema=hr:hr7 dumpfile=hrexpdp2.dmp logfile=hr.log directory=LOG_FILE_DIR
Processing object type SCHEMA_EXPORT/USER
Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
Processing object type SCHEMA_EXPORT/ROLE_GRANT
Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "HR7"."TAB1"                                9.796 MB  100000 rows
. . imported "HR7"."COUNTRIES"                           6.460 KB      25 rows
. . imported "HR7"."DEPARTMENTS"                         7.125 KB      27 rows
. . imported "HR7"."JOBS"                                7.109 KB      19 rows
. . imported "HR7"."JOB_HISTORY"                         7.195 KB      10 rows
. . imported "HR7"."LOCATIONS"                           8.437 KB      23 rows
. . imported "HR7"."REGIONS"                             5.546 KB       4 rows
. . imported "HR7"."TBSPAR_ORDER":"SYS_P601"             8.210 KB       0 rows
. . imported "HR7"."TDE1"                                    0 KB       0 rows
Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
Processing object type SCHEMA_EXPORT/VIEW/VIEW
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT

```

可以看到 钱包打开的情况，数据可以正常导入。



#### 总结

在钱包关闭的情况下，exp和expdp 都无法导出加密数据，其他非加密表可以导出。

在钱包打开的情况下，exp 不支持导出 tde 加密数据，包括加密表空间下的所有数据，及加密列的表都无法导出。imp 可以正常到exp 导出的数据导入到数据库。

在钱包打开的情况下，expdp 支持导出 tde 加密数据，包括加密表空间下的所有数据，及加密列的表都可以正常导出。impdp 也可以把expdp 导出的数据完整的导入。



## tde rman  测试

### rman 不压缩备份测试

```
RMAN> BACKUP DATABASE FORMAT '/oradata/tdecdb/backup/0318/bak_%U';

Starting backup at 2022-03-18 09:03:38
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=206 device type=DISK
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00003 name=/oradata/tdecdb/sys/sysaux01.dbf
input datafile file number=00001 name=/oradata/tdecdb/sys/system01.dbf
input datafile file number=00004 name=/oradata/tdecdb/undo/undotbs01.dbf
input datafile file number=00006 name=/oradata/tdecdb/data/users01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:03:39
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:04:04
piece handle=/oradata/tdecdb/backup/0318/bak_1c0omgfb_1_1 tag=TAG20220318T090339 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00060 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_sysaux_k15x87vb_.dbf
input datafile file number=00059 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_system_k15x87v3_.dbf
input datafile file number=00063 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_tdetbs_k34t95d6_.dbf
input datafile file number=00062 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_pttbs_k34t7zm1_.dbf
input datafile file number=00061 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_test_tde_k18b3y43_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:04:04
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:04:11
piece handle=/oradata/tdecdb/backup/0318/bak_1d0omgg4_1_1 tag=TAG20220318T090339 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:07
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00011 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_sysaux_k15zvlmn_.dbf
input datafile file number=00010 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_system_k15zvlmh_.dbf
input datafile file number=00015 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpvbp5_.dbf
input datafile file number=00016 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpxvfo_.dbf
input datafile file number=00017 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpyohb_.dbf
input datafile file number=00014 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigfie_t_k1vpcznp_.dbf
input datafile file number=00013 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_put_tbs_k1vp39l0_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:04:11
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:04:18
piece handle=/oradata/tdecdb/backup/0318/bak_1e0omggb_1_1 tag=TAG20220318T090339 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:07
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00055 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_sysaux_k32gtl45_.dbf
input datafile file number=00054 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_system_k32gtl3p_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:04:18
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:04:25
piece handle=/oradata/tdecdb/backup/0318/bak_1f0omggi_1_1 tag=TAG20220318T090339 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:07
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00007 name=/oradata/tdecdb/sys/pdbseed/sysaux01.dbf
input datafile file number=00005 name=/oradata/tdecdb/sys/pdbseed/system01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:04:25
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:04:32
piece handle=/oradata/tdecdb/backup/0318/bak_1g0omggp_1_1 tag=TAG20220318T090339 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:07
Finished backup at 2022-03-18 09:04:32

Starting Control File and SPFILE Autobackup at 2022-03-18 09:04:32
piece handle=/u01/app/oracle/fast_recovery_area/TDECDB/autobackup/2022_03_18/o1_mf_s_1099645472_k37po13v_.bkp comment=NONE
Finished Control File and SPFILE Autobackup at 2022-03-18 09:04:33

RMAN> exit


Recovery Manager complete.
[oracle@tcloud_for_12c_tdecdb:/oradata/tdecdb/backup/0318]$ ll
```

rman 不压缩恢复测试





### 总结

不压缩备份的时候，钱包关闭打开的情况都可以备份。

### rman 压缩备份测试

#### 钱包关闭 rman 压缩测试

```
RMAN>  backup as compressed backupset full database format '/oradata/tdecdb/backup/0318/full_bk1_%u%p%s.rmn';

Starting backup at 2022-03-18 09:06:06
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=12 device type=DISK
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00003 name=/oradata/tdecdb/sys/sysaux01.dbf
input datafile file number=00001 name=/oradata/tdecdb/sys/system01.dbf
input datafile file number=00004 name=/oradata/tdecdb/undo/undotbs01.dbf
input datafile file number=00006 name=/oradata/tdecdb/data/users01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:06:06
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:07:01
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1i0omgju150.rmn tag=TAG20220318T090606 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:55
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00060 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_sysaux_k15x87vb_.dbf
input datafile file number=00059 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_system_k15x87v3_.dbf
input datafile file number=00063 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_tdetbs_k34t95d6_.dbf
input datafile file number=00062 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_pttbs_k34t7zm1_.dbf
input datafile file number=00061 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_test_tde_k18b3y43_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:07:01
RMAN-03009: failure of backup command on ORA_DISK_1 channel at 03/18/2022 09:07:02
ORA-19914: unable to encrypt backup
ORA-28365: wallet is not open
continuing other job steps, job failed will not be re-run
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00011 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_sysaux_k15zvlmn_.dbf
input datafile file number=00010 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_system_k15zvlmh_.dbf
input datafile file number=00015 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpvbp5_.dbf
input datafile file number=00016 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpxvfo_.dbf
input datafile file number=00017 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpyohb_.dbf
input datafile file number=00014 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigfie_t_k1vpcznp_.dbf
input datafile file number=00013 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_put_tbs_k1vp39l0_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:07:03
RMAN-03009: failure of backup command on ORA_DISK_1 channel at 03/18/2022 09:07:04
ORA-19914: unable to encrypt backup
ORA-28365: wallet is not open
continuing other job steps, job failed will not be re-run
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00055 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_sysaux_k32gtl45_.dbf
input datafile file number=00054 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_system_k32gtl3p_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:07:04
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:07:29
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1l0omglo153.rmn tag=TAG20220318T090606 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00007 name=/oradata/tdecdb/sys/pdbseed/sysaux01.dbf
input datafile file number=00005 name=/oradata/tdecdb/sys/pdbseed/system01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:07:29
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:07:54
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1m0omgmh154.rmn tag=TAG20220318T090606 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
RMAN-00571: ===========================================================
RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
RMAN-00571: ===========================================================

RMAN-03009: failure of backup command on ORA_DISK_1 channel at 03/18/2022 09:07:02
ORA-19914: unable to encrypt backup
ORA-28365: wallet is not open
RMAN-03009: failure of backup command on ORA_DISK_1 channel at 03/18/2022 09:07:04
ORA-19914: unable to encrypt backup
ORA-28365: wallet is not open

RMAN> exit


Recovery Manager complete.
[oracle@tcloud_for_12c_tdecdb:/oradata/tdecdb/backup/0318]$ s

SQL*Plus: Release 12.1.0.2.0 Production on Fri Mar 18 09:08:43 2022

Copyright (c) 1982, 2014, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options


Session altered.

sys@(594_CDB$ROOT)>  ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.

sys@(594_CDB$ROOT)>
sys@(594_CDB$ROOT)>
sys@(594_CDB$ROOT)>
sys@(594_CDB$ROOT)> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
[oracle@tcloud_for_12c_tdecdb:/oradata/tdecdb/backup/0318]$ rman target /

Recovery Manager: Release 12.1.0.2.0 - Production on Fri Mar 18 09:09:32 2022

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

connected to target database: TDECDB (DBID=1299067025)

RMAN>  backup as compressed backupset full database format '/oradata/tdecdb/backup/0318/full_bk1_%u%p%s.rmn2b';

Starting backup at 2022-03-18 09:09:47
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=405 device type=DISK
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00003 name=/oradata/tdecdb/sys/sysaux01.dbf
input datafile file number=00001 name=/oradata/tdecdb/sys/system01.dbf
input datafile file number=00004 name=/oradata/tdecdb/undo/undotbs01.dbf
input datafile file number=00006 name=/oradata/tdecdb/data/users01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:09:47
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:10:42
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1n0omgqr155.rmn2b tag=TAG20220318T090947 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:55
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00060 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_sysaux_k15x87vb_.dbf
input datafile file number=00059 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_system_k15x87v3_.dbf
input datafile file number=00063 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_tdetbs_k34t95d6_.dbf
input datafile file number=00062 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_pttbs_k34t7zm1_.dbf
input datafile file number=00061 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_test_tde_k18b3y43_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:10:43
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:11:08
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1o0omgsi156.rmn2b tag=TAG20220318T090947 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00011 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_sysaux_k15zvlmn_.dbf
input datafile file number=00010 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_system_k15zvlmh_.dbf
input datafile file number=00015 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpvbp5_.dbf
input datafile file number=00016 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpxvfo_.dbf
input datafile file number=00017 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpyohb_.dbf
input datafile file number=00014 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigfie_t_k1vpcznp_.dbf
input datafile file number=00013 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_put_tbs_k1vp39l0_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:11:08
RMAN-03009: failure of backup command on ORA_DISK_1 channel at 03/18/2022 09:11:09
ORA-19914: unable to encrypt backup
ORA-28365: wallet is not open
continuing other job steps, job failed will not be re-run
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00055 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_sysaux_k32gtl45_.dbf
input datafile file number=00054 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_system_k32gtl3p_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:11:09
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:11:34
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1q0omgtd158.rmn2b tag=TAG20220318T090947 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00007 name=/oradata/tdecdb/sys/pdbseed/sysaux01.dbf
input datafile file number=00005 name=/oradata/tdecdb/sys/pdbseed/system01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:11:34
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:11:59
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1r0omgu6159.rmn2b tag=TAG20220318T090947 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
RMAN-00571: ===========================================================
RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
RMAN-00571: ===========================================================

RMAN-03009: failure of backup command on ORA_DISK_1 channel at 03/18/2022 09:11:09
ORA-19914: unable to encrypt backup
ORA-28365: wallet is not open
```

#### 钱包打开 rman 压缩测试

```
RMAN> backup as compressed backupset full database format '/oradata/tdecdb/backup/0318/full_bk1_%u%p%s.open';

Starting backup at 2022-03-18 09:26:55
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=11 device type=DISK
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00003 name=/oradata/tdecdb/sys/sysaux01.dbf
input datafile file number=00001 name=/oradata/tdecdb/sys/system01.dbf
input datafile file number=00004 name=/oradata/tdecdb/undo/undotbs01.dbf
input datafile file number=00006 name=/oradata/tdecdb/data/users01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:26:56
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:27:51
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1s0omhr0160.open tag=TAG20220318T092656 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:55
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00060 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_sysaux_k15x87vb_.dbf
input datafile file number=00059 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_system_k15x87v3_.dbf
input datafile file number=00063 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_tdetbs_k34t95d6_.dbf
input datafile file number=00062 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_pttbs_k34t7zm1_.dbf
input datafile file number=00061 name=/oradata/tdecdb/data/TDECDB/D87EC3294E251264E0530418000A849E/datafile/o1_mf_test_tde_k18b3y43_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:27:51
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:28:16
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1t0omhsn161.open tag=TAG20220318T092656 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00011 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_sysaux_k15zvlmn_.dbf
input datafile file number=00010 name=/oradata/tdecdb/data/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_system_k15zvlmh_.dbf
input datafile file number=00015 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpvbp5_.dbf
input datafile file number=00016 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpxvfo_.dbf
input datafile file number=00017 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigf_tbs_k1vpyohb_.dbf
input datafile file number=00014 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_bigfie_t_k1vpcznp_.dbf
input datafile file number=00013 name=/tmp/TDECDB/D87F6220B46B36BCE0530418000A94A3/datafile/o1_mf_put_tbs_k1vp39l0_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:28:16
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:28:41
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1u0omhtg162.open tag=TAG20220318T092656 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00055 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_sysaux_k32gtl45_.dbf
input datafile file number=00054 name=/oradata/tdecdb/data/TDECDB/DA4CB73192371EFAE0530418000AE013/datafile/o1_mf_system_k32gtl3p_.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:28:41
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:29:06
piece handle=/oradata/tdecdb/backup/0318/full_bk1_1v0omhu9163.open tag=TAG20220318T092656 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
channel ORA_DISK_1: starting compressed full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00007 name=/oradata/tdecdb/sys/pdbseed/sysaux01.dbf
input datafile file number=00005 name=/oradata/tdecdb/sys/pdbseed/system01.dbf
channel ORA_DISK_1: starting piece 1 at 2022-03-18 09:29:06
channel ORA_DISK_1: finished piece 1 at 2022-03-18 09:29:31
piece handle=/oradata/tdecdb/backup/0318/full_bk1_200omhv2164.open tag=TAG20220318T092656 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:25
Finished backup at 2022-03-18 09:29:31

Starting Control File and SPFILE Autobackup at 2022-03-18 09:29:31
piece handle=/u01/app/oracle/fast_recovery_area/TDECDB/autobackup/2022_03_18/o1_mf_s_1099646971_k37r3vtd_.bkp comment=NONE
Finished Control File and SPFILE Autobackup at 2022-03-18 09:29:32
```

### 总结

压缩备份的时候，需要钱包都打开，cdb和pdb都需要打开，否则为打开钱包的db无法进行备份。也就是压缩备份的时候，rman 会对tde加密数据进行解密，然后再进行压缩。

