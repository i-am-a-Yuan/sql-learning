### Ch4-中级 SQL

##### on

```sql
-- a. 使用 where 指定隐式连接的条件 | ID 显示两次 都为 ID
select * from student, takes
where student.ID = takes.ID;
-- b. 使用 using 接受属性组 指定同一名称的属性 | ID 只显示一次
select * from student join takes using (ID)
-- c. 使用 on 指定显式连接的条件 | ID 显示两次 都为 ID
select * from student join takes on student.ID = takes.ID;
-- c 比 a 好在区分连接条件和其他条件 比 b 好在属性名无需相同
-- on 的表现在外连接中与 where 不同
```

很别扭地，要只显示一次 ID ，就只能进行重命名和罗列所有属性名

```sql
select student.ID as ID, name, dept_name, tot_cred, course_id, sec_id ...
```

以上的共性是，对于 takes 中没有出现的 ID，不会显示在结果当中，这就使得“查看每个学生的选课情况”变得不完美，不会显示没有选课的学生信息，要解决这个问题，需要外连接。

而所谓**自然连接**，就是指定了相同属性名作为了连接条件，故而 `natural` 不会和 `on` 或 `using` 同时出现，同时该属性名也只出现一次。

#### 外连接

自然连接 `from student natural join takes` 仅显示非空元组（在主键上二者都有的）

左外连接 保留运算符左边的部分的所有元组，对于不匹配的部分，添上空值null，右外连接同理。

```sql
select * from student -- takes 中并非所有学生都选课
natural left outer join takes;
-- 等价于 注意两个关系顺序！
select * from takes
natural right outer join student; -- 这时 takes 的元组会显示在表的左边 但是内容一致
```

整体效果：

```sql
select * from student left outer join takes on student.ID = takes.ID;
select * from student natural left outer join takes; -- 二者等价
```

*作为使用全外连接的例子，考虑查询:“显示Comp.Sci.系所有学生以及他们在2009年春季选修的所有课程段的列表。2009年春季开设的所有课程段都必须显示，即使没有Comp. Sci.系的学生选修这些课程段”。此查询可写为:

```sql
select * from
	(select * from student
    where dept_name = 'Comp. Sci.')
    natural full outer join
    (select * from takes
    where semester = 'Spring' and year = 2009);
```

##### 外连接中的on

前者 `select * from student left outer join takes on student.ID = takes.ID;` 的结果包括元组 (70557, Snow, Physics, 0, null, null, null,...)，而

```sql
select * from student left outer join takes on true
where student.ID = takes.ID;
```

的结果中不包含该元组。

在outer的外连接中，on 服务于外连接的子句，而 where 服务于整体查询的条件，是放在最后的。故而上句中从 left 连接得到的结果会排除掉 Snow 元组。

##### 常规连接

内连接 (inner join) = 常规连接，不显示不匹配的，默认的 join。

外连接 (outer join) 显示不匹配的。

连接类型： inner join, left outer join, right outer join, all outer join

连接条件： natural, on \<predicate\>, using (A_1, A_2, ...)

#### 视图

 安全（Safety）一个职员不应该看到教师的工资值，应该看到的是

```sql
select ID, name, dept_name
from instructor
```

用户视角（User Perspective）教师可以看到开设课程的信息与所在教室，希望有一个关于Physics系在2009年秋季学期所开设的所有课程段的列表，其中包括每个课程段在哪栋建筑的哪个房间授课的信息。

```sql
select course.course_id, 
from course, section
where course.course_id = section.course_id
and course.sec_id = 'Physics'
and course.semester = 'Fall'
and course.year = 2009;
```

查询后保存的结果独立于数据库，意味着查询结果不会随着数据库发生变化而改变。我们需要“虚关系”在需要的时候被执行，得到最新的数据库信息。

视图（View）对用户可见的虚关系，DBS存储与视图关系相关联的查询表达式，当视图被访问时，其中的元组才通过查询被创建出来。

###### 定义与使用

```sql
# 给职工看的教师表
create view faculty as
select ID, name, dept_name
from instructor;
# Physics 系 2009 年 秋季 课程
create view physics_fall_2009 as
  select course.course_id, 
  from course, section
  where course.course_id = section.course_id
  and course.sec_id = 'Physics'
  and course.semester = 'Fall'
  and course.year = 2009;
# 在刚才的 physics_fall_2009 中查询 在 Watson 大楼开设的课
select course_id
from physics_fall_2009 
where building = 'Watson';
# 创建每个系的所有工资总和的视图 departments_total_salary(dept_name, total_salary)
create view departments_total_salary(dept_name, total_salary) as
  select dept_name, sum(salary)
  from instructor
  group by dept_name;
```

若定义视图的关系被修改，视图就会过期。所以DBS存储视图的定义本身，而不存储它的查询结果。无论何时执行包含视图的查询，视图关系就会被重新计算。

###### 由视图定义的视图

对以上 Watson 大楼的 2009 秋季 Physics 课程信息，定义为一个视图。

```sql
create view physics_fall_2009_watson as
  select course_id
  from physics_fall_2009 
  where building = 'Watson';
```

##### 物化视图

物化视图（materialized view）**存储在数据库中的视图关系**。如果用于定义视图的实际关系发生改变，视图也跟着修改。

例如视图 departments_total_salary 是物化视图的话，instructor 中加入或删除或更改，那么其视图查询结果也会更改。

物化视图维护（materialized view maintenance）保持物化视图一直在最新状态的过程。即视图维护（view maintenance）

保持最新 或 周期性维护物化视图。频率高，响应快速；定义得多，回答快，存储开销大。

##### 更新

看起来，视图和关系有些类似，那么视图能不能更新？我们考虑

```sql
insert into faculty
  values ('30765', 'Green', 'Music')
```

但是视图来源是 instructor(ID, name, dept_name, salary) ，这个插入只包括3个值，只应该有两个可能：

- 拒绝插入，报错
- 给那个空的属性插入 null 

这个问题就是关于：视图对数据库的修改权力有多大？这个更改是合理合法的吗？

对于多表查询语句定义的视图，对这个视图的修改将关系到多个数据库表，该怎么办？

```sql
create view instructor_info as 
select ID, name, building 
from instructor, department
where instructor.dept_name = department.dept_name;
# 对该视图进行插入
insert into instructor_info
values ('69987', 'White', 'Taylor')
```

在关系 instructor 和 department 中，所插入的教师数据是原本没有的。那么难道唯一可能的方法是:向instructor中插人元组('69987',White’,null,null)，并向department 中插人元组(null，Taylor’，null)。

**不同的数据库系统指定了不同的条件以允许更新视图关系**。

##### 可更新的视图

updatable view

- from 子句中只有一个数据库关系
- select 子句中只包含关系的属性名，不包含任何表达式、聚集或 distinct 声明
- 任何没有出现在 select 子句中的属性可以取空值，没有 not null 约束，也不构成主码
- 查询中不含有 group by 或 having 子句。

例子：可以对以下视图进行 update/insert/delete

```sql
create view history_instructors as
select from instructor
where dept_name = 'History';
```

其中限制了 `dapt_name = 'History'`，所以插入元组 `('25566','Brown', 'Biology', 100000)` ，会在 `instructor` 中成功插入，但是不会显示。

SQL 默认允许这样的更新执行，在末尾加上 `with check option`，这样，对于视图的更新内容就必须满足视图中的条件定义，所以上述的元组就无法被插入了，会被拒绝。

？？？SQL：1999 对于视图上的 RUD 有更复杂的规则集。

#### 事务

transaction：由查询和（或）更新语句的序列组成。当一条 SQL 语句被执行，就隐式地开始了一个事务。

Commit Work：提交当前事务 / Rollback Work：回滚当前事务

##### 完整性约束

完整性约束（Integrity Constraints）：保证授权用户对数据库所作的修改不会破坏数据的一致性。防止对数据的意外破坏。

例子：非null / 两个教师的标识不能相同 / course中的 dept_name 必须在 department 中存在 / 预算必须为正数

完整性约束被看成是数据库模式设计过程中的一部分。两种构造约束的方式：

create table 中 或 alter table table-name add constraint  

##### 单个关系上的约束

```sql
primary key
not null
unique
check (<predicate>)
```

###### not null

除了主键外，名称等必要信息都不应该为空值

```sql
name varchar(20) not null
budget numeric(12, 2) not null
```

###### unique

```sql
unique(A1,A2,...Am)
```

unique声明指出属性（A1,A2,...,Am）形成一个候选码；即在关系中没有两个元组能在所有列出的属性上取值相同，候选码中的属性可以是null。

###### check

关系中的每个元组都必须满足谓词P。check 创建了一个强大的类型系统。例如 `check(budget > 0)`

```sql
create table section
  (course_id varchar (8),
  sec_id varchar (8),
  semester varchar (6),
  year numeric (4, 0)
  building varchar (15),
  room_number varchar (7),
  time_slot_id varchar (4),
  primary key(course_id, sec_id, semester, year),
  check(semester in ('Fall', 'Winter', 'Spring', 'Summer')));
```

check 应该像 where 一样后面可以跟任意的谓词，然而现在的数据库并不完全支持。

###### 参照完整性

参照完整性（referential integrity）子集依赖（subset dependency）

在外码声明中，course中的 dept_name 必须在 department 中存在

```sql
foreign key (dept_name) references department
```

**更一般地**，令关系 $r_1$ 和 $r_2$ 的属性集分别为 $R_1$ 和 $R_2$，主码分别为 $K_1$ 和 $K_2$。如果要求对 $r_2$ 中任意元组 $t_2$，均存在 $r_1$ 中元组 $t_1$ 使得 $t_1.K_1=t_2.\alpha$，我们称 $R_2$ 的子集 $\alpha$ 为参照关系 $r_1$ 中 $K_1$ 的外码（foreign key）。**这意味着外码和主码一样可以指定多个属性。**

参照完整性约束要求，只要 $r_2$ 中的 $\alpha$ 属性值能在 $r_1$ 中的 $K_1$ 中找到即可，不要求 $K_1$ 是主键。而**外键约束要求** $K_1$ 是主键。默认情况下，SQL中外码参照的是被参照表中的主码属性。

显式指定被参照关系的属性列表，这个属性列表必须是一个**候选码**，即需要 `primary key` 约束或 `unique` 约束。？？也可以不必是候选码p4.4.7

当违反参照完整性约束时，通常的处理是拒绝执行导致完整性破坏的操作（即进行更新操作的事务被回滚）。但是在 **foreign key**子句中可以指明：如果被参照关系上的删除或更新动作违反了约束，那么系统必须采取一些步骤通过修改参照关系中的元组来恢复完整性约束，而不是拒绝这样的动作。例如 course 中：

```sql
create table course
  (...
  foreign key (dept_name) references department
      on delete cascade
      on update cascade,
  ...);
```

###### 级联操作

> 由于有了与外码声明相关联的 **on delete cascade** 子句，如果删除department 中的元组导致了此参照完整性约束被违反，则删除并不被系统拒绝，而是对course关系作“级联”删除，即删除参照了被删除系的元组。类似地，如果更新被参照字段时违反了约束，则更新操作并不被系统拒绝，而是将course中参照的元组的dept_name 字段也改为新值。SQL还允许 foreign key 子句指明除 cascade 以外的其他动作，如果约束被违反:可将参照域(这里是dept_name)置为null(用set null代替cascade)，或者置为域的默认(用 set default)。

对于删除操作 **delete**，被参照表中元组被删除，则参照表的外码无法找到对应的主码（外码约束），在级联删除的外码声明下，**course** 中的对应的 dept_name 也会被一并删除。即 当 department 中的 `dept_name = Biology` 被删除的时候，course 中所有 `Biology` 系的课也一并删除。

对于更新操作 **update**，被参照表中的元组被更新，级联更新的外码声明下，参照表中的该属性值也同步更新。即在 department 中的 dept_name 一旦变动， course 中的所属系也全部变动。

可以想象，一个被参照表可以有多个参照表，故而一个外码依赖链涉及多个关系，也可以是自身。**如果一个级联更新或删除导致的对约束的违反不能通过进一步的级联操作解决，则系统中止该事务。**于是，该事务所作的所有改变及级联动作将被撤销。（猜测：？？因此如果外键参照自身，那么它恐怕很难被轻易更改，需要慎重考虑）

一个简单的、有所帮助的例子：

```sql
create table foo (
    a varchar(10) primary key,
    b varchar(10),
    c varchar(10)
);

create table bar (
    d varchar(10) primary key,
    e varchar(10),
    f varchar(10),
    foreign key (e) references foo(a) -- 显式地指明外键
        on update cascade
        on delete set null -- 不同的处理方式
);
```

###### 事务中对完整性约束的违反

一个事务可能包括几个步骤，在某一步之后完整性约束也许会暂时被违反，但是后面的某一步也许就会消除这个违反。例如，在关系 **person(name, spouse)**中，插入一对配偶**(Mary, John), (John, Mary)**，无论先擦汗如哪个元组，都会先违反外码约束，然后又满足外码约束。

为了应对**暂时的完整性约束的违反**，SQL标准允许将**initially deferred** 子句加入到约束声明中；这样完整性约束不是在事务的中间步骤上检查，而是在事务结束的时候检查。**一个约束可以被指定为可延迟的（defferrable）**，这意味着默认情况下它会被立即检查，但是在需要的时候可以延迟检查。对于声明为可延迟的约束，执行 **set constraints constraint-list deferred** 语句，会导致对指定约束的检查被延迟到该事务结束后执行。

许多数据库（MySQL\\SQL Server）不支持延迟约束，默认做法是**立即检查约束**（initially immediate）。PostgreSQL/Oracle/SQLite支持 **deferrable initailly deferred**。

在上例中，如果spouse不是not null，那么就可以先插入含null的元组，再设置其spouse 属性的值。

###### QA:

这是我自己设想的一个例子，能帮助解答一些疑惑：

> 关系foo(a, b, c)都为varchar(10)，a是主键 bar(d, e, f, g)都为varchar(10)，d是主键 指定bar中e是foo的a的外键，保证e有update的级联操作，但是删除a后要在e属性置为null 
> 指定foo中c是bar的f外键，不支持f更新的级联操作，但是删除f后c更新默认值"unknown" 指定bar中g是bar的d的外键

回答：

```sql
create table foo (
    a varchar(10) primary key,
    b varchar(10),
    c varchar(10) default 'unknown'
);

create table bar (
    d varchar(10) primary key,
    e varchar(10),
    f varchar(10) unique,
    g varchar(10),
    constraint fk_bar_e
        foreign key (e)
        references foo(a)
        on update cascade
        on delete set null,
    constraint fk_bar_g
        foreign key (g)
        references bar(d)
);

alter table foo
add constraint fk_foo_c
foreign key (c)
references bar(f)
on delete set default;
```

###### 复杂check条件与断言

在外键约束中，可以通过级联或默认的方式应对被参照关系的变化。check条件的谓语中如果出现子查询，则同样复杂得多。

```sql
check (time_slot_id in (select time_slot_id from time_slot))
```

这个check关联了 section 和 time_slot，需要应对二者的变动。
一个是当section中加入了元组时，要满足check条件；
另一个是当time_slot变动时，也要满足check条件。

复杂check条件需要应对当前关系的变动，也要满足子查询涉及的更新，开销是巨大的。

**断言（assertion）**，一个谓词构成的数据库条件。
包含域约束（domain constraint）和参照完整性约束（referential integrity constraint），如 `check (semester in ('Fall', 'Winter', 'Spring', 'Summer'))`、`check (year > 1701 and year < 2100) `和 `foreign key (course_id) references course 	on delete cascade`。它们是断言的特殊形式。但是，依然有很多约束不能仅用这几种特殊形式来表达。

- 对于student关系中的每个元组，它在属性tot_cred 上的取值必须等于该生所成功修完课程的学分总和。
- 每位教师不能在同一个学期的同一个时间段在两个不同的教室授课。

断言的形式：**create assertion \<assertion-name\> check \<predicate\>**

由于SQL不提供“for all X, P(X)”结构（其中P是一个谓词），我们只好通过等价的“not exists X such that not P(X)”结构来实现此约束，表示如下：

```sql
create assertion credits_earned_constraint check
	(not exists (select ID
                from student
                where tot_cred <> (select sum(credits)
                                  from takes natural join course
                                  where student.ID = takes.ID
                                  and grade is not null and grade <> 'F')))
```

对应的，第二个约束应该是

```sql
create assertion teaches_classroom_constraint check (
    not exists (
        select ID, semester, year, time_slot_id, building, room_number
    	from teaches natural join section
        group by ID, semester, year, time_slot_id
    	having count(distinct building || ',' || room_number) > 1));
```



------

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

