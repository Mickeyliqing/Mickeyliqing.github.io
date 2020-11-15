# Hadoop常见错误
## 1：安装`Hadoop`集群配置注意事项

- 配置`hosts`的时候注意主机名，不能使用带下划线的主机名。例如：`worker_01`，不能这样配置，要用`worker01`。配置`worker_01`，`Hadoop`集群会解析不到。
- 初次启动的时候应该严格按照启动顺序来进行。初次启动的启动顺序为：

1、启动`ZK`

```xshell
zkServer.sh start
```

2、启动`Journalnode`

```xshell
hadoop-daemon.sh start journalnode
```

3、格式化`HDFS`

```xshell
hdfs namenode -format
```

- （在`Master`上格式化，格式化成功后需要将 `NameNode` 元数据目录的内容，复制到其他未格式化的 `NameNode` 上。元数据存储目录就是我们在 `hdfs-site.xml` 中使用 `dfs.namenode.name.dir` 属性指定的目录。）

```xshell
scp -r /home/hadoop/namenode/data worker01:/home/hadoop/namenode/
```

4、格式化`ZKFC`

```xshell
hdfs zkfc -formatZK
```

5、启动`HDFS`

```xshell
start-dfs.sh
```

6、启动`YARN`

```xshell
start-yarn.sh
```

- （一般情况下`worker03` 上的`ResourceManager`启动不起来，需要单独启动）

```xshell
yarn-daemon.sh start resourcemanager
```

## 2：`Hadoop` 集群部署在了`Linux`虚拟机上，然后在`Win`上编程测试。

**2.1、（出现连接不成功的问题）(`192.168.48.129`) 是虚拟机的`IP`。**

**解决：**

- 1：代码里

```java
private static final String HDFS_PATH = "hdfs://192.168.48.129:9000";
```

写成`IP`地址和端口的形式。

- 2：配置 `core-site.xml` 里也配置成IP和端口的形式。

```xml
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://192.168.48.129:9000</value>
</property>
```


