## 版本：12.1.0.2.0 测试表空间加密

### 测试 undo 表空间加密

```
SQL> create undo tablespace test1_undo datafile   size 20m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
create undo tablespace test1_undo datafile   size 20m	ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT)
                                                        *
ERROR at line 1:
ORA-30024: Invalid specification for CREATE UNDO TABLESPACE


SQL>
```

### 测试 temp 表空间

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

### 测试普通表空间加密

```
SQL> create   tablespace put_tbs  datafile size 2m  autoextend on  next 50m maxsize 20480m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);

Tablespace created.

SQL>
```

### 测试big file表空间加密

```
SQL> CREATE bigfile TABLESPACE bigf_tbs2e DATAFILE SIZE 150M ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);

Tablespace created.

SQL>
```

