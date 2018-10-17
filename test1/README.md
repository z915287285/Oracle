# 实验一：分析SQL执行计划，执行SQL语句的优化指导
----- 2016级软件工程3班朱泓超
## 查询1 ##
``` SQL
set autotrace on
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
```
### 查询结果1 ###
![查询结果](https://github.com/z915287285/Oracle/blob/master/test1/1.png)
### 执行计划1 ###
![执行计划](https://github.com/z915287285/Oracle/blob/master/test1/1-2.png)
### 统计信息1 ###
![统计信息](https://github.com/z915287285/Oracle/blob/master/test1/1-1.png)

## 查询2 ##
``` SQL
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
```
### 查询结果2 ###
![查询结果](https://github.com/z915287285/Oracle/blob/master/test1/2.png)
### 执行计划2 ###
![执行计划](https://github.com/z915287285/Oracle/blob/master/test1/2-1.png)
### 统计信息2 ###
![统计信息](https://github.com/z915287285/Oracle/blob/master/test1/2-2.png)
