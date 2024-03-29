### HTTP状态码

200：请求成功
301：表示永久重定向，原来的资源已经不存在了，要使用新的URL
400：语法错误，无法被服务器解析
403：服务器已经接收请求，但是拒绝执行
404：表示请求的资源不存在
500：服务器内部错误，无法处理请求
503：表示服务器繁忙，暂时无法响应

### HTTP请求方法

GET：获取资源数据
POST：提交资源数据
PUT：更新资源数据
DELETE：删除资源数据
HEAD：读取资源的元数据
OPTIONS：读取资源所支持的所有请求方法
TRACE：回显服务器收到的请求
CONNECT：保留，为将来使用

### HTTP与HTTPS的区别

我认为HTTP就是制定了客户端与服务端之间通讯的规则，通过约定好的规则来交换信息；而HTTPS并不是一个新的协议，只是在HTTP的基础上多加了一层Secure，所以HTTPS = HTTP + TLS/SSL组合而成的. 两者的区别：

1. 单从写法上来看，一个http://，另一个是https://
2. HTTP是一个不安全的协议，在传输过程中可能会遭受窃取、数据伪造、监听，而HTTPS通过密钥交换算法、签名算法、摘要算法来达到安全.
3. HTTP的默认端口是80，HTTPS的默认端口是443

### HTTP GET和POST区别

1. 通常在以GET方式发送请求时是在URL后面加上参数的形式，较为不安全，而POST方式是将参数放在请求体中，对用户来说是不可见的.
2. GET请求的URL有长度限制，而POST把参数放在请求体中，没有长度限制
3. GET请求在浏览器中反复的回退/前进操作是无害的，而POST操作会再次提交表单
4. GET在请求过程中会产生一个TCP数据包，POST在发送过程中会产生两个TCP数据包. 对于GET方式的请求，浏览器会把请求头与请求体一并发送出去，而对于POST来说，浏览器先发送请求头，服务器响应100 continue，浏览器在发送请求体（有人认为GET和POST其实都只发送了一次，取决于浏览器）.

### TCP和UDP的区别

- 概念
1. UDP能够容忍数据包丢失、具有低延迟、能够发送大量的数据包、不需要所谓的握手操作，从而加快了通信速度.
2. TCP确保连接的建立和数据包的发送、支持错误重传机制、能够在网络拥堵的情况下延迟发送、能识别有害的数据包.

- 区别
1. TCP在发送数据前必须先建立连接，UDP无需建立连接就可以直接发送大量数据
2. TCP会按照特定顺序重新排列数据包，UDP数据包没有固定顺序，所有数据包都是相互独立的
3. TCP传输速度慢，UDP传输速度快
4. TCP的头部有20个字节，UDP的头部只需要8个字节
5. TCP需要三次握手来建立连接，UDP没有跟踪连接，发出去了就是发出去了，才不管有没有收到
6. TCP会进行错误校验，并能够进行错误恢复，UDP也会错误检查，但是会丢弃错误的数据包
7. TCP有发送确认，UDP没有发送确认
8. TCP会使用握手协议，比如SYN、SYN-ACK、ACK，UDP无握手协议
9. TCP是可靠的，确保数据发送到目的地，UDP不能保证数据送达

### TCP三次握手四次挥手

三次握手：通常所说的三次握手是指建立连接，在建立连接时，客户端会发送一个SYN信号，这个信号带有一个随机数X，服务端收到信号后，打开客户端连接，发送SYN-ACK信号，此时带有两个序列号，一个是作为ACK的序列号，此值等于X+1，另外一个序列号是发送数据包的随机数Y，此时服务端处于SYN_RECV状态，客户端收到信号后，最后发送一个ACK信号，带有序列号Y+1表示确认接收，此时客户端处于ESTABLISHED状态.

四次挥手：指的是释放连接的过程，客户端要断开连接时会发送一个FIN信号到服务端，此时客户端处理FIN_WAIT_1状态，服务端收到信号后会立刻返回ACK，表示我知道了，客户端收到ACK信号后，就进入到了FIN_WAIT_2状态，然后等待来自服务端的FIN信号，一段时间后，服务端发送FIN信号给客户端，告诉它可以关闭了，客户端收到FIN信号后，由FIN_WAIT_2变成了TIME_WAIT状态，为了防止信息丢失，此时客户端可以发送ACK到服务端，一段时间后，客户端上的所有资源都会被释放.

### HTTP1.0/1.1/2.0的区别

1. HTTP1.0中使用了if-modified-since和expires来作为缓存失效的标准，不支持断点续传，认为每台计算机只能绑定一个IP，所以没有主机域hostname
2. HTTP1.1默认使用长连接，通过keep-alive设置，新增了E-tag、if-unmodifier-since、if-match来控制缓存失效，支持断点续传，通过请求头中的Rang来实现
3. 使用HPACK算法来进行头部压缩（客户端和服务端同时维护一张头信息表来消除重复的部分）、使用二级制格式来传输、支持多路复用、服务器推送、数据流的优先级，即优先级高的请求服务器会优先响应

### HTTP2.0的缺陷

主要问题在于多个HTTP请求复用一个TCP连接，而下层的TCP协议是不知道有多少个HTTP请求的，当某一个请求发生丢包发生重传的时候，所有的HTTP请求都必须等待这个包被重传，也就是说所有的HTTP都会被阻塞住，由于是传输层的问题，所以HTTP3将TCP改成了UDP，可是我们知道UDP是不可靠协议，接着又提供了QUIC协议，该协议实现了类似TCP的可靠性传输，哪个流的数据包丢失了就阻塞哪个流，其他流不会受到影响，但QUIC是新协议，很多网络设备压根就不认识，没办法进行正确的处理

### HTTP的优缺点
优点：
1. 简单，只包含header + body信息，其中header采用的是key-value的形式，简单易学
2. 易于扩展和灵活，HTTP的请求方法、头部字段、状态码都可以自定义，加上在处于OSI模型中的应用层，其下层可以随意变化
3. 跨平台、应用广泛，浏览器、各种APP都在使用
缺点：
1. 无状态，对于一些具有关联性的操作来说比较难处理，比如说每次访问都需要验证用户信息
2. 明文传输，很容易被窃取信息
3. 不安全，因为使用的是明文，这些信息有可能会被窃听、篡改、伪装

### 地址栏输入URL发生了什么

浏览器中输入域名，去查找域名是否被本地DNS缓存，如果找到就直接返回，如果找不到就向网络中发起一个DNS查询，查询到指定的IP后就开始建立TCP连接，发送请求，服务端响应请求

### 分析TCP三次握手失败

1. 当客户端发起的TCP第一次握手SYN包，在重传超时时间内没有收到服务端的ACK，就会重传SYN数据包，每次重传超时的RTO（即从数据发送时刻算起，超时这个时间就会重传；RTT：数据发送时刻到接收到确认的时刻的差值）时翻倍上涨的，直到SYN包的重传次数达到tcp_syn_retries（可以查看值：cat /proc/sys/net/ipv4/tcp_syn_retries，默认值是5）值后，客户端便不会再发送SYN包
2. 当客户端发起的TCP第一次握手SYN包，服务端接收到SYN包后会发送SYN_ACK包，此时如果客户端未能在指定时间内接收到SYN_ACK包后，服务端会重复发送SYN_ACK包，发送的次数是tcp_synack_retries，同时客户端也会重复发送SYN包，它认为服务端没有收到SYN包，重试的次数仍然是tcp_syn_retries（默认值是5）
3. 当客户端发起的TCP第一次握手SYN包，服务端正常接收并给客户端发送SYN_ACK，此时处于SYN_RECV状态，客户端接收到后给服务端发送ACK，如果服务端未能接收到ACK，则会一直尝试发送SYN_ACK，尝试的次数由tcp_synack_retries值控制，超时改值后服务端的连接就关闭了，而对于客户端来说，如果再次期间没有发送数据包，那么它会有一个保活机制，也就是说在指定时间（默认是2个小时）后启动保活机制，每经过75秒后就检测服务端是否还活着，一共检测9次，超时这个次数后客户端就会断开连接；而如果客户端在这个期间发送了数据包，而一直没有收到服务端的响应，那么也就会进行重传，重传次数由tcp_retries2值控制（默认值是15），超过该值后客户端就会断开连接.


### redis数据结构

string（字符串）、hash（字典）、list（列表）、set（集合）、zset（有序列表）
1. string在redis内部存储就是一个字符串
2. hash内部存储value为一个HashMap
3. list内部是一个双向链表
4. set内部实现是一个value为null的HashMap
5. zset内部使用HashMap和跳跃表来保证数据的存储和有序

### redis和memcached的比较

1. redis支持多种数据结构，memcached支持简单的key-value数据结构
2. redis支持持久化，memcached不支持
3. 内存管理机制不同
4. 集群管理的不同，redis采用master-slave方式，memcached采用一致性hash

### 持久化知识点

1. RDB使用全量持久化，AOF做增量持久化（记录操作日志）
2. 触发RDB快照的时机：
    - 执行save或bgsave命令，其中save会阻塞redis服务器进程，在创建RDB文件之前是不能执行其他命令，而bgsave命令会fork（fork过程中会阻塞主进程）一个子进程，该子进程会创建RDB文件，而服务器可以继续处理其他命令
    - 配置文件设置save seconds changes规则，自动间隔性执行bgsave命令，表示在seconds秒内，至少有changes次变化，就会触发bgsave命令
3. fork一个子进程，子进程会把数据集先写入临时文件，写入成功之后，再替换之前的RDB文件，用二进制压缩存储，这样可以保证RDB文件始终存储的是完整的持久化内容
4. AOF持久化会把执行写命令以追加的方式记录到AOF文件中，默认情况下没有开启
5. 开启AOF持久化功能后，服务器每执行一条写命令，都会把该命令以追加的方式记录到aof_buf缓冲区，并不是直接写入文件，避免每次都写入磁盘减少IO次数
6. 对于何时把aof_buf缓冲区的文件写入到AOF文件中，有三种策略
    - appendfsync always：每个写命令都同步写入磁盘
    - appendfsync everysec：每秒同步一次，由一个线程专门执行（默认策略）
    - appendfsync no：由操作系统来决定什么时候执行同步
7. AOF重写：随着时间的推移，AOF文件会越来越大，如果使用这些文件来还原那么所需要的时间也会很长，而且通常会有一些冗余的命令（重复设置、被删除的数据、过期数据），所以AOF文件是可以有压缩的空间的，这就是AOF重写的目的，不过它并不是对现有的AOF文件进行操作，而是通过读取当前数据库的状态来实现的，分为手动触发重写或自动触发重写
    - 手动触发：执行bgrewriteaof，机制跟bgsave类似
    - 自动触发：根据auto-aof-rewrite-percentage和auto-aof-rewrite-min-size 64mb配置来自动执行bgrewriteaof命令，例如：
        # 表示当AOF文件的体积大于64MB，且AOF文件的体积比上一次重写后的体积大了一倍（100%）时，会执行bgrewriteaof命令
        auto-aof-rewrite-percentage 100
        auto-aof-rewrite-min-size 64mb
    
8. 重写内部原理：
    - 重写会造成大量的写操作，所以服务器会fork一个子进程来创建一个新的AOF文件
    - 在重写期间，服务器会继续处理其他命令，如果有写的命令，仍然追加到aof_buf缓冲区，同时还会追加到aof_rewrite_buf重写缓冲区，这样子可以保证原有的AOF能够继续使用
    - 当子进程完成重写之后，会给父进程一个信号，然后父进程会把aof_rewrite_buf缓冲区的内容写入到新的AOF文件中
    - 最后使用新的AOF文件替换掉旧的AOF文件，以后有新的写命令都会追加到新的AOF文件中

### 为什么说单机redis的内存越大有可能会阻塞主进程

父进程通过fork操作可以创建子进程；子进程创建后，父子进程共享代码段，不共享进程的数据空间，但是子进程会获得父进程的数据空间的副本。在操作系统fork的实际实现中，基本都采用了写时复制技术，即在父/子进程试图修改数据空间之前，父子进程实际上共享数据空间；但是当父/子进程的任何一个试图修改数据空间时，操作系统会为修改的那一部分(内存的一页)制作一个副本。虽然fork时，子进程不会复制父进程的数据空间，但是会复制内存页表（页表相当于内存的索引、目录）；父进程的数据空间越大，内存页表越大，fork时复制耗时也会越多。在Redis中，无论是RDB持久化的bgsave，还是AOF重写的bgrewriteaof，都需要fork出子进程来进行操作。如果Redis内存过大，会导致fork操作时复制内存页表耗时过多；而Redis主进程在进行fork时，是完全阻塞的，也就意味着无法响应客户端的请求，会造成请求延迟过大。

### RDB和AOF的优缺点

RDB优点：
1. RDB快照是经过压缩过的文件，保存着某个时间点的数据集，适合做数据的备份
2. 最大化redis的性能，服务器只需要fork一个子进程来完成RDB文件的创建和写入，并不需要父进程执行IO操作
3. 与AOF相比，恢复大数据集的时候更快
RDB缺点：
1. RDB是定时持久化，可能会丢失几分钟的数据
2. 当数据集较大时，fork的子进程要完成快照会比较耗CPU

AOF优点：
1. 数据更完整，秒级数据丢失
2. AOF文件是一个只进行追加的日志文件，且内容是可读的，适合误删紧急恢复
AOF缺点：
1. 相同的数据集，AOF文件的体积要比RDB文件的体积大，数据恢复也比较慢

### 一致性Hash算法的理解

集群中要确定数据应该放在哪一个节点上，较为简单的算法是哈希取余，即计算key的哈希值，然后对节点数量取余，从而决定数据应该落在哪一个节点上，该算法最大的问题就是当新增或删除节点的时候，系统中的数据要重新计算数据应该落在哪一个节点上，引发了数据大规模迁移；还有另外一种算法是一致性哈希，其实现原理是将整个哈希值空间组织成一个圆环，范围是0-2^32-1，将节点映射到该圆环上，对于每个数据来说，也都会通过hash算法映射到圆环上，确定了具体位置后就沿着圆环顺时针的方向行走，找到的第一个节点就是表明要将数据放在此位置上，如果也发生节点的增删，只会影响单个节点上的数据，对于节点较少的情况来说，会造成数据负载的不平衡（数据倾斜）；而后又引入了虚拟节点的概念，称之为槽，每个节点包含一定数量的槽，每个槽包含一定范围内的哈希值，在redis集群中，槽的数量是16384，计算hash%16384得出属于哪个槽，在根据槽与节点之间的关系得出要落在哪个节点上

### 主从复制全量复制工作机制

1. 主节点可以读写，从节点只能读，不能写
2. 主节点可以有多个从节点，一个从节点只能有一个主节点
3. 当主节点读写数据变化时，会直接同步到从节点

当slave启动后，主动向master发送SYNC命令，master接收到SYNC命令后在后台保存快照（RDB持久化）和缓存保存快照这段时间的命令，然后将保存的快照文件和缓存的命令发送给slave，slave接收到后就加载这些文件和命令，如果slave开启了AOF，则在同步后还会执行bgrewriteaof命令，至此以后，master每次接收到的写命令都会同步发送给slave，保证主从数据一致性

### 主从部分复制工作机制

1. 复制积压缓冲区，用于存储slave断开后的这段时间要同步的数据
2. 复制偏移量，通过移动偏移量来决定要将哪些数据传送到slave
3. slave将自身的偏移量发送给master节点后，master根据偏移量和缓冲区大小决定是否能执行部分复制
4. 如果偏移量之后的数据仍然都在缓冲区里，则执行部分复制，如果不在（缓冲区是有大小限制的，随着不断的累计之前的数据已经被清除，repl-backlog-size）则执行全量复制
5. 还有一种情况，每个redis节点在启动时都会自动生成一个随机ID（runid），每次启动都不一样
6. 主从节点初次复制时，master将自己的runid发送给slave，slave会将其保存起来，当断线重连时，slave会将runid发送给master，master根据runid判断是否进行部分复制
7. 如果slave保存的runid和master的runid相同，说明主从节点之前同步过，master会继续尝试使用部分复制，决定能够部分复制的因素还是要看偏移量和缓冲区
8. 如果slave保存的runid和master的runid不同，说明slave在断线前同步的master并不是当前的主节点，只能进行全量复制

### master-slave连接超时的判断

1. 判断master-slave超时的参数是repl-timeout，对于master与slave都有效
2. 对于master来说，每秒调用1次定时函数replicationCron，在其中判断当前时间距离上次收到各个slave的REPLICONF ACK的时间，是否超过了repl-timeout值，如果超过了则释放对应的连接
3. 对于slave来说，也是在定时函数replicationCron进行判断
    - 如果当前处于连接建立阶段，且距离上次收到master的信息的时间超过了repl-timeout，则释放与master的连接
    - 如果处于数据同步阶段，且收到master的RDB文件的时间超时，则停止数据同步，释放连接
    - 如果处于命令传播阶段，且距离上次收到master的ping命令或者数据的时间已超过repl-timeout，则释放与master的连接

### 哨兵的几个概念

1. 定时任务，每个哨兵节点维护了3个定时任务
    - 通过向master发送info命令获取最新的master-slave结构
    - 通过发布订阅功能获取其他哨兵节点的信息
    - 通过向其他master节点发送ping命令进行心跳检测，判断是否下线

2. 主观下线，在心跳检测的定时任务中，如果其他master节点超过一定时间没有回复，哨兵节点就会将其进行主观下线，顾名思义，主观下线就是一个哨兵"主观"地判断下线
3. 客观下线，哨兵节点在对master节点进行主观下线后，会通过命令询问其他哨兵节点该master节点的状态，如果判断master节点下线的哨兵数量达到一定数值，则对该master节点进行客观下线
4. 选举领导者哨兵，当master节点被判断为客观下线后，各个哨兵节点会协商选举出一个领导者哨兵，并由该领导者节点对其进行故障转移操作
5. 监视该master节点的所有哨兵都有可能成为领导者，选举使用的算法是Raft算法，基本思路是先到先得，即在一轮选举中，哨兵A向B（master节点?）发送成为领导者的申请，如果B没有同意过其他哨兵，则会同意A成为领导者，一般来说，哨兵选择的过程很快，谁先完成客观下线，一般就能成为领导者
6. 故障转移，选举出哨兵节点后，开始进行故障转移操作，有3个步骤：
    - 在slave节点中选择新的master节点，原则是，首先过滤掉不健康的slave节点（何为不健康），然后选择优先级最高的slave节点（slave-priority指定），如果优先级无法区分，则选择复制偏移量最大的slave节点（说明数据比其他slave节点全），若是仍无法区分，则选择runid最小的slave节点
    - 通过slave no one命令，让选出来的slave节点成为master节点，并通过slaveof命令让其他节点成为其从节点
    - 将已经下线的master节点设置为新的主节点的从节点，当旧master重新上线后，它会成为新的主节点的从节点

### 主从、哨兵的优缺点

主从优点
1. 实现高可用，读操作的负载均衡，数据的多机备份
主从缺点
1. 故障恢复无法自动化、写操作无法负载均衡，存储能力受到单机的限制

哨兵优点
1. 在主从复制的基础上实现了自动化故障恢复
哨兵缺点
1. 写操作仍然无法负载均衡，存储能力受到单机的限制
2. 哨兵无法对将从节点的故障进行转移

### 集群下的故障转移

1. 通过定时任务发送PING消息检测其他节点状态；节点下线分为主观下线和客观下线；客观下线后选取从节点进行故障转移。在故障转移阶段，需要由主节点投票选出哪个从节点成为新的主节点；从节点选举胜出需要的票数为N/2+1；其中N为主节点数量(包括故障主节点)，但故障主节点实际上不能投票。因此为了能够在故障发生时顺利选出从节点，集群中至少需要3个主节点(且部署在不同的物理机上)。

### Hash Tag

Hash Tag原理：当一个key包含{}的时候，不对整个key做hash，而仅对{}包括的字符串做hash，可以将不同的key分配在同一个槽里。

### 缓存穿透

1. 缓存穿透指的是缓存和数据库都查不到的数据。正常使用缓存的流程是，先查询缓存，缓存找不到就查数据库，如果数据库查到了就放进缓存中，如果查不到就不放入缓存中，这就导致了在面对大量的请求不存在数据时有可能会导致数据库崩掉。解决方案是在数据库查不到数据的时候会将key-null数据放入到缓存中，并设置过期时间

### 缓存击穿

1. 缓存击穿指的是缓存中不存在，但数据库中存在（一般是缓存时间到期），这时由于并发用户多，同时缓存的数据已经过期了，结果都跑到数据库去查询数据了，引起数据库的压力。解决方案是设置热点数据永远不过期；接口做限流或者降级或者熔断；

### 缓存雪崩

1.  缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。解决方案是缓存数据的过期时间设置成随机，防止同一时间大量数据过期现象发生；设置缓存数据永不过期

### 服务降级

当服务器压力剧增时为了保证核心功能的可用性，而选择性降低一些功能的可用性，或者直接关闭该功能

### 服务熔断

当某个服务不可用时，为了避免整个系统出现雪崩，暂时停止对该服务的调用，倾向于对外部服务的调用

### 过期策略

1. 定时删除，默认100ms就随机抽取一些设置了过期时间的key，检查是否过期，过期了删除掉
2. 惰性删除，当查询一个过期的key时会删除掉
3. 内存淘汰机制，6种策略

### redis为什么快

1. 它是单线程，避免了上下文切换，也不用去考虑锁的问题
2. 完全基于内存
3. 数据结构简单，对数据的操作也简单
4. 多路IO复用模型可以处理多个请求，利用select、poll、epoll同时监控多个流的IO事件

### redis事务

redis事务本质上是一组命令的集合，可以一次执行多条命令，这些命令会按顺序执行，即使有其中一条命令报错了，之前的命令也不会发生回滚，之后的命令会继续执行

### explain

通常是type、key、ref、rows、extra属性比较重要

### 索引规则

1. 全值匹配我最爱，比如说有一个复合索引，那么在查询语句中按照顺序书写，并且不会跳过，如
    - select * from tb_name where name = '' and age = 1 and pos = 3
    - select * from tb_name where name = '' and pos = 3
    第一个sql语句要好于第二个

2. 最佳左前缀法则
3. 不要在索引列上做任何操作，比如函数、计算、类型转换，会导致索引失效
4. 查询条件中某个字段若采用范围条件来表示，那么后面的列都将不会使用索引
5. 尽量使用覆盖索引，减少使用select * 
6. mysql在使用!、<>时无法使用索引会导致全表扫描
7. is null、is not null也无法使用索引
8. like以通配符开头（'%abc' ），索引也会失效
9. 字符串不加单引号也会导致索引失效
10. 少用or，用它来连接时索引也会失效


### mysql性能低分析流程

1. 开启慢查询日志，设置阈值，比如超过5分钟的就是慢SQL，并将其抓取出来
2. explain + 慢SQL分析
3. show profile
4. 数据库的参数调优

### ACID
1. 原子性、一致性、持久性、隔离性
2. undo log保证了数据库的原子性和隔离性，redo log保证了持久性
3. 当事务对数据库进行修改时，innodb会生成对应的undo log，当事务执行失败或调用rollback，导致事务需要回滚，便可以利用
undo log中的信息将数据回滚到修改之前的样子，undo log是逻辑日志，它记录的是sql执行相关的信息，当发生回滚时，innodb会根据
undo log的内容与之前相反的工作，对于每个insert，回滚时会执行delete，对于delete，回滚时会执行insert，对于update，回滚时会
执行一个相反的update，把数据改回去
4. 数据是存放在磁盘上，如果每次读写都需要硬盘IO，那效率就很低，所以innodb提供了缓存（buffer pool），缓存中包含了磁盘数据页的映射，作为访问数据库的缓冲；当从数据库读取数据时，首先会从buffer pool中读取，如果buffer pool中没有，则从磁盘上读取后放入到buffer pool中；当数据库写入数据时，会首先写入buffer pool，buffer pool中修改的数据会定期刷新到磁盘中（这一过程成为刷脏），但是会有一个问题，如果mysql挂掉了，而此时buffer pool中修改的数据还没有刷新到磁盘中，就会导致数据的丢失，也就没办法保证持久性，于是redo log来了，当数据修改时，除了修改buffer pool外，还要把这些操作记录在redo log中，当事务提交时，会调用fsync接口对redo log进行刷盘，也就是将更新的redo log写入到磁盘中，如果mysql挂掉了，那么可以通过redo log来恢复，redo log采用的是预写入日志，所有修改先写入日志，再更新到buffer pool中

### 锁机制的基本原理

1. 事务在修改数据之前，需要先获取相应的锁，获得锁之后事务便可以修改数据，在事务操作期间，这部分数据是锁定的，其他事务如果需要修改数据，需要等待当前事务提交或者回顾后释放锁


### 脏读、不可重复读、幻读

1. 脏读，事务A可以读到其他事务未提交的数据
2. 不可重复读，事务A先后两次读取同一条数据，两次的结果不一样，脏读读到的是其他事务未提交的数据，不可重复读是其他事务已经提交的数据
3. 幻读，事务A先后两次查询数据，两次的结果的条数不同，不可重复读指的是数据变了，幻读是数据的行数变了

### 事务的隔离级别

隔离级别          脏读           不可重复读         幻读
读未提交          可能             可能            可能
读已提交          不可能           可能            可能
重复读            不可能           不可能          可能
串行化            不可能           不可能          不可能

### MVCC 
    
MVCC，多版本的并发控制，同一个时刻，不同的事务读取到的数据可能是不同的。innodb中每行数据都有两个隐藏列，一个是data_trx_id（更新这条行记录的事务ID），一个是data_roll_ptr（指向undo log的指针，每条undo log也会指向更早版本的undo log，从而形成一条版本链）。在RU级别下，直接读取版本的最新记录就可以了，对于串行化来说，则是通过加锁互斥来访问数据，不需要MVCC的帮助，因此MVCC云心在RC和RR这两个级别上，现在最重要的问题是版本链中哪些版本对当前事务可见，为了解决这个问题，设计了ReadView。

- 在RR级别下，每个事务执行第一个select语句时会将当前系统中的所有的活跃事务（开启事务但还未提交修改）拷贝到一个列表来生成ReadView，后续所有的select都是复用这个ReadView

- 在RC级别下，每个事务每次执行select语句时，都会重新将当前系统中的所有的活跃事务拷贝到一个列表来生成ReadView

ReadView中是当前活跃的事务ID列表，称之为m_ids，其中最小值为up_limit_id，最大值为low_limit_id，事务ID是事务开启时innodb分配的，其大小决定了事务开启的先后顺序，因为可以根据事务ID的大小关系来决定版本记录的可见性，判断流程如下：

- 如果访问的数据行的trx_id小于m_ids中的最小值up_limit_id，说明生成该版本（数据行）的事务在ReadView生成前就已经提交了，所以该版本可以被当前事务访问

- 如果访问的数据行的trx_id大于m_ids中的最大值low_limit_id，说明生成该版本（数据行）的事务在ReadView生成后才生成，所以该版本不可以被当前事务访问，需要根据undo log找到前一个版本，然后根据前一个版本的data_trx_id重新判断可见性

- 如果访问的数据行的trx_id处于m_ids的最大值与最小值之间，那就需要判断一下trx_id的值是不是在m_ids列表中，如果在，说明创建ReadView时生成该版本（数据行）的事务还是活跃的，因此该版本不可以被访问，需要查找undo log查找上一个版本，然后根据上一个版本的data_trx_id从头计算一次可见性；如果不在，说明创建ReadView时生成该版本（数据行）的事务已经被提交了，该版本可以被访问

### 聚簇索引非聚簇索引、覆盖索引、联合索引

- 聚簇索引也叫聚集索引，是一种数据存储方式，聚簇索引的叶子节点保存了一行记录的所有列信息，也就是说，聚簇索引的叶子节点中，包含了一个完成的记录行。
- 非聚簇索引也叫辅助索引、普通索引，它的叶子节点只包含了一个主键值，通过非聚簇索引查找记录要先找到主键，然后通过主键再到聚簇索引中找到对应的记录，这个过程被称为回表
- 如果辅助索引能够得到查询的信息，就不需要在进行回表了，这个就是覆盖索引
- 同时对多个列创建的索引称为联合索引，当对多列创建索引后，并不是只要包含了创建索引的列就能使用索引，索引的使用要遵循最左前缀匹配原则。假设对（A,B,C）创建索引，那么只有以下场景使用索引：
    - 对列（A,B,C）/（A,C）或者（A,B）进行查询时会匹配索引，对（C,A）或者（B,C）来说不能使用索引
    - 通配符只能使用`like 'value%'`形式，不能使用`like '%value'`，否则会造成全表扫描
    - 索引列不能做任何操作，包括运算、函数计算、类型转换
    - 索引列不能包含范围值查询，如>/</between等会导致后面的列无法匹配索引
    - 索引列不能使用!=/<>/is null/is not null
    - 对于字符串记得加单引号
    - 少用or
- 索引下推，在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数，比如有联合索引age、name，首先根据age、name的值过滤出符合条件的主键ID，然后采取聚簇索引中找对应的记录
- 唯一索引，不允许有相同索引值的索引，主键索引就是唯一索引
- 哈希索引，对于每一行数据，存储引擎都会对所有的索引列计算出一个哈希码，哈希索引将所有行算出的哈希码存储在索引中，并为每一个哈希码维护指向具体某一行的指针，哈希索引的时间复杂度是O（1）

### 为什么使用B+树

B树是一颗多路平衡查找树，也就是说每个非叶子节点可以有多个子树，因此当总节点数量相同时，B树的高度远小于AVL树和红黑树。对于B+树来说，它只有叶子节点是存储真实的数据，而非叶子节点存储的是键

- B+树的非叶子节点只包含键，而不包含真实数据，因此每个节点可以存储的记录会比B树多，所需要的高度也会更低，访问时所需要的IO次数就更少
- 局部性原理，当一个数据被使用时，其附近的数据有较大概率在短时间内被使用，B树将键相近的数据存储在同一个节点，当访问其中数据时，数据库会将整个节点读到缓存中，而当它临近的数据被访问时就可以直接在缓存中读取，无需进行磁盘IO
- 更少的IO次数，由于每个节点存储的记录数更多，所以对局部性原理的利用更好，缓存命中率更高
- 更适合范围查询，B树中进行范围查询时，首先需要先找到要查找的下限，然后对B树进行中序遍历，直到找到要查找的上限，而B+树的范围查询只需要对链表进行遍历即可（叶子节点之间通过双向链表连接）
- 更稳定的查询效率，B树的查询时间复杂度在1到树高之间（分别对应记录在根节点和叶子节点），而B+树的查询时间复杂度则稳定为树高，因为所有数据都在叶子节点上

### B+树的缺陷

由于键会重复，因此会占用更多的空间

### 其他

在innodb存储引擎中，select操作的不可重复读通过MVCC解决，而update、delete的不可重复读通过record lock（记录锁）解决，insert的不可重复读问题通过next-key lock（record lock + cap lock）解决

### mysql主从复制流程

- 第一步，master在每次提交事务完成数据更新前，将改变记录到二进制日志中
- 第二步，slave启动一个IO线程来，读取master上bin log中的事件，并记录到slave自己的中继日志中
- 第三步，slave还会启动一个线程，该线程从中继日志中读取事件，并在备库中执行，从而实现备库数据的更新


### 主键和外键的区别

1. 一张表中主键是唯一的，不允许空值，外键可以有多个，可以重复，允许空值

### 外键要创建索引吗？

如果子表中的外键不创建索引，将导致两个问题：

1. 影响性能 -> 如果子表外键没有创建索引，那么当父表查询关联子表时，子表将进行全表扫描.
2. 影响并发 -> 如果子表外键没有创建索引，那么在子表进行DML操作时，将会锁住整个父表.
所以，我们应该尽量考虑在外键上面创建索引

### AQS/CountDownLatch/CyclicBarrier/Semaphore

- CountDownLatch：允许一个线程或多个线程等待特定操作的完成，这些操作时在其他的线程中进行，比如多个任务并发执行，直到所有任务都执行完毕，基于AQS
- CyclicBarrier：用于等待多个线程到达栅栏位置，基于ReentrantLock
- Semaphore：信号量，用于控制同时访问特定资源的线程数量，基于AQS
- AQS：是同步器的框架、基石，内部通过一个同步资源的状态位和一个等待队列来实现，也就是说同步资源的管理它已经帮你做好了（什么情况下入队列，什么情况下唤醒等待中的线程，也就是说出队列），如果要自定义同步器的话，只需要定义获取资源和释放资源的逻辑即可，内部还提供了获取资源的方式，独占（ReentrantLock）和共享（Semaphore/CountDownLatch）
- CountDownLatch的计数器只能使用一次，CyclicBarrier的计数器可以被重置，可以使用多次，所以CyclicBarrier能够处理更为复杂的场景；CyclicBarrier还提供了一些其他有用的方法，比如可以获取阻塞的线程数量


### spring的生命周期

1. 创建BeanFactory对象，首先会先验证XML文件的正确性，接着就开始解析XML文件，并将解析后的结果封装成BeanDefinition对象放入到Map中
2. 注册特殊的Bean、BeanPostProcessor、BeanFactoryPostProcessor
3. 注册MessageSource、事件广播器
4. 实例化Bean、注入属性
5. 执行init-method、afterPropertiesSet方法

### Spring BeanFactory和FactoryBean的区别

1. BeanFactory是IOC容器最基本的形式，定义了IOC容器应当遵守的最基本规范，它有很多种实现，如DefaultListableBeanFactory、XMLBeanFactory、ApplicationContext，都是附加了某种功能的实现。
2. Spring通过反射机制实例化Bean，某些情况下实例化Bean过程比较复杂，如果按照传统的方式，则需要在bean中提供大量的配置信息，这时采用编码的方式可能会得到一个简单的方案，为此Spring提供了一个FactoryBean接口
用户可以通过实现该接口来定制化实例化Bean的逻辑。

### 注意项

1. 通常情况下在实例化Bean时，其内部的属性也会一并实例化，但是某种情况下并不会实例化，就是其属性实现了`BeanNameAware`或`BeanFactoryAware`或`BeanClassLoaderAware`
2. 每个Bean都可以有多个别名，那么它是有一个Map，专门用来存储多个别名，比如一个对象有name = [a、b、c]别名，那么这个Map就会存储三条记录，分别是[a = car]、[b = car]、[c = car]，其中car指的是对象的id
   如果这时候在通过alias标签添加别名的话，比如name = a, alias = d，又会在Map中添加一个记录[d = a]，那么这个时候如果通过别名d来找对象的话，它是这么找的，如果通过d找到a，然后在根据a找到对应的car，然后在根据
   car查找，最后发现没有对象value，则证明car这个值正是bean对象的id，如果仍然可以找到对应的记录的话则证明仍然是对象的别名。spring在解析XML文件时会校验别名的循环引用问题，比如[a = d] [d = a]这样子一种情况


### spring mvc 基本原理

1. 用户发送请求至前端控制器DispatcherServlet
2. DispatcherServlet收到请求调用HandlerMapping处理器映射器
3. 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet
4. DispatcherServlet调用HandlerAdapter处理器适配器
5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)
6. Controller执行完成返回ModelAndView
7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器
9. ViewReslover解析后返回具体View
10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）
11. DispatcherServlet响应用户。


### mybatis基本原理

通过Resource加载配置文件，生成一个inputstream输入流，创建sqlSessionFactoryBuilder对象，通过该对象的builder方法构建一个sqlSessionFactory对象，由sqlSessionFactory对象生成sqlSession，通过statement id找到对应的statement，通过传入的参数进行一系列判断生成要执行的sql，通过jdbc执行sql，然后把结果封装成map、list等返回

### spring security认证基本原理

spring security定义了一组过滤器链条，每个过滤器都有要负责的功能，比如处理用户退出的逻辑、校验用户信息、匿名处理、校验权限，其中最重要的就是UsernamePasswordAuthenticationFilter，由该过滤器来负责进行客户端的认证，还有一个比较重要的过滤器是FilterSecurityInterceptor，它负责做权限校验，即判断是否有权限调用指定方法

### spring boot 自动配置

起先SpringApplication.run代码，内部会new SpringApplication对象，在构造函数中会调用getSpringFactoriesInstances方法，最终调用到SpringFactoriesLoader#loadFactoryNames，实际上最后加载的就是spring.factories文件中对应的配置

### 过滤器和拦截器的区别

1. 过滤器是通过函数回调，拦截器是通过Java反射机制
2. 过滤器依赖servlet容器，拦截器不依赖
3. 过滤器几乎对所有请求都起作用，拦截器只能对action起作用
4. 过滤器只会被调用一次，拦截器可以被多次调用
5. 过滤器只能在请求的前后使用，拦截器可以详细到每个方法
6. 过滤器不能使用IOC容器中的对象，拦截器则可以

### resultType和resultMap的区别

基本映射：（resultType）使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功.（数据库，实体，查询字段,,这些全部都得一一对应）
高级映射：（resultMap）如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系.（高级映射，字段名称可以不一致，通过映射来实现）