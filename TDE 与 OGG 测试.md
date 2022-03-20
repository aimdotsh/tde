TDE 与 OGG 测试

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

 