---
title: "Sql With Recursive Cte 递归"
date: 2023-05-09T11:37:56+08:00
draft: false
description: "Sql With Recursive Cte 递归"
categories:

tags:

---
# 准备内容
```sql
-- ddl
-- users definition
CREATE TABLE `users` (`id` INTEGER PRIMARY KEY AUTOINCREMENT, "name" VARCHAR(255) NOT NULL, pid int default 0);

INSERT INTO USERS(ID,NAME,PID)VALUES
(1,'A',0),
(2,'B',1),
(3,'C',2),
(4,'CC',2),
(5,'BB',1);

SELECT * FROM USERS;

WITH RECURSIVE U AS(
	SELECT *,1 AS LEVEL FROM USERS WHERE PID = 0
	UNION 
	SELECT USERS.*,LEVEL + 1 FROM USERS ,U 
	WHERE USERS.PID = U.ID
)
SELECT * FROM U;


--alter table users add column pid int default 0;
--insert into users(phone)
--alter table users rename column phone to name;

--alter table users drop updatedAt;
```
# 表数据
id|name|pid|
--|----|---
 1|a   |  0|
 2|b   |  1|
 3|c   |  2|
 4|cc  |  2|
 5|bb  |  1|

# 递归并增加level字段
## SQL
```sql
WITH RECURSIVE U AS(
	SELECT *,1 AS LEVEL FROM USERS WHERE PID = 0
	UNION 
	SELECT USERS.*,LEVEL + 1 FROM USERS ,U 
	WHERE USERS.PID = U.ID
)
SELECT * FROM U;
```
## 结果
id|name|pid|level|
--|----|---|-----|
 1|a   |  0|    1|
 2|b   |  1|    2|
 5|bb  |  1|    2|
 3|c   |  2|    3|
 4|cc  |  2|    3|

递归的实质是用`AS`后面第一句`sql`(`SELECT *,1 AS LEVEL FROM USERS WHERE PID = 0`)作为基础条件，`UNION`后面的语句(`SELECT USERS.*,LEVEL + 1 FROM USERS ,U 
	WHERE USERS.PID = U.ID`)循环执行来拿到结果返回到`U临时表`。
refer: https://learnsql.com/blog/sql-recursive-cte/