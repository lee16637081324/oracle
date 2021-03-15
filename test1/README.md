## 实验目的

分析SQL执行计划，执行SQL语句的优化指导。理解分析SQL语句的执行计划的重要作用。

## 实验内容

- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。
- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。

## 教材中的查询语句

查询1：

```
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
from hr.departments d,hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT','Sales')
GROUP BY d.department_name;
```

- 查询2

```
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
FROM hr.departments d,hr.employees e
WHERE d.department_id = e.department_id
GROUP BY d.department_name
HAVING d.department_name in ('IT','Sales');
```

执行上面两个比较复杂的返回相同查询结果数据集的SQL语句，通过分析SQL语句各自的执行计划，判断哪个SQL语句是最优的。最后将你认为最优的SQL语句通过sqldeveloper的优化指导工具进行优化指导，看看该工具有没有给出优化建议

![image-20210315152253940](C:\Users\liyongqi\AppData\Roaming\Typora\typora-user-images\image-20210315152253940.png)

![image-20210315152314078](C:\Users\liyongqi\AppData\Roaming\Typora\typora-user-images\image-20210315152314078.png)

分别执行两个查询语句发现查询二的SQL语句是最优的，效率最高。因为查询一每次查询时要满足两个条件，查询department_name是否在（‘IT’，‘Sales’）中要查询d.department_id = e.department_id，查询二不会这样，通过sqldeveloper优化指导工具进行优化指导发现：oracle优化指导查找结果：无建议无原理无并未给出优化建议。

![image-20210315152400344](C:\Users\liyongqi\AppData\Roaming\Typora\typora-user-images\image-20210315152400344.png)

自己设计语句

```
 SELECT d.department_name,count(e.job_id)as "部门总人数",
	avg(e.salary)as "平均工资"
	from hr.departments d
    INNER JOIN
    hr.employees e
	on d.department_id = e.department_id
	and d.department_name='IT'
    OR  d.department_name='Sales'
	GROUP BY d.department_name;
```

使用sqldeveloper的优化指导工具进行优化指导

![image-20210315153201387](C:\Users\liyongqi\AppData\Roaming\Typora\typora-user-images\image-20210315153201387.png)

查找结果：在执行计划的行 ID 4 处发现开销很大的笛卡尔积操作。	

建议：考虑从此语句中移去断开连接的表或视图, 或者添加引用它的联接条件。	

原理：应尽可能避免笛卡尔积操作, 因为它的开销很大, 并且会产生大量数据。

索引语句对比其他两个查询较长更加详细，不过使用了INNER JOIN语句，较多使用and和or语句，并且排序，这样效率较高。

