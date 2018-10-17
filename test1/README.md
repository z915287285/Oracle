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

## 结果分析与比较 ##
```
从上面两种查询方法结果来看，查询的最终结果一致，存在一定的效率差。
从执行计划分析来看主要从以下方面来比对：
                                1.查询所用时间
                                2.内存占用
                                3.CPU总占用率
```
#### 综上分析 ####
有如下表格:
<table>
  <tr> 
    <th></th> 
    <th>查询1</th> 
    <th>查询2</th> 
  </tr>
  <tr> 
    <td>CPU总占用量比</td> 
    <td>10%</td> 
    <td>18%</td> 
  </tr> 
  <tr>
    <td>TIME</td>
    <td>00:00:07</td> 
    <td>00:00:05</td> 
  </tr>
  <tr> 
    <td>内存占用（字节）</td> 
    <td>1008</td> 
    <td>3665</td> 
  </tr> 
</table>

### 总得分析来看，查询1的方法相比查询2的方法时间更短。但是，占用内存，和CPU占用率比较大。 ###
### 根据个人分析来看，从时间和内存占用角度分析，第一种查询方法最优。 ###
### 如下图是sql developer的优化建议： ###
![优化建议](https://github.com/z915287285/Oracle/blob/master/test1/op1.png)
从上图看出，优化方案并没有什么变化，说明本查询语句已经是最优化的。

## 自定义优化SQL语句 ##
### 根据查找的资料以及相关知识分析，打算从以between替换in和exists替换in， ###
```SQL
set autotrace on
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name exists(select 1 from d where department_name between 'IT' and 'Sales')
GROUP BY department_name
```
参考资料来源于:
1. [资料1]https://www.cnblogs.com/xiezhi/p/6247423.html
2. [资料2]https://blog.csdn.net/ltaihyy/article/details/78331975
3. 本书方法
