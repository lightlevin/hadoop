Hadoop2(Yarn)
=============

Preparations
------------

* add hosts to file /etc/hosts

```
192.168.31.201  master01
192.168.31.202  slave01
192.168.31.203  slave02
```

* add users

```
adduser hadoop
adduser yarn
vi /etc/group  #add yarn to group hadoop
```

* JDK

```
# install jdk and add related env var to /etc/profile
export JAVA_HOME=/opt/jdk
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH 
```

* Directory

```
# hadoop home
mkdir -p /opt/hadoop
chown hadoop:hadoop /opt/hadoop
su - hadoop
cd /opt/hadoop
aira2c http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0.tar.gz
tar -zxvf hadoop-2.2.0.tar.gz

export HADOOP_HOME=/opt/hadoop/hadoop-2.2.0/
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
	
# the following two var not work for som reason
# use JAVA_LIBRARY_PATH instead
# export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
# export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export JAVA_LIBRARY_PATH=/opt/hadoop/hadoop-2.2.0/lib/native:$JAVA_LIBRARY_PATH


# hdfs dir
mkdir -p /opt/hadoop-data/dfs/name
mkdir -p /opt/hadoop-data/dfs/data
chown -R hadoop:hadoop /opt/hadoop-data

# yarn dir
mkdir -p /opt/yarn/local
mkdir -p /opt/yarn/logs
chown -R yarn:yarn /opt/yarn

```


* configure passwordless login

```
# login as hadoop
ssh-keygen -t  rsa
.ssh/id_rsa.pub >> .ssh/authorized_keys 
scp .ssh/authorized_keys slave01:~/.ssh/
scp .ssh/authorized_keys slave02:~/.ssh/

# execute on all hosts
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

#test
ssh slave01
ssh slave02

```

* compile hadoop 64-bit native lib

```
# install tools
apt-get install build-essential g++ autoconf automake libtool cmake zlib1g-dev pkg-config libssl-dev  maven

# install protobuf 2.5

wget http://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
cd protobuf-2.5.0
./configure --prefix=/usr/local && make && make install
vi /etc/ld.so.conf.d/usr_local_lib.conf # add /usr/local/lib directory 
ldconfig -v

aria2c "http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0-src.tar.gz"
aria2c "http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0-src.tar.gz.mds"
md5sum hadoop-2.2.0-src.tar.gz
cat hadoop-2.2.0-src.tar.gz.mds # make sure checksum is ok 
cd hadoop-2.2.0-src

# apply patch
wget https://issues.apache.org/jira/secure/attachment/12614482/HADOOP-10110.patch
patch -p0 < HADOOP-10110.patch

mvn package -Pdist,native -DskipTests -Dtar

# save these files in hadoop-dist/target/hadoop-2.2.0/lib/native to replace 32-bit files
```

Configuration
-------------

* download hadoop binary files and untar to /opt/hadoop

* update the lib/native file to 64-bit lib

* hadoop-env.sh  yarn-env.sh mapred-env.sh

```
# add env var JAVA_HOME
export JAVA_HOME=/opt/jdk

```

* core-site.xml

```xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master01:9000/</value>
                <description>The name of the default file system. A URI whose scheme and authority determine the FileSystem implementation. The uri's scheme determines the config property (fs.SCHEME.impl) naming the FileSystem implementation class. The uri's authority is used to determine the host, port, etc. for a filesystem.</description>
        </property>

        <property>
                <name>dfs.permissions.superusergroup</name>
                <value>hadoop</value>
        </property>
        <property>
                <name>hadoop.proxyuser.hduser.hosts</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.hduser.groups</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.native.lib</name> 
                <value>true</value>
                <description>Should native hadoop libraries, if present, be used.</description>
        </property>
</configuration>
```

* hdfs-site.xml

```xml
<configuration>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:///opt/hadoop-data/dfs/name</value>
                <description>Determines where on the local filesystem the DFS name node should store the name table.If this is a comma-delimited list of directorie
s,then name table is replicated in all of the directories,for redundancy.</description>
        </property>
   
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:///opt/hadoop-data/dfs/data</value>
                <description>Determines where on the local filesystem an DFS data node should store its blocks.If this is a comma-delimited list of directories,then data will be stored in all named directories,typically on different devices.Directories that do not exist are ignored.
                </description>
        </property>
   
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
        <property>
                <name>dfs.permission</name>
                <value>true</value>
        </property>
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property> 
</configuration>

```

* mapred-site.xml

```xml
<configuration>
        <property>
                 <name>mapreduce.framework.name</name>
                 <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>master01:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>master01:19888</value>
        </property>
        <property>
                <name>yarn.app.mapreduce.am.staging-dir</name>
                <value>/user</value>
        </property>
</configuration>

```

* yarn-site.xml

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>master01:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>master01:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>master01:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>master01:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>master01:8088</value>
        </property>
        <property>
                <description>Classpath for typical applications.</description>
                <name>yarn.application.classpath</name>
                <value>
                        $HADOOP_CONF_DIR,
                        $HADOOP_COMMON_HOME/*,$HADOOP_COMMON_HOME/lib/*,
                        $HADOOP_HDFS_HOME/*,$HADOOP_HDFS_HOME/lib/*,
                        $HADOOP_MAPRED_HOME/*,$HADOOP_MAPRED_HOME/lib/*,
                        $YARN_HOME/*,$YARN_HOME/lib/*
                 </value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>

        <property>
                <name>yarn.nodemanager.local-dirs</name>
                <value>file:///opt/yarn/local</value>
        </property>
        <property>
                <name>yarn.nodemanager.log-dirs</name>
                <value>file:///opt/yarn/logs</value>
        </property>
        <property>
                <description>Where to aggregate logs</description>
                <name>yarn.nodemanager.remote-app-log-dir</name>
                <value>hdfs://var/log/hadoop-yarn/apps</value>
        </property>

</configuration>

```

* salves

```
slave01
slave02
```

Copy files to Slaves
--------------------


```shell
cd /opt/hadoop
scp -r hadoop-2.2.0 slave01:/opt/hadoop
scp -r hadoop-2.2.0 slave02:/opt/hadoop

```

Start HDFS and Yarn
-------------------

```shell
# format hdfs
hdfs namenode -format

# start hdfs
start-dfs.sh

# start yarn
start-yarn.sh

# check processes
jps

#.......

#stops
stop-yarn.sh
stop-dfs.sh

```

* Web UI

http://master01:8088/cluster

http://master01:50070



