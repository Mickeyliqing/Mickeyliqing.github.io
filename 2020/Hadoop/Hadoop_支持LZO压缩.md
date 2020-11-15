# Hadoop_支持LZO压缩
## 第一步：安装 `LZO`库。
**（`hadoop` 用户在 `home` 目录下新建的文件夹 `LZO`，以下所有的安装均在这个目录下）**

- 安装`lzo`：`wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz` 。
- 解压。
- 进入解压目录。
- 编译参数。
   1. 本例安装在 `/home/LZO/lzo2.10` 目录下（`lzo2.10`）为新建文件夹。
   2. 编译命令 `./configure --enable-shared --prefix=/home/LZO/lzo2.10` 。
- 编译安装：`make && make install` 。
- 进入安装目录 `/home/LZO/lzo2.10` 。
- 执行命令。

```xshell
    cp -r lib/* /usr/lib/
    cp -r lib/* /home/hadoop/hadoop/hadoop-2.7.2/lib/native/
```

**注意**

- 64位的系统执行如下命令：

```xshell
    cp -r lib/* /usr/lib64/
    cp -r lib/* /home/hadoop/hadoop/hadoop-2.7.2/lib/native/Linux-amd64-64/
```

- 教程里执行的命令是：`cp -r lib/* /home/hadoop/hadoop/hadoop-2.7.2/lib/native/Linux-i386-32/` 但是我没找到 `Linux-i386-32` 这个目录。

## 第二步：安装 `hadoop-lzo` 包。

- 安装 `hadoop-lzo`： `wget https://github.com/twitter/hadoop-lzo/archive/master.zip --no-check-certificate  -O master.zip` 。
- 解压 `unzip master.zip`。
- 编译。
   1. 修改目录 `hadoop-lzo-master` 里的 `pom.xml` ,把 `hadoop.current.version` 的属性修改成自己使用版本的。
   2. 进入 `hadoop-lzo-master`目录，执行：

```
    export C_INCLUDE_PATH=/home/LZO/lzo-2.10/include
    export LIBRARY_PATH=/home/LZO/lzo-2.10/lib
```

然后编译 `mvn clean package -Dmaven.test.skip=true`。`

把编译好的文件拷贝到对应目录：

```
    cp target/native/Linux-amd64-64/*  /home/hadoop/hadoop/hadoop-2.7.2/lib/native/
    cp target/hadoop-lzo-0.4.21-SNAPSHOT.jar /home/hadoop/hadoop/hadoop-2.7.2/share/hadoop/mapreduce/lib
```

**注意**

- 这里用到了 `Maven`，首先先保证 `Maven` 已经安装，并配置成功。

- 当执行 `mvn clean package -Dmaven.test.skip=true` 命令出错的时候，试在清空 `Maven` 加载架包生成的目录。

## 第三步：修改配置文件。

- 在 `Hadoop` 中的 `/home/hadoop/hadoop/hadoop-2.7.2/etc/hadoop/hadoop-env.sh` 加上
 ` export LIBRARY_PATH=/home/LZO/lzo-2.10/lib` 。
- 在 `Hadoop` 中的 `/home/hadoop/hadoop/hadoop-2.7.2/etc/hadoop/core-site.xml` 加上。

```xml
<property>
    <name>io.compression.codecs</name>
    <value>org.apache.hadoop.io.compress.GzipCodec,
           org.apache.hadoop.io.compress.DefaultCodec,
           com.hadoop.compression.lzo.LzoCodec,
           com.hadoop.compression.lzo.LzopCodec,
           org.apache.hadoop.io.compress.BZip2Codec
        </value>
</property>

<property>
    <name>io.compression.codec.lzo.class</name>
    <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>
```

- 在 `Hadoop` 中的 `/home/hadoop/hadoop/hadoop-2.7.2/etc/hadoop/mapred-site.xml` 加上。

```xml
<property>
    <name>mapred.compress.map.output</name>
    <value>true</value>
</property>

<property>
    <name>mapred.map.output.compression.codec</name>
    <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>

<property>
    <name>mapred.child.env</name>
    <value>LD_LIBRARY_PATH=/home/LZO/lzo-2.10/lib</value>
</property>
```

## 第四步：验证。

在 `Hive` 里新建一个表：

```sql
create table lzo(
    id int,
    name string)
    STORED AS INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
    OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat';
```

没有报错，及初步成功。

