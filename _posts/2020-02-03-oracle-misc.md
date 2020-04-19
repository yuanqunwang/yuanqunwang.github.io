**ORA-65096**: invalid common user or role name

**Cause:** An attempt was made to create a common user or role with a name that was not valid for common users or roles. In addition to the usual rules for user and role names, common user and role names must start with C## or c## and consist only of ASCII characters.

**Action:** Specify a valid common user or role name.

`SQL> alter session set "_ORACLE_SCRIPT"=true;`



* docker 数据库帐号密码：

  ```shell
  SQL> sqlplus sys/Oradoc_db1 as sysdba
  SQL> sqlplus ot/ot; 
  ```

  

### rownum

```shell
SQL> select ..., rownum
     from t
     where <where clause>
     group by <grouping clause>
     having <having clause>
     order by <order clause>
```

上述sql的执行顺序：

1. from/where 子句
2. 对每个符合where条件的一行数据赋值rownum，同时rownum增1
3. group by 子句
4. having 子句
5. order by 子句
6. select 子句



### 常用的数据字典

数据字典的组合规则为：[范围]_[类型]， 比如：user_users, user_tables等

### 范围

* user
* all
* dba

### 类型

* users
* tables
* views
* indexes
* procedures

### 外键的限制

关系p(x)与关系c(y, x)，其中关系c的x属性是关系p的选属性的外键，当对关系p的x键做更新操作或删除关系p中的某一行时，在没有为c(x)创建索引时，会锁定整个c关系。

### 查找锁对象，并kill对应的session

```sql
  SELECT oracle_username || ' (' || s.osuser || ')' username,
         s.sid || ',' || s.serial# sess_id,
         owner || '.' || object_name object,
         object_type,
         DECODE (l.block, 0, 'Not Blocking', 1, 'Blocking', 2, 'Global') STATUS,
         DECODE (v.locked_mode,
                 0, 'None',
                 1, 'Null',
                 2, 'Row-S (SS)',
                 3, 'Row-X (SX)',
                 4, 'Share',
                 5, 'S/Row-X (SSX)',
                 6, 'Exclusive',
                 TO_CHAR (lmode))
            mode_held
    FROM v$locked_object v,
         dba_objects d,
         v$lock l,
         v$session s
   WHERE     v.object_id = d.object_id
         AND v.object_id = l.id1
         AND v.session_id = s.sid
ORDER BY oracle_username, session_id;
```

或

```sql
SELECT c.owner,
  c.object_name,
  c.object_type,
  b.sid,
  b.serial#,
  b.STATUS,
  b.osuser,
  b.machine
FROM v$locked_object a ,
  v$session b,
  dba_objects c
WHERE b.sid     = a.session_id
AND a.object_id = c.object_id;
```

kill对应的session：

```shell
SQL> ALTER SYSTEM KILL SESSION '579, 703' IMMEDIATE;
```











