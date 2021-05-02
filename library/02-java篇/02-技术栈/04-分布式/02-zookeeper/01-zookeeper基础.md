# zookeeper基础

> 时间：<span style="color:#42B983;font-weight:bold;">2021年4月27日09:32:01</span>

## 一、zookeeper数据结构

<span style="color:#42B983;font-weight:bold;">简述：</span>ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称做一个
ZNode，每个ZNode都可以通过其路径唯一标识。

<span style="color:#FC5531; font-weight:bold;">ZooKeeper = 文件系统 + 监听通知机制 + ACL</span>

![image-20210502165741208](amWiki/images/lib_img/image-20210502165741208.png)

### 1、Znode节点类型 

* <span style="color:#FC5531; font-weight:bold;">持久化目录节点（PERSISTENT）</span>
  客户端与zookeeper断开连接后，该节点依旧存在
* <span style="color:#FC5531; font-weight:bold;">持久化顺序编号目录节点（PERSISTENT_SEQUENTIAL）</span>
  客户端与zookeeper断开连接后，该节点依旧存在，Zookeeper会给该节点按照顺序编号
* <span style="color:#FC5531; font-weight:bold;">临时目录节点（EPHEMERAL）</span>
  客户端与zookeeper断开连接后，该节点被删除
* <span style="color:#FC5531; font-weight:bold;">临时顺序编号目录节点（EPHEMERAL_SEQUENTIAL）</span>
  客户端与zookeeper断开连接后，该节点被删除，Zookeeper会给该节点按照顺序编号



### 2、Znode节点结构

<span style="color:#42B983;font-weight:bold;">结构：</span>  znode包含了数据、子节点引用、访问权限等等。

![image-20210502165804827](amWiki/images/lib_img/image-20210502165804827.png)

* <span style="color:#FC5531; font-weight:bold;">data：    </span>

  Znode存储的数据信息。 

  <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px">每一个znode默认能够存储1MB的数据（对于记录状态性质的数据来说，够了）</span></br>

* <span style="color:#FC5531; font-weight:bold;">ACL：</span>    

  记录Znode的访问权限，即哪些人或哪些IP可以访问本节点。

* <span style="color:#FC5531; font-weight:bold;">stat： </span>

  包含Znode的各种元数据，比如事务ID、版本号、时间戳、大小等等。 

* <span style="color:#FC5531; font-weight:bold;">child：</span>     

  当前节点的子节点引用，类似于二叉树的左孩子右孩子。

### 3、Znode状态信息

* stat

  <span style="color:#42B983;">-cZxid </span> 就是 Create ZXID，表示节点被创建时的事务ID。</br>
  <span style="color:#42B983;">-ctime</span> 就是 Create Time，表示节点创建时间。</br>
  <span style="color:#42B983;">-mZxid</span> 就是 Modified ZXID，表示节点最后一次被修改时的事务ID。</br>
  <span style="color:#42B983;">-mtime</span> 就是 Modified Time，表示节点最后⼀次被修改的时间。</br>
  <span style="color:#42B983;">-pZxid</span> 表示该节点的子节点列表最后⼀次被修改时的事务 ID。只有子节点列表变更才会更新 pZxid，子节点内容变更不会更新。</br>
  <span style="color:#42B983;">-cversion</span> 表示子节点的版本号。</br>
  <span style="color:#42B983;">-dataVersion</span> 表示内容版本号。</br>
  <span style="color:#42B983;">-aclVersion</span> 标识acl版本</br>
  <span style="color:#42B983;">-ephemeralOwner</span> 表示创建该临时节点时的会话 sessionID，如果是持久性节点那么值为 0 </br>
  <span style="color:#42B983;">-dataLength </span>表示数据⻓长度声明。</br>



## 二 、Watcher机制

<span style="color:#42B983;font-weight:bold;">简述：</span> <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">Znode发生变化（Znode本身的增加，删除，修改，以及子Znode的变化）可以通过Watch机制通知到客户端。</span>Watch事件具有one-time trigger（一次性触发）的特性，如果Watch监视的Znode有变化，那么就会通知设置该Watch的客户端。

### 1、watcher特征

* <span style="color:#FC5531; font-weight:bold;">一次性触发器</span>

  客户端在Znode设置了Watch时，如果Znode内容发生改变，那么客户端就会获得Watch事件。

* <span style="color:#FC5531; font-weight:bold;">异步发送客户端</span>

  Watch事件是异步发送到Client。Zookeeper可以保证客户端发送过去的更新顺序是有序的。

* <span style="color:#FC5531; font-weight:bold;">更新节点数据</span>

  Znode改变有很多种方式，例如：节点创建，节点删除，节点改变，子节点改变等等。Zookeeper维护了两个Watch列表，一个节点数据Watch列表，另一个是子节点Watch列表。

### 2、watcher运行机制

<span style="color:#42B983;font-weight:bold;">Watcher   的组成：</span><span style="color:#FC5531; font-weight:bold;"> zk客户端，zk服务器，watcherManager</span>

<span style="color:#42B983;font-weight:bold;">Watcher工作机制：</span> <span style="color:#FC5531; font-weight:bold;">客户端注册 Watcher、服务器处理 Watcher 和客户端回调 Watcher</span>

![image-20210502165827943](amWiki/images/lib_img/image-20210502165827943.png)

![image-20210502165850092](amWiki/images/lib_img/image-20210502165850092.png)

#### ☛ 客户端注册watcher

![image-20210502165910531](amWiki/images/lib_img/image-20210502165910531.png)

<span style="color:#42B983;font-weight:bold;">在发送一个 Watch 监控事件的会话请求时，ZooKeeper 客户端主要做了两个工作：</span> </br>

1. <span style="color:#FC5531; font-weight:bold;"> 标记该会话是一个带有 Watch 事件的请求</span></br>
2. <span style="color:#FC5531; font-weight:bold;"> 将 Watch 事件存储到 </span>ZKWatchManager</br>

#### ☛ 服务器处理watcher

<span style="color:#42B983;font-weight:bold;">Zookeeper 服务端处理 Watch 事件基本也做了两个工作：</span>

1. <span style="color:#FC5531; font-weight:bold;"> 解析收到的请求是否带有 Watch 注册事件</span></br>
2. <span style="color:#FC5531; font-weight:bold;"> 将对应的 Watch 事件存储到 WatchManager</span></br>

当 ZooKeeper 服务器接收到一个客户端请求后，首先会对请求进行解析，判断该请求是否包含 Watch 事件.

ZooKeeper 底层是通过 FinalRequestProcessor 类中的 processRequest 函数实现的。当 getDataRequest.getWatch() 值为 True 时，表明该请求需要进行 Watch 监控注册。并通过 zks.getZKDatabase().getData 函数将 Watch 事件注册到服务端的 WatchManager 中

#### ☛ 客户端回调watcher

![image-20210502165929240](amWiki/images/lib_img/image-20210502165929240.png)



## 三、zookeeper环境搭建

> <span style="color:#42B983;font-weight:bold;">zookeeper版本号 & 环境：</span> <span style="color:#FC5531; font-weight:bold;">  3.7.0      jdk1.8         linux </span>   

### 1、单机环境

#### ☛ jdk安装

> jdk安装不是介绍重点（网上有很多教程），这里我们用的jdk1.8

#### ☛ zookeeper安装

* <span style="color:#42B983;font-weight:bold;">下载并解压zookeeper安装包</span>

~~~ shell
$ tar -xzvf [zookeeper_install_package]
~~~



* <span style="color:#42B983;font-weight:bold;">进入安装目录创建data目录</span>

~~~shell
# 进入zk目录
$ cd [zookeeper_home]

# 创建data目录
$ mkdir data
~~~



* <span style="color:#42B983;font-weight:bold;">修改zk配置文件，指定data目录</span>

~~~shell
# 进入zk配置目录
$ cd %zk_home%/conf

# 配置zoo.cfg
$ mv zoo_sample.cfg zoo.cfg

# 指定data目录  修改zoo.cfg中的data属性 如下：
dataDir=%zk_home%/data
~~~



* <span style="color:#42B983;font-weight:bold;">启/停zk</span>

~~~shell
# 命令行 启动zk
$ ./zkServer.sh start	

# 命令行 停止zk
$ ./zkServer.sh stop

~~~

### 2、集群环境

#### ☛ 准备工作

* <span style="color:#42B983;font-weight:bold;">安装jdk</span>

  安装jdk不是重点不做赘述（度娘or谷狗），这里我们用jdk1.8

  

* <span style="color:#42B983;font-weight:bold;">上传zookeeper安装包</span>

  上传zk_install_package到你喜欢的目录

  

* <span style="color:#42B983;font-weight:bold;">装备三份zk</span>

  * <span style="color:#FC5531; font-weight:bold;">  复制三份zk，并分别命名如下：</span>

  ~~~shell
  [custom_path]/zookeeper_01
  [custom_path]/zookeeper_01
  [custom_path]/zookeeper_01
  ~~~

  * <span style="color:#FC5531; font-weight:bold;">  分别为上边的zk，配置cfg文件，并指定data目录以及端口号如下：</span>

  ~~~shell
  # zk节点一
  clientPort=2181
  dataDir=[custom_path]/zookeeper_01/data
  
  # zk节点二
  clientPort=2182
  dataDir=[custom_path]/zookeeper_02/data
  
  # zk节点三
  clientPort=2183
  dataDir=[custom_path]/zookeeper_03/data
  ~~~



#### ☛  集群配置

* <span style="color:#42B983;font-weight:bold;">创建myid文件</span>

  ![image-20210502141009861](amWiki/images/lib_img/image-20210502141009861.png)

  <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">在每个zookeeper的 data 目录下创建一个 myid 文件，内容分别是1、2、3 。</span></br>

  <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">这个文件就是记录 每个服务器的ID</span></br>



* <span style="color:#42B983;font-weight:bold;">配置集群服务其列表</span>

![image-20210502140729022](amWiki/images/lib_img/image-20210502140729022.png)

~~~ shell
# server.服务器ID=服务器IP地址：服务器之间通信端口：服务器之间投票选举端口
server.1=ip:28881:3881
server.2=ip:28882:3882
server.3=ip:28883:3883
# 注：这里的“服务器之间通信端口”切记不要和clientPort混淆 （配置一样）
~~~



#### ☛  集群启动

* <span style="color:#42B983;font-weight:bold;">集群服务启动</span>

  <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">依次启动三个zk实例，其中有一个leader和两个follower</span>



## 四、zookeeper基本使用

### 1、命令行

* <span style="color:#42B983;font-weight:bold;">服务启/停、状态检查</span>

~~~ shell
# zk服务启动
$ ./zkServer.sh start

# zk服务停止
$ ./zkServer.sh stop

# zk状态检查
$ ./zkServer.sh status
~~~



* <span style="color:#42B983;font-weight:bold;">连接zk服务</span>

~~~ shell
# 默认连接本地zk
$ sh %zk_home%/bin/zkCli.sh

# 连接指定ip端口的zk服务
$ sh %zk_home%/bin/zkCli.sh -server ip:port
~~~



* <span style="color:#42B983;font-weight:bold;">查看节点信息</span>

  ![image-20210502144727846](amWiki/images/lib_img/image-20210502144727846.png)

~~~shell
# 注：以下命令都是在连接到zk_server后执行的
# 查看根节点信息
$ ls / [watch]

# 查看指定目录节点的信息
$ ls /[zNode_name] [watch]

# 查看节点信息（包含更新次数等信息）
$ ls -s /[zNode_name] [watch]    			#在老版本中这样使用：ls2 /[zNode_name]

# 注：[watch] 监听机制（一次）：监听当前节点目录是否发生变化（添加、删除当前节点或子节点）
~~~



* <span style="color:#42B983;font-weight:bold;">创建节点</span>

  ![image-20210502144800580](amWiki/images/lib_img/image-20210502144800580.png)

~~~shell
# 创建节点 注：要指定节点值
$ create [-s] [-e] /ryanc-site site.ryanc
# 注：-s：有序节点； -e：临时节点
~~~



* <span style="color:#42B983;font-weight:bold;">查看节点的值</span>

~~~shell
# 普通查看
$ get /[zNode_name] [watch]
# 注：[watch] 监听机制（一次）：监听当前节点的值是否发生变化
~~~



* <span style="color:#42B983;font-weight:bold;">更新节点的值</span>

~~~shell
# 普通更新
$ set /[zNode_name] [nValue]
~~~



* <span style="color:#42B983;font-weight:bold;">查看节点状态</span>

~~~shell
$ stat /[zNode_name]
~~~



* <span style="color:#42B983;font-weight:bold;">删除节点</span>

~~~shell
# 普通删除 注：不能删除子节点不为空的节点
$ delete /[zNode_name]

# 递归删除
$ rmr /[zNode_name]
~~~







### 2、api调用

#### ☛ 添加maven依赖

~~~xml
<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.7.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>
</dependency>
~~~



#### ☛  连接zk

~~~ java
// 创建zk对象，连接zk—server
// 参数列表：
// 		connectString => zk_ip:zk_client_port
//		sessionTimeout => 超时时间
// 		watcher 	  => watcher机制

ZooKeeper zk = new ZooKeeper("192.168.8.124:2181",2000,new Watcher() {
    @Override
    public void process(WatchedEvent event) {
        System.out.println("触发了 "+ event.getType() + "类型的事件。。。");
    }
});
~~~



* <span style="color:#42B983;font-weight:bold;">创建节点</span>

~~~java
// zk对象创建节点
// 参数列表：
// 		path  => 节点名称（目录）
//		data  => 节点值
//		acl   => 节点访问权限
//  	cMode => 节点类型：临时节点、持久节点、顺序节点。。。

// 创建父节点
String zn_path= zk.create("/test_l", "api创建".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

// 创建子节点
String child_zn_path = zk.create("/test_l/child", "api创建,子节点".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT))
~~~



* <span style="color:#42B983;font-weight:bold;">获取节点值</span>

~~~java
// 获取节点的值
// 参数列表：
//		path	=> 节点目录
//		watch	=> 是否监听
// 		stat	=> 指定数据版本等等
byte[] b_data = zk.getData("/test_l/child", false, null);
String data = new String(b_data);
~~~



* <span style="color:#42B983;font-weight:bold;">更新节点值</span>

~~~ java
// 更新节点值
// 参数列表：
//		path	=> 待更新节点目录
//		data	=> 待更新的内容
//		version	=> 指定要匹配的版本 -1：表是匹配任何版本
Stat stat = zk.setData("/test_lll", "update datas".getBytes(), -1);
String data = new String(zk.getData("/test_lll", false, null));
System.out.println(data);
~~~



* <span style="color:#42B983;font-weight:bold;">判断节点是否存在</span>

~~~java
// 参数列表：
//		path	=> 带判断的节点目录
//		watch	=> 是否监听
Stat stat_exists = zk.exists("/test_llls", false);
System.out.println(stat_exists);
~~~





* <span style="color:#42B983;font-weight:bold;">删除节点</span>

~~~java
// 参数列表：
//		path	=> 待删除节点目录
//		version	=> 指定要匹配的版本 -1：表是匹配任何版本
zk.delete("/test_lll/child", -1);
System.out.println(zk.exists("/test_lll/child", false)); // 查看是否删除成功
~~~



## 附录

### 参考

👉  [zookeeper_微毂的博客-CSDN博客](https://blog.csdn.net/weixin_42749734/category_10783522.html)

👉  [Zookeeper数据结构_HelloWorld搬运工-CSDN博客_zookeeper数据结构](https://blog.csdn.net/wufaliang003/article/details/76043892)