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
