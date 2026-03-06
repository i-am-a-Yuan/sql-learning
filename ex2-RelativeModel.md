本文档为《数据库系统概念 第六版》的习题回复，仅做练习和交流使用，不保证正确性。

参考答案：[《数据库系统概念第六版》配套课后习题答案_人人文库网](https://www.renrendoc.com/paper/211097282/211100042.html#view)

### Ch2 关系模型介绍

##### 2.1

主码：关系中的一个候选码；一个候选码是一个最小的超码，超码是一个或多个属性的集合，其取值唯一识别关系中的元组。 

图2-14中关系中，employee : person-name;works : person-name;company : company-name.

##### 2.2

外码(foreign key)约束，对于参照关系(referencing relation)中作为外码的属性中任意的值，都要在被参照关系(referenced relation) 中找到唯一的对应。

`instructor.insert(10111, Ostrom, Economics, 110,000)`，此时 dept_name 作为从 instructor 到 department 的外键，而department中dept_name 中不存在值 `Economics`，违背了参照完整性约束(referential integrity constraint)。

`department . delete(Biology, Watson, 90,000)`，此时在 instructor 中 dept_name 查找不到在`departmetn`中对应的值 `Biology`。

##### 2.3

一门特定的课在一天中的特定时间开始，但不会在多于一个特定时间结束。课程时间由一个特定的日期和一个特定的开始时间决定。

##### 2.4

一个特别的关系实例没有重复的教师名，不代表这样的一个关系模式中的教师名不会重复，基于现实考量，一个教师表的设计必须考虑重复的教师名，故而不能将教师名 `instrcutor-name` 作为主码。

##### 2.5

$\sigma_{s\_id = ID}(student \times advisor)$

对于每个有指导教师的学生，在学生信息的后面追加一列指导教师的ID，若一个学生有多个指导教师，则该学生信息重复多次，若一个学生没有指导教师，则该学生信息不出现在该表（关系）中。

##### 2.6

a. $(\sigma_{year\geq 2009}(takes) \Join student)$  ：在上课表中提取出2009年及以后的学生信息，并根据ID追加筛选出的学生的学术信息；

b. $(\sigma_{year \geq 2009} (takes \Join student))$  ：在上课表后根据ID追加每个学生的信息，在筛选出2009年及以后的学生，结果和a.相同，a.的先筛选后连接更好。

c. $\prod_{ID, name, course\_id}(student \Join takes)$  ：在上课表后根据ID追加每个学生信息，但是只保留ID、学生姓名、课程号。

##### 2.7

a. $\prod_{person-name} (\sigma_{city = ''Miami''}(employee))$

b. $\prod_{person-name} (\sigma_{salary \geq 100,000}(works))$

c.  $\prod_{person-name} (\sigma_{city = ''Miami'' \and slary \geq 100,000}(employee \Join works))$

##### 2.8

a. $\prod_{branch_name} (\sigma_{branch-city=Chicago}(branch))$

b. $\prod_{customer_name} (\sigma_{branch_name=Downtown}(borrower \Join loan))$

##### 2.9

branch(*branch_name*,  ))



### Ch3 SQL