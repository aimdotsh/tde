tde exp/imp 测试

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

可以看到exp 无法导出 加密的表。