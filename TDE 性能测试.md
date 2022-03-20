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



## 创建表空间

pttbs 非加密表空间，用于存放普通表，TDE列加密表

```
create   tablespace pttbs  datafile size 2m  autoextend on  next 50m ;
```

tdetbs TDE表空间

```
create   tablespace tdetbs  datafile size 2m  autoextend on  next 50m maxsize 20480m   ENCRYPTION USING 'AES256' DEFAULT STORAGE(ENCRYPT);
```



## 准备测试表、测试数据

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



## 测试对执行计划的影响



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

 

加密表作为被驱动表可以使用索引

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

