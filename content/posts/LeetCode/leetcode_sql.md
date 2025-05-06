---
author: "LongWei"
title: "高频SQL50题刷题笔记"
date: "2024-11-24"
tags: ["SQL"]
categories: ["LeetCode"]
series: ["LeetCode"]
aliases: ["leetcode_sql50"]
ShowToc: true
TocOpen: true
---

## 高频SQL50题

### 查询

#### Where&GroupBy&HAVING

```mysql
SELECT id, revenue  # 某个用户2021年可能有多项记录
FROM users
Where year = 2021

SELECT id, SUM(revenue) 
FROM users
Where year = 2021  # 1. 先筛选出此年份所有记录
Group By id        # 2. 再根据id分组 3. 必须结合SUM() 或者其他聚合函数，因此这个有多列，需要处理为单值。

SELECT id
FROM users
Where year = 2021          # 1. 先筛选记录
Group By id                # 2. 分组
HAVING SUM(revenue) > 1000  #  3. 聚合分组字段记录 并 判断
```



#### 查找没买过东西的顾客

```mysql
# 1. NOT EXISTS
SELECT `name` AS `Customers`
FROM `Customers` AS c
WHERE NOT EXISTS (
    SELECT `customerId`
    FROM `Orders` AS o
    Where o.customerId = c.id    
)
```

<img src="http://verification.longcoding.top/FlT4eqfOPSYMmcNCB5mh_bpamx5B" alt="Image" style="float:left; margin-right: 10px;" />

```mysql
# 2. Left Join 
# example
SELECT c.id AS id, c.name, o.id AS oid
FROM customers AS c LEFT JOIN orders AS o
on c.id = o.customerId 

SELECT `name` as `Customers`
FROM customers AS c LEFT JOIN orders AS o
on c.id = o.customerId 
WHERE o.id is NULL
```





#### [计算特殊奖金](https://leetcode.cn/problems/calculate-special-bonus/)

编写解决方案，计算每个雇员的奖金。如果一个雇员的 id 是 **奇数** 并且他的名字不是以 `'M'` **开头**，那么他的奖金是他工资的 `100%` ，否则奖金为 `0` 。

返回的结果按照 `employee_id` 排序。

IF用法  IF(condition, True, False) AS alias

```mysql
select `employee_id`, IF(name not like 'M%' and employee_id%2 = 1, salary, 0) AS `bonus` 
from Employees 
Order by employee_id ASC
```



#### [购买了产品 A 和产品 B 却没有购买产品 C 的顾客](https://leetcode.cn/problems/customers-who-bought-products-a-and-b-but-not-c/)

Where 中 多重 判断

```mysql
select customer_id, customer_name 
from Customers  as c
where exists (
    select o.customer_id
    from Orders as o
    where o.customer_id=c.customer_id and product_name = 'A'
) and exists (
    select o.customer_id
    from Orders as o
    where o.customer_id=c.customer_id and product_name = 'B'
) and not exists (
    select o.customer_id
    from Orders as o
    where o.customer_id=c.customer_id and product_name = 'C'
)
Order By c.customer_id  ASC
```

连表，分组判断

```mysql
SELECT c.customer_id, c.customer_name 
FROM `Orders` as o left join `Customers` as c
on o.customer_id = c.customer_id
Group By o.customer_id
HAVING SUM(product_name = 'A') > 0 and SUM(product_name = 'B') > 0 and SUM(product_name = 'C') = 0
Order By c.customer_id ASC
```

```mysql
[o.id, o.customerId, o.product_name, c.customerId, c.name]
 1       1            'A'              1           X
 2       1            'B'              1           X
 3       2            'A'              2           Y
 4       2            'B'              2           Y
 5       2            'C'              2           Y
 
 Group By c.customerId
 1       1            'A'              1           X
 2       1            'B'              1           X
 
 3       2            'A'              2           Y
 4       2            'B'              2           Y
 5       2            'C'              2           Y
 
 SUM(express condition) ? condition # 对于String是统计条数
```



#### 每位学生成绩最好的科目

所有有相同科目成绩一样，选择科目id小的

```mysql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| course_id     | int     |
| grade         | int     |
+---------------+---------+
```

\# 1

```mysql
# 查某学生成绩最好的那些科目，因为可能有重复的
SELECT student_id, MAX(grade) AS grade
From Enrollments
Group By student_id
```

\# 2

```mysql
# 查某学生最好的那些科目中 courseId最小的
SELECT student_id, MIN(courseId) AS course_id, grade
From Enrollments
# 行 => 最好的那些科目
Where (student_id, grade) in (
	SELECT student_id, MAX(grade) AS grade
	From Enrollments
	Group By student_id
)       
Group By student_id
Order By student_id ASC
```





### 连接

**Left Join**

拓展左边，即使右表某些行不存在（会出现重复行）

=> **Right Join **同理

```mysql
SELECT *
FROM `customers` as c
LEFT JOIN orders as o
ON c.id = o.customerId
```

![image-20241119220005030](http://verification.longcoding.top/FpSCM4lwemQYDfDXFZaf_m9G1knV)



**Inner Join** => 只展示左右表均成功对接上的行



#### 没有卖出的商家

写一个解决方案, 报告所有在 `2020` 年度没有任何卖出的卖家的名字。

返回结果按照 `seller_name` **升序排列。**

```mysql
SELECT seller_name
FROM Seller
WHERE seller_id not in (
    SELECT DISTINCT Orders.seller_id
    FROM Orders  
    WHERE Orders.sale_date like '2020%'
)
ORDER By seller_name ASC
```



#### [排名靠前的旅行者](https://leetcode.cn/problems/top-travellers/)

返回的结果表单，以 `travelled_distance` **降序排列** ，如果有两个或者更多的用户旅行了相同的距离, 那么再以 `name` **升序排列** 。

```mysql
SELECT name, COALESCE(SUM(distance), 0) AS travelled_distance
From Users Left join Rides
On Rides.user_id = Users.id  
Group By Rides.user_id 
ORDER BY travelled_distance DESC, name ASC;  # 使用别名排序
```



#### [607. 销售员](https://leetcode.cn/problems/sales-person/)

编写解决方案，找出没有任何与名为 **“RED”** 的公司相关的订单的所有销售人员的姓名。

```mysql
SELECT name
FROM SalesPerson
WHERE sales_id not in (
    SELECT DISTINCT Orders.sales_id
    FROM Orders LEFT JOIN Company 
    ON Orders.com_id = Company.com_id
    WHERE name = 'RED'
)
```



#### [计算布尔表达式的值](https://leetcode.cn/problems/evaluate-boolean-expression/)

```
输入：
Variables 表:
+------+-------+
| name | value |
+------+-------+
| x    | 66    |
| y    | 77    |
+------+-------+

Expressions 表:
+--------------+----------+---------------+
| left_operand | operator | right_operand |
+--------------+----------+---------------+
| x            | >        | y             |
| x            | <        | y             |
| x            | =        | y             |
| y            | >        | x             |
| y            | <        | x             |
| x            | =        | x             |
+--------------+----------+---------------+

输出:
+--------------+----------+---------------+-------+
| left_operand | operator | right_operand | value |
+--------------+----------+---------------+-------+
| x            | >        | y             | false |
| x            | <        | y             | true  |
| x            | =        | y             | false |
| y            | >        | x             | true  |
| y            | <        | x             | false |
| x            | =        | x             | true  |
+--------------+----------+---------------+-------+
```

```mysql
SELECT e.left_operand, e.operator, e.right_operand,
CASE e.operator
    WHEN '>' THEN IF(v1.value>v2.value, 'true', 'false')
    WHEN '<' THEN IF(v1.value<v2.value, 'true', 'false') 
    ELSE IF(v1.value=v2.value, 'true', 'false') 
END value

FROM Expressions e
LEFT JOIN Variables v1 ON e.left_operand = v1.name 
LEFT JOIN Variables v2 ON e.right_operand = v2.name 
```

考点：多重连表 e.(...)v1.(...)v2.(...)

条件判断表达式

case [sign]

​	when exp then result[if(exp, true, false)]

​	when exp then result

​	else result

end



---

#### [查询球队积分](https://leetcode.cn/problems/team-scores-in-football-tournament/)

分别查询主队得分和客队得分。再进行汇总统计 

IFNULL(SUM(num_points), 0)  \# IFNULL(Origin, NULL-value)

```mysql
SELECT t.team_id, t.team_name, IFNULL(SUM(num_points), 0) as num_points
FROM Teams AS t
LEFT JOIN (
    SELECT host_team AS team_id,
    CASE
        WHEN host_goals > guest_goals THEN 3
        WHEN host_goals = guest_goals THEN 1 
    END AS num_points
    FROM Matches
    UNION ALL
    SELECT guest_team AS team_id,
    CASE
        WHEN guest_goals > host_goals THEN 3
        WHEN host_goals = guest_goals THEN 1 
    END AS num_points
    FROM Matches
) AS m ON t.team_id = m.team_id
Group By t.team_id
Order BY num_points DESC, t.team_id ASC 
```

\# 1. LEFT JOIN 将相关的右表补充至左表 ！若1vN则产生重复记录(聚合该记录)

\# 2. UNION(ALL) 两张表需字段相同，上下拼接为一张表，ALL则保留重复记录

\# 3. case 空条件

​	     when ? then x

​		...

​		ELSE y   

​		end



### 聚合函数

----

#### 2020最后一次登录

```mysql
SELECT MAX(Feild)  # <= 只要组内最大的记录
	   Min(Feild)  # <= 只要组内最小的记录
...

Group By
```



#### [仓库经理](https://leetcode.cn/problems/warehouse-manager/)

```mysql
SELECT Warehouse.name AS warehouse_name, 
SUM(units*Products.Width*Products.Length*Products.Height) AS volume   # 聚合组内数据
FROM Warehouse 
LEFT JOIN Products ON Warehouse.product_id = Products.product_id  # 补齐信息
Group By Warehouse.name                                           # 分组
```



#### 订单最多的客户

```mysql
SELECT customer_number 
FROM Orders 
GROUP BY customer_number
ORDER BY COUNT(*) DESC  # 对组进行排序
LIMIT 1                 # Top位置


SELECT o.customer_number
FROM (
	SELECT customer_number, COUNT(*) AS count_number
	FROM Orders 
	GROUP BY customer_number
) o
ORDER BY o.count_number DESC
LIMIT 1
```



#### [查找每个员工花费的总时间](https://leetcode.cn/problems/find-total-time-spent-by-each-employee/)

多次分组 & 聚合

```mysql
SELECT `day`, emp_id, SUM(out_time-in_time)
FROM Employees
GROUP BY emp_id, event_day 
```



#### 及时食物配送

考点：计算公式

\#1  ROUND(X*100, 2)   33.33

\#2  SUM(IF(X, 1, 0)) / COUNT(ID)

**按列操作**

```mysql
SELECT ROUND((SUM(IF(order_date = customer_pref_delivery_date, 1, 0)))/COUNT(delivery_id)*100, 2)  AS immediate_percentage 
FROM Delivery
```



#### 苹果和桔子

```mysql
SELECT sale_date,
(SUM(IF(fruit='apples', sold_num, 0)) - SUM(IF(fruit='oranges', sold_num, 0))) AS diff
FROM Sales
Group By sale_date
```



⭐ 分别聚合组内不同条件的记录



#### 两个人的通话记录

考点，多次分组进行聚合统计。

题干中，呼叫与接收不分。



```mysql
SELECT from_id AS person1, to_id AS person2, COUNT(*) AS call_count,SUM(duration) AS total_duration 

FROM (

  SELECT IF(from_id < to_id, from_id, to_id) AS from_id, IF(from_id < to_id, to_id, from_id) AS to_id, duration

  FROM Calls

) AS c

Group BY from_id, to_id
```





### 排序和分组

#### [银行账户概要 II](https://leetcode.cn/problems/bank-account-summary-ii/)

编写解决方案, 报告余额高于 10000 的所有用户的名字和余额. 账户的余额等于包含该账户的所有交易的总和

```mysql
SELECT name, balance
FROM Users 
LEFT JOIN (
    SELECT account, SUM(amount) AS balance
    FROM Transactions
    Group BY account) AS trans On Users.account = trans.account
Where balance > 10000
```



#### [查找重复的电子邮箱](https://leetcode.cn/problems/duplicate-emails/)

编写解决方案来报告所有重复的电子邮件

```mysql
SELECT email AS Email
FROM Person
Group By email
HAVING count(*) > 1
```



#### [合作过至少三次的演员和导演](https://leetcode.cn/problems/actors-and-directors-who-cooperated-at-least-three-times/)

编写解决方案找出合作过至少三次的演员和导演的 id 对 `(actor_id, director_id)`

```mysql
SELECT actor_id, director_id
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(*) >= 3
```



消费者下单频率

67月，每个月均消费至少100的用户

```mysql
SELECT o.customer_id, name
FROM () AS o
LEFT JOIN Customers ON o.customer_id = Customers.customer_id


# 其中
SELECT customer_id, order_date, (quantity*price) AS cost
FROM Orders 
LEFT JOIN Product ON Orders.product_id = Product.product_id
WHERE month(order_date) in (6,7)
Group BY customer_id
HAVING SUM(IF(month(order_date) = 6, cost, 0)) >= 100 and
       SUM(IF(month(order_date) = 7, cost, 0)) >= 100
```





#### 可以放心投资的国家

分步实现：

1️⃣ 将呼叫\接听联合

2️⃣ 将对应人员及其国家准备好

3️⃣ 再次连接，并按国家分组且聚合(HAVING进行条件筛选)

```mysql
SELECT name as country 
FROM 
(
    SELECT caller_id AS call_id, duration FROM Calls 
    UNION ALL
    SELECT callee_id AS call_id, duration FROM Calls
) a
LEFT JOIN 
(
    SELECT id, Country.name
    FROM Person 
    LEFT JOIN Country on SUBSTR(phone_number, 1, 3) = country_code
) b
ON a.call_id = b.id
Group BY name
HAVING AVG(duration) > (SELECT AVG(duration) FROM Calls)
```





### 高级查询和连接

`HAVING` 子句用来对 **聚合结果** 进行过滤，而 `HAVING` 子句无法直接使用非聚合列



#### 页面推荐

编写解决方案，向`user_id` = 1 的用户，推荐其朋友们喜欢的页面。不要推荐该用户已经喜欢的页面。



先find他的所有朋友。 他->朋友 and 朋友 ->

```mysql
SELECT DISTINCT page_id as recommended_page 
FROM (SELECT user2_id as user_id
    FROM Friendship 
    WHERE user1_id = 1
    UNION
    SELECT user1_id as user_id
    FROM Friendship 
    WHERE user2_id = 1) AS fs
LEFT JOIN Likes ls ON fs.user_id = ls.user_id 
WHERE page_id not in (
    SELECT page_id
    FROM Likes 
    WHERE user_id = 1
)
ORDER BY recommended_page ASC
```



#### [树节点](https://leetcode.cn/problems/tree-node/)

CASE ... END  按列来，每次处理列的一个元素

```mysql
SELECT id AS `id`, 
CASE
    WHEN id = 1 THEN "Root"
    WHEN id in (SELECT atree.p_id FROM tree atree) THEN "Inner"
    ELSE "Leaf"
END AS type 
FROM Tree 
```



#### [游戏玩法分析 III](https://leetcode.cn/problems/game-play-analysis-iii/)

编写一个解决方案，同时报告每组玩家和日期，以及玩家到 **目前为止** 玩了多少游戏。也就是说，玩家在该日期之前所玩的游戏总数。详细情况请查看示例。



利用笛卡尔积， 自己跟自己所有的记录进行连接。暴力遍历

```mysql
SELECT a1.player_id, a1.event_date, SUM(a2.games_played) AS games_played_so_far
FROM Activity a1, Activity a2  # 笛卡儿积，o(n) x o(n) = o(n^2) 条纪录
WHERE a1.player_id = a2.player_id and a1.event_date >= a2.event_date   # 只要自己和自己进行连接的记录
GROUP BY a1.player_id, a1.event_date
```



#### [大满贯数量](https://leetcode.cn/problems/grand-slam-titles/)

编写解决方案，找出每一个球员**赢**得大满贯**比赛的次数**。结果不包含没有赢得比赛的球员的ID 。 

思路： 行 -> 列 再聚合

```
Players 表：
+-----------+-------------+
| player_id | player_name |
+-----------+-------------+
| 1         | Nadal       |
| 2         | Federer     |
| 3         | Novak       |
+-----------+-------------+
Championships 表：
+------+-----------+---------+---------+---------+
| year | Wimbledon | Fr_open | US_open | Au_open |
+------+-----------+---------+---------+---------+
| 2018 | 1         | 1       | 1       | 1       |
| 2019 | 1         | 1       | 2       | 2       |
| 2020 | 2         | 1       | 2       | 2       |
+------+-----------+---------+---------+---------+
输出：
+-----------+-------------+-------------------+
| player_id | player_name | grand_slams_count |
+-----------+-------------+-------------------+
| 2         | Federer     | 5                 |
| 1         | Nadal       | 7                 |
+-----------+-------------+-------------------+
```



```mysql
SELECT res.player_id, ps.player_name, res.grand_slams_count 
FROM (
    SELECT player_id, COUNT(*) AS grand_slams_count
    FROM (
        SELECT Wimbledon player_id
        FROM Championships 
        UNION ALL
        SELECT Fr_open player_id
        FROM Championships 
        UNION ALL
        SELECT US_open player_id
        FROM Championships 
        UNION ALL
        SELECT Au_open player_id
        FROM Championships 
    ) AS rec
    GROUP BY rec.player_id 
) AS res
LEFT JOIN Players AS ps ON res.player_id = ps.player_id
```



#### [ 应该被禁止的 Leetflex 账户](https://leetcode.cn/problems/leetflex-banned-accounts/)

编写解决方案，查找那些应该被禁止的Leetflex帐户编号 `account_id` 。 如果某个帐户在某一时刻从两个不同的网络地址登录了，则这个帐户应该被禁止。



```
LogInfo table:
+------------+------------+---------------------+---------------------+
| account_id | ip_address | login               | logout              |
+------------+------------+---------------------+---------------------+
| 1          | 1          | 2021-02-01 09:00:00 | 2021-02-01 09:30:00 |
| 1          | 2          | 2021-02-01 08:00:00 | 2021-02-01 11:30:00 |
| 2          | 6          | 2021-02-01 20:30:00 | 2021-02-01 22:00:00 |
| 2          | 7          | 2021-02-02 20:30:00 | 2021-02-02 22:00:00 |
| 3          | 9          | 2021-02-01 16:00:00 | 2021-02-01 16:59:59 |
| 3          | 13         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |
| 4          | 10         | 2021-02-01 16:00:00 | 2021-02-01 17:00:00 |
| 4          | 11         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |
+------------+------------+---------------------+---------------------+
```



```mysql
AND ((a.login between b.login and b.logout) OR (a.logout between b.login and b.logout))
```



**GROUP BY 分组**

**HAVING 作用于组，挑选指定的组**



### 子查询

#### 各部门最高收入的员工 (可能有多个)

```mysql
SELECT d.name AS Department, e1.name AS Employee, e1.salary AS Salary 
FROM Employee e1 
LEFT JOIN Department d ON e1.departmentId = d.id
WHERE e1.salary = (
    SELECT MAX(salary)
    FROM Employee e2
    GROUP BY e2.departmentId
    HAVING e2.departmentId = e1.departmentId
)
```

效率不高！！！



优化：

```mysql
SELECT d.name AS Department, e1.name AS Employee, e1.salary AS Salary 
FROM Employee e1 
LEFT JOIN Department d ON e1.departmentId = d.id
WHERE (e1.departmentId,e1.salary) in (
    SELECT e2.departmentId, MAX(salary)
    FROM Employee e2
    GROUP BY e2.departmentId
)
```

![image-20241123202209491](http://verification.longcoding.top/Fic8qb3rJO4MKNDEcSJxAHSLTfx8)



生成临时表（避免子查询 重复的执行）

```mysql
WITH MaxSalaries AS (
    SELECT departmentId, MAX(salary) AS max_salary
    FROM Employee
    GROUP BY departmentId
)
```



#### [每件商品的最新订单](https://leetcode.cn/problems/the-most-recent-orders-for-each-product/)

可能有多个

```mysql
With LatestDate AS (
    SELECT product_id, MAX(order_date) AS order_date
    FROM Orders
    GROUP BY product_id
)


SELECT p.product_name, o.product_id, o.order_id, o.order_date
FROM Orders AS o
LEFT JOIN LatestDate AS ld ON o.product_id = ld.product_id
LEFT JOIN Products AS p ON o.product_id = p.product_id
WHERE o.order_date = ld.order_date
ORDER BY p.product_name ASC, o.product_id ASC, o.order_id ASC
```



#### 最近的三笔订单

学习， 可分组打index标记，标记按照ORDER顺序

```mysql
ROW_NUMBER() OVER (
	PARTITION BY ?
    ORDER BY ?
)
```

![image-20241123210524866](http://verification.longcoding.top/FuHsPK9Jy0sbdBP2KV1F8HIHvJwm)

```mysql
WITH ORDER_INFO AS (
    SELECT c.name, c.customer_id, o.order_id, o.order_date,
    ROW_NUMBER() OVER (
        PARTITION BY o.customer_id
        ORDER BY o.order_date DESC
    ) AS row_num
    FROM Orders AS o
    LEFT JOIN Customers AS c ON o.customer_id = c.customer_id
)

SELECT name AS customer_name, customer_id, order_id, order_date
FROM ORDER_INFO AS o
WHERE o.row_num <= 3
ORDER BY customer_name ASC, customer_id ASC, order_date DESC
```

先生成临时表，避免重复生成子查询



#### 每天的最大金额

先筛选出每天的最大金额，再执行后续的判断

```mysql
WITH MAX_AMOUNT AS (
    SELECT DATE(`day`) AS `day`, MAX(amount) max_amount 
    FROM Transactions AS t
    GROUP BY DATE(`day`)
)


SELECT transaction_id
FROM Transactions AS t
LEFT JOIN MAX_AMOUNT AS ma ON DATE(t.day) = ma.day
WHERE (DATE(t.day), t.amount) = (ma.day, ma.max_amount)
ORDER BY transaction_id ASC
```



直接给每天的 订单，按照金额， 打index标记

❗ RANK 相同值会获得相同的排名

```mysql
RANK() OVER (
) 
```

而 ROW_NUMBER，idx严格递增

```mysql
WITH Temp AS (
    SELECT transaction_id, `day`, amount,
    rank() OVER (
        PARTITION BY DATE(`day`)
        ORDER BY amount DESC
    ) AS idx
    FROM Transactions AS t
)

SELECT transaction_id
FROM Temp as t
WHERE idx = 1
ORDER BY transaction_id ASC
```





---

### 窗口函数和公共表表达式



#### [ 项目员工 III](https://leetcode.cn/problems/project-employees-iii/)

找出每个项目 **经验最丰富的员工(可能有多个)**

直接1. 分项目 2. 项目内按经验程度进行RANK 3. 再筛选

```mysql
WITH Temp AS (
    SELECT p.project_id, p.employee_id,
    RANK() OVER (
        PARTITION BY p.project_id
        ORDER BY e.experience_years DESC
    ) AS idx
    FROM Project  AS p
    LEFT JOIN Employee as e ON p.employee_id = e.employee_id
)


SELECT project_id, employee_id
FROM Temp
WHERE idx = 1
```



#### [每位顾客最经常订购的商品](https://leetcode.cn/problems/the-most-frequently-ordered-products-for-each-customer/)

![image-20241123220011980](http://verification.longcoding.top/FrHfIlb60X2tBlwp4ZKyTO3Gjq_b)

主SQL GROUP BY 之后，与 RANK() 的操作

```mysql
SELECT 
    customer_id,
    product_id,
    COUNT(*) AS order_count,
    RANK() OVER (ORDER BY COUNT(*) DESC) AS RK
FROM Orders
GROUP BY customer_id, product_id;
```

结果：

![image-20241123220106683](http://verification.longcoding.top/Fr4AA3oRXuQHg-x2t0VztB5gF9kd)

添加 PARTITION BY customer_id 进行隔离

```mysql
SELECT 
    customer_id,
    product_id,
    COUNT(*) AS order_count,
    RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(*) DESC) AS RK
FROM Orders
GROUP BY customer_id, product_id;
```

结果：

![image-20241123220202936](http://verification.longcoding.top/FiZarXoCwCQwP7-J8FqBhWKmZPxb)



=> 结论。 1. 先执行主SQL的GROUP BY，进行分区，再RANK(), RANK()按照 ORDER BY(必须聚合为单行)进行排序， 利用PARTITION进行排序隔离。



答案

```mysql
WITH Temp AS (
    SELECT 
        customer_id,
        product_id,
        COUNT(*) AS order_count,
        RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(*) DESC) AS RK  
    FROM Orders
    GROUP BY customer_id, product_id
)

SELECT t.customer_id, t.product_id, p.product_name 
FROM Temp AS t
LEFT JOIN Products AS p ON t.product_id = p.product_id 
WHERE RK = 1  # 最高的 NO.1
```





---

#### [访问日期之间最大的空档期](https://leetcode.cn/problems/biggest-window-between-visits/)

LEAD(column, offset, defualt) OVER (PARTITION BY ... ORDER BY ...) 找后面的行。其中LEAD表示引领。

LAG(...)   找前面的行， 滞后

```mysql
SELECT user_id, MAX(DATEDIFF(after_day, visit_date)) AS biggest_window
FROM (
    SELECT user_id, visit_date,
    LEAD(visit_date, 1, '2021-1-1') OVER (PARTITION BY user_id ORDER BY visit_date ASC) AS after_day
    FROM UserVisits AS uv
) AS t
GROUP BY user_id
```

考点：行之间的差值



---

**CTE Common Table Expression**

#### [向公司 CEO 汇报工作的所有人](https://leetcode.cn/problems/all-people-report-to-the-given-manager/)

查询树状结构节点  列转行

依次暴力枚举

```mysql
SELECT employee_id
FROM (
    SELECT employee_id
    FROM Employees 
    WHERE manager_id = 1 and employee_id != 1
    UNION
    SELECT employee_id
    FROM Employees
    WHERE manager_id in (
        SELECT employee_id
        FROM (SELECT employee_id
              FROM Employees 
              WHERE manager_id = 1 and employee_id != 1) AS e1
    )
    UNION 
    SELECT employee_id
    FROM Employees
    WHERE manager_id in (
        SELECT employee_id
        FROM (SELECT employee_id
            FROM Employees
            WHERE manager_id in (
                SELECT employee_id
                FROM (SELECT employee_id
                      FROM Employees 
                      WHERE manager_id = 1 and employee_id != 1) AS e2
            )) e3
    )
) AS e
```

太低效了！！！ SB操作，疯狂连续查询



优化

```mysql
SELECT employee_id
FROM (
    SELECT employee_id
    FROM Employees 
    WHERE manager_id = 1 and employee_id != 1  # ✔️
    UNION
    
    SELECT employee_id
    FROM Employees
    WHERE manager_id in (
        SELECT employee_id
        FROM Employees 
        WHERE manager_id = 1 and employee_id != 1
    )
    UNION 
    
    SELECT employee_id
    FROM Employees
    WHERE manager_id in (
        SELECT employee_id
        FROM Employees
        WHERE manager_id in (
            SELECT employee_id
            FROM Employees 
            WHERE manager_id = 1 and employee_id != 1
        )
    )
) AS e
```

---



利用连接来计算

```mysql
SELECT DISTINCT e4.employee_id
FROM Employees e1
LEFT JOIN Employees e2 ON e1.employee_id = e2.manager_id  
LEFT JOIN Employees e3 ON e2.employee_id = e3.manager_id  
LEFT JOIN Employees e4 ON e3.employee_id = e4.manager_id  
WHERE e1.employee_id = 1 and e4.employee_id != 1
```

```java
1 Boss => {
    	1 Boss     => {
                       1 Boss                 => {
                           					     1 Boss 
                                                 2 Bob
       											 77 Robert
                       }
                       2 Bob                  => 4 Daniel
        			   77 Robert              => null
        }
        2 Bob       =>  4 Daniel              => null   
        77 Robert   => null                   => null   
}
```



#### 寻找连续区间的开始和结束位置



思路：

```mysql
data:  1,2,3,7,8,10
idx :  1,2,3,4,5, 6
diff:  0,0,0,3,3, 4   <= GROUP BY diff => MIN(idx) AS Start_ID, MAX(idx) AS End_ID
```

```mysql
WITH Temp AS (
    SELECT log_id,
    ROW_NUMBER() OVER (ORDER BY log_id ASC) AS idx
    FROM Logs 
)

SELECT MIN(log_id) AS start_id, MAX(log_id) AS end_id
FROM (
    SELECT log_id, (log_id-idx ) AS diff
    FROM Temp 
) AS t
GROUP BY diff
```



#### 查找处于成绩中游的学生

参加的考试，没有一科是最高分或者最低分。科科中等

逻辑： 1. 判断每一科，再统计所有科目

```mysql
WITH MAXMIN AS (
    SELECT exam_id, MIN(score) min_score, MAX(score) max_score
    FROM Exam 
    GROUP BY exam_id  
)  


SELECT s.student_id, s.student_name  
FROM (
    SELECT e.exam_id, e.student_id, 
    CASE
        WHEN score != min_score and score != max_score THEN 1
        ELSE 0
    END AS is_medium
    FROM Exam AS e
    LEFT JOIN MAXMIN m ON e.exam_id = m.exam_id
) AS t
LEFT JOIN Student AS s ON t.student_id = s.student_id
GROUP BY t.student_id
HAVING SUM(t.is_medium) = COUNT(t.is_medium)
ORDER BY student_id ASC
```



反过来： 将尖子生和垫底生排除在外即可





---

#### [寻找没有被执行的任务对](https://leetcode.cn/problems/find-the-subtasks-that-did-not-execute/)



```mysql
with recursive all_subtask(task_id, subtask_id) as (
    SELECT task_id, subtasks_count 
		FROM Tasks
    UNION ALL
		
    SELECT task_id, subtask_id-1 
		FROM all_subtask 
		where subtask_id-1 > 0
)


SELECT task_id, subtask_id
From all_subtask 
WHERE (task_id, subtask_id) not in (SELECT task_id, subtask_id FROM Executed)
```



```mysql
with recursive TableName(param1, param2) AS (
	SELECT param1, param2  // 基础结果
	FROM BaseTable
    UNION ALL
    // 下面式子会被重复递归
    SELECT param1, param2-1  // 新的结果
    FROM TableName         // 上一轮的结果
    where param2-1 > 0  // 递归终止条件
)
```





