

## ORACLE 在CDB 的 TDE 配置

> 3 Configuring Transparent Data Encryption
>
> https://docs.oracle.com/database/121/ASOAG/configuring-transparent-data-encryption.htm#ASOAG10474
>
> 透明数据加密
>
> https://www.oracle.com/technetwork/topics/security/index-092808-zhs.html
>
> Database Advanced Security Guidxs
>
> https://docs.oracle.com/database/121/ASOAG/toc.htm



### Step 1: Set the Software Keystore Location in the sqlnet.ora File

```
mkdir -p /etc/ORACLE/WALLETS/tdecdb
chown -R oracle:oinstall /etc/ORACLE/WALLETS/tdecdb
```

<!--建议将钱包目录放到非 ORACLE 目录下面-->

使用root 创建单独的目录，权限授予给 oracle 用户。



```sql
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```



### Step 2: Create the Software Keystore

创建用户

需要 `ADMINISTER KEY MANAGEMENT` or `SYSKM` privilege.

```
SQL> create user c##sec_admin identified by "Password123";
User created.
SQL> grant SYSKM to c##sec_admin;
Grant succeeded.
SQL> exit
```

```
sqlplus c##sec_admin as syskm
Enter password: password
Connected.
```

> ADMINISTER KEY MANAGEMENT CREATE KEYSTORE 'keystore_location' IDENTIFIED BY software_keystore_password;

```sql
SQL> ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY Password23;
keystore altered.
```

是在 CDB$ROOT 上执行，不能在pdb级别执行。

```
SQL> show user
USER is "SYSKM"
SQL>  ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY Password23;
 ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY Password23
*
ERROR at line 1:
ORA-65040: operation not allowed from within a pluggable database


SQL> show con_name

CON_NAME
------------------------------
TDEPDB
SQL>

SQL> show con_name

CON_NAME
------------------------------
CDB$ROOT
SQL>
SQL> ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY Password23;

keystore altered.

运行此语句后，作为keystore的 ewallet.p12文件将出现在密钥库/etc/ORACLE/WALLETS/tdecdb 即 sqlnet.ora 配置的位置

SQL>  ! ls -l /etc/ORACLE/WALLETS/tdecdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 21 10:42 ewallet.p12

SQL>
```



```
create pluggable database tdepdb admin user pdbadmin identified by "Password123";
alter pluggable database tdepdb open  instances=all;
alter pluggable database tdepdb save state instances=all;
```



### Step 3: Open the Software Keystore

> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY software_keystore_password [CONTAINER = ALL | CURRENT];

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
keystore altered.
SQL> ! ls -l /etc/ORACLE/WALLETS/tdecdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 17 09:32 ewallet.p12
```

### Step 4: Set the Software TDE Master Encryption Key

> ADMINISTER KEY MANAGEMENT SET KEY [USING TAG 'tag'] IDENTIFIED BY keystore_password [WITH BACKUP [USING 'backup_identifier']] [CONTAINER = ALL | CURRENT];

```sql
ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY Password23 with backup USING 'tdetest01_key_bak';
```

> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys

```
SQL>  select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys;

KEY_ID							     TAG	KEYSTORE_TYPE	  USER				     CON_ID BACKED_UP
------------------------------------------------------------ ---------- ----------------- ------------------------------ ---------- ---------
AcBapz/5vk9Av29fJl/NzJwAAAAAAAAAAAAAAAAAAAAAAAAAAAAA			SOFTWARE KEYSTORE SYSKM 				  0 NO

```

### Step 5: Encrypt Your Data

#### Encrypting Columns in Tables

```
SQL> CREATE TABLESPACE TEST_pdbtde datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);

Tablespace created.

SQL>  CREATE TABLE Persons (
PersonID int ENCRYPT,
LastName varchar(255) ENCRYPT,
FirstName varchar(255) ENCRYPT,
Address varchar(255) ENCRYPT,
City varchar(255) ENCRYPT
);  2    3    4    5    6    7
PersonID int ENCRYPT,
*
ERROR at line 2:
ORA-28336: cannot encrypt SYS owned objects

```



```
SQL> create user c##tdecdb identified by "Password23";

User created.

SQL> grant dba to user c##tdecdb;
grant dba to user c##tdecdb
             *
ERROR at line 1:
ORA-00987: missing or invalid username(s)


SQL> grant dba to c##tdecdb;

Grant succeeded.

[oracle@tcloud ~]$ sqlplus c##tdecdb/Password23

SQL> show user;
USER is "C##TDECDB"
SQL> show con_name

CON_NAME
------------------------------
CDB$ROOT
SQL>   CREATE TABLE employee (
     first_name VARCHAR2(128),
     last_name VARCHAR2(128),
     empID NUMBER,
     salary NUMBER(6) ENCRYPT
);    2    3    4    5    6

Table created.



```

Encrypting Columns in Tables

![image-20220218155417032](https://raw.githubusercontent.com/aimdotsh/photo/master/typ/image-20220218155417032.png)

![](https://raw.githubusercontent.com/aimdotsh/photo/master/typ/20220217111503.png)

![](https://raw.githubusercontent.com/aimdotsh/photo/master/typ/20220218154709.png)