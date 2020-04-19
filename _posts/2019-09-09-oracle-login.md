---
title: Oracle Login Notes
tags: [Oracle, Tips, Tools]
---
## Oracle 默认用户

* 超级管理员：sys/change_on_install
* 普通管理员：system/manager
* 普通用户：scott/tiger
* 大数据用户：sh/sh

## sqlplus连接数据库

语法：

```shell
sqlplus [ [<option>] [{logon | /nolog}] [<start>]]
<logon> is: {<username>[/password][@<connect_identifier>] | /}
            [as {sysdba | sysoper | sysasm}]
```

常见用法：

以系统管理员的角色登录管理：

```shell
sqlplus / as sysdba
```

不登录：

```shell
sqlplus /nolog
SQL>connect username/password as sysdba@net_service_name
```

调用本地命令(ls)：

```shell
SQL>host ls
```





