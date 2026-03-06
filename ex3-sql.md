### Ch3

##### 3.1

```sql
-- a.
select title
from course
where dept_name = 'Comp. Sci.' and credits = 3;
-- b. 错误 应该从授课关系中找到教师
select distinct ID
from takes
where course_id in (select course_id
                   from teaches natural join instructor
                   where instructor.name = 'Einstein');
-- b.ans
select distinct takes.ID
from takes, instructor, course
where takes.course_id = teaches.course_id and
	takes.sec_id = teaches.sec_id and
	takes.semester = teaches.semester and
	takes.year = teaches.year and
	teaches.ID = instructor.ID and
	instructor.name = 'Einstein';
-- c.
select max (salary)
from instructor;
-- d.
select all name
from instructor
where salary = (select max (salary) from instructor);
-- e. 1 空缺时显示NULL
select course_id, sec_id,
	(select count(ID)
    from takes
    where takes.course_id = section.course_id and 
    takes.sec_id = section.sec_id and 
    takes.semester = section.semester and
    takes.year = section.year)
    as enrollment
from section
where semester = 'Fall' and year = 2009;
-- e. 2 这个方案更好 会显示0
select course_id, sec_id,
(select count(ID)
from takes
where takes.course_id = section.course_id and 
    takes.sec_id = section.sec_id and 
    takes.semester = section.semester and
    takes.year = section.year) as enrollment
from section
where section.semester = 'Fall' and section.year = 2009
group by course_id, sec_id, semester, year;
-- e. 1 wrong  问题在于如果没有主键和section一一对应那么0不会出现
select course_id, sec_id, count(ID)
from takes
where semester = 'Fall' and year = 2009
group by course_id, sec_id, semester, year;

-- f. 之后同一用 group by 方法
select max(enrollment) as max_enrollment from (
select count(ID) as enrollment
    from section, takes
    where takes.course_id = section.course_id and 
    takes.sec_id = section.sec_id and 
    takes.semester = section.semester and
    takes.year = section.year and
    takes.semester = 'Fall' and takes.year = 2017
    group by takes.course_id, takes.sec_id
    );

-- f. you can use 'with' subquery, which is better and could show 0 if no students takes any section
with sec_enrollment as 
(select takes.course_id, takes.sec_id, count(ID) as enrollment
from section, takes
where takes.course_id = section.course_id and 
    takes.sec_id = section.sec_id and 
    takes.semester = section.semester and
    takes.year = section.year and
 	takes.semester = 'Fall' and takes.year = 2009
group by takes.course_id, takes.sec_id)
select max(enrollment) from sec_enrollment;
-- g.
with sec_enrollment as 
(select takes.course_id, takes.sec_id, count(ID) as enrollment
from section, takes
where takes.course_id = section.course_id and 
    takes.sec_id = section.sec_id and 
    takes.semester = section.semester and
    takes.year = section.year and
 	takes.semester = 'Fall' and takes.year = 2009
group by takes.course_id, takes.sec_id)
select course.id from sec_enrollment
where enrollment = (select max(enrollment) from sec_enrollment);


```

```sql
-- a1.
select course_id, sec_id, semester, year, 
	(select count(ID) -- 对 section 的每一行做子查询
     from takes
     where takes.course_id = section.course_id and 
     takes.sec_id = section.sec_id and 
	 takes.semester = section.semester and
     takes.year = section.year) as enrollment
from section
where section.semester = 'Fall' and section.year = 2009;
-- b1.
select section.course_id, section.sec_id, semester, year, count(ID) as enrollment -- 自然连接后做分组聚集
from section, takes
where takes.course_id = section.course_id and 
    takes.sec_id = section.sec_id and 
    takes.semester = section.semester and
    takes.year = section.year and
    section.semester = 'Fall' and section.year = 2009
group by takes.course_id, takes.sec_id;
-- b.2
select section.course_id, section.sec_id, semester, year, count(ID) as enrollment -- 自然连接后做分组聚集
from section join takes on
	takes.course_id = section.course_id and 
    takes.sec_id = section.sec_id and 
    takes.semester = section.semester and
    takes.year = section.year and
where section.semester = 'Fall' and section.year = 2009
group by takes.course_id, takes.sec_id;
```

##### 3.2

```sql
-- a.
select sum(grade_points.points * course.credits)
from takes natural join grade_points, course
where takes.course_id = course.course_id and ID = '12345'
group by ID;
-- a.ans not perfect
select sum(credits * points)
from takes, course, grade_points
where takes.grade = grade_points.grade and
	and takes.course_id = course.course_id and
	ID = '12345';
-- a.ans2
(select sum(credits * points)
from takes, course, grade_points
where takes.grade = grade_points.grade and
	and takes.course_id = course.course_id and
	ID = '12345')
union
(select 0 -- 如果该学生在 student 中 且不在 takes 中
from student
where ID = '12345' and
	not exists (select * from takes where ID = '12345'))
```

```sql
-- b.
(select sum(credits * points) / sum(credits) as GPA
from takes, course, grade_points
where takes.grade = grade_points.grade and
	and takes.course_id = course.course_id and
	ID = '12345')
union
(select null as GIA
from student -- 同样补足
where ID = '12345' and
	not exists (select * from takes where ID = '12345'))
```

```sql
-- c.
(select ID, sum(credits * points) / sum(credits) as GPA
from takes, course, grade_points
where takes.grade = grade_points.grade and
	and takes.course_id = course.course_id and
	ID = '12345')
union
(select ID, null as GPA # join 
 # appear the student who has not taken any course
from student
where ID = '12345' and
	not exists (select * from takes where takes.ID = student.ID))
```

##### 3.3

```sql
-- a.
update from instructor
set salary = salary * 1.10
where dept_name = 'Comp. Sci.';
-- b. 错了
delete from section
where section.sec_id in (
(select distinct sec_id from takes)
    except
(select distinct sec_id from section));
-- b. right
delete from course
where course_id not in 
	(select course_id from section )
-- c.
insert into instructor -- 省略 values()
	select ID, name, dept_name, 10000
	from student
    where tot_cred >= 100);
```

##### 3.4 

```sql
-- a. 一个人出过多次事故 算同一人
select count(*)
from participated natural join car
where year = 2009;
-- a.
select count()
from accident, participated, person
where accident.report_number = participated.report_number and
	participated.driver_id = person.driver_id and
	accident.date between date '2009-01-01' and date '2009-12-31';
-- b.
insert into accident -- 事故信息 事故编号 日期 地点
	values (4007, '2009-09-01', 'Berkeley');
insert into participated
	select o.driver_id, c.license, 4007, 3000
	from person p, owns o, car c
	where p.name = 'Jones' and p.driver_id = o.driver_id and
		o.license = c.license and c. model = 'Toyota';
-- c.
delete from car
where model = 'Mazda' and
	license in (select license -- 从 is 改成 in 因为 个人的车牌号不唯一
              from person p, owns o
              where p.driver_id = o.driver_id
              and p.name = 'John Smith');
-- 仅仅删除了 车辆存在 该车的 车牌号关系和事故关系依然存在
```

##### 3.5

```sql
-- a.
select ID, 
	case
		when (score < 40) then 'F'
		when (score < 60) then 'C'
		when (score < 80) then 'B'
		else 'A'
from marks;
-- b.
with grades as
(select ID, 
	case
		when (score < 40) then 'F'
		when (score < 60) then 'C'
		when (score < 80) then 'B'
		else 'A'
 	end as grade -- 不要忘了 end as
from marks)
select grade count(grade) -- 不要忘了 grade 必须要包含 group by 里的
from grades
group by grade;
```

##### 3.6

```sql
select dept_name
from deptment
where dept_name like (select lower(dept_name)
                     from deptment)
-- 错了
select dept_name
from department
where lower(dept_name) like '%sci%'
```

##### 3.7 

```sqlite
select	distinct p.a1
from p, r1, r2
where p.a1 = r1.a1 or p.a1 = r2.a1
-- 只有r1 r2 都不为空的时候 这个查询查找p.a1与 r1.a1 或 r2.a1 中相同的值
-- 如果 r1 r2 至少有一个为空 这个查询得到的笛卡尔积为空 所以结果为空
-- 同样如果 p 为空 结果也为空
```

##### 3.8

```sql
-- a. 1 错误
select customer_name
from depositor
where customer_name not in (select customer_name
                           from borrower);
-- 1修改后
select distinct customer_name
from depositor d
where not exists (select 1 from borrower b
                 where b.customer_name = depositor.customer_name);

-- a. 2s mysql 不支持 except
(select customer_name
from depositor)
except
(select customer_name
from borrower)

-- b. 名字为 Smith 的人不唯一
with smith as
(select customer_street, customer_city 
from customer
where customer_name = 'Smith')
select customer_name
from customer as C, smith as S
where C.customer_street = S.customer_street
and C.customer_city = S.customer_city
-- 超复杂 存在逻辑问题 不能把 street 和 city 分开来匹配
select customer_name
from customer as C, smith as S
where C.customer_street in
	(select customer_street
     from customer
     where customer_name = 'Smith')
and C.customer_city in
	(select customer_city
     from customer
     where customer_name = 'Smith')
-- 但是可以使用 子查询再匹配
select customer_name
from customer as C, smith as S
where (customer_street, customer_city) in
	(select customer_street, customer_city
     from customer
     where customer_name = 'Smith')
-- b.2 同时满足两个属性相同的关系 同一个关系内部使用自连接
select F.customer_name
from customer F join customer S using(customer_street, customer_city)
where S.customer_name = 'Smith'
-- b.3 使用子查询的自连接
select customer_name
from customer C
where exists (select 1  --  select 1 很常用
             from customer S
             where C.customer_street = S.customer_stree
             and C.customer_city = S.customer_city);
-- c.
select branch_name
from depositor d, account a, customer c
where d.customer_name = c.customer_name
and d.account_name = a.account
and c.customer_city = 'Harrison'
```

##### 3.9 employee dbs fig3-20

```sql
-- a. 正确
select employee_name, city
from works, employee
where works.employee_name = employee.employee_name
and works.company_name = 'First Bank Corporation';
-- b. 
select employee_name, street,city
from works, employee
where works.employee_name = employee.employee_name
and works.company_name = 'First Bank Corporation'
and works.salary > 10000;
-- b. 2可以改进
select e.employee_name, e.street, e.city
from works as w join employee as e on works.employee_name = employee.employee_name
where works.company_name = 'First Bank Corporation'
and w.salary > 10000
-- b. 参考
select * from employee
where employee_name in 
	(select employee_name
	from works
    where works.company_name = 'First Bank Corporation'
	and works.salary > 10000)
-- c.
select employee_name, city
from employee
where not exists (select 1
                 from works
                 where company_name = 'First Bank Corporation'
                 and works.employee_name = employee.employee_name
)
-- c. reference
select employee_name
from employee
where employee_name not in 
	(select employee_name -- 是主键 故而没有 null 
     from works
     where company_name = 'First Bank Corporation'
    );
-- d.
select employee_name
from works as S
where salary > all (select salary -- 错了 在 all 的子查询中不应该进行连接 不能名称相同 这改变了语义
                   from works as T
                   where S.employee_name = T.employee_name 
                    and company_name = 'Small Bank Corporation')
-- d. reference
select employee_name
from works as S
where salary > all (select salary
                   from works as T
                   where company_name = 'Small Bank Corporation')
-- d. 如果考虑一个员工在不止一个公司 计算工资总和 问题就困难了
with emp_total_salary as
(select employee_name, sum(salary)as total_salary
from works
group by employee_name
)
select employee_name
from emp_total_salary
where total_salary > all
(select total_salary
from emp_total_salary, works
where works.company_name=’Small Bank Corporation’and
emp_total_salary.employee_name=works.employee_name
)
-- e.不懂 应该是找到和 SBC 同一个城市的所有公司 但是
select T.company_name
from company T
where (select R.city
      from company R
      where R.company_name = T.company_name)
      contains
      (select S.city
      from company S
      where S.company_name = 'Small Bank Corporation')
-- e. 仅使用 标准 SQL 因为 contains 已经被废除了
select S.company_name
from company S  -- A contains B <=> not exists (B except A)
where not exists (
    (select city -- 符合的城市
     from company
     where company_name = 'Small Bank Corporation')
    except
    (select R.city -- 作为超集 传递到父查询
      from company R
      where R.company_name = S.company_name));
-- f. 中规中矩 CTE
with company_emp_cnt as
(select company_name, count(employee_name) as count_employee
from works
group by company_name)
select company_name
from company_emp_cnt
where count_employee = (select max(count_employee) from company_emp_cnt);
-- f. 一步到位 使用了 distinct
select company_name
from works
group by company_name
having count(distinct employee_name) >= all
	(select count (distinct employee_name)
    from works
    group by company_name)

-- g. 中规中矩 CTE
with company_salary_avg as
(select company_name, avg(salary)) as salary_avg
from works
group by company_name)
select company_name
from company_emp_cnt
where salary_avg > (select salary_avg
                    from company_salary_avg
                   where company_name = 'First Bank Corporation');
-- g. better
select company_name
from works
group by company_name
having avg(salary) >
	(select avg(salary)
    from works
    where company_name = 'First Bank Corporation')
```

##### 3.10 fig3-20

```sql
-- a. 正确 没问题
update employee
set city = 'Newtown' where employee_name = 'Jones'
-- b. 
update works as W
set salary = case
	when salary <= 100000 then salary * 1.10
	when salary > 100000 then salary * 1.03
	end
where W.employee_name in (select M.manager_name
                         from managers as M)
and W.company_name = 'First Bank Corporation'
-- 小改进 中英两版有所不同 注意
update works as W
set salary = salary *
	(case
		when salary <= 100000 then 1.10
		else 1.03
	end
where W.employee_name in (select M.manager_name
                         from managers as M)
and W.company_name = 'First Bank Corporation'

```

##### ex3.11

```sql
-- Find the names of all students who have taken at least one Comp. Sci. course; make sure there are no duplicate names in the result.
-- a.1 正确的
select distinct S.name
from student as S
where exists (
	(select course_id 
    from course as C
    where dept_name = 'Comp. Sci.')
	intersect -- 交 Mysql 不支持
	(select course_id
    from takes as T
    where T.ID = S.ID))
-- a.2 正确的
select distinct S.name
from student as S, takes as T
where S.ID = T.ID
and T.course_id in (select course_id -- 主键非空
                   from course as C
                   where dept_name = 'Comp. Sci');
-- a.2 改进
... from student join takes on ID
-- a.3 reference
select distinct name -- 最好加上 distinct
from student natural join takes natural join course
where course.dept_name = 'Comp. Sci'
-- Find the IDs and names of all students who have not taken any course offering before Spring 2009.
-- b. -- 逻辑错误 section 不代表开设时间 例 CS-101 2008 2010
select distinct S.ID, S.name 
from student as S
where not exists ( -- 二者相交的集合为空
    (select course_id
     from takes as T
     where S.ID = T.ID)
    intersect
    (select course_id
     from section where year < 2009))
-- b. reference
select ID, name
from student
except
select ID, name
from student natural join takes -- 排除所有在2009年之前的选课
where year < 2009;
-- c. 简单即正确
select dept_name, max(salary)
from instructor 
group by dept_name;
-- d. 简单即正确
select dept_name, min(max_salary)
from (select dept_name, max(salary) as max_salary
from instructor 
group by dept_name);
-- group by dept_name;

```

##### 3.12 [](2026年3月4日)

```sql
-- a
insert into course
values ("CS-001", "Weekly Seminar", "Comp. Sci.", 0)
-- b. 
insert into section
values ("CS-001", 1, "Fall", 2009, 122, "A") -- 后面3个 null
-- c.
insert into takes
	select ID, "CS-001", 1, "Fall", 2009, 0 -- 0 改成 null
	from student;
	where dept_name = "Comp. Sci.";
-- d.
delete from takes
where ID = (select ID from student
           where name = "Chavez")
and course_id = "CS-001";
-- d. reference
delete from takes
where course_id = 'CS-001' and section_id = 1
and year = 2009 and semester = 'Fall'
and ID in (select ID from student --  姓名为 Chavez 的学生不唯一
           where name = "Chavez");

-- e. reference
-- 删除课程 CS-001 但是事先没有在 takes teaches section 中删除和 CS-001 相关的内容，会发生什么
-- section/takes/teaches 中的 外键 course_id 参照于 course 
-- 所以我们必须首先删除在 takes 中的元组 然后才能删除 course 中的 CS-001
-- 作为外键冲突的结果，执行删除的交易会发生回滚
-- As a result of the foreign key violation, the transaction that performs delete would be rolled back
-- 也就是会删除失败
-- f.
-- 类似地，为了避免执行失败的发生，我们需要首先删除takes中有关 database 的部分
delete from takes 
where course_id in
	(select course_id from course
    where lower(title) like '%datebase%')


```

##### 3.13 - fig3.18

```sql
create table person
(	driver_id	varchar(10),
	name		varchar(40),
	address		varchar(100),
	primary key (driver_id));
create table car
(	license		varchar(8),
	model		varchar(30),
	year		int
	primary key (license));
create table accident
(	report_number	varchar(6), -- 后面例题给的是 AR2197 应该字符串更合适
	date		numeric,
	location	varchar(100),
	primary key (report_name));
create table owns
(	-- license		varchar(8) not null,
    driver_id	varchar(10),
    license		varchar(8),
    primary key (driver_id),
	foreign key (driver_id) references on person,
	foreign key (license) references on car);
create table participated 
(	report_number	varchar(6),
 	license	varchar(8),
 	driver_id	varchar(10),
 	damage_amount	int,    
    primary key (report_number, license)
    foreign key (report_number) references on accident,
	foreign key (license) references on license,
	);
```

person (driver_id, name, address)
car (license, model, year)
accident (report_number, date, location)
owns (driver_id, license)
participated (*report_number, *license, driver_id, damage_amount)

##### 3.14

```sql
-- a.
select count(1) -- 计算的是 参与数 而不是 事故数
from participated natural join owns natural join person -- 改用显示join
where person.name = "John Smith"; -- 属性的值应该用单引号'' 或 ``
-- 双引号作为标识符和别名
-- a. reference
select count(distinct P.report_number)
from participated P on A.report_number = P.report_number
join owns O on P.license = O.license
join person Pe on O.driver_id = Pe.driver_id
where person.name = 'John Smith';
-- a. alternative
select count(*)
from accident
where exists
	(select *
    from participated, owns, person
    where owns.driver_id = person.driver_id
    and person.name = 'John Smith'
    and accident.report_number = participated.report_number)
-- b.
update participated
set damage_amount = 3000
where report_number = "AR2197"
and license = "AABB2000";
```

branch (branch_name, branch_city, assets)
customer (customer_name, customer_street, customer_city)
loan (loan_number, branch_name, amount)
borrower (customer_name, loan_number)
account (account_number, branch_name, balance)
depositor (customer_name, account_number)

##### 3.15

```sql
-- a. Find all customers who have an account at all the branches located in “Brooklyn”
-- a. my answer
select distinct customer_name -- distinct 可以限定多个属性 区别于 unique
from depositors as D
where not exists (
(select branch_name
from (account natural join account) as A
where branch_city = "Brooklyn"
and D.account_number = A.account_number)
except
(select branch_name
from branch
where branch_city = "Brooklyn") -- 不确定对不对
)
-- a. my answer
select distinct customer_name -- distinct 可以限定多个属性 在 select 后
-- unique 用于 where 判断子查询是否存在重复行
from depositor as D
where not exists (
    (select branch_name
	from branch
	where branch_city = 'Brooklyn')
    except
    (select A.branch_name
	from (account natural join account) as A
	where branch_city = 'Brooklyn'
	and D.account_number = A.account_number)
	))
-- a. better answer
select D.customer_name
from depositor D
where not exists (
    (select B.branch_name
	from branch B
	where B.branch_city = 'Brooklyn')
	except
	(select distinct A.branch_name
	from depositor D2
	join account A on D2.account_number = A.account_number
	join branch B2 on A.branch_name = B2.branch_name
	where D2.customer_name = D.customer_name
	and B2.branch_city = 'Brooklyn')
);

-- a. reference
with branchcount as
	(select count(*)
    from branch
    where branch_city = 'Brooklyn')
select customer_name
from customer c
where branchcount = 
	(select count(distinct branch_name)
    from (customer natural join depositor natural join account
         natural join branch) as d
    where d.customer_name = c.customer_name)


```

##### 3.16

```sql
-- a.
select employee_name
from works
where company_name = 'First Bank Corporation';
-- b.
select employee_name
from employee E 
join works W on E.employee_name = W.employee_name
join company C on W.company_name = C.company_name
where E.city = C.city;
-- c.
select E1.employee_name
from employee E1
join managers M on E1.employee_name = M.employee_name
join employee E2 on M.manager_name = E2.employee_name
where E1.street = E2.street
and E1.city = E2.city;
-- d.
select employee_name from works as W join
(select company_name, avg(salary) as avg_salary
from works
group by company_name) as A
on W.company_name = A.company_name
where W.salary > A.avg_salary;
-- d. reference
select employee_name
from works T
where salary > (select avg(salary)
               from works S
               where T.company_name = S.company_name)
-- e.
select company_name from
(select company_name, sum(salary) as sum_salary
from works
group by company_name) as A
where A.sum_salary = min(A.sum_salary) -- 错的
-- e. reference
select company_name
from works
group by company_name
having sum(salary) <= all (select sum(salary)
                          from works
                          group by company_name)

```



##### 3.17-fig3.20

```sql
-- a.
update works
set salary = salary * 1.10;
where company_name = "First Bank Corporation"
-- b.
update works
set salary = salary * 1.10
where employee_name in
(select manager_name from managers)
and company_name = "First Bank Corporation"
-- c.
delete from works
where compnay_name = "Small Bank Corporation"
```

employee (employee_name, street, city)
works (employee_name, company_name, salary)
company (company_name, city)
managers (employee_name, manager_name)

##### 3.18

有的数据暂时没有录入，有待补充或更改；以空白 null 作为初始状态；作为一种条件判断而存在便于查询；

##### 3.19

<> all === not in

```sql
```



##### 3.20



### Case

instructor (ID, name, dept_name, salary)
course (course_id, title, dept_name, credits) 课程信息
department (dept_name, building, budget)
section (course_id, sec_id, semester, year, building, room_number, time_slot_id) 开设的课程班次， 课程段
teaches (ID, course_id, sec_id, semester, year) 教师授课
takes (ID, course_id, sec_id, semester, year, grade) 学生选课
student (ID, name, dept_name, tot_cred)
advisor (s_id, i_id)
classroom (building, room_number, capacity)
time_slot (time_slot_id, day, start_time, end_time)

