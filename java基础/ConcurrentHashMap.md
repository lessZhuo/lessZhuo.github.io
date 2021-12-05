## 1 put() 方法
#### JDK1.7中的实现：  
ConCurrentHashMap 和 HashMap 的put()方法实现基本类似，所以主要讲一下为了实现并发性，ConCurrentHashMap 1.7 有了什么改变   
需要定位 2 次 （segments[i]，segment中的table[i]）   
由于引入segment的概念，所以需要   
1. 先通过key的 rehash值的高位 和 segments数组大小-1 相与得到在 segments中的位置
2. 然后在通过 key的rehash值 和 table数组大小-1 相与得到在table中的位置

没获取到 segment锁的线程，没有权力进行put操作，不是像HashTable一样去挂起等待，而是会去做一下put操作前的准备：   
1. table[i]的位置(你的值要put到哪个桶中)
2. 通过首节点first遍历链表找有没有相同key
3. 在进行1、2的期间还不断自旋获取锁，超过 64次 线程挂起！

#### JDK1.8中的实现：
先拿到根据 rehash值 定位，拿到table[i]的 首节点first，然后：
1. 如果为 null ，通过 CAS 的方式把 value put进去
2. 如果 非null ，并且 first.hash == -1 ，说明其他线程在扩容，参与一起扩容
3. 如果 非null ，并且 first.hash != -1 ，Synchronized锁住 first节点，判断是链表还是红黑树，遍历插入。

## 2 get() 方法
#### JDK1.7中的实现：
由于变量 value 是由 volatile 修饰的，java内存模型中的 happen before 规则保证了 对于 volatile 修饰的变量始终是 写操作 先于 读操作 的，并且还有 volatile 的 内存可见性 保证修改完的数据可以马上更新到主存中，所以能保证在并发情况下，读出来的数据是最新的数据。  
如果get()到的是null值才去加锁。
#### JDK1.8中的实现：
和 JDK1.7类似

##  3 resize() 方法 
#### JDK1.7中的实现：
跟HashMap的 resize() 没太大区别，都是在 put() 元素时去做的扩容，所以在1.7中的实现是获得了锁之后，在单线程中去做扩容（1.new个2倍数组 2.遍历old数组节点搬去新数组）。
#### JDK1.8中的实现：
jdk1.8的扩容支持并发迁移节点，从old数组的尾部开始，如果该桶被其他线程处理过了，就创建一个 ForwardingNode 放到该桶的首节点，hash值为-1，其他线程判断hash值为-1后就知道该桶被处理过了。

## 4 计算size
#### JDK1.7中的实现：
1. 先采用不加锁的方式，计算两次，如果两次结果一样，说明是正确的，返回。
2. 如果两次结果不一样，则把所有 segment 锁住，重新计算所有 segment的 Count 的和

#### JDK1.8中的实现：
由于没有segment的概念，所以只需要用一个 baseCount 变量来记录ConcurrentHashMap 当前 节点的个数。
1. 先尝试通过CAS 修改 baseCount
2. 如果多线程竞争激烈，某些线程CAS失败，那就CAS尝试将 CELLSBUSY 置1，成功则可以把 baseCount变化的次数 暂存到一个数组 counterCells 里，后续数组 counterCells 的值会加到 baseCount 中。
3. 如果 CELLSBUSY 置1失败又会反复进行CASbaseCount 和 CAScounterCells数组

触发扩容的操作
![Image](https://user-images.githubusercontent.com/91466593/144745972-9aadaff0-5f5c-4d2c-8f1c-e5da350e65cb.png)
总结一下：
(1) 元素个数达到扩容阈值。
(2) 调用 putAll 方法，但目前容量不足以存放所有元素时。
(3) 某条链表长度达到8，但数组长度却小于64时。
![Image](https://user-images.githubusercontent.com/91466593/144745987-dff05131-0304-407c-b690-16ce60cb60b9.png)

CPU核数与迁移任务hash桶数量分配的关系
![Image](https://user-images.githubusercontent.com/91466593/144745993-948a1bac-00db-4f1f-a0f6-900d879f50f8.png)
单线程下线程的任务分配与迁移操作
![Image](https://user-images.githubusercontent.com/91466593/144746007-0d40347f-c294-43cd-a677-47cf840048a2.png)
多线程如何分配任务？
![Image](https://user-images.githubusercontent.com/91466593/144746015-90d31999-d470-422d-9fde-2d4e2d29558e.png)

普通链表如何迁移？
![Image](https://user-images.githubusercontent.com/91466593/144746020-29520547-0118-4d77-9a70-4fbed4819d0c.png)

什么是 lastRun 节点？
![Image](https://user-images.githubusercontent.com/91466593/144746028-20447522-61cd-43d7-a0ad-b4caf98d33ab.png)

红黑树如何迁移？
![Image](https://user-images.githubusercontent.com/91466593/144746031-b583cba6-e3fe-4fa1-8803-04b245962d96.png)

hash桶迁移中以及迁移后如何处理存取请求？
![Image](https://user-images.githubusercontent.com/91466593/144746042-04319400-178e-4885-a270-79feaa78d450.png)

多线程迁移任务完成后的操作

![Image](https://user-images.githubusercontent.com/91466593/144746043-d9108376-25f0-41a6-b542-0015424469e2.png)

