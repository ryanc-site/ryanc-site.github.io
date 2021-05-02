# zookeeper进阶

> 时间：<span style="color:#42B983;font-weight:bold;">2021年05月02日17:09:24</span>

## 一、源码解析

### 1、源码下载



### 2、启动流程



### 3、选主流程



### 4、watcher机制



## 二、zk应用场景

### 1、配置中心

#### ☛ 场景 & 需求

<span style="color:#42B983;font-weight:bold;">简述：</span>集群上有很多个节点运行同一个任务，这个任务会有一些可能经常改变的配置参数</br>
要求是当配置参数改变之后能够很快地同步到每个节点上</br>
如果将这些配置参数放在本地文件中则每次都要修改本地文件费时费力还可能会有遗漏</br>
所以这个时候一个比较自然的想法就是将配置单独提取出来作为一个服务</br>



#### ☛  选型分析

* <span style="color:#42B983;font-weight:bold;">配置中心的核心</span>

  * <span style="color:#FC5531; font-weight:bold;"> 低延时：</span> <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">配置改变后能够尽快的将最新配置同步给每一个节点。</span>
  * <span style="color:#FC5531; font-weight:bold;"> 高可用：</span> <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">配置中心需要能够稳定不间断的提供服务。</span>

* <span style="color:#42B983;font-weight:bold;">配置中心实现</span>

  <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;">发布/订阅系统一般有两种设计模式，分别是推(Push)模式和拉(Pull)模式。</span>

  * <span style="color:#FC5531; font-weight:bold;"> 推模式：</span><span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">服务端主动将数据更新发送给所有订阅的客户端。</span>
  * <span style="color:#FC5531; font-weight:bold;"> 拉模式：</span><span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">客户端通过采用定时轮询拉取。</span>



#### ☛ 实现分析

<span style="color:#42B983;font-weight:bold;">分析：</span> ZooKeeper采用的是推拉相结合的方式：</br>

客户端向服务端注册自己需要关注的节点，一旦该节点的数据发生变更，那么服务端就会向相应的客户端发送Watcher事件通知，客户端接收到这个消息通知之后，需要主动到服务端获取最新的数据。 </br>

如果将配置信息存放到ZK上进行集中管理，那么通常情况下，应用在启动的时候会主动到ZK服务器上进行一次配置信息的获取，同时，在指定上注册一个Watcher监听，这样一来，但凡配置信息发生变更，服务器都会实时通知所有订阅的客户端，从而达到实时获取最新配置信息的目的。</br>

![image-20210502195628408](amWiki/images/lib_img/image-20210502195628408.png)

* <span style="color:#FC5531; font-weight:bold;"> 配置信息特性</span>

  * 数据量通常比较小
  * 数据内容在运行时会发生变化
  * 集群中各机器共享、配置一致

* <span style="color:#FC5531; font-weight:bold;"> 配置存储</span>

  在进行配置管理之前，首先我们需要将初始化配置存储到ZK上去，一般情况下，我们可以在ZK上选取一个数据节点用于配置的存储，我们将需要集中管理的配置信息写入到该数据节点中去。

* <span style="color:#FC5531; font-weight:bold;"> 配置获取</span>

  集群中每台机器在启动初始化阶段，首先会从上面提到的ZK的配置节点上读取数据库信息，同时，客户端还需要在该配置节点上注册一个数据变更的Watcher监听，一旦发生节点数据变更，所有订阅的客户端都能够获取数据变更通知。

* <span style="color:#FC5531; font-weight:bold;"> 配置变更</span>

  在系统运行过程中，可能会出现需要进行数据切换的情况，这个时候就需要进行配置变更。借助ZK，我们只需要对ZK上配置节点的内容进行更新，ZK就能够帮我们将数据变更的通知发送到各个客户端，每个客户端在接收到这个变更通知后，就可以重新进行最新数据的获取。

#### ☛代码实现

<span style="color:#42B983;font-weight:bold;">简述：</span> <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;">代码包括如下三个部分:配置客户端类ConfigClient、应用服务类Service、本地配置类MyConfig,关系如下：</span></br>

<span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">ConfigClient通过与zookeeper服务器交互，修改MyConfig；</span></br>

<span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">Service在业务逻辑中直接使用MyConfig；</span> </br>

<span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">Service启动时，完成ConfigClient的启动、初始化。</span> </br>

* <span style="color:#FC5531; font-weight:bold;">客户端类ConfigClient</span>

~~~ java
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import java.io.IOException;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;
 
/**
 * +--------------------------------+ <br>
 * | Even in a galaxy far,far away.  | <br>
 * | 即使是在遥远的星河里也一样  !      |<br>
 * +--------------------------------+ <br>
 * @author ryan
 * @date May 2, 2021
 * @desc 
 * 		实际生产中，配置中心的客户端以java注解的形式使用
 * 		通过maven引入依赖，随着服务的启动，spring容器自动加载
 *
 */
public class ConfigClient {
 
    private static final Logger log = LoggerFactory.getLogger(ConfigClient.class);
 
    private ZooKeeper zkClient;
 
    private String servicePath;
 
    private Map<String, String> localConfigs;
 
    /**
     * 服务启动时调用，将服务节点记录到/config-center节点下
     */
    public void init(String appkey) throws IOException {
        zkClient = new ZooKeeper("192.168.8.124:2181", 15000,
                event -> log.info("event:{}", event));
        this.servicePath = "/config-center/" + appkey;
        createNode(servicePath, new byte[0]);
        localConfigs = new HashMap<>(10);
    }
 
    /**
     * 注意，调用create时，data还被当做上下文参数ctx的值；在异步回调时，如果发生connectionLoss，ctx可以作为data再次调用create方法
     */
    private void createNode(String path, byte[] data) {
        zkClient.create(path, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT, new AsyncCallback.StringCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, String name) {
                switch (KeeperException.Code.get(rc)) {
                    case CONNECTIONLOSS:
                        createNode(path, (byte[]) ctx);
                        break;
                    case OK:
                        log.info("{} create successfully!", path);
                        break;
                    case NODEEXISTS:
                        log.info("{} exists!", path);
                        break;
                    default:
                        log.error("Something went wrong: ", KeeperException.create(KeeperException.Code.get(rc), path));
                }
            }
        }, data);
    }
 
    /**
     * 服务启动时调用，将本地的每个配置项作为节点，记录到服务节点下
     */
    public void registerConfig(String fieldName, Object configValue) {
        byte[] data = configValue == null ? new byte[0] : configValue.toString().getBytes();
        String configPath = this.servicePath + "/" + fieldName;
        createNode(configPath, data);
        localConfigs.put(configPath, fieldName);
        reloadConfig(configPath);
    }
 
    /**
     * DataCallback：异步方式查询数据，如果连接丢失，重新执行reloadConfig
     * Watcher：监听节点数据变更，即监听配置的值发生变更
     */
    public void reloadConfig(String configPath) {
        zkClient.getData(configPath, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                switch (event.getType()) {
                    case NodeDataChanged:
                        log.info("数据修改事件发生:{}", event.getPath());
                        reloadConfig(event.getPath());
                }
            }
        }, new AsyncCallback.DataCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, byte[] data, Stat stat) {
                switch (KeeperException.Code.get(rc)) {
                    case CONNECTIONLOSS:
                        reloadConfig(path);
                    case OK:
                        String fieldName = localConfigs.get(path);
                        String strValue = new String(data);
                        reloadValue(fieldName, strValue);
                }
 
            }
        }, new Stat());
    }
 
    /**
     * 通过反射的方式，将配置的变更set到应用服务的MyConfig类的对应field中
     * 实际生产中，依托配置中心的客户端field级注解：不必将所有配置统一写到特定的类中；配置项名称不必与字段名称fieldName相同
     */
    private void reloadValue(String fieldName, String strValue) {
        try {
            Field field = MyConfig.class.getDeclaredField(fieldName);
            field.setAccessible(true);
            if (field.getType() == String.class) {
                field.set(MyConfig.getInstance(), strValue);
            } else if (field.getType() == Integer.class) {
                field.set(MyConfig.getInstance(), Integer.valueOf(strValue));
            }
        } catch (IllegalAccessException e) {
            log.error("reload zk config to local config fail:", e);
        } catch (NoSuchFieldException e) {
            log.error("reload zk config to local config fail:", e);
        }
    }
}
~~~



* <span style="color:#FC5531; font-weight:bold;">应用服务类Service</span>

~~~java
import org.apache.log4j.BasicConfigurator;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import java.io.IOException;
import java.util.Scanner;
 
/**
 * +--------------------------------+ <br>
 * | Even in a galaxy far,far away.  | <br>
 * | 即使是在遥远的星河里也一样  !      |<br>
 * +--------------------------------+ <br>
 * @author ryan
 * @date May 2, 2021
 * @desc 服务类
 *
 */
public class Service {
 
    private static final Logger log = LoggerFactory.getLogger(ConfigClient.class);
 
    /**
     * 服务唯一标识
     */
    private String appkey = "service-1";
    private ConfigClient configClient;
    /**
     * 应用服务初始化
     * 在实际生产中，通过spring完成configClient的初始化、依赖注入
     */
    public void init() throws IOException {
        configClient = new ConfigClient();
        configClient.init(appkey);
        configClient.registerConfig("whiteList", MyConfig.getInstance().whiteList);
        configClient.registerConfig("limit", MyConfig.getInstance().limit);
    }
    public static void main(String[] args) {
    	BasicConfigurator.configure();
        Service service = new Service();
        try {
            service.init();
            Thread.sleep(2000);
        } catch (Exception e) {
        	log.error("start fail....");
            System.exit(-1);
        }
        Scanner sc = new Scanner(System.in);
        while (true) {
            log.info("回车查看当前配置：");
            sc.nextLine();
            log.info("service config whiteList: {}, limit:{}", MyConfig.getInstance().getWhiteList(),
                    MyConfig.getInstance().limit);
        }
    }
}
~~~



* <span style="color:#FC5531; font-weight:bold;">本地配置类MyConfig</span>

~~~java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
 
/**
 * +--------------------------------+ <br>
 * | Even in a galaxy far,far away.  | <br>
 * | 即使是在遥远的星河里也一样  !      |<br>
 * +--------------------------------+ <br>
 * @author ryan
 * @date May 2, 2021
 * @desc 
 * 		实际生产中：
 * 			① myConfig可以作为单例bean托管在spring容器中
 * 			② 利用java field级注解，不必将每一项配置统一写到特定类中，
 *
 */
public class MyConfig {
 
    private volatile static MyConfig instance;
 
    private MyConfig() {
    }
 
    /**
     * 单例模式：延迟加载与双重检查锁定
     */
    public static MyConfig getInstance() {
        if (instance == null) {
            synchronized (MyConfig.class) {
                if (instance == null) {
                    instance = new MyConfig();
                }
            }
        }
        return instance;
    }
 
    /**
     * 配置1：白名单列表
     */
    public String whiteList = "lileim,hanmeimei";
 
    public List<String> getWhiteList() {
        try {
            return new ArrayList(Arrays.asList(whiteList.split(",")));
        } catch (Exception e) {
            return new ArrayList<>();
        }
    }
 
    /**
     * 配置2：限制
     */
    public Integer limit = 10;
 
}
~~~



###  2、负载均衡



### 3、命名服务



### 4、DNS服务



### 5、集群管理



### 6、分布式锁

#### ☛  场景 & 需求

<span style="color:#42B983;font-weight:bold;">简述：</span> 在分布式微服务场景中，同一个微服务的多个节点同时对资源进行操作时就会有分布式同步问题（类似于多线程操作资源场景）

#### ☛  选型分析

<span style="color:#42B983;font-weight:bold;">分布式锁的核心</span>

* <span style="color:#FC5531; font-weight:bold;"> 独占：</span> <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">任何时候只能有个微服务获取到该锁</span>
* <span style="color:#FC5531; font-weight:bold;"> 时序性：</span> <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">所有参与获取锁的微服务都会被安排执行</span>

#### ☛ 实现分析

<span style="color:#42B983;font-weight:bold;">实现思路：</span> 为每一个执行的客户端都创建一个有序临时节点；每个客户端都判断自己的临时节点序号是不是最小的：

* 序号最小：则获取锁执行相关操作，释放锁（删除节点）
* 序号非最小：则监听前一个节点，当前一个节点删除时，获取锁，一次类推

![image-20210502194804999](amWiki/images/lib_img/image-20210502194804999.png)

#### ☛ 代码实现

<span style="color:#42B983;font-weight:bold;">简述：</span> <span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;">代码包括如下三个部分:分布式锁工具类ZkLock、库存数据模型stock、分布式锁测试StockMain,关系如下：</span></br>

<span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">ZkLock通过实现Lock对外提供lock API</span></br>

<span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">stock 库存数据模型，角色共享资源</span> </br>

<span style=" font-weight:bold;border-bottom:1px dashed #000; height:50px;width:350px;color:#0081EF">StockMain 用多线程模拟分布式场景，测试分布式锁</span> </br>

* ZkLock

~~~java
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import java.util.List;
import java.util.TreeSet;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;


/**
 * +--------------------------------+ <br>
 * | Even in a galaxy far,far away.  | <br>
 * | 即使是在遥远的星河里也一样  !      |<br>
 * +--------------------------------+ <br>
 * @author ryan
 * @date May 2, 2021
 * @desc 基于zk 实现的分布式锁
 *
 */
public class ZkLock implements Lock {


    //zk客户端
    private ZooKeeper zk;
    //zk是一个目录结构，locks
    private String root = "/locks";
    //锁的名称
    private String lockName;
    //当前线程创建的序列node
    private ThreadLocal<String> nodeId = new ThreadLocal<>();
    //用来同步等待zkclient链接到了服务端
    private CountDownLatch connectedSignal = new CountDownLatch(1);
    private final static int sessionTimeout = 3000;
    private final static byte[] data= new byte[0];


    public ZkLock(String config, String lockName) {
        this.lockName = lockName;

        try {
            zk = new ZooKeeper(config, sessionTimeout, new Watcher() {

                @Override
                public void process(WatchedEvent event) {
                    // 建立连接
                    if (event.getState() == Event.KeeperState.SyncConnected) {
                        connectedSignal.countDown();
                    }
                }

            });

            connectedSignal.await();
            Stat stat = zk.exists(root, false);
            if (null == stat) {
                // 创建根节点
                zk.create(root, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    class LockWatcher implements Watcher {
        private CountDownLatch latch = null;

        public LockWatcher(CountDownLatch latch) {
            this.latch = latch;
        }

        @Override
        public void process(WatchedEvent event) {

            if (event.getType() == Event.EventType.NodeDeleted)
                latch.countDown();
        }
    }

    @Override
    public void lock() {
        try {
            // 创建临时子节点
            String myNode = zk.create(root + "/" + lockName , data, ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL);

            System.out.println(Thread.currentThread().getName()+myNode+ "created");

            // 取出所有子节点
            List<String> subNodes = zk.getChildren(root, false);
            TreeSet<String> sortedNodes = new TreeSet<>();
            for(String node :subNodes) {
                sortedNodes.add(root +"/" +node);
            }

            String smallNode = sortedNodes.first();


            if (myNode.equals( smallNode)) {
                // 如果是最小的节点,则表示取得锁
                System.out.println(Thread.currentThread().getName()+ myNode+"get lock");
                this.nodeId.set(myNode);
                return;
            }

            String preNode = sortedNodes.lower(myNode);

            CountDownLatch latch = new CountDownLatch(1);
            Stat stat = zk.exists(preNode, new LockWatcher(latch));// 同时注册监听。
            // 判断比自己小一个数的节点是否存在,如果不存在则无需等待锁,同时注册监听
            if (stat != null) {
                System.out.println(Thread.currentThread().getName()+myNode+
                        " waiting for " + root + "/" + preNode + " released lock");

                latch.await();// 等待，这里应该一直等待其他线程释放锁
                nodeId.set(myNode);
                latch = null;
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }


    @Override
    public void unlock() {
        try {
            System.out.println(Thread.currentThread().getName()+ "unlock ");
            if (null != nodeId) {
                zk.delete(nodeId.get(), -1);
            }
            nodeId.remove();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }



    @Override
    public void lockInterruptibly() throws InterruptedException {

    }

    @Override
    public boolean tryLock() {
        return false;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return false;
    }



    @Override
    public Condition newCondition() {
        return null;
    }
}

~~~

* Stock

~~~java
/**
 * +--------------------------------+ <br>
 * | Even in a galaxy far,far away.  | <br>
 * | 即使是在遥远的星河里也一样  !      |<br>
 * +--------------------------------+ <br>
 * @author ryan
 * @date May 2, 2021
 * @desc 分布式锁 演示 数据模型
 *
 */
public class Stock {

    //库存为1
    private static int num = 1;

    public static boolean reduseStock(){

        if(num>0){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            num--;
            return true;
        }
         return false;
    }
}

~~~

* StockMain

~~~java

/**
 * +--------------------------------+ <br>
 * | Even in a galaxy far,far away.  | <br>
 * | 即使是在遥远的星河里也一样  !      |<br>
 * +--------------------------------+ <br>
 * @author ryan
 * @date May 2, 2021
 * @desc 分布式锁测试类
 *
 */
public class StockMain {

    private static ZkLock zkLock;

    static{
        zkLock = new ZkLock("127.0.0.1:2181","stock_zk");
    }


    static class StockThread implements Runnable{

        @Override
        public void run() {

        	// 上锁
            zkLock.lock();

            // 扣减库存
            boolean b = new Stock().reduseStock();

            // 解锁
            zkLock.unlock();

            if(b){
                System.out.println(Thread.currentThread().getName()+":扣减库存成功！");
            }else{
                System.out.println(Thread.currentThread().getName()+":扣减库存失败！");
            }
        }
    }

    public static void main(String[] args) {
        new Thread(new StockThread(),"用户1").start();
        new Thread(new StockThread(),"用户2").start();
    }
}

~~~



### 7、分布式队列



## 附录

### 参考

👉 [zookeeper-配置中心](https://blog.csdn.net/ye201622021113/article/details/112760952)

