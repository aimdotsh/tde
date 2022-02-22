* [在Oracle PDB中进行TDE配置](#在oracle-pdb中进行tde配置)
  * [0、创建 WALLETS 的文件夹](#0创建-wallets-的文件夹)
  * [1、编辑 sqlnet.ora](#1编辑-sqlnetora)
  * [2、创建 Software Keystore](#2创建-software-keystore)
  * [3、启用 Software Keystore](#3启用-software-keystore)
     * [cdb级别即  root container 上启用](#cdb级别即--root-container-上启用)
     * [登录 pdb](#登录-pdb)
     * [检查状态](#检查状态)
     * [启用 pdb  Software Keystore](#启用-pdb--software-keystore)
     * [检查状态](#检查状态-1)
     * [关闭KEYSTORE](#关闭keystore)
  * [4、 创建 Master Encryption Key](#4-创建-master-encryption-key)
     * [创建 Master Encryption Key](#创建-master-encryption-key)
     * [查询 Master Encryption Key](#查询-master-encryption-key)
  * [5、加密数据](#5加密数据)

# 在Oracle PDB中进行TDE配置

测试信息：

版本：12.1.0.2

PDBNAME

```
create pluggable database tdepdb admin user pdbadmin identified by "Password123";
alter pluggable database tdepdb open  instances=all;
alter pluggable database tdepdb save state instances=all;
```

## 0、创建 WALLETS 的文件夹

使用root 创建单独的目录，权限授予给 oracle 用户。

```shell
mkdir -p /etc/ORACLE/WALLETS/tdecdb
chown -R oracle:oinstall /etc/ORACLE/WALLETS/tdecdb
```

<!--建议将钱包目录放到非 ORACLE 目录下面-->

## 1、编辑 sqlnet.ora

编辑 $ORACLE_HOME/network/admin/sqlnet.ora 新增 ENCRYPTION_WALLET_LOCATION 配置

```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```

## 2、创建 Software Keystore

本步骤需要在root 容器即在  CDB$ROOT 上操作执行。

需要 `ADMINISTER KEY MANAGEMENT` or `SYSKM` privilege. 或者使用 sys / as sysdba创建，本次单独创建用户。

```sql
SQL> create user c##sec_admin identified by "Password123";
User created.
SQL> grant SYSKM to c##sec_admin;
Grant succeeded.
SQL> exit
```

登录 

```sql
sqlplus c##sec_admin as syskm
Enter password: password
Connected.
```



```sql
SQL> ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY Password23;
keystore altered.
```

是在 CDB$ROOT 上执行，不能在pdb级别执行。

创建智慧，作为keystore的 ewallet.p12文件将出现在密钥库/etc/ORACLE/WALLETS/tdecdb 即 sqlnet.ora 配置的位置

```sql
SQL>  ! ls -l /etc/ORACLE/WALLETS/tdecdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 21 10:42 ewallet.p12
```

## 3、启用 Software Keystore

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
keystore altered.
SQL> ! ls -l /etc/ORACLE/WALLETS/tdecdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 17 09:32 ewallet.p12
```

在 pdb 上启用的前提是需要在 root container 上启用。可以通过 添加 CONTAINER = ALL 参数，使所有的 pdb 都生效生效，如果需要单独启用某个pdb，需要先在  root container 上启用，然后再在 pdb 上启用。默认是 current，只针对当前 pdb 生效。

### cdb级别即  root container 上启用

```
SQL>  show con_name
CON_NAME
------------------------------
CDB$ROOT
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
keystore altered.
```

### 登录 pdb

然后登录 tdepdb pdb，启用 Software Keystore

```
SQL>  alter session set container=TDEPDB;

Session altered.
SQL> show con_name
CON_NAME
------------------------------
TDEPDB
```

### 检查状态

```
SQL>  select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER		    STATUS

-------------------- ------------------------------ ------------------------------

FILE		     /etc/ORACLE/WALLETS/tdecdb/    CLOSED
```

### 启用 pdb  Software Keystore

```
SQL>  ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

### 检查状态

第一创建启动没有创建 MASTER_KEY的时候，状态是 OPEN_NO_MASTER_KEY

```
WRL_TYPE	     WRL_PARAMETER			      STATUS
-------------------- ---------------------------------------- ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN_NO_MASTER_KEY
```

如果之前测试过，创建过 MASTER_KEY，状态是 open

```
SQL>   select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET;
WRL_TYPE	     WRL_PARAMETER		    STATUS
-------------------- ------------------------------ ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/    OPEN
```

### 关闭KEYSTORE

关闭 的话使用如下命令

```
ADMINISTER KEY MANAGEMENT SET KEYSTORE CLOSE IDENTIFIED BY "Password23" CONTAINER=ALL;
```

## 4、 创建 Master Encryption Key

每个pdb 都需要创建  Encryption Key。可以通过CONTAINER = ALL 参数创建所有pdb都生效。如果不创建会提示如下ORA-28374: typed master key not found in wallet

```sql
ERROR at line 1:
ORA-28374: typed master key not found in wallet
```

### 创建 Master Encryption Key

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY Password23 with backup; 
keystore altered.
```

### 查询 Master Encryption Key

```
select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys
```

```sql
SQL> set lines 200
SQL> col tag for a20
SQL> col KEY_ID for a60
SQL> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys
KEY_ID							     TAG		  KEYSTORE_TYPE     USER			       CON_ID BACKED_UP
------------------------------------------------------------ -------------------- ----------------- ------------------------------ ---------- ---------
AWu5OW5+ME/0v1fb0UTWgLQAAAAAAAAAAAAAAAAAAAAAAAAAAAAA				  SOFTWARE KEYSTORE SYS 				    0 NO
```

`WITH BACKUP`创建密钥库的备份。对于基于密码的密钥库，您必须使用此选项。或者，您可以使用该`USING`子句添加备份的简要说明。将此说明用单引号 (' ') 括起来。此标识符附加到命名的密钥库文件（例如，作为备份标识符）。

## 5、加密数据

经过以上配置，就可以对pdb数据进行加密了，包括表的列加密和表空间加密，跟nocdb的列加密和表空间加密类似。详见 [Step 5: Encrypt Your Data](https://github.com/aimdotsh/tde/blob/main/在Oracle非租户环境进行TDE配置.md#step-5-encrypt-your-data)

