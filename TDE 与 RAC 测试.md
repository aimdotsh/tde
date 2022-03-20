[TOC]



### TDE 与 RAC 测试

> **How to configure TDE in pluggable database in 12c for standalone and RAC environment (Doc ID 2107821.1)**

 This document details step by step instructions to configure TDE in 12c pluggable database for standalone and RAC environment.



### 1. Add the below entry in sqlnet.ora file

```
ENCRYPTION_WALLET_LOCATION =
      (SOURCE = (METHOD = FILE)
          (METHOD_DATA =
               (DIRECTORY = /cdbrdbms/etc/$ORACLE_SID)
          )
      )
```

### 2. Create Key store on CDB database and generate encryption key for CDB

```
ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/cdbrdbms/etc/MTc12c1' IDENTIFIED BY "welcome1";
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY "welcome1";
ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY "welcome1" WITH BACKUP;
```

### 3. Verify the wallet has been opened in CDB database

```
select * from v$encryption_wallet;

WRL_TYPE
--------------------
WRL_PARAMETER
--------------------------------------------------------------------------------
STATUS WALLET_TYPE WALLET_OR FULLY_BAC
------------------------------ -------------------- --------- ---------
CON_ID
----------
FILE
/cdbrdbms/etc/MTc12c1/
OPEN PASSWORD SINGLE NO
0
```

### 4. Open the respective PDB and set the PDB as current database 

```
SQL> show pdbs

CON_ID CON_NAME OPEN MODE RESTRICTED
---------- ------------------------------ ---------- ----------
2 PDB$SEED READ ONLY NO
3 MTC12P1 MOUNTED
4 MTC12P2 MOUNTED

SQL> alter pluggable database MTC12P2 open;

Pluggable database altered.

SQL> show pdbs

CON_ID CON_NAME OPEN MODE RESTRICTED
---------- ------------------------------ ---------- ----------
2 PDB$SEED READ ONLY NO
3 MTC12P1 MOUNTED
4 MTC12P2 READ WRITE NO
```

```
SQL> alter session set container=MTC12P2;

Session altered.

SQL> show con_name

CON_NAME
------------------------------
MTC12P2
```

### 5. Open the keystore in that PDB and generate encryption key for the PDB

```
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY "welcome1";

keystore altered

SQL> show pdbs

CON_ID CON_NAME OPEN MODE RESTRICTED
---------- ------------------------------ ---------- ----------
4 MTC12P2 READ WRITE NO
SQL> select * from v$encryption_wallet;

WRL_TYPE
--------------------
WRL_PARAMETER
--------------------------------------------------------------------------------
STATUS WALLET_TYPE WALLET_OR FULLY_BAC
------------------------------ -------------------- --------- ---------
CON_ID
----------
FILE
/cdbrdbms/etc/MTc12c1/
OPEN_NO_MASTER_KEY PASSWORD SINGLE UNDEFINED
0
```

 

```
SQL> ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY "welcome1" with backup;

keystore altered.

SQL> select * from v$encryption_wallet;

WRL_TYPE
--------------------
WRL_PARAMETER
--------------------------------------------------------------------------------
STATUS WALLET_TYPE WALLET_OR FULLY_BAC
------------------------------ -------------------- --------- ---------
CON_ID
----------
FILE
/cdbrdbms/etc/MTc12c1/
OPEN PASSWORD SINGLE NO
0
```

### 6. Create encrypted tablespace

```
SQL> show pdbs

CON_ID CON_NAME OPEN MODE RESTRICTED
---------- ------------------------------ ---------- ----------
4 MTC12P2 READ WRITE NO


SQL> create tablespace enc128_ts
datafile '/cdbrdbms/64bit/app/oracle/oradata/MTc12c1/MTc12p2/Test_encrption.dbf'
size 1M autoextend on next 1M
encryption using 'AES128'
default storage (encrypt)
/ 2 3 4 5 6

Tablespace created.
```



###7.  For RAC environment

#### 7.1 Make sure encryption_wallet_location parameter is configured in sqlnet.ora file of all other RAC nodes

```
ENCRYPTION_WALLET_LOCATION =
         (SOURCE = (METHOD = FILE)
              (METHOD_DATA =
                       (DIRECTORY = /cdbrdbms/etc/$ORACLE_SID)
                )
          )
```

#### 7.2 Copy the wallet file ewallet.p12 from first RAC node to all other RAC nodes ENCRYPTION_WALLET_LOCATION directory.

Important Note: It is recommended to use shared file location like ACFS or ASM location to hold the TDE wallet file in case of RAC DB.

 
