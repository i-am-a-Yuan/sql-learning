### Ch3 SQL

数据定义语言 DDL：

- 每个关系的模式
- 每个属性的取值类型
- 完整性约束
- 每个关系维护的索引集合
- 每个关系的安全性和权限信息
- 每个关系在磁盘上的物理存结构

#### 基本类型

char(n) \ varchar(n) \ int \ smallint \ numeric(p, d) \ real,double precision \ float(n).

```sql
create table department
	(dept_name varchar (20), # A1  D1 # attribute domain
    building varchar (15),
    budget numeric(12,2),
    primary key (dept_name)); # constraint
```

```sql
primary key(Aj1, Aj2, ...);
foreign key(Ak1, Ak2, ...);
```

#### sql 定义

```sql
create table course
	(course_id	varchar (7),
    title	varchar (50),
    dept_name	varchar (20),
    credits	numeric (2,0),
    primary key (course_id),
    foreign key (dept_name) references department);
create table instructor
	(ID  varchar (5),
     name	varchar (20) not null, #### not null
     salary	varchar (20),
     primary key (ID),
     foreign key (dept_name) references department);
create table section
	(course_id	varchar (8),
    sec_id	varchar(8),
    semester	varchar (6),
    year	numeric (4,0),
    building	varchar (15),
    room_number	varchar	(7),
    time_slot_id	varchar	(4),
    primary	key (ID),
    foreign key (dept_name) references department);
create teaches
	(ID	varchar (5),
    course_id	varchar (8),
    sec_id	varchar (8),
    semester	varchar (6),
    year	numeric (4, 0),
    primary key (ID, couse_id, sec_id, semester, year),
    foreign key (course_id, sec_id, semester, year) references section,
    foreign key (ID) references instructor);
```

##### 新创建关系的插入

```sql
# insert tuple into table
insert into instructor
	values (10211, 'Smith', 'Biology', 66000);
# delete all tuples from table but keep table
delete from student;
# delete a particular table
drop table r;
# alter to append/delete attribute
alter table r add A D;
alter table r drop A;
```

#### Sql 查询

##### 单关系查询

```sql
select distinct dept_name from instructor;
select all dept_name from instructor;
select ID, name, dept_name, salary * 1.1
	from instructor;
select name from instructor 
	where dept_name = 'Comp. Sci. ' and salary > 70000;
```

##### 多关系查询

```sql
select name, instructor.dept_name, building
from instructor, department
where instructor.dept_name = department.dept_name
```

解释：1）from 做笛卡儿积 2）where 筛选 3）输出 select 的特定属性

```sql
# 先做笛卡尔积 再选择
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID;
# 指定部门 dept
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID and instructor.dept_name = 'Comp. Sci.';
# 由于 dept_name 在二者唯一 可简化
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID and dept_name = 'Comp. Sci.';
```

##### 自然连接

由于 `insnatructor` 和 `teaches` 中有相同名称的属性 `ID`，故而在不用 `where` 指定时，可以进行自然连接(natural join)。

```sql
# 原查询
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID;
# 自然连接
select name, course_id
from instructor natural join teaches;
# 二者结果相同
# 一般写成
select name, title
from instructor natural join teaches, course # 一个是自然连接的关系 另一个是关系course
where teaches.course_id = course.course_id;
# 一个失败的查询
select name, title
from instructor natural join teaches natural join course;
# 自然连接 (ID, name, dept_name, salary, course_id, sec_id, semester, year)
# 关系 (course_id, title, dept_name, credits)
# 包含两组相同的属性 course_id dept_name 要相同的话
# 那就不会有输出

```

所以需要指定相同的属性

```sql
select name, title
from (instructor natural join teaches) natural join course using (course_id);
```

#### 基本运算

##### 更名运算

```sql
# 重命名属性名
select name as instructor_name, course_id # 显示成新的属性名 但数据不变
from instructor, teaches
where instructor.ID = teahes.ID;
# 重命名关系名
select T.name, S.course_id
from instructor as T, teaches as S # T 和 S 称为 表别名 table alias
where T.ID = S.ID;
# 找出满足下面条件的所有教师的姓名 他们的工资至少比 Biology 系某一个教师的工资要搞
select distinct T.name
from instructor as T, intructor as S
where T.salary > S.salary and S.dept_name = 'Biology';
```

##### 字符串运算

```sql
like 'Intro%'; # 以 Intro 开头的字符
like '%Comp%'; # 匹配含有 Comp 的字符
like '___'; # 匹配三个字符
like '___%'; # 匹配至少三个字符
like 'ab\%cd%' escape '\'; # 匹配所有以 ab%cd 开头的字符串
```

##### *

```sql
select instructor.* from instructor, teaches where instructor.ID = teaches.ID;
```

##### order

```sql
select name from instructor where dept_name = 'Physics' order by name;
select * from instructor order by salary desc, name asc;
```

##### where 子句

```sql
select name from instructor where salary [not] between 90000 and 100000;
select name from instructor where salary <= 100000 and salary >= 90000;
select name, course_id from instructor, teaches 
where instructor.ID = teaches.ID and dept_name = 'Biology'; # 等价于如下
select name, course_id from instructor, teaches 
where (instructor.ID, dept_name) = (teaches.ID, 'Biology');
```

#### 集合运算

```sql
# 2009 Fall [CS-101 CS-347 PHY-101]
select course_id from section where semester = 'Fall' and year = 2009
# 2010 Spring [CS-101 CS-315 CS-319 CS-319 FIN-201 HIS-351 MU-199]
select course_id from section where semester = 'Spring' and year = 2010
# Union 并 自动去除重复 [CS-101 CS-315 CS-319 CS-347 FIN-201 HIS-351 MU-199 PHY-101]
(select course_id from section where semester = 'Fall' and year = 2009)
union
(select course_id from section where semester = 'Spring' and year = 2010)
# Union 并 保留重复 [CS-101 CS-101 CS-315 CS-319 CS-319 CS-347 FIN-201 HIS-351 MU-199 PHY-101]
(select course_id from section where semester = 'Fall' and year = 2009)
union all
(select course_id from section where semester = 'Spring' and year = 2010)
# Intersect 交 [CS-101]
(select course_id from section where semester = 'Fall' and year = 2009)
intersect [all]
(select course_id from section where semester = 'Spring' and year = 2010)
# Except 差 [CS-347 PHY-101]
(select course_id from section where semester = 'Fall' and year = 2009)
except [all]
(select course_id from section where semester = 'Spring' and year = 2010)

```

#####  空值

```sql
1 < null; not (1 < null); => unknown # 既不是 is null 也不是 is not null 除 t f 外的第三个逻辑值
true and unknown => unknown;false and unknown => false;	unknown and unknown => unknown;
true or unknown => true;	false or unknown => unknown;unknown or unknown => unknown;
not unknown => unknown;
if r.A is null => (1 < r.A) and not (1 < r.A) => unknown;

select name from instructor where salary is null;
```

元组中的null

```sql
where salary is null # 
('A', null) = ('A', null) # distinct 中 => true
where null = null # => unknown
```

#### 聚集函数

```sql
*avg;	min;	max;	*sum;	count; # * 作用于数字对象
```

##### 基本聚集

```sql
select avg (salary) as avg_salary # 计算平均工资并对字段重命名
from instructor
where dept_name = 'Comp. Sci.';
select count (distinct ID) # 按授课教师计数 对多个教师只计数一次
from teaches
where semester = 'Spring' and year = 2010;
select count (*) from course; # 计算课程的元组数 默认计数重复值
select count (distinct *) from course; # 这是不允许的
select min/max (distinct salary); # 使用min/max是允许distinct的
```

##### 分组聚集

```sql
# 找出每个系的平均工资
select dept_name, avg(salary) as avg_salary
from instructor
group by dept_name;
# 所有教师的平均工资
select avg (salary) from instructor;
# 找出每个系在2010年春季学期教授一门课程的教师人数
select dept_name, count (ID) as instr_count
from instructor natural join teaches # 按 ID 匹配
group by dept_name; # 属性应当出现在 select 中
# 一个错误的案例 
select dept_name, ID, avg (salary) # 无法确定哪个 ID 会被输出 查询失败
from instructor group by dept_name;
```

##### having 子句

```sql
# 查询教师平均工资在 42000 以上的系
select dept_name, avg (salary) as avg_salary
from instructor group by dept_name
having avg (salary) > 42000; # having 中属性也应当在 select 中出现
```

1）先从 from 中得到一个关系 2）根据 where 子句筛选元组 3）根据 group by 形成分组

4）根据 having 子句筛选分组 5）计算 select 属性的元组

```sql
# 对于在2009年讲授的每个课程段，如果课程段有至少2名学生选课，找出选修该课程段的所有学生的总学分(tot_cred)的平均值
select title, avg (tot_cred) as avg_tot_cred
from takes natural join course
where year = 2009
group by course_id, semester, year, sec_id
having count (ID) >= 2;
```

#### 嵌套子查询

##### 集合成员资格

```sql
# 找出在2009年秋季和2010年春季学期同时开课的所有课程
select distinct course_id
from section
where semester = 'Fall' and year = 2009
and course_id [not] in (select course_id
	from section
	where semester = 'Spring' and year = 2010);
# 找出既不叫"Mozart"，也不叫"Einstein"的教师
select distinct name
from instructor
where name not in ('Mozart', 'Einstein');
# 找出（不同的）学生总数，他们选修了ID为10101的教师所讲授的课程段
select count (distinct ID)
from takes
where (course_id, sec_id, semester, year) in 
	(select course_id, sec_id, semester, year
	from teaches
	where teaches.ID = '10101')
```

##### 集合的比较

在[3.4.1](#更名运算)中

```sql
# 找出满足下面条件的所有教师的姓名 他们的工资至少比 Biology 系某一个教师的工资要搞
select distinct T.name
from instructor as T, intructor as S
where T.salary > S.salary and S.dept_name = 'Biology';
# 也可以写成
select name
from instructor 
where salary > some 
	(select salary
	from instructor
	where dept_name = 'Biology');  # = some 等价于 in ; <> some 不等价于 not in
# 比所有 Biology 系的教师工资都高
# 也可以写成
select name
from instructor 
where salary > all 
	(select salary
	from instructor
	where dept_name = 'Biology'); # <> all 等价于 not in ; = all 不等价于 in

```

###### 说明

对于 some 和 all 的理解就是，[集合A] [逻辑符号] some [集合B] 意思就是对集合A的某个元素a，与[逻辑符号] [集合B]的所有元素得到了一组布尔值，some 的结果是对这组布尔值取或，而 all 对这组布尔值取与。对A[a], B[bcd] 来说，若 b < a < c < d，A > some B是否通过 =  (a > b) | (a > c) | (a > d) = 1，而 A > all B 是否通过 = (a > b) & (a > c) & (a > d) = 0。

因此，假设 $a = b \neq c \neq d$，A <> some B 就意味着 (a <> b) | (a <> c) | (a <> d) = 0 | 1 | 1 = 1，结果为 true，而 A not in B，结果为 false，故而二者不同。

```sql
# 找出平均工资最高的系
select dept_name, avg (salary)
from instructor
group by dept_name;

select dept_name
from instructor
group by dept_name
having salary > all
	(select avg (salary)
	from instructor
	group by dept_name);
```

##### 空关系测试

```sql
# 找出在2009年秋季和2010年春季学期同时开课的所有课程 # 另一种方式
select distinct course_id
from section as S
where semester = 'Fall' and year = 2009
and exists (select *
	from section as T
	where semester = 'Spring' and year = 2010
           and S.course_id = T.course_id);
# 先得到 S1 再从 S1 中筛选出 T1 的
```

##### 超集查询

A contains B := not exists (B except A) $A \supset B := B - A = \varnothing$

```sql
# not exists
# 找出选修了  Biology 系开设的所有课程的学生
# 即 某个学生的所有选修课程的集合 包含了 Biology 系的所有课程
select S.ID, S.name
from student as S
where not exists (
	(select course_id # id of Biology courses # 作为 B
	from course
	where dept_name = 'Biology')
	except
	(select course_id
    from takes as T
    where S.ID = T.ID));
```

##### 重复元组存在性测试?

若元组集合没有重复值，则 unique 返回 true *未被广泛实现？

```sql
# 找出所有在 2009 年最多开设一次的课程
select T.course_id
from course as T
where unique (select R.course_id
             from section as R
             where T.course_id = R.course_id and 
             	R.year = 2009);
# 一种等价表述
select T.course_id
from course as T
where 1 >= (select count (R.course_id)
             from section as R
             where T.course_id = R.course_id and 
             	R.year = 2009);
# 找出所有在 2009 年最少开设两次的课程
select T.course_id
from course as T
where not unique (select R.course_id
             from section as R
             where T.course_id = R.course_id and 
             	R.year = 2009);
```

##### from 子句中的子查询

select-from-where 的结果是一个关系，而 from 的输入也是一个关系，故而 from 中可以以一个查询结果作为输入。

```sql
# "找出系平均工资超过42000美元的那些系中教师的平均工资"
select dept_name, avg_salary
from (select dept_name, avg (salary) as avg_salary
      from instructor
      group by dept_name)
where avg_salary > 42000;
```

这个from 子查询类似于使用excel的过程了

也可以对 from 子查询得到的关系进行重命名（关系）

```sql
select dept_name, avg_salary
from (select dept_name, avg (salary) #as avg_salary
      from instructor
      group by dept_name)
      as dept_avg #(dept_name, avg_salary) Oracle 不允许
where avg_salary > 42000;
```

having 也有无能为力的时候

```sql
# 找出所有系中工资总额最大的系
select dept_name, max (tot_salary)
from (select dept_name, sum(salary) as tot_salary
     from instructor
     group by dept_name)

```

SQL:2003 lateral 支持 (mysql, IBM DB2) 	没有lateral 子句的话，子查询就不能访问外层查询的相关变量

```sql
# 打印每位教师的姓名、工资和他们系的平均工资
select name, salary, avg_salary
from instructor I1, lateral (select avg (salary) as avg_salary
                           from instructor I2
                            where I2.dept_name = I1.dept_name);
```

##### with 子句

SQL:1999 一个临时查询，临时定义命名一个查询的关系结果，避免了嵌套查询带来的阅读问题

```sql
# 找出具有最大预算值的系
with max_budget (value) as
	(select max (budget)
    from department)
select budget
from department, max_budget
where department.budget = max_budget.value;
# 查出所有工资总额大于所有系平均工资总额的系
with dept_total (dept_name, value) as
	(select dept_name, sum (instructor.salary)
    from instructor
    group by dept_name),
    dept_total_avg (value) as
    (select avg (value)
    from dept_total)
select dept_name
from dept_total, dept_total_avg
where dept_total.value > dept_total_avg.value;
```

##### 标量子查询

SQL允许子查询出现在返回单个值的表达式能够出现的任何地方，只要该子查询只返回包含单个属性的单个元组;这样的子查询称为标量子查询(scalar subquery)。例如，一个子查询可以用到下面例子的select子句中，这个例子列出所有的系以及它们拥有的教师数:

```sql
select dept_name,
	(select count (*)
    from instructor
    where department.dept_name = instructor.dept_name)
from department;
# 相当于
select dept_name, count (name)
from instructor
group by dept_name;
```

#### 数据库修改

delete 删除符合条件的整个元组

```sql
delete from r where q; # 删除符合q 的 r
delete from instructor where salary between 80000 and 200000;
delete from instructor; # 删除整个instructor
# 删除在 Watson 大楼工作的系的教师
delete from instructor
where dept_name in (select dept_name
                   from department
                   where building = 'Watson');
# 删除工资低于大学教师平均工资的教师记录
delete from instructor
where salary < (select avg (salary) 
               from instructor);
```

##### 插入

```sql
# 支持属性无序插入
insert into course (course_id, title, dept_name, credits)
	values ('CS-437', 'Database Systems', 'Comp. Sci. ', 4);
# 支持 select 的查询结果作为值插入
# 让 Music 系的每个修满144学分的学生称为 Music 系的老师 salary 18000
insert into instructor 
	select ID, name, dept_name, 18000
	from student
	where dept_name = 'Music' and tot_cred > 144;
```

#####  更新

```sql
# 给所有低于 70000 美元的教师涨工资 1.05 倍
update instructor
set salary = salary * 1.05
where salary < 70000;
# 支持 select 子查询
update instructor
set salary = salary * 1.03
where salary < (select avg (salary)
               from instructor);
# 有顺序的更新
update instructor
set salary = salary * 1.03
where salary > 100000;
update instructor
set salary = salary * 1.05
where salary <= 100000;
# case
update instructor
set salary = case
		when salary <= 100000 then salry * 1.05
		else salary * 1.03
	end;
-- 把学生的 tot_cred 设置为该生学完课程的学分综合 且 该课程学分不能为F和空
update student as S
set tot_cred = (select sum (credits)
               from takes natural join course
               where S.ID = takes.ID and  -- 使用了 update 的变量
               takes.grade <> 'F' and
               takes.grade is not null);
-- 把没有上完任何课的学生的学分设定为 0 
update student as S
set tot_cred = (
    select case 
    	when sum (credits) is not null then 
    		sum (credits)
    	else 0
    	end
    	    from takes natural join course
            where S.ID = takes.ID and  # 使用了 update 的变量
            takes.grade <> 'F' and
            takes.grade is not null)

```





### Case

instructor (ID, name, dept_name, salary)
course (course_id, title, dept_name, credits)
department (dept_name, building, budget)
section (course_id, sec_id, semester, year, building, room_number, time_slot_id)
teaches (ID, course_id, sec_id, semester, year)
takes (ID, course_id, sec_id, semester, year, grade)
student (ID, name, dept_name, tot_cred)
advisor (s_id, i_id)
classroom (building, room_number, capacity)
time_slot (time_slot_id, day, start_time, end_time)

