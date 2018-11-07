# 实验3：创建分区表

## 实验目的：

   掌握分区表的创建方法，掌握各种分区方式的使用场景。

## 实验内容

- 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
- 使用你自己的账号创建本实验的表，表创建在上述3个分区，自定义分区策略。
- 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
- 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
- 进行分区与不分区的对比实验。

## 实验步骤

- 第一步（利用System用户分配权限给CC用户）：

   查看cc用户权限
   
   ![界面1](https://github.com/z915287285/Oracle/blob/master/test3/1.png)
   
   给予权限(分配表空间)
   
   ![界面2](https://github.com/z915287285/Oracle/blob/master/test3/2.png)

- 第二步 (在三个表空间建分区表)：

  ``` SQL
  [oracle@deep02 ~]$ sqlplus cc/123@pdborcl

   SQL*Plus: Release 12.1.0.2.0 Production on 星期三 11月 7 14:17:57 2018

   Copyright (c) 1982, 2014, Oracle.  All rights reserved.

   上次成功登录时间: 星期三 10月 31 2018 15:02:14 +08:00

   连接到:
   Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
   With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

    CREATE TABLE orders
   (
        order_id NUMBER(10, 0) NOT NULL
   STORAGE (   BUFFER_POOL DEFAULT )
   NOCOMPRESS NOPARALLEL
   PARTITION BY RANGE (order_date)
   (
    PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (
    TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',
    'NLS_CALENDAR=GREGORIAN'))
    , customer_name VARCHAR2(40 BYTE) NOT NULL
    , customer_tel VARCHAR2(40 BYTE) NOT NULL
        , order_date DATE NOT NULL
    , employee_id NUMBER(6, 0) NOT NULL
    , discount NUMBER(8, 2) DEFAULT 0
        , trade_receivable NUMBER(8, 2) DEFAULT 0
   TABLESPACE USERS02
    PCTFREE 10
    INITRANS 1
   )
    STORAGE
   (
    INITIAL 8388608
   TABLESPACE USERS
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
   PCTFREE 10 INITRANS 1
   )
   NOCOMPRESS NO INMEMORY
   STORAGE (   BUFFER_POOL DEFAULT )
   , PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (
   TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',
   'NLS_CALENDAR=GREGORIAN'))
   NOLOGGING
   NOCOMPRESS NOPARALLEL
   PARTITION BY RANGE (order_date)
   (
    PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (
    TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',
    'NLS_CALENDAR=GREGORIAN'))
    NOLOGGING
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 1
    STORAGE
   (
    INITIAL 8388608
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
   )
   NOCOMPRESS NO INMEMORY
   , PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (
   TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',
   'NLS_CALENDAR=GREGORIAN'))
   NOLOGGING
   TABLESPACE USERS02
    PCTFREE 10
    INITRANS 1
    STORAGE
   (
    INITIAL 8388608
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
   )
   NOCOMPRESS NO INMEMORY
   , PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (
   TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS',
   'NLS_CALENDAR=GREGORIAN'))
   NOLOGGING
   TABLESPACE USERS03
   );





   表已创建。
   
   ALTER TABLE ORDERS
  2  ADD CONSTRAINT ORDERS_PK PRIMARY KEY
  3  (
  4    ORDER_ID
  5  )
  6  ENABLE;

   表已更改。

  ```
  包括主键设置
  
  
 - 第三步 (创建order_details)
 
 ``` SQL
      CREATE TABLE order_details
  2  (
id NUMBER(10, 0) NOT NULL
  4  , order_id NUMBER(10, 0) NOT NULL
, product_id VARCHAR2(40 BYTE) NOT NULL
  6  , product_num NUMBER(8, 2) NOT NULL
  7  , product_price NUMBER(8, 2) NOT NULL
  8  , CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)
REFERENCES orders  (  order_id   )
ENABLE
)
TABLESPACE USERS
PCTFREE 10 INITRANS 1
STORAGE (   BUFFER_POOL DEFAULT )
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1)
 17  (
PARTITION PARTITION_BEFORE_2016
NOLOGGING
 20  TABLESPACE USERS
 21  PCTFREE 10 INITRANS 1
STORAGE
(
 INITIAL 8388608
 NEXT 1048576
 26   MINEXTENTS 1
(
 INITIAL 8388608
 MAXEXTENTS UNLIMITED
 BUFFER_POOL DEFAULT
)
NOCOMPRESS NO INMEMORY,
PARTITION PARTITION_BEFORE_2017
 MAXEXTENTS UNLIMITED
NOLOGGING
TABLESPACE USERS02
PCTFREE 10 INITRANS 1
STORAGE
(
 INITIAL 8388608
 NEXT 1048576
 MINEXTENTS 1
 MAXEXTENTS UNLIMITED
 BUFFER_POOL DEFAULT
)
TABLESPACE USERS03
NOCOMPRESS NO INMEMORY,
PARTITION PARTITION_BEFORE_2018
NOLOGGING
TABLESPACE USERS03
 47  );

表已创建。
 ```
 
- 第四步骤 (插入数据)

Orders表，先创建序列
``` SQL
    CREATE SEQUENCE seq_test
  2   　　　　INCREMENT BY 1
  3   　　　　START WITH 1
  4  ;

序列已创建。
```
再执行三次循环语句
``` SQL
   Begin
  2    for i in 1..3000
  3    loop
  4           insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(seq_test.nextval,'李四',12345,to_date('2015-02-14','yyyy-mm-dd'),seq_test.nextval,seq_test.nextval);
  5     end loop;
  6     commit;
  7  end;
  8  /

PL/SQL 过程已成功完成。

   Begin
  2    for i in 1..3000
  loop
  4           insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(seq_test.nextval,'朱泓超',12345,to_date('2016-05-20','yyyy-mm-dd'),seq_test.nextval,seq_test.nextval);
  5     end loop;
  6     commit;
end;
  8  /

PL/SQL 过程已成功完成。
   
Begin
  loop
  for i in 1..3000
  3    loop
  4           insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(seq_test.nextval,'吴伟辉',12345,to_date('2017-05-14','yyyy-mm-dd'),seq_test.nextval,seq_test.nextval);
   end loop;
  6     commit;
end;
  8  /

PL/SQL 过程已成功完成。


```

order_details表插入数据

同样先创建序列

``` SQL
   CREATE SEQUENCE seq_test1
 　　　　INCREMENT BY 1
  3   　　　　START WITH 1
  4  ;

序列已创建。
```


插入数据


```SQL
   Begin
  for i in 1..10000
  loop
         insert into order_details(ID,
         ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(seq_test1.nextval,seq_test1.nextval,to_char(seq_test1.nextval),40,300);
   end loop;
   commit;
end;
/
```

- 第五步 联合查询

``` SQL
   SELECT * FROM orders,order_details  
   Where orders.order_id = order_details.order_id AND
   orders.order_date<=to_date('2018-05-14','yyyy-mm-dd')
```
![界面3](https://github.com/z915287285/Oracle/blob/master/test3/3.png)

# 总结：分区与不分区
## - 分区的话，如果进行大数据量（上百万）的查询，速度和效率会更高，不分区则相反
## - 本实验的话，数据量在一万条，个人感觉不是太明显，但是却有一点细微的效率差别
## - 本实验，受益良多，学会创建分区表以及联合查询
