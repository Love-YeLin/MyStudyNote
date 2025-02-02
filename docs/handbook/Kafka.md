# Kafka

1.什么是kafka
<details>
<summary>答案</summary>
<p>kafka是一个分布式的发布-订阅消息系统和一个强大的队列，可以处理大量的数据，并可以将消息从一个端点传递到另一个端点。Kafka适合离线和在线消费消息。Kafka消息保存在磁盘上，并在集群内复制以防止数据丢失。</p>
</details>

2.为什么要使用kafka
<details>
<summary>答案</summary>
<p>1.缓冲和消峰：上游数据有突发流量时，下游可能抗不住，而kafka可以在中间起一个缓冲的作用，把消息暂存在kafka中，下游可以慢慢的消费kafka中的消息</p>
<p>2.解耦和扩展性:消息队列可以作为一个接口层，解耦重要的业务流程。</p>
<p>3.异步处理:有些操作并不需要立即执行，可以将其写入kafka，异步执行</p>
<p>4.kafka可以堆积请求，即使消费者挂掉也不影响主要业务的正常进行</p>
<p>5.通过kafka可以使得一个生产的消息可以被不同业务的消费者消费</p>
</details>


3.kafka的架构
<details>
<summary>答案</summary>
<p>kafka包含多个核心组件 消费者，生产者，Broker,Topic,Partition,Zookeeper,Controller,Replication</p>
<p>消费者从broker从取消息</p>
<p>生产者向broker发消息</p>
<p>Broker是一个kafka实例，kafka集群由多个broker组成，一个broker包含多个topic</p>
<p>kafka通过topic对消息分类，topic可以看作是一个队列,生产者向某个topic发消息，消费者从某个topic取消息</p>
<p>Partition是为了实现扩展性，提高并发能力，将一个topic分成多个Partition，每个Partition都是一个有序队列，每个Partition分布在不同的broker</p>
<p>Replication 用于实现备份的功能，保证集群中某个节点故障，该分区的数据不会丢失并且能够正常工作，一个partition有多个副本，一个副本有一个leader和多个follower</p>
<p>leader 每个分区多个副本中的主副本，负责接收生产者发送的消息，负责给消费者提供消息</p>
<p>follower 每个分区多个副本中的从副本，负责从主副本中同步数据，当主副本宕机的时候，还会成为新的主副本</p>
<p>offset 表示消费者消息的位置信息</p>
<p>zookeeper 负责存储和管理kafka的集群信息</p>
</details>

4.kafka会丢失消息吗？如何解决
<details>
<summary>答案</summary>
<p>
首先消息的传递有3个阶段，从生产者发送给主副本，消费者从主副本消费数据</p>
<p>
在生产者发送给主副本的这个阶段，有一个配置参数ack,ack=0表示生产者发送消息之后，不等待主副本的响应直接返回，很容易造成消息丢失。ack=1表示当接收到主副本接收成功就放回，ack=-1或all时表示所有主副本和从副本都接收成功时才表示成功 
</p>
<p>
在主副本存储数据的阶段，是先将数据写入操作系统缓冲区，再异步刷盘，所以在刷盘之前宕机可能丢失数据，但是kafka可以通过配置实现同步刷盘，也可以通过多分区多副本机制，最大限度保证数据不丢失。在从副本拉去到主副本的数据之前，主副本宕机，新主副本也可能丢失数据
</p>
<p>
在消费者从主副本消费数据的阶段，有两个操作，一个是处理数据，一个是提交offset，这个操作的顺序可以由系统参数解决，先处理数据，再提交offset会导致，可能在提交offset之前,消费者宕机，会导致消息被重复消费，如果先提交offset，再处理数据，会导致数据丢失。
</p>
</details>
5.导致kafka消费顺序乱序的原因？如何解决
<details>
<summary>答案</summary>
<p>
1.一个主题存在多个分区，消息分散在不同的分区上，导致消息乱序
2.消费者重试机制，一个消费者消费失败并决定重试，而同一消费组的另一个消费者已经消费了新的消息，导致消息乱序
3.生产者ACK机制中开启ack=0，先发送的数据因为网络拥塞而延迟，后发送的数据先到达，导致消息乱序
解决办法
1.一个主题只设置一个分区
2.生产者通过key指定发往的分区，从而保证有序
3.将ack参数设置为1
</p>
</details>

6.Kafka组消费之Rebalance机制
<details>
<summary>答案</summary>
<p>rebalance，让所有消费者达成共识。触发Rebalance机制的条件包括消费组成员发生变化，分区数量发生变化,订阅的主题数量发生变化</p>
<p>rebalance机制的主要流程</p>
<p>当消费组刚创建时，每个消费者会创建消费者协调器实例，然后获取对应的组协调器，向组协调器请求加入消费组。第一个加入消费组的消费者将成为leader，然后leader将进行选择分区分配策略。包括按分区号排序进行均分，顺序轮流分配，均衡并且尽量保持与上次相同。分配好分配后将分区结果同步给消费者</p>
</details>

7.Kafka如何保证高可用
<details>
<summary>答案</summary>
<p>1.Kafka采用集群架构，由多个broker组成，每个broker存储一部分数据，当某个broker宕机，其他broker也可以正常工作</p>
<p>2.kafka通过数据冗余来保证高可用，每个主题由多个分区组成，每个分区分布在不同的broker上，并在多个broker上复制，即使某个broker故障，也可以从其他的broker获取数据</p>
<p>3.消费组 kafka的消费组可以保证消息的高可用，一个消费组包含多个消费者，每个消费者负责某个分区的消息，当某个消费者宕机，其他消费者会接替他的工作</p>
<p>4.监控和故障转移 kafka会实时监控集群的状况，当某个broker出现故障时，会进行故障转移，将该broker的分区迁移到其他的broker上。保证数据的可用性</p>
</details>


8.Kafka的ISR机制
<details>
<summary>答案</summary>
<p>
ISR是指同步副本集，与leader保持同步的所有副本的集合。当某个副本，落后leader太多时，会被移除ISR列表，当落后的副本追上leader时，又会重新加入ISR列表，当leader宕机时，会从ISR列表从选取一个副本作为leader。在生产者的ACK机制中，ack=-1或all时，也需要等待所有ISR列表中的副本都收到消息时，才返回响应。从而保证kafka的可靠性和可用性
</p>
</details>

9.Kafka的LEO和HW机制
<details>
<summary>答案</summary>
<p>LEO表示最新的日志偏移量，分为leader leo, follower local leo, follower remote leo， leader leo 表示主副本的最新偏移量，当有日志写入时，这个值会被更新。follower local leo是存储在follower 副本上的最新偏移量,当follower收到从leader拉取到的数据时，会更新该值。follower remote leo是指存储在leader副本上的follower的最新偏移量,当leader收到follower的拉取请求的时候，会更新该值。</p>
<p>HW表示高水位，表示已经被所有副本接收的最大日志偏移量，分为 leader hw, follower hw, leader hw表示主副本的高水位，当有日志写入或者有follower拉取数据或者有follower宕机或者副本成为leader时，会更新leader hw, leader hw 值为 leader leo 和 follower remote leo 取min。follower hw表示从副本的高水位,当follower收到从leader拉取的数据时，会更新该值为follower local leo 和 leader hw的min值</p>
<p>Leader Epoch 表示当前版本号对应的起始偏移量,可以使得副本重启后不再以来HW来对日志进行截断，使得数据不一致和丢失。当副本重启后，根据当前副本的版本号，向leader拉取最后一个offset，然后进行截断。如果当前节点成为leader,则更新leader epoch</p>
工作流程：
<p>
Leader收到消息，更新leader leo
Follower请求拉取数据
Leader收到请求拉取数据
更新follower remote leo
更新leader hw = min(leader leo, min(follower local leo...))
follower 收到拉取的数据
follower 更新 follower local leo
follower 更新 follower hw = min(leader hw, follower local leo)
</p>
</details>

10.Kafka如何防止消息积压
<details>
<summary>答案</summary>
<p>1.增加消费者的数量，可以提高消费的速度</p>
<p>2.增加分区数，提高并行能力</p>
<p>3.给key添加随机后缀，使得key均匀的分布到不同的分区</p>
<p>4.消费者批量消费消息，提高消费效率</p>
<p>5.开启异步提交offset或自动提交offset</p>
</details>

11.Kafka吞吐量高的原理
<details>
<summary>答案</summary>
<p>1.顺序读写磁盘，充分利用了操作系统的预读机制，因此有着较高的读写速度</p>
<p>2.使用了零拷贝技术，通过sendfile方法，允许操作系统将数据从pagecache直接发送到socket缓冲区，然后拷贝到网卡，这样避免重复复制数据，大大提高了性能</p>
<p>3.采用了分区分段+索引的思想 将消息按主题分类，每个主题的数据是按照一个个分区存储在不同的broker上的，每个分区的数据又是分段存储的，kafka又为每个段建立了索引，提升了读取数据的性能和操作的并行度</p>
<p>4.kafka采用了批量读写，在向kafka写入数据时，将会按批次写入，减少延迟和网络开销</p>
<p>5.kafka采用了批量压缩技术，将同一个批次的消息一起压缩，支持多种压缩协议，减少了网络IO的消耗</p>
</details>
12.Kafka存储的原理
<details>
<summary>答案</summary>
<p>
1.kafka的消息是按主题分类的，每个主题的数据文件又是分区存储的，每个分区的数据又是分段存储的，kafka为每个段的数据建立了稀疏索引，当需要查找一个数据时，通过二分查找找到对应的段，然后通过稀疏索引，找到他在文件中的位置，稀疏索引是每隔4KB就添加一个索引。
2.Kafka还采用了pagecache，由操作系统负责写入磁盘，减少了磁盘IO的消耗
3.kafka还采用了零拷贝技术,使用sendfile+pagecache，直接将数据从pagecache发送到socket，然后拷贝到网卡，避免了重复复制数据，提高了性能
4.Kafka采用了顺序读写，有效的降低了寻址时间，提高了效率
</p>
</details>

13.kafka消费者采用的是推还是拉?为什么?
<details>
<summary>答案</summary>
<p>采用的是拉，因为如果采用推，会导致broker发送多少消息，消费者就要消费多少消息，可能会导致网络拥塞，消费者负载增加。而采用拉可以让消费者根据自己的消费能力控制拉去速度，但是可能拉取到空的消息，所以要控制拉取间隔</p>
</details>


14.kafka如何判断一个节点是否存活?
<details>
<summary>答案</summary>
<p>1.节点必须维护和Zookeeper的连接，Zookeeper通过心跳机制检查每个节点的连接</p>
<p>2.从节点要与主节点同步，不能落后主节点太多</p>
</details>


15.Kafka 与传统消息系统之间的三个关键区别
<details>
<summary>答案</summary>
<p>
1.kafka将日志持久化到磁盘，这些日志可以被重复读取
2.kafka是一个分布式系统，以集群的方式运行，保证分区容错和高可用
3.kafka支持实时的流式处理
</p>
</details>

16.Kafka怎么做到最多消费一次
<details>
<summary>答案</summary>
<p>1.在ack机制中，选择ack=0，这样可以保证不会重复收到消息</p>
<p>2.在提交offset的选项，选择手动提交同步提交，先提交offset，再处理数据</p>
<p>3.开启kafka幂等性，ack=all并且retries>1。可以避免重复接收消息</p>
</details>

17.Kafka可靠性如何保证?
<details>
<summary>答案</summary>
<p>
1.消息确认机制：生产者向对应的topic发送消息，通过消息确认机制来保证消息的可靠性，ack=0，表示生产者将消息发送出去就认为已经成功写入kafka，ack=1表示主副本收到消息就直接放回响应，不等从副本复制完数据。ack=-1或all表示等待所有主副本和从副本都收到消息才返回响应
2.分区副本机制：kafka通过分区副本机制来保证消息的可靠性，一个分区有一个主副本和0到多个从副本，能够保证即使一个broker宕机，也不会数据丢失，从副本会定期从主副本拉取数据
3.Leader选举机制：每个分区维护一个ISR列表，表示与leader同步的副本列表，如果一个从副本落后主副本太多，将会被移除ISR列表，落后的副本追上了主副本也会被加入ISR列表，主副本宕机后，会从ISR列表中选举新leader,能够保证消息的可靠性
</p>
</details>

18.Kafka能否脱离zookeeper?脱离zookeeper如何管理节点
<details>
<summary>答案</summary>
<p>
可以，最新的Kafka已经使用使用KRaft来管理Kafka集群的元数据
</p>
</details>

19.kafka偏移量维护在哪里
<details>
<summary>答案</summary>
<p>kafka的偏移量存储在kafka集群内的consumer_offset中，消费者可以自动提交offset,也可以手动提交offset</p>
</details>

20.kafka如果有台机器挂掉会发生什么
<details>
<summary>答案</summary>
<p>一开始，节点启动时，都会和zk维护一个连接，然后节点挂掉后，zk会通过心跳机制发现该节点离线，然后会将该节点的信息从zk中移除掉，并会重新分配分区和副本，并且将离线的副本移除ISR列表，然后重新进行leader选举</p>
</details>

21.kafka中生产者发送消息的具体流程？
<details>
<summary>答案</summary>
<p>首先主线程会先创建producer record，其中包含主题，分区，键,值和时间戳。</p>
<p>然后会将其序列化，然后如果没有指定分区号则会通过分区器选择一个分区。</p>
<p>然后将其写入Producer Accumulator。这个是用于缓冲消息的双端队列,数据积累到一定大小或超过一定时间就会被发送</p>
<p>sender线程会从producer accumulator中拉取数据，构造请求发送到broker</p>

</details>
22.kafka中消费者消费消息的具体流程?
<details>
<summary>答案</summary>
<p>消费者首先会找到自己的组协调器，然后向组协调器发起加入消费组的请求，加入消费组后，消费者leader会为其指定分区分配方案，并同步给所有消费者。消费者根据自己负责的分区，进行拉取数据，处理数据并提交offset</p>
</details>

23.Kafka LEO和HW在没有epoch的情况下，数据不一致和数据丢失的场景
<details>
<summary>答案</summary>
<p>
1.数据不一致: 当follower的hw落后于leader的hw时,如果此时follower和leader都宕机,follower先重启，成为了新的leader，然后收到了新的消息，更新leo和hw，然后旧的leader重启，成为了follower,旧的leader向新的leader拉取数据，发现新leader的hw和自己相同，故不发生改变，但是此时数据已经产生了不一致。
2.数据丢失: 当follower的hw落后于leader的hw时，如果此时follower宕机，重启后follower根据hw将日志进行截断，然后向leader拉取数据，但此时leader宕机,follower成为leader，然后旧的leader重启后，成为了follower，旧leader向新leader拉取数据，然后发现新leader的hw更小，故将自己的hw更新，并进行截断。从而导致数据丢失
</p>
</details>

24.kafka生产者发送消息的方式
<details>
<summary>答案</summary>
<p>发送并忘记，同步发送，异步发送+回调函数</p>
</details>


25.kafka于其他消息队列相比之下的优点?
<details>
<summary>答案</summary>
<p>高吞吐量，低延迟。持久性</p>
</details>