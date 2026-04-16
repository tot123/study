下面是**按你本次实际操作过程整理的完整安装文档（可复用）**，已去掉坑点，直接照做即可。

---

# 📘 Kafka + Spark + DolphinScheduler 安装文档（单机版）

## 🧱 一、环境信息

```text
用户：hadoop
主机名：debian188
安装目录：/home/hadoop/local
软件包目录：/home/hadoop/bigdata/software
JDK：11
已安装：Hadoop + HBase
```

---

# ⚙️ 二、基础目录准备

```bash
mkdir -p /home/hadoop/bigdata/software
mkdir -p /home/hadoop/local
mkdir -p /home/hadoop/logs/{kafka,spark,dolphinscheduler}
mkdir -p /home/hadoop/run/{kafka,spark,dolphinscheduler}
```

---

# ☕ 三、JDK 配置

```bash
cat >> ~/.bashrc <<'EOF'
export JAVA_HOME=/home/hadoop/local/java/jdk-11.0.30+7
export PATH=$JAVA_HOME/bin:$PATH
EOF

source ~/.bashrc
```

验证：

```bash
java -version
```

---

# 🚀 四、Kafka 3.9.x 安装（KRaft）

## 1）下载

```bash
cd /home/hadoop/bigdata/software

wget https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/3.9.2/kafka_2.13-3.9.2.tgz
```

---

## 2）解压

```bash
tar -xzf kafka_2.13-3.9.2.tgz -C /home/hadoop/local
ln -sfn /home/hadoop/local/kafka_2.13-3.9.2 /home/hadoop/local/kafka
```

---

## 3）配置环境变量

```bash
cat >> ~/.bashrc <<'EOF'
export KAFKA_HOME=/home/hadoop/local/kafka
export PATH=$KAFKA_HOME/bin:$PATH
EOF

source ~/.bashrc
```

---

## 4）修改配置（KRaft）

```bash
mkdir -p /home/hadoop/local/kafka/data/kraft-logs

sed -i 's|^log.dirs=.*|log.dirs=/home/hadoop/local/kafka/data/kraft-logs|' $KAFKA_HOME/config/kraft/server.properties

sed -i 's|^#*listeners=.*|listeners=PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093|' $KAFKA_HOME/config/kraft/server.properties

sed -i 's|^#*advertised.listeners=.*|advertised.listeners=PLAINTEXT://debian188:9092|' $KAFKA_HOME/config/kraft/server.properties

sed -i 's|^#*controller.listener.names=.*|controller.listener.names=CONTROLLER|' $KAFKA_HOME/config/kraft/server.properties

sed -i 's|^#*inter.broker.listener.name=.*|inter.broker.listener.name=PLAINTEXT|' $KAFKA_HOME/config/kraft/server.properties
```

---

## 5）初始化

```bash
KAFKA_CLUSTER_ID="$($KAFKA_HOME/bin/kafka-storage.sh random-uuid)"

$KAFKA_HOME/bin/kafka-storage.sh format \
  -t "$KAFKA_CLUSTER_ID" \
  -c $KAFKA_HOME/config/kraft/server.properties
```

---

## 6）启动

```bash
nohup $KAFKA_HOME/bin/kafka-server-start.sh \
  $KAFKA_HOME/config/kraft/server.properties \
  > /home/hadoop/logs/kafka/server.log 2>&1 &
```

---

## 7）验证

```bash
jps | grep Kafka
ss -lntp | grep 9092
```

---

## 8）测试

```bash
kafka-topics.sh --bootstrap-server debian188:9092 --create --topic test-topic --partitions 1 --replication-factor 1
```

---

# ⚡ 五、Spark 3.5.8 安装

## 1）下载

```bash
cd /home/hadoop/bigdata/software

wget https://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-3.5.8/spark-3.5.8-bin-hadoop3.tgz
```

---

## 2）解压

```bash
tar -xzf spark-3.5.8-bin-hadoop3.tgz -C /home/hadoop/local
ln -sfn /home/hadoop/local/spark-3.5.8-bin-hadoop3 /home/hadoop/local/spark
```

---

## 3）配置

```bash
cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
cp $SPARK_HOME/conf/workers.template $SPARK_HOME/conf/workers
```

```bash
cat > $SPARK_HOME/conf/spark-env.sh <<'EOF'
export JAVA_HOME=/home/hadoop/local/java/jdk-11.0.30+7
export HADOOP_HOME=/home/hadoop/hadoop
export HADOOP_CONF_DIR=/home/hadoop/hadoop/etc/hadoop
export SPARK_MASTER_HOST=debian188
EOF
```

```bash
echo debian188 > $SPARK_HOME/conf/workers
```

---

## 4）启动

```bash
$SPARK_HOME/sbin/start-master.sh
$SPARK_HOME/sbin/start-worker.sh spark://debian188:7077
```

---

## 5）验证

```bash
jps | egrep 'Master|Worker'
ss -lntp | egrep '7077|8080|8081'
```

---

## 6）测试任务

```bash
spark-submit \
  --master spark://debian188:7077 \
  --class org.apache.spark.examples.SparkPi \
  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.8.jar 10
```

---

# 🔄 六、Spark + Kafka 联调

```bash
spark-shell \
  --master spark://debian188:7077 \
  --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.8
```

执行：

```scala
val df = spark.readStream.format("kafka")
  .option("kafka.bootstrap.servers","debian188:9092")
  .option("subscribe","test-topic")
  .load()

val lines = df.selectExpr("CAST(value AS STRING)")

lines.writeStream.format("console").start().awaitTermination()
```

---

# 🐬 七、DolphinScheduler 3.4.1 安装

## 1）下载

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/apache/dolphinscheduler/3.4.1/apache-dolphinscheduler-3.4.1-bin.tar.gz
```

---

## 2）解压

```bash
tar -xzf apache-dolphinscheduler-3.4.1-bin.tar.gz -C /home/hadoop/local
ln -sfn /home/hadoop/local/apache-dolphinscheduler-3.4.1-bin /home/hadoop/local/dolphinscheduler
```

---

## 3）启动（关键）

⚠️ 注意：3.4.1 已改变启动方式

```bash
chmod -R +x /home/hadoop/local/dolphinscheduler/standalone-server/bin

cd /home/hadoop/local/dolphinscheduler/standalone-server
./bin/start.sh
```

---

## 4）验证

```bash
jps | grep StandaloneServer
ss -lntp | grep 12345
```

---

## 5）访问

```text
http://debian188:12345
```

账号：

```text
admin / dolphinscheduler123
```

---

# 🎯 八、调度 Spark 任务

DolphinScheduler → 新建 Shell 任务：

```bash
/home/hadoop/local/spark/bin/spark-submit \
  --master spark://debian188:7077 \
  --class org.apache.spark.examples.SparkPi \
  /home/hadoop/local/spark/examples/jars/spark-examples_2.12-3.5.8.jar 10
```

---

# 🧩 九、最终架构

```text
Kafka → Spark → DolphinScheduler
```

---

# 🛠 十、统一控制脚本

```bash
bdctl start
bdctl stop
bdctl restart
bdctl status
```

---

# ⚠️ 常见坑总结（你这次踩过的）

| 问题                    | 原因        |
| --------------------- | --------- |
| Spark 报 selectExpr 错误 | REPL 换行问题 |
| DS 启动失败               | 用错脚本      |
| start.sh 权限不足         | 未 chmod   |
| Kafka 启动异常            | 未 format  |

---

# ✅ 完成状态

你当前已经实现：

```text
✔ Kafka（KRaft）
✔ Spark（Standalone）
✔ Spark Streaming（Kafka）
✔ DolphinScheduler（Standalone）
✔ 调度能力
```

---

# 🚀 下一步建议（生产）

如果你要继续，我可以带你升级：

* Spark on YARN
* Kafka 集群（3节点）
* DolphinScheduler + MySQL
* Prometheus + Grafana

直接说一句：**上生产版** 👍
