### 1. topic和partition相关的很多细节，比如副本同步策略，分区分配策略等
#### 1. kafka选择了全部完成同步，才发送ack，虽然延迟高点，但是数据不会大量冗余
#### 2. 分区分配策略：
1. RoundRobin，轮询发送给每个消费者
2. Range 根据消费者数量进行分配每个消费券消费的Partition的范围
### 2. kafka如何保证exactly once
1. ack=-1 保证所有副本都ack才发送ack
2. 幂等性：生产者在初始化时候会被分配一个PID，broker会对《pid，PArtition，seqNumber作为缓存》，相同的前缀只会持久化一次
### 3. Kafka作为消息队列，它可解决什么样的问题？
1. 解耦
2. 可恢复性
3. 缓冲
4. 峰值处理
5. 异步通讯
### 4. kafka总结下来真的就高吞吐、高可靠、高可用三点
#### 1. 高吞吐：
生产者异步、压缩、批量发送啦、网络模型I/O多路复用高效啦、写入pageCache啦、顺序I/O啦、baseOffset形成跳表啦、零拷贝啦
#### 2. 高可靠：
如何做到不重不漏不乱序？
1. exactly one 不重不漏
2. 不乱序列：LEO（每个副本最大的offset和HW（全部最小的leo）
* 1. leader故障，从新选举后，follower从新leader的HW开始向后全部删除。再重新跟leader同步数据
* 2. followe故障，从上次的Hw后全部删除，跟leader同步数据，如果leo追上leader的HW，就可以加入isr
#### 3. 高可用：
Controller HA、PartitionHA
1. ISR，用来选举新leader的表格，如果超过延迟时间会把Follower提出ISR，存入OSR（新加入的OSR存放的地方）
### 5. kafka的零拷贝？
这个我从那个经典例子，将一个硬盘中的文件通过网络发送出去讲述了零拷贝，Gather Copy以及mmap零拷贝

