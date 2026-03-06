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

##### 外连接中on

前者 `select * from student left outer join takes on student.ID = takes.ID;` 的结果包括元组 (70557, Snow, Physics, 0, null, null, null,...)，而

```sql
select * from student left outer join takes on true
where student.ID = takes.ID;
```

的结果中不包含该元组。

在outer的外连接中，on 服务于外连接的子句，而 where 服务于整体查询的条件，是放在最后的。故而从 left 连接得到的结果会排除掉 Snow 元组。

##### 常规连接

内连接 (inner join) = 常规连接，不显示不匹配的，默认的 join。

外连接 (outer join) 显示不匹配的。

连接类型： inner join, left outer join, right outer join, all outer join

连接条件： natural, on \<predicate\>, using (A_1, A_2, ...)

#### 视图

####  