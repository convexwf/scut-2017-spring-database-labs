# 实验二 数据库的安全和完整性约束

## 2.1 视图创建

<1> 新增表 `Credits(SNO, SumCredit, NoPass)`，表示学生已通过选修课程的合计学分数，以及不及格的课程数。

```sql
CREATE TABLE Credits(
    SNO VARCHAR(10),
    SumCredit INT,
    NoPass INT,
    PRIMARY KEY(SNO),
    FOREIGN KEY(SNO) REFERENCES Students(SNO)
);
```

<2> 创建视图 `Student_Grade(Sname,Cname,Grade)`，表示学生选修课程及成绩的详细信息。

```sql
CREATE VIEW Student_Grade(Sname, Cname, Grade) AS
SELECT SNAME, CNAME, GRADE
FROM Students, Courses, SC
WHERE Students.SNO = SC.SNO AND Courses.CNO = SC.CNO;
```

## 2.2 触发器创建

<1> 创建触发器 `Upd_Credit`：当在 SC 表中插入一条选课成绩，自动触发 `Upd_Credit`，完成在 `Credits` 表中修改该学生的合计学分数和不及格的课程数。

```sql
CREATE OR REPLACE TRIGGER Upd_Credit
AFTER INSERT ON SC
FOR EACH ROW
DECLARE
    CREDIT INT;
BEGIN
    SELECT CREDIT INTO CREDIT
    FROM Courses
    WHERE Courses.CNO = :NEW.CNO;
    IF :NEW.GRADE < 60 THEN
        UPDATE Credits
        SET SumCredit = SumCredit + CREDIT, NoPass = NoPass + 1
        WHERE SNO = :NEW.SNO;
    ELSE
        UPDATE Credits
        SET SumCredit = SumCredit + CREDIT
        WHERE SNO = :NEW.SNO;
    END IF;
END;
/
```

```sql
-- 测试触发器
INSERT INTO SC VALUES('2018001', '3-105', 85);
SELECT * FROM Credits WHERE SNO = '2018001';

INSERT INTO SC VALUES('2018001', '3-106', 55);
SELECT * FROM Credits WHERE SNO = '2018001';
```

<2> 创建 `Upd_StuView` 触发器(Instead of触发器)：当对视图 `Student_Grade` 插入数据项操作时，自动触发 `Upd_StuView`，完成对 SC 表的插入操作。

比如，当执行 `Insert into Student_Grade values('王刚', '数据库', 54)`，则触发器完成另一插入操作：`Insert into SC values('980201', 'CS-110', 54)`。

另外，需要检查当前插入的学生和课程是否已在 Students 和 Courses 表中存在，如不存在，不执行任何操作，并提示用户错误信息。

```sql
CREATE OR REPLACE TRIGGER Upd_StuView
INSTEAD OF INSERT ON Student_Grade
FOR EACH ROW
DECLARE
    SNO VARCHAR(10);
    CNO VARCHAR(10);
BEGIN
    SELECT SNO INTO SNO
    FROM Students
    WHERE SNAME = :NEW.SNAME;
    SELECT CNO INTO CNO
    FROM Courses
    WHERE CNAME = :NEW.CNAME;
    IF SNO IS NULL OR CNO IS NULL THEN
        DBMS_OUTPUT.PUT_LINE('Error: Student or Course not exist!');
    ELSE
        INSERT INTO SC VALUES(SNO, CNO, :NEW.GRADE);
    END IF;
END;
/
```

```sql
-- 测试触发器
INSERT INTO Student_Grade VALUES('王刚', '数据库', 54);
SELECT * FROM SC WHERE SNO = '980201
```

<3> 删除 SC 中所有的主键和外键，使用触发器实现 SNO 和 CNO 的约束定义。

```sql
-- 删除 SC 表中的主键和外键
ALTER TABLE SC DROP CONSTRAINT PK_SC;
ALTER TABLE SC DROP CONSTRAINT FK_SC_SNO;
ALTER TABLE SC DROP CONSTRAINT FK_SC_CNO;
```

```sql
-- 创建触发器
CREATE OR REPLACE TRIGGER Upd_SC
BEFORE INSERT ON SC
FOR EACH ROW
DECLARE
    SNO VARCHAR(10);
    CNO VARCHAR(10);
BEGIN
    SELECT SNO INTO SNO
    FROM Students
    WHERE SNAME = :NEW.SNAME;
    SELECT CNO INTO CNO
    FROM Courses
    WHERE CNAME = :NEW.CNAME;
    IF SNO IS NULL OR CNO IS NULL THEN
        DBMS_OUTPUT.PUT_LINE('Error: Student or Course not exist!');
        RAISE_APPLICATION_ERROR(-20001, 'Student or Course not exist!');
    ELSE
        :NEW.SNO := SNO;
        :NEW.CNO := CNO;
    END IF;
END;
/
```

```sql
-- 测试触发器
INSERT INTO SC VALUES('980201', 'CS-110', 54);
SELECT * FROM SC WHERE SNO = '980201';
```

## 2.3 设计安全机制

在该数据库系统中，有三类用户：

1. 学生，权限包括：查询所有的课程信息，根据学号和课程号来查询成绩。但不允许修改任何数据。(必做)只能查询自己的成绩，不能查询别人的成绩。（选做）
2. 老师：权限包括：查询有关学生及成绩的所有信息，有关课程的所有信息，但不允许修改任何数据。
3. 教务员：权限包括：查询和修改任何有关学生和课程的信息，但不允许查询和修改数据库中其它任何表，视图等数据库对象。

要求：安全控制必须仅由数据库一端来实现，不考虑由应用程序来控制。

为此，需要创建三个用户，登录时密码验证；分别授予各类权限，并测试权限的控制是否有效。

```sql
-- 创建学生用户
CREATE USER student IDENTIFIED BY pass_student;
-- 授予学生用户查询权限
GRANT CONNECT, RESOURCE TO student;
GRANT SELECT ON Courses TO student;
GRANT SELECT ON SC TO student;
-- 授予学生用户只能查询自己成绩权限
GRANT SELECT ON SC TO student WHERE SNO = '2018001';
```

```sql
-- 创建老师用户
CREATE USER teacher IDENTIFIED BY pass_teacher;
-- 授予老师用户查询权限
GRANT CONNECT, RESOURCE TO teacher;
GRANT SELECT ON Students, Courses, SC TO teacher;
```

```sql
-- 创建教务员用户
CREATE USER manager IDENTIFIED BY pass_manager;
-- 授予教务员用户查询和修改权限
GRANT CONNECT, RESOURCE TO manager;
GRANT all ON Students, Courses TO manager;
```
