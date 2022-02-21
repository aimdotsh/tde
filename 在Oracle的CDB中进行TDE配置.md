

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
>
> 创建 pdb 语句。
>
> ```
> create pluggable database tdepdb admin user pdbadmin identified by "Password123";
> alter pluggable database tdepdb open  instances=all;
> alter pluggable database tdepdb save state instances=all;
> ```
>
> ```
> create pluggable database tdepdb2 admin user pdbadmin identified by "Password123";
> alter pluggable database tdepdb2 open  instances=all;
> alter pluggable database tdepdb2 save state instances=all;
> ```
>
> 



[TOC]

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



### Step 3: Open the Software Keystore

> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY software_keystore_password [CONTAINER = ALL | CURRENT];
>
> - *`software_keystore_password`* is the same password that you used to create the keystore in [Step 2: Create the Software Keystore](https://docs.oracle.com/database/121/ASOAG/configuring-transparent-data-encryption.htm#GUID-F098129B-BBFF-4C86-B119-80AB706DB2A1).
> - `CONTAINER` is for use in a multitenant environment. Enter `ALL` to set the keystore in all of the PDBs in this CDB, or `CURRENT` for the current PDB.

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
keystore altered.
SQL> ! ls -l /etc/ORACLE/WALLETS/tdecdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 17 09:32 ewallet.p12
```

可以添加 CONTAINER = ALL 参数，此cdb 所有的pdb生效，使用   `CONTAINER` `=` `ALL`,  必须在root 容器执行，CDB$ROOT ，并且有  `ADMINISTER` `KEY` `MANAGEMENT` or `SYSKM`  权限。

默认是 current，只针对当前 pdb 生效。

可以通过V$ENCRYPTION_WALLET 视图查看状态。

```
 WRL_TYPE	     WRL_PARAMETER					STATUS

-------------------- -------------------------------------------------- ------------------------------

FILE		     /etc/ORACLE/WALLETS/tdecdb/			CLOSED

SQL> l
  1* select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET
SQL>

```



open 之后的状态为 OPEN_NO_MASTER_KEY

```

WRL_TYPE	     WRL_PARAMETER					STATUS
-------------------- -------------------------------------------------- ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/			OPEN_NO_MASTER_KEY

SQL>
```



```
SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 4 TDEPDB2			  READ WRITE NO
SQL>  select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER					STATUS
-------------------- -------------------------------------------------- ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/			CLOSED

SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.

SQL> col  WRL_PARAMETER for a30
SQL> select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER					STATUS
-------------------- -------------------------------------------------- ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/			OPEN_NO_MASTER_KEY

*
ERROR at line 1:
ORA-28365: wallet is not open
如果状态没有open 会提示 ORA-28365: wallet is not open。

```



### Step 4: Set the Software TDE Master Encryption Key

> ADMINISTER KEY MANAGEMENT SET KEY [USING TAG 'tag'] IDENTIFIED BY keystore_password [WITH BACKUP [USING 'backup_identifier']] [CONTAINER = ALL | CURRENT];

```sql
ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY Password23 with backup USING 'tdetest01_key_bak';
ADMINISTER KEY MANAGEMENT SET KEY USING TAG 'masterkey' IDENTIFIED BY Password23 with backup USING 'tdetest01_key_bak';
```

> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys

```sql
SQL> col tag for a20
SQL> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys;

KEY_ID							     TAG	KEYSTORE_TYPE	  USER				     CON_ID BACKED_UP
------------------------------------------------------------ ---------- ----------------- ------------------------------ ---------- ---------
AcBapz/5vk9Av29fJl/NzJwAAAAAAAAAAAAAAAAAAAAAAAAAAAAA			SOFTWARE KEYSTORE SYSKM 				  0 NO

SQL> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys;


KEY_ID									       TAG		    KEYSTORE_TYPE     USER				 CON_ID BACKED_UP
------------------------------------------------------------------------------ -------------------- ----------------- ------------------------------ ---------- ---------
AWeotlcyUk+tv7GiOqqILFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA						    SOFTWARE KEYSTORE SYS				      0 NO
AcZdrtdJe0+fvxhez7ShUQIAAAAAAAAAAAAAAAAAAAAAAAAAAAAA						    SOFTWARE KEYSTORE SYS				      0 YES
AaKq/ko7lk9VvzWShBQ73qkAAAAAAAAAAAAAAAAAAAAAAAAAAAAA						    SOFTWARE KEYSTORE SYS				      0 YES


```

> # ADMINISTER KEY MANAGEMENT
>
> https://docs.oracle.com/database/121/SQLRF/statements_1003.htm#SQLRF55976

```
SQL>  CREATE TABLESPACE TEST_pdbtde datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
 CREATE TABLESPACE TEST_pdbtde datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT)
*
ERROR at line 1:
ORA-28374: typed master key not found in wallet
如果不创建 Master Encryption Key 会提示 ORA-28374: typed master key not found in wallet
```

`WITH BACKUP`创建密钥库的备份。对于基于密码的密钥库，您必须使用此选项。或者，您可以使用该`USING`子句添加备份的简要说明。将此说明用单引号 (' ') 括起来。此标识符附加到命名的密钥库文件（例如，作为备份标识符）。

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

#### Encrypting Tablespaces

字短加密和表空间加密跟 nocdb的方式相同。

<img src="https://raw.githubusercontent.com/aimdotsh/photo/master/typ/image-20220218155417032.png" style="zoom:50%;" />

![](https://raw.githubusercontent.com/aimdotsh/photo/master/typ/20220217111503.png)

<img src="https://raw.githubusercontent.com/aimdotsh/photo/master/typ/20220218154709.png" alt="style=&quot;zoom:50%;&quot;" style="zoom:40%;" />



