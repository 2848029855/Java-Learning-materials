# MySQL知识点

## 1. 常用函数

```mysql
概念：类似于java的方法，将一组逻辑语句封装在方法体中，对外暴露方法名
好处：1、隐藏了实现细节  2、提高代码的重用性
调用：select 函数名(实参列表) 【from 表】;
特点：
	①叫什么（函数名）
	②干什么（函数功能）

分类：
	1、单行函数
	如 concat、length、ifnull等
	2、分组函数
	
	功能：做统计使用，又称为统计函数、聚合函数、组函数
	
常见函数：
	一、单行函数
	字符函数：
	length:获取字节个数(utf-8一个汉字代表3个字节,gbk为2个字节)
	concat
	substr
	instr
	trim
	upper
	lower
	lpad
	rpad
	replace
	
	数学函数：
	round
	ceil
	floor
	truncate
	mod
	
	日期函数：
	now
	curdate
	curtime
	year
	month
	monthname
	day
	hour
	minute
	second
	str_to_date
	date_format
	其他函数：
	version
	database
	user
	控制函数
	if
	case
*/

去重
SELECT DISTINCT `department_id` FROM `departments`

拼接
SELECT 
  CONCAT(first_name,last_name) AS 姓名 
FROM
  employees 
  
是否为空
SELECT 
  IFNULL(`commission_pct`, 0) AS 奖金率,
  commission_pct 
FROM
  employees 

#一、字符函数

#1.length 获取参数值的字节个数
SELECT LENGTH('john');
SELECT LENGTH('张三丰hahaha');

SHOW VARIABLES LIKE '%char%'

#2.concat 拼接字符串

SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;

#3.upper、lower
SELECT UPPER('john');
SELECT LOWER('joHn');
#示例：将姓变大写，名变小写，然后拼接
SELECT CONCAT(UPPER(last_name),LOWER(first_name))  姓名 FROM employees;

#4.substr、substring
注意：索引从1开始
#截取从指定索引处后面所有字符
SELECT SUBSTR('李莫愁爱上了陆展元',7)  out_put;

#截取从指定索引处指定字符长度的字符
SELECT SUBSTR('李莫愁爱上了陆展元',1,3) out_put;


#案例：姓名中首字符大写，其他字符小写然后用_拼接，显示出来

SELECT CONCAT(UPPER(SUBSTR(last_name,1,1)),'_',LOWER(SUBSTR(last_name,2)))  out_put
FROM employees;

#5.instr 返回子串第一次出现的索引，如果找不到返回0

SELECT INSTR('杨不殷六侠悔爱上了殷六侠','殷八侠') AS out_put;

#6.trim

SELECT LENGTH(TRIM('    张翠山    ')) AS out_put;

SELECT TRIM('aa' FROM 'aaaaaaaaa张aaaaaaaaaaaa翠山aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa')  AS out_put;

#7.lpad 用指定的字符实现左填充指定长度

SELECT LPAD('殷素素',2,'*') AS out_put;

#8.rpad 用指定的字符实现右填充指定长度

SELECT RPAD('殷素素',12,'ab') AS out_put;


#9.replace 替换

SELECT REPLACE('周芷若周芷若周芷若周芷若张无忌爱上了周芷若','周芷若','赵敏') AS out_put;



#二、数学函数

#round 四舍五入
SELECT ROUND(-1.55);
SELECT ROUND(1.567,2);


#ceil 向上取整,返回>=该参数的最小整数

SELECT CEIL(-1.02);

#floor 向下取整，返回<=该参数的最大整数
SELECT FLOOR(-9.99);

#truncate 截断

SELECT TRUNCATE(1.69999,1);

#mod取余
/*
mod(a,b) ：  a-a/b*b

mod(-10,-3):-10- (-10)/(-3)*（-3）=-1
*/
SELECT MOD(10,-3);
SELECT 10%3;


#三、日期函数

#now 返回当前系统日期+时间
SELECT NOW();

#curdate 返回当前系统日期，不包含时间
SELECT CURDATE();

#curtime 返回当前时间，不包含日期
SELECT CURTIME();


#可以获取指定的部分，年、月、日、小时、分钟、秒
SELECT YEAR(NOW()) 年;
SELECT YEAR('1998-1-1') 年;

SELECT  YEAR(hiredate) 年 FROM employees;

SELECT MONTH(NOW()) 月;
SELECT MONTHNAME(NOW()) 月;


#str_to_date 将字符通过指定的格式转换成日期

SELECT STR_TO_DATE('1998-3-2','%Y-%c-%d') AS out_put;

#查询入职日期为1992--4-3的员工信息
SELECT * FROM employees WHERE hiredate = '1992-4-3';

SELECT * FROM employees WHERE hiredate = STR_TO_DATE('4-3 1992','%c-%d %Y');


#date_format 将日期转换成字符

SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put;

#查询有奖金的员工名和入职日期(xx月/xx日 xx年)
SELECT last_name,DATE_FORMAT(hiredate,'%m月/%d日 %y年') 入职日期
FROM employees
WHERE commission_pct IS NOT NULL;


#四、其他函数

SELECT VERSION();
SELECT DATABASE();
SELECT USER();


#五、流程控制函数
#1.if函数： if else 的效果

SELECT IF(10<5,'大','小');

SELECT last_name,commission_pct,IF(commission_pct IS NULL,'没奖金，呵呵','有奖金，嘻嘻') 备注
FROM employees;




#2.case函数的使用一： switch case 的效果

/*
java中
switch(变量或表达式){
	case 常量1：语句1;break;
	...
	default:语句n;break;


}

mysql中

case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1;
when 常量2 then 要显示的值2或语句2;
...
else 要显示的值n或语句n;
end
*/

/*案例：查询员工的工资，要求

部门号=30，显示的工资为1.1倍
部门号=40，显示的工资为1.2倍
部门号=50，显示的工资为1.3倍
其他部门，显示的工资为原工资

*/


SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salary
END AS 新工资
FROM employees;



#3.case 函数的使用二：类似于 多重if
/*
java中：
if(条件1){
	语句1；
}else if(条件2){
	语句2；
}
...
else{
	语句n;
}

mysql中：

case 
when 条件1 then 要显示的值1或语句1G
when 条件2 then 要显示的值2或语句2
。。。
else 要显示的值n或语句n
end
*/

#案例：查询员工的工资的情况
如果工资>20000,显示A级别
如果工资>15000,显示B级别
如果工资>10000，显示C级别
否则，显示D级别


SELECT salary,
CASE 
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资级别
FROM employees;
```

## 2.分组查询

```mysql
/*
语法：

select 查询列表
from 表
【where 筛选条件】
group by 分组的字段
【order by 排序的字段】;

特点：
1、和分组函数一同查询的字段必须是group by后出现的字段
2、筛选分为两类：分组前筛选和分组后筛选
		针对的表			位置		连接的关键字
分组前筛选	原始表				group by前	where
	
分组后筛选	group by后的结果集    		group by后	having

问题1：分组函数做筛选能不能放在where后面
答：不能

问题2：where——group by——having

一般来讲，能用分组前筛选的，尽量使用分组前筛选，提高效率

3、分组可以按单个字段也可以按多个字段
4、可以搭配着排序使用
*/
```

#### 2.2分组查询案例

```mysql
#案例：查询每个工种每个部门的最低工资,并按最低工资降序

SELECT job_id,MIN(salary)
FROM employees
GROUP BY department_id,job_id
ORDER BY MIN(salary) DESC;


#1.查询各job_id的员工工资的最大值，最小值，平均值，总和，并按job_id升序
SELECT MAX(salary),MIN(salary),AVG(salary),SUM(salary),job_id
FROM employees
GROUP BY job_id
ORDER BY job_id

#2.查询员工最高工资和最低工资的差距（DIFFERENCE）
SELECT MAX(salary) - MIN(salary)
FROM employees

#3.查询各个管理者手下员工的最低工资，其中最低工资不能低于6000，没有管理者的员工不计算在内
SELECT MIN(salary),manager_id
FROM employees
WHERE manager_id IS NOT NULL
GROUP BY manager_id
HAVING MIN(salary) >= 6000

#4.查询所有部门的编号，员工数量和工资平均值,并按平均工资降序
SELECT department_id,COUNT(*),AVG(salary) a
FROM employees
GROUP BY department_id
ORDER BY a DESC

#5.选择具有各个job_id的员工人数
SELECT COUNT(*) 个,job_id
FROM employees
GROUP BY job_id

```



## 3.连接查询

```mysql
/*
含义：又称多表查询，当查询的字段来自于多个表时，就会用到连接查询

笛卡尔乘积现象：表1 有m行，表2有n行，结果=m*n行

发生原因：没有有效的连接条件
如何避免：添加有效的连接条件

分类：

	按年代分类：
	sql92标准:仅仅支持内连接
	sql99标准【推荐】：支持内连接+外连接（左外和右外）+交叉连接
	
	按功能分类：
		内连接： join
			等值连接 ==
			非等值连接 != 
			自连接 
		外连接：
			左外连接 left
			右外连接 right
			全外连接 
		
		交叉连接
*/
```

```mysql

#案例2：查询员工名和对应的部门名
USE myemployees

SELECT last_name,department_name
FROM employees,departments
WHERE employees.`department_id` = departments.`department_id`

#查询员工名、工种号、工种名
SELECT e.last_name,e.job_id,j.job_title
FROM employees e,jobs j
WHERE e.`job_id` = j.`job_id`

#3、两个表的顺序是否可以调换


#查询员工名、工种号、工种名
SELECT e.last_name,e.job_id,j.job_title
FROM employees e,jobs j
WHERE j.`job_id` = e.`job_id`

#4、可以加筛选

#案例：查询有奖金的员工名、部门名
SELECT last_name,department_name,commission_pct
FROM employees e, departments d
WHERE e.`department_id` = d.`department_id`
AND e.`commission_pct` IS NOT NULL

#案例2：查询城市名中第二个字符为o的部门名和城市名

SELECT department_name,city
FROM departments d,locations l
WHERE d.`location_id` = l.`location_id`
AND city LIKE '_o%'

#5、可以加分组

#案例1：查询每个城市的部门个数

SELECT COUNT(*) 个数,city
FROM departments d,locations l
WHERE d.`location_id` = l.`location_id`
GROUP BY city

#案例2：查询有奖金的每个部门的部门名和部门的领导编号和该部门的最低工资
SELECT department_name,d.manager_id,MIN(salary)
FROM departments d,employees e
WHERE d.`department_id` = e.`department_id`
AND commission_pct IS NOT NULL
GROUP BY department_name,d.manager_id

#6、可以加排序

#案例：查询每个工种的工种名和员工的个数，并且按员工个数降序
SELECT job_title,COUNT(*)
FROM jobs j,employees e
WHERE j.`job_id` = e.`job_id`
GROUP BY job_title
ORDER BY COUNT(*) DESC


#3-------------------------------------------------

#7、可以实现三表连接？   

#案例：查询员工名、部门名和所在的城市

SELECT last_name 员工名,department_name 部门名,city 城市
FROM employees e,departments d,locations l
WHERE e.`department_id` = d.`department_id`
AND d.`location_id` = l.`location_id`
AND city LIKE 's%'
ORDER BY department_name DESC

#2、非等值连接


#案例1：查询员工的工资和工资级别

SELECT salary,grade_level
FROM emmployees e,job_grades g
WHERE salary BETWEEN g.

#3、自连接
SELECT e.employee_id,e.last_name,m.employee_id,m.last_name
FROM employees e,employees m
WHERE e.`manager_id` = m.`employee_id`
```

### 3.2查询练习案例

```mysql

#1. 显示所有员工的姓名，部门号和部门名称。
SELECT e.last_name,d.department_id,d.department_name
FROM employees e, departments d
WHERE e.`employee_id` = d.`department_id`

#2. 查询 90 号部门员工的 job_id 和 90 号部门的 location_id
SELECT job_id,location_id
FROM employees e, departments d
WHERE e.`department_id` = d.`department_id`
AND e.`department_id` = 90

#3. 选择所有有奖金的员工的
last_name , department_name , location_id , city

SELECT last_name,department_name,l.location_id,city
FROM employees e,departments d,locations l
WHERE e.department_id = d.department_id
AND d.location_id = l.location_id
AND e.commission_pct IS NOT NULL


#4. 选择city在Toronto工作的员工的
last_name , job_id , department_id , department_name

SELECT last_name , job_id , d.department_id , department_name
FROM employees e,departments d,locations l
WHERE e.department_id = d.department_id
AND d.location_id = l.location_id
AND city = 'Toronto';


#5.查询每个工种、每个部门的部门名、工种名和最低工资

SELECT department_name,job_title,MIN(salary) 最低工资
FROM employees e,departments d,jobs j
WHERE e.department_id = d.department_id
AND e.job_id = j.job_id
GROUP BY department_name,job_title

#6.查询每个国家下的部门个数大于 2 的国家编号
SELECT country_id,COUNT(*) 部门个数
FROM departments d,locations l
WHERE d.`location_id` = l.`location_id`
GROUP BY country_id

#7、选择指定员工的姓名，员工号，以及他的管理者的姓名和员工号，结果类似于下面的格式
employees Emp# manager Mgr# 
kochhar  101    king 100

SELECT e.last_name employees,e.employee_id 'Emp3',m.last_name mnager,m.employee_id 'Mgr#'
FROM employees e, employees m
WHERE e.manager_id java= m.employee_id
AND e.last_name='Kochhar'
```

## 4.等值连接

```mysql
#1、等值连接
#案例1.查询员工名、部门名

SELECT last_name,department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id

#案例2.查询名字中包含e的员工名和工种名（添加筛选）
SELECT last_name,job_title
FROM employees e
INNER JOIN jobs j
ON e.job_id = j.job_id
WHERE e.last_name LIKE '%e%'


#3. 查询部门个数>3的城市名和部门个数，（添加分组+筛选）
SELECT city,COUNT(*) 部门个数
FROM departments d
INNER JOIN locations l
ON d.`location_id`=l.`location_id`
GROUP BY city
HAVING COUNT(*) > 3

#①查询每个城市的部门个数
#②在①结果上筛选满足条件的

#案例4.查询哪个部门的员工个数>3的部门名和员工个数，并按个数降序（添加排序）

#①查询每个部门的员工个数
SELECT COUNT(*),department_name
FROM employees e
INNER JOIN departments d
ON e.`department_id` = d.department_id
GROUP BY department_name

#② 在①结果上筛选员工个数>3的记录，并排序
SELECT COUNT(*),department_name
FROM employees e
INNER JOIN departments d
ON e.`department_id` = d.department_id
GROUP BY department_name
HAVING COUNT(*) > 3
ORDER BY COUNT(*) DESC


#5.查询员工名、部门名、工种名，并按部门名降序（添加三表连接）
SELECT last_name,department_name,job_title
FROM emplo yees e
INNER JOIN departments d ON e.`department_id` = d.`department_id`
INNER JOIN jobs j ON e.`job_id` = j.`job_id`
ORDER BY department_name
```

### 4.1子查询

```mysql

 
#进阶7：子查询
/*
含义：
出现在其他语句中的select语句，称为子查询或内查询
外部的查询语句，称为主查询或外查询

分类：
按子查询出现的位置：
	select后面：
		仅仅支持标量子查询
	
	from后面：
		支持表子查询
	where或having后面：★
		标量子查询（单行） √
		列子查询  （多行） √
		
		行子查询
		
	exists后面（相关子查询）
		表子查询
按结果集的行列数不同：
	标量子查询（结果集只有一行一列）
	列子查询（结果集只有一列多行）
	行子查询（结果集有一行多列）
	表子查询（结果集一般为多行多列）



*/


#一、where或having后面
/*
1、标量子查询（单行子查询）
2、列子查询（多行子查询）

3、行子查询（多列多行）

特点：
①子查询放在小括号内
②子查询一般放在条件的右侧
③标量子查询，一般搭配着单行操作符使用
> < >= <= = <>

列子查询，一般搭配着多行操作符使用
in、any/some、all

④子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果

*/
#1.标量子查询★

#案例1：谁的工资比 Abel 高?

#①查询Abel的工资
SELECT salary
FROM employees
WHERE last_name = 'Abel'

#②查询员工的信息，满足 salary>①结果
SELECT *
FROM employees
WHERE salary > (
	SELECT salary
	FROM employees
	WHERE last_name = 'Abel'
)

#案例2：返回job_id与141号员工相同，salary比143号员工多的员工 姓名，job_id 和工资

#①查询141号员工的job_id
SELECT job_id
FROM employees
WHERE employee_id = 141
#②查询143号员工的salary

SELECT salary
FROM employees
WHERE employee_id = 143


#③查询员工的姓名，job_id 和工资，要求job_id=①并且salary>②
SELECT last_name,job_id,salary
FROM employees
WHERE job_id = (
	SELECT job_id
	FROM employees
	WHERE employee_id = 141
) AND salary >(
	SELECT salary
	FROM employees
	WHERE employee_id = 143
)


#案例3：返回公司工资最少的员工的last_name,job_id和salary

#①查询公司的 最低工资
SELECT MIN(salary)
FROM employees


#②查询last_name,job_id和salary，要求salary=①
SELECT last_name,job_id,salary
FROM employees
WHERE salary = (
	SELECT MIN(salary)
	FROM employees
)


#案例4：查询最低工资大于50号部门最低工资的部门id和其最低工资

#①查询50号部门的最低工资
SELECT MIN(salary)
FROM employees
WHERE department_id = 50

#②查询每个部门的最低工资
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id

#③ 在②基础上筛选，满足min(salary)>①
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary) > (
	SELECT MIN(salary)
	FROM employees
	WHERE department_id = 50
)
#非法使用标量子查询

SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary) > (
	SELECT MIN(salary)
	FROM employees
	WHERE department_id = 250
)


#2.列子查询（多行子查询）★
#案例1：返回location_id是1400或1700的部门中的所有员工姓名

#①查询location_id是1400或1700的部门编号
SELECT DISTINCT department_id
FROM departments
WHERE location_id IN (1400,1700)

#②查询员工姓名，要求部门号是①列表中的某一个
SELECT last_name
FROM employees
WHERE department_id <>ALL(
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id IN (1400,1700)
)


#案例2：返回其它工种中比job_id为‘IT_PROG’工种任一工资低的员工的员工号、姓名、job_id 以及salary


#①查询job_id为‘IT_PROG’部门任一工资
SELECT DISTINCT salary
FROM employees
WHERE job_id = 'IT_PROG'

#②查询员工号、姓名、job_id 以及salary，salary<(①)的任意一个
任意 所有 ALL 任一 ANY

SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary <ANY(
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'
) AND job_id <> 'IT_PROG'
 #或
 SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary <(
	SELECT MAX(salary)
	FROM employees
	WHERE job_id = 'IT_PROG'
) AND job_id <> 'IT_PROG'


#案例3：返回其它部门中比job_id为‘IT_PROG’部门所有工资都低的员工   的员工号、姓名、job_id 以及salary
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary <ALL(
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'
) AND job_id <> 'IT_PROG'

#或



#3、行子查询（结果集一行多列或多行多列）

#案例：查询员工编号最小并且工资最高的员工信息

SELECT *
FROM employees
WHERE (employee_id,salary) = (
	SELECT MIN(employee_id),MAX(salary)
	FROM employees
)

#①查询最小的员工编号
SELECT MIN(employee_id)
FROM employees

#②查询最高工资
SELECT MAX(salary)
FROM employees

#③查询员工信息
SELECT *
FROM employees
WHERE employee_id = (
	SELECT MIN(employee_id)
	FROM employees
) AND salary = (
	SELECT MAX(salary) 
	FROM employees
)

#二、select后面
/*
仅仅支持标量子查询
*/

#案例：查询每个部门的员工个数

SELECT d.*,(
	SELECT COUNT(*)
	FROM employees e
	WHERE e.department_id = d.department_id
) 个数
FROM departments d
 
 #案例2：查询员工号=102的部门名
 
 SELECT (
	SELECT department_name
	FROM `departments` d
	INNER JOIN `employees` e
	ON d.department_id = e.department_id
	WHERE e.employee_id = 102
 ) 部门名

#三、from后面
/*
将子查询结果充当一张表，要求必须起别名
*/

#案例：查询每个部门的平均工资的工资等级
#①查询每个部门的平均工资
SELECT AVG(salary),department_id
FROM employees
GROUP BY department_id

#②连接①的结果集和job_grades表，筛选条件平均工资 between lowest_sal and highest_sal
SELECT ag_dep.*,g.grade_level
FROM (
	SELECT AVG(salary),department_id
	FROM employees
	GROUP BY department_id
) ag_dep
INNER JOIN job_grades g
ON ag_dep.ag BETWEEN lowest_sal AND highest_sal



#四、exists后面（相关子查询）

/*
语法：
exists(完整的查询语句)
结果：
1或0
*/

#案例1：查询有员工的部门名
SELECT EXISTS(SELECT employee_id FROM employees WHERE salary = 300000)
#in
SELECT department_name
FROM departments d
WHERE d.`department_id` IN
(
	SELECT department_id
	FROM employees 
	
)
#exists 是否返回值 返回 true1或false0

SELECT department_name
FROM departments d
WHERE EXISTS (
	SELECT *
	FROM employees e
	WHERE d.department_id =e.department_id
)


#案例2：查询没有女朋友的男神信息
USE girls


SELECT * FROM boys 
SELECT * FROM beauty
#in
SELECT bo.*
FROM boys bo
WHERE bo.id NOT IN (
	SELECT boyfriend_id
	FROM beauty b
	WHERE bo.id = b.boyfriend_id
)

#exists
SELECT bo.*
FROM boys bo
WHERE NOT EXISTS (
	SELECT boyfriend_id
	FROM beauty b
	WHERE bo.id = b.boyfriend_id
)
```

#### 4.1.1子查询练习

```mysql
#1.	查询和Zlotkey相同部门的员工姓名和工资

#①查询Zlotkey的部门
SELECT department_id
FROM employees
WHERE last_name = 'Zlotkey'

#②查询部门号=①的姓名和工资
SELECT last_name,salary
FROM employees
WHERE department_id = (
	SELECT department_id
	FROM employees
	WHERE last_name = 'Zlotkey'
)

#2.查询工资比公司平均工资高的员工的员工号，姓名和工资。

#①查询平均工资
SELECT AVG(salary)
FROM employees


#②查询工资>①的员工号，姓名和工资。
SELECT last_name,salary,employee_id
FROM employees e
WHERE salary > (
	SELECT AVG(salary)
	FROM employees
)

#3.查询各部门中工资比本部门平均工资高的员工的员工号, 姓名和工资

#①查询各部门的平均工资
SELECT AVG(salary),department_id
FROM employees
GROUP BY department_id

#②连接①结果集和employees表，进行筛选
SELECT last_name,salary,employee_id
FROM employees e
INNER JOIN (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id
) ag_dep
ON e.department_id = ag_dep.department_id
WHERE salary > ag_dep.ag

#4.	查询和姓名中包含字母u的员工在相同部门的员工的员工号和姓名
#①查询姓名中包含字母u的员工的部门
SELECT DISTINCT department_id
FROM employees
WHERE last_name LIKE '%u%'

#②查询部门号=①中的任意一个的员工号和姓名
SELECT last_name,employee_id
FROM employees
WHERE department_id IN (
	SELECT DISTINCT department_id
	FROM employees
	WHERE last_name LIKE '%u%'
)

#5. 查询在部门的location_id为1700的部门工作的员工的员工号

#①查询location_id为1700的部门
SELECT department_id
FROM departments 
WHERE location_id = 1700
#②查询部门号=①中的任意一个的员工号
SELECT employee_id
FROM employees
WHERE department_id  = ANY (
	SELECT department_id
	FROM departments 
	WHERE location_id = 1700
)


#6.查询管理者是King的员工姓名和工资

#①查询姓名为king的员工编号
SELECT employee_id
FROM employees 
WHERE last_name = 'K_ing'

#②查询哪个员工的manager_id = ①
SELECT last_name,salary
FROM employees
WHERE manager_id IN (
	SELECT employee_id
	FROM employees 
	WHERE last_name = 'K_ing'
)


#7.查询工资最高的员工的姓名，要求first_name和last_name显示为一列，列名为 姓.名


#①查询最高工资
SELECT MAX(salary)
FROM employees

#②查询工资=①的姓.名
SELECT CONCAT(first_name,last_name)
FROM employees
WHERE salary = (
	SELECT MAX(salary)
	FROM employees
)
```

### 4.2子查询经典案例

```mysql
#1. 查询工资最低的员工信息: last_name, salary
#1.1、查询最低工资
SELECT MIN(salary)
FROM employees

#2.2、查询last_name,salary
SELECT last_name,salary
FROM employees
WHERE salary = (
	SELECT MIN(salary)
	FROM employees
)



#2. 查询平均工资最低的部门信息

#2.1 最低工资
SELECT department_id
FROM employees
GROUP BY department_id
ORDER BY AVG(salary)
LIMIT 1
#2.2 查询1结果的最低工资
SELECT * 
 FROM departments
 WHERE department_id = (
	SELECT department_id
	FROM employees
	GROUP BY department_id
	ORDER BY AVG(salary)
	LIMIT 1 
 )


#3. 查询平均工资最低的部门信息和该部门的平均工资
SELECT AVG(salary),department_id
FROM employees
GROUP BY department_id
#2查询最低工资的部门编号
SELECT department_id
FROM employees
GROUP BY department_id
ORDER BY AVG(salary)
LIMIT 1
#3.
SELECT d.*,ag
FROM departments d
JOIN (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id
	ORDER BY AVG(salary)
	LIMIT 1
) ag_dep
ON d.department_id = ag_dep.department_id


#4. 查询平均工资最高的 job 信息
#1、查询每个job的平均工资
SELECT AVG(salary),job_id
FROM employees
GROUP BY job_id
ORDER BY AVG(salary) DESC
LIMIT 1


#2.查询job信息
SELECT * 
FROM jobs
WHERE job_id = (
	SELECT job_id
	FROM employees
	GROUP BY job_id
	ORDER BY AVG(salary) DESC
	LIMIT 1
)

#5. 查询平均工资高于公司平均工资的部门有哪些?
#1.查询平均工资
SELECT AVG(salary)
FROM employees

#2.查询每个部门的平均工资
SELECT AVG(salary)
FROM employees
GROUP BY department_id

#3、筛选2的结果 满足平均工资 > 1
 
#6. 查询出公司中所有 manager 的详细信息.
#拆线呢所有manager的员工编号
SELECT DISTINCT manager_id
FROM employees

#查询详细信息 满足manager_id =1
SELECT *
FROM employees
WHERE employee_id = ANY (
	SELECT DISTINCT manager_id
	FROM employees
)

#7. 各个部门中 最高工资中最低的那个部门的 最低工资是多少


#查询各部门的最高工资的部门编号
SELECT department_id
FROM employees
GROUP BY department_id
ORDER BY MAX(salary)
LIMIT 1

#查询1结果的最低工资
SELECT MIN(salary)
FROM employees
WHERE department_id = (
	SELECT department_id
	FROM employees
	GROUP BY department_id
	ORDER BY MAX(salary)
	LIMIT 1
)


#8. 查询平均工资最高的部门的 manager 的详细信息: last_name, department_id, email,salary

#查询平均工资最高的部门编号
SELECT department_id
FROM employees
GROUP BY department_id
ORDER BY AVG(salary) DESC
LIMIT 1

#将employees和department连接查询  筛选条件
SELECT 
  last_name,
  d.department_id,
  email,
  salary 
FROM
  employees e 
  INNER JOIN departments d 
    ON d.manager_id = e.employee_id 
WHERE d.department_id = 
  (SELECT 
    department_id 
  FROM
    employees 
  GROUP BY department_id 
  ORDER BY AVG(salary) DESC 
  LIMIT 1)
```

### 4.3子查询作业

```mysql
#一、查询每个专业的学生人数
SELECT majorid,COUNT(*)
FROM major
GROUP BY majorid;

#二、查询参加考试的学生中，每个学生的平均分、最高分
SELECT AVG(score),MAX(score),studentno
FROM result
GROUP BY studentno

#三、查询姓张的每个学生的最低分大于60的学号、姓名
SELECT s.studentname,s.studentno,MIN(score)
FROM student s
INNER JOIN result r 
ON s.studentno = r.studentno
WHERE s.studentname LIKE '%张%'
GROUP BY s.studentno
HAVING MIN(r.score) > 60

#四、查询专业生日在“1988-1-1”后的学生姓名、专业名称

SELECT studentname,majorname
FROM student s
JOIN major m
ON s.`majorid` = m.majorid
WHERE DATEDIFF(borndate,'1988-1-1') > 0
# datediff 前面数大于后面数 返回整数

#五、查询每个专业的男生人数和女生人数分别是多少
#方式一:
SELECT COUNT(*),sex,majorid
FROM student
GROUP BY sex,majorid;
#方式二
SELECT majorid,
(SELECT COUNT(*) FROM student WHERE sex = '男' AND majorid = s.majorid) 男,
(SELECT COUNT(*) FROM student WHERE sex = '女' AND majorid = s.majorid) 女
FROM student s
GROUP BY majorid

#六、查询专业和张翠山一样的学生的最低分

#查询张翠三的专业编号
SELECT majorid 
FROM student
WHERE studentname = '张翠山'

#查询编号=1的所有学生编号
SELECT studentno
FROM student
WHERE majorid = (
	SELECT majorid 
	FROM student
	WHERE studentname = '张翠山'
)
#查询最低分
SELECT 
  MIN(score) 
FROM
  result 
WHERE studentno IN 
  (SELECT 
    studentno 
  FROM
    student 
  WHERE majorid = 
    (SELECT 
      majorid 
    FROM
      student 
    WHERE studentname = '张翠山'))
    
#七、查询大于60分的学生的姓名、密码、专业名
SELECT studentname,loginpwd,m.`majorname`,r.score
FROM student s
INNER JOIN major m
ON s.majorid = m.majorid
INNER JOIN result r
ON s.studentno = r.studentno
WHERE r.score > 60

#八、按邮箱位数分组，查询每组的学生个数
SELECT COUNT(*) 个数
FROM student 
GROUP BY LENGTH(email)
#九、查询学生名、专业名、分数
SELECT studentname,majorname,score
FROM student s
JOIN major m 
ON m.majorid = s.`majorid`
LEFT JOIN result r
ON r.studentno = s.`studentno`

#十、查询哪个专业没有学生，分别用左连接和右连接实现
SELECT m.majorid,m.majorname,s.studentno
FROM major m
LEFT JOIN student s ON m.`majorid` = s.`majorid`
WHERE s.studentno IS NULL

SELECT m.majorid,m.majorname,s.studentno
FROM student s
RIGHT JOIN  major m ON m.`majorid` = s.`majorid`
WHERE s.studentno IS NULL


#十一、查询没有成绩的学生人数
SELECT COUNT(*)
FROM student s
LEFT JOIN result r ON r.studentno = s.`studentno`
WHERE r.id IS NULL
```

## 5.union

```java
#进阶9：联合查询
/*
union 联合 合并：将多条查询语句的结果合并成一个结果

语法：
查询语句1
union
查询语句2
union
...


应用场景：
要查询的结果来自于多个表，且多个表没有直接的连接关系，但查询的信息一致时

特点：★
1、要求多条查询语句的查询列数是一致的！
2、要求多条查询语句的查询的每一列的类型和顺序最好一致
3、union关键字默认去重，如果使用union all 可以包含重复项

*/


#引入的案例：查询部门编号>90或邮箱包含a的员工信息

SELECT * FROM employees WHERE email LIKE '%a%' OR department_id>90;;

SELECT * FROM employees  WHERE email LIKE '%a%'
UNION
SELECT * FROM employees  WHERE department_id>90;


#案例：查询中国用户中男性的信息以及外国用户中年男性的用户信息

SELECT id,cname FROM t_ca WHERE csex='男'
UNION ALL
SELECT t_id,tname FROM t_ua WHERE tGender='male';

```





## 6.DDL

### 6.1修改多表

```mysql
#多表更新

#2.修改多表的记录

#案例 1：修改张无忌的女朋友的手机号为114
UPDATE boys bo
INNER JOIN beauty b ON bo.id = b.`boyfriend_id`
SET b.`phone` = '119',bo.`userCP` = 1000
WHERE bo.`boyName` = '张无忌'


#案例2：修改没有男朋友的女神的男朋友编号都为2号
UPDATE boys bo
RIGHT JOIN beauty b ON bo.id = b.`boyfriend_id`
SET b.`boyfriend_id` =2
WHERE bo.`id` IS NULL
```

### 6.2删除

```mysql
#DML语言
/*
数据操作语言：
插入：insert
修改：update
删除：delete

*/

#一、插入语句
#方式一：经典的插入
/*
语法：
insert into 表名(列名,...) values(值1,...);

*/
SELECT * FROM beauty;
#1.插入的值的类型要与列的类型一致或兼容
INSERT INTO beauty(id,NAME,sex,borndate,phone,photo,boyfriend_id)
VALUES(13,'唐艺昕','女','1990-4-23','1898888888',NULL,2);

#2.不可以为null的列必须插入值。可以为null的列如何插入值？
#方式一：
INSERT INTO beauty(id,NAME,sex,borndate,phone,photo,boyfriend_id)
VALUES(13,'唐艺昕','女','1990-4-23','1898888888',NULL,2);

#方式二：

INSERT INTO beauty(id,NAME,sex,phone)
VALUES(15,'娜扎','女','1388888888');


#3.列的顺序是否可以调换
INSERT INTO beauty(NAME,sex,id,phone)
VALUES('蒋欣','女',16,'110');


#4.列数和值的个数必须一致

INSERT INTO beauty(NAME,sex,id,phone)
VALUES('关晓彤','女',17,'110');

#5.可以省略列名，默认所有列，而且列的顺序和表中列的顺序一致

INSERT INTO beauty
VALUES(18,'张飞','男',NULL,'119',NULL,NULL);

#方式二：
/*

语法：
insert into 表名
set 列名=值,列名=值,...
*/


INSERT INTO beauty
SET id=19,NAME='刘涛',phone='999';


#两种方式大pk ★


#1、方式一支持插入多行,方式二不支持

INSERT INTO beauty
VALUES(23,'唐艺昕1','女','1990-4-23','1898888888',NULL,2)
,(24,'唐艺昕2','女','1990-4-23','1898888888',NULL,2)
,(25,'唐艺昕3','女','1990-4-23','1898888888',NULL,2);

#2、方式一支持子查询，方式二不支持

INSERT INTO beauty(id,NAME,phone)
SELECT 26,'宋茜','11809866';

INSERT INTO beauty(id,NAME,phone)
SELECT id,boyname,'1234567'
FROM boys WHERE id<3;

#二、修改语句

/*

1.修改单表的记录★

语法：
update 表名
set 列=新值,列=新值,...
where 筛选条件;

2.修改多表的记录【补充】

语法：
sql92语法：
update 表1 别名,表2 别名
set 列=值,...
where 连接条件
and 筛选条件;

sql99语法：
update 表1 别名
inner|left|right join 表2 别名
on 连接条件
set 列=值,...
where 筛选条件;


*/


#1.修改单表的记录
#案例1：修改beauty表中姓唐的女神的电话为13899888899

UPDATE beauty SET phone = '13899888899'
WHERE NAME LIKE '唐%';

#案例2：修改boys表中id好为2的名称为张飞，魅力值 10
UPDATE boys SET boyname='张飞',usercp=10
WHERE id=2;



#2.修改多表的记录

#案例 1：修改张无忌的女朋友的手机号为114

UPDATE boys bo
INNER JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`phone`='119',bo.`userCP`=1000
WHERE bo.`boyName`='张无忌';



#案例2：修改没有男朋友的女神的男朋友编号都为2号

UPDATE boys bo
RIGHT JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`boyfriend_id`=2
WHERE bo.`id` IS NULL;

SELECT * FROM boys;


#三、删除语句
/*

方式一：delete
语法：

1、单表的删除【★】
delete from 表名 where 筛选条件

2、多表的删除【补充】

sql92语法：
delete 表1的别名,表2的别名
from 表1 别名,表2 别名
where 连接条件
and 筛选条件;

sql99语法：

delete 表1的别名,表2的别名
from 表1 别名
inner|left|right join 表2 别名 on 连接条件
where 筛选条件;



方式二：truncate
语法：truncate table 表名;

*/

#方式一：delete
#1.单表的删除
#案例：删除手机号以9结尾的女神信息

DELETE FROM beauty WHERE phone LIKE '%9';
SELECT * FROM beauty;


#2.多表的删除

#案例：删除张无忌的女朋友的信息

DELETE b
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id` = bo.`id`
WHERE bo.`boyName`='张无忌';


#案例：删除黄晓明的信息以及他女朋友的信息
DELETE b,bo
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id`=bo.`id`
WHERE bo.`boyName`='黄晓明';



#方式二：truncate语句

#案例：将魅力值>100的男神信息删除
TRUNCATE TABLE boys ;



#delete pk truncate【面试题★】

/*

1.delete 可以加where 条件，truncate不能加

2.truncate删除，效率高一丢丢
3.假如要删除的表中有自增长列，
如果用delete删除后，再插入数据，自增长列的值从断点开始，
而truncate删除后，再插入数据，自增长列的值从1开始。
4.truncate删除没有返回值，delete删除有返回值

5.truncate删除不能回滚，delete删除可以回滚.

*/

SELECT * FROM boys;

DELETE FROM boys;
TRUNCATE TABLE boys;
INSERT INTO boys (boyname,usercp)
VALUES('张飞',100),('刘备',100),('关云长',100);

```

## 7.库和表管理

```mysql
#DDL
/*

数据定义语言

库和表的管理

一、库的管理
创建、修改、删除
二、表的管理
创建、修改、删除

创建： create
修改： alter
删除： drop

*/

#一、库的管理
#1、库的创建
/*
语法：
create database  [if not exists]库名;
*/


#案例：创建库Books

CREATE DATABASE IF NOT EXISTS books ;


#2、库的修改

RENAME DATABASE books TO 新库名;

#更改库的字符集

ALTER DATABASE books CHARACTER SET gbk;


#3、库的删除

DROP DATABASE IF EXISTS books;




#二、表的管理
#1.表的创建 ★

/*
语法：
create table 表名(
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	...
	列名 列的类型【(长度) 约束】


)


*/
#案例：创建表Book

CREATE TABLE book(
	id INT,#编号
	bName VARCHAR(20),#图书名
	price DOUBLE,#价格
	authorId  INT,#作者编号
	publishDate DATETIME#出版日期



);


DESC book;

#案例：创建表author
CREATE TABLE IF NOT EXISTS author(
	id INT,
	au_name VARCHAR(20),
	nation VARCHAR(10)

)
DESC author;


#2.表的修改

/*
语法
alter table 表名 add|drop|modify|change column 列名 【列类型 约束】;
add 添加列
drop 删除列
modify 修改 类型约束
change 更改
*/

#①修改列名

ALTER TABLE book CHANGE COLUMN publishdate pubDate DATETIME;


#②修改列的类型或约束
ALTER TABLE book MODIFY COLUMN pubdate TIMESTAMP;

#③添加新列
ALTER TABLE author ADD COLUMN annual DOUBLE; 

#④删除列
ALTER TABLE book_author DROP COLUMN  annual;
#⑤修改表名

ALTER TABLE author RENAME TO book_author;

DESC book;




#3.表的删除

DROP TABLE IF EXISTS book_author;

SHOW TABLES;


#通用的写法：

DROP DATABASE IF EXISTS 旧库名;
CREATE DATABASE 新库名;


DROP TABLE IF EXISTS 旧表名;
CREATE TABLE  表名();



#4.表的复制

INSERT INTO author VALUES
(1,'村上春树','日本'),
(2,'莫言','中国'),
(3,'冯唐','中国'),
(4,'金庸','中国');

SELECT * FROM Author;
SELECT * FROM copy2;
#1.仅仅复制表的结构

CREATE TABLE copy LIKE author;

#2.复制表的结构+数据
CREATE TABLE copy2 
SELECT * FROM author;

#只复制部分数据
CREATE TABLE copy3
SELECT id,au_name
FROM author 
WHERE nation='中国';


#仅仅复制某些字段

CREATE TABLE copy4 
SELECT id,au_name
FROM author
WHERE 0;

```

### 7.2库和表习题

```mysql
#1.	创建表dept1
NAME	NULL?	TYPE
id		INT(7)
NAME		VARCHAR(25)


USE test;

CREATE TABLE dept1(
	id INT(7),
	NAME VARCHAR(25)
	

);
#2.	将表departments中的数据插入新表dept2中

CREATE TABLE dept2
SELECT department_id,department_name
FROM myemployees.departments;


#3.	创建表emp5
NAME	NULL?	TYPE
id		INT(7)
First_name	VARCHAR (25)
Last_name	VARCHAR(25)
Dept_id		INT(7)

CREATE TABLE emp5(
id INT(7),
first_name VARCHAR(25),
last_name VARCHAR(25),
dept_id INT(7)

);


#4.	将列Last_name的长度增加到50

ALTER TABLE emp5 MODIFY COLUMN last_name VARCHAR(50);
#5.	根据表employees创建employees2

CREATE TABLE employees2 LIKE myemployees.employees;

#6.	删除表emp5
DROP TABLE IF EXISTS emp5;

#7.	将表employees2重命名为emp5

ALTER TABLE employees2 RENAME TO emp5;

#8.在表dept和emp5中添加新列test_column，并检查所作的操作

ALTER TABLE emp5 ADD COLUMN test_column INT;
#9.直接删除表emp5中的列 dept_id
DESC emp5;
ALTER TABLE emp5 DROP COLUMN test_column;
```



## 8.常用数据类型

```mysql
#常见的数据类型
/*
数值型：
	整型
	小数：
		定点数
		浮点数
字符型：
	较短的文本：char、varchar
	较长的文本：text、blob（较长的二进制数据）

日期型：
	


*/

#一、整型
/*
分类：
tinyint、smallint、mediumint、int/integer、bigint
1	 2		3	4		8

特点：
① 如果不设置无符号还是有符号，默认是有符号，如果想设置无符号，需要添加unsigned关键字
② 如果插入的数值超出了整型的范围,会报out of range异常，并且插入临界值
③ 如果不设置长度，会有默认的长度
长度代表了显示的最大宽度，如果不够会用0在左边填充，但必须搭配zerofill使用！

*/

#1.如何设置无符号和有符号

DROP TABLE IF EXISTS tab_int;
CREATE TABLE tab_int(
	t1 INT(7) ZEROFILL,
	t2 INT(7) ZEROFILL 

);

DESC tab_int;


INSERT INTO tab_int VALUES(-123456);
INSERT INTO tab_int VALUES(-123456,-123456);
INSERT INTO tab_int VALUES(2147483648,4294967296);

INSERT INTO tab_int VALUES(123,123);


SELECT * FROM tab_int;


#二、小数
/*
分类：
1.浮点型
float(M,D)
double(M,D)
2.定点型
dec(M，D)
decimal(M,D)

特点：

①
M：整数部位+小数部位
D：小数部位
如果超过范围，则插入临界值

②
M和D都可以省略
如果是decimal，则M默认为10，D默认为0
如果是float和double，则会根据插入的数值的精度来决定精度

③定点型的精确度较高，如果要求插入数值的精度较高如货币运算等则考虑使用


*/
#测试M和D

DROP TABLE tab_float;
CREATE TABLE tab_float(
	f1 FLOAT,
	f2 DOUBLE,
	f3 DECIMAL
);
SELECT * FROM tab_float;
DESC tab_float;

INSERT INTO tab_float VALUES(123.4523,123.4523,123.4523);
INSERT INTO tab_float VALUES(123.456,123.456,123.456);
INSERT INTO tab_float VALUES(123.4,123.4,123.4);
INSERT INTO tab_float VALUES(1523.4,1523.4,1523.4);



#原则：
/*
所选择的类型越简单越好，能保存数值的类型越小越好

*/

#三、字符型
/*
较短的文本：

char
varchar

其他：

binary和varbinary用于保存较短的二进制
enum用于保存枚举
set用于保存集合


较长的文本：
text
blob(较大的二进制)

特点：



	写法		M的意思					特点			空间的耗费					效率
char	char(M)		最大的字符数，可以省略，默认为1		固定长度的字符		比较耗费	高

varchar varchar(M)	最大的字符数，不可以省略		可变长度的字符		比较节省	低
*/



CREATE TABLE tab_char(
	c1 ENUM('a','b','c')


);


INSERT INTO tab_char VALUES('a');
INSERT INTO tab_char VALUES('b');
INSERT INTO tab_char VALUES('c');
INSERT INTO tab_char VALUES('m');
INSERT INTO tab_char VALUES('A');

SELECT * FROM tab_set;



CREATE TABLE tab_set(

	s1 SET('a','b','c','d')



);
INSERT INTO tab_set VALUES('a');
INSERT INTO tab_set VALUES('A,B');
INSERT INTO tab_set VALUES('a,c,d');


#四、日期型

/*

分类：
date只保存日期
time 只保存时间
year只保存年

datetime保存日期+时间
timestamp保存日期+时间


特点：

			字节		范围							时区等的影响
datetime	8			1000——9999	                    不受
timestamp	4	    	1970-2038	                    受

*/


CREATE TABLE tab_date(
	t1 DATETIME,
	t2 TIMESTAMP

);



INSERT INTO tab_date VALUES(NOW(),NOW());

SELECT * FROM tab_date;


SHOW VARIABLES LIKE 'time_zone';

SET time_zone='+9:00';
```

## 9.常用约束

```mysql
#常见约束

/*


含义：一种限制，用于限制表中的数据，为了保证表中的数据的准确和可靠性


分类：六大约束
	NOT NULL：非空，用于保证该字段的值不能为空
	比如姓名、学号等
	DEFAULT:默认，用于保证该字段有默认值
	比如性别
	PRIMARY KEY:主键，用于保证该字段的值具有唯一性，并且非空
	比如学号、员工编号等
	UNIQUE:唯一，用于保证该字段的值具有唯一性，可以为空
	比如座位号
	CHECK:检查约束【mysql中不支持】
	比如年龄、性别
	FOREIGN KEY:外键，用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值
		在从表添加外键约束，用于引用主表中某列的值
	比如学生表的专业编号，员工表的部门编号，员工表的工种编号
	

添加约束的时机：
	1.创建表时
	2.修改表时
	

约束的添加分类：
	列级约束：
		六大约束语法上都支持，但外键约束没有效果
		
	表级约束：
		
		除了非空、默认，其他的都支持
		
		
主键和唯一的大对比：

			保证唯一性  是否允许为空    一个表中可以有多少个   是否允许组合
	主键	√				×		        至多有1个           √，但不推荐
	唯一	√				√		        可以有多个          √，但不推荐
外键：
	1、要求在从表设置外键关系
	2、从表的外键列的类型和主表的关联列的类型要求一致或兼容，名称无要求
	3、主表的关联列必须是一个key（一般是主键或唯一）
	4、插入数据时，先插入主表，再插入从表
	删除数据时，先删除从表，再删除主表


*/

CREATE TABLE 表名(
	字段名 字段类型 列级约束,
	字段名 字段类型,
	表级约束

)
CREATE DATABASE students;
#一、创建表时添加约束

#1.添加列级约束
/*
语法：

直接在字段名和类型后面追加 约束类型即可。

只支持：默认、非空、主键、唯一
*/

USE students;
DROP TABLE stuinfo;
CREATE TABLE stuinfo(
	id INT PRIMARY KEY,#主键
	stuName VARCHAR(20) NOT NULL UNIQUE,#非空
	gender CHAR(1) CHECK(gender='男' OR gender ='女'),#检查
	seat INT UNIQUE,#唯一
	age INT DEFAULT  18,#默认约束
	majorId INT REFERENCES major(id)#外键

);

CREATE TABLE major(
	id INT PRIMARY KEY,
	majorName VARCHAR(20)
);

#查看stuinfo中的所有索引，包括主键、外键、唯一
SHOW INDEX FROM stuinfo;


#2.添加表级约束
/*

语法：在各个字段的最下面
 【constraint 约束名】 约束类型(字段名) 
*/

DROP TABLE IF EXISTS stuinfo;
CREATE TABLE stuinfo(
	id INT,
	stuname VARCHAR(20),
	gender CHAR(1),
	seat INT,
	age INT,
	majorid INT,
	
	CONSTRAINT pk PRIMARY KEY(id),#主键
	CONSTRAINT uq UNIQUE(seat),#唯一键
	CONSTRAINT ck CHECK(gender ='男' OR gender  = '女'),#检查
	CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)#外键
	
    #references 指向那个表的的字段
    #foreign key 外键
);

SHOW INDEX FROM stuinfo;

#通用的写法：★

CREATE TABLE IF NOT EXISTS stuinfo(
	id INT PRIMARY KEY,
	stuname VARCHAR(20),
	sex CHAR(1),
	age INT DEFAULT 18,
	seat INT UNIQUE,
	majorid INT,
	CONSTRAINT fk_stuinfo_major 
	KEY(majorid) REFERENCES major(id)

);



#二、修改表时添加约束

/*
1、添加列级约束
alter table 表名 modify column 字段名 字段类型 新约束;

2、添加表级约束
alter table 表名 add 【constraint 约束名】 约束类型(字段名) 【外键的引用】;


*/
DROP TABLE IF EXISTS stuinfo;
CREATE TABLE stuinfo(
	id INT,
	stuname VARCHAR(20),
	gender CHAR(1),
	seat INT,
	age INT,
	majorid INT
)
DESC stuinfo;
#1.添加非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20)  NOT NULL;
#2.添加默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT DEFAULT 18;
#3.添加主键
#①列级约束
ALTER TABLE stuinfo MODIFY COLUMN id INT PRIMARY KEY;
#②表级约束
ALTER TABLE stuinfo ADD PRIMARY KEY(id);

#4.添加唯一

#①列级约束
ALTER TABLE stuinfo MODIFY COLUMN seat INT UNIQUE;
#②表级约束
ALTER TABLE stuinfo ADD UNIQUE(seat);


#5.添加外键
ALTER TABLE stuinfo ADD CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id); 

#三、修改表时删除约束

#1.删除非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20) NULL;

#2.删除默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT ;

#3.删除主键
ALTER TABLE stuinfo DROP PRIMARY KEY;

#4.删除唯一
ALTER TABLE stuinfo DROP INDEX seat;

#5.删除外键
ALTER TABLE stuinfo DROP FOREIGN KEY fk_stuinfo_major;

SHOW INDEX FROM stuinfo;

```

### 9.2常用约束例子

```mysql
#1.向表emp2的id列中添加PRIMARY KEY约束（my_emp_id_pk）

ALTER TABLE emp2 MODIFY COLUMN id INT PRIMARY KEY;
ALTER TABLE emp2 ADD CONSTRAINT my_emp_id_pk PRIMARY KEY(id);

#2.	向表dept2的id列中添加PRIMARY KEY约束（my_dept_id_pk）

#3.	向表emp2中添加列dept_id，并在其中定义FOREIGN KEY约束，与之相关联的列是dept2表中的id列。
ALTER TABLE emp2 ADD COLUMN dept_id INT;
ALTER TABLE emp2 ADD CONSTRAINT fk_emp2_dept2 FOREIGN KEY(dept_id) REFERENCES dept2(id);

		位置		支持的约束类型			是否可以起约束名
列级约束：	列的后面	语法都支持，但外键没有效果	不可以
表级约束：	所有列的下面	默认和非空不支持，其他支持	可以（主键没有效果）
```

## 10.自增列

```mysql
#标识列
/*
又称为自增长列
含义：可以不用手动的插入值，系统提供默认的序列值

特点：
1、标识列必须和主键搭配吗？不一定，但要求是一个key
2、一个表可以有几个标识列？至多一个！
3、标识列的类型只能是数值型
4、标识列可以通过 SET auto_increment_increment=3;设置步长
可以通过 手动插入值，设置起始值
*/

#一、创建表时设置标识列

DROP TABLE IF EXISTS tab_identity;
CREATE TABLE tab_identity(
	id INT  ,
	NAME FLOAT UNIQUE AUTO_INCREMENT,
	seat INT 
);
TRUNCATE TABLE tab_identity;

INSERT INTO tab_identity(id,NAME) VALUES(NULL,'john');
INSERT INTO tab_identity(NAME) VALUES('lucy');
SELECT * FROM tab_identity;

SHOW VARIABLES LIKE '%auto_increment%';

SET auto_increment_increment=3;
```

## 11.事务

```mysql
#TCL
/*
Transaction Control Language 事务控制语言

事务：
一个或一组sql语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行。

案例：转账

张三丰  1000
郭襄	1000

update 表 set 张三丰的余额=500 where name='张三丰'
意外
update 表 set 郭襄的余额=1500 where name='郭襄'


事务的特性：
ACID
原子性：一个事务不可再分割，要么都执行要么都不执行
一致性：一个事务执行会使数据从一个一致状态切换到另外一个一致状态
隔离性：一个事务的执行不受其他事务的干扰
持久性：一个事务一旦提交，则会永久的改变数据库的数据.



事务的创建
隐式事务：事务没有明显的开启和结束的标记
比如insert、update、delete语句

delete from 表 where id =1;

显式事务：事务具有明显的开启和结束的标记
前提：必须先设置自动提交功能为禁用

set autocommit=0;

步骤1：开启事务
set autocommit=0;
start transaction;可选的
步骤2：编写事务中的sql语句(select insert update delete)
语句1;
语句2;
...

步骤3：结束事务
commit;提交事务
rollback;回滚事务

savepoint 节点名;设置保存点



事务的隔离级别：
		  脏读		不可重复读	幻读
read uncommitted：√		√		√
read committed：  ×		√		√
repeatable read： ×		×		√
serializable	  ×             ×               ×


mysql中默认 第三个隔离级别 repeatable read
oracle中默认第二个隔离级别 read committed
查看隔离级别
select @@tx_isolation;
设置隔离级别
set session|global transaction isolation level 隔离级别;




开启事务的语句;
update 表 set 张三丰的余额=500 where name='张三丰'

update 表 set 郭襄的余额=1500 where name='郭襄' 
结束事务的语句;



*/

SHOW VARIABLES LIKE 'autocommit';
SHOW ENGINES;

#1.演示事务的使用步骤

#开启事务
SET autocommit=0;
START TRANSACTION;
#编写一组事务的语句
UPDATE account SET balance = 1000 WHERE username='张无忌';
UPDATE account SET balance = 1000 WHERE username='赵敏';

#结束事务
ROLLBACK;
#commit;

SELECT * FROM account;


#2.演示事务对于delete和truncate的处理的区别

SET autocommit=0;
START TRANSACTION;

DELETE FROM account;
ROLLBACK;

#3.演示savepoint 的使用
SET autocommit=0;
START TRANSACTION;
DELETE FROM account WHERE id=25;
SAVEPOINT a;#设置保存点
DELETE FROM account WHERE id=28;
ROLLBACK TO a;#回滚到保存点


SELECT * FROM account;

```

## 12.视图

```mysql
#视图
/*
含义：虚拟表，和普通表一样使用
mysql5.1版本出现的新特性，是通过表动态生成的数据

比如：舞蹈班和普通班级的对比
	创建语法的关键字	是否实际占用物理空间	使用

视图	create view		只是保存了sql逻辑	增删改查，只是一般不能增删改

表	create table		保存了数据		增删改查


*/

#案例：查询姓张的学生名和专业名
SELECT stuname,majorname
FROM stuinfo s
INNER JOIN major m ON s.`majorid`= m.`id`
WHERE s.`stuname` LIKE '张%';

CREATE VIEW v1
AS
SELECT stuname,majorname
FROM stuinfo s
INNER JOIN major m ON s.`majorid`= m.`id`;

SELECT * FROM v1 WHERE stuname LIKE '张%';


#一、创建视图
/*
语法：
create view 视图名
as
查询语句;

*/
USE myemployees;

#1.查询姓名中包含a字符的员工名、部门名和工种信息
#①创建
CREATE VIEW myv1
AS

SELECT last_name,department_name,job_title
FROM employees e
JOIN departments d ON e.department_id  = d.department_id
JOIN jobs j ON j.job_id  = e.job_id;


#②使用
SELECT * FROM myv1 WHERE last_name LIKE '%a%';






#2.查询各部门的平均工资级别

#①创建视图查看每个部门的平均工资
CREATE VIEW myv2
AS
SELECT AVG(salary) ag,department_id
FROM employees
GROUP BY department_id;

#②使用
SELECT myv2.`ag`,g.grade_level
FROM myv2
JOIN job_grades g
ON myv2.`ag` BETWEEN g.`lowest_sal` AND g.`highest_sal`;



#3.查询平均工资最低的部门信息

SELECT * FROM myv2 ORDER BY ag LIMIT 1;

#4.查询平均工资最低的部门名和工资

CREATE VIEW myv3
AS
SELECT * FROM myv2 ORDER BY ag LIMIT 1;


SELECT d.*,m.ag
FROM myv3 m
JOIN departments d
ON m.`department_id`=d.`department_id`;




#二、视图的修改

#方式一：
/*
create or replace view  视图名
as
查询语句;

*/
SELECT * FROM myv3 

CREATE OR REPLACE VIEW myv3
AS
SELECT AVG(salary),job_id
FROM employees
GROUP BY job_id;

#方式二：
/*
语法：
alter view 视图名
as 
查询语句;

*/
ALTER VIEW myv3
AS
SELECT * FROM employees;

#三、删除视图

/*

语法：drop view 视图名,视图名,...;
*/

DROP VIEW emp_v1,emp_v2,myv3;


#四、查看视图

DESC myv3;

SHOW CREATE VIEW myv3;


#五、视图的更新

CREATE OR REPLACE VIEW myv1
AS
SELECT last_name,email,salary*12*(1+IFNULL(commission_pct,0)) "annual salary"
FROM employees;

CREATE OR REPLACE VIEW myv1
AS
SELECT last_name,email
FROM employees;


SELECT * FROM myv1;
SELECT * FROM employees;
#1.插入

INSERT INTO myv1 VALUES('张飞','zf@qq.com');

#2.修改
UPDATE myv1 SET last_name = '张无忌' WHERE last_name='张飞';

#3.删除
DELETE FROM myv1 WHERE last_name = '张无忌';

#具备以下特点的视图不允许更新


#①包含以下关键字的sql语句：分组函数、distinct、group  by、having、union或者union all

CREATE OR REPLACE VIEW myv1
AS
SELECT MAX(salary) m,department_id
FROM employees
GROUP BY department_id;

SELECT * FROM myv1;

#更新
UPDATE myv1 SET m=9000 WHERE department_id=10;

#②常量视图
CREATE OR REPLACE VIEW myv2
AS

SELECT 'john' NAME;

SELECT * FROM myv2;

#更新
UPDATE myv2 SET NAME='lucy';





#③Select中包含子查询

CREATE OR REPLACE VIEW myv3
AS

SELECT department_id,(SELECT MAX(salary) FROM employees) 最高工资
FROM departments;

#更新
SELECT * FROM myv3;
UPDATE myv3 SET 最高工资=100000;


#④join
CREATE OR REPLACE VIEW myv4
AS

SELECT last_name,department_name
FROM employees e
JOIN departments d
ON e.department_id  = d.department_id;

#更新

SELECT * FROM myv4;
UPDATE myv4 SET last_name  = '张飞' WHERE last_name='Whalen';
INSERT INTO myv4 VALUES('陈真','xxxx');



#⑤from一个不能更新的视图
CREATE OR REPLACE VIEW myv5
AS

SELECT * FROM myv3;

#更新

SELECT * FROM myv5;

UPDATE myv5 SET 最高工资=10000 WHERE department_id=60;



#⑥where子句的子查询引用了from子句中的表

CREATE OR REPLACE VIEW myv6
AS

SELECT last_name,email,salary
FROM employees
WHERE employee_id IN(
	SELECT  manager_id
	FROM employees
	WHERE manager_id IS NOT NULL
);

#更新
SELECT * FROM myv6;
UPDATE myv6 SET salary=10000 WHERE last_name = 'k_ing';
```

### 12.1视图习题

```mysql
#一、创建视图emp_v1,要求查询电话号码以‘011’开头的员工姓名和工资、邮箱

CREATE OR REPLACE VIEW emp_v1
AS 
SELECT last_name,salary,email,phone_number
FROM employees
WHERE phone_number LIKE '515%'

#二、创建视图emp_v2，要求查询部门的最高工资高于12000的部门信息


CREATE OR REPLACE VIEW emp_v2
AS

SELECT MAX(salary) ma,department_id
FROM employees
GROUP BY department_id
HAVING MAX(salary) > 12000

SELECT d.*,m.mx_dep
FROM departments d
JOIN emp_v2 m
ON m.department_id = d.department_id

--------------------------------
# 创建book表 
bid 整型
bname 字符型 唯一值
price 浮点型
btypeid 类型编号
 
CREATE TABLE Book(
	bid INT PRIMARY KEY,
	bname VARCHAR(20) UNIQUE NOT NULL,
	price FLOAT DEFAULT 10,
	btypeid INT,
	FOREIGN KEY(btypeid) REFERENCES booType(id)
)

#开启事务插入一行记录 并结束
SET autocommit = 0;
INSERT INTO book(bid,bname,price,btypeid)
vaues(1,'小李',100,1)

ROLLBACK;

#创建视图 实现查询价格大于100的书名和类型名
CREATE VIEW myv1
AS 
SELECT bname,NAME
FROM book b
JOIN bookType t ON b.typeid = t.id
WHERE price > 100

#修改视图 实现查询价格在90-120之间的书名和价格
CREATE OR REPLACE VIEW myv1
AS
SELECT bname,price
FROM book 
WHERE price BETWEEN 90 AND 100

#删除刚刚创建的视图
DROP VIEW myv1
```

## 13变量

```mysql
#变量
/*
系统变量：
	全局变量
	会话变量

自定义变量：
	用户变量
	局部变量

*/
#一、系统变量
/*
说明：变量由系统定义，不是用户定义，属于服务器层面
注意：全局变量需要添加global关键字，会话变量需要添加session关键字，如果不写，默认会话级别
使用步骤：
1、查看所有系统变量
show global|【session】variables;
2、查看满足条件的部分系统变量
show global|【session】 variables like '%char%';
3、查看指定的系统变量的值
select @@global|【session】系统变量名;
4、为某个系统变量赋值
方式一：
set global|【session】系统变量名=值;
方式二：
set @@global|【session】系统变量名=值;

*/
#1》全局变量
/*
作用域：针对于所有会话（连接）有效，但不能跨重启
*/
#①查看所有全局变量
SHOW GLOBAL VARIABLES;
#②查看满足条件的部分系统变量
SHOW GLOBAL VARIABLES LIKE '%char%';
#③查看指定的系统变量的值
SELECT @@global.autocommit;
#④为某个系统变量赋值
SET @@global.autocommit=0;
SET GLOBAL autocommit=0;

#2》会话变量
/*
作用域：针对于当前会话（连接）有效
*/
#①查看所有会话变量
SHOW SESSION VARIABLES;
#②查看满足条件的部分会话变量
SHOW SESSION VARIABLES LIKE '%char%';
#③查看指定的会话变量的值
SELECT @@autocommit;
SELECT @@session.tx_isolation;
#④为某个会话变量赋值
SET @@session.tx_isolation='read-uncommitted';
SET SESSION tx_isolation='read-committed';

#二、自定义变量
/*
说明：变量由用户自定义，而不是系统提供的
使用步骤：
1、声明
2、赋值
3、使用（查看、比较、运算等）
*/

#1》用户变量
/*
作用域：针对于当前会话（连接）有效，作用域同于会话变量
*/

#赋值操作符：=或:=
#①声明并初始化
SET @变量名=值;
SET @变量名:=值;
SELECT @变量名:=值;

#②赋值（更新变量的值）
#方式一：
	SET @变量名=值;
	SET @变量名:=值;
	SELECT @变量名:=值;
#方式二：
	SELECT 字段 INTO @变量名
	FROM 表;
#③使用（查看变量的值）
SELECT @变量名;


#2》局部变量
/*
作用域：仅仅在定义它的begin end块中有效
应用在 begin end中的第一句话
*/

#①声明
DECLARE 变量名 类型;
DECLARE 变量名 类型 【DEFAULT 值】;


#②赋值（更新变量的值）

#方式一：
	SET 局部变量名=值;
	SET 局部变量名:=值;
	SELECT 局部变量名:=值;
#方式二：
	SELECT 字段 INTO 具备变量名
	FROM 表;
#③使用（查看变量的值）
SELECT 局部变量名;


#案例：声明两个变量，求和并打印

#用户变量
SET @m=1;
SET @n=1;
SET @sum=@m+@n;
SELECT @sum;

#局部变量
DECLARE m INT DEFAULT 1;
DECLARE n INT DEFAULT 1;
DECLARE SUM INT;
SET SUM=m+n;
SELECT SUM;


#用户变量和局部变量的对比

			作用域					定义位置				语法
用户变量	当前会话				会话的任何地方			加@符号，不用指定类型
局部变量	定义它的BEGIN END中 	BEGIN END的第一句话		一般不用加@,需要指定类型
			
```



## 14.函数

```mysql
#函数
/*
含义：一组预先编译好的SQL语句的集合，理解成批处理语句
1、提高代码的重用性
2、简化操作
3、减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率

区别：

存储过程：可以有0个返回，也可以有多个返回，适合做批量插入、批量更新
函数：有且仅有1 个返回，适合做处理数据后返回一个结果

*/

#一、创建语法
CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型
BEGIN
	函数体
END
/*

注意：
1.参数列表 包含两部分：
参数名 参数类型

2.函数体：肯定会有return语句，如果没有会报错
如果return语句没有放在函数体的最后也不报错，但不建议

return 值;
3.函数体中仅有一句话，则可以省略begin end
4.使用 delimiter语句设置结束标记

*/

#二、调用语法
SELECT 函数名(参数列表)


#------------------------------案例演示----------------------------
#1.无参有返回
#案例：返回公司的员工个数
CREATE FUNCTION myf1() RETURNS INT
BEGIN

	DECLARE c INT DEFAULT 0;#定义局部变量
	SELECT COUNT(*) INTO c#赋值
	FROM employees;
	RETURN c;
	
END $

SELECT myf1()$


#2.有参有返回
#案例1：根据员工名，返回它的工资

CREATE FUNCTION myf2(empName VARCHAR(20)) RETURNS DOUBLE
BEGIN
	SET @sal=0;#定义用户变量 
	SELECT salary INTO @sal   #赋值
	FROM employees
	WHERE last_name = empName;
	
	RETURN @sal;
END $

SELECT myf2('k_ing') $

#案例2：根据部门名，返回该部门的平均工资

CREATE FUNCTION myf3(deptName VARCHAR(20)) RETURNS DOUBLE
BEGIN
	DECLARE sal DOUBLE ;
	SELECT AVG(salary) INTO sal
	FROM employees e
	JOIN departments d ON e.department_id = d.department_id
	WHERE d.department_name=deptName;
	RETURN sal;
END $

SELECT myf3('IT')$

#三、查看函数

SHOW CREATE FUNCTION myf3;

#四、删除函数
DROP FUNCTION myf3;

#案例
#一、创建函数，实现传入两个float，返回二者之和

CREATE FUNCTION test_fun1(num1 FLOAT,num2 FLOAT) RETURNS FLOAT
BEGIN
	DECLARE SUM FLOAT DEFAULT 0;
	SET SUM=num1+num2;
	RETURN SUM;
END $

SELECT test_fun1(1,2)$
```

15.存储过程

```mysql
#存储过程和函数
/*
存储过程和函数：类似于java中的方法
好处：
1、提高代码的重用性
2、简化操作



*/
#存储过程
/*
含义：一组预先编译好的SQL语句的集合，理解成批处理语句
1、提高代码的重用性
2、简化操作
3、减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率



*/

#一、创建语法

CREATE PROCEDURE 存储过程名(参数列表)
BEGIN

	存储过程体（一组合法的SQL语句）
END

#注意：
/*
1、参数列表包含三部分
参数模式  参数名  参数类型
举例：
in stuname varchar(20)

参数模式：
in：该参数可以作为输入，也就是该参数需要调用方传入值
out：该参数可以作为输出，也就是该参数可以作为返回值
inout：该参数既可以作为输入又可以作为输出，也就是该参数既需要传入值，又可以返回值

2、如果存储过程体仅仅只有一句话，begin end可以省略
存储过程体中的每条sql语句的结尾要求必须加分号。
存储过程的结尾可以使用 delimiter 重新设置
语法：
delimiter 结束标记
案例：
delimiter $
*/


#二、调用语法

CALL 存储过程名(实参列表);

#--------------------------------案例演示-----------------------------------
#1.空参列表
#案例：插入到admin表中五条记录

SELECT * FROM admin;

DELIMITER $
CREATE PROCEDURE myp1()
BEGIN
	INSERT INTO admin(username,`password`) 
	VALUES('john1','0000'),('lily','0000'),('rose','0000'),('jack','0000'),('tom','0000');
END $


#调用
CALL myp1()$

#2.创建带in模式参数的存储过程

#案例1：创建存储过程实现 根据女神名，查询对应的男神信息

CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))
BEGIN
	SELECT bo.*
	FROM boys bo
	RIGHT JOIN beauty b ON bo.id = b.boyfriend_id
	WHERE b.name=beautyName;
	

END $

#调用
CALL myp2('柳岩')$

#案例2 ：创建存储过程实现，用户是否登录成功

CREATE PROCEDURE myp4(IN username VARCHAR(20),IN PASSWORD VARCHAR(20))
BEGIN
	DECLARE result INT DEFAULT 0;#声明并初始化
	
	SELECT COUNT(*) INTO result#赋值
	FROM admin
	WHERE admin.username = username
	AND admin.password = PASSWORD;
	
	SELECT IF(result>0,'成功','失败');#使用
END $

#调用
CALL myp3('张飞','8888')$


#3.创建out 模式参数的存储过程
#案例1：根据输入的女神名，返回对应的男神名

CREATE PROCEDURE myp6(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20))
BEGIN
	SELECT bo.boyname INTO boyname
	FROM boys bo
	RIGHT JOIN
	beauty b ON b.boyfriend_id = bo.id
	WHERE b.name=beautyName ;
	
END $


#案例2：根据输入的女神名，返回对应的男神名和魅力值

CREATE PROCEDURE myp7(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20),OUT usercp INT) 
BEGIN
	SELECT boys.boyname ,boys.usercp INTO boyname,usercp
	FROM boys 
	RIGHT JOIN
	beauty b ON b.boyfriend_id = boys.id
	WHERE b.name=beautyName ;
	
END $


#调用
CALL myp7('小昭',@name,@cp)$
SELECT @name,@cp$



#4.创建带inout模式参数的存储过程
#案例1：传入a和b两个值，最终a和b都翻倍并返回

CREATE PROCEDURE myp8(INOUT a INT ,INOUT b INT)
BEGIN
	SET a=a*2;
	SET b=b*2;
END $

#调用
SET @m=10$
SET @n=20$
CALL myp8(@m,@n)$
SELECT @m,@n$


#三、删除存储过程
#语法：drop procedure 存储过程名
DROP PROCEDURE p1;
DROP PROCEDURE p2,p3;#×

#四、查看存储过程的信息
DESC myp2;×
SHOW CREATE PROCEDURE  myp2;
```

## 15.流程控制

```mysql
#流程控制结构
/*
顺序、分支、循环

*/

#一、分支结构
#1.if函数
/*
语法：if(条件,值1，值2)
功能：实现双分支
应用在begin end中或外面

*/

#2.case结构
/*
语法：
情况1：类似于switch
case 变量或表达式
when 值1 then 语句1;
when 值2 then 语句2;
...
else 语句n;
end 

情况2：
case 
when 条件1 then 语句1;
when 条件2 then 语句2;
...
else 语句n;
end 

应用在begin end 中或外面


*/

#3.if结构

/*
语法：
if 条件1 then 语句1;
elseif 条件2 then 语句2;
....
else 语句n;
end if;
功能：类似于多重if

只能应用在begin end 中

*/

#案例1：创建函数，实现传入成绩，如果成绩>90,返回A，如果成绩>80,返回B，如果成绩>60,返回C，否则返回D

CREATE FUNCTION test_if(score FLOAT) RETURNS CHAR
BEGIN
	DECLARE ch CHAR DEFAULT 'A';
	IF score>90 THEN SET ch='A';
	ELSEIF score>80 THEN SET ch='B';
	ELSEIF score>60 THEN SET ch='C';
	ELSE SET ch='D';
	END IF;
	RETURN ch;
	
	
END $

SELECT test_if(87)$

#案例2：创建存储过程，如果工资<2000,则删除，如果5000>工资>2000,则涨工资1000，否则涨工资500


CREATE PROCEDURE test_if_pro(IN sal DOUBLE)
BEGIN
	IF sal<2000 THEN DELETE FROM employees WHERE employees.salary=sal;
	ELSEIF sal>=2000 AND sal<5000 THEN UPDATE employees SET salary=salary+1000 WHERE employees.`salary`=sal;
	ELSE UPDATE employees SET salary=salary+500 WHERE employees.`salary`=sal;
	END IF;
	
END $

CALL test_if_pro(2100)$

#案例1：创建函数，实现传入成绩，如果成绩>90,返回A，如果成绩>80,返回B，如果成绩>60,返回C，否则返回D

CREATE FUNCTION test_case(score FLOAT) RETURNS CHAR
BEGIN 
	DECLARE ch CHAR DEFAULT 'A';
	
	CASE 
	WHEN score>90 THEN SET ch='A';
	WHEN score>80 THEN SET ch='B';
	WHEN score>60 THEN SET ch='C';
	ELSE SET ch='D';
	END CASE;
	
	RETURN ch;
END $

SELECT test_case(56)$



#二、循环结构
/*
分类：
while、loop、repeat

循环控制：

iterate类似于 continue，继续，结束本次循环，继续下一次
leave 类似于  break，跳出，结束当前所在的循环

*/

#1.while
/*

语法：

【标签:】while 循环条件 do
	循环体;
end while【 标签】;

联想：

while(循环条件){

	循环体;
}

*/

#2.loop
/*

语法：
【标签:】loop
	循环体;
end loop 【标签】;

可以用来模拟简单的死循环



*/

#3.repeat
/*
语法：
【标签：】repeat
	循环体;
until 结束循环的条件
end repeat 【标签】;


*/

#1.没有添加循环控制语句
#案例：批量插入，根据次数插入到admin表中多条记录
DROP PROCEDURE pro_while1$
CREATE PROCEDURE pro_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	WHILE i<=insertCount DO
		INSERT INTO admin(username,`password`) VALUES(CONCAT('Rose',i),'666');
		SET i=i+1;
	END WHILE;
	
END $

CALL pro_while1(100)$


/*

int i=1;
while(i<=insertcount){

	//插入
	
	i++;

}

*/


#2.添加leave语句

#案例：批量插入，根据次数插入到admin表中多条记录，如果次数>20则停止
TRUNCATE TABLE admin$
DROP PROCEDURE test_while1$
CREATE PROCEDURE test_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	a:WHILE i<=insertCount DO
		INSERT INTO admin(username,`password`) VALUES(CONCAT('xiaohua',i),'0000');
		IF i>=20 THEN LEAVE a;
		END IF;
		SET i=i+1;
	END WHILE a;
END $


CALL test_while1(100)$


#3.添加iterate语句

#案例：批量插入，根据次数插入到admin表中多条记录，只插入偶数次
TRUNCATE TABLE admin$
DROP PROCEDURE test_while1$
CREATE PROCEDURE test_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 0;
	a:WHILE i<=insertCount DO
		SET i=i+1;
		IF MOD(i,2)!=0 THEN ITERATE a;
		END IF;
		
		INSERT INTO admin(username,`password`) VALUES(CONCAT('xiaohua',i),'0000');
		
	END WHILE a;
END $


CALL test_while1(100)$

/*

int i=0;
while(i<=insertCount){
	i++;
	if(i%2==0){
		continue;
	}
	插入
	
}

*/

```

15.1流程案例

```mysql
/*一、已知表stringcontent
其中字段：
id 自增长
content varchar(20)

向该表插入指定个数的，随机的字符串
*/
DROP TABLE IF EXISTS stringcontent;
CREATE TABLE stringcontent(
	id INT PRIMARY KEY AUTO_INCREMENT,
	content VARCHAR(20)
	
);
DELIMITER $
CREATE PROCEDURE test_randstr_insert(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	DECLARE str VARCHAR(26) DEFAULT 'abcdefghijklmnopqrstuvwxyz';
	DECLARE startIndex INT;#代表初始索引
	DECLARE len INT;#代表截取的字符长度
	WHILE i<=insertcount DO
		SET startIndex=FLOOR(RAND()*26+1);#代表初始索引，随机范围1-26
		SET len=FLOOR(RAND()*(20-startIndex+1)+1);#代表截取长度，随机范围1-（20-startIndex+1）
		INSERT INTO stringcontent(content) VALUES(SUBSTR(str,startIndex,len));
		SET i=i+1;
	END WHILE;

END $

CALL test_randstr_insert(10)$
```

