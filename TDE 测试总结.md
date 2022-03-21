

[TOC]

* [一、ORACLE 12.1.0.2 TDE 配置](#一oracle-12102-tde-配置)
   * [1.1 在Oracle非租户环境进行TDE配置](#11-在oracle非租户环境进行tde配置)
      * [1.1.1、创建钱包文件夹并编辑 sqlnet.ora](#111创建钱包文件夹并编辑-sqlnetora)
      * [1.1.2、创建 Software Keystore](#112创建-software-keystore)
      * [1.1.3、打开 Software Keystore](#113打开-software-keystore)
      * [1.1.4、设置 Master Encryption Key](#114设置-master-encryption-key)
      * [1.1.5、加密数据](#115加密数据)
         * [a.加密列数据](#a加密列数据)
         * [b.加密表空间](#b加密表空间)
         * [c.创建非加密表空间进行 strings 对比](#c创建非加密表空间进行-strings-对比)
   * [1.2 ORACLE 在CDB 的 TDE 配置](#12-oracle-在cdb-的-tde-配置)
      * [1.2.1、创建钱包文件夹并编辑 sqlnet.ora](#121创建钱包文件夹并编辑-sqlnetora)
      * [1.2.2、创建 Software Keystore](#122创建-software-keystore)
      * [1.2.3、 打开 Software Keystore](#123-打开-software-keystore)
      * [1.2.4、设置 TDE Master Encryption Key](#124设置-tde-master-encryption-key)
      * [1.2.5、 加密数据](#125-加密数据)
         * [a.加密列数据](#a加密列数据-1)
         * [b.加密表空间](#b加密表空间-1)
      * [1.2.6、修改 Software Keystore Password](#126修改-software-keystore-password)
      * [1.2.7、TDE 检查状态的脚本](#127tde-检查状态的脚本)
   * [1.3 在Oracle PDB中进行TDE配置](#13-在oracle-pdb中进行tde配置)
      * [1.3.1、创建 WALLETS 的文件夹](#131创建-wallets-的文件夹)
      * [1.3.2、编辑 sqlnet.ora](#132编辑-sqlnetora)
      * [1.3.3、创建 Software Keystore](#133创建-software-keystore)
      * [1.3.4、启用 Software Keystore](#134启用-software-keystore)
         * [a.CDB 级别即  root container 上启用](#acdb-级别即--root-container-上启用)
         * [b.登录 PDB](#b登录-pdb)
         * [c.检查状态](#c检查状态)
         * [d.启用 PDB  Software Keystore](#d启用-pdb--software-keystore)
         * [e.关闭KEYSTORE](#e关闭keystore)
      * [1.3.5、 创建 Master Encryption Key](#135-创建-master-encryption-key)
         * [a.创建 Master Encryption Key](#a创建-master-encryption-key)
         * [b.检查 Master Encryption Key](#b检查-master-encryption-key)
   * [1.4 TDE 配置总结](#14-tde-配置总结)
* [二、PDB的创建、克隆、插拔等功能测试](#二pdb的创建克隆插拔等功能测试)
   * [2.1、测试前的准备](#21测试前的准备)
   * [2.2、PDB 的创建、克隆](#22pdb-的创建克隆)
      * [2.2.1、 使用seed创建新的 PDB](#221-使用seed创建新的-pdb)
      * [2.2.2、 从本地 PDB 进行克隆](#222-从本地-pdb-进行克隆)
         * [a.源 PDB 未启用 TDE加密进行克隆](#a源-pdb-未启用-tde加密进行克隆)
         * [b.源库启用 TDE加密进行克隆](#b源库启用-tde加密进行克隆)
            * [i) .钱包打开进行克隆数据库测试](#i-钱包打开进行克隆数据库测试)
               * [克隆数据库不包含数据](#克隆数据库不包含数据)
               * [克隆数据库包含数据](#克隆数据库包含数据)
            * [ii). 钱包关闭进行测试](#ii-钱包关闭进行测试)
               * [查看alert日志](#查看alert日志)
               * [查看视图信息](#查看视图信息)
               * [查看钱包状态](#查看钱包状态)
               * [首先从源库导出密钥](#首先从源库导出密钥)
               * [目标库导入](#目标库导入)
               * [然后导入密钥](#然后导入密钥)
               * [查看状态](#查看状态)
               * [重启数据库](#重启数据库)
               * [告警已解决](#告警已解决)
   * [2.3、总结](#23总结)
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
   * [2.3、TDE PDB 插拔测试](#23tde-pdb-插拔测试)
      * [1、拔出PDB](#1拔出pdb)
         * [1.1、导出密钥](#11导出密钥)
         * [1.2、关闭 PDB](#12关闭-pdb)
         * [1.3、拔出 PDB](#13拔出-pdb)
      * [2、 插入目标CDB中](#2-插入目标cdb中)
         * [2.1、插入目标 CDB](#21插入目标-cdb)
         * [2.2、 检查插入的PDB](#22-检查插入的pdb)
            * [2.2.1、打开目标数据库](#221打开目标数据库)
            * [2.2.2、检查日志](#222检查日志)
         * [2.3、导入密钥key](#23导入密钥key)
            * [2.3.1、目标库打开钱包](#231目标库打开钱包)
            * [2.3.2、导入密钥](#232导入密钥)
            * [2.3.3、查看状态](#233查看状态)
            * [2.3.4、重启数据库](#234重启数据库)
      * [3、PDB 插拔总结](#3pdb-插拔总结)
         * [3.1、从源库导出密钥](#31从源库导出密钥)
         * [3.2、目标库导入](#32目标库导入)
            * [3.2.1、打开钱包](#321打开钱包)
            * [3.2.2、导入密钥](#322导入密钥)
         * [3.3、检查钱包状态](#33检查钱包状态)
         * [3.4、重启数据库](#34重启数据库)
         * [3.5、检查日志确认日志无误](#35检查日志确认日志无误)
* [三、TDE 表空间与列加密测试](#三tde-表空间与列加密测试)
   * [3.1 表空间加密测试](#31-表空间加密测试)
      * [3.1.1、测试 undo 表空间加密](#311测试-undo-表空间加密)
      * [3.1.2、测试 temp 表空间](#312测试-temp-表空间)
      * [3.1.3、测试普通表空间加密](#313测试普通表空间加密)
      * [3.1.4、测试big file表空间加密](#314测试big-file表空间加密)
   * [3.2、列加密测试](#32列加密测试)
      * [3.2.1、增加一个加密列](#321增加一个加密列)
      * [3.2.2、更新加密列](#322更新加密列)
      * [3.2.3、修改表中未加密的列](#323修改表中未加密的列)
      * [3.2.4、修改普通列为加密列](#324修改普通列为加密列)
      * [3.2.5、取消加密列](#325取消加密列)
      * [3.2.6、索引列加密](#326索引列加密)
      * [3.2.7、加密与物化视图](#327加密与物化视图)
      * [3.2.8、加密与DBLINK](#328加密与dblink)
      * [3.2.9、分区表测试](#329分区表测试)
   * [3.3、测试钱包关闭/打开的情况下访问非加密列及加密列的情况](#33测试钱包关闭打开的情况下访问非加密列及加密列的情况)
      * [3.3.1、钱包打开的情况访问数据测试](#331钱包打开的情况访问数据测试)
      * [3.3.2、钱包关闭的情况访问数据测试](#332钱包关闭的情况访问数据测试)
   * [3.4、表空间及列加密总结](#34表空间及列加密总结)
* [四、TDE 性能测试](#四tde-性能测试)
   * [4.1、测试前数据准备](#41测试前数据准备)
      * [4.1.1、创建表空间](#411创建表空间)
      * [4.1.2、准备测试表、测试数据](#412准备测试表测试数据)
   * [4.2、测试对执行计划的影响](#42测试对执行计划的影响)
   * [4.3、加密表作为被驱动表可以使用索引](#43加密表作为被驱动表可以使用索引)
* [五、TDE 与 ADG 测试](#五tde-与-adg-测试)
   * [5.1、DG 钱包配置准备](#51dg-钱包配置准备)
      * [5.1.1、复制源库的密钥到目标库的tde 钱包路径](#511复制源库的密钥到目标库的tde-钱包路径)
      * [5.1.2、DG 配置 sqlnet.ora](#512dg-配置-sqlnetora)
      * [5.1.3、DG 数据库启动到 mount 状态](#513dg-数据库启动到-mount-状态)
      * [5.1.4、DG 启用 Software Keystore](#514dg-启用-software-keystore)
   * [5.2、rman dumpicate 搭建 DG](#52rman-dumpicate-搭建-dg)
      * [5.2.1、主库配置归档模式](#521主库配置归档模式)
         * [a.主库配置 force logging 模式](#a主库配置-force-logging-模式)
         * [c.主库配置归档模式](#c主库配置归档模式)
         * [c.检查归档模式](#c检查归档模式)
      * [5.2.2、配置静态监听及tns](#522配置静态监听及tns)
         * [a.重启监听](#a重启监听)
         * [b.配置 tns](#b配置-tns)
         * [c.测试 tns](#c测试-tns)
      * [5.2.3、新增 redo log file 和 standby log file](#523新增-redo-log-file-和-standby-log-file)
      * [5.2.4、主库修改数据库参数](#524主库修改数据库参数)
      * [5.2.5、复制密码文件到备库](#525复制密码文件到备库)
      * [5.2.6、备库启动数据库实例](#526备库启动数据库实例)
         * [a.备库创建审计日志目录](#a备库创建审计日志目录)
         * [b.备库启动数据库实例到 nomount 状态](#b备库启动数据库实例到-nomount-状态)
      * [5.2.7、rman duplicate 搭建DG从库](#527rman-duplicate-搭建dg从库)
         * [a.查看DG数据同步详情](#a查看dg数据同步详情)
         * [b.dginfo.sh使用方法：](#bdginfosh使用方法)
* [六、TDE exp/expdp 及 rman 测试](#六tde-expexpdp-及-rman-测试)
   * [6.1、TDE exp/imp 测试](#61tde-expimp-测试)
      * [6.1.1、exp 导出数据测试](#611exp-导出数据测试)
         * [a.钱包打开的情况下测试exp](#a钱包打开的情况下测试exp)
         * [b.钱包关闭情况下exp导出](#b钱包关闭情况下exp导出)
         * [c.总结](#c总结)
      * [6.1.2、TDE imp 导入数据测试](#612tde-imp-导入数据测试)
      * [6.1.3、TDE imp 导入数据总结](#613tde-imp-导入数据总结)
      * [6.1.4、expdp 导出数据测试](#614expdp-导出数据测试)
         * [a.钱包打开的情况 expdp 测试](#a钱包打开的情况-expdp-测试)
         * [b.钱包关闭的情况 expdp 测试](#b钱包关闭的情况-expdp-测试)
         * [c.总结](#c总结-1)
      * [TDE impdp 导入数据测试](#tde-impdp-导入数据测试)
         * [a.impdp 在钱包关闭的情况下导入](#aimpdp-在钱包关闭的情况下导入)
         * [b.impdp 在钱包打开的情况下导入](#bimpdp-在钱包打开的情况下导入)
         * [c.总结](#c总结-2)
   * [6.2、TDE rman  测试](#62tde-rman--测试)
      * [6.2.1、rman 不压缩备份测试](#621rman-不压缩备份测试)
      * [总结](#总结)
      * [6.2.2、rman 压缩备份测试](#622rman-压缩备份测试)
         * [a.钱包关闭 rman 压缩测试](#a钱包关闭-rman-压缩测试)
         * [b.钱包打开 rman 压缩测试](#b钱包打开-rman-压缩测试)
      * [总结](#总结-1)
      * [rman 恢复](#rman-恢复)
         * [1、目标库启用 tde](#1目标库启用-tde)
            * [1.1、编辑 sqlnet.ora](#11编辑-sqlnetora)
            * [1.2、将源库的key 复制到目标库](#12将源库的key-复制到目标库)
            * [1.3、启用  Software Keystore](#13启用--software-keystore)
         * [2、复制备份文件到目标环境](#2复制备份文件到目标环境)
         * [3、恢复参数文件](#3恢复参数文件)
         * [4、恢复控制文件](#4恢复控制文件)
         * [5、启动数据库到mount 状态](#5启动数据库到mount-状态)
         * [6、restore database;](#6restore-database)
         * [7、recover database](#7recover-database)
      * [tde rman 恢复总结](#tde-rman-恢复总结)
* [七、TDE 与 RAC 测试](#七tde-与-rac-测试)
   * [7.1、 创建密钥目录并编辑 sqlnet.ora](#71-创建密钥目录并编辑-sqlnetora)
   * [7.2、创建 Key store](#72创建-key-store)
   * [7.3、检查 CDB 钱包状态](#73检查-cdb-钱包状态)
   * [7.4、打开 PDB](#74打开-pdb)
   * [7.5、在 PDB 上打开钱包](#75在-pdb-上打开钱包)
   * [7.6、创建加密表空间](#76创建加密表空间)
   * [7.7、RAC环境总结](#77rac环境总结)
* [八、TDE 与 OGG 测试](#八tde-与-ogg-测试)

# 一、ORACLE 12.1.0.2 TDE 配置

## 1.1 在Oracle非租户环境进行TDE配置

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

### 1.1.1、创建钱包文件夹并编辑 sqlnet.ora

```
mkdir -p /etc/ORACLE/WALLETS/nocdb
chown -R oracle:oinstall /etc/ORACLE/WALLETS/nocdb
```

vi "$ORACLE_HOME"/network/admin/sqlnet.ora

```
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/nocdb)))
```



### 1.1.2、创建 Software Keystore

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



### 1.1.3、打开 Software Keystore

> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY software_keystore_password [CONTAINER = ALL | CURRENT];

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
SQL>  ! ls -l /etc/ORACLE/WALLETS/nocdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 18 10:09 ewallet.p12

```

### 1.1.4、设置 Master Encryption Key

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

### 1.1.5、加密数据

#### a.加密列数据

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

#### b.加密表空间

加密表空间，需要 create tablespace 权限。

```sql
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

#### c.创建非加密表空间进行 strings 对比

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





## 1.2 ORACLE 在CDB 的 TDE 配置

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
> 创建 PDB 语句。
>
> ```
> create pluggable database tdepdb admin user pdbadmin identified by "Password123";
> alter pluggable database tdepdb open  instances=all;
> alter pluggable database tdepdb save state instances=all;
> ```



### 1.2.1、创建钱包文件夹并编辑 sqlnet.ora

```shell
mkdir -p /etc/ORACLE/WALLETS/tdecdb
chown -R oracle:oinstall /etc/ORACLE/WALLETS/tdecdb
```

<!--建议将钱包目录放到非 ORACLE 目录下面-->

使用root 创建单独的目录，权限授予给 oracle 用户。



```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```



### 1.2.2、创建 Software Keystore

创建用户

需要 `ADMINISTER KEY MANAGEMENT` or `SYSKM` privilege.

```sql
SQL> create user c##sec_admin identified by "Password123";
User created.
SQL> grant SYSKM to c##sec_admin;
Grant succeeded.
SQL> exit
```

```sql
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

```sql
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



### 1.2.3、 打开 Software Keystore

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

默认是 current，只针对当前 PDB 生效。

可以通过V$ENCRYPTION_WALLET 视图查看状态。

```sql
 WRL_TYPE	     WRL_PARAMETER					STATUS

-------------------- -------------------------------------------------- ------------------------------

FILE		     /etc/ORACLE/WALLETS/tdecdb/			CLOSED

SQL> l
  1* select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET
SQL>

```



open 之后的状态为 OPEN_NO_MASTER_KEY

```sql
WRL_TYPE	     WRL_PARAMETER					STATUS
-------------------- -------------------------------------------------- ------------------------------
FILE		     /etc/ORACLE/WALLETS/tdecdb/			OPEN_NO_MASTER_KEY

SQL>
```



```sql
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



关闭 的话使用如下命令

```
ADMINISTER KEY MANAGEMENT SET KEYSTORE CLOSE IDENTIFIED BY "Password23" CONTAINER=ALL;
```



### 1.2.4、设置 TDE Master Encryption Key

> ADMINISTER KEY MANAGEMENT SET KEY [USING TAG 'tag'] IDENTIFIED BY keystore_password [WITH BACKUP [USING 'backup_identifier']] [CONTAINER = ALL | CURRENT];

```sql
ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY Password23 with backup USING 'tdetest01_key_bak';
ADMINISTER KEY MANAGEMENT SET KEY USING TAG 'masterkey' IDENTIFIED BY Password23 with backup USING 'tdetest01_key_bak';
```

> select key_id,tag,KEYSTORE_TYPE,USER,CON_ID,BACKED_UP from  v$encryption_keys

```sql
SQL> set lines 200
SQL> col tag for a20
SQL> col KEY_ID for a60
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

> ADMINISTER KEY MANAGEMENT
>
> https://docs.oracle.com/database/121/SQLRF/statements_1003.htm#SQLRF55976

```sql
SQL>  CREATE TABLESPACE TEST_pdbtde datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
 CREATE TABLESPACE TEST_pdbtde datafile  size 10M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT)
*
ERROR at line 1:
ORA-28374: typed master key not found in wallet
如果不创建 Master Encryption Key 会提示 ORA-28374: typed master key not found in wallet
```

`WITH BACKUP`创建密钥库的备份。对于基于密码的密钥库，您必须使用此选项。或者，您可以使用该`USING`子句添加备份的简要说明。将此说明用单引号 (' ') 括起来。此标识符附加到命名的密钥库文件（例如，作为备份标识符）。

### 1.2.5、 加密数据

#### a.加密列数据

```sql
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



```sql
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

#### b.加密表空间

字短加密和表空间加密跟 nocdb的方式相同。

详见 [Step 5: Encrypt Your Data](https://github.com/aimdotsh/tde/blob/main/在Oracle非租户环境进行TDE配置.md#step-5-encrypt-your-data)

```sql
 select tablespace_name, encrypted from dba_tablespaces;
```

查看表空间是否加密

### 1.2.6、修改 Software Keystore Password

由于cdb和pdb统一使用一个钱包，修改cdb的密码之后pdb的密码也一并修改。

> ADMINISTER KEY MANAGEMENT ALTER KEYSTORE PASSWORD IDENTIFIED BY
> old_password SET new_password [WITH BACKUP [USING 'backup_identifier']];

如果要修改密码需要保证其状态为 open 状态，否则会报

```
ERROR at line 1:
ORA-46658: keystore not open in the container
```

以下是打开命令

```
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY newPassword23;
```

修改密码具体测试如下：

```sql
ADMINISTER KEY MANAGEMENT ALTER KEYSTORE PASSWORD IDENTIFIED BY Password23 set newPassword23 WITH BACKUP;
keystore altered.
SQL>
```

修改成功。

### 1.2.7、TDE 检查状态的脚本

```sql
SELECT * FROM V$ENCRYPTION_WALLET;
SELECT * FROM V$ENCRYPTION_KEYS;
SELECT WRL_PARAMETER,STATUS,WALLET_TYPE FROM V$ENCRYPTION_WALLET;
SELECT KEY_ID,KEYSTORE_TYPE FROM V$ENCRYPTION_KEYS;
SELECT KEY_ID FROM V$ENCRYPTION_KEYS;
SELECT KEYSTORE_TYPE FROM V$ENCRYPTION_KEYS;
SELECT WRL_PARAMETER FROM V$ENCRYPTION_WALLET;
SELECT STATUS FROM V$ENCRYPTION_WALLET;
SELECT * FROM V$ENCRYPTED_TABLESPACES;
SELECT TABLESPACE_NAME, ENCRYPTED FROM DBA_TABLESPACES;
SELECT * FROM DBA_ENCRYPTED_COLUMNS;
```

 





## 1.3 在Oracle PDB中进行TDE配置

测试信息：

版本：12.1.0.2

PDBNAME

```
create pluggable database tdepdb admin user pdbadmin identified by "Password123";
alter pluggable database tdepdb open  instances=all;
alter pluggable database tdepdb save state instances=all;
```

### 1.3.1、创建 WALLETS 的文件夹

使用root 创建单独的目录，权限授予给 oracle 用户。

```shell
mkdir -p /etc/ORACLE/WALLETS/tdecdb
chown -R oracle:oinstall /etc/ORACLE/WALLETS/tdecdb
```

<!--建议将钱包目录放到非 ORACLE 目录下面-->

### 1.3.2、编辑 sqlnet.ora

编辑 $ORACLE_HOME/network/admin/sqlnet.ora 新增 ENCRYPTION_WALLET_LOCATION 配置

```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```

### 1.3.3、创建 Software Keystore

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

### 1.3.4、启用 Software Keystore

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
keystore altered.
SQL> ! ls -l /etc/ORACLE/WALLETS/tdecdb
total 4
-rw-r--r-- 1 oracle oinstall 2408 Feb 17 09:32 ewallet.p12
```

在 PDB 上启用的前提是需要在 root container 上启用。可以通过 添加 CONTAINER = ALL 参数，使所有的 PDB 都生效生效，如果需要单独启用某个pdb，需要先在  root container 上启用，然后再在 PDB 上启用。默认是 current，只针对当前 PDB 生效。

#### a.CDB 级别即  root container 上启用

```
SQL>  show con_name
CON_NAME
------------------------------
CDB$ROOT
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
keystore altered.
```

#### b.登录 PDB

然后登录 tdepdb pdb，启用 Software Keystore

```
SQL>  alter session set container=TDEPDB;

Session altered.
SQL> show con_name
CON_NAME
------------------------------
TDEPDB
```

#### c.检查状态

```
SQL>  select WRL_TYPE,WRL_PARAMETER,status from	V$ENCRYPTION_WALLET;

WRL_TYPE	     WRL_PARAMETER		    STATUS

-------------------- ------------------------------ ------------------------------

FILE		     /etc/ORACLE/WALLETS/tdecdb/    CLOSED
```

#### d.启用 PDB  Software Keystore

```
SQL>  ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

在没有创建 MASTER_KEY的时候，第一次启动其状态是 OPEN_NO_MASTER_KEY

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

#### e.关闭KEYSTORE

关闭 的话使用如下命令

```
ADMINISTER KEY MANAGEMENT SET KEYSTORE CLOSE IDENTIFIED BY "Password23" CONTAINER=ALL;
```

### 1.3.5、 创建 Master Encryption Key

每个pdb 都需要创建  Encryption Key。可以通过CONTAINER = ALL 参数创建所有pdb都生效。如果不创建会提示如下ORA-28374: typed master key not found in wallet

```sql
ERROR at line 1:
ORA-28374: typed master key not found in wallet
```

#### a.创建 Master Encryption Key

```sql
SQL> ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY Password23 with backup; 
keystore altered.
```

#### b.检查 Master Encryption Key

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

## 1.4 TDE 配置总结

纵观以上几种情况，在12cR1环境配置CDB，主要是0.创建 WALLETS 的文件夹；1.编辑 sqlnet.ora；2.创建 Software Keystore；3.启用 Software Keystore；4.创建 Master Encryption Key 五个步骤。



# 二、PDB的创建、克隆、插拔等功能测试

## 2.1、测试前的准备

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
> 通过 配置环境变量可以很方便的看到是在哪个 PDB 下面操作。
>
> vi $ORACLE_HOME/sqlplus/admin/glogin.sql
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

## 2.2、PDB 的创建、克隆 

### 2.2.1、 使用seed创建新的 PDB



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

使用seed创建新的 TDE加密pdb 完成。

删除 pdb

```sql
 alter Pluggable database PDBORCL close;
 drop pluggable database PDBORCL including datafiles;
```

### 2.2.2、 从本地 PDB 进行克隆

> ![Description of Figure 38-3 follows](http://pic.liups.com/images/2022/03/13/202203131726296.png)
>
> ```sql
> SQL>  CREATE PLUGGABLE DATABASE hrpdb FROM salespdb no data STORAGE unlimited;
> Pluggable database created.
> SYS@ora12c> col name for a15
> SYS@ora12c> select con_id, name,open_mode from v$containers;
>  CON_ID NAME            OPEN_MODE
> ---------- --------------- --------------------
>       1 CDB$ROOT        READ WRITE
>       2 PDB$SEED        READ ONLY
>       3 SALESPDB        READ ONLY
>       4 HRPDB           MOUNTED
> ```
>
> ```
> show pdbs;
> 
>  CON_ID CON_NAME			  OPEN MODE  RESTRICTED
> ---------- ------------------------------ ---------- ----------
> 	 2 PDB$SEED			  READ ONLY  NO
> 	 3 TDEPDB			  READ WRITE NO
> 	 4 TDEPDB2			  READ WRITE NO
> 	 6 SALESPDB			  READ WRITE NO
> 
> CREATE PLUGGABLE DATABASE hrpdb_tde FROM TDEPDB  STORAGE unlimited;
> ```



#### a.源 PDB 未启用 TDE加密进行克隆

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

#### b.源库启用 TDE加密进行克隆

分两种情况进行测试

##### i) .钱包打开进行克隆数据库测试

###### 克隆数据库不包含数据

```sql
 CREATE PLUGGABLE DATABASE hrpdbtdenodata FROM TDEPDB no data STORAGE unlimited;
Pluggable database created.
sys@tdecdb(26)>  alter Pluggable database hrpdbtdenodata open;

Warning: PDB altered with errors.
	6 HRPDBTDENODATA		  READ WRITE YES
```

###### 克隆数据库包含数据

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



##### ii). 钱包关闭进行测试

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





###### 查看alert日志

```shell
***************************************************************
WARNING: Pluggable Database HRPDB_CLOSE with PDB id - 8 is
         altered with errors or warnings. Please look into
         PDB_PLUG_IN_VIOLATIONS view for more details.
***************************************************************
```

###### 查看视图信息

```sql
 select * from PDB_PLUG_IN_VIOLATIONS;
TIME			       NAME	       CAUSE		    TYPE      ERROR_NUMBER	 LINE MESSAGE						 STATUS    ACTION
------------------------------ --------------- -------------------- --------- ------------ ---------- -------------------------------------------------- --------- --------------------------------------------------
14-MAR-22 07.27.02.585205 AM   HRPDB_TDE       Wallet Key Needed    ERROR		 0	    1 PDB needs to import keys from source.		 PENDING   Import keys from source.
14-MAR-22 07.34.35.744234 AM   HRPDB_TDECLOSE  Wallet Key Needed    ERROR		 0	    1 PDB needs to import keys from source.		 PENDING   Import keys from source.
14-MAR-22 07.37.33.449191 AM   HRPDB_CLOSE     Wallet Key Needed    ERROR		 0	    1 PDB needs to import keys from source.		 PENDING   Import keys from source.

```

提示： PDB needs to import keys from source

###### 查看钱包状态

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

###### 首先从源库导出密钥

源库导出密钥

```sql
ADMINISTER KEY MANAGEMENT EXPORT ENCRYPTION KEYS WITH SECRET "mySecret" TO '/tmp/export.p12' IDENTIFIED BY Password23;
```

###### 目标库导入

导入之前需要保证 钱包是 open 状态，否则

```shell
ERROR at line 1:
ORA-46658: keystore not open in the container
```

```sql
sys@tdecdb(26)> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

###### 然后导入密钥

```sql
sys@tdecdb(26)>  administer key management import encryption keys  with secret "mySecret" from '/tmp/export.p12' identified by  Password23 with backup;

keystore altered.
```

###### 查看状态

```sql
select * from 	V$ENCRYPTION_WALLET;
WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID

-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------

FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN			     PASSWORD		  SINGLE    NO		       0

```

###### 重启数据库

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

###### 告警已解决

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

## 2.3、总结

源库钱包不管是否关闭，克隆的数据库是否包含数据，都可以进行克隆，库可以 open，状态 RESTRICTED 为yes ，会有个  Warning: PDB altered with errors. 查询钱包状态是 close，无法直接查询加密数据

### 3、 从远程 PDB 进行克隆

> 命令跟从本地克隆是差不多的，就是把本地的 PDB 名字换成了远程pdb名字@dblink。
>
> ```
> CREATE PLUGGABLE DATABASE hrpdb FROM salespdb no data STORAGE unlimited;
> CREATE PLUGGABLE DATABASE ORA12CPDB2 FROM PDBTEST@dlink NO DATA STORAGE unlimited;
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

## 2.3、TDE PDB 插拔测试

> https://docs.oracle.com/database/121/ADMIN/cdb_plug.htm#ADMIN13855

### 1、拔出PDB

在插入PDB时有以下两个因素要考虑。

源CDB和目标CDB要有相同的字节存储顺序，字符集要么相同，要么是子集。本次先测试没有配置tde加密的pdb 插拔操作。

       操作时在create pluggable database 中使用using子句，指定一个XML的PDB元数据文件或一个压缩的PDB归档文件(.pdb类型的文件)来创建一个PDB。
       XML元数据文件描述了未插入的PDB和与PDB相关的文件(例如数据文件和钱包文件)。
       归档文件包括 XML 元数据文件和PDB文件。
       当指定 XML 元数据时，XML 文件包含PDB文件的完整路径。
       当指定 .pdb 归档文件时，XML 元数据文件只包含相对文件名。

数据文件存储目录

       通过.pdb归档文件插入PDB，不需要添加参数来指定数据文件的存放位置，因为oracle会将归档文件解压到当前目录，当前目录就是数据文件的存放位置。
    
        通过XML元数据文件插入PDB，需要添加source_file_name_convert 或者source_file_directory字句来指定文件存放路径。因为XML元数据文件里存放的是PDB之前的数据文件的绝对路径，该路径不一定存在所以需要指定该参数。

**拔出 PDB 操作示例**

本示例将 tdepdb 插拔到相同cdb 实例下面的 cbtdepdb 。

#### 1.1、导出密钥

源库导出密钥

```sql
ADMINISTER KEY MANAGEMENT EXPORT ENCRYPTION KEYS WITH SECRET "mySecret" TO '/tmp/export.p12' IDENTIFIED BY Password23;
```

**拔出PDB  被拔出的PDB只能被删除,不能做其它操作,比如打开等.** 

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
```

#### 1.2、关闭 PDB

```
sys@(25_TDEPDB)> alter pluggable database TDEPDB close;
Pluggable database altered.
```

#### 1.3、拔出 PDB 

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

将 PDB 拔下之后，是无法再打开次数据pdb，只能进行drop操作了。

```
sys@(214_CDB$ROOT)> alter session set container=TDEPDB;
Session altered.
sys@(214_CDB$ROOT)> startup
ORA-65086: cannot open/close the pluggable database
```



### 2、 插入目标CDB中

使用XML文件, 由于是在同一个cdb 中进行测试，需要将源pdb 删除掉，删除的时候保留数据文件，如果不删除源pdb 会报以下

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

#### 2.1、插入目标 CDB

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

#### 2.2、 检查插入的PDB

##### 2.2.1、打开目标数据库

```
sys@(11_CDB$ROOT)> alter pluggable database CBTDEPDB open;
Warning: PDB altered with errors.
```

同样显示有告警

##### 2.2.2、检查日志

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

#### 2.3、导入密钥key

##### 2.3.1、目标库打开钱包

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

##### 2.3.2、导入密钥

```sql
sys@tdecdb(26)>  administer key management import encryption keys  with secret "mySecret" from '/tmp/export.p12' identified by  Password23 with backup;

keystore altered.
```

##### 2.3.3、查看状态

```sql
select * from 	V$ENCRYPTION_WALLET;
WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID

-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------

FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN			     PASSWORD		  SINGLE    NO		       0

```

##### 2.3.4、重启数据库

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



### 3、PDB 插拔总结

通过以上测试：数据库的创建、插拔、克隆等，如果源 PDB 启用了 tde，那么在跟普通PDB的操作多了一步导入密钥key的操作。

#### 3.1、从源库导出密钥

源库导出密钥

```sql
ADMINISTER KEY MANAGEMENT EXPORT ENCRYPTION KEYS WITH SECRET "mySecret" TO '/tmp/export.p12' IDENTIFIED BY Password23;
```

#### 3.2、目标库导入

导入之前需要保证 钱包是 open 状态，否则

```shell
ERROR at line 1:
ORA-46658: keystore not open in the container
```

##### 3.2.1、打开钱包

```sql
sys@tdecdb(26)> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;

keystore altered.
```

##### 3.2.2、导入密钥

```sql
sys@tdecdb(26)>  administer key management import encryption keys  with secret "mySecret" from '/tmp/export.p12' identified by  Password23 with backup;

keystore altered.
```

#### 3.3、检查钱包状态

```sql
select * from 	V$ENCRYPTION_WALLET;
WRL_TYPE	     WRL_PARAMETER			      STATUS			     WALLET_TYPE	  WALLET_OR FULLY_BAC	  CON_ID

-------------------- ---------------------------------------- ------------------------------ -------------------- --------- --------- ----------

FILE		     /etc/ORACLE/WALLETS/tdecdb/	      OPEN			     PASSWORD		  SINGLE    NO		       0

```

#### 3.4、重启数据库

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

#### 3.5、检查日志确认日志无误

```
set linesize 1000;
column MESSAGE format a50;
column name format a16;
column ACTION format a64;
column type format a16;
column cause format a24;

sys@(11_CDB$ROOT)> select name,cause,type,status,message,action from pdb_plug_in_violations where name ='CBTDEPDB';
```



#  三、TDE 表空间与列加密测试

## 3.1 表空间加密测试

### 3.1.1、测试 undo 表空间加密

```
SQL> create undo tablespace test1_undo datafile   size 20m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
create undo tablespace test1_undo datafile   size 20m	ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT)
                                                        *
ERROR at line 1:
ORA-30024: Invalid specification for CREATE UNDO TABLESPACE


SQL>
```

### 3.1.2、测试 temp 表空间

```
SQL> create temporary tablespace user_temp  tempfile size 50m  autoextend on  next 50m maxsize 20480m
  2  ;

Tablespace created.

SQL> create temporary tablespace user_temp2  tempfile size 50m  autoextend on  next 50m maxsize 20480m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
create temporary tablespace user_temp2	tempfile size 50m  autoextend on  next 50m maxsize 20480m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT)
                                                                                                    *
ERROR at line 1:
ORA-25139: invalid option for CREATE TEMPORARY TABLESPACE


SQL>
```

### 3.1.3、测试普通表空间加密

```
SQL> create   tablespace put_tbs  datafile size 2m  autoextend on  next 50m maxsize 20480m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);

Tablespace created.

SQL>
```

### 3.1.4、测试big file表空间加密

```
SQL> CREATE bigfile TABLESPACE bigf_tbs2e DATAFILE SIZE 150M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);

Tablespace created.

SQL>
```

## 3.2、列加密测试

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

### 3.2.1、增加一个加密列

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

### 3.2.2、更新加密列 

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

### 3.2.3、修改表中未加密的列



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

### 3.2.4、修改普通列为加密列

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

### 3.2.5、取消加密列

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

### 3.2.6、索引列加密

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



###  3.2.7、加密与物化视图

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

### 3.2.8、加密与DBLINK

由于目前 TDE配置之后，数据库的连接跟普通的连接方式相同，索引dblink的创建方式跟不创建tde的方式相同。

```sql
CREATE DATABASE LINK dl4tdepdb CONNECT TO system IDENTIFIED BY Password123 USING 'TDEPDB';
```

### 3.2.9、分区表测试

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





## 3.3、测试钱包关闭/打开的情况下访问非加密列及加密列的情况

### 3.3.1、钱包打开的情况访问数据测试

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

### 3.3.2、钱包关闭的情况访问数据测试

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

## 3.4、表空间及列加密总结

钱包关闭的情况测试发现：在钱包关闭的情况下可以访问没有加密的列，但不能够访问加密的列，同样 dml 操作涉及到加密列也是无法进行操作加密列，但是如果 dml 操作中没有加密列 dml操作是可以进行的，比如删除、更新数据。

# 四、TDE 性能测试

## 4.1、测试前数据准备



> **Known Performance Issues When Using TDE and Indexes on the Encrypted Columns (Doc ID 728292.1)**

```
create table hr.t_test( id varchar2(10),name varchar2(100));
insert into hr.t_test select level,'test'||level from dual connect by level<1000000;
create table hr.t_test2( id varchar2(10),name varchar2(100));
insert into hr.t_test2 select level,'test'||level from dual connect by level<1000000;

declare
v_num number;
v_id varchar2(10):='1';
begin
  while 1 = 1 loop
    select count(1) into v_num from hr.t_test where id=v_id;
  end loop;
end;
/

update hr.t_test set name='test1' where id='1';

```



### 4.1.1、创建表空间

pttbs 非加密表空间，用于存放普通表，TDE列加密表

```
create   tablespace pttbs  datafile size 2m  autoextend on  next 50m ;
```

tdetbs TDE表空间

```
create   tablespace tdetbs  datafile size 2m  autoextend on  next 50m maxsize 20480m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
```



### 4.1.2、准备测试表、测试数据

TDE表空间加密表

```
create table hr.tdetab (id number,c varchar2(100),e varchar2(50)) tablespace tdetbs;
insert into hr.tdetab select rownum,'测试数据index'||rownum,'天气真好index'||rownum from dual connect by rownum<=100000;
create index hr.idx_tdetab_tbs_id on hr.tdetab(id);
```

普通表空间，列加密

```
create table hr.tdetab_col (
id number ENCRYPT USING 'AES256' NO SALT,
c varchar2(100) ENCRYPT USING 'AES256' NO SALT,
e varchar2(50) ENCRYPT USING 'AES256' NO SALT
) tablespace pttbs;
insert into hr.tdetab_col select rownum,'测试数据index'||rownum,'天气真好index'||rownum from dual connect by rownum<=100000;
create index hr.idx_tdetab_id on hr.tdetab_col(id);
```

非加密表

```
create table hr.tab1 (id number,c varchar2(100),e varchar2(50)) tablespace pttbs;
insert into hr.tab1 select rownum,'测试数据index'||rownum,'天气真好index'||rownum from dual connect by rownum<=100000;
create index hr.idx_tab1_id on hr.tab1(id);
```



## 4.2、测试对执行计划的影响



```sql
hr@(604_CBTDEPDB)> explain plan for select * from hr.tdetab_col where id between 20 and 30;

Explained.

hr@(604_CBTDEPDB)> select * from table(dbms_xplan.display);

PLAN_TABLE_OUTPUT
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Plan hash value: 666432512

--------------------------------------------------------------------------------
| Id  | Operation	  | Name       | Rows  | Bytes | Cost (%CPU)| Time     |
--------------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |	       |    28 |  5432 |   684	 (1)| 00:00:01 |
|*  1 |  TABLE ACCESS FULL| TDETAB_COL |    28 |  5432 |   684	 (1)| 00:00:01 |
--------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   1 - filter(INTERNAL_FUNCTION("ID")>=20 AND
	      INTERNAL_FUNCTION("ID")<=30)

Note
-----
   - dynamic statistics used: dynamic sampling (level=2)

18 rows selected.
```

 

```
hr@(604_CBTDEPDB)>  explain plan for select * from hr.tdetab where id between 20 and 30;

Explained.

hr@(604_CBTDEPDB)>  select * from table(dbms_xplan.display);

PLAN_TABLE_OUTPUT
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Plan hash value: 1987618856

---------------------------------------------------------------------------------------------------------
| Id  | Operation			    | Name		| Rows	| Bytes | Cost (%CPU)| Time	|
---------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT		    |			|    11 |  1012 |     3   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID BATCHED| TDETAB		|    11 |  1012 |     3   (0)| 00:00:01 |
|*  2 |   INDEX RANGE SCAN		    | IDX_TDETAB_TBS_ID |    11 |	|     2   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("ID">=20 AND "ID"<=30)

Note
-----
   - dynamic statistics used: dynamic sampling (level=2)

18 rows selected.

hr@(604_CBTDEPDB)>
```

 

## 4.3、加密表作为被驱动表可以使用索引

```
hr@(604_CBTDEPDB)>  explain plan for select * from hr.tab1 a,hr.tdetab_col b where a.id=b.id and a.c='test2000';

Explained.
hr@(604_CBTDEPDB)>  select * from table(dbms_xplan.display);

PLAN_TABLE_OUTPUT
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Plan hash value: 2927452657

----------------------------------------------------------------------------------------------
| Id  | Operation		     | Name	     | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT	     |		     |	  17 |	4862 |	 461   (1)| 00:00:01 |
|   1 |  NESTED LOOPS		     |		     |	  17 |	4862 |	 461   (1)| 00:00:01 |
|   2 |   NESTED LOOPS		     |		     |	  17 |	4862 |	 461   (1)| 00:00:01 |
|*  3 |    TABLE ACCESS FULL	     | TAB1	     |	  17 |	1564 |	 410   (1)| 00:00:01 |
|*  4 |    INDEX RANGE SCAN	     | IDX_TDETAB_ID |	   1 |	     |	   1   (0)| 00:00:01 |
|   5 |   TABLE ACCESS BY INDEX ROWID| TDETAB_COL    |	   1 |	 194 |	   3   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   3 - filter("A"."C"='test2000')
   4 - access("B"."ID"=INTERNAL_FUNCTION("A"."ID"))

Note
-----
   - dynamic statistics used: dynamic sampling (level=2)
   - this is an adaptive plan

23 rows selected.

hr@(604_CBTDEPDB)>
```



```
 hr@(604_CBTDEPDB)>  explain plan for select * from hr.tab1 a,hr.tdetab_col b where a.id=b.id and b.c='test2000';

Explained.

hr@(604_CBTDEPDB)>  select * from table(dbms_xplan.display);

PLAN_TABLE_OUTPUT
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Plan hash value: 2930059104

---------------------------------------------------------------------------------
| Id  | Operation	   | Name	| Rows	| Bytes | Cost (%CPU)| Time	|
---------------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |		|    25 |  7150 |  1093   (1)| 00:00:01 |
|*  1 |  HASH JOIN	   |		|    25 |  7150 |  1093   (1)| 00:00:01 |
|*  2 |   TABLE ACCESS FULL| TDETAB_COL |    28 |  5432 |   684   (1)| 00:00:01 |
|   3 |   TABLE ACCESS FULL| TAB1	| 85991 |  7725K|   410   (1)| 00:00:01 |
---------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   1 - access("B"."ID"=INTERNAL_FUNCTION("A"."ID"))
   2 - filter("B"."C"='test2000')

Note
-----
   - dynamic statistics used: dynamic sampling (level=2)

20 rows selected.

hr@(604_CBTDEPDB)>


```

SALT列加密创建索引时，索引中存的是密文，因使用SALT属性的加密列对于相同的明文，每次加密后的密文是不同的，所以如果使用SALT数据的加密列无法创建索引，报错如下
ORA-28338: Column(s) cannot be both indexed and encrypted with salt







# 五、TDE 与 ADG 测试

说明：采用rman duplicate 进行搭建ADG，与常规的ADG相同，但是需要在 duplicate 之前，配置好tde，并将钱包打开。

ADG 建议配置自动登录钱包。

```
ADMINISTER KEY MANAGEMENT CREATE  AUTO_LOGIN KEYSTORE FROM KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY pwd200;
```

搭建DG，需要将源库的密钥复制到目标库，配置 sqlnet.ora ，然后启用Software Keystore，即可，以下是具体步骤。

## 5.1、DG 钱包配置准备

### 5.1.1、复制源库的密钥到目标库的tde 钱包路径

```
cd /etc/ORACLE/WALLETS/tdecdb/
scp ewallet.p12 oracle@目标ip:/etc/ORACLE/WALLETS/tdecdb
```

### 5.1.2、DG 配置 sqlnet.ora 

编辑 $ORACLE_HOME/network/admin/sqlnet.ora 新增 ENCRYPTION_WALLET_LOCATION 配置

```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```

### 5.1.3、DG 数据库启动到 mount 状态

```
startup nomount
```

### 5.1.4、DG 启用 Software Keystore

```
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
```

## 5.2、rman dumpicate 搭建 DG

### 5.2.1、主库配置归档模式

主库进行如下操作进行force logging和归档配置

#### a.主库配置 force logging 模式

```shell
ALTER DATABASE FORCE LOGGING;
```

#### c.主库配置归档模式

```shell
shutdown immediate
startup mount
alter database archivelog;
alter database open;
```

#### c.检查归档模式

```shell
SQL> select LOG_MODE, FORCE_LOGGING from v$database;

LOG_MODE     FOR
------------ ---
ARCHIVELOG   YES

SQL>
```

### 5.2.2、配置静态监听及tns

DG 上配置静态配置 /data/app/oracle/product/11.2.0.4/dbhome_1

```shell
vi  /data/app/oracle/product/11.2.0.4/dbhome_1/network/admin/listener.ora
新增
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = orcl)
      (ORACLE_HOME =  /data/app/oracle/product/11.2.0.4/dbhome_1)
      (SID_NAME = orcl)
    )
  )

```

#### a.重启监听

```shell
lsnrctl stop
lsnrctl start
```

#### b.配置 tns

主从都修改 tnsnames.ora，新增如下信息

cd  /data/app/oracle/product/11.2.0.4/dbhome_1/network/admin

vi tnsnames.ora

```shell
orcldg1 =
  (description =
    (address = (protocol = tcp)(host = 8.8.31.170)(port = 1521))
    (connect_data =
      (server = dedicated)
      (service_name = orcl)
    )
  )

orcl =
  (description =
    (address = (protocol = tcp)(host = 8.8.31.70)(port = 1521))
    (connect_data =
      (server = dedicated)
      (service_name = orcl)
    )
  )

```

#### c.测试 tns 

```shell
[oracle@ecsdbdg1 admin]$ tnsping orcl

TNS Ping Utility for Linux: Version 11.2.0.1.0 - Production on 04-MAR-2022 22:17:43

Copyright (c) 1997, 2009, Oracle.  All rights reserved.

Used parameter files:
 /data/app/oracle/product/11.2.0.4/dbhome_1/network/admin/sqlnet.ora


Used TNSNAMES adapter to resolve the alias
Attempting to contact (description = (address = (protocol = tcp)(host = 8.8.31.70)(port = 1521)) (connect_data = (server = dedicated) (service_name = orcl)))
OK (20 msec)
[oracle@ecsdbdg1 admin]$ tnsping orcldg1

TNS Ping Utility for Linux: Version 11.2.0.1.0 - Production on 04-MAR-2022 22:17:51

Copyright (c) 1997, 2009, Oracle.  All rights reserved.

Used parameter files:
 /data/app/oracle/product/11.2.0.4/dbhome_1/network/admin/sqlnet.ora


Used TNSNAMES adapter to resolve the alias
Attempting to contact (description = (address = (protocol = tcp)(host = 8.8.31.170)(port = 1521)) (connect_data = (server = dedicated) (service_name = orcl)))
OK (0 msec)
[oracle@ecsdbdg1 admin]$

```

### 5.2.3、新增 redo log file 和 standby log file

主库上进行 redo log file 和 standby log file 操作

```sql
 alter database add logfile group 4'/data/oracle/app/oradata/orcl/redo04.log' size 600m;
 alter database add logfile group 5'/data/oracle/app/oradata/orcl/redo05.log' size 600m;
 alter database add logfile group 6'/data/oracle/app/oradata/orcl/redo06.log' size 600m;
 /data/oracle/oradata/
alter database add standby logfile thread 1 group 11 ('/data/oracle/oradata/sty_redo01.log') size 629145600;
alter database add standby logfile thread 1 group 12 ('/data/oracle/oradata/sty_redo02.log') size 629145600;
alter database add standby logfile thread 1 group 13 ('/data/oracle/oradata/sty_redo03.log') size 629145600;
alter database add standby logfile thread 1 group 14 ('/data/oracle/oradata/sty_redo04.log') size 629145600;
alter database add standby logfile thread 1 group 15 ('/data/oracle/oradata/sty_redo05.log') size 629145600;
alter database add standby logfile thread 1 group 16 ('/data/oracle/oradata/sty_redo06.log') size 629145600;
```

### 5.2.4、主库修改数据库参数

```sql
alter system set log_archive_config='dg_config=(orcl,orcldg1)';
alter system set log_archive_dest_1='location=/data/oracle/arch valid_for=(all_logfiles,all_roles) db_unique_name=orcl';
alter system set log_archive_dest_2='SERVICE=orcldg1  lgwr async valid_for=(ONLINE_LOGFILES,primary_role) db_unique_name=orcldg1';
alter system set standby_file_management='AUTO';
alter system set fal_server=orcldg1;
alter system set log_archive_dest_state_2=enable;
```

### 5.2.5、复制密码文件到备库

```shell
cd $ORACLE_HOME/dbs
scp scp orapworcl 8.8.31.170:/data/app/oracle/product/11.2.0.4/dbhome_1/
```

### 5.2.6、备库启动数据库实例

#### a.备库创建审计日志目录

```shell
mkdir -p  /data/app/oracle/admin/orcl/adump
```

#### b.备库启动数据库实例到 nomount 状态

```shell
cd $ORACLE_HOME/dbs
touch initorcl.ora
echo "db_name=orcl">initorcl.ora
```

###  5.2.7、rman duplicate 搭建DG从库

rman 登录数据库

oracle 用户执行 rman 命令，然后使用如下脚本进行DG搭建。

```shell
connect target sys/Password123@orcl 
connect auxiliary sys/Password123@orcldg1 
 run {
    allocate channel c1 type disk;
    allocate channel c2 type disk;
    allocate channel c3 type disk;
    allocate channel c4 type disk;
    allocate auxiliary channel s1 type disk;
    allocate auxiliary channel s2 type disk;
    allocate auxiliary channel s3 type disk;
    allocate auxiliary channel s4 type disk;
    duplicate target database
         for standby
         from active database nofilenamecheck
         dorecover
         spfile
         parameter_value_convert 'orcl','orcl'
         set db_unique_name='orcldg1'
         set cluster_database='false'
         set fal_server='orcl'
         set remote_listener=''
         set local_listener=''
         set standby_file_management='AUTO'
         set log_archive_config='dg_config=(orcl,orcldg1)'
         set log_archive_dest_1='location=/data/oracle/arch valid_for=(all_logfiles,all_roles) db_unique_name=orcldg1'
         set log_archive_dest_2='SERVICE=orcl lgwr async VALID_FOr=(ONLINE_LOGFILES,primary_role) DB_UNIQUE_NAME=orcl'
         set log_archive_dest_state_2='enable'
        ;
       sql channel c1 "alter system archive log current";
       sql channel s1 "alter database open";
       sql channel s1 "alter database recover managed standby database using current logfile  disconnect";
     }
```



#### a.查看DG数据同步详情

查看脚本为 /home/oracle/dginfo.sh，内容如下

```shell
#!/usr/bin/env sh
sqlplus / as sysdba <<EOF
set lines 123
set pages 200
col  CTIME format a20
col  NAME format a20
col  VALUE format a20
col  DATUM_TIME format a20
show parameter service_name
select open_mode, DATABASE_ROLE from v\$database;
SELECT  TO_NUMBER( SUBSTR ( (SUBSTR (VALUE, 5)), 0, 2) * 3600 + SUBSTR ( (SUBSTR (VALUE, 5)), 4, 2) * 60 + SUBSTR ( (SUBSTR (VALUE, 5)), 7, 2)) dgbehind, TO_CHAR (SYSDATE, 'yyyymmdd hh24:mi:ss'
) CTIME, NAME, VALUE,DATUM_TIME FROM V\$DATAGUARD_STATS WHERE NAME ='apply lag';
select process,block#,blocks ,status ,sequence# from v\$managed_standby;
exit
EOF
```

#### b.dginfo.sh使用方法：

使用 oracle 用户 执行 sh dginfo.sh


![image-20220307164108078](http://pic.liups.com/images/2022/03/07/202203071641123.png)



# 六、TDE exp/expdp 及 rman 测试

## 6.1、TDE exp/imp 测试

### 6.1.1、exp 导出数据测试

#### a.钱包打开的情况下测试exp

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

#### b.钱包关闭情况下exp导出

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

#### c.总结

无论钱包是否开启，可以看到exp 都无法导出 加密的表。

### 6.1.2、TDE imp 导入数据测试

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

### 6.1.3、TDE imp 导入数据总结

imp 可以正常imp导出的数据正常导入。

### 6.1.4、expdp 导出数据测试

#### a.钱包打开的情况 expdp 测试

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

#### b.钱包关闭的情况 expdp 测试

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

#### c.总结

钱包关闭的情况，加密数据无法正常导出，其他数据可以正常导出。钱包打开的情况所有数据都可以正常导出。

### TDE impdp 导入数据测试

#### a.impdp 在钱包关闭的情况下导入

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

钱包关闭的情况加密数据及加密表空间里的表及数据无法正常导入。

#### b.impdp 在钱包打开的情况下导入

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



#### c.总结

在钱包关闭的情况下，exp和expdp 都无法导出加密数据，其他非加密表可以导出。

在钱包打开的情况下，exp 不支持导出 TDE加密数据，包括加密表空间下的所有数据，及加密列的表都无法导出。imp 可以正常到exp 导出的数据导入到数据库。

在钱包打开的情况下，expdp 支持导出 TDE加密数据，包括加密表空间下的所有数据，及加密列的表都可以正常导出。impdp 也可以把expdp 导出的数据完整的导入。



## 6.2、TDE rman  测试

### 6.2.1、rman 不压缩备份测试

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

### 总结

不压缩备份的时候，钱包关闭打开的情况都可以备份。

### 6.2.2、rman 压缩备份测试

#### a.钱包关闭 rman 压缩测试

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

#### b.钱包打开 rman 压缩测试

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



### rman 恢复

#### 1、目标库启用 tde

使用root 创建单独的目录，权限授予给 oracle 用户。

```shell
mkdir -p /etc/ORACLE/WALLETS/tdecdb
chown -R oracle:oinstall /etc/ORACLE/WALLETS/tdecdb
```

##### 1.1、编辑 sqlnet.ora

编辑 $ORACLE_HOME/network/admin/sqlnet.ora 新增 ENCRYPTION_WALLET_LOCATION 配置

```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```

##### 1.2、将源库的key 复制到目标库

```
cd /etc/ORACLE/WALLETS/tdecdb/
scp ewallet.p12 oracle@目标ip:/etc/ORACLE/WALLETS/tdecdb
```

##### 1.3、启用  Software Keystore

启动数据库到nomount状态，然后启用   Software Keystore

```
SQL> ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
keystore altered.
```

#### 2、复制备份文件到目标环境

```
scp *.open  oracle@101.33.229.100:/oradata/tdecdb/backup/0318/
scp  /u01/app/oracle/fast_recovery_area/TDECDB/autobackup/2022_03_18/o1_mf_s_1099646971_k37r3vtd_.bkp oracle@101.33.229.100:/oradata/tdecdb/backup/0318/
```

#### 3、恢复参数文件

```
RMAN>  restore spfile from '/u01/app/oracle/fast_recovery_area/TDECDB/autobackup/2022_03_18/o1_mf_s_1099646971_k37r3vtd_.bkp';

Starting restore at 2022-03-19 22:01:08
using channel ORA_DISK_1

channel ORA_DISK_1: restoring spfile from AUTOBACKUP /u01/app/oracle/fast_recovery_area/TDECDB/autobackup/2022_03_18/o1_mf_s_1099646971_k37r3vtd_.bkp
channel ORA_DISK_1: SPFILE restore from AUTOBACKUP complete
Finished restore at 2022-03-19 22:01:10

RMAN> exit
```

#### 4、恢复控制文件

```
RMAN> RESTORE CONTROLFILE FROM '/u01/app/oracle/fast_recovery_area/TDECDB/autobackup/2022_03_18/o1_mf_s_1099646971_k37r3vtd_.bkp';

Starting restore at 2022-03-19 22:04:49
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=398 device type=DISK

channel ORA_DISK_1: restoring control file
channel ORA_DISK_1: restore complete, elapsed time: 00:00:01
output file name=/oradata/tdecdb/sys/control01.ctl
output file name=/oradata/tdecdb/redo/control02.ctl
Finished restore at 2022-03-19 22:04:51
```

#### 5、启动数据库到mount 状态

```
alter database mout
```

#### 6、restore database;

```
RMAN> restore database;
```

#### 7、recover database

```
RMAN> recover database
```



### tde rman 恢复总结

rman 恢复的时候，需要将源库的密钥复制奥目标库，配置 sqlnet.ora ，然后启用Software Keystore，即可。

0、复制源库的密钥到目标库的tde 钱包路径

```
cd /etc/ORACLE/WALLETS/tdecdb/
scp ewallet.p12 oracle@目标ip:/etc/ORACLE/WALLETS/tdecdb
```

1、配置 sqlnet.ora 

编辑 $ORACLE_HOME/network/admin/sqlnet.ora 新增 ENCRYPTION_WALLET_LOCATION 配置

```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```

2、启用 Software Keystore

```
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
```

剩下的操作跟常规rman 恢复相同。





# 七、TDE 与 RAC 测试

> **How to configure TDE in pluggable database in 12c for standalone and RAC environment (Doc ID 2107821.1)**

 This document details step by step instructions to configure TDE in 12c pluggable database for standalone and RAC environment.



## 7.1、 创建密钥目录并编辑 sqlnet.ora 



```
ENCRYPTION_WALLET_LOCATION =
      (SOURCE = (METHOD = FILE)
          (METHOD_DATA =
               (DIRECTORY = /cdbrdbms/etc/$ORACLE_SID)
          )
      )
```

## 7.2、创建 Key store 

Create Key store on CDB database and generate encryption key for CDB

```
ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/cdbrdbms/etc/MTc12c1' IDENTIFIED BY "welcome1";
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY "welcome1";
ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY "welcome1" WITH BACKUP;
```

## 7.3、检查 CDB 钱包状态

Verify the wallet has been opened in CDB database

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

## 7.4、打开 PDB

Open the respective PDB and set the PDB as current database 

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

## 7.5、在 PDB 上打开钱包

Open the keystore in that PDB and generate encryption key for the PDB

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

## 7.6、创建加密表空间

Create encrypted tablespace

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



## 7.7、RAC环境总结

 RAC环境下需要注意以下

1、确保 RAC 的每个节点的 sqlnet.ora 都被正确修改

Make sure encryption_wallet_location parameter is configured in sqlnet.ora file of all other RAC nodes

```
ENCRYPTION_WALLET_LOCATION =
         (SOURCE = (METHOD = FILE)
              (METHOD_DATA =
                       (DIRECTORY = /cdbrdbms/etc/$ORACLE_SID)
                )
          )
```

2、将加密钱包文件存放到共享磁盘比如ACFS或者ASM磁盘。

7.2 Copy the wallet file ewallet.p12 from first RAC node to all other RAC nodes ENCRYPTION_WALLET_LOCATION directory.

Important Note: It is recommended to use shared file location like ACFS or ASM location to hold the TDE wallet file in case of RAC DB.

# 八、TDE 与 OGG 测试

> **Step by Step Guide to Configure GoldenGate Extract in Classic Mode to capture TDE in 11.1.1.1 and up (Doc ID 1451327.1)**

**1. Apply database 10395645 for oracle 10.2.0.5 or 11.2.0.2**
oracle 11.2.0.3 patchset includes this patch.

**2. Run prvtclkm.plb, and grant privilege to gguser**
(missing this step may cause extract error:  PLS-00201: identifier 'SYS.DBMS_INTERNAL_CLKM' must be declared)

```sql
- cd /app/goldengate/source  (go to OGG installation home)
- login as sysdba
- @prvtclkm.plb
SQL> @prvtclkm.plb
Package created.
Library created.
Package body created.
 (The dbms_internal_cklm.plb mentioned in installation guide is not correct. it should be prvtclkm.plb, and more details are in Note 1357929.1)
- grant execute on sys.dbms_internal_clkm to gguser;
```


**3. Configure/Create wallet**
(1) configure sqlnet.ora

编辑 $ORACLE_HOME/network/admin/sqlnet.ora 新增 ENCRYPTION_WALLET_LOCATION 配置

```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```

## 

(2) create wallet with mkstore, as oracle user

```
cd /app/oracle/product/wallet
[oracle@localhost wallet]$ mkstore -wrl . -create
Oracle Secret Store Tool : Version 11.2.0.3.0 - Production
Copyright (c) 2004, 2011, Oracle and/or its affiliates. All rights reserved.
Enter password:  welcome1
Enter password again: welcome1
[oracle@localhost wallet]$ ls -l
total 8
-rw-------. 1 oracle oinstall 3589 Apr 19 12:56 cwallet.sso
-rw-------. 1 oracle oinstall 3512 Apr 19 12:56 ewallet.p12
```

(3) set encryption key and open the wallet

```
SQL> alter system set encryption key identified by "welcome1";  
(the password is same as the one when the wallet was created, and is case-sensitive. i.e., welcome1 will not work, while "welcome1" works.)

[oracle@localhost wallet]$ ls -l
total 16
-rw-------. 1 oracle oinstall 4298 Apr 19 13:05 cwallet.sso
-rw-------. 1 oracle oinstall 4221 Apr 19 13:05 ewallet.p12
```



```
[oracle@localhost wallet]$ mkstore -wrl . -list
Oracle Secret Store Tool : Version 11.2.0.3.0 - Production
Copyright (c) 2004, 2011, Oracle and/or its affiliates. All rights reserved.
Enter wallet password:
Oracle Secret Store entries:
ORACLE.SECURITY.DB.ENCRYPTION.AQPzY6F6rE++v2GDWhRfl58AAAAAAAAAAAAAAAAAAAAAAAAAAAAA
ORACLE.SECURITY.DB.ENCRYPTION.MASTERKEY
ORACLE.SECURITY.TS.ENCRYPTION.BYxe8gX+JaWda5meFCZfAx4CAwAAAAAAAAAAAAAAAAAAAAAAAAAA
```

(4) create entry for ORACLEGG

```
[oracle@localhost wallet]$ mkstore -wrl . -createEntry ORACLE.SECURITY.CL.ENCRYPTION.ORACLEGG
Oracle Secret Store Tool : Version 11.2.0.3.0 - Production
Copyright (c) 2004, 2011, Oracle and/or its affiliates. All rights reserved.
Your secret/Password is missing in the command line
Enter your secret/Password: welcome2
Re-enter your secret/Password: welcome2
Enter wallet password: welcome1

[oracle@localhost wallet]$ mkstore -wrl . -list
Oracle Secret Store Tool : Version 11.2.0.3.0 - Production
Copyright (c) 2004, 2011, Oracle and/or its affiliates. All rights reserved.
Enter wallet password: welcome1
Oracle Secret Store entries:
ORACLE.SECURITY.CL.ENCRYPTION.ORACLEGG
ORACLE.SECURITY.DB.ENCRYPTION.AQPzY6F6rE++v2GDWhRfl58AAAAAAAAAAAAAAAAAAAAAAAAAAAAA
ORACLE.SECURITY.DB.ENCRYPTION.MASTERKEY
ORACLE.SECURITY.TS.ENCRYPTION.BYxe8gX+JaWda5meFCZfAx4CAwAAAAAAAAAAAAAAAAAAAAAAAAAA
```

(5) log in as sysdba, close and re-open the wallet, and switch logfile

```
SQL> alter system set encryption wallet close identified by "welcome1";
System altered.
SQL> alter system set encryption wallet open identified by "welcome1";
System altered.
SQL> alter system switch logfile;   ----- repeate this steps to recycle all the redo groups.
System altered.

if wallet is created with autologin, restarting may hit error that can be ignored.

extract should be positioned after this point.


(without doing this step, the extract may fail with ORA-28360 error. if ora-28360 still happens after wallet open/close, the database may need to be restarted.)

For RAC database, copy the wallet to other nodes also as following:

\- close wallet from all the instances.

\- copy the wallet from above instance where it was created, to all othet instances.

\- re-open the wallet from all the instances.
```

**4. generate the encrypted password for ORACLEGG (welcome2 in this example)** 

```
GGSCI (localhost.localdomain) 3> ENCRYPT PASSWORD welcome2 BLOWFISH ENCRYPTKEY DEFAULT
Using default key...
Encrypted password: AACAAAAAAAAAAAIALFYGAHVENIMJLFPH
Algorithm used: BLOWFISH
```

**5. create extract**

```
add the encrypted password to "DBOPTIONS DECRYPTPASSWORD" in extract parameter file

extract e1
userid gguser, password gguser
DBOPTIONS DECRYPTPASSWORD AACAAAAAAAAAAAIALFYGAHVENIMJLFPH ENCRYPTKEY DEFAULT
exttrail ./dirdat/ef
table ABC.*;



GGSCI (localhost.localdomain) 1> add extract e1, tranlog, begin now
EXTRACT added.
GGSCI (localhost.localdomain) 2> add exttrail ./dirdat/ef, extract e1, megabytes 200
EXTTRAIL added.
GGSCI (localhost.localdomain) 3> start E1


```

**6. test a TDE transaction**

```
(1) Create table with TDE column and insert a row.
login as abc/abc
SQL> create table s1 (a number primary key, b varchar2(10) encrypt);
Table created.

SQL> insert into s1 values (1,'abc');
1 row created.
SQL> commit;
Commit complete.

(2) check trail record captured by extract
from the trail with logdump:
2012/04/19 13:40:04.000.000 Insert        Len  20 RBA 1071
Name: ABC.S1
After Image:                       Partition 4  G s
 0000 0005 0000 0001 3100 0100 0700 0000 0361 6263 | ........1........abc
Column   0 (x0000), Len   5 (x0005)
 0000 0001 31                   | ....1
Column   1 (x0001), Len   7 (x0007)
 0000 0003 6162 63                 | ....abc
```

 
