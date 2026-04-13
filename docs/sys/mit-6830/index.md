# MIT 6.830 数据库课程概览

2021 版 

基于 Java 实现的单机关系型数据库。

各 Lab 的关键：

- Lab1 基本组件、位图
- Lab2 火山模型、LRU-K
- Lab3 Join 的成本估算框架、System R 优化器
- Lab4 两阶段锁、等待图检测死锁
- Lab5 B+ 树
- Lab6 ARIES 恢复算法

## 使用

### 准备工作

ant 编译，将程序打包为 jar 包

```bash
ant
```

创建 `emp.txt` `dept.txt` （最后留一个空行）

```txt
1,"Tom",  12,1
2,"Jerry",22,1
3,"Tuffy",32,1
4,"John", 42,1
5,"Alice",23,2
6,"Bob",  33,2
7,"Brown",43,2
8,"Carol",44,3
9,"David",54,3

```

```txt
1,"Engineer"
2,"Sale"
3,"HR"

```

终端中运行命令，将 txt 文件转换为 SimpleDB 可识别的 `.dat` 格式。

```bash
java -jar dist/simpledb.jar convert emp.txt 4 "int,string,int,int"
java -jar dist/simpledb.jar convert dept.txt 2 "int,string"
```

创建文件 `catalog.txt`，里面指定表名，字段名，字段类型。

```txt
emp(id int, name string, age int, dept_id int)
dept(id int, name string)
```

### 启动

```bash
java -jar dist/simpledb.jar parser catalog.txt
```

### 查

查询 `age > 35` 的员工信息，部门名称的查询需要以 `emp.dept_id = dept.id` 的条件关联 `emp` `dept` 两张表。

```sql
select emp.id, emp.name, emp.age, dept.name
from emp, dept
where emp.dept_id = dept.id # Join 条件，相当于 emp join dept on emp.dept_id = dept.id，只是 SimpleDB 解析器不支持这种写法
and emp.age > 35; # Filter 条件
```

```
Started a new transaction tid = 2
Added scan of table emp
Added scan of table dept
Added join between emp.dept_id and dept.id
Added select list field emp.id
Added select list field emp.name
Added select list field emp.age
Added select list field dept.name
The query plan is:
         π(emp.id,emp.name,emp.age,dept.name),card:5
         |
      ⨝(hash)(dept.id=emp.dept_id),card:5
   ______|______
   |           |
   |           σ(emp.age>35),card:5
   |           |
 scan(dept)  scan(emp)

emp.id	emp.name	emp.age	dept.name	
--------------------------------------------------
4	"John"	42	"Engineer"	
7	"Brown"	43	"Sale"	
8	"Carol"	44	"HR"	
9	"David"	54	"HR"	

 4 rows.
Transaction 2 committed.
----------------
0.01 seconds
```

### 增

```sql
insert into emp values (10, 'Musk', 53, 1);
```

```
Started a new transaction tid = 3
	
-----
1	

 1 rows.
Transaction 3 committed.
----------------
0.01 seconds
```

### 删

```sql
delete from emp where emp.id = 10;
```

```
Started a new transaction tid = 4
Added scan of table emp
Added select list field null.*
	
-----
1	

 1 rows.
Transaction 4 committed.
----------------
0.01 seconds
```

### 事务

SimpleDB 自动开启事务，每条 SQL 的执行伴随着一个事务的开始与提交/回滚。

也支持手动开启事务，命令如下：

```sql
# 开启事务，只读权限
set transaction read only;
# 开启事务，读写权限
set transaction read write;
# 提交事务
commit;
# 回滚事务
rollback;
```

### 退出

```sql
exit;
```

## 系统崩溃测试

```sql
insert into emp values (10, 'Musk', 53, 1);
```

执行完这条 SQL 后，数据会被写入日志。

由于是 NO-FORCE 策略，脏页依旧存在于缓冲池，未被刷新到磁盘中。

随后并不执行 `exit;`，直接强制中断数据库。

再次启动数据库，`LogFile.recover()` 方法会根据日志恢复数据。

```sql
select * from emp;
```

发现数据已经被刷新到了磁盘中。