#  Zookeeper

## Zookeeper理论

### 概述

- Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的Apache项目。

- Zookeeper从设计模式角度来理解：是一个基于**观察者模式**设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应，从而实现集群中类似Master/Slave管理模式

- Zookeeper=文件系统+通知机制

**zookeeper工作原理**

![1617930535862](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/090857-369368.png)

### Zookeeper特点

![1617931160003](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/195024-175415.png)

1. Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。
2. 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。
3. 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
4. 更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。
5. 数据更新原子性，一次数据更新要么成功，要么失败。事务更新操作。
6. 实时性，在一定时间范围内，Client能读到最新数据。

### Zookeeper数据结构

ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称做一个ZNode。每一个ZNode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识。

![1617931860232](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/093101-203239.png)

###  Zookeeper应用场景

提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。

**统一命名服务**

![1617931938749](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/195319-273487.png)

**统一配置管理**

![1617931977123](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/195320-141958.png)

**统一集群管理**

![1617932156847](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/093600-488755.png)

**服务器动态上下线**

![1617932212392](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/093654-932481.png)

**负载均衡**

![1617932246030](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/093727-4775.png)

## Zookeeper安装配置

### 本地模式安装部署

1. 安装前准备

   1. 安装Jdk
   2. 拷贝Zookeeper安装包到Linux系统下
   3. 解压到指定目录

2. 修改配置文件

   将/opt/module/zookeeper-3.4.10/conf这个路径下的zoo_sample.cfg修改为zoo.cfg；

3. 打开zoo.cfg文件，修改dataDir路径：

~~~ java
//zookeeper数据的存储路径
dataDir=/opt/module/zookeeper-3.4.10/zkData
~~~

4. 在/opt/module/zookeeper-3.4.10/这个目录上创建zkData文件夹
5. 操作Zookeeper

~~~ java
// 1 启动Zookeeper
bin/zkServer.sh start
//查看状态
bin/zkServer.sh status
//启动客户端
bin/zkCli.sh 
//查看文件信息
ls /
//退出客户端
quit
//停止kafka
bin/zkServer.sh stop
~~~

### 配置参数解读

Zookeeper中的配置文件zoo.cfg中参数含义解读如下：

- tickTime =2000：通信心跳数，Zookeeper服务器与客户端心跳时间，单位毫秒，也就是两秒钟一个心跳。
  - Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每tickTime时间就会发送一个心跳，时间单位为毫秒。
  - 它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)
- initLimit=10：LF初始通信时限
  - 集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。超过20秒就认为leader和follower链接不上了。这个是初始链接时间限制。
- syncLimit=5：LF同步通信时限
  - 集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。初始通信之后的最大延迟时间
- dataDir：数据文件目录+数据持久化路径
  - 主要用于保存Zookeeper中的数据。
- clientPort=2181：客户端连接端口
  - 监听客户端连接的端口。客户端访问服务器时候服务器进程的端口号。

## Zookeeper的内部原理

### 选举机制

半数选举机制是在启动zookeeper节点时候选举leader使用的机制。

- 半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器。
- Zookeeper虽然在配置文件中并没有指定Master和Slave。但是，Zookeeper工作时，是有一个节点为Leader，其他则为Follower，Leader是通过内部的选举机制临时产生的。
- 以一个简单的例子来说明整个选举的过程。
  - 假设有五台服务器组成的Zookeeper集群，它们的id从1-5，同时它们都是最新启动的，也就是没有历史数据，在存放数据量这一点上，都是一样的。假设这些服务器依序启动

![1618029880095](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/124528-906860.png)

1. 服务器1启动，此时只有它一台服务器启动了，它发出去的报文没有任何响应，所以它的选举状态一直是LOOKING状态。因为此时选举server1的只有一台机器，没有达到一半数量
2. 服务器2启动，它与最开始启动的服务器1进行通信，互相交换自己的选举结果，由于两者都没有历史数据，所以id值较大的服务器2胜出，但是由于没有达到超过半数以上的服务器都同意选举它(这个例子中的半数以上是3)，所以服务器1、2还是继续保持LOOKING状态。每一个节点上来投票首先投自己，如果自己没有被选举到的话就投票给id值较大的节点。
3. 服务器3启动，根据前面的理论分析，服务器3成为服务器1、2、3中的老大，而与上面不同的是，此时有三台服务器选举了它，所以它成为了这次选举的Leader。
4. 服务器4启动，根据前面的分析，理论上服务器4应该是服务器1、2、3、4中最大的，但是由于前面已经有半数以上的服务器选举了服务器3，所以它只能接收当小弟的命了。
5. 服务器5启动，同4一样当小弟。

### 节点类型

![1618030088913](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1618030088913.png)

通过节点名称后面的编号，我们可以推断出节点的上线的顺序。

###  Stat结构体

- czxid-创建节点的事务zxid
  - 每次修改ZooKeeper状态都会收到一个zxid形式的时间戳，也就是ZooKeeper事务ID。事务ID是ZooKeeper中所有修改总的次序。每个修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1在zxid2前发生。
- ctime - znode被创建的毫秒数(从1970年开始)
- mzxid - znode最后更新的事务zxid
- mtime - znode最后修改的毫秒数(从1970年开始)
- pZxid-znode最后更新的子节点zxid
- cversion - znode子节点变化号，znode子节点修改次数
- dataversion - znode数据变化号
- aclVersion - znode访问控制列表的变化号
- ephemeralOwner- 如果是临时节点，这个是znode拥有者的session id。如果不是临时节点则是0。
- dataLength- znode的数据长度
- numChildren - znode子节点数量

### 监听器原理(重点)

![1618035217887](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/141350-517761.png)

### 写数据流程

![1618035512017](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/141832-157162.png)



## Zookeeper实战

### 分布式安装部署

1. 同步/opt/module/zookeeper-3.4.10目录内容到hadoop101、hadoop102
2. 配置服务器编号

~~~ java
//在/opt/module/zookeeper-3.4.10/这个目录下创建zkData
//在/opt/module/zookeeper-3.4.10/zkData目录下创建一个myid的文件
//添加myid文件，注意一定要在linux里面创建，在notepad++里面很可能乱码
//在文件中添加与server对应的编号：0
//并分别在hadoop101、hadoop102上修改myid文件中内容为1,2，注意，编号的值可以随便添加，只要保证唯一即可
~~~

3. 配置zoo.cfg文件‘

~~~ java
//重命名/opt/module/zookeeper-3.4.10/conf这个目录下的zoo_sample.cfg为zoo.cfg
//打开zoo.cfg文件,修改数据存储路径配置
dataDir=/opt/module/zookeeper-3.4.10/zkData
//增加如下配置
#######################cluster##########################
server.0=hadoop100:2888:3888
server.1=hadoop101:2888:3888
server.2=hadoop102:2888:3888
//同步zoo.cfg配置文件
~~~

- 配置参数解读：server.A=B:C:D。
  - **A**是一个数字，表示这个是第几号服务器；
  - 集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
  - **B**是这个服务器的ip地址；
  - **C**是这个服务器与集群中的Leader服务器交换信息的端口；
  - **D**是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

### 集群操作

1. 启动集群

~~~ java
//需要在每一台机器上启动集群
bin/zkServer.sh start
~~~

2. 查看节点状态

~~~ java
//在启动节点的时候，会使用半数机制选举出leader，其余的作为follower存在
bin/zkServer.sh status
~~~

### 客户端命令操作

| 命令基本语法       | 功能描述                                               |
| ------------------ | ------------------------------------------------------ |
| help               | 显示所有操作命令                                       |
| ls path [watch]    | 使用 ls 命令来查看当前znode中所包含的内容              |
| ls2 path   [watch] | 查看当前节点数据并能看到更新次数等数据                 |
| create             | 普通创建   -s  含有序列   -e  临时（重启或者超时消失） |
| get path   [watch] | 获得节点的值                                           |
| set                | 设置节点的具体值                                       |
| stat               | 查看节点状态                                           |
| delete             | 删除节点                                               |
| rmr                | 递归删除节点                                           |

**案例**

1. 进入客户端

~~~ java
bin/zkCli.sh
~~~

2. 显示所有命令

~~~ java
help
~~~

3. 查看当前znode中所包含的内容

~~~ java
 ls /
~~~

4. 查看当前节点详细数据

~~~ java
ls2 /
~~~

5. 分别创建2个普通节点

~~~ java
//创建一个节点必须存储数据，否则无法创建成功
create /sanguo jinlian

//查看节点
[zk: localhost:2181(CONNECTED) 3] ls /
[zookeeper, sanguo]

//创建多级目录
[zk: localhost:2181(CONNECTED) 4] create /sanguo/shuguo liubei
Created /sanguo/shuguo
[zk: localhost:2181(CONNECTED) 5] ls /sanguo
[shuguo]
~~~

6. 获得节点的值

~~~ java
[zk: localhost:2181(CONNECTED) 7] get /sanguo
jinlian
cZxid = 0x100000004
ctime = Sat Apr 10 13:23:54 CST 2021
mZxid = 0x100000004
mtime = Sat Apr 10 13:23:54 CST 2021
pZxid = 0x100000005
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1
~~~

7. 创建短暂节点

~~~ java
//-e参数表示创建短暂节点，如果此时客户端退出，那么wuguo这个数据就回消失
[zk: localhost:2181(CONNECTED) 8] create -e /sanguo/wuguo "sunquan"
Created /sanguo/wuguo
[zk: localhost:2181(CONNECTED) 9] ls /sanguo 
[wuguo, shuguo]
~~~

8. 创建带序号的节点

如果原来没有序号节点，序号从0开始依次递增。如果原节点下已有2个节点，则再排序时从2开始，以此类推。

~~~ java
//创建带序号的节点
[zk: localhost:2181(CONNECTED) 10] create -s /sanguo/weiguo "caocao"
Created /sanguo/weiguo0000000002
[zk: localhost:2181(CONNECTED) 11] ls /sanguo
[wuguo, weiguo0000000002, shuguo]
~~~

9. 修改节点数据值

~~~ java
[zk: localhost:2181(CONNECTED) 12] set /sanguo/wuguo "daqiao"
cZxid = 0x100000006
ctime = Sat Apr 10 13:28:19 CST 2021
mZxid = 0x100000008
mtime = Sat Apr 10 13:36:52 CST 2021
pZxid = 0x100000006
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x78ba3417480001
dataLength = 6
numChildren = 0
[zk: localhost:2181(CONNECTED) 13] get /sanguo/wuguo
daqiao //修改后的值
cZxid = 0x100000006
ctime = Sat Apr 10 13:28:19 CST 2021
mZxid = 0x100000008
mtime = Sat Apr 10 13:36:52 CST 2021
pZxid = 0x100000006
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x78ba3417480001
dataLength = 6
numChildren = 0
~~~

10. 节点的值变化监听

~~~ java
//在hadoop101主机上注册监听/shuguo节点数据变化
[zk: localhost:2181(CONNECTED) 15] get /sanguo/shuguo watch
liubei
cZxid = 0x100000005
ctime = Sat Apr 10 13:25:00 CST 2021
mZxid = 0x100000005
mtime = Sat Apr 10 13:25:00 CST 2021
pZxid = 0x100000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
//在hadoop102主机上修改/shuguo节点的数据
[zk: localhost:2181(CONNECTED) 1] set /sanguo/shuguo "zhangfei"        
cZxid = 0x100000005
ctime = Sat Apr 10 13:25:00 CST 2021
mZxid = 0x10000000b
mtime = Sat Apr 10 13:41:33 CST 2021
pZxid = 0x100000005
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 8
numChildren = 0
//观察hadoop101主机收到数据变化的监听
WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/sanguo/shuguo
~~~

> 注意：注册监听，只能够监听一次变化，多次变化，无法监听

11. 节点的子节点变化监听（路径变化）

~~~ java
//在hadoop101主机上注册监听/sanguo节点的子节点变化
[zk: localhost:2181(CONNECTED) 20] ls /sanguo watch                  
[wuguo, tang, weiguo0000000002, shuguo, xiaorui, han]
  
//在hadoop101主机/sanguo节点上创建子节点
[zk: localhost:2181(CONNECTED) 5] create /sanguo/qin "xiaoqiao"   
Created /sanguo/qin

//观察hadoop101主机收到子节点变化的监听
[zk: localhost:2181(CONNECTED) 1] 
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/sanguo
~~~

> 节点的值变化监听使用的是set,子节点变化使用的是ls

12. 删除节点

~~~ java
 delete /sanguo/tang
~~~

13. 递归删除节点

~~~ java
rmr /sanguo //删除sanguo下面的所有节点
~~~

14. 查看节点状态

~~~ java
[zk: localhost:2181(CONNECTED) 5] stat /sanguo
cZxid = 0x100000004
ctime = Sat Apr 10 13:23:54 CST 2021
mZxid = 0x100000004
mtime = Sat Apr 10 13:23:54 CST 2021
pZxid = 0x100000013
cversion = 10
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 6
~~~

### Zookeeper的Api操作

#### 获取客户端

~~~ java
public class ZookeeperTest {

    private String connectString="hadoop100:2181,hadoop101:2181,hadoop102:2181";

    ZooKeeper zooKeeper;

//    超时时间
    private int sessionTime=2000;

    public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
        ZookeeperTest zookeeperTest = new ZookeeperTest();
        zookeeperTest.getClint();
        zookeeperTest.createNode();

    }

    /**
     * 获取一个zookeeper客户端
     * @throws IOException
     */
    public  void getClint() throws IOException {
         zooKeeper = new ZooKeeper(connectString, sessionTime, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

                // 收到事件通知后的回调函数（用户的业务逻辑）
                System.out.println(watchedEvent.getType() + "--" + watchedEvent.getPath());

                // 再次启动监听
                try {
                    zooKeeper.getChildren("/", true);
                } catch (Exception e) {
                    e.printStackTrace();
                }

            }
        });
    }
}
~~~

#### 创建节点

~~~ java
/**
     * 创建一个节点
     */
    public void createNode() throws KeeperException, InterruptedException {
//        CreateMode表示创建持久的节点
        zooKeeper.create("/rzf","xiaorui".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

    }
~~~

#### 获取子节点并且监听变化

~~~ java
 /**
     * 获取子节点并且监听数据的变化
     */

    public void getDataAndWatch() throws KeeperException, InterruptedException {
//        首先获取根目录下的所有节点，false参数表示是否进行监控
        List<String> children = zooKeeper.getChildren("/", false);

        for (String s:children) {
            System.out.println(s);
        }
    }

~~~

#### 判断某一个节点是否存在

~~~ java
 /**
     * 判断某一个节点是否存在
     */
    public void isExists() throws KeeperException, InterruptedException {
        zooKeeper.exists("/rzf",false);
    }
~~~

## 监听服务器节点动态上下线案例

某分布式系统中，主节点可以有多台，可以动态上下线，任意一台客户端都能实时感知到主节点服务器的上下线。

![1618040908401](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1618040908401.png)

**实现**

1. 先在集群上创建/servers节点

~~~ java
create /servers "servers" Created /servers
~~~

2. 服务器端向Zookeeper注册代码

~~~ java
public class DistributeServer {

    private String connectString="hadoop100:2181,hadoop101:2181,hadoop102:2181";

    ZooKeeper zooKeeper;

    //    超时时间
    private int sessionTime=2000;

    /**
     * 获取链接
     * @throws IOException
     */
    public void getConnect() throws IOException {
        zooKeeper=new ZooKeeper(connectString, sessionTime, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

            }
        });
    }

    /**
     * 客户端注册，每次运行都根据hostname进行客户端注册
     * @param hostname
     * @throws KeeperException
     * @throws InterruptedException
     */
    public void regist(String hostname) throws KeeperException, InterruptedException {
//        注册就是向zookeeper写入数据
        zooKeeper.create("/server/servsrs",hostname.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

    }

    public void business() throws InterruptedException {
        Thread.sleep(Long.MAX_VALUE);
    }

    public static void main(String[] args) throws IOException, KeeperException, InterruptedException {

        DistributeServer distributeServer = new DistributeServer();
//        链接zookeeper集群
        distributeServer.getConnect();

//        注册节点
        distributeServer.regist(args[0]);

//        业务逻辑
        distributeServer.business();

    }
}

~~~

3. 客户端代码

~~~ java
public class DistributeClient {


    private String connectString="hadoop100:2181,hadoop101:2181,hadoop102:2181";

    ZooKeeper zooKeeper;

    //    超时时间
    private int sessionTime=2000;

    /**
     * 获取链接
     * @throws IOException
     */
    public void getConnect() throws IOException {
        zooKeeper=new ZooKeeper(connectString, sessionTime, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
//                监听后会执行这里的代码,写在这里可以进行多次监听
                try {
                    getChild();
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        });
    }

    /**
     * 做监听操作
     * @throws KeeperException
     * @throws InterruptedException
     */
    public void getChild() throws KeeperException, InterruptedException {
//        存储服务器节点的主机名称
        ArrayList<String> host = new ArrayList<>();

        List<String> children = zooKeeper.getChildren("/server", true);
//        遍历所有节点
        for (String s:children) {
//            获取节点s下面的值
            byte[] data = zooKeeper.getData("/server/" + s, false, null);
            host.add(new String(data));
        }

//        打印主机名
        System.out.println(host);
    }

    public void business() throws InterruptedException {
        Thread.sleep(Long.MAX_VALUE);
    }

    public static void main(String[] args) throws IOException, KeeperException, InterruptedException {

        DistributeClient distributeClient = new DistributeClient();
        distributeClient.getConnect();
        distributeClient.getChild();

    }
}
~~~

## 常见面试题

1. 请简述ZooKeeper的选举机制
2. ZooKeeper的监听原理是什么？
3. ZooKeeper的部署方式有哪几种？集群中的角色有哪些？集群最少需要几台机器？
   1. 部署方式单机模式、集群模式
   2. 角色：Leader和Follower
   3. 集群最少需要机器数：3
4. ZooKeeper的常用命令

ls create get delete set…