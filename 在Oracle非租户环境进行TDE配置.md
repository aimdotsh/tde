

## 在Oracle非租户环境进行TDE配置

版本：12.1.0.2 nocdb

```
SQL> show pdbs
SQL> select CDB from v$database;
CDB
---
NO
```



| TDE与非多租户 | 非多租户的TDE配置 | 在Oracle非租户环境进行TDE配置 | √    |
| ------------- | ----------------- | ----------------------------- | ---- |

### Step 1: Set the Software Keystore Location in the sqlnet.ora File

```
mkdir -p /etc/ORACLE/WALLETS/nocdb
chown -R oracle:oinstall /etc/ORACLE/WALLETS/nocdb
```

```
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/nocdb)))
```



### Step 2: Create the Software Keystore

创建用户

需要 `ADMINISTER KEY MANAGEMENT` or `SYSKM` privilege.

```
SQL> create user sec_admin identified by "Password123";
User created.
SQL> grant SYSKM to sec_admin;
Grant succeeded.
SQL> exit
```

```
sqlplus sec_admin as syskm
Enter password: password
Connected.
SQL> show user
USER is "SYSKM"
SQL>
```

> ADMINISTER KEY MANAGEMENT CREATE KEYSTORE 'keystore_location' IDENTIFIED BY software_keystore_password;

```sql
SQL> ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/etc/ORACLE/WALLETS/nocdb' IDENTIFIED BY Password23;

keystore altered.
```



### Step 3: Open the Software Keystore

> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY software_keystore_password [CONTAINER = ALL | CURRENT];

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
SQL>  ! ls -l /etc/ORACLE/WALLETS/nocdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 18 10:09 ewallet.p12

```

### Step 4: Set the Software TDE Master Encryption Key

> ADMINISTER KEY MANAGEMENT SET KEY [USING TAG 'tag'] IDENTIFIED BY keystore_password [WITH BACKUP [USING 'backup_identifier']] [CONTAINER = ALL | CURRENT];

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY Password23 with backup USING 'nocdb_tde_key_bak';

keystore altered.
```

> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys

```sql
SQL> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys;

KEY_ID									       TAG		    KEYSTORE_TYPE     USER				 CON_ID BACKED_UP
------------------------------------------------------------------------------ -------------------- ----------------- ------------------------------ ---------- ---------
AZE+jaWrrk8ivxzQvU6S8VUAAAAAAAAAAAAAAAAAAAAAAAAAAAAA						    SOFTWARE KEYSTORE SYSKM				      0 NO

```

### Step 5: Encrypt Your Data

#### Encrypting Columns in Tables

```
SQL> CREATE TABLE employee (
     first_name VARCHAR2(128),
     last_name VARCHAR2(128),
     empID NUMBER,
     salary NUMBER(6) ENCRYPT
);   2    3    4    5    6
     salary NUMBER(6) ENCRYPT
     *
ERROR at line 5:
ORA-28336: cannot encrypt SYS owned objects
```

无法加密sys 用户下的对象，创建普通用户

```
SQL> create user utde identified by "tdenocdb";

User created.

SQL> grant connect,resource to utde;

Grant succeeded.

[oracle@tcloud ~]$ sqlplus utde/tdenocdb
SQL> show user
USER is "UTDE"

SQL>   CREATE TABLE employee (
     first_name VARCHAR2(128),
     last_name VARCHAR2(128),
     empID NUMBER,
     salary NUMBER(6) ENCRYPT
);    2    3    4    5    6

Table created.
```

普通用户可以直接创建加密列空间，需要用create /aler table权限，如果要创建加密表空间，需要creae tablespace 权限。



```
SQL> CREATE TABLESPACE TEST_pdbtde datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);

Tablespace created.
```

