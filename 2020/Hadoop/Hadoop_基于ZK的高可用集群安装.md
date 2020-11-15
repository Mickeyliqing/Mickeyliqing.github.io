# `Hadoop` 基于 `ZK` 的高可用集群安装
## 注意事项
**1、安装 `ZooKeeper` 配置注意事项** 
- 1、新建 `data` 文件夹，注意这个文件夹的权限。
- 2、新建文件 `myid` ,注意这个文件的权限。
- 3、要分别向 `myid` 写入1,2,3。

**2、修改了 `hosts` 主机名后，各个主机需要再次执行**
```xshell
ssh master
ssh worker_01
ssh worker_02
```

**3、第一次启动的时候，需要格式化，第二次在启动的时候就不需要在格式化了，但 `worker03` 上的 `ResourceManager` 启动不起来，要单独启动。**

**4、从新修改了 `Hadopp` 集群的配置文件后，不需要从新在格式化` NameNode` 。**

**5、`Namenode` 主备切换不成功的原因**
（两者修改其中一项即可）

- 1：没有安装：`psmisc`.

- 2：配置里没有配：`shell(/bin/true)××`

**6、`hadoop version` 不显示**

- 6.1、在配置文件里` vim  /etc/profile` 里加

```config
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

- 6.2、配置生效：

```xshell
source /etc/profile
```

**7、WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable**

- 7.1、下载：`hadoop-native-64-2.*.0.tar`。

- 7.2、执行

```xshell
tar -xvf hadoop-native-64-2.7.0.tar -C $HADOOP_HOME/lib
```

- 7.3、添加环境变量：`vim /etc/profile`

```config
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
```

- 7.4、配置生效：

```xshell
source /etc/profile
```

## 安装步骤

### 第一步

**首先准备需要的软件：（最好注意一下版本）。**

- JDK （1.8）

- ZooKeeper（3.4.6）

- Hadoop（2.7.2）

### 第二步：

**准备硬件。**

- 这里准备三台计算机，分别是 `master`，`worker01`，`worker02`。

- `master` 和 `worker01` 做 `NameNode`主备。

- `worker01` 和 `worker02` 做 `ResourceManager`主备。

### 第三步：

**确定安装顺序**

(首次安装，请严格按照顺序来执行)

**root 用户下执行**

- 修改 `host`

- 关闭防火墙

- 上传文件

- JDK安装

**新建 `hadoop`用户，并赋值 `root` 权限**

**`hadoop` 用户执行**

- 免密登录（本机和各个服务器之间）

- 安装 `zookeeper` 并配置

- 安装 `hadoop` 集群并配置

**在进行安装和配置的时候，一定要注意用户，以及权限，尤其新建文件夹和新建文件操作**

### 第四步

**JDK安装和配置**

**（三台机器都要这样操作）**

- 1：上传文件到指定位置。

- 2：解压。

- 3：修改系统配置（`vim /etc/profile`）。

```config
    export JAVA_HOME=/home/JDK/jdk1.8.0_191
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
    export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
```
- 4：配置生效 `source /etc/profile`。

- 5：验证 `java -version`。

### 第五步

**ZooKeeper 安装和配置**

**（三台机器都要这样操作）**

- 1：上传文件到指定目录。

- 2：解压。

- 3：在 `zookeeper` 目录下新建 `data` 文件夹。

- 4：`conf` 目录下 `zoo_sample.cfg`重名为 `zoo.cfg`，添加如下配置。

```config
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/usr/local/zookeeper/data/
    dataLogDir=/usr/local/zookeeper/log/
    clientPort=2181
    # server.1 这个1是服务器的标识，可以是任意有效数字，标识这是第几个服务器节点，这个标识要写到dataDir目录下面myid文件里
    # 指名集群间通讯端口和选举端口
    server.1=master:2287:3387
    server.2=worker01:2287:3387
    server.3=worker02:2287:3387
```

- 5：进入`data` 目录，新建文件 `myid`（`touch myid`）。

- 6：在 `master` 机器上执行 `echo 1 > myid` 。

- 7：在 `worker01` 机器上执行 `echo 2 > myid` 。

- 8：在 `worker02` 机器上执行 `echo 3 > myid` 。

(`myid` 的大小是两个字节【也就是只有一个数字；不要有空格】{查看方法就是vi进去以后光标闪烁是在 `1` 上，并且移动光标移动不了)。

- 9：配置环境变量。

```config
    export ZOOKEEPER_HOME=/home/ZK/zookeeper
    export PATH=$PATH:${ZOOKEEPER_HOME}/bin
```

- 10：配置项生效 `source /etc/profile` 。

- 11：启动 `ZK zkServer.sh start`。

- 12：查看状态 `zkServer.sh status` 。

(如果启动出错，注意查看启动日志，问题就一目了然了)

### 第六步

**Hadoop安装和配置**

**（三台机器都要这样操作）**

- 1：上传文件到指定的位置。

- 2：解压。

- 3：配置环境变量。

```config
    export HADOOP_HOME=/home/HA/hadoop
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export HADOOP_PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
    export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
    export PATH=$PATH:${HADOOP_PATH}
```

- 4：环境变量生效  `source /etc/profile`。

- 5：Hadoop集群配置。
   1. `hadoop-evn.sh`。
   2. `core-site.xml`。
   3. `hdfs-site.xml`。
   4. `yarn-site.xml`。
   5. `mapred-site.xml`。
   6. `slaves`。

- 5.1、`hadoop-env.sh`。

```config
export JAVA_HOME=/home/JDK/jdk1.8.0_191
```

- 5.2、`core-site.xml`。

```xml
<configuration>
	  <property>
	  <!-- 指定 namenode 的 hdfs 协议文件系统的通信地址 和hdfs-site.xml 里的配置value相同-->
		  <name>fs.defaultFS</name>
		  <value>hdfs://mycluster</value>
	  </property>
	  <property>
		  <name>hadoop.tmp.dir</name>
       	  <value>file:/home/HA/hadoop/tmp</value>
	  </property>
          <property>
		  <name>ha.zookeeper.quorum</name>
		  <value>master:2181,worker01:2181,worker02:2181</value>
	  </property>
          <property>
		  <name>ha.zookeeper.session-timeout.ms</name>
		  <value>10000</value>
	  </property>
</configuration>
```

- 5.3、`hdfs-site.xml`。

```xml
<configuration>
        <property>
            <name>dfs.replication</name>
            <value>3</value>
        </property>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/home/HA/hadoop/tmp/dfs/name</value>
        </property>
	 <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/home/HA/hadoop/tmp/dfs/data</value>
        </property>
        <property>
            <name>dfs.permissions</name>
            <value>false</value>
        </property>
        <property>
        <!-- 指定 namenode 的 hdfs 协议文件系统的通信地址 和hdfs-site.xml 里的配置value相同-->
            <name>dfs.nameservices</name>
            <value>mycluster</value>
        </property>
        <property>
            <name>dfs.ha.namenodes.mycluster</name>
            <value>nn1,nn2</value>
        </property>
        <property>
            <name>dfs.namenode.rpc-address.mycluster.nn1</name>
            <value>master:9000</value>
        </property>
        <property>
            <name>dfs.namenode.rpc-address.mycluster.nn2</name>
            <value>worker01:9000</value>
        </property>
        <property>
            <name>dfs.namenode.http-address.mycluster.nn1</name>
            <value>master:50070</value>
        </property>
        <property>
            <name>dfs.namenode.http-address.mycluster.nn2</name>
            <value>worker01:50070</value>
        </property>
        <property>
            <name>dfs.namenode.shared.edits.dir</name>
            <value>qjournal://master:8485;worker01:8485;worker02:8485/mycluster</value>
        </property>
        <property>
            <name>dfs.journalnode.edits.dir</name>
            <value>/home/HA/hadoop/journalnode/data</value>
        </property>
        <property>
            <name>dfs.ha.fencing.methods</name>
            <value>sshfence
		shell(/bin/true)
		    </value>
        </property>
        <property>
            <name>dfs.ha.fencing.ssh.private-key-files</name>
            <value>/home/hadoop/.ssh/id_rsa</value>
        </property>
        <property>
            <name>dfs.ha.fencing.ssh.connect-timeout</name>
            <value>30000</value>
        </property>
        <property>
            <name>dfs.client.failover.proxy.provider.mycluster</name>
            <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
        </property>
        <property>
            <name>dfs.ha.automatic-failover.enabled</name>
            <value>true</value>
        </property>
</configuration>
```

- 5.4、`yarn-site.xml`。

```xml
<configuration>
  <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
  </property>
  <property>
      <name>yarn.log-aggregation-enable</name>
      <value>true</value>
  </property>
  <property>
      <name>yarn.log-aggregation.retain-seconds</name>
      <value>86400</value>
  </property>
  <property>
      <name>yarn.resourcemanager.ha.enabled</name>
      <value>true</value>
  </property>
  <property>
      <name>yarn.resourcemanager.cluster-id</name>
      <value>yarn</value>
  </property>
  <property>
      <name>yarn.resourcemanager.ha.rm-ids</name>
      <value>rm1,rm2</value>
  </property>
  <property>
      <name>yarn.resourcemanager.hostname.rm1</name>
      <value>worker01</value>
  </property>
  <property>
      <name>yarn.resourcemanager.hostname.rm2</name>
      <value>worker02</value>
  </property>
  <property>
      <name>yarn.resourcemanager.webapp.address.rm1</name>
      <value>worker01:8088</value>
  </property>
  <property>
      <name>yarn.resourcemanager.webapp.address.rm2</name>
      <value>worker02:8088</value>
  </property>
  <property>
      <name>yarn.resourcemanager.zk-address</name>
      <value>master:2181,worker01:2181,worker02:2181</value>
  </property>
  <property>
      <name>yarn.resourcemanager.recovery.enabled</name>
      <value>true</value>
  </property>
  <property>
      <name>yarn.resourcemanager.store.class</name>
      <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
  </property>
</configuration>
```

- 5.5、`mapred-site.xml`。

```xml
<configuration>
  <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
  </property>
</configuration>
```

- 5.6、`slaves`。

```
master
worker01
worker02
```

- 配置所有从属节点的主机名或 `IP` 地址，每行一个。所有从属节点上的 `DataNode`服务和 `NodeManager` 服务都会被启动。
- 注意不要有空格。

### 第七步

**配置压缩包**

**（三台机器都要这样操作）**

把` hadoop-native-64-2.7.0.tar` 解压到 `hadoop-2.7.2/lib/native` 和 `hadoop-2.7.2/lib` 目录下。

（之前搭建伪分布式的时候，出现不能加载本地库的情况，这里在这里就直接把这一步给用上了，这里也可以不先这样操作，加载有问题后，在处理也一样。）

这里还有一步操作，那就是配置文件 `etc/profile` 里加：

```config
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
```

### 第八步

**第一次启动**

- 1、启动 `ZooKeeper`。

（三台机器都要启动）

```xshell
zkServer.sh start
```

- 2、启动 `Journalnode`。

 （三台机器都要启动）

```xshell
hadoop-daemon.sh start journalnode
```

- 3、初始化 `NameNode`。

 （`master`上操作）

```xshell
 hdfs namenode -format
```

- 4、初始化之后。

（执行初始化命令后，需要将 `NameNode` 元数据目录的内容，复制到其他未格式化的 `NameNode` 上。元数据存储目录就是我们在 `hdfs-site.xml` 中使用 `dfs.namenode.name.dir` 属性指定的目录。）

```xshell
scp -r /home/hadoop/namenode/data worker01:/home/hadoop/namenode/
```

- 5、初始化`HA`状态。

（任意一台机器）

```xshell
hdfs zkfc -formatZK
```

（在任意一台 `NameNode` 上使用以下命令来初始化 `ZooKeeper` 中的 `HA` 状态）

- 6、启动 `HDFS`。

 （`master`上操作）

```xshell
 start-dfs.sh
```

- 7、启动 `YARN`。

 （`worker01`上操作）

```xshell
 start-yarn.sh
```

（需要注意的是，这个时候 `worker02` 上的 `ResourceManager` 服务通常是没有启动的，需要手动启动）

```xshell
yarn-daemon.sh start resourcemanager
```

### 第九步

**第二次启动**

- 1、启动 `ZooKeeper` 。

（三台机器都要启动）

```xshell
zkServer.sh start 
```

- 2、`master`上启动 `HDFS`。

```xshell
 start-dfs.sh
```

- 3、`worker01` 上启动 `YARN`。

```xshell
 start-yarn.sh
```

- 4、`worker02` 上启动 `ResourceManager`。

```xshell
 yarn-daemon.sh start resourcemanager
```




