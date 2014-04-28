Spark
=====

Cluster Mode 
------------

* [Cluster Mode Overview](http://spark.apache.org/docs/latest/cluster-overview.html)

### Standalone

### Apache Mesos

### Hadoop YARN

* [Launching Spark on YARN](http://spark.apache.org/docs/latest/running-on-yarn.html)

#### Building a YARN-Enabled Assembly JAR

##### Download Spark
  - http://spark.apache.org/downloads.html

##### Build

```shell
SPARK_HADOOP_VERSION=2.2.0 SPARK_YARN=true sbt/sbt assembly

tar -czvf spark-0.9.1-on-hadoop-2.2.0.tgz bin conf assembly/target/scala-2.10/spark-assembly-0.9.1-hadoop2.2.0.jar ./examples/target/scala-2.10/spark-examples-assembly-0.9.1.jar

```

##### Distribute Spark to Cluster

* Scp to all datanodes local filesystem

* Upload to HDFS



#### YARN Client Mode

YARN client mode, which submits the Spark application to YARN, and runs the Spark driver in the client Spark process that submits the application.

#### YARN Cluster Mode

YARN cluster mode, which submits the Spark application to YARN, and runs the Spark driver in the ApplicationMaster in YARN.

#### Running APP in YARN

For both YARN client and YARN cluster modes, first upload the Spark assembly JAR to your HDFS. Then set the SPARK_JAR environment variable to this HDFS path.

```shell
$ source /etc/spark/conf/spark-env.sh
$ hdfs dfs -mkdir -p /user/spark/share/lib
$ hdfs dfs -put target/scala-2.10/spark-assembly-0.9.1-hadoop2.2.0.jar /user/spark/share/lib/
$ SPARK_JAR=hdfs://master01:9000/user/spark/share/lib/spark-assembly-0.9.1-hadoop2.2.0.jar

```

##### YARN Client Mode

```shell
# step into spark dir
export HADOOP_CONF_DIR=/opt/hadoop/hadoop-2.2.0/etc/hadoop
export YARN_CONF_DIR=/opt/hadoop/hadoop-2.2.0/etc/hadoop

# Prepare the classpath
$ source /etc/spark/conf/spark-env.sh
$ SPARK_CLASSPATH=/your/additional/classpath
$ SPARK_JAR=hdfs://<nn>:<port>/user/spark/share/lib/spark-assembly.jar
$ $SPARK_HOME/bin/spark-class [<spark-config-options>]  \
    org.apache.spark.examples.SparkPi yarn-client 10


$ # or

export HADOOP_CONF_DIR=/opt/hadoop/hadoop-2.2.0/etc/hadoop
export YARN_CONF_DIR=/opt/hadoop/hadoop-2.2.0/etc/hadoop

SPARK_JAR=hdfs://master01:9000/user/spark/share/lib/spark-assembly-0.9.1-hadoop2.2.0.jar \
APP_JAR=hdfs://master01:9000/user/spark/share/lib/spark-examples-assembly-0.9.1.jar \

SPARK_JAR=hdfs://master01:9000/user/spark/share/lib/spark-assembly-0.9.1-hadoop2.2.0.jar \
SPARK_YARN_APP_JAR=hdfs://master01:9000/user/spark/share/lib/spark-examples-assembly-0.9.1.jar \
bin/run-example org.apache.spark.examples.SparkPi yarn-client


SPARK_JAR=./assembly/target/scala-2.10/spark-assembly-0.9.1-hadoop2.2.0.jar \
SPARK_YARN_APP_JAR=examples/target/scala-2.10/spark-examples-assembly-0.9.1.jar \
bin/run-example org.apache.spark.examples.SparkPi yarn-client

SPARK_YARN_MODE=true \
SPARK_JAR=./assembly/target/scala-2.10/spark-assembly-0.9.1-hadoop2.2.0.jar \
SPARK_YARN_APP_JAR=examples/target/scala-2.10/spark-examples-assembly-0.9.1.jar \
MASTER=yarn-client bin/spark-shell

SPARK_JAR=hdfs://master01:9000/user/spark/share/lib/spark-assembly-0.9.1-hadoop2.2.0.jar \
SPARK_YARN_APP_JAR=hdfs://master01:9000/user/spark/share/lib/spark-examples-assembly-0.9.1.jar \
bin/spark-class org.apache.spark.examples.SparkPi
--jar hdfs://master01:9000/user/spark/share/lib/spark-examples-assembly-0.9.1.jar yarn-client 10
bin/spark-class --jar examples/target/scala-2.10/spark-examples-assembly-0.9.1.jar --class org.apache.spark.examples.SparkPi yarn-client 10
bin/spark-class --jar examples/target/scala-2.10/spark-examples-assembly-0.9.1.jar  org.apache.spark.examples.SparkPi yarn-client 10

```

you can launch the Spark shell in YARN client mode as follows:

```shell
SPARK_JAR=hdfs://master01:9000/user/spark/share/lib/spark-assembly-0.9.1-hadoop2.2.0.jar \
SPARK_YARN_APP_JAR=hdfs://master01:9000/user/spark/share/lib/spark-examples-assembly-0.9.1.jar \
MASTER=yarn-client \
bin/spark-shell
```

##### YARN Cluster Mode

```shell
# ~~ source /path/to/spark/conf/spark-env.sh~~ 
# step into spark dir
export HADOOP_CONF_DIR=/opt/hadoop/hadoop-2.2.0/etc/hadoop
export YARN_CONF_DIR=/opt/hadoop/hadoop-2.2.0/etc/hadoop

SPARK_JAR=hdfs://master01:9000/user/spark/share/lib/spark-assembly-0.9.1-hadoop2.2.0.jar \
APP_JAR=hdfs://master01:9000/user/spark/share/lib/spark-examples-assembly-0.9.1.jar \
bin/spark-class org.apache.spark.deploy.yarn.Client \
	--jar $APP_JAR
	--class org.apache.spark.examples.SparkPi
	--args yarn-standalone
	--args 10

SPARK_JAR=hdfs://master01:9000/user/spark/share/lib/spark-assembly-0.9.1-hadoop2.2.0.jar \
APP_JAR=hdfs://master01:9000/user/spark/share/lib/spark-examples-assembly-0.9.1.jar \
bin/spark-class org.apache.spark.deploy.yarn.Client \
	--jar $APP_JAR  \
	--class org.apache.spark.examples.SparkPi \
	--args yarn-standalone \
	--num-workers 2 \
	--master-memory 385m \
	--worker-memory 385m \
	--worker-cores 1
```

Streaming
---------

