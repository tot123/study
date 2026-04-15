下面是基于你整个过程整理的一份**可复用、可落地的 Hadoop 单机安装文档（最终稳定版）**。
我已经帮你把踩过的坑（Java17、环境污染、YARN 启动失败等）全部固化进去了。

---

# 🧱 Hadoop 单机伪分布式安装文档（最终版）

> 适用：Debian / Ubuntu
> Hadoop：3.3.6
> Java：OpenJDK 17（含兼容修复）

---

# 一、环境准备

## 1️⃣ 创建专用用户

```bash
sudo groupadd hadoop
sudo useradd -g hadoop -m -s /bin/bash hadoop
sudo passwd hadoop
```

切换用户：

```bash
su - hadoop
```

---

## 2️⃣ 安装基础依赖

```bash
sudo apt update
sudo apt install -y curl tar gzip openssh-server rsync
```

---

## 3️⃣ 安装 Java（关键）

```bash
sudo apt install -y openjdk-17-jdk
```

验证：

```bash
java -version
```

---

## 4️⃣ 获取 JAVA_HOME

```bash
readlink -f $(which java)
```

例如：

```bash
/usr/lib/jvm/java-17-openjdk-amd64/bin/java
```

👉 JAVA_HOME：

```bash
/usr/lib/jvm/java-17-openjdk-amd64
```

---

# 二、安装 Hadoop

## 1️⃣ 下载

```bash
cd ~
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
```

---

## 2️⃣ 解压

```bash
tar -xzf hadoop-3.3.6.tar.gz
mv hadoop-3.3.6 hadoop
```

最终目录：

```bash
/home/hadoop/hadoop
```

---

# 三、环境变量配置

编辑：

```bash
vim ~/.bashrc
```

追加：

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export HADOOP_HOME=/home/hadoop/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:/usr/bin:/bin

export HDFS_NAMENODE_USER=hadoop
export HDFS_DATANODE_USER=hadoop
export HDFS_SECONDARYNAMENODE_USER=hadoop
export YARN_RESOURCEMANAGER_USER=hadoop
export YARN_NODEMANAGER_USER=hadoop
```

生效：

```bash
source ~/.bashrc
```

---

# 四、配置 SSH 免密

```bash
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

测试：

```bash
ssh localhost
```

---

# 五、配置 Hadoop

进入：

```bash
cd $HADOOP_HOME/etc/hadoop
```

---

## 1️⃣ hadoop-env.sh（⚠️ 关键）

```bash
vim hadoop-env.sh
```

加入：

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# 解决 Java17 兼容问题（必须）
export HADOOP_OPTS="$HADOOP_OPTS \
--add-opens=java.base/java.lang=ALL-UNNAMED \
--add-opens=java.base/java.lang.invoke=ALL-UNNAMED \
--add-opens=java.base/java.util=ALL-UNNAMED"
```

---

## 2️⃣ core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>

    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/tmp</value>
    </property>
</configuration>
```

---

## 3️⃣ hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/hadoop/dfs/name</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/hadoop/dfs/data</value>
    </property>
</configuration>
```

---

## 4️⃣ mapred-site.xml

```bash
cp mapred-site.xml.template mapred-site.xml
```

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

---

## 5️⃣ yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,PATH</value>
    </property>
</configuration>
```

---

## 6️⃣ workers

```bash
echo localhost > workers
```

---

# 六、初始化 HDFS

```bash
mkdir -p ~/dfs/name ~/dfs/data ~/tmp
hdfs namenode -format
```

⚠️ 只执行一次

---

# 七、启动脚本（推荐）

```bash
mkdir -p ~/scripts
```

创建统一控制脚本：

```bash
cat > ~/scripts/hadoop.sh <<'EOF'
#!/usr/bin/env bash

export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export HADOOP_HOME=/home/hadoop/hadoop
export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:/usr/bin:/bin

case "$1" in
  start)
    hdfs --daemon start namenode
    hdfs --daemon start datanode
    hdfs --daemon start secondarynamenode
    yarn --daemon start resourcemanager
    yarn --daemon start nodemanager
    ;;
  stop)
    yarn --daemon stop nodemanager
    yarn --daemon stop resourcemanager
    hdfs --daemon stop datanode
    hdfs --daemon stop secondarynamenode
    hdfs --daemon stop namenode
    ;;
  restart)
    $0 stop
    sleep 2
    $0 start
    ;;
  status)
    jps
    ;;
  *)
    echo "用法: $0 {start|stop|restart|status}"
    ;;
esac
EOF
```

赋权：

```bash
chmod +x ~/scripts/hadoop.sh
```

---

# 八、启动

```bash
~/scripts/hadoop.sh start
```

查看：

```bash
jps
```

---

## ✅ 正常输出

```bash
NameNode
DataNode
SecondaryNameNode
ResourceManager
NodeManager
Jps
```

---

# 九、验证

## 1️⃣ HDFS

```bash
hdfs dfs -mkdir -p /user/hadoop
echo "hello hadoop" > /tmp/test.txt
hdfs dfs -put -f /tmp/test.txt /user/hadoop/
hdfs dfs -cat /user/hadoop/test.txt
```

---

## 2️⃣ YARN

```bash
yarn node -list
```

---

## 3️⃣ MapReduce

```bash
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 2 10
```

---

# 十、Web UI

* NameNode： [http://localhost:9870](http://localhost:9870)
* YARN： [http://localhost:8088](http://localhost:8088)

---

# 🚨 常见问题总结

## ❌ NodeManager 启动失败

👉 原因：Java17 模块限制
👉 解决：必须加

```bash
--add-opens=java.base/java.lang=ALL-UNNAMED
```

---

## ❌ jps 找不到

👉 原因：JDK 不完整
👉 解决：安装 `openjdk-17-jdk`

---

## ❌ hadoop 命令找不到

👉 原因：PATH 未配置
👉 解决：检查 `HADOOP_HOME/bin`

---

## ❌ 环境变量污染

👉 避免：

* sdkman
* nvm
* root PATH

---

# ✅ 最终结论

你这套环境最终稳定状态是：

* ✔ Hadoop 3.3.6
* ✔ Java 17（带 add-opens 修复）
* ✔ 单机伪分布式
* ✔ 自定义控制脚本
* ✔ 环境隔离干净

---

# 🚀 下一步建议

你现在可以继续：

* HBase（依赖 Zookeeper）
* Hive（+ MySQL）
* Spark（Standalone 或 YARN）

---

如果你要，我可以下一步直接给你：

👉 **Hadoop + HBase 一键脚本（不会踩坑版）**
