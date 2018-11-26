# 实验5：PL/SQL编程

## 实验目的：

``` 
了解PL/SQL语言结构
了解PL/SQL变量和常量的声明和使用方法
学习条件语句的使用方法
学习分支语句的使用方法
学习循环语句的使用方法
学习常用的PL/SQL函数
学习包，过程，函数的用法。
```

## 实验场景：

- 假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
- 本实验以实验四为基础

## 实验步骤

### 第一步  声明包，查询函数以及存储过程

实现如下：

``` sql
create or replace PACKAGE MyPack IS
  /*
  声明包MyPack中有：
  一个函数:Get_SaleAmount(V_DEPARTMENT_ID NUMBER)，
  一个过程:Get_Employees(V_EMPLOYEE_ID NUMBER)
  */
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
  PROCEDURE Get_Employees(V_EMPLOYEE_ID NUMBER);
END MyPack;
/

Package MYPACK 已编译
```

### 第二步  创建包，函数以及存储过程

实现如下：

``` sql
 create or replace PACKAGE BODY MyPack 
 IS -- 创建MyPack 包
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER-- 定义Get_SaleAmount函数
  AS
    N NUMBER(20,2); 
    BEGIN
      SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O,EMPLOYEES E
      WHERE O.EMPLOYEE_ID=E.EMPLOYEE_ID AND E.DEPARTMENT_ID =V_DEPARTMENT_ID;
      RETURN N;
    END;

  PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)-- 创建存储过程
  AS
    LEFTSPACE VARCHAR(2000);
    begin
      --通过LEVEL判断递归的级别
      LEFTSPACE:=' ';
      --使用游标
      for v in
      (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
      START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
      CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID)
      LOOP
        DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')|| V.EMPLOYEE_ID||' '||v.NAME);
      END LOOP;
    END;
END MyPack;

Package Body MYPACK 已编译

```

### 第三步 进行测试

实现如下:

函数 Get_SaleAmount()测试：

``` sql
select count(*) from orders;
select MyPack.Get_SaleAmount(11) AS 部门11应收金额,MyPack.Get_SaleAmount(12) AS 部门12应收金额 from dual;
```

结果：
``` 
    Count
1   1000

```

``` 
     部门11应收金额      部门12应收金额
1    70028304.26	      69994476.88
```

过程Get_Employess()测试

``` sql
set serveroutput on
DECLARE
  V_EMPLOYEE_ID NUMBER;    
BEGIN
  V_EMPLOYEE_ID := 1;
  MYPACK.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;  
  V_EMPLOYEE_ID := 11;
  MYPACK.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;    
END;
/

```
结果：

``` 
1 李董事长
    11 张总
        111 吴经理
        112 白经理
    12 王总
        121 赵经理
        122 刘经理
11 张总
    111 吴经理
    112 白经理

PL/SQL 过程已成功完成。

```

## 实验总结

本实验基于实验四进行实验，主要内容是声明并创建包，函数以及过程。
我总结了以下几个实验要点：
```
1.在创建之前必须先声明。
2.创建的时候注意数据类型，以及数据大小。
3.注意递归查询方式。

通过本次实验，受益良多。学习了包创建，函数设计，以及过程的建立（尤其是调用递归查询的思想）。
```
