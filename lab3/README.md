# 实验三 SQL 编程

## 3.1 创建 Student 数据库

采用实验一的建库脚本和数据插入脚本创建 Student 数据库。

## 3.2 创建存储过程

<1> 创建存储过程 `Add_Student(SNO, SNAME, SEX, BIRTHDAY, HEIGHT, DEPT)`，根据输入参数，插入一条学生记录。

```sql
CREATE OR REPLACE PROCEDURE Add_Student(
    P_SNO CHAR(12),
    P_SNAME VARCHAR(10),
    P_SEX VARCHAR(10),
    P_BIRTHDAY DATE,
    P_HEIGHT NUMERIC(3, 2),
    P_DEPT VARCHAR(20)
)
AS
BEGIN
    INSERT INTO Students VALUES
    (P_SNO, P_SNAME, P_SEX, P_BIRTHDAY, P_HEIGHT, P_DEPT);
END;
/
```

<2> 创建存储过程 `Upd_Grade(SNO, CNO, GRADE)`，根据输入参数，修改某学生选课的成绩。

```sql
CREATE OR REPLACE PROCEDURE Upd_Grade(
    P_SNO CHAR(12),
    P_CNO VARCHAR(10),
    P_GRADE INT
)
AS
BEGIN
    UPDATE SC
    SET GRADE = P_GRADE
    WHERE SNO = P_SNO AND CNO = P_CNO;
END;
/
```

<3> 创建存储过程 `Disp_Student(SNO, SUM_CREDIT OUT, AVG_GRADE OUT)`，根据 SNO 参数显示该学生的有关信息。

要求：根据 SNO 参数显示该学生的有关信息，包括：

- 学号，姓名，性别，年龄，身高，系别，所有选修的课程及成绩；
- 显示输出参数 `SUM_CREDIT`（表示选修课程的总学分）及 `AVG_GRADE`（表示3学分以上的课程的平均成绩）。

```sql
CREATE OR REPLACE PROCEDURE Disp_Student(
    P_SNO CHAR(12),
    SUM_CREDIT OUT INT,
    AVG_GRADE OUT NUMERIC(3, 2)
)
AS
BEGIN
    -- put_line 显示学生信息
    FOR R IN (
        SELECT * FROM Students WHERE SNO = P_SNO
    ) LOOP
        DBMS_OUTPUT.PUT_LINE('学号：' || R.SNO);
        DBMS_OUTPUT.PUT_LINE('姓名：' || R.SNAME);
        DBMS_OUTPUT.PUT_LINE('性别：' || R.SEX);
        DBMS_OUTPUT.PUT_LINE('年龄：' || (SYSDATE - R.BDATE) / 365);
        DBMS_OUTPUT.PUT_LINE('身高：' || R.HEIGHT);
        DBMS_OUTPUT.PUT_LINE('系别：' || R.DEPARTMENT);
    END LOOP;

    -- put_line 显示选修课程及成绩
    FOR R IN (
        SELECT CNAME, GRADE
        FROM SC JOIN Courses ON SC.CNO = Courses.CNO
        WHERE SNO = P_SNO
    ) LOOP
        DBMS_OUTPUT.PUT_LINE('课程：' || R.CNAME || '，成绩：' || R.GRADE);
    END LOOP;

    SELECT SUM(CREDIT) INTO SUM_CREDIT
    FROM SC JOIN Courses ON SC.CNO = Courses.CNO
    WHERE SNO = P_SNO;

    SELECT AVG(GRADE) INTO AVG_GRADE
    FROM SC JOIN Courses ON SC.CNO = Courses.CNO
    WHERE SNO = P_SNO AND CREDIT >= 3;
END;
/
```

<4> 创建存储过程 `CAL_GPA(SNO, GPA OUT)`，根据 SNO 参数输出并显示该学生的 GPA 值。

GPA 计算方法如下：

|GRADE(G) | GRADEPOINT(GP) |
|---------|-----------------|
|G >= 85  | 4               |
|85 > G >= 75 | 3           |
|75 > G >= 60 | 2           |
|60 > G    | 1               |

$$
GPA = \frac{\sum{GP \times CREDIT}}{\sum{CREDIT}}
$$

```sql
CREATE OR REPLACE PROCEDURE CAL_GPA(
    P_SNO CHAR(12),
    GPA OUT NUMERIC(3, 2)
)
AS
BEGIN
    SELECT SUM(GP * CREDIT) / SUM(CREDIT) INTO GPA
    FROM (
        SELECT GRADE, CREDIT,
            CASE
                WHEN GRADE >= 85 THEN 4
                WHEN GRADE >= 75 THEN 3
                WHEN GRADE >= 60 THEN 2
                ELSE 1
            END AS GP
        FROM SC JOIN Courses ON SC.CNO = Courses.CNO
        WHERE SNO = P_SNO
    );
END;
/
```
