# 实验2：用户管理 - 掌握管理角色、权根、用户的能力，并在用户之间共享对象。 

## 实验内容： ##
  #### Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求： ####
  1. 在pdborcl插接式数据中创建一个新的本地角色con_res_view，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，这样任何拥有con_res_view的用户就同时拥有这三种权限。
  2. 创建角色之后，再创建用户new_user，给用户分配表空间，设置限额为50M，授予con_res_view角色。
  3. 最后测试：用新用户new_user连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。
  
## 实验步骤（Git Bash实现） ##
- 第一步：以system登录到pdborcl，创建角色con_res_view和用户new_user，并授权和分配空间：
  ``` 
  SQL
  [oracle@deep02 ~]$ sqlplus system/123@pdborcl

  SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 14:32:13 2018

  Copyright (c) 1982, 2014, Oracle.  All rights reserved.

  上次成功登录时间: 星期三 10月 24 2018 14:21:42 +08:00

  连接到:
  Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
  With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

  SQL> CREATE ROLE zz;

  角色已创建。

  SQL> GRANT connect,resource,CREATE VIEW TO zz;

  授权成功。

  SQL> CREATE USER cc IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;

  用户已创建。

  SQL> ALTER USER cc QUOTA 50M ON users;

  用户已更改。

  SQL> GRANT zz TO cc;

  授权成功。
  ```
  

- 第二步：新用户new_user连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
``` SQL
  [oracle@deep02 ~]$ sqlplus cc/123@pdborcl

  SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 14:35:49 2018

  Copyright (c) 1982, 2014, Oracle.  All rights reserved.


  连接到:
  Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
  With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

  SQL> show user;
  USER 为 "CC"
  SQL> CREATE TABLE mytable (id number,name varchar(50));

  表已创建。

  SQL> INSERT INTO mytable(id,name)VALUES(1,'zhc');

  已创建 1 行。

  SQL> INSERT INTO mytable(id,name)VALUES(2,'cc');

  已创建 1 行。

  SQL> INSERT INTO mytable(id,name)VALUES(3,'hcc');

  已创建 1 行。

  SQL>  CREATE VIEW myview AS SELECT name FROM mytable;

  视图已创建。

  SQL> SELECT * FROM myview;

  NAME
  --------------------------------------------------------------------------------
  zhc
  cc
  hcc

  SQL> GRANT SELECT ON myview TO hr;

  授权成功。
  ```
  
- 第三步：用户hr连接到pdborcl，查询new_user授予它的视图myview
``` SQL
  [oracle@deep02 ~]$ sqlplus hr/123@pdborcl

  SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 14:40:03 2018

  Copyright (c) 1982, 2014, Oracle.  All rights reserved.

  上次成功登录时间: 星期三 10月 24 2018 14:23:29 +08:00

  连接到:
  Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
  With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

  SQL> SELECT * FROM cc.myview;

  NAME
  --------------------------------------------------------------------------------
  zhc
  cc
  hcc
```


  
