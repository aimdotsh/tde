[TOC]

   * [TDE 与 ADG 测试](#tde-与-adg-测试)
      * [0、复制源库的密钥到目标库的tde 钱包路径](#0复制源库的密钥到目标库的tde-钱包路径)
      * [1、DG 配置 sqlnet.ora](#1dg-配置-sqlnetora)
      * [2、DG 数据库启动到 mount 状态](#2dg-数据库启动到-mount-状态)
      * [3、DG 启用 Software Keystore](#3dg-启用-software-keystore)
      * [4、rman dumpicate 搭建 DG](#4rman-dumpicate-搭建-dg)
         * [配置归档模式](#配置归档模式)
         * [配置 force logging 模式](#配置-force-logging-模式)
         * [配置归档模式](#配置归档模式-1)
         * [检查归档模式](#检查归档模式)
         * [配置静态监听及tns](#配置静态监听及tns)
         * [重启监听](#重启监听)
         * [配置 tns](#配置-tns)
         * [测试 tns](#测试-tns)
         * [新增 redo log file 和 standby log file](#新增-redo-log-file-和-standby-log-file)
         * [主库修改数据库参数](#主库修改数据库参数)
         * [复制密码文件到备库](#复制密码文件到备库)
         * [备库启动数据库实例](#备库启动数据库实例)
         * [rman duplicate 搭建DG从库](#rman-duplicate-搭建dg从库)
         * [查看DG数据同步详情](#查看dg数据同步详情)

## TDE 与 ADG 测试

说明：采用rman duplicate 进行搭建ADG，与常规的ADG相同，但是需要在 duplicate 之前，配置好tde，并将钱包打开。

ADG 建议配置自动登录钱包。

```
ADMINISTER KEY MANAGEMENT CREATE  AUTO_LOGIN KEYSTORE FROM KEYSTORE '/etc/ORACLE/WALLETS/tdecdb' IDENTIFIED BY pwd200;
```

搭建DG，需要将源库的密钥复制到目标库，配置 sqlnet.ora ，然后启用Software Keystore，即可，以下是具体步骤。

### 0、复制源库的密钥到目标库的tde 钱包路径

```
cd /etc/ORACLE/WALLETS/tdecdb/
scp ewallet.p12 oracle@目标ip:/etc/ORACLE/WALLETS/tdecdb
```

### 1、DG 配置 sqlnet.ora 

编辑 $ORACLE_HOME/network/admin/sqlnet.ora 新增 ENCRYPTION_WALLET_LOCATION 配置

```shell
vi /u01/app/oracle/product/12.1.0.2/dbhome_1/network/admin/sqlnet.ora

ENCRYPTION_WALLET_LOCATION=
  (SOURCE=
   (METHOD=FILE)
    (METHOD_DATA=
     (DIRECTORY=/etc/ORACLE/WALLETS/tdecdb)))
```

### 2、DG 数据库启动到 mount 状态

```
startup nomount
```

### 3、DG 启用 Software Keystore

```
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY Password23;
```

### 4、rman dumpicate 搭建 DG

#### 配置归档模式

主库进行如下操作进行force logging和归档配置

#### 配置 force logging 模式

```shell
ALTER DATABASE FORCE LOGGING;
```

#### 配置归档模式

```shell
shutdown immediate
startup mount
alter database archivelog;
alter database open;
```

#### 检查归档模式

```shell
SQL> select LOG_MODE, FORCE_LOGGING from v$database;

LOG_MODE     FOR
------------ ---
ARCHIVELOG   YES

SQL>
```

#### 配置静态监听及tns

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

#### 重启监听

```shell
lsnrctl stop
lsnrctl start
```

#### 配置 tns

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

#### 测试 tns 

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

#### 新增 redo log file 和 standby log file

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

#### 主库修改数据库参数

```sql
alter system set log_archive_config='dg_config=(orcl,orcldg1)';
alter system set log_archive_dest_1='location=/data/oracle/arch valid_for=(all_logfiles,all_roles) db_unique_name=orcl';
alter system set log_archive_dest_2='SERVICE=orcldg1  lgwr async valid_for=(ONLINE_LOGFILES,primary_role) db_unique_name=orcldg1';
alter system set standby_file_management='AUTO';
alter system set fal_server=orcldg1;
alter system set log_archive_dest_state_2=enable;
```

#### 复制密码文件到备库

```shell
cd $ORACLE_HOME/dbs
scp scp orapworcl 8.8.31.170:/data/app/oracle/product/11.2.0.4/dbhome_1/
```

#### 备库启动数据库实例

备库创建审计日志目录

 sqlplus sys/Password123@orcl as sysdba



```shell
mkdir -p  /data/app/oracle/admin/orcl/adump
```

备库启动数据库实例到 nomount 状态

```shell
cd $ORACLE_HOME/dbs
touch initorcl.ora
echo "db_name=orcl">initorcl.ora
```

####  rman duplicate 搭建DG从库

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



#### 查看DG数据同步详情

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

使用方法：

使用 oracle 用户 执行 sh dginfo.sh

![image-20220307164108078](http://pic.liups.com/images/2022/03/07/202203071641123.png)
