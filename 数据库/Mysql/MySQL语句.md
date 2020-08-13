# 查询

## 模糊查询

一般模糊查询语句为：

```mysql
SELECT 字段 FROM 表 WHERE 某字段 Like 条件
```

一般会使用到符号%，它表示0个或多个字符。

**示例**

```mysql
#查询姓呱的学生名单
select * from student_info where student_name like '呱%';
#查询姓名中最后一个字是呱的学生名单
select * from student_info where student_name like '%呱';
#查询姓名里有呱字的学生名单
select * from student_info where student_name like '%呱%';
```

**使用索引**

我们在name列建立一个普通索引，当查询的关键字在开头时：

```
explain select * from student_info where student_name like '猴%';
```

我们发现显示结果的possible_keys和key都显示sql语句使用到了name索引。

> `possible_keys` 表示 MySQL 在查询时, 能够使用到的索引. 注意, 即使有些索引在 `possible_keys` 中出现, 但是并不表示此索引会真正地被 MySQL 使用到. MySQL 在查询时具体使用了哪些索引, 由 `key` 字段决定.

但如果查询的关键字是'%猴%'或者'%猴'，则不会使用到索引。



## 查询汇总和分组查询

（1）查询课程号为'0002'的总成绩

```mysql
 select sum(score) from score_info where course_id='0002'
```

（2）查询选课的学生人数，并将结果放在自定义的student_cnt里，其中distinct表示去重。

```mysql
 select count(distinct student_id) as student_cnt from score_info 
```

（3）查询各科成绩的最高分和最低分，并按照课程号划分组：

```mysql
 select course_id, max(score) as high_score, min(score) as min_score
 from score_info
 group by course_id;
```

（4）查询每门课程被选修的学生数

```mysql
 select course_id, count(student_id)
 from score_info
 group by course_id;
```

（5）查询学生中男生，女生人数。group by表示按照sex来区分count(*)的值，若没有 group by sex 则count返回的是总人数。

```mysql
 select sex,  count(*)
 from student_info
 group by sex;
```

（6）查询平均成绩大于60分学生的学号和平均成绩

```mysql
 select student_id, avg(score)
 from score_info
 group by student_id
having  avg(score) > 60

# where 后面跟着查询语句，此处不会用到
# group by 分表示按什么分组，此处是学号
# having 表示对分组结果指示条件，此处的条件是平均成绩大于60分
```

（7）查询至少选修两门课程的学生学号

```mysql
 select student_id, count(score) as choose_course_cnt
 from score_info
 group by student_id
 having count(score) >=2
```

（8）查找同名同姓学生名单并统计同名人数（至少有2个重名才统计）

```mysql
 select student_name, count(*) 
 from student_info
 group by student_name
 having count(*) >=2
```

（9）查询成绩小于70的课程并按课程号从大到小排列

```mysql
 select course_id
 from score_info
where score <70
 order by course_id desc
 
#order by放在having 后面
```

（10）查询每门课程的平均成绩，结果按平均成绩升序排序，平均成绩相同时，按课程号降序排列

```mysql
 select course_id, avg(score) as avg_score
 from score_info
 group by course_id 
 order by avg_score, course_id desc;
```







# 参考资料

[explain各字段含义](https://segmentfault.com/a/1190000008131735#item-3-4)

[查询面试题](https://zhuanlan.zhihu.com/p/38354000)