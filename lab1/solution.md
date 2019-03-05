# 实验一 交互式SQL的使用

## 1.1 创建 Student 数据库

Student 数据库包含三个表：学生表(Students)、课程表(Courses)和成绩表(SC)。

表结构如下：

- Students(SNO, SNAME, SEX, BDATE, HEIGHT, DEPARTMEENT)
- Courses(CNO, CNAME, LHOUR, CREDIT, SEMESTER)
- SC(SNO, CNO, GRADE)

```sql
create table Students(
    SNO CHAR(12) PRIMARY KEY,
    SNAME VARCHAR(10),
    SEX VARCHAR(10),
    BDATE Date,
    HEIGHT NUMERIC(3, 2),
    DEPARTMENT VARCHAR(20)
);

create table Courses(
    CNO VARCHAR(10) PRIMARY KEY,
    CNAME VARCHAR(20),
    LHOUR INT,
    CREDIT INT,
    SEMESTER VARCHAR(10)
);

create table SC(
    SNO CHAR(12),
    CNO VARCHAR(10),
    GRADE INT,
    PRIMARY KEY(SNO, CNO),
    FOREIGN KEY(SNO) REFERENCES Students(SNO),
    FOREIGN KEY(CNO) REFERENCES Courses(CNO)
);
```

## 1.2 完成以下查询和更新操作

<1> 查询身高大于 1.80m 的男生的学号和姓名；

```sql
SELECT SNO, SNAME
FROM Students
WHERE HEIGHT > 1.80;
```

<2> 查询计算机系秋季所开课程的课程号和学分数；

```sql
SELECT distinct CNO, CREDIT
FROM Courses
WHERE SEMESTER = '秋' AND CNO LIKE 'CS%';
```

<3> 查询选修计算机系秋季所开课程的男生的姓名、课程号、学分数、成绩；

```sql
SELECT SNAME, SC.CNO, CREDIT, GRADE
FROM Students JOIN SC ON Students.SNO = SC.SNO JOIN Courses ON SC.CNO = Courses.CNO
WHERE SEMESTER = '秋' AND CNO LIKE 'CS%' AND SEX = '男';
```

<4> 查询至少选修一门电机系课程的女生的姓名（假设电机系课程的课程号以EE开头）；

```sql
SELECT distinct SNAME
FROM Students JOIN SC ON Students.SNO = SC.SNO JOIN Courses ON SC.CNO = Courses.CNO
WHERE CNO LIKE 'EE%' and SEX = '女';
```

<5> 查询每位学生已选修课程的门数和总平均成绩；

```sql
SELECT SNO, SNAME, COUNT(CNO) AS COURSE_NUM, AVG(GRADE) AS AVG_GRADE
FROM Students LEFT JOIN SC ON Students.SNO = SC.SNO
GROUP BY SNO, SNAME;
```

<6> 查询每门课程选课的学生人数，最高成绩，最低成绩和平均成绩；

```sql
SELECT CNO, COUNT(SNO) AS STUDENT_NUM, MAX(GRADE) AS MAX_GRADE, MIN(GRADE) AS MIN_GRADE, AVG(GRADE) AS AVG_GRADE
FROM Courses LEFT JOIN SC ON Courses.CNO = SC.CNO
GROUP BY CNO;
```

<7> 查询所有课程的成绩都在 80 分以上的学生的姓名、学号、且按学号升序排列；

```sql
SELECT SNO, SNAME
FROM Students JOIN SC ON Students.SNO = SC.SNO
GROUP BY SNO, SNAME
HAVING MIN(GRADE) >= 80
ORDER BY SNO ASC;
```

<8> 查询缺成绩的学生的姓名，缺成绩的课程号及其学分数；

```sql
SELECT SNAME, SC.CNO, CREDIT
FROM Students JOIN SC ON Students.SNO = SC.SNO JOIN Courses ON SC.CNO = Courses.CNO
WHERE GRADE IS NULL;
```

<9> 查询有一门以上(含一门)三个学分以上课程的成绩低于 70 分的学生的姓名；

```sql
SELECT distinct SNAME
FROM Students JOIN SC ON Students.SNO = SC.SNO JOIN Courses ON SC.CNO = Courses.CNO
WHERE CREDIT >= 3 AND GRADE < 70;
```

<10> 查询 1984~1986 年出生的学生的姓名，总平均成绩及已修学分数。

```sql
SELECT SNAME, AVG(GRADE) AS AVG_GRADE, SUM(CREDIT) AS TOTAL_CREDIT
FROM Students JOIN SC ON Students.SNO = SC.SNO JOIN Courses ON SC.CNO = Courses.CNO
WHERE BDATE BETWEEN TO_DATE('1984-01-01', 'YYYY-MM-DD') AND TO_DATE('1986-12-31', 'YYYY-MM-DD')
GROUP BY SNAME;
```

<11> 在 STUDENT 和 SC 关系中，删去 SNO 以 "01" 开头的所有记录。

```sql
DELETE FROM SC
WHERE SNO LIKE '01%';
```

<12> 在 Students 关系中增加以下记录：

<201504090101  何平 女  1987-03-02  1.62>
<201504080130  向阳 男  1986-12-1   1.75>

```sql
INSERT INTO
Students
VALUES
('201504090101', '何平', '女', TO_DATE('1987-03-02', 'YYYY-MM-DD'), 1.62, NULL);

INSERT INTO
Students
VALUES
('201504080130', '向阳', '男', TO_DATE('1986-12-11', 'YYYY-MM-DD'), 1.75, NULL);
```

<13> 将课程 CS-221 的学分数增为 3，讲课时数增为 60

```sql
UPDATE Courses
SET CREDIT = 3, LHOUR = 60
WHERE CNO = 'CS-221';
```

## 1.3 补充题

<1> 统计各系的男生和女生的人数。

```sql
SELECT DEPARTMENT, SEX, COUNT(SNO) AS STUDENT_NUM
FROM Students
GROUP BY DEPARTMENT, SEX
ORDER BY DEDEPARTMENT ASC;
```

<2> 列出学习过 "编译原理"，"数据库" 或 "体系结构" 课程，且这些课程的成绩之一在 90 分以上的学生的名字。

```sql
select distinct SNAME
from Students JOIN SC ON Students.SNO = SC.SNO JOIN Courses ON SC.CNO = Courses.CNO
where CNAME in ('编译原理', '数据库', '体系结构') and GRADE >= 90;
```

<3> 列出未修选‘电子技术’课程，但选修了‘数字电路’或‘数字逻辑’课程的学生数。

```sql
SELECT
    COUNT(DISTINCT SNO) AS STUDENT_NUM
FROM
    SC
    JOIN Courses ON SC.CNO = Courses.CNO
WHERE
    SNO NOT IN (
        SELECT
            DISTINCT SNO
        FROM
            SC
            JOIN Courses ON SC.CNO = Courses.CNO
        WHERE
            CNAME = '电子技术'
    )
    and SNO IN (
        SELECT
            DISTINCT SNO
        FROM
            SC
            JOIN Courses ON SC.CNO = Courses.CNO
        WHERE
            CNAME = '数字电路'
            OR CNAME = '数字逻辑'
    );
```

<4> 按课程排序列出所有学生的成绩，尚无学生选修的课程，也需要列出，相关的学生成绩用 NULL 表示。

```sql
SELECT SNO, CNO, CBAME, GRADE
FROM SC RIGHT JOIN Courses ON SC.CNO = Courses.CNO
GROUP BY CNO, CNAME, SNO, GRADE
ORDER BY CNO ASC;
```

<5> 列出平均成绩最高的学生名字和成绩。(SELECT句中不得使用TOP n子句)

```sql
create view test1(SNO, SNAME, AVG_GRADE) as
SELECT SNO, SNAME, AVG(GRADE) AS AVG_GRADE
FROM Students JOIN SC ON Students.SNO = SC.SNO
GROUP BY SNO, SNAME;
```

```sql
SELECT SNAME, AVG_GRADE
FROM test1
WHERE AVG_GRADE = (SELECT MAX(AVG_GRADE) FROM test1);
```

## 1.4 选做题

对每门课增加“先修课程”的属性，用来表示某一门课程的先修课程，每门课程应可记录多于一门的先修课程。

要求

1) 修改表结构的定义，应尽量避免数据冗余，建立必要的主键，外键。
2) 设计并插入必要的测试数据，完成以下查询：列出有资格选修数据库课程的所有学生。(该学生已经选修过数据库课程的所有先修课，并达到合格成绩。)

```sql
create table Prerequisites(
    CNO VARCHAR(10),
    PREREQUISITE VARCHAR(10),
    PRIMARY KEY(CNO, PREREQUISITE),
    FOREIGN KEY(CNO) REFERENCES Courses(CNO),
    FOREIGN KEY(PREREQUISITE) REFERENCES Courses(CNO)
);

INSERT INTO
Prerequisites
VALUES
('CS-101', 'CS-100');

INSERT INTO
Prerequisites
VALUES
('CS-102', 'CS-101');

INSERT INTO
Prerequisites
VALUES
('CS-201', 'CS-101');

INSERT INTO
Prerequisites
VALUES
('CS-201', 'CS-102');

INSERT INTO
Prerequisites
VALUES
('CS-202', 'CS-201');

INSERT INTO
Prerequisites
VALUES
('CS-203', 'CS-201');
```

```sql
SELECT SNAME
FROM Students JOIN SC ON Students.SNO = SC.SNO JOIN Courses ON SC.CNO = Courses.CNO
WHERE Courses.CNO = 'CS-203' AND SC.GRADE >= 60;
```
