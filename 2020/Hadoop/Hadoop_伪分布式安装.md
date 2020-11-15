# Hadoop伪分布式安装
## 1：添加 `hadoop` 用户
新增一个用户 `hadoop` ,以后就用 `hadoop` 这个用户对 `Hadoop` 进行管理。

- 新增`hadoop` 用户 ：`adduser hadoop`。
- 为`hadoop` 用户添加管理员权限：赋值`root` 权限--->修改 （`vim /etc/sudoers`）,找到下面一行`root  ALL =（ALL）ALL` 下添加 `hadoop ALL = (ALL) ALL`。

**执行完这个操作，接下来的命令就切换到 `hadoop` 用户下进行。**

## 2： 配置环境

- 安装`shh` 服务器：`sudo apt-get install openssh-server`
- 登录本机：`ssh localhost`

输入密码，登录成功，之后执行`exit` 退出登录。

- 设置无密登录
  1. `cd ~/.ssh/` 若⽬录不存在，则再次执⾏ `ssh localhost`。
  2. `ssh-keygen -t rsa` 不⽤管提⽰，⼀直按回⻋。
  3. `cat ./id_rsa.pub >> ./authorized_keys` 加⼊授权。
  4. `ssh localhost` 测试⽆密码登录。

执行第三步的时候，最好看下当前的目录，如果当前目录在 `/.ssh` 下。那么只需要执行`cat id_rsa.pub >> authorized_keys` 即可。

查看当前目录下下的文件命令：`ls -a`。

## 3：安装 `JDK`

1：上传文件到指定位置。

2：解压。

3：修改系统配置（`vim /etc/profile`）。

```config
export JAVA_HOME=/home/JDK/jdk1.8.0_191
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
```
4：配置生效 `source /etc/profile`。

5：验证` java -version`。

## 4：安装 `Hadoop`

- 下载对应版本的`hadoop`，放在指定的目录下，然后解压。
- 修改文件夹属性：`sudo chown hadoop:hadoop -R /home/hadoop`。

文件解压放在了`/home/hadoop` 目录下。

- 检查是否安装成功：进入解压后的目录，执行：`./bin/hadoop version`。

文件目录为：`/home/hadoop/hadoop-2.7.2`。

## 5：配置伪分布式

(注意文件目录为：`/home/hadoop/hadoop-2.7.2`)

- `core-site.xml`

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name> 
        <value>file:/home/hadoop/tmp</value>
    </property>

    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

- `hdfs-site.xml`

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/home/hadoop/tmp/dfs/name</value>

    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/home/hadoop/tmp/dfs/data</value>

    </property>
</configuration>
```

- `mapred-site.xml`

```xml
<configuration> 
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

- `yarn-site.xml`

```xml
<configuration> 
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

## 6：设置 `HADOOP_HOME`

- `vim /etc/profile` 在`JAVA_HOME` 下添加如下两行：

```config
 export HADOOP_HOME=/home/hadoop/hadoop-2.7.2
 export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
```

- 修改`hadoop_env.sh` 的`JAVA_HOME` 把这个值写死 （这个值就是上文配置里的`JAVA_HOME`）。

## 7：格式化 `NameNode`

- （在`HADOOP_HOME` 目录下执行）
```xshell
./bin/hdfs namenode-formar 
```

## 8：开启 `NameNode` 和`DataNode`

```xshell
./sbin/start-dfs.sh
```

## 9：关闭 `NameNode `和`DataNode`

```xshell
./sbin/stop-dfs.sh
```

## 10：启动 `YARN`

```xshell
./sbin/start-yarn.sh
```

## 11：关闭 `YARN`

```xshell
./sbin/stop-yarn.sh
```

- (可以使用` jps` 命令查看启动的进程)


