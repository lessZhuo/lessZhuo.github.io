###	1. hadoop 三大组件简介
####	1. NameNode
1. 管理namespace
2. 配置副本策略-？
3. 管理数据块的映射
4. 处理客户端的读写请求
####	2.DataNode
1. 存储实际的数据块
2. 执行真正的读写操作
3. 文件块设置大小取决与寻址时间和磁盘传输速度
####	3. SecondNameNode
1. 紧急情况下恢复NameNode
2. 辅组NameNode，分担工作量，定期合并Fsimage和Edits
###	2. Hdfs写流程中如果DataNode突然宕机了怎么办？
###	3. Hdfs写流程中如果DataNode 使用NameNode的好处
###	4. 使用NameNode的好处（或者说有中心节点的好处）
###	5. datanode， namenode的工作原理
###	6. SeconderyNameNode 的作用
###	7. 块大小及其原因
###	8. Hdfs读数据流程 源码级别（要回答出来rpc）
###	9. Hdfs写数据流程 源码级别（要回答出来rpc）速度限制、
#### 1.HDFS客户端通过DistributedFileSystem做好上传文件的事情；
* 1. 新建一个RPC客户端，发送给NameNode进行请求，检查文件路径是否存在，是否有权限添加，返回给客服端应答
* 2. 客户端创建一个DataStreamer，在DataStreamer里面创建一个队列DataQueue，DataQueue进行挂起等待，同时DataStreamer从NameNode通过机架感知获取Block块可以写到哪些DataNode上（距离最近的datanode节点，机架感知可以知道最合适的副本节点，不同机架不同设备）
#### 2.然后通过 FsDataNodeOutStream模块向DataNode请求建立数据传输通道，节点1收到后会跟后续节点进行建立通道，每个节点跟后续每个节点建立通道，然后所有节点逐级应答客户端请求；最后客户全部收到后表示建立成功
* 1. 然后开始真正写数据，先写chunk和checksum，127个形成一个packet，packet会直接向dataqueue写数据（会先检查是不是满了）
* 2. packet写完之后会通知dataqueue里面已经有数据，可以将数据向datanode写出，DataStreamer将数据通过Socket发出
1. 将dataQueue里面对应的packet发送给DataNode一份
2. 将对应的packet移除，放入AckQueue队列队尾
#### 3.DataNode接受对应的数据，先持久化到磁盘，再向下一节点发送
* 1. 发送数据后会逐级应答，会应答到ResponerProssor，ResponerProssor收集了所有点应答，会把ackQueue的packet删除，如果没有收集所有应答，会让AckQueue发送给DataQueue，然后再进行发送
###	10. Mapreduce shffule原理 源码级别（要回答出来锁  多线程 以及缓存写磁盘交换）
###	11. YARN 的任务提交流程
1. MR程序提交到客户端所在的节点(yarnRunner)
2. YarnRunner向ResourceManager申请一个Application
3. ResourceMa'nager返回所需要应用程序的资源路径给YarnRunner
4. 应用程序把资源提交到HDFS上
5. YarnRunner向RSmanager申请运行MRAppMaster
6. RM将用户请求初始化为一个task，放入任务调度器里面
7. 当NodeManager领到这个任务时候，将创建一个容器，并且在里面运行MRappMaster，并将刚刚所提交的资源拷贝到本地
8. MRAM向RM申请运行maptask，RM将maptask分派给N个Nodemanager
9. NM领到任务后创建容器，MRappmaster向N个NM发送程序启动脚本，运行MAPTASK
10. 运行结束后再向ResourceManager申请运行ReduceTask
11. 运行结束后MRappMaster会向RM申请注销自己
###	12. wordcount 细节
###	13. mr给一个特别大的数据集排序，分区间要有序怎么做
###	14. MapReduce的过程？为什么要排序
####	1. Map阶段
1. Read阶段：MapTask通过用户编写的RecordReader，从输入的inputsplit中解析出一个个key/value
2. Map阶段：把解析出来的key/value通过用户编写的map（）函数进行处理，生成新的key/value
3. Collect阶段：处理完后，通过调用OutputCollctor.collect进行处理，生成分区，并将其刷写进入环形缓冲区
4. Spill阶段：当环形缓冲区达到溢写阈值时候，会将数据进行分区排序，并且分区内key排序，或者压缩等处理，最后刷到到磁盘一个小文件。
5. Combine阶段；所有数据处理完之后，会对所有的临时文件进行合并成一个大文件，并且生成相应的索引文件，合并过程中是以分区单位进行递归合并。
####	2. Reducer阶段
1. Copy阶段：Reduce从Maptask上拷贝一份数据，如果大小太大则放入磁盘，不然九九放入内存。
2. Merge阶段：拷贝数据同时，启动两个线程对内存和磁盘上的文件进行合并，防止磁盘文件过多
3. Sort阶段：根据对key进行进行聚合处理，由于之前MapTask已经分区内排序过，最后只需要进行一次归并排序就行。最后作为Reduce（）函数输入
4. Reduce阶段：根据用户编写的reduce（）函数输出到HDFS上
###	15. MR 过程中有几次排序过程
###	16. Map默认是HashPartitioner 如何自定义分区
1. 编写一个类继承Partitioner，重写getPartition方法，然后在job里面指定用这个类分区
###	17. Mapreduce 为什么适合适合大数据存储
###	18. PB级大数据处理时，比如join操作，如何优化
###	19. 是否搭建过Hadoop集群，怎么搭建的
###	20. 数据倾斜问题怎么解决
###	21. 发生了数据倾斜怎么处理
1. 不影响业务情况下，在map端提前聚合
2. 增加reducer个数
3. 设置合理分区
4. 或者对于同个key过大，可以增加随机值在key上面，然后通过两次reducer，先局部聚合再全局聚合
###	22. map join 为什么能解决数据倾斜
###	23. 两个表都很大怎么去解决数据倾斜
###	24. Hadoop和spark的区别是什么
###	25. hadoop1.0和2.0的区别

