---
title: Mysql Tips
tags: [Mysql, Tips]
---
#Mysql Tips

##Mysql管理

### Mac环境重置Mysql root密码

1. Stop the mysqld server.  Typically this can be done by from 'System Prefrences' > MySQL > 'Stop MySQL Server'

2.  Start the server in safe mode with privilege bypass

    From a terminal: 

   ```shell
   sudo /usr/local/mysql/bin/mysqld_safe --skip-grant-tables
   ```

3. In a new terminal window:

   method1:

   ```shell
   $sudo /usr/local/mysql/bin/mysql -u root
   mysql>FLUSH PRIVILEGES;
   mysql>ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
   ```

   mehod2:

   1. mysql 5.7之前

      ```shell
      UPDATE mysql.user SET Password = PASSWORD('password') WHERE User = 'root';
      ```

      mysql 5.7及之后版本

      ```shell
      UPDATE mysql.user SET authentication_string = PASSWORD('password') WHERE User = 'root';
      ```

   2. To make the change take effect, reload the stored user information with the following command:

      ```shell
      FLUSH PRIVILEGES;
      ```

4. Stop the mysqld server again and restart it in normal mode.

   ### 更改密码

   ```shell
   mysqladmin -u root -p'OLDPASSWORD' password NEWPASSWORD
   ```



### 远程连接mysql数据库

1. 设置绑定ip地址，该配置文件为`/etc/mysql/mysql.conf.d/mysqld.cnf`， 配置选项如下：

   ```shell
   bind-address		= 127.0.0.1 ( The default. )
   bind-address		= XXX.XXX.XXX.XXX ( The ip address of your Public Net interface. )
   bind-address		= ZZZ.ZZZ.ZZZ.ZZZ ( The ip address of your Service Net interface. )
   bind-address		= 0.0.0.0 ( All ip addresses. )
   ```

   

2. 给用户授予远程连接的权限

   ```shell
   grant all privileges on *.* to 'root'@'%' identified by 'root_passwd' with grant option;
   ```

   

   执行sql文件：

   1. 未登录到mysql，指定数据库名称：

      ```shell
      $ mysql db_name < sql_file
      ```

      未登录到mysql，sql_file 中有`use db_name`语句：

      ```shell
      $ mysql < sql_file
      ```

   2. 已登录到mysql

      ```shell
      mysql> source sql_file;
      ```

      或者：

      ```shell
      mysql> \. sql_file;
      ```

      

      

   

   

   

   


