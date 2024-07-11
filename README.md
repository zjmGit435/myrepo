# myrepo
## 位图
位图就是
## 布隆过滤器
布隆过滤器也是利用哈希函数衍生的数据结构，主要就是一个位图，多个hash函数
对于两个值A，B，有三个hash函数，得到其hash值为 1 3 5； 2 3 6
因此位图就是  0101010 
              0011001
所以只要1 3 5位置有0，说明不存在A，如果2 3 6位置有0，说明不存在B；
如果1 3 5位置为1，说明可能存在A；如果2 3 6位置为1，说明可能存在B
但是存在误判的概率
## MySQL执行一条语句的流程
1. 连接MySQL数据库，使用TCP三次握手连接，会**检查权限**
2. 查询缓存，如果是SELECT语句，命中直接返回结果
3. 分析器解析SQL语句，分为多个字段（命令）解析，分为词法和语法分析，前者提取关键字，后者检查语法错误
4. 优化器优化SQL，比如优化索引查询方式等等，然后**检查权限**，执行
5. 引擎将结果返回给Server层（回表），然后判断是否满足查询条件
## weak_ptr如何解决循环引用
weak_ptr构造一个对象不会增加引用计数，比如：
'''C++
shared_ptr<A> a;
weak_ptr<A> b;
'''
b不会增加A对象引用计数，因此释放a就会调用析构函数释放掉b。
注意weak_ptr是可以通过lock()方法提升为强引用指针
## git rebase 和 merge的区别
git checkout A
git rebase B
将B的根拼接到A最新的位置后面
A ：a->b->c
B ：d->e->f
rebase:a->b->c->d->e->f

git checkout A
git merge B (把B merge 到A)
A ：a->b->c
B ：d->e->f
rebase:
      |c|->d->e->f
a->b->|c|
rebase会把当前分支的根连接到要合并分支最新的节点，从而变成同一条流

git提交方法：
git add .
git commit -m "first commit" 
git rebase -i HEAD~n  会把最近n个提交弄出来，可以选择合并这些节点为一个提交等等，但是历史记录不会丢失
git push -f origin br_a  强推到分支br_a

## SQL学习
### 查询
1. SELECT * FROM table WHERE feature <> 2 ORDER BY feature; （排序）
2. SELECT * FROM table DISTINCT WHERE feature = 2 ORDER BY feature DESC; (去重，降序)
### 连接
1. SELECT * FROM table1 JOIN table2 ON table1.feature = table2.feature；（直接合并，如果不匹配的行，比如有一列属性为空，会被忽略）
2. SELECT * FROM table1 LEFT JOIN table 2 ON  table1.feature = table2.feature; (合并，但是不匹配的行也会被放进来，只不过为空的会被写为NULL，注意这个是保留所有table1的行)
3. SELECT * FROM table1 RIGHT JOIN table 2 ON  table1.feature = table2.feature;(合并，但是不匹配的行也会被放进来，只不过为空的会被写为NULL，注意这个是保留所有table2的行)
4. GROUP BY feature(聚合：将结果按照指定的列进行聚合，如果不加可能查询结果会少)
5. DATEDIFF(A, B)= 1  (时间差，得到满足B-A的日期为1的条件)
### 自连接和交叉连接
自连接就是只用自身表，但是需要取别名
交叉连接：两张表，生成笛卡尔积，也就是交集
### 创建一张新表查表
SELECT * FROM (SELECT feature FROM table1 WHERE feature IS NULL) as a; 
#### 题目：1280. 学生们参加各科测试的次数
输入：
Students table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
Subjects table:
+--------------+
| subject_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+
Examinations table:
+------------+--------------+
| student_id | subject_name |
+------------+--------------+
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |
+------------+--------------+
输出：
  +------------+--------------+--------------+----------------+
  | student_id | student_name | subject_name | attended_exams |
  +------------+--------------+--------------+----------------+
  | 1          | Alice        | Math         | 3              |
  | 1          | Alice        | Physics      | 2              |
  | 1          | Alice        | Programming  | 1              |
  | 2          | Bob          | Math         | 1              |
  | 2          | Bob          | Physics      | 0              |
  | 2          | Bob          | Programming  | 1              |
  | 6          | Alex         | Math         | 0              |
  | 6          | Alex         | Physics      | 0              |
  | 6          | Alex         | Programming  | 0              |
  | 13         | John         | Math         | 1              |
  | 13         | John         | Physics      | 1              |
  | 13         | John         | Programming  | 1              |
  +------------+--------------+--------------+----------------+
解释：
结果表需包含所有学生和所有科目（即便测试次数为0）：
Alice 参加了 3 次数学测试, 2 次物理测试，以及 1 次编程测试；
Bob 参加了 1 次数学测试, 1 次编程测试，没有参加物理测试；
Alex 啥测试都没参加；
John  参加了数学、物理、编程测试各 1 次。
1. 首先需要统计每个人考每科的次数，使用Examinations表即可完成:
    SELECT Examinations.student_id, Examinations.subject_name， COUNT(*) AS attended_exams FROM Examinations GROUP BY Examinations.student_id, Examinations.subject_name;
    | student_id | subject_name | attended_exams |
    | ---------- | ------------ | -------------- |
    | 1          | Math         | 3              |
    | 1          | Physics      | 2              |
    | 1          | Programming  | 1              |
    | 2          | Programming  | 1              |
    | 13         | Math         | 1              |
    | 13         | Programming  | 1              |
    | 13         | Physics      | 1              |
    | 2          | Math         | 1              |
2. 可以看到少了6，因为6的结果是0，默认会不记录，因此需要使用一张全记录表来匹配：
    SELECT * FROM Students CROSS JOIN Subjects
    | student_id | student_name | subject_name |
    | ---------- | ------------ | ------------ |
    | 1          | Alice        | Programming  |
    | 1          | Alice        | Physics      |
    | 1          | Alice        | Math         |
    | 2          | Bob          | Programming  |
    | 2          | Bob          | Physics      |
    | 2          | Bob          | Math         |
    | 13         | John         | Programming  |
    | 13         | John         | Physics      |
    | 13         | John         | Math         |
    | 6          | Alex         | Programming  |
    | 6          | Alex         | Physics      |
    | 6          | Alex         | Math         |
3. 接下来只需要将两个表拼接，而且一定要保留第二张表的所有内容，因此将第一张表LEFT JOIN 即可，然后由于有一些值会被填充为NULL，所以将其替换为0即可
    SELECT Students.student_id, Students.student_name,sub.subject_name, IFNULL(a.attended_exams,0) as attended_exams
    FROM Students
    CROSS JOIN Subjects
    LEFT JOIN (SELECT Examinations.student_id, Examinations.subject_name， COUNT(*) AS attended_exams FROM Examinations GROUP BY Examinations.student_id, Examinations.subject_name) AS a
    ORDER BY Students.student_id, sub.subject_name;
    | student_id | student_name | subject_name | attended_exams |
    | ---------- | ------------ | ------------ | -------------- |
    | 1          | Alice        | Math         | 3              |
    | 1          | Alice        | Physics      | 2              |
    | 1          | Alice        | Programming  | 1              |
    | 2          | Bob          | Math         | 1              |
    | 2          | Bob          | Physics      | 0              |
    | 2          | Bob          | Programming  | 1              |
    | 6          | Alex         | Math         | 0              |
    | 6          | Alex         | Physics      | 0              |
    | 6          | Alex         | Programming  | 0              |
    | 13         | John         | Math         | 1              |
    | 13         | John         | Physics      | 1              |
    | 13         | John         | Programming  | 1              |

   
