### 1.  给个联合索引的例子，问会不会走索引？联合索引的底层？
### 2. Hive count(distinct)有几个reduce 海量数据会有什么问题
### 3. 了解HiveSQL吗？讲讲分析函数？
### 4. hive遇到过慢查询吗？（比如有的map任务很慢）如何解决？
#### 1. 数据倾斜处理方法
1. groupBY
2. mapjoin
3. 开启数据倾斜时负载均衡
4. 设置多Reduce个数
### 5. 一条HQL从代码到执行的过程
### 6. Hive的文件存储格式都有哪些？
### 7. HQL：行转列、列转行
### 8. HQL：一张表字段有 id，name，date，其中id有重复，问如何拿到最新的date对应的id的name？
### 9. Hive 内部表和外部表的区别
### 10. 如果现在有个hql，是select count(*) from 表，这个是怎么翻译为MR任务的（就解释了一下计算key的方法，map并行执行任务，reduce的一个过程）
### 11. hive有什么简单的优化方法
1. MapJoin
2. 行列过滤
3. 列式存储
4. 采用分区技术
5. 合理设置Map和Reduce数量
6. 合并小文件，开启JVM重用
7. 采用压缩
8. 采用tez和spark引擎
### 12.hive SQL怎么转化为MapReduce的



