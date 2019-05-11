# 实验四 事务的管理

## 4.1 创建 Student 数据库

采用实验一的建库脚本和数据插入脚本创建 Student 数据库。

## 4.2 测试事务隔离级别

分别设置不同的隔离级别，让两个并发事务交错执行的程序或事务，能分别显示每种隔离级别下，是否出现丢失更新，脏读，读值不可复现以及幻象记录四种情况。

事务隔离级别：一个事务对数据库的修改与并行的另一个事务的隔离程度。

两个并发事务同时访问数据库表相同的行时，可能存在以下三个问题：

1. 幻想读：事务 T1 读取一条指定 where 条件的语句，返回结果集。此时事务 T2 插入一行新记录，恰好满足 T1 的 where 条件。然后 T1 使用相同的条件再次查询，结果集中可以看到 T2 插入的记录，这条新纪录就是幻想。
2. 不可重复读取：事务 T1 读取一行记录，紧接着事务 T2 修改了 T1 刚刚读取的记录，然后 T1 再次查询，发现与第一次读取的记录不同，这称为不可重复读。
3. 脏读：事务 T1 更新了一行记录，还未提交所做的修改，这个 T2 读取了更新后的数据，然后 T1 执行回滚操作，取消刚才的修改，所以 T2 所读取的行就无效，也就是脏数据。

为了处理这些问题，SQL标准定义了以下几种事务隔离级别

- `READ UNCOMMITTED` 幻想读、不可重复读和脏读都允许。
- `READ COMMITTED` 允许幻想读、不可重复读，不允许脏读
- `REPEATABLE READ` 允许幻想读，不允许不可重复读和脏读
- `SERIALIZABLE` 幻想读、不可重复读和脏读都不允许

Oracle 数据库支持 `READ COMMITTED` 和 `SERIALIZABLE` 这两种事务隔离级别。所以 Oracle 不支持脏读。

SQL 标准所定义的默认事务隔离级别是 `SERIALIZABLE`，但是 Oracle 默认使用的是 `READ COMMITTED`。

```sql
-- 幻想读测试
-- 事务 1
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
SELECT * FROM SC WHERE SNO = '2018001';
-- 事务 2
BEGIN TRANSACTION;
INSERT INTO SC VALUES('2018001', 'CS-110', 90);
COMMIT;
-- 事务 1
SELECT * FROM SC WHERE SNO = '2018001';
COMMIT;
```

```sql
-- 不可重复读测试
-- 事务 1
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
SELECT * FROM SC WHERE SNO = '2018001';
-- 事务 2
BEGIN TRANSACTION;
UPDATE SC SET GRADE = 90 WHERE SNO = '2018001' AND CNO = 'CS-110';
COMMIT;
-- 事务 1
SELECT * FROM SC WHERE SNO = '2018001';
COMMIT;
```

```sql
-- 脏读测试
-- 事务 1
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
SELECT * FROM SC WHERE SNO = '2018001';
-- 事务 2
BEGIN TRANSACTION;
UPDATE SC SET GRADE = 90 WHERE SNO = '2018001' AND CNO = 'CS-110';
COMMIT;
-- 事务 1
SELECT * FROM SC WHERE SNO = '2018001';
ROLLBACK;
```

## 4.3 备份与恢复

<1> 备份数据库。使用expdp命令备份数据库。

```sql
-- 备份数据库
expdp system/123456@orcl dumpfile=student.dmp logfile=student.log
```

<2> 删除SC表。删除SC表中的所有数据。

```sql
-- 删除SC表
TRUNCATE TABLE SC;
```

<3> 恢复到删除之前。使用impdp命令恢复数据库。

```sql
-- 恢复数据库
impdp system/123456@orcl dumpfile=student.dmp logfile=student.log
```
