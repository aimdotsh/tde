

## 在Oracle非租户环境进行TDE配置

   * [在Oracle非租户环境进行TDE配置](#在oracle非租户环境进行tde配置)
      * [Step 1: Set the Software Keystore Location in the sqlnet.ora File](#step-1-set-the-software-keystore-location-in-the-sqlnetora-file)
      
      * [Step 2: Create the Software Keystore](#step-2-create-the-software-keystore)
      
      * [Step 3: Open the Software Keystore](#step-3-open-the-software-keystore)
      
      * [Step 4: Set the Software TDE Master Encryption Key](#step-4-set-the-software-tde-master-encryption-key)
      
      * [Step 5: Encrypt Your Data](#step-5-encrypt-your-data)
         * [Encrypting Columns in Tables](#encrypting-columns-in-tables)
         
         * [Encrypting Tablespaces](#encrypting-tablespaces)
         
         * [创建非加密表空间进行 strings 对比](#创建非加密表空间进行-strings-对比)
         
           

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

#### Encrypting Tablespaces

加密表空间，需要 create tablespace 权限。

```
SQL> CREATE TABLESPACE TEST_tde datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);

Tablespace created.

sqlplus utde/tdenocdb
  CREATE TABLE testde (
     first_name VARCHAR2(128),
     last_name VARCHAR2(128),
     empID NUMBER,
     salary NUMBER(6) 
) tablespace TEST_tde; 
```

```
[oracle@tcloud ~]$ ./ora dbf|grep TEST_TDE
TEST_TDE		       /u01/app/oracle/oradata/NOCDB/datafile/o1_mf_test_tde_k0y2f9jc_.dbf			   10 NO		  0		   0
[oracle@tcloud ~]$ [oracle@tcloud ~]$  strings /u01/app/oracle/oradata/NOCDB/datafile/o1_mf_test_tde_k0y2f9jc_.dbf|more
}|{z
NOCDB
|bAglbA
TEST_TDE
 n0!
!`,#
Facf
6wd`
Jf!G0
_y*Z
@?Pi
R}|fN
<0gp
_41w
sFx5C7
n1%/lK
J?[
WYP|B
. h(
01!u[2)
b:h63
{X4~f
tW9c1)
52qF7
Kp\^
huGk@Aw
4-Gz
PLP6
8r`r~N
D(Z7
gTxVa
OQbI
\Gyx^
|=I]
;~5W
EV-h+
yVL%
vX"M
J73V
@B59
;W	7]
r1!0}?b
Z>W!Wd
d6mm
```

#### 创建非加密表空间进行 strings 对比

```
CREATE TABLESPACE TEST_ENCRY
datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
create user u4tde identified by u4notde123 default tablespace TEST_ENCRY;
grant dba to u4tde;
sqlplus u4tde/u4notde123
CREATE TABLE u4tde.Persons (
PersonID int ENCRYPT,
LastName varchar(255) ENCRYPT,
FirstName varchar(255) ENCRYPT,
Address varchar(255) ENCRYPT,
City varchar(255) ENCRYPT
);
 
insert into u4tde.Persons values('898899','MOHAMMAD','SHADAB','SURREY HILLS','SYDNEY');
commit;
insert into u4tde.Persons values('898899','CHRIS','MARTIN','SURREY HILLS','SYDNEY');
commit;
insert into u4tde.Persons values('898899','YORKE','THOM','MANSFIELD','MANCHESTER');
commit;
[oracle@tcloud ~]$ strings /u01/app/oracle/oradata/NOCDB/datafile/o1_mf_test_enc_k0y4wwff_.dbf|more
}|{z
NOCDB
bAglbA
TEST_ENCRY
r*H|
XMMC
CVK^
wlkv
+&-|
-e8G'
U#I4B
+|/L
Y*(y[
!zRc
7Ivb
B*~`K
z2!X
bs6^
;Z[US
{:y7
AA}W/Ck
q?dq
C<k>VPxb
a(E9g
Yd#bW
&ug3
q}QU
erP1E
UKn0
\W1H
nna;
IOM!:
8h 	k
N*"w/vCf
"}|u=
h5/4s
UL(}
t<<5
	tFH
ap7`
arlH
1r%yv`
n@.)
{{n+9
```



```
create tablespace test datafile  size 10M;
create user u4notde identified by u4notde default tablespace test;
alter user u4notde identified by u4notde123;
grant dba to u4notde;
sqlplus u4notde/u4notde123
CREATE TABLE u4notde.Persons (
PersonID int,
LastName varchar(255),
FirstName varchar(255),
Address varchar(255),
City varchar(255)
);
 
insert into u4notde.Persons values('898899','MOHAMMAD','SHADAB','SURREY HILLS','SYDNEY');
commit;
insert into u4notde.Persons values('898899','CHRIS','MARTIN','SURREY HILLS','SYDNEY');
commit;
insert into u4notde.Persons values('898899','YORKE','THOM','MANSFIELD','MANCHESTER');
commit;
[oracle@tcloud ~]$ ./ora dbf|grep -iw test
TEST			       /u01/app/oracle/oradata/NOCDB/datafile/o1_mf_test_k0y4zm02_.dbf				   10 NO		  0		   0
[oracle@tcloud ~]$
[oracle@tcloud ~]$ strings /u01/app/oracle/oradata/NOCDB/datafile/o1_mf_test_k0y4zm02_.dbf
}|{z
NOCDB
bAglbA
TEST
YORKE
THOM	MANSFIELD
MANCHESTER,
CHRIS
MARTIN
SURREY HILLS
SYDNEY,
MOHAMMAD
SHADAB
SURREY HILLS
SYDNEY
[oracle@tcloud ~]$
```

